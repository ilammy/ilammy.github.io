---
layout: post
title:  "Making TLS connections with NSURLSession"
date:   2021-02-14 12:00:00 +0900
tags:   how-to macos tls
---

### Making TLS connections with NSURLSession

Here we will explore how to use `NSURLSession` to make TLS connections
which are *not* HTTPS.

Sure, you can easily talk to servers via HTTPS, but what if you need a different protocol?
As always with frameworks,
you are in the world of hurt once you venture outside of the envisioned scope.

This is *especially* true when things come to cryptography (which TLS is).
You'd probably better off relying on the system implementations of cryptography services,
verified by security consultants for big monies.
God forbid you do your own crypto.

Accompanying source code is available here:
[~ilammy/nsurlsession-tls-example](https://git.sr.ht/~ilammy/nsurlsession-tls-example)

The information here applies to macOS and was accurate circa macOS 10.15.
It could probably work with iOS as well, but I have no clue and I don't care.

<img width=571 src="/images/2021/02-14/example-com.png" alt="Screenshot of example.com HTTP response">

#### Initializing NSURLSession

First you need to have your session up and going:

```objective-c
// You need a configuration. Start with one of the premade configurations.
// This one will keep everything in memory, as opposed to an on-disk cache.
NSURLSessionConfiguration *configuration =
    NSURLSessionConfiguration.ephemeralSessionConfiguration;

// Cut off old TLS versions. (Arguably, you can start with TLS v1.3 now.)
configuration.TLSMinimumSupportedProtocolVersion =
    tls_protocol_version_TLSv12;

// Make a session. You'll need a delegate to validate TLS certificates later.
// You can supply your own operations queue if you'd like, or let NSURLSession
// create a worker thread for you, like here.
NSURLSession *session =
    [NSURLSession sessionWithConfiguration:configuration
                                  delegate:self
                             delegateQueue:nil];
```

#### Making TCP connections

Then make a TCP connection to the server.
You need bidirectional streaming, not download some file or something.
This is what `NSURLSessionStreamTask` is for.
You make it from `NSURLSession` like this:

```objective-c
NSURLSessionStreamTask *task =
    [session streamTaskWithHostName:@"example.com"
                               port:443];
```

(Oh, you want to inherit from this class? Tough luck. Think Different™)

#### Initiating TLS handshake

This one is easy, just start the task:

```objective-c
[task startSecureConnection];
[task resume];
```

You can talk via non-TLS before you send the `startSecureConnection` message,
if you'd like to do STARTTLS-like stuff to upgrade to TLS.

#### Verifying TLS certificates

Now this is where things get interesting.
You can delegate certificate validation to the OS by doing nothing.
If the server certificate is valid and signed by a CA trusted by the system,
your connection succeeds.
Otherwise it does not.

But what if you want to validate the certificate yourself?
Suppose, you'd like to do certificate pinning,
or you'd like to bypass all this PKI monstrosity altogether
and support self-signed certificates in a TOFU manner?

Well, Apple got you covered:
[Performing Manual Server Trust Authentication](https://developer.apple.com/documentation/foundation/url_loading_system/handling_an_authentication_challenge/performing_manual_server_trust_authentication?language=objc)
and
[Overriding TLS Chain Validation Correctly](https://developer.apple.com/library/archive/documentation/NetworkingInternet/Conceptual/NetworkingTopics/Articles/OverridingSSLChainValidationCorrectly.html).

You get to inspect the certificate in the following *delegate* method:

```objective-c
- (void)    URLSession:(NSURLSession *)session
   didReceiveChallenge:(NSURLAuthenticationChallenge *)challenge
     completionHandler:(void (^)(NSURLSessionAuthChallengeDisposition disposition,
                                 NSURLCredential *credential))completionHandler
{
    // validate your certificate here and call `completionHandler`
}
```

You can also use the per-task equivalent:

```objective-c
- (void)    URLSession:(NSURLSession *)session
                  task:(NSURLSessionTask *)task  // <== NEW!
   didReceiveChallenge:(NSURLAuthenticationChallenge *)challenge
     completionHandler:(void (^)(NSURLSessionAuthChallengeDisposition disposition,
                                 NSURLCredential *credential))completionHandler
{
    // validate your certificate here and call `completionHandler`
}
```

which will be called *only if* the session-global one is not defined.

When you do validate the certificate, you call the `completionHandler` with your verdict:

  - `NSURLSessionAuthChallengeUseCredential` – assert it's okay to proceed
  - `NSURLSessionAuthChallengePerformDefaultHandling` – delegate to the system
  - `NSURLSessionAuthChallengeCancelAuthenticationChallenge` – abort connection
  - `NSURLSessionAuthChallengeRejectProtectionSpace` –
    dunno how to handle this authentication challenge, try the next one (if available)

Oh, right.
First, you need to make sure that you're dealing with server certificate authentication,
not some other form of authentication that NSURLSession may want from you:

```objective-c
//    didReceiveChallenge:(NSURLAuthenticationChallenge *)challenge

if (![challenge.protectionSpace.authenticationMethod
      isEqualToString:NSURLAuthenticationMethodServerTrust])
{
    completionHandler(NSURLSessionAuthChallengePerformDefaultHandling, nil);
    return;
}
```

#### Getting to TLS certificates

Now you'd like to get to the TLS certificates presented by the server.
After you get through the Apple-speak with *domains*, *realms*, and *protection spaces*,
here's how you inspect the certificate chain:

```objective-c
SecTrustRef trust = challenge.protectionSpace.serverTrust;
CFIndex certificateCount = SecTrustGetCertificateCount(trust);
```

Yay, Core Foundation types!
Make sure the certificate chain has enough certificates for you to dereference,
or else you're going to get a segfault.

```objective-c
SecCertificateRef certificate = SecTrustGetCertificateAtIndex(trust, 0);
```

#### Computing certificate fingerprints

Now you have the leaf certificate.
Typically, you don't want to inspect the certificate each time you see it.
You can do a thorough inspection only once
and then remember the certificate's *fingerprint* to avoid doing it again
(well, if the certificate is still within its validity timeframe).

```objective-c
@import CommonCrypto;

NSMutableData *fingerprint =
    [NSMutableData dataWithLength:CC_SHA256_DIGEST_LENGTH];

CFDataRef certificateData = SecCertificateCopyData(certificate);
CC_SHA256(CFDataGetBytePtr(certificateData),
          (CC_LONG)CFDataGetLength(certificateData),
          fingerprint.mutableBytes);
CFRelease(certificateData);
```

(Make sure to release Core Foundation types properly, or else leaks happen.)

The above snippet computes certificate fingerprint as a hash sum of the
**DER encoding** of the X.509 certificate presented by the server.
This representation has a benefit of being consistent.

This is also how OpenSSL makes fingerprints when you do

```
openssl x509 -in cert.pem -noout -sha256 -fingerprint
```

So you can cross-check your implementation.

#### Certificate fingerprints with OpenSSL

Connect to the server you need:

```
openssl s_client -connect example.com:443
```

Grab the certificate in PEM form:

```
-----BEGIN CERTIFICATE-----
MIIG1TCCBb2gAwIBAgIQD74IsIVNBXOKsMzhya/uyTANBgkqhkiG9w0BAQsFADBP

[ omitted for brevity ]

vUzLnF7QYsJhvYtaYrZ2MLxGD+NFI8BkXw==
-----END CERTIFICATE-----
```

and compute the fingerprint with `openssl x509`.

You can do it in a “one-liner”:

```
openssl s_client -connect example.com:443 < /dev/null 2>&1 \
| sed -n '/-----BEGIN CERTIFICATE-----/,/-----END CERTIFICATE-----/p' \
| openssl x509 -noout -sha256 -fingerprint
```

which at the time of writing produces the following output:

```
SHA256 Fingerprint=20:0D:CA:FA:76:7C:84:50:EC:E6:44:87:9C:06:2A:0C:DF:52:24:0F:E0:5B:B7:EB:28:46:11:C3:AE:F3:EC:2E
```

#### Accessing certificate values

Now this is where the fun part begins.
Apple's Security Framework does *not* provide you with nice and tidy accessors to certificate parts.
You have to do the meat job yourself.
Thankfully, at least you don't have to parse X.509 data yourself.

In order to get access to certificate values, you need a dictionary.
Tell the Security Framework what fields you are interested in, and you get them.

```objective-c
NSArray<NSString *> *keys = @[
    (NSString *)kSecOIDX509V1ValidityNotAfter,
    (NSString *)kSecOIDX509V1ValidityNotBefore,
];
CFDictionaryRef values = SecCertificateCopyValues(certificate,
                                                  (CFArrayRef)keys,
                                                  nil);
```

(The last argument is an output pointer `CFErrorRef*` for errors,
if you are interested in them.
If not, just check that the returned dictionary is not null.)

The dictionary is indexed first by OIDs (rendered as strings),
then by more labels providing introspection into values:

```
{
    "2.16.840.1.113741.2.1.1.1.6" = {
        label               = "Not Valid Before";
        "localized label"   = "Not Valid Before";
        type                = number;
        value               = 627868800;
    };
    "2.16.840.1.113741.2.1.1.1.7" = {
        label               = "Not Valid After";
        "localized label"   = "Not Valid After";
        type                = number;
        value               = 662169599;
    };
}
```

You can use *toll-free bridging* – what a cute name for a concept! –
in order to use Core Foundation types more easily with Objective-C.
Swift has its own interfaces which are even easier to use.

```objective-c
NSDictionary<NSString *, NSDictionary<NSString *, id>> *values =
    CFBridgingRelease(SecCertificateCopyValues(certificate,
                                               (CFArrayRef)keys,
                                               nil));
```

Just remember that you still need to take care of ownership properly.
For example, `SecCertificateCopyValues()` returns a retained pointer
so you can't use the `(__bridged NSDictionary *)` cast in this case.

#### Validating expiration dates

One of the important parts of the certificate trust is the **validity range**.
A certificate may be provisioned beforehand and as such won't be valid until a certain date.
A certificate will also expire at a certain date and won't be valid after that moment.
X.509 certificates encode those as “not valid before” and “not valid after” *extensions*.

You get to them like this via the dictionary returned by `SecCertificateCopyValues()`:

```objective-c
NSNumber *notBefore = values[(NSString *)kSecOIDX509V1ValidityNotBefore]
                            [(NSString *)kSecPropertyKeyValue];
```

(It won't hurt to check the types via the `kSecPropertyKeyType` label.
It contains a string with the type name.
Numbers have `kSecPropertyTypeNumber`, for example.)

But that's a number.
How do you get a timestamp?
Well, this *is* a timestamp, but remember: you're in Apple world so you need to **Think Different™**

The number is the number of seconds since epoch,
but Apple platforms use a different epoch – 2001-01-01T00:00:00Z –
not the UNIX epoch that all *reasonable people*™ use.

Adjust the value to get a proper timestamp like this:

```objective-c
int64_t notBeforeUNIX = (int64_t)notBefore.longLongValue // in a galaxy far far away
                      + (int64_t)kCFAbsoluteTimeIntervalSince1970;
```

(Note the type conversions.
Use *signed 64-bit values* so that you don't have to deal with the “year 2038 problem”.
Also, `CFTimeInterval` is actually `double` but certificate timestamps typically have second precision.)

Now do the same with `kSecOIDX509V1ValidityNotAfter` and you got yourself the validity range of the certificate.

You want to check it against the current time as reported by the Security Framework:

```objective-c
//    didReceiveChallenge:(NSURLAuthenticationChallenge *)challenge
//
// SecTrustRef trust = challenge.protectionSpace.serverTrust;

int64_t verifyTimeUNIX = (int64_t)SecTrustGetVerifyTime(trust)
                       + (int64_t)kCFAbsoluteTimeIntervalSince1970;
```

Make sure it stays within prescribed bounds and take care for off-by-one errors:

```objective-c
if (verifyTimeUNIX < notBeforeUNIX) {
    // Certificate is not yet valid.
    completionHandler(NSURLSessionAuthChallengeCancelAuthenticationChallenge, nil);
    return;
}
if (verifyTimeUNIX > notAfterUNIX) {
    // Certificate has expired.
    completionHandler(NSURLSessionAuthChallengeCancelAuthenticationChallenge, nil);
    return;
}
```

#### Validating domain assignment

You should also probably verify that the certificate is issued
to the entity you expect to be connecting to.
Typically a domain name is used, but there are other options which are not considered here.

X.509 certificates have a bunch of places where the domain name can be encoded.
Modern certificates typically contain a **Subject Alternative Name** extension
which specifies *a list* of the names the certificate subject goes by.
There may be one or more domain names among them.
Historically, CAs have placed the domain names into the **Common Name** field
of the certificate *subject name* field,
which typically contains a *distinguished name*.
(I say “typically” because *technically* the certificate can contain whatever.)

So, you'd want to get all those names and check whether the name of the server you're connecting to is among them.

```objective-c
NSArray<NSString *> *keys = @[
    (NSString *)kSecOIDCommonName,
    (NSString *)kSecOIDSubjectAltName,
];
NSDictionary<NSString *, NSDictionary<NSString *, id>> *values =
    CFBridgingRelease(SecCertificateCopyValues(certificate,
                                               (CFArrayRef)keys,
                                               nil));
```

For SANs, you're looking at an array of dictionaries with some values:

```
{
    "2.5.29.17" = {
        label             = "2.5.29.17";
        "localized label" = "2.5.29.17";
        type  = section;
        value = (
            {
                label               = Critical;
                "localized label"   = Critical;
                type                = string;
                value               = No;
            },
            {
                label               = "DNS Name";
                "localized label"   = "DNS Name";
                type                = string;
                value               = "www.example.org";
            },
            {
                label               = "DNS Name";
                "localized label"   = "DNS Name";
                type                = string;
                value               = "example.com";
            },
            {
                label               = "DNS Name";
                "localized label"   = "DNS Name";
                type                = string;
                value               = "example.edu";
            },
            {
                label               = "DNS Name";
                "localized label"   = "DNS Name";
                type                = string;
                value               = "example.net";
            },
            {
                label               = "DNS Name";
                "localized label"   = "DNS Name";
                type                = string;
                value               = "example.org";
            },
            {
                label               = "DNS Name";
                "localized label"   = "DNS Name";
                type                = string;
                value               = "www.example.com";
            },
            {
                label               = "DNS Name";
                "localized label"   = "DNS Name";
                type                = string;
                value               = "www.example.edu";
            },
            {
                label               = "DNS Name";
                "localized label"   = "DNS Name";
                type                = string;
                value               = "www.example.net";
            }
        );
    };
}
```

and the Common Name is an array of strings:

```
{
    "2.5.4.3" =     {
        label               = CN;
        "localized label"   = CN;
        type  = array;
        value = (
            "www.example.org"
        );
    };
}
```

Security Framework does all the heavy lifting of figuring out how to parse that from whatever the certificate contains.
Not all alternative names are domain names.
And the *Subject Name* may contain multiple *Common Names*, which may not be domain names either.

Anyhow, here's a list of DNS names present in the certificate used by `example.com`:

  - `example.com`
  - `example.edu`
  - `example.net`
  - `example.org`
  - `www.example.com`
  - `www.example.edu`
  - `www.example.net`
  - `www.example.org`

One of them is for `example.com` that we're trying to connect to.
Seems legit then.

SANs should be matched first.
Actually, it's legacy behavior and discouraged to treat the Common Name as a domain name
when there are no SANs present in the certificate.

If you're going to be serious about matching DNS names, keep the following in mind:

  - domain names are case-insensitive
  - domain names may have a trailing dot
  - certificates may contain *wildcard* domain names

Therefore if you see `*.example.com` in the list then `WHATEVER.GOES.EXAMPLE.COM.` should match.

#### What to do next

Well, it's up to you.
Once the validation is complete and assuming the connection when smoothly,
you can exchange data over the connection in privacy.
Since we're connected to `example.com` on the HTTPS port 443, why not make a request:

```objective-c
NSString *requestString =
    @"GET / HTTP/1.0\r\n"
    @"Host: example.com\r\n"
    @"Connection: close\r\n"
    @"\r\n";

[session writeData:[requestString dataUsingEncoding:NSUTF8StringEncoding]
           timeout:0
 completionHandler:^(NSError *error) {
    // ignore
}];
[session closeWrite];
```

and read the following response back:

```http
HTTP/1.0 200 OK
Accept-Ranges: bytes
Age: 540190
Cache-Control: max-age=604800
Content-Type: text/html; charset=UTF-8
Date: Sat, 13 Feb 2021 06:29:43 GMT
Etag: "3147526947"
Expires: Sat, 20 Feb 2021 06:29:43 GMT
Last-Modified: Thu, 17 Oct 2019 07:18:26 GMT
Server: ECS (oxr/830C)
Vary: Accept-Encoding
X-Cache: HIT
Content-Length: 1256
Connection: close
```

```html
<!doctype html>
<html>
<head>
    <title>Example Domain</title>

    <meta charset="utf-8" />
    <meta http-equiv="Content-type" content="text/html; charset=utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <style type="text/css">
    body {
        background-color: #f0f0f2;
        margin: 0;
        padding: 0;
        font-family: -apple-system, system-ui, BlinkMacSystemFont, "Segoe UI", "Open Sans", "Helvetica Neue", Helvetica, Arial, sans-serif;

    }
    div {
        width: 600px;
        margin: 5em auto;
        padding: 2em;
        background-color: #fdfdff;
        border-radius: 0.5em;
        box-shadow: 2px 3px 7px 2px rgba(0,0,0,0.02);
    }
    a:link, a:visited {
        color: #38488f;
        text-decoration: none;
    }
    @media (max-width: 700px) {
        div {
            margin: 0 auto;
            width: auto;
        }
    }
    </style>
</head>

<body>
<div>
    <h1>Example Domain</h1>
    <p>This domain is for use in illustrative examples in documents. You may use this
    domain in literature without prior coordination or asking for permission.</p>
    <p><a href="https://www.iana.org/domains/example">More information...</a></p>
</div>
</body>
</html>
```
