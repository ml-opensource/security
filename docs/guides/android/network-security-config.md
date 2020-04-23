# Network security config

![Android](https://img.shields.io/badge/platform-android-success)

The Network Security Configuration feature lets apps customize their network security settings in a safe, declarative configuration file without modifying app code. These settings can be configured for specific domains and for a specific app.

The official docs can be read here: <https://developer.android.com/training/articles/security-config>

## Disable clear text traffic / HTTPS only

*NOTE*: AndroidManifest.xml specifies the following `android:usesCleartextTraffic`. The default value for apps that target API level 27 or lower is "true". Apps that target API level 28 or higher default to "false".

**Which means that the following isn't necessary in apps targeting API level 28 or higher.**

```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <domain-config cleartextTrafficPermitted="false">
        <domain includeSubdomains="true">*.vapor.cloud</domain>
    </domain-config>
</network-security-config>
```
