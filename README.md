# SGR Vision — Android SDK (AAR 배포 저장소)

온디바이스 실시간 영상 화질 개선 SDK의 바이너리(AAR) 배포 저장소입니다.
**Media3(ExoPlayer)** 기반, **OpenGL ES** 렌더링. Web · iOS SDK 와 동일한 화질 개선 코어(`profiles.json` 스펙 공유, 파이프라인 1:1 동일).

이 저장소는 컴파일된 **`aar` 만** 호스팅하는 **raw GitHub Maven 저장소**이며 소스는 포함하지 않습니다. **인증 토큰이 필요 없습니다.**

- 좌표: `com.sgrsoft.vision:vision`
- 최신 버전: **0.1.2**
- 백엔드: OpenGL ES (Media3 `Effect` 파이프라인). CNN 패스는 ES 3.1 compute 지원 시 사용, 미지원 시 자동 생략.

## 요구 사항

- **minSdk 24** · compileSdk 34 · **JDK 17** · Kotlin

## 설치

```kotlin
// settings.gradle.kts
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
        maven { url = uri("https://raw.githubusercontent.com/SGRsoft-Dev/vision-android-sdk/main") }
    }
}
```
```kotlin
// app/build.gradle.kts
dependencies {
    implementation("com.sgrsoft.vision:vision:0.1.2")
}
```

> 인증 토큰 불필요. 공개 API 가 Media3 `@UnstableApi` 표면을 노출하므로 호출부에 `@OptIn(UnstableApi::class)` 가 필요합니다.
> Media3(`media3-common`/`exoplayer`/`effect`/`ui` `1.4.1`)는 `api` 의존성으로 전이 포함됩니다.

## 빠른 시작

```kotlin
import androidx.media3.common.MediaItem
import androidx.media3.common.util.UnstableApi
import androidx.media3.exoplayer.ExoPlayer
import androidx.media3.ui.PlayerView
import com.sgrsoft.vision.VisionEnhancer
import com.sgrsoft.vision.VisionOptions
import com.sgrsoft.vision.VisionProfile

@OptIn(UnstableApi::class)
fun setup(context: android.content.Context, url: String) {
    val player = ExoPlayer.Builder(context).build().apply {
        setMediaItem(MediaItem.fromUri(url)); prepare(); playWhenReady = true
    }
    val playerView = PlayerView(context).apply { this.player = player }

    // 플레이어에 화질 개선 이펙트 체인 부착
    val enhancer = VisionEnhancer(
        player = player,
        options = VisionOptions(enabled = true, profile = VisionProfile.detail),
        context = context,
        licenseKey = "VSK-XXXXX-XXXXX-XXXXX-XXXXX",   // 선택 — 검증 전까지 원본 패스스루
        onLicense = { state -> /* pending / valid / invalid */ },
    )

    // 이펙트 재생성 없이 라이브 갱신:
    enhancer.update(VisionOptions(enabled = true, profile = VisionProfile.ott))

    // 해제:
    // enhancer.destroy(); player.release()
}
```

### 프로필

`default` · `detail` · `ott` · `education` · `cctv` · `anime` · `oled` · `retro` · `off`

### 라이선스

`licenseKey` 를 지정하면 파이프라인이 원본(패스스루)으로 시작하고, 앱의 `packageName` 으로 키가 검증된 후에만 효과를 적용합니다. `INTERNET` 권한이 필요합니다(라이브러리에 이미 선언됨).

## 예제 앱

소스 선택 · 상태 패널 · 효과 토글 · 프리셋 그리드 · 전체화면 · 다크모드를 포함한 Compose 데모가 SDK 모노레포 `vision-sdk-android/example` 에 있습니다.

---

© SGRsoft. Proprietary.
