# Debugger Check

![Android](https://img.shields.io/badge/platform-android-success)

## Checking the Debuggable Flag

The `android:debuggable` flag determinants whether or not an application can be debugged when running. If the flag is set in release builds, we can assume that our Manifest has been tampered with.

```kotlin
val isDebuggable = applicationInfo.flags and ApplicationInfo.FLAG_DEBUGGABLE != 0
 ```

## Checking if a Debugger is attached

Android's `Debug` class offers a static method to determine whether a debugger is connected.

```kotlin
val isDebuggerAttached = Debug.isDebuggerConnected() || Debug.waitingForDebugger()
```

Following example shows how to guard some sensitive code against debuggers.

```kotlin
fun runSensitiveCalculation() = guardDebugger {
    // No debugger attached
    // TODO: Calculation
}

/**
 * Executes [function] only if no debugger is attached.
 */
fun guardDebugger(function: (() -> Unit)) {
    val isDebuggerAttached = Debug.isDebuggerConnected() || Debug.waitingForDebugger()
    if (!isDebuggerAttached) {
        function.invoke()
    }
}
```
