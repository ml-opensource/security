# Certificate pinning

![iOS](https://img.shields.io/badge/platform-iOS-blue)

Certificate pinning is the process of associating a host with its certificate or public key. In other words, you explicitly tell your app which certificates to accept by including them in your app bundle. This is used to prevent the "man in the middle" attacks. You can pin a certificate or its public key. The difference is that certificates expire (typically in one or two years), whereas the public keys do not necessarily. So if your certificate expires, you need to release a new version of the app with a valid certificate. You can also include future certificates in the app's bundle and prolong the refresh period.

Certificate pinning iOS implementation:

1. Obtain your SSL/TLS certificate(s) from the server and include it in you app's bundle
2. Apply certificate pinning to your NSURLSession delegate
3. Assign a delegate to the instance of NSURLSession

## 1. Obtain your certificate

Go to your endpoint (api.sample.com or sample.com, wherever you want to do SSL pinning) from your browser and click on the green lock icon > certificate.

![Certificate](../images/certificate.png)

Drag and drop certificate icon to your Finder to save it locally. Then drag it to Xcode and include it in your app's bundle.

## 2. Apply certificate pinning to your NSURLSession delegate

In order to use certificate pinning with NSURLSession you will make use of URLSession delegate and its urlSession(_:didReceive:completionHandler:) method.
Inside this method you will just compare the server certificate with the bundled certificate and only if they are equal, you establish the network connection.

Sample delegate class:

```swift
struct Certificate {
    let certificate: SecCertificate
    let data: Data
}

/// Delegate object that is passed to an instance of NSURLSession as its delegate.
class PinningDelegate: NSObject {
    /// Certificate names without the .cer extension
    let certificatesNames: [String]
    let pinnedUrl: URL
    let bundle: Bundle

    private lazy var bundledCertificates: [Certificate] = {
        return certificatesNames.compactMap {
            guard let file = bundle.url(forResource: $0, withExtension: "cer"),
                let data = try? Data(contentsOf: file),
                let cert = SecCertificateCreateWithData(nil, data as CFData) else {
                    return nil
            }
            return Certificate(certificate: cert, data: data)
        }
    }()

    init(certificatesNames: [String], pinnedUrl: URL, bundle: Bundle = .main) {
        self.certificatesNames = certificatesNames
        self.pinnedUrl = pinnedUrl
        self.bundle = bundle
    }

    // Evaluates trust for the specified certificate. Compares it against another certificate.
    private func validate(certificate: Certificate, against certData: Data, using secTrust: SecTrust) -> Bool {
        let certArray = [certificate.certificate] as CFArray
        SecTrustSetAnchorCertificates(secTrust, certArray)
        var result = SecTrustResultType.invalid
        SecTrustEvaluate(secTrust, &result)
        let isValid = (result == .unspecified || result == .proceed)

        //Validate host certificate against pinned certificate.
        return isValid && certData == certificate.data
    }
}

extension PinningDelegate: URLSessionDelegate {
    /// Handles the comparison of local and remote certificate for certificate pinning
    func urlSession(_ session: URLSession, didReceive challenge: URLAuthenticationChallenge,
                   completionHandler: @escaping (URLSession.AuthChallengeDisposition, URLCredential?)
                   -> Void) {
        // Testing the challenge type and host name of a server trust authentication challenge
        let protectionSpace = challenge.protectionSpace
        guard protectionSpace.authenticationMethod ==
            NSURLAuthenticationMethodServerTrust,
            let pinnedHost = pinnedUrl.host,
            protectionSpace.host.contains(pinnedHost) else {
                completionHandler(.performDefaultHandling, nil)
                return
        }

        // Get the server trust instance
        guard let serverTrust = protectionSpace.serverTrust else {
            completionHandler(.cancelAuthenticationChallenge, nil)
            return
        }

        // Get the leaf certificate
        let certificateCount = SecTrustGetCertificateCount(serverTrust)
        guard certificateCount > 0, let certificate = SecTrustGetCertificateAtIndex(serverTrust, 0) else {
            completionHandler(.cancelAuthenticationChallenge, nil)
            return
        }

        // Compare the leaf certificate with local certificates
        let serverCertificate = SecCertificateCopyData(certificate) as Data

        func isValidCertificate(_ certificate: Certificate) -> Bool {
            return validate(certificate: certificate, against: serverCertificate, using: serverTrust)
        }

        if bundledCertificates.contains(where: isValidCertificate) {
            // One of the local certificates matches server certificate
            completionHandler(.useCredential, URLCredential(trust: serverTrust))
        } else {
            // No valid cert available
            completionHandler(.cancelAuthenticationChallenge, nil)
        }
    }
}
```

## 3. Assign a delegate to the instance of NSURLSession

Now when you create NSURLSession you just assign its delegate to make sure the connection is verified.

```swift
            let session = URLSession(
                configuration: URLSessionConfiguration.ephemeral,
                delegate: PinningDelegate(),
                delegateQueue: nil)
```
