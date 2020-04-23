# Local Cache

![Android](https://img.shields.io/badge/platform-android-success)

## Files

Following code shows how to create and read encrypted files using `androidx.security.crypto.EncryptedFile.Builder`:

```groovy
implementation "androidx.security:security-crypto:1.0.0-alpha02"
```

Find the latest Security library version [here](https://developer.android.com/jetpack/androidx/releases/security).

```kotlin
val masterKeyAlias: String = MasterKeys.getOrCreate(MasterKeys.AES256_GCM_SPEC)

val file = File(context.getFilesDir(), "secret_data")
val encryptedFile: EncryptedFile = EncryptedFile.Builder(
    file,
    context,
    masterKeyAlias,
    EncryptedFile.FileEncryptionScheme.AES256_GCM_HKDF_4KB
).build()

// Write to the encrypted file
val encryptedOutputStream: FileOutputStream = encryptedFile.openFileOutput()

// Read the encrypted file
val encryptedInputStream: FileInputStream = encryptedFile.openFileInput()
```

## SharedPreferences

Following code shows how to encrypt keys and values using `androidx.security.crypto.EncryptedSharedPreferences`:

```groovy
implementation "androidx.security:security-crypto:1.0.0-alpha02"
```

Find the latest Security library version [here](https://developer.android.com/jetpack/androidx/releases/security).

```kotlin
val masterKeyAlias: String = MasterKeys.getOrCreate(MasterKeys.AES256_GCM_SPEC)

val sharedPreferences: SharedPreferences = EncryptedSharedPreferences.create(
    "secret_shared_prefs",
    masterKeyAlias,
    context,
    EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
    EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
)

// Use the shared preferences and editor as you normally would
val editor = sharedPreferences.edit()
```

## Room

Room currently does not support encryption features out of the box but there is [SQLCipher](https://github.com/sqlcipher/android-database-sqlcipher) which is an open source library that provides secure 256-bit AES encryption of SQLite database files.

Following code shows how to to configure Room to use SQLCipher for Android:

```groovy
implementation 'net.zetetic:android-database-sqlcipher:4.3.0'
implementation "androidx.sqlite:sqlite:2.0.1"
```

```kotlin
val passphrase: ByteArray = SQLiteDatabase.getBytes(userEnteredPassphrase)
val factory = SupportFactory(passphrase)
val room: SomeDatabase = Room.databaseBuilder(activity, SomeDatabase::class.java, DB_NAME)
    .openHelperFactory(factory)
    .build()
```

Now, Room will make all of its database requests using SQLCipher for Android instead of the framework copy of SQLCipher - meaning you can use Room as you normally would.
