# App Transport Security

![iOS](https://img.shields.io/badge/platform-iOS-blue)

**[App Transport Security](https://developer.apple.com/documentation/bundleresources/information_property_list/nsapptransportsecurity)** is enabled by default. This means that iOS won't allow loading any resource over an unsecured HTTP connection, but only on HTTPS.

Bypassing ATS and enabling HTTP loads is highly unadvisable and you will have to justify it when submitting the app. This should be the last resort, do everything you can to load the resources over HTTPS instead of HTTP. If we control the backend, talk to the backend and server guys and tell them to serve the content over HTTPS instead. If we don't control the server, try finding the same resource(s) at a different URL, served over HTTPS. If that's not possible, argue with the client and inform them about the security risks disabling ATS involve.

<details>
<summary>Here's how to disable ATS:</summary>

> This is not recommended and should only be used during development.

If you need to bypass ATS, you can configure the `Info.plist` file (right click on the file and choose Open as > Source Code).

**Example for completely disabling secure transfer security:**

```xml
<key>NSAppTransportSecurity</key>
<dict>
  <!--Include to allow all connections (DANGER)-->
  <key>NSAllowsArbitraryLoads</key>
      <true/>
</dict>
```

**Example for Per-Domain Exceptions:**

```xml
<key>NSAppTransportSecurity</key>
<dict>
  <key>NSExceptionDomains</key>
  <dict>
    <key>domaintobypass.com</key>
    <dict>
      <!--Include to allow subdomains-->
      <key>NSIncludesSubdomains</key>
      <true/>
      <!--Include to allow HTTP requests-->
      <key>NSExceptionAllowsInsecureHTTPLoads</key>
      <true/>
      <!--Include to specify minimum TLS version-->
      <key>NSExceptionMinimumTLSVersion</key>
      <string>TLSv1.1</string>
    </dict>
  </dict>
</dict>
```

**Property List Keys**

* `NSAllowsArbitraryLoadsInWebContent`- A Boolean value indicating whether all App Transport Security restrictions are disabled for requests made from web views (default: NO)
* `NSAllowsArbitraryLoadsForMedia`- A Boolean value indicating whether all App Transport Security restrictions are disabled for requests made using the AV Foundation framework (default: NO)
* `NSRequiresCertificateTransparency`- Certificate Transparency (CT) is a protocol that ATS can use to identify mistakenly or maliciously issued X.509 certificates. Set the value for the NSRequiresCertificateTransparency key to YES to require that for a given domain, server certificates are supported by valid, signed CT timestamps from at least two CT logs trusted by Apple.
* `NSAllowsLocalNetworking`- A Boolean value indicating whether to allow loading of local resources (default: NO)
* `NSExceptionRequiresForwardSecrecy` - Set the value for this key to NO to override the requirement that a server support perfect forward secrecy (PFS) for the given domain (default: YES)

</details>
