# SGR Vision — Android SDK

On-device, real-time video enhancement for **Media3 / ExoPlayer**, rendered on **OpenGL ES**.
Same enhancement core as the [Web](https://github.com/SGRsoft-Dev) and [iOS](https://github.com/SGRsoft-Dev/vision-ios-sdk) SDKs — shared `profiles.json` spec, 1:1 pipeline parity.

- **Backend:** OpenGL ES (Media3 `Effect` pipeline). CNN pass uses ES 3.1 compute when available, gracefully skipped otherwise.
- **minSdk:** 24 · **compileSdk:** 34 · **JDK:** 17
- **Latest version:** `0.1.0`

## Install (GitHub Packages)

Artifacts are hosted on the GitHub Packages Maven registry for this repo. GitHub Packages **requires authentication even for reads**, so every consumer needs a GitHub token with the `read:packages` scope.

**1. Provide credentials** (do not commit them) — e.g. in `~/.gradle/gradle.properties`:

```properties
gpr.user=YOUR_GITHUB_USERNAME
gpr.key=ghp_xxxxxxxxxxxxxxxxxxxx   # token with read:packages
```

**2. Add the repository** in `settings.gradle.kts`:

```kotlin
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
        maven {
            url = uri("https://maven.pkg.github.com/SGRsoft-Dev/vision-android-sdk")
            credentials {
                username = providers.gradleProperty("gpr.user").orNull
                    ?: System.getenv("GITHUB_ACTOR")
                password = providers.gradleProperty("gpr.key").orNull
                    ?: System.getenv("GITHUB_TOKEN")
            }
        }
    }
}
```

**3. Add the dependency** in your module's `build.gradle.kts`:

```kotlin
implementation("com.sgrsoft.vision:vision:0.1.0")
```

Media3 (`media3-common`, `media3-exoplayer`, `media3-effect`, `media3-ui` `1.4.1`) is exposed as an `api` dependency and pulled in transitively.

## Quick start

```kotlin
import androidx.media3.exoplayer.ExoPlayer
import androidx.media3.ui.PlayerView
import com.sgrsoft.vision.VisionEnhancer
import com.sgrsoft.vision.VisionOptions
import com.sgrsoft.vision.VisionProfile

val player = ExoPlayer.Builder(context).build().apply {
    setMediaItem(MediaItem.fromUri(url)); prepare(); playWhenReady = true
}
val playerView = PlayerView(context).apply { this.player = player }

// Attach the enhancement effect chain to the player.
val enhancer = VisionEnhancer(
    player = player,
    options = VisionOptions(enabled = true, profile = VisionProfile.detail),
    context = context,
    licenseKey = "VSK-XXXXX-XXXXX-XXXXX-XXXXX",   // optional — gated until verified
    onLicense = { state -> /* pending / valid / invalid */ },
)

// Live updates without recreating the effect:
enhancer.update(VisionOptions(enabled = true, profile = VisionProfile.ott))

// On teardown:
enhancer.destroy(); player.release()
```

### Profiles

`default` · `detail` · `ott` · `education` · `cctv` · `anime` · `oled` · `retro` · `off`

### Licensing

When `licenseKey` is set, the pipeline starts in pass-through (original video) and applies the
effect only after the key is verified for your app's `packageName`. Requires the `INTERNET`
permission (already declared by the library).

## Example app

A full Compose demo (source picker, status panel, effect toggles, preset grid, fullscreen,
dark mode) lives in the SDK monorepo under `vision-sdk-android/example`.

---

© SGRsoft. Proprietary.
