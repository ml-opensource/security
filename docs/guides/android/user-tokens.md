# Encrypting user tokens

![Android](https://img.shields.io/badge/platform-android-success)

For encrypting user sensitive data on Android we can use Nodes' [Locksmith](https://github.com/nodes-android/locksmith) library to handle the encryption. It uses Android Keystore system lets you store cryptographic keys in a container to make it more difficult to extract from the device.

As a alternative to Locksmith library we can use Google's [Security](https://developer.android.com/jetpack/androidx/releases/security) library.

## Using Locksmith

Add the dependency for the lib:

```groovy
implementation 'dk.nodes.locksmith:core:1.2.2'
```

Init the library on the `Application` class:

```kotlin
override fun onCreate() {
    super.onCreate()

    // Configure our Locksmith instance
    val locksmithConfiguration = LocksmithConfiguration()
    locksmithConfiguration.keyValidityDuration = 120

    // Start our locksmith instance
    Locksmith.init(this, locksmithConfiguration)
}
```

This is a example on how to store values with Locksmith:

```kotlin
// String Encrypt/Decrypt
val encryptedString = Locksmith.instance.encryptionManager.encryptString("userToken")
val decryptedString = Locksmith.instance.encryptionManager.decryptString("userToken")
```

After that you can store the encrypted value to the `SharedPreferences` as usual.

## Using Security

Security its Google library to encryption. You can read more details about it [here](https://developer.android.com/topic/security/data).

*Note that for using this library the **minSdk** needs to be 23.*

Add the dependency for the lib:

```groovy
implementation 'androidx.security:security-crypto:1.0.0-alpha02'
```

This is a example to create the `EncryptedSharedPreferences`:

```kotlin
val encryptedFileName = "encryptedSharedPref"

val encryptedSharedPref = EncryptedSharedPreferences.create(
    encryptedFileName,
    MasterKeys.getOrCreate(MasterKeys.AES256_GCM_SPEC),
    context,
    EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
    EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
)
```

After creating you can use it the same way as a normal `SharedPreferences`:

```kotlin
encryptedSharedPref.edit().putString("DEMO_KEY", "This is a demo value").apply()
```

To open an already created `EncryptedSharedPreferences` just do the same as to create it:

```kotlin
val encryptedFileName = "encryptedSharedPref"

val encryptedSharedPref = EncryptedSharedPreferences.create(
    encryptedFileName,
    MasterKeys.getOrCreate(MasterKeys.AES256_GCM_SPEC),
    context,
    EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
    EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
)

val result = encryptedSharedPref.getString("DEMO_KEY", "")
// result = "This is a demo value"
```
