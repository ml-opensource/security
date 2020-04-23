# HTTPS

![Android](https://img.shields.io/badge/platform-android-success)

## Disabling clear text traffic / http

AndroidManifest.xml specifies the following `android:usesCleartextTraffic`.
The default value for apps that target API level 27 or lower is "true". Apps that target API level 28 or higher default to "false".

To prevent this on devices below API level 28, add the following to the network config:

```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <domain-config cleartextTrafficPermitted="false">
        <domain includeSubdomains="true">*.vapor.cloud</domain>
    </domain-config>
</network-security-config>
```

## OkHttp

Specific security vs. connectivity decisions are implemented by ConnectionSpec. OkHttp includes four built-in connection specs:

- RESTRICTED_TLS is a secure configuration, intended to meet stricter compliance requirements.
- MODERN_TLS is a secure configuration that connects to modern HTTPS servers.
- COMPATIBLE_TLS is a secure configuration that connects to secure–but not current–HTTPS servers.
- CLEARTEXT is an insecure configuration that is used for <http://> URLs.

 By default, OkHttp will attempt a MODERN_TLS connection. If the security requirements are higher, we can set the connection spec to RESTRICTED_TLS - [TLS 1.2 and TLS 1.3](https://github.com/square/okhttp/blob/master/okhttp/src/main/java/okhttp3/ConnectionSpec.kt#L329)
