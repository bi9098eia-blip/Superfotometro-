# DESIGN.md — Especificaciones Técnicas y Arquitectura: App Android de Cámara & Light Painting

## 1. Visión General del Producto y Contexto
Aplicación nativa para Android (Kotlin / Jetpack Compose) de alto rendimiento orientada a la fotografía profesional de **Light Painting** (pintura con luz), estelas y captura de larga exposición interactiva en tiempo real. 

La app permite previsualizar la acumulación de fotogramas mediante procesamiento computacional por GPU, manipular parámetros manuales de cámara estilo DSLR/Pro, alternar modos de fusión en caliente (incluso en pausa) y operar el disparador con las manos libres mediante reconocimiento de voz en el dispositivo.

---

## 2. Permisos Nativos de Android y Manifiesto
La aplicación requiere permisos para el hardware de captura de imagen y el procesamiento de audio para comandos de voz:

```xml
<!-- AndroidManifest.xml -->
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.lightpainting.camera">

    <!-- Acceso a sensores de cámara avanzados (Camera2 / CameraX) -->
    <uses-permission android:name="android.permission.CAMERA" />
    <uses-feature android:name="android.hardware.camera" android:required="true" />
    <uses-feature android:name="android.hardware.camera.autofocus" android:required="false" />
    <uses-feature android:name="android.hardware.camera.manual_sensor" android:required="true" />

    <!-- Reconocimiento de comandos por voz offline / manos libres -->
    <uses-permission android:name="android.permission.RECORD_AUDIO" />

    <!-- Gestión de almacenamiento de capturas fotográficas en alta resolución -->
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"
        android:maxSdkVersion="28" />
    <uses-permission android:name="android.permission.READ_MEDIA_IMAGES" />

    <application
        android:allowBackup="false"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.LightPaintingCamera.Fullscreen">
        ...
    </application>
</manifest>
```

---

## 3. Guía Estética y Tokens de Diseño (UI/UX)
Diseñada bajo directrices de **Modo Oscuro AMOLED estricto (#000000)** para maximizar la batería de la pantalla OLED y evitar contaminación lumínica en entornos oscuros o sesiones nocturnas.

### Paleta de Colores
* **Background AMOLED**: `#000000` (100% negro puro para píxeles apagados)
* **Surface Container / Glass HUD**: `#0A0D12` / `rgba(10, 13, 18, 0.75)` con desenfoque de fondo (`backdrop-filter: blur(16px)`)
* **Color Accent Primario (Neón Luminous Teal)**: `#00FFCC` (Interactividad, sliders activos, estado de grabación y foco)
* **Color Accent Secundario (Magenta Neón)**: `#FF007F` (Modos de sombra y alertas de sobreexposición)
* **Status Alert / Rec**: `#FF3B30` (Grabación de larga exposición activa)
* **Status Pause**: `#FFB800` (Ámbar brillante para estado en Pausa con overlay activo)
* **High-Contrast Canvas Overlay**: Fusión `Lighten / Max Blend` con ajuste gamma para revisión en pausa.
* **Tipografía**: Sans-Serif moderna geométrica con monospace tabular para valores numéricos (ISO, obturación, histograma, temporizador).

---

## 4. Arquitectura de UI y Flujo de Interacción

```
+-------------------------------------------------------------+
|                       LIVE VIEWFINDER                       |
|   (Camera2 SurfaceTexture / OpenGL ES 3.0 Real-time FBO)    |
|                                                             |
|  [HUD Superior]                                             |
|  - Nivel / Giroscopio     - Histograma RGB      - Battery   |
|  - Widget Voz: [Activo "Escuchando: Play / Pausa"]          |
|                                                             |
|  [HUD Lateral Flotante - Ajustes Manuales Pro]              |
|  - ISO Slider (50 - 6400)                                   |
|  - Shutter Speed (1/8000s - 30s / Bulb Infinito)            |
|  - Enfoque Manual (Focus Peaking Neón Verde)                |
|  - Umbral de Luz (Filtro mín/máx de luminancia)             |
|                                                             |
|  [Superposición de Lienzo Acumulado en Pausa]               |
|  - Congelamiento de buffer con realce de contraste          |
|                                                             |
|  [Barra Inferior de Modos de Renderizado]                   |
|  - [ Pintar con Luz ]  |  [ Pintar con Sombra ]  |  [ Estela / Fade ]
|                                                             |
|  [Barra de Disparo y Control]                               |
|  - Acceso a Intervalómetro  - Obturador Multifunción  - Galería
+-------------------------------------------------------------+
```

### Modos de Renderizado (Shaders OpenGL / RenderScript / Vulkan)
1. **Pintar con Luz (Light Blend / Maximum Mode)**: Mantiene el valor máximo de píxel entre el lienzo acumulado y el fotograma actual ($P_{out} = \max(P_{prev}, P_{curr})$). Excluye sombras y zonas bajo el umbral de luminancia.
2. **Pintar con Sombra (Shadow / Minimum Mode)**: Invierte la regla de acumulación para pintar siluetas oscuras sobre fondos o fuentes de luz ($P_{out} = \min(P_{prev}, P_{curr})$).
3. **Modo Estela (Fade / Decay Trails)**: Aplica un decaimiento exponencial al búfer acumulado ($P_{prev} \times \alpha$) creando estelas de luz dinámicas que se desvanecen gradualmente.

---

## 5. Módulo de Reconocimiento y Control por Voz (Hands-Free)
Para evitar vibraciones y trepidación del trípode al tocar la pantalla:
* **Motor:** `Vosk-Android` / `Android SpeechRecognizer` en modo local sin latencia de red.
* **Palabras clave soportadas:**
  * `"Iniciar"` / `"Play"` $\rightarrow$ Inicia acumulación o reanuda buffer tras pausa.
  * `"Pausa"` $\rightarrow$ Detiene la acumulación manteniendo el sensor en vivo y activa el overlay de contraste.
  * `"Guardar"` $\rightarrow$ Renderiza y guarda el archivo DNG RAW / PNG sin pérdida en disco.
* **Feedback Visual:** Onda de voz reactiva en color Neón `#00FFCC` en la parte superior central.

---

## 6. Panel de Intervalómetro y Automatización
* **Disparo Diferido (Self-Timer)**: 0s, 3s, 5s, 10s.
* **Modo Exposición Fija**: Duración configurable (1s a 60 min).
* **Secuencia de Ciclos (Intervalómetro)**:
  * Retardo previo (Lead-in Delay).
  * Duración de disparo continuo activo.
  * Duración de pausa automática (para preparación del artista / cambio de herramienta de luz).
  * Repeticiones programadas (1 a $\infty$).

---

## 7. Estructura de Proyecto Recomendada (Android Jetpack)
```
app/
 ├── src/main/
 │    ├── java/com/lightpainting/camera/
 │    │    ├── core/
 │    │    │    ├── camera/ (CameraX & Camera2 Manager, ManualControls)
 │    │    │    ├── opengl/ (GLSurfaceView, Shaders, FBO Frame Accumulator)
 │    │    │    └── voice/  (VoiceCommandService, AudioHotwordDetector)
 │    │    ├── domain/model/ (CaptureState, BlendMode, ExposureParams)
 │    │    ├── presentation/
 │    │    │    ├── viewfinder/ (LiveViewScreen, CameraHUD, Sliders)
 │    │    │    ├── automation/ (IntervalometerDialog, TimerConfig)
 │    │    │    └── theme/      (Theme.kt, Color.kt [AMOLED & Neons])
 │    │    └── MainActivity.kt
 │    └── res/
 └── build.gradle.kts
```

---

## 8. Guía de Instalación y Despliegue (README.md rápido)

### Requisitos Previos
* Android Studio Iguana / Jellyfish o superior.
* JDK 17+.
* Dispositivo físico Android 10+ (API level 29+) compatible con `CAMERA2_HARDWARE_LEVEL_FULL` o `LIMITED` para control manual de sensor.

### Pasos de Compilación y Ejecución
1. Clonar el repositorio:
   ```bash
   git clone https://github.com/tu-organizacion/android-lightpainting-camera.git
   cd android-lightpainting-camera
   ```
2. Sincronizar dependencias Gradle:
   ```bash
   ./gradlew clean build
   ```
3. Instalar en el dispositivo de prueba conectado vía ADB:
   ```bash
   ./gradlew installDebug
   ```
4. Aceptar los permisos en tiempo de ejecución de Cámara y Micrófono al primer lanzamiento.
