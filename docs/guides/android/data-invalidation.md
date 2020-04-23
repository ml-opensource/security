# Data invalidation

![Android](https://img.shields.io/badge/platform-android-success)

## Disable autoBackup for certain SharedPreferences / File locations

Full documentation is here: <https://developer.android.com/guide/topics/data/autobackup#IncludingFiles>

Short version:

`AndroidManifest.xml`:

```xml
<application ...
    android:fullBackupContent="@xml/my_backup_rules">
</application>
```

Add `my_backup_rules.xml` in `res/xml/`. The following sample backs up all shared preferences except device.xml:

```xml
<?xml version="1.0" encoding="utf-8"?>
<full-backup-content>
    <include domain="sharedpref" path="."/>
    <exclude domain="sharedpref" path="device.xml"/>
</full-backup-content>
```

## Good habits

Say you implement a bunch of repositories, which saves user sensitive data. You could add an interface and implement that across those repos:

```kotlin
interface SensitiveRepository {
  fun clear()
}


class PostRepository @Inject constructor() : SensitiveRepository {
  // ...
  
  override fun clear() {
    // ...
  }
}

class UserRepository @Inject constructor() : SensitiveRepository {
  // ...
  
  override fun clear() {
    // ...
  }
}

class LogoutInteractor @Inject constructor(
  private val postRepository: PostRepository,
  private val userRepository: UserRepository,
  // ...
) {
  suspend fun logout() {
    postRepository.clear()
    userRepository.clear()
    // ...
  }
}
```

## Uninstall

When the user uninstalls an app, we can tell Android to prompt whether the user wants to clear user data.  

The documentation isn't clear on exactly what is meant, but a good guess would be SharedPreferences and anything else being backend up by autoBackup

<https://developer.android.com/guide/topics/manifest/application-element.html#fragileuserdata>
