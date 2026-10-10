# DESIGN.md — Especificación Técnica & Context Prompt Optimizado para Gemini

> **Target LLM Context:** Google Gemini 1.5 Pro / Flash & Multimodal Code Generation  
> **Arquitectura:** Android Nativo (Kotlin 2.0+, Jetpack Compose, Camera2 NDK/Vulkan/OpenGL ES 3.2)  
> **Dominio:** Fotografía Computacional Pro, Light Painting en Vivo, Audio Hotword Hands-Free

---

## 1. System Prompt & Context Framing para Gemini

```markdown
Eres un Arquitecto de Software Android Senior y Diseñador de Sistemas UI/UX de alto rendimiento.
Especialidades requeridas:
1. Jetpack Compose con renderizado custom (Canvas, OpenGL ES 3.2 SurfaceView/TextureView, Vulkan).
2. Android Camera2 API con capacidades manuales (SENSOR_EXPOSURE_TIME, SENSOR_SENSITIVITY, LENS_FOCUS_DISTANCE).
3. Procesamiento de fotogramas en GPU a 60fps con shaders de acumulación de brillo (Lighten, Darken, Decay).
4. Reconocimiento de voz local offline (SpeechRecognizer / On-Device Hotword Detector) para disparador manos libres.
5. Directrices visuales estrictas: AMOLED Pitch Black (#000000), Neón Luminous Teal (#00FFCC) y Glassmorphism oscuro.
```

---

## 2. Permisos del Sistema y Manifiesto Android

```xml
<?xml version="1.0" encoding="utf-8"?>
<!-- AndroidManifest.xml optimizado para hardware de captura manual y audio local -->
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.lightpainting.camera">

    <!-- Acceso a cámara de bajo nivel -->
    <uses-permission android:name="android.permission.CAMERA" />
    <uses-feature android:name="android.hardware.camera" android:required="true" />
    <uses-feature android:name="android.hardware.camera.autofocus" android:required="false" />
    <uses-feature android:name="android.hardware.camera.manual_sensor" android:required="true" />
    <uses-feature android:name="android.hardware.camera.manual_post_processing" android:required="true" />

    <!-- Reconocimiento de voz en dispositivo (Zero-Latency Hands-Free) -->
    <uses-permission android:name="android.permission.RECORD_AUDIO" />

    <!-- Almacenamiento Scoped Storage (DNG RAW y PNG sin compresión) -->
    <uses-permission android:name="android.permission.READ_MEDIA_IMAGES" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"
        android:maxSdkVersion="28" />

    <!-- Aceleración gráfica por hardware para buffers OpenGL/Vulkan -->
    <uses-feature android:glEsVersion="0x00030002" android:required="true" />

    <application
        android:name=".LightPaintingApp"
        android:allowBackup="false"
        android:hardwareAccelerated="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.LightPaintingCamera.Fullscreen">

        <activity
            android:name=".presentation.MainActivity"
            android:exported="true"
            android:screenOrientation="portrait"
            android:configChanges="orientation|screenSize|screenLayout|keyboardHidden">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>
</manifest>
```

---

## 3. Design Tokens (JSON Schema para Gemini Theme Generation)

```json
{
  "theme": "AMOLED_Night_Pro",
  "palette": {
    "background": {
      "pitch_black": "#000000",
      "surface_container_dark": "rgba(10, 13, 18, 0.85)",
      "surface_container_high": "rgba(18, 22, 30, 0.90)",
      "surface_border": "rgba(255, 255, 255, 0.12)"
    },
    "accents": {
      "teal_neon_primary": "#00FFCC",
      "teal_neon_glow": "rgba(0, 255, 204, 0.35)",
      "magenta_neon_secondary": "#FF007F",
      "amber_pause_warning": "#FFB800",
      "red_recording": "#FF3B30"
    },
    "content": {
      "on_surface_primary": "#FFFFFF",
      "on_surface_secondary": "#A0A8B5",
      "on_surface_disabled": "rgba(255, 255, 255, 0.38)"
    }
  },
  "typography": {
    "font_ui": "system-ui, -apple-system, Roboto, sans-serif",
    "font_data_metrics": "ui-monospace, JetBrains Mono, monospace",
    "scales": {
      "display_metrics": "28px / 700 / tabular-nums",
      "hud_label": "11px / 600 / uppercase / letter-spacing: 0.08em",
      "body_regular": "14px / 400"
    }
  },
  "effects": {
    "hud_blur": "backdrop-filter: blur(20px)",
    "neon_elevation": "box-shadow: 0 0 16px rgba(0, 255, 204, 0.45)"
  }
}
```

---

## 4. Arquitectura de Estados y Lógica de Fusión (GPU Shaders)

### 4.1 Máquina de Estados de Captura (`CaptureEngineState`)

```
             ┌───────────────┐
             │   PREPARADO   │ <-----------------┐
             └───────┬───────┘                   │
                     │  [Comando "Play" / Tap]   │
                     ▼                           │
             ┌───────────────┐                   │ [Guardar / Reset]
  ┌────────> │  GRABANDO     │                   │
  │          │ (Acumulación) │                   │
  │          └───────┬───────┘                   │
  │                  │                           │
  │ [Play / Voz]     │ [Comando "Pausa" / Tap]   │
  │                  ▼                           │
  │          ┌───────────────┐                   │
  └───────── │    EN PAUSA   │ ──────────────────┘
             │(Buffer Freeze)│
             └───────────────┘
```

* **Estado PREPARADO**: Sensor Camera2 en vivo, FPS dinámico, Focus Peaking opcional activado.
* **Estado GRABANDO**: Render target en FBO secundario acumulando píxeles según shader activo.
* **Estado EN PAUSA**: 
  - La cámara sigue enviando fotogramas en vivo a la pantalla.
  - El buffer acumulado se congela en memoria GPU.
  - Se activa el **High-Contrast Canvas Overlay**: visualización del lienzo acumulado con contraste gamma reforzado ($1.25\times$) sobre el visor en vivo.
  - El usuario puede modificar el modo de fusión en caliente sin perder la estela ya pintada.

### 4.2 Fragment Shader: Reglas de Modos de Fusión (GLSL)

```glsl
#version 300 es
precision highp float;

in vec2 vTexCoord;
out vec4 fragColor;

uniform sampler2D uLiveSensorTexture;    // Stream en vivo de Camera2
uniform sampler2D uAccumulatedTexture;   // Lienzo previo acumulado en GPU FBO
uniform int uBlendMode;                  // 0: Light, 1: Shadow, 2: Fade/Trail
uniform float uDecayFactor;              // e.g. 0.965 para Modo Estela
uniform float uMinThreshold;             // Umbral mínimo de luminancia [0.0, 1.0]

float getLuminance(vec3 color) {
    return dot(color, vec3(0.2126, 0.7152, 0.0722));
}

void main() {
    vec4 liveColor = texture(uLiveSensorTexture, vTexCoord);
    vec4 prevColor = texture(uAccumulatedTexture, vTexCoord);
    float liveLum = getLuminance(liveColor.rgb);

    if (uBlendMode == 0) { // Pintar con Luz (Light Blend Mode)
        if (liveLum >= uMinThreshold) {
            fragColor = max(prevColor, liveColor);
        } else {
            fragColor = prevColor;
        }
    } 
    else if (uBlendMode == 1) { // Pintar con Sombra (Shadow / Inverted Mode)
        fragColor = min(prevColor, liveColor);
    } 
    else if (uBlendMode == 2) { // Modo Estela Dinámica (Decay Trails)
        vec4 decayed = prevColor * uDecayFactor;
        fragColor = max(decayed, liveColor);
    }
}
```

---

## 5. Módulo de Reconocimiento y Control por Voz (Hands-Free)

### Configuración del Motor en Dispositivo (Zero Latency)
* **Objetivo:** Evitar trepidación al tocar el disparador en el trípode.
* **Servicio:** `OnDeviceSpeechRecognizer` o motor de gramática fonética offline (`Vosk` / `Picovoice Porcupine`).
* **Diccionario de Comandos en Español:**
  * `"Iniciar"` / `"Play"` $\rightarrow$ `CaptureIntent.START_OR_RESUME`
  * `"Pausa"` $\rightarrow$ `CaptureIntent.PAUSE_ACCUMULATION`
  * `"Guardar"` $\rightarrow$ `CaptureIntent.SAVE_FRAME_TO_DISK`
* **Feedback Visual:** Componente HUD `VoiceWaveWidget` con onda de audio animada en Neón Teal `#00FFCC` que pulsa al detectar habla.

---

## 6. Panel de Intervalómetro y Rutinas Automatizadas

### Modelo de Datos Kotlin para Gemini (`IntervalometerConfig.kt`)

```kotlin
data class IntervalometerRoutine(
    val initialDelaySeconds: Int = 3,       // Temporizador de cuenta regresiva previa
    val exposureTimeSeconds: Int = 15,      // Tiempo de acumulación activa continua
    val autoPauseSeconds: Int = 5,          // Pausa entre tomas para cambio de foco de luz
    val cycleCount: Int = 1,                // Número de ejecuciones (1 a Infinito)
    val playAudioBeepOnCycle: Boolean = true // Aviso auditivo al inicio/fin de cada ciclo
)
```

---

## 7. Estructura de Directorios para Código Android Jetpack Compose

```
app/
 ├── src/main/
 │    ├── java/com/lightpainting/camera/
 │    │    ├── core/
 │    │    │    ├── camera/
 │    │    │    │    ├── Camera2Manager.kt
 │    │    │    │    └── ManualControlsHandler.kt
 │    │    │    ├── opengl/
 │    │    │    │    ├── FrameAccumulatorFBO.kt
 │    │    │    │    ├── BlendShaderRenderer.kt
 │    │    │    │    └── FocusPeakingShader.kt
 │    │    │    └── voice/
 │    │    │         ├── VoiceCommandService.kt
 │    │    │         └── HotwordDetector.kt
 │    │    ├── domain/
 │    │    │    ├── model/
 │    │    │    │    ├── CaptureState.kt
 │    │    │    │    ├── BlendMode.kt
 │    │    │    │    └── ManualExposureSettings.kt
 │    │    │    └── repository/
 │    │    │         └── ImageCaptureRepository.kt
 │    │    ├── presentation/
 │    │    │    ├── viewfinder/
 │    │    │    │    ├── ViewfinderScreen.kt
 │    │    │    │    ├── components/
 │    │    │    │    │    ├── CameraHUDOverlay.kt
 │    │    │    │    │    ├── ProSlidersBar.kt
 │    │    │    │    │    ├── ModeSelectorSlider.kt
 │    │    │    │    │    └── VoiceWaveWidget.kt
 │    │    │    ├── intervalometer/
 │    │    │    │    ├── IntervalometerDialog.kt
 │    │    │    │    └── IntervalometerViewModel.kt
 │    │    │    └── theme/
 │    │    │         ├── Color.kt
 │    │    │         ├── Theme.kt
 │    │    │         └── Type.kt
 │    │    └── MainActivity.kt
 │    └── res/
 │         ├── raw/
 │         │    ├── accumulate_frag.glsl
 │         │    └── accumulate_vert.glsl
 │         └── values/
 └── build.gradle.kts
```

---

## 8. Guía de Instalación y Prompt de Ejecución para Gemini

Para pedirle a Gemini generar cualquier módulo específico de este sistema, usar la siguiente plantilla de prompt:

```text
Usa las especificaciones técnicas de DESIGN.md (App Android Cámara & Light Painting):
- Arquitectura: Jetpack Compose + Camera2 + OpenGL ES 3.2
- Tema: AMOLED Pitch Black (#000000) con acento Neón Teal (#00FFCC)
- Genera el código completo de [Insertar Módulo: ej. ViewfinderScreen.kt / BlendShaderRenderer.kt]
- Cumple con los estados de Captura (Preparado, Grabando, En Pausa con High-Contrast Overlay)
- Código limpio en Kotlin 2.0 idiomático con StateFlow y Corrutinas.
```
