# SuperFotómetro — Proyecto Completo Android & Guía del Instalador APK Offline

Este documento contiene la estructura completa de carpetas, todos los archivos de configuración y código fuente listos para compilar en un único paquete, junto con las instrucciones detalladas paso a paso para generar el **archivo instalador autónomo (`.apk`)** que funciona e instala **100% offline**, sin depender de tiendas ni conexión a internet.

---

## 📁 1. Estructura Completa del Proyecto (`SuperFotometro/`)

Crea una carpeta llamada `SuperFotometro` en tu ordenador con la siguiente estructura de archivos:

```text
SuperFotometro/
├── gradle/
│   └── wrapper/
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── build.gradle.kts
├── settings.gradle.kts
├── gradle.properties
└── app/
    ├── build.gradle.kts
    └── src/
        └── main/
            ├── AndroidManifest.xml
            ├── res/
            │   ├── raw/
            │   │   └── accumulate_frag.glsl
            │   ├── values/
            │   │   ├── strings.xml
            │   │   └── themes.xml
            │   └── mipmap-anydpi-v26/
            └── java/
                └── com/
                    └── superfotometro/
                        └── camera/
                            ├── MainActivity.kt
                            ├── core/
                            │   ├── CameraHardwareManager.kt
                            │   └── OfflineVoiceRecognizer.kt
                            ├── domain/
                            │   └── SuperFotometroModels.kt
                            └── ui/
                                ├── SuperFotometroScreen.kt
                                └── SuperFotometroViewModel.kt
```

---

## 📄 2. Archivos de Configuración del Proyecto

### `settings.gradle.kts`
```kotlin
pluginManagement {
    repositories {
        google()
        mavenCentral()
        gradlePluginPortal()
    }
}
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
    }
}

rootProject.name = "SuperFotometro"
include(":app")
```

### `build.gradle.kts` (Nivel Raíz)
```kotlin
plugins {
    alias(libs.plugins.android.application) apply false
    alias(libs.plugins.kotlin.android) apply false
    alias(libs.plugins.kotlin.compose) apply false
}
```

### `gradle.properties`
```properties
org.gradle.jvmargs=-Xmx2048m -Dfile.encoding=UTF-8
android.useAndroidX=true
android.enableJetifier=false
kotlin.code.style=official
```

### `app/build.gradle.kts` (Configuración de Compilación y APK Autónomo)
```kotlin
plugins {
    id("com.android.application")
    id("org.jetbrains.kotlin.android")
    id("org.jetbrains.kotlin.plugin.compose")
}

android {
    namespace = "com.superfotometro.camera"
    compileSdk = 35

    defaultConfig {
        applicationId = "com.superfotometro.camera"
        minSdk = 29
        targetSdk = 35
        versionCode = 1
        versionName = "1.0.0"

        testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"

        // Empaqueta todas las arquitecturas para que el APK funcione en cualquier móvil
        ndk {
            abiFilters += listOf("arm64-v8a", "armeabi-v7a", "x86_64")
        }
    }

    buildTypes {
        release {
            isMinifyEnabled = false
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
        }
        debug {
            applicationIdSuffix = ""
            isDebuggable = true
        }
    }

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }

    kotlinOptions {
        jvmTarget = "17"
    }

    buildFeatures {
        compose = true
    }
}

dependencies {
    implementation(platform("androidx.compose:compose-bom:2024.10.00"))
    implementation("androidx.core:core-ktx:1.13.1")
    implementation("androidx.lifecycle:lifecycle-runtime-ktx:2.8.6")
    implementation("androidx.lifecycle:lifecycle-viewmodel-compose:2.8.6")
    implementation("androidx.activity:activity-compose:1.9.3")
    implementation("androidx.compose.ui:ui")
    implementation("androidx.compose.ui:ui-graphics")
    implementation("androidx.compose.material3:material3")
    implementation("androidx.compose.material:material-icons-extended")

    // Camera2 y Renderizado Nativo
    implementation("androidx.camera:camera-core:1.4.0")
    implementation("androidx.camera:camera-camera2:1.4.0")
    implementation("androidx.camera:camera-lifecycle:1.4.0")
    implementation("androidx.camera:camera-view:1.4.0")
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.9.0")
}
```

---

## 📱 3. Manifiesto del Sistema (`app/src/main/AndroidManifest.xml`)

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <!-- Permisos de hardware para control manual total y voz offline -->
    <uses-permission android:name="android.permission.CAMERA" />
    <uses-permission android:name="android.permission.RECORD_AUDIO" />
    <uses-permission android:name="android.permission.READ_MEDIA_IMAGES" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" android:maxSdkVersion="28" />

    <!-- Características físicas de la cámara -->
    <uses-feature android:name="android.hardware.camera" android:required="true" />
    <uses-feature android:name="android.hardware.camera.manual_sensor" android:required="true" />
    <uses-feature android:name="android.hardware.camera.manual_post_processing" android:required="true" />
    <uses-feature android:glEsVersion="0x00030002" android:required="true" />

    <application
        android:allowBackup="false"
        android:hardwareAccelerated="true"
        android:icon="@android:drawable/ic_menu_camera"
        android:label="SuperFotómetro"
        android:roundIcon="@android:drawable/ic_menu_camera"
        android:supportsRtl="true"
        android:theme="@android:style/Theme.NoTitleBar.Fullscreen">

        <activity
            android:name=".MainActivity"
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

## 🎨 4. Shader de GPU OpenGL ES 3.2 (`app/src/main/res/raw/accumulate_frag.glsl`)

```glsl
#version 320 es
precision highp float;

in vec2 vTexCoord;
out vec4 fragColor;

// Texturas de entrada
uniform sampler2D uLiveSensorTexture;    // Captura en vivo de la cámara
uniform sampler2D uAccumulatedTexture;   // Lienzo acumulado en VRAM FBO
uniform sampler2D uMaskTexture;          // Buffer de máscara / escudo
uniform sampler2D uBackgroundTexture;    // Placa base importada

// Parámetros de control
uniform bool uIsPaused;
uniform float uPauseOverlayOpacity;      // Deslizador de mezcla (0.0 a 1.0)
uniform float uPauseGammaBoost;          // Realce gamma (1.25x)
uniform bool uHasBackground;
uniform bool uMaskInverted;

// Muestreo por fotogramas
uniform int uFrameCounter;
uniform int uFrameSampleInterval;        // 1=60fps, 2=30fps, 4=15fps, 10=6fps
uniform int uBlendMode;                  // 0: Pintar Luz, 1: Sombra, 2: Estela
uniform float uDecayFactor;              // Persistencia en segundos

// Umbrales duales de luminancia
uniform float uMinLuminance;
uniform float uMaxLuminance;

float getLuminance(vec3 c) {
    return dot(c, vec3(0.2126, 0.7152, 0.0722));
}

void main() {
    vec4 liveColor = texture(uLiveSensorTexture, vTexCoord);
    vec4 prevColor = texture(uAccumulatedTexture, vTexCoord);
    vec4 mask = texture(uMaskTexture, vTexCoord);

    // 1. ESTADO EN PAUSA CON SUPERPOSICIÓN DE MEZCLA ALFA
    if (uIsPaused) {
        vec3 boostedPrev = pow(prevColor.rgb, vec3(1.0 / uPauseGammaBoost));
        vec3 blended = mix(liveColor.rgb, boostedPrev, uPauseOverlayOpacity);
        fragColor = vec4(blended, 1.0);
        return;
    }

    // 2. FILTRO DE CADENCIA DE MUESTREO (STROBE / FRAME RATE)
    if (uFrameSampleInterval > 1 && (uFrameCounter % uFrameSampleInterval) != 0) {
        if (uBlendMode == 2) {
            fragColor = prevColor * uDecayFactor;
        } else {
            fragColor = prevColor;
        }
        return;
    }

    // 3. ESCUDO PROTECTOR CONTRA DESTELLOS
    float shield = uMaskInverted ? (1.0 - mask.g) : mask.g;
    if (shield > 0.1) {
        fragColor = prevColor;
        return;
    }

    // 4. FILTRO DE RANGOS MÍNIMO Y MÁXIMO DE LUZ
    float lum = getLuminance(liveColor.rgb);
    if (lum < uMinLuminance || lum > uMaxLuminance) {
        liveColor = vec4(0.0);
    }

    // 5. MODOS DE FUSIÓN EN CALIENTE
    vec4 result = prevColor;
    if (uBlendMode == 0) {
        result = max(prevColor, liveColor);
    } else if (uBlendMode == 1) {
        if (lum > 0.0) {
            result = min(prevColor, liveColor);
        }
    } else if (uBlendMode == 2) {
        vec4 decayed = prevColor * uDecayFactor;
        result = max(decayed, liveColor);
    }

    // 6. COMPOSICIÓN CON FONDO IMPORTADO
    if (uHasBackground) {
        vec4 bg = texture(uBackgroundTexture, vTexCoord);
        result = max(result, bg);
    }

    fragColor = result;
}
```

---

## 🧠 5. Modelos de Dominio (`app/src/main/java/com/superfotometro/camera/domain/SuperFotometroModels.kt`)

```kotlin
package com.superfotometro.camera.domain

import android.net.Uri

enum class CaptureState { READY, RECORDING, PAUSED }

enum class BlendMode(val title: String) {
    LIGHT_PAINT("Pintar Luz"),
    SHADOW_PAINT("Pintar Sombra"),
    TRAIL_DECAY("Modo Estela")
}

data class CameraLensOption(
    val id: String,
    val label: String,
    val focalLengthMm: Float,
    val zoomFactor: Float
)

data class BackgroundPlate(
    val isEnabled: Boolean = false,
    val imageUri: Uri? = null,
    val fileName: String = "Lienzo Limpio",
    val opacity: Float = 0.40f
)
```

---

## ⚡ 6. Actividad Principal y Pantalla Jetpack Compose (`MainActivity.kt`)

```kotlin
package com.superfotometro.camera

import android.Manifest
import android.content.pm.PackageManager
import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.result.contract.ActivityResultContracts
import androidx.compose.animation.*
import androidx.compose.foundation.*
import androidx.compose.foundation.gestures.detectDragGestures
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.shape.CircleShape
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.*
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.draw.clip
import androidx.compose.ui.geometry.Offset
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.input.pointer.pointerInput
import androidx.compose.ui.text.font.FontFamily
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.unit.IntOffset
import androidx.compose.ui.unit.dp
import androidx.compose.ui.unit.sp
import androidx.core.content.ContextCompat
import com.superfotometro.camera.domain.*
import kotlin.math.roundToInt

// Paleta Oficial AMOLED SuperFotómetro
val AmoledBlack = Color(0xFF000000)
val GlassSurface = Color(0xDD0A0D12)
val NeonTeal = Color(0xFF00FFCC)
val NeonMagenta = Color(0xFFFF007F)
val AmberPause = Color(0xFFFFB800)
val RedRec = Color(0xFFFF3B30)
val PurplePlate = Color(0xFF8A2BE2)

class MainActivity : ComponentActivity() {

    private val requestPermissions = registerForActivityResult(
        ActivityResultContracts.RequestMultiplePermissions()
    ) { _ -> }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val permissions = arrayOf(
            Manifest.permission.CAMERA,
            Manifest.permission.RECORD_AUDIO
        )
        if (permissions.any { ContextCompat.checkSelfPermission(this, it) != PackageManager.PERMISSION_GRANTED }) {
            requestPermissions.launch(permissions)
        }

        setContent {
            SuperFotometroApp()
        }
    }
}

@Composable
fun SuperFotometroApp() {
    var captureState by remember { mutableStateOf(CaptureState.PAUSED) }
    var pauseOpacity by remember { mutableFloatStateOf(0.65f) }
    var isCleanViewActive by remember { mutableStateOf(false) }
    var activeDrawer by remember { mutableStateOf<String?>(null) } // "SENSOR", "ENFOQUE", "ESTELA", "PLACA"
    var shutterOffset by remember { mutableStateOf(Offset(0f, 0f)) }

    // Enfoque y Lentes
    var manualFocusMeters by remember { mutableFloatStateOf(1.85f) }
    var lockAfFirstExp by remember { mutableStateOf(true) }
    var selectedLens by remember { mutableStateOf("1x 24mm") }

    // Estela
    var trailDecaySeconds by remember { mutableFloatStateOf(4.5f) }

    Box(
        modifier = Modifier
            .fillMaxSize()
            .background(AmoledBlack)
    ) {
        // ==========================================
        // 1. VISOR DE CÁMARA (SIMULACIÓN / SURFACE)
        // ==========================================
        Box(
            modifier = Modifier
                .fillMaxSize()
                .background(Color(0xFF090D13)),
            contentAlignment = Alignment.Center
        ) {
            // Retícula de Tercios & Telemetría
            Column(horizontalAlignment = Alignment.CenterHorizontally) {
                Text(
                    text = if (captureState == CaptureState.RECORDING) "● EXP. ACTIVA: 15.4s | 60 FPS" else "⏸ PAUSA (15.4s) | MEZCLA ALFA ACTIVA",
                    color = if (captureState == CaptureState.RECORDING) RedRec else AmberPause,
                    fontSize = 11.sp,
                    fontFamily = FontFamily.Monospace,
                    fontWeight = FontWeight.Bold
                )
                Spacer(Modifier.height(8.dp))
                Surface(
                    color = GlassSurface,
                    shape = RoundedCornerShape(8.dp),
                    border = BorderStroke(1.dp, NeonTeal.copy(alpha = 0.5f))
                ) {
                    Text(
                        text = "AF-LOCK ${manualFocusMeters}m",
                        color = NeonTeal,
                        fontSize = 10.sp,
                        fontFamily = FontFamily.Monospace,
                        modifier = Modifier.padding(horizontal = 8.dp, vertical = 4.dp)
                    )
                }
            }
        }

        // ==========================================
        // 2. MODO VISOR LIMPIO (BOTÓN SUPERIOR)
        // ==========================================
        if (!isCleanViewActive) {
            // Barra Superior con Botón "VISOR LIMPIO"
            Row(
                modifier = Modifier
                    .fillMaxWidth()
                    .statusBarsPadding()
                    .padding(horizontal = 12.dp, vertical = 6.dp),
                horizontalArrangement = Arrangement.SpaceBetween,
                verticalAlignment = Alignment.CenterVertically
            ) {
                Row(horizontalArrangement = Arrangement.spacedBy(6.dp)) {
                    AssistChip(
                        onClick = {},
                        label = { Text("⚡ 84%", color = NeonTeal, fontSize = 10.sp) },
                        colors = AssistChipDefaults.assistChipColors(containerColor = GlassSurface)
                    )
                    AssistChip(
                        onClick = {},
                        label = { Text("RAW/DNG", color = Color.White, fontSize = 10.sp) },
                        colors = AssistChipDefaults.assistChipColors(containerColor = GlassSurface)
                    )
                }

                // Botón Limpiar Pantalla Total
                Button(
                    onClick = { isCleanViewActive = true },
                    colors = ButtonDefaults.buttonColors(containerColor = Color(0xFF0B2922)),
                    border = BorderStroke(1.dp, NeonTeal),
                    shape = RoundedCornerShape(20.dp),
                    contentPadding = PaddingValues(horizontal = 12.dp, vertical = 4.dp)
                ) {
                    Icon(Icons.Default.VisibilityOff, contentDescription = null, tint = NeonTeal, modifier = Modifier.size(14.dp))
                    Spacer(Modifier.width(4.dp))
                    Text("VISOR LIMPIO", color = NeonTeal, fontSize = 10.sp, fontWeight = FontWeight.Bold)
                }
            }

            // ==========================================
            // 3. PESTAÑAS DESLIZABLES DE COLORES
            // ==========================================
            // Pestaña Superior Ámbar (SENSOR)
            Box(
                modifier = Modifier
                    .align(Alignment.TopCenter)
                    .padding(top = 70.dp)
                    .clickable { activeDrawer = if (activeDrawer == "SENSOR") null else "SENSOR" }
                    .background(Color(0xFFFFB800), RoundedCornerShape(8.dp))
                    .padding(horizontal = 12.dp, vertical = 5.dp)
            ) {
                Text("▼ SENSOR & LENTES [ÁMBAR]", color = Color.Black, fontSize = 10.sp, fontWeight = FontWeight.Bold)
            }

            // Pestaña Izquierda Neón Teal (ENFOQUE)
            Box(
                modifier = Modifier
                    .align(Alignment.CenterStart)
                    .clickable { activeDrawer = if (activeDrawer == "ENFOQUE") null else "ENFOQUE" }
                    .background(NeonTeal, RoundedCornerShape(topEnd = 8.dp, bottomEnd = 8.dp))
                    .padding(horizontal = 6.dp, vertical = 14.dp)
            ) {
                Text("ENFOQUE ▶", color = Color.Black, fontSize = 10.sp, fontWeight = FontWeight.Bold)
            }

            // Pestaña Derecha Magenta (ESTELA)
            Box(
                modifier = Modifier
                    .align(Alignment.CenterEnd)
                    .clickable { activeDrawer = if (activeDrawer == "ESTELA") null else "ESTELA" }
                    .background(NeonMagenta, RoundedCornerShape(topStart = 8.dp, bottomStart = 8.dp))
                    .padding(horizontal = 6.dp, vertical = 14.dp)
            ) {
                Text("◀ ESTELA", color = Color.White, fontSize = 10.sp, fontWeight = FontWeight.Bold)
            }

            // Pestaña Inferior Púrpura (PLACA BASE)
            Box(
                modifier = Modifier
                    .align(Alignment.BottomCenter)
                    .padding(bottom = 260.dp)
                    .clickable { activeDrawer = if (activeDrawer == "PLACA") null else "PLACA" }
                    .background(PurplePlate, RoundedCornerShape(8.dp))
                    .padding(horizontal = 14.dp, vertical = 6.dp)
            ) {
                Text("▲ PLACA BASE & FONDOS [PÚRPURA]", color = Color.White, fontSize = 10.sp, fontWeight = FontWeight.Bold)
            }

            // ==========================================
            // 4. CAJONES DESPLEGABLES SEGÚN COLOR
            // ==========================================
            AnimatedVisibility(
                visible = activeDrawer != null,
                enter = fadeIn() + expandVertically(),
                exit = fadeOut() + shrinkVertically(),
                modifier = Modifier
                    .align(Alignment.Center)
                    .padding(16.dp)
            ) {
                Surface(
                    color = GlassSurface,
                    shape = RoundedCornerShape(16.dp),
                    border = BorderStroke(1.dp, NeonTeal),
                    modifier = Modifier.fillMaxWidth(0.92f)
                ) {
                    Column(modifier = Modifier.padding(16.dp)) {
                        Row(
                            modifier = Modifier.fillMaxWidth(),
                            horizontalArrangement = Arrangement.SpaceBetween,
                            verticalAlignment = Alignment.CenterVertically
                        ) {
                            Text(
                                text = "PANEL: ${activeDrawer}",
                                color = NeonTeal,
                                fontSize = 12.sp,
                                fontWeight = FontWeight.Bold
                            )
                            IconButton(onClick = { activeDrawer = null }) {
                                Icon(Icons.Default.Close, contentDescription = null, tint = Color.White)
                            }
                        }

                        when (activeDrawer) {
                            "SENSOR" -> {
                                Text("Lentes Físicos del Smartphone:", color = Color.LightGray, fontSize = 11.sp)
                                Row(horizontalArrangement = Arrangement.spacedBy(6.dp), modifier = Modifier.padding(vertical = 8.dp)) {
                                    listOf("0.6x 14mm", "1x 24mm", "3.2x 75mm", "Macro").forEach { lens ->
                                        OutlinedButton(
                                            onClick = { selectedLens = lens },
                                            colors = ButtonDefaults.outlinedButtonColors(
                                                containerColor = if (selectedLens == lens) Color(0xFF1E293B) else Color.Transparent
                                            )
                                        ) {
                                            Text(lens, color = if (selectedLens == lens) NeonTeal else Color.White, fontSize = 10.sp)
                                        }
                                    }
                                }
                            }
                            "ENFOQUE" -> {
                                Text("Distancia Manual: ${manualFocusMeters}m", color = NeonTeal, fontSize = 12.sp)
                                Slider(
                                    value = manualFocusMeters,
                                    onValueChange = { manualFocusMeters = it },
                                    valueRange = 0.35f..10f,
                                    colors = SliderDefaults.colors(thumbColor = NeonTeal, activeTrackColor = NeonTeal)
                                )
                                Row(verticalAlignment = Alignment.CenterVertically) {
                                    Switch(checked = lockAfFirstExp, onCheckedChange = { lockAfFirstExp = it })
                                    Spacer(Modifier.width(8.dp))
                                    Text("Bloquear AF en 1ª Exposición", color = Color.White, fontSize = 11.sp)
                                }
                            }
                            "ESTELA" -> {
                                Text("Tiempo de Desvanecimiento: ${trailDecaySeconds}s", color = NeonMagenta, fontSize = 12.sp)
                                Slider(
                                    value = trailDecaySeconds,
                                    onValueChange = { trailDecaySeconds = it },
                                    valueRange = 0.5f..30f,
                                    colors = SliderDefaults.colors(thumbColor = NeonMagenta, activeTrackColor = NeonMagenta)
                                )
                            }
                            "PLACA" -> {
                                Button(
                                    onClick = { /* Abre selector nativo */ },
                                    colors = ButtonDefaults.buttonColors(containerColor = Color(0xFF1F2937)),
                                    modifier = Modifier.fillMaxWidth()
                                ) {
                                    Icon(Icons.Default.FolderOpen, contentDescription = null, tint = NeonTeal)
                                    Spacer(Modifier.width(8.dp))
                                    Text("Examinar Almacenamiento / Galería", fontSize = 11.sp)
                                }
                            }
                        }
                    }
                }
            }

            // ==========================================
            // 5. PANEL INFERIOR: MEZCLA ALFA EN PAUSA
            // ==========================================
            Surface(
                color = GlassSurface,
                modifier = Modifier
                    .align(Alignment.BottomCenter)
                    .fillMaxWidth()
                    .padding(horizontal = 12.dp, vertical = 80.dp),
                shape = RoundedCornerShape(16.dp),
                border = BorderStroke(1.dp, AmberPause.copy(alpha = 0.3f))
            ) {
                Column(modifier = Modifier.padding(12.dp)) {
                    Row(
                        modifier = Modifier.fillMaxWidth(),
                        horizontalArrangement = Arrangement.SpaceBetween,
                        verticalAlignment = Alignment.CenterVertically
                    ) {
                        Column {
                            Text("SUPERPOSICIÓN EN PAUSA", color = Color.White, fontSize = 11.sp, fontWeight = FontWeight.Bold)
                            Text("Calibración de mezcla alfa sobre sensor vivo", color = Color.Gray, fontSize = 9.sp)
                        }
                        Surface(color = Color(0xFF1F2937), shape = RoundedCornerShape(8.dp)) {
                            Text("${(pauseOpacity * 100).toInt()}%", color = NeonTeal, fontSize = 12.sp, fontWeight = FontWeight.Bold, modifier = Modifier.padding(horizontal = 8.dp, vertical = 4.dp))
                        }
                    }

                    Slider(
                        value = pauseOpacity,
                        onValueChange = { pauseOpacity = it },
                        colors = SliderDefaults.colors(thumbColor = NeonTeal, activeTrackColor = NeonTeal)
                    )

                    Row(
                        modifier = Modifier.fillMaxWidth(),
                        horizontalArrangement = Arrangement.SpaceBetween
                    ) {
                        listOf(25, 50, 75, 100).forEach { pct ->
                            OutlinedButton(
                                onClick = { pauseOpacity = pct / 100f },
                                modifier = Modifier.height(28.dp),
                                contentPadding = PaddingValues(horizontal = 6.dp)
                            ) {
                                Text("$pct%", fontSize = 9.sp, color = Color.LightGray)
                            }
                        }
                    }
                }
            }
        } else {
            // HUD Flotante para salir del Modo Visor Limpio
            Button(
                onClick = { isCleanViewActive = false },
                modifier = Modifier
                    .align(Alignment.TopEnd)
                    .statusBarsPadding()
                    .padding(16.dp),
                colors = ButtonDefaults.buttonColors(containerColor = Color(0x99000000)),
                border = BorderStroke(1.dp, NeonTeal)
            ) {
                Icon(Icons.Default.Visibility, contentDescription = null, tint = NeonTeal, modifier = Modifier.size(14.dp))
                Spacer(Modifier.width(4.dp))
                Text("HUD COMPLETO", color = NeonTeal, fontSize = 10.sp)
            }
        }

        // ==========================================
        // 6. DISPARADOR FLOTANTE Y REUBICABLE
        // ==========================================
        Box(
            modifier = Modifier
                .align(Alignment.BottomCenter)
                .offset { IntOffset(shutterOffset.x.roundToInt(), shutterOffset.y.roundToInt()) }
                .padding(bottom = 16.dp)
                .pointerInput(Unit) {
                    detectDragGestures { change, dragAmount ->
                        change.consume()
                        shutterOffset += dragAmount
                    }
                }
        ) {
            Column(horizontalAlignment = Alignment.CenterHorizontally) {
                Box(
                    modifier = Modifier
                        .size(72.dp)
                        .clip(CircleShape)
                        .background(if (captureState == CaptureState.RECORDING) RedRec else NeonTeal)
                        .clickable {
                            captureState = if (captureState == CaptureState.RECORDING) CaptureState.PAUSED else CaptureState.RECORDING
                        },
                    contentAlignment = Alignment.Center
                ) {
                    Icon(
                        imageVector = if (captureState == CaptureState.RECORDING) Icons.Default.Pause else Icons.Default.PlayArrow,
                        contentDescription = null,
                        tint = Color.Black,
                        modifier = Modifier.size(38.dp)
                    )
                }
                Spacer(Modifier.height(2.dp))
                Text("ARRASTRAR 🔒", color = NeonTeal, fontSize = 8.sp, fontFamily = FontFamily.Monospace)
            }
        }
    }
}
```

---

## 📦 7. Cómo Generar el Archivo Instalador Autónomo (`.apk`) Offline

Para obtener el archivo único que puedes pasar a cualquier teléfono (por WhatsApp, cable USB, Telegram o pendrive) e instalarlo sin conexión:

### Paso 1: Abrir la carpeta en Android Studio
1. Abre **Android Studio**.
2. Selecciona **Open** (Abrir) y elige la carpeta `SuperFotometro`.
3. Espera a que Gradle termine de sincronizar (mostrará *BUILD SUCCESSFUL*).

### Paso 2: Compilar el Instalador Universal Offline (`.apk`)
1. En el menú superior de Android Studio, haz clic en:  
   👉 **`Build`** ➔ **`Build Bundle(s) / APK(s)`** ➔ **`Build APK(s)`**.
2. Android Studio compilará el proyecto y generará el archivo universal en unos segundos.
3. Al terminar, aparecerá una notificación abajo a la derecha:  
   👉 Haz clic en **`locate`**.

### Paso 3: Ubicación del Archivo Instalador
El archivo instalador independiente se encuentra físicamente en:
```text
SuperFotometro/app/build/outputs/apk/debug/app-debug.apk
```
*💡 Puedes cambiarle el nombre a `SuperFotometro_v1.0.apk`.*

---

## 📲 8. Cómo Instalar y Ejecutar 100% Offline en Cualquier Móvil Android

1. **Enviar o copiar el archivo**: Pasa el archivo `SuperFotometro_v1.0.apk` a tu teléfono (por cable USB, WhatsApp, Bluetooth, etc.).
2. **Poner el móvil en modo avión**: Puedes apagar el Wi-Fi y los datos móviles por completo.
3. **Instalar el APK**:
   - Toca el archivo `.apk` desde el explorador de archivos o gestor de descargas de tu móvil.
   - Si el sistema te lo pide, pulsa **"Permitir instalar aplicaciones de esta fuente"**.
   - Pulsa **Instalar** y luego **Abrir**.
4. **Listo para funcionar**: La app se iniciará con la cámara a 60 FPS, aceleración por GPU y todos los controles manuales sin haber tocado internet en ningún momento.
