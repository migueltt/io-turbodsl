```text
 - ..- .-. -... --- -.. ... .-.. - ..- .-. -... --- -.. ... .-..
     ___________       ______
       ___  ___/        __/ /           ________   _____  ___
       __  /__  __ _,___ / /_   ____     __  __ \ / ___/ /  /
    ____  // / / // ___// __ \ / __ \ ____  / / / \ \   /  /
     __  // /_/ // /   / (_/ // (_/ /  __  /_/ /___\ \ /  /___
   _____/ \__,_//_/   /_,___/ \____/ _________//_____//______/
   io.turbodsl
 - ..- .-. -... --- -.. ... .-.. - ..- .-. -... --- -.. ... .-..
```
> A _DSL-engine_ to **turbo-charge** your Kotlin development.

> `io.turbodsl` will not make your application faster, but will make your development easier,
> and most importantly, write asynchronous code in a natural way using DSL expressions.

## Releases
- `io.turbodsl:io-turbodsl-core:1.0.0` : Initial release
- `io.turbodsl:io-turbodsl-core:1.1.0` : Minor bug fixes
- `io.turbodsl:io-turbodsl-core:1.2.0` : Breaking changes
  - Rename `context` to `input`
  - Rename `dispatcher` to `context` to follow Kotlin standards for `CoroutineContext`
  - Retry-mechanisms extracted into `retry {...}` expression - refactored to its own `RetryScope`

## Configuration
```
// build.gradle.kts
repositories {
    maven()
    // Also through github-packages
    maven {
        name = "GitHubPackages"
        url = uri("https://maven.pkg.github.com/migueltt/io-turbodsl")
        credentials {
            username = <github-user>
            password = <github-personal-access-token>
        }
    }
}
dependencies {
    implementation("io.turbodsl:io-turbodsl-core:<version>")
}
```

## Tutorials, code-samples
Check https://github.com/migueltt/io-turbodsl-tutorial
```kotlin
TurboScope.execute {
    job { ... }
    async(
      job1 = asyncJob {...},
      job2 = asyncJob {...},
      // up to 10 jobs
      job10 = asyncJob {...},
    ) { ok, r1, r2, ..., r10 ->
        // process results
    }
    async {
        // Add any number of asynchronous jobs
        asyncJob {...}
        :
    }
}
```
## Concepts & Architecture
Check https://www.turbodsl.io

---

## Fundamentals
- Everything is based on DSL expressions using [Kotlin approach](https://kotlinlang.org/docs/type-safe-builders.html)
- _Scopes_ are created internally to maintain structure and call hierarchy
- Each _scope_ allows additional DSL expressions, making `TurboDSL` usage more idiomatic
- The runtime overhead is minimal, comparing to pure Kotlin coroutines implementation.

## Features
- Each scope has its own `input`. If it is not explicitly specified, it inherits the parent's `input`.
  - This allows to avoid referencing variables across scopes.
  - Provide immutability on how incoming data is processed. This is critical when having asynchronous code.
- Asynchronous execution allows 3 different modes:
    - `AsyncMode.CancelNone`: Continue even if any job fails.
    - `AsyncMode.CancelFirst`: Cancel running jobs as soon as something fails, keeping results from those completed jobs.
    - `AsyncMode.CancelAll`: If one job fails, cancel all other jobs.
- Introduce initial delays, if required.
- Specify timeout (maximum execution time), if required.
- Specify retry mechanisms:
  - Number of retries
  - Delays between retries.
  - Delay factor to increase or decrease delay on subsequent retries
  - Retry strategy:
    - `RetryMode.OnTimeoutOnly`: Retry only when `timeout` limit has been reached.
    - `RetryMode.OnErrorOnly`: Retry only when an unexpected error was thrown.
    - `RetryMode.Always`: Retry always, for any error.
    - `RetryMode.Never`: Never retry.

