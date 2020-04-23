# Public Key Pinning

![Android](https://img.shields.io/badge/platform-android-success)

Here is a an example of the network_security_config.xml file with a pinned certificate on the domain `vapor.cloud` and every subdomain i.e. `project-staging.vapor.cloud`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <domain-config>
        <domain includeSubdomains="true">vapor.cloud</domain>
        <pin-set>
            <pin digest="SHA-256">BhI78NSITmovXeo9CktEHOfLqF+znJ5TmIEyjlemfdw=</pin>
        </pin-set>
    </domain-config>
</network-security-config>
```

The main issue is how you get get the correct certificate. The easiest way to do it is to add a certificate pinner to okhttp client (with a random certificate). When the SSL handshake fails, the exception will contain the various domain certificates.

```kotlin
    @Provides
    @AppScope
    fun provideHttpClient(authInterceptor: AuthInterceptor): OkHttpClient {

        val certPinner = CertificatePinner.Builder()
                .add("project-staging.vapor.cloud",
                        "sha256/Hx0QrYgLW930RhadyERdgbcmth9ONj/Oh02YqZyfCIg=")
                .build()

        val clientBuilder = OkHttpClient.Builder()
                .connectTimeout(45, TimeUnit.SECONDS)
                .readTimeout(60, TimeUnit.SECONDS)
                .writeTimeout(60, TimeUnit.SECONDS)
                .certificatePinner(certPinner)
                .addInterceptor(authInterceptor)
                .addInterceptor(NMetaInterceptor(dk.danskespil.quick.BuildConfig.FLAVOR))
```

This outputs the following which you can copy paste ind and correct:

```text
 <-- HTTP FAILED: javax.net.ssl.SSLPeerUnverifiedException: Certificate pinning failure!
    Peer certificate chain:

    sha256/Hx0QrYgLW930RhAWuRfdtWLv1h9ONj/Oh02YqZyfCIg=: CN=*.vapor.cloud,OU=Domain Control Validated
     sha256/amMeV6gb9QNx0Zf7FtJ19Wa/t2B7KpCF/1n2Js3UuSU=: CN=AlphaSSL CA - SHA256 - G2,O=GlobalSign nv-sa,C=BE
     sha256/K87oWBWM9UZfyddvDfoxL+8lpNyoUB2ptGtn0fv6G2Q=: CN=GlobalSign Root CA,OU=Root CA,O=GlobalSign nv-sa,C=BE

    Pinned certificates for quick-staging.vapor.cloud:     sha256/BhI78NSITmovXeo9CktEHOfLqF+znJ5TmIEyjlemfdw=
```
