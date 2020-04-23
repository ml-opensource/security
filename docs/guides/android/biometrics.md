# AndroidX biometric authentication

![Android](https://img.shields.io/badge/platform-android-success)

Please go read the [official documentation](https://developer.android.com/training/sign-in/biometric-auth) for a deeper read through, below is the shortened version.

## Setup

```groovy
implementation 'androidx.biometric:biometric:1.0.1'
```

## Example implementation

**Authenticator.kt** helper class for suspending authentication

```kotlin
class Authenticator @Inject constructor(

){

    private fun createPromptInfo(): BiometricPrompt.PromptInfo {
        return BiometricPrompt.PromptInfo.Builder()
                .setTitle(Translation.fingerprint.settingsHeader)
                .setSubtitle(Translation.fingerprint.settingsContent)
                .setNegativeButtonText(Translation.paymentCards.createPasswordPlaceholder)
                .setConfirmationRequired(true)
                .build()
    }

    inner class ContinuationAuthenticationCallback(private val continuation: Continuation<AuthenticatorResult>): BiometricPrompt.AuthenticationCallback() {
        override fun onAuthenticationError(errorCode: Int, errString: CharSequence) {
            continuation.resume(AuthenticatorResult.Error(errorCode, errString))
        }

        override fun onAuthenticationSucceeded(result: BiometricPrompt.AuthenticationResult) {
            continuation.resume(AuthenticatorResult.Success)
        }

        override fun onAuthenticationFailed() {
            continuation.resume(AuthenticatorResult.Failure)
        }
    }

    suspend fun authenticate(fragment: Fragment, executor: Executor): AuthenticatorResult {
        return suspendCoroutine { continuation ->
            BiometricPrompt(fragment, executor, ContinuationAuthenticationCallback(continuation)).authenticate(createPromptInfo())
        }
    }
}

sealed class AuthenticatorResult {
    object Success: AuthenticatorResult()
    data class Error(val errorCode: Int, val errString: CharSequence): AuthenticatorResult()
    object Failure: AuthenticatorResult()
}
```

In our Fragment:

```kotlin
fun promptAuthentication() = scope.launch {
  val result = scope.withContext(Dispatchers.Main) { Authenticator().authenticate(this, Executors.newSingleThreadExecutor()) }
  val refreshResult = if(result is AuthenticatorResult.Success) {
    viewModel.login()
  }
}
```
