# SGR Vision — Android SDK (AAR 배포 저장소)

온디바이스 실시간 영상 화질 개선 SDK의 바이너리(AAR) 배포 저장소입니다.
**Media3(ExoPlayer)** 기반, **OpenGL ES** 렌더링. Web · iOS SDK 와 동일한 화질 개선 코어(`profiles.json` 스펙 공유, 파이프라인 1:1 동일).

이 저장소는 컴파일된 **`aar` 만** 호스팅하는 **raw GitHub Maven 저장소**이며 소스는 포함하지 않습니다. **인증 토큰이 필요 없습니다.**

- 좌표: `com.sgrsoft.vision:vision`
- 최신 버전: **0.1.6**
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
    implementation("com.sgrsoft.vision:vision:0.1.7")
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
        licenseKey = "VSK-XXXXX-XXXXX-XXXXX-XXXXX",   // 선택 — 효과 즉시 적용→2초 후 검증, 실패 시 원본
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

**낙관적 게이팅** — `licenseKey` 를 지정하면 효과가 즉시 적용되고 2초 후 검증되며, 실패 시 효과를 끄고 원본으로 되돌립니다. 검증은 앱의 `packageName` 으로 수행합니다. `INTERNET` 권한이 필요합니다(라이브러리에 이미 선언됨).

`packageName` 은 SDK 가 프레임워크(`ActivityThread.currentPackageName()`)에서 직접 읽으며, 전달한 `context.packageName` 은 폴백일 뿐이라 다른 앱 ID 로 위조할 수 없습니다.

### DRM 콘텐츠

DRM(Widevine 등) 보호 콘텐츠는 secure surface 라 GL 처리가 불가해 화질 개선을 적용할 수 없습니다. 대신 SDK 가 암호화 비디오 트랙(`Format.drmInitData`)을 감지하면 이펙트를 떼어 **검정 대신 원본을 그대로 재생**합니다(`VisionState.drmFallback`). 클리어 소스로 바뀌면 자동 재부착됩니다.

## 예제 앱

소스 선택 · 상태 패널 · 효과 토글 · 프리셋 그리드 · 전체화면 · 다크모드를 포함한 Compose 데모가 SDK 모노레포 `vision-sdk-android/example` 에 있습니다.

---

© SGRsoft. Proprietary.
