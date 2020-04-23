# Source code obfuscation

![Android](https://img.shields.io/badge/platform-android-success)

Obfuscation is a technic that allows to make the source or machine code difficult for humans to understand. Due to the fact that Android source code can be easily extracted from the APK file, all APKs that are publicly accessible must be obfuscated. Source code is intellectual property of the company and it becomes important to protect it to avoid branding issues and source code modification and following distribution of malicious APK through non-standard app stores.

## R8 / ProGuard

Since Android Gradle plugin 3.4.0, the plugin no longer uses `ProGuard` to perform compile-time code optimization. Instead, the plugin works with the `R8` compiler to handle the following compile-time tasks

- Code shrinking -  detects and safely removes unused classes, fields and methods
- Resource shrinking -  removes unused resources
- **Obfuscation**
- Optimization - inspects and rewrites your code to further reduce the size of your app’s DEX files

### Enable obfuscation

To enable shrinking, obfuscation, and optimization, include the following in your project-level build.gradle file.

```groovy
android {
    buildTypes {
        release {
            // Enables code shrinking, obfuscation, and optimization for only
            // your project's release build type.
            minifyEnabled true

            // Enables resource shrinking, which is performed by the
            // Android Gradle plugin.
            shrinkResources true

            // Includes the default ProGuard rules files that are packaged with
            // the Android Gradle plugin. To learn more, go to the section about
            // R8 configuration files.
            proguardFiles getDefaultProguardFile(
                    'proguard-android-optimize.txt'),
                    'proguard-rules.pro'
        }
    }
    ...
}
```

> Note thats because these compile-time optimizations increase the build time of your project it’s best to enable these compile-time tasks when building the release of your app that you test prior to publishing.

#### Hiding Sensitive Configuration Data

Obfuscation merely changes variables names, not values, so any API-tokens stored as part of the configuration, in build.gradle for example, will be easily accessible. While you can't hide it completely, you can make it a bit harder for a 3rd part to obtain.

##### Split & Encode

This solution revolves around you encoding the value you want to hide using `Base64` and then splitting the resulted value into three or more parts. Then the parts can be combined together and decoded during the runtime. Example of hiding key `SECRET` with value `euQiQ8uOH6vlLsDmTE3RGDiO35ScGfivnkmGGwKa`

```text
// Encode Using Base64
euQiQ8uOH6vlLsDmTE3RGDiO35ScGfivnkmGGwKa --> Base64 --> ZXVRaVE4dU9INnZsTHNEbVRFM1JHRGlPMzVTY0dmaXZua21HR3dLYQ==

// Split Result into several parts
SECRET_1 = ZXVRaVE4dU9INn
SECRET_2 = ZsTHNEbVRFM1JHR
SECRET_3 = GlPMzVTY0dmaX
SECRET_4 = Zua21HR3dLYQ==
```

The `build.gradle` will then be following:

```groovy
defaultConfig {

  ...

  buildConfigField "String", "SECRET_1", "\"ZXVRaVE4dU9INn\""
  buildConfigField "String", "SECRET_2","\"ZsTHNEbVRFM1JHR\""
  buildConfigField "String", "SECRET_3", "\"GlPMzVTY0dmaX\""
  buildConfigField "String", "SECRET_4", "\"Zua21HR3dLYQ==\""

}
```

Then we can retrieve original value during the runtime:

```kotlin
val encodedSecret =
  BuildConfig.AWS_KEY1 +
  BuildConfig.SECRET_1 +
  BuildConfig.SECRET_2 +
  BuildConfig.SECRET_3 +
  BuildConfig.SECRET_4

val secret = String(Base64.decode(encodedSecret))

```

##### Hidden in Native

Native code is far more harder to decompile and make sense of. You can take advantage of that by storing keys in the native module using `NDK`. Refer to [this article](https://medium.com/programming-lite/securing-api-keys-in-android-app-using-ndk-native-development-kit-7aaa6c0176be) for more information

### Rules modification

R8 automatically performs the compile-time optimization tasks, but you can disable certain tasks or customize R8’s behavior through ProGuard rules files.

#### Data Classes

To fix errors and force R8 to keep certain code, add a `-keep` line in the ProGuard rules file. This is most common for data classes as they must be deserialized using reflection.  For example:

```groovy
  // Keep one data class
  -keep public class dk.nodes.example.domain.MyDataClass

  // Keep all data classes in certain package
  -keep public class dk.nodes.example.data.model.**{*;}
  ```

> You can also you `@Keep` annotation to achieve similar result.

#### Dealing with with 3rd party libraries

When using 3rd party libraries/dependencies, ProGuard rules could be a part of the artifact, published by the developer as a recommendation.

When there is no ProGuard rules available you can try running the build and looking into build output, from this you will be able to see if you need to add `-dontwarn` rule to ignore the warnings. Or when running the application you might receive an `NotFoundException` which will tell you which `-keep` rule to add. Example:

```groovy
// Ignore warnings from these namespaces
-dontwarn org.joda.time.**
-dontwarn dk.nodes.locksmith.**

// To avoid ClassNotFoundException and similar
-keep class androidx.appcompat.widget.** { *; }
```

### Debugging

Obfuscation makes debugging and and crashes investigation a bit tricky. For ones `R8/ProGuard` will remove some metadata that is not important for app execution but might be useful for developers, source file names and line numbers for example. You can tell `R8/ProGuard` to keep using `-keepattributes` rule.

```groovy
-keepattributes SourceFile, LineNumberTable

```

Another thing is to make sense about obfuscated stacktrace of the crash that happened in production. It is acieveable by using `mapping.txt` file. Every time you make a release build, `R8/ProGuard` will a generate a **new** `mapping.txt` file that can be used to de-obfuscate the code. R8 saves the file in the `<module- name>/build/outputs/mapping/<build-type>/` directory.

### More Info

- [Android Obfuscation Docs](https://developer.android.com/studio/build/shrink-code.html#decode-stack-trace)
