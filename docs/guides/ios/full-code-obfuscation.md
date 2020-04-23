# Full code obfuscation on iOS

![iOS](https://img.shields.io/badge/platform-iOS-blue)

Full code obfuscation is the technique through which the names of your classes, variables, struct, objects, etc are renamed to something that people can't understand. So after obfuscation, the source code will basically look like this:

```swift
class fjiovh4894bvic: XbuinvcxoDHFh3fjid {
  func cxncjnx8fh83FDJSDd() {
    return vPAOSNdcbif372hFKF()
  }
}
```

We use [SwiftShield](https://github.com/rockbruno/swiftshield) for full code obfuscation.

Our CI setup is able to obfuscate the source code. To enable full source code obfuscation on the CI, all you need to do is add in your `project.yml` the following flag: `obfuscate: true`.
