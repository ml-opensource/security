# Jailbreak Detection

![iOS](https://img.shields.io/badge/platform-iOS-blue)

Jailbreaking on iOS means that the OS installation was tampered with to allow arbitrary code to run as well as disable some essential security features that the OS provides (codesigning, kernel protection, sandboxing etc). This makes it easier to modify the application's data or even the binary itself.

Here's a snippet of code that can be used to detect if your app is running on compromised devices:

```swift

import SystemConfiguration
import MachO
import Foundation

// MARK: - RootedRepository -

public protocol RootedRepository {
    func sandboxIntegrityCompromised() -> Bool
    func jailbreakFileCheck() -> Bool
    func symbolicLinkingCheck() -> Bool
    func dyldCheck() -> Bool
}

// MARK: - IsSafeFromRooted -

public protocol DeviceCompromised {
    var isCompromised: Bool { get }
}

// MARK: - RootedManager -

public struct RootedManager: DeviceCompromised {

    public var isCompromised: Bool {
        return sandboxIntegrityCompromised() || jailbreakFileCheck() || symbolicLinkingCheck() || dyldCheck()
    }

    public init() {
        #if DEBUG
        NSLog("********************************* Start Rooted check *********************************")
        NSLog("sandboxIntegrityCompromised \(sandboxIntegrityCompromised())")
        NSLog("jailbreakFileCheck \(jailbreakFileCheck())")
        NSLog("symbolicLinkingCheck \(symbolicLinkingCheck())")
        NSLog("dyldCheck \(dyldCheck())")
        NSLog("********************************* End Rooted check ***********************************")
        #endif
    }

}

// MARK: - ext RootedRepository

extension RootedManager: RootedRepository {

    public func sandboxIntegrityCompromised() -> Bool {
        var pid: pid_t = -1
        var status: Int32 = 0

        posix_spawn(&pid, "", nil, nil, [], nil);
        waitpid(pid, &status, WEXITED);

        if pid >= 0 {
            return true
        } else {
            return false
        }
    }

    public func jailbreakFileCheck() -> Bool {
        var s = stat()

        if stat("/Applications/Cydia.app", &s) >= 0 {
            return true
        } else if stat("/Library/MobileSubstrate/MobileSubstrate.dylib", &s) >= 0 {
            return true
        } else if stat("/var/cache/apt", &s) >= 0 {
            return true
        } else if stat("/var/lib/cydia", &s) >= 0 {
            return true
        } else if stat("/var/log/syslog", &s) >= 0 {
            return true
        } else if stat("/var/tmp/cydia.log", &s) >= 0 {
            return true
        } else if stat("/bin/bash", &s) >= 0 {
            return true
        } else if stat("/bin/sh", &s) >= 0 {
            return true
        } else if stat("/usr/sbin/sshd", &s) >= 0 {
            return true
        } else if stat("/usr/libexec/ssh-keysign", &s) >= 0 {
            return true
        } else if stat("/etc/ssh/sshd_config", &s) >= 0 {
            return true
        } else if stat("/etc/apt", &s) >= 0 {
            return true
        }

        return false
    }

    public func symbolicLinkingCheck() -> Bool {
        var s = stat()
        if lstat("/Applications", &s) != 1 {
            if (s.st_mode == 0 && S_IFLNK == 0) {
                return true
            }
        } else if lstat("/Library/Ringtones", &s) != 1 {
            if (s.st_mode == 0 && S_IFLNK == 0) {
                return true
            }
        } else if lstat("/Library/Wallpaper", &s) != 1 {
            if (s.st_mode == 0 && S_IFLNK == 0) {
                return true
            }
        } else if lstat("/usr/arm-apple-darwin9", &s) != 1 {
            if (s.st_mode == 0 && S_IFLNK == 0) {
                return true
            }
        } else if lstat("/usr/include", &s) != 1 {
            if (s.st_mode == 0 && S_IFLNK == 0) {
                return true
            }
        } else if lstat("/usr/libexec", &s) != 1 {
            if (s.st_mode == 0 && S_IFLNK == 0) {
                return true
            }
        } else if lstat("/usr/share", &s) != 1 {
            if (s.st_mode == 0 && S_IFLNK == 0) {
                return true
            }
        }

        return false
    }

    public func dyldCheck() -> Bool {
        //Get count of all currently loaded DYLD
        let count = _dyld_image_count()

        for  i in 0..<count {
            //Name of image (includes full path)
            let dyld = _dyld_get_image_name(i)
            if strstr(dyld, "MobileSubstrate") == nil {
                continue
            } else {
                return true
            }
        }

        return false
    }

}

```

To be used like this:

```swift
let rootedManager: DeviceCompromised = RootedManager()

#if TARGET_OS_IPHONE
if rootedManager.isCompromised {
    #warning ("handle this in a nicer way")
 fatalError("device compromised")
}
#endif
```
