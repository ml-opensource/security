# File/App Tampering

![Android](https://img.shields.io/badge/platform-android-success)

## Verify File Integrity

Cryptographic hash functions are widely used to check data for errors. If you know the hash of a downloaded file, you can confirm if your copy is identical.

Android's `MessageDigest` class supports most popular algorithms such as `MD5`, `SHA-1` or `SHA-256`. Though, as of 2020 you should avoid weaker algorithms like `"MD5"` or `"SHA-1"` since those have been exploited and formally deprecated by NIST.

Following code shows how to get a SHA-256 checksum of a File's InputStream:

```kotlin
fun generateChecksum(inputStream: InputStream, algorithm : String = "SHA-256"): String? = try {
    val digest: MessageDigest = MessageDigest.getInstance(algorithm)
    val hash: ByteArray = digest.digest(inputStream.readBytes())
    hash.toHexString()
} catch (exception: NoSuchAlgorithmException) {
    exception.printStackTrace()
    null
}

fun ByteArray.toHexString() : String{
    val hexChars = "0123456789ABCDEF".toCharArray()
    val result = StringBuffer()

    forEach {
        val octet = it.toInt()
        val firstIndex = (octet and 0xF0).ushr(4)
        val secondIndex = octet and 0x0F
        result.append(hexChars[firstIndex])
        result.append(hexChars[secondIndex])
    }

    return result.toString()
}
```

## Verify App Signature

Developers must sign applications with their private key before the app can be installed on user devices. The resulting app signature will be broken if the APK is altered in any way.

Use following class to verify your app signature at runtime:

```kotlin
object AppSignatureValidator {

    enum class Result {
        VALID,
        INVALID,
        UNKNOWN
    }

    // TODO: Set value for expectedSignature
    //  - Run app with AppSignatureValidator.validate()
    //  - Check logs for 'EXPECTED_SIGNATURE' and set value
    //  - Remove line Log.d("EXPECTED_SIGNATURE",...
    private const val expectedSignature = "" // TODO: SET!

    fun validate(context: Context): Result {
        context.getAppSignature()?.string()?.let { currentSignature ->

            Log.d("EXPECTED_SIGNATURE", currentSignature) // TODO: REMOVE!

            return if (currentSignature == expectedSignature) {
                Result.VALID
            } else {
                Result.INVALID
            }
        }
        return Result.UNKNOWN
    }

    private fun Context.getAppSignature(): Signature? = if (Build.VERSION.SDK_INT < 28) {
        packageManager.getPackageInfo(
            packageName,
            PackageManager.GET_SIGNATURES
        ).signatures.firstOrNull()
    } else {
        packageManager.getPackageInfo(
            packageName,
            PackageManager.GET_SIGNING_CERTIFICATES
        ).signingInfo.apkContentsSigners.firstOrNull()
    }

    private fun Signature.string(): String? = try {
        val signatureBytes = toByteArray()
        val digest = MessageDigest.getInstance("SHA")
        val hash = digest.digest(signatureBytes)
        Base64.encodeToString(hash, Base64.NO_WRAP)
    } catch (exception: Exception) {
        null
    }
}
```

You may want to hide your expectedSignature String according to [Hiding Sensitive Configuration Data](./full-code-obfuscation.md).

## Verify App Installer

Each app has access to the identifier of the app that installed it. Therefore, we can verify that our app was installed e.g. via Google Play Store.

```kotlin
enum class Installer(val id: String) {
    GOOGLE_PLAY_STORE(id = "com.android.vending"),
    AMAZON_APP_STORE(id = "com.amazon.venezia")
}

fun Context.verifyInstaller(installer: Installer): Boolean {
    return packageManager.getInstallerPackageName(packageName).startsWith(installer.id)
}
```
