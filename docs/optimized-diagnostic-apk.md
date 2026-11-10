# Optimized diagnostic APK

Build target: `./gradlew :app:assembleFossDiagnostic`

This FOSS diagnostic variant inherits the release configuration, keeps R8
minification and resource shrinking, and uses Android's automatically generated
debug signing key. No GitHub keystore secrets or manually generated signing
material are required. It is **not** a debuggable process: this preserves
release-like execution behavior. Runtime logs can still be read through logcat,
and the diagnostic variant explicitly enables verbose Koin logging.

The application ID has the `.diagnostic` suffix, allowing side-by-side
installation with the release app. A different fresh CI debug certificate
is normally created on each clean runner, so upgrades may require uninstalling
the previous diagnostic APK first.

The workflow copies the checked-in release baseline/startup profile text to
the diagnostic variant source sets before building. The build lists profile
assets found inside the APK for inspection and uploads the R8 mapping file.

Install the universal APK from GitHub Actions artifacts:

```bash
adb install -r AniHyou-foss-diagnostic-universal.apk
adb logcat -c
adb logcat -v threadtime
```

To reduce noise while retaining error messages from all processes:

```bash
adb logcat -v threadtime '*:V'
```

These logcat commands need a connected device and Android platform tools.
The workflow records full Gradle `--info --stacktrace` output to
`diagnostic-build.log`. It does not automatically collect device-runtime logs.
Android/third-party code that does not emit logs cannot be made to log merely
by changing build settings. Minification may still change stack traces; use
the uploaded R8 `mapping.txt` to retrace those stacks.

The full optimization/debugging tradeoff matters: `isDebuggable=false` retains
release-like runtime behavior but does not allow attaching a normal Java/Kotlin
debugger. Debug signing is independent of `android:debuggable`.
