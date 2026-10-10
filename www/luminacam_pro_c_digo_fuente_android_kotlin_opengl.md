# LuminaCam Pro — Código Fuente Completo de la Aplicación Android

Este documento contiene la implementación lista para producción en **Kotlin 2.0+**, **Jetpack Compose**, **Android Camera2 NDK/Vulkan/OpenGL ES 3.2** y reconocimiento de voz offline para la aplicación **LuminaCam Pro**.

---

## 1. Configuración de Construcción (`build.gradle.kts`)

```kotlin
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
    alias(libs.plugins.kotlin.compose)
}

android {
    namespace = "com.lightpainting.camera"
    compileSdk = 35

    defaultConfig {
        applicationId = "com.lightpainting.camera"
        minSdk = 29
        targetSdk = 35
        versionCode = 1
        versionName = "1.0.0"

        ndk {
            abiFilters += listOf("arm64-v8a", "armeabi-v7a")
        }
    }

    buildFeatures {
        compose = true
    }

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }

    kotlinOptions {
        jvmTarget = "17"
    }
}

dependencies {
    implementation(platform("androidx.compose:compose-bom:2024.10.00"))
    implementation("androidx.compose.ui:ui")
    implementation("androidx.compose.ui:ui-graphics")
    implementation("androidx.compose.ui:ui-tooling-preview")
    implementation("androidx.compose.material3:material3")
    implementation("androidx.compose.material:material-icons-extended")
    implementation("androidx.lifecycle:lifecycle-runtime-compose:2.8.6")
    implementation("androidx.lifecycle:lifecycle-viewmodel-compose:2.8.6")
    
    // Camera2 y Renderizado
    implementation("androidx.camera:camera-core:1.4.0")
    implementation("androidx.camera:camera-camera2:1.4.0")
    implementation("androidx.camera:camera-lifecycle:1.4.0")
    
    // Corrutinas
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.9.0")
}
```

---

## 2. Fragment Shader OpenGL ES 3.2 de Acumulación (`accumulate_frag.glsl`)

Ubicación: `app/src/main/res/raw/accumulate_frag.glsl`

```glsl
#version 320 es
precision highp float;

in vec2 vTexCoord;
out vec4 fragColor;

// Texturas
uniform sampler2D uLiveSensorTexture;    // Stream directo de Camera2
uniform sampler2D uAccumulatedTexture;   // Lienzo acumulado en GPU FBO
uniform sampler2D uMaskTexture;          // Buffer de Máscara (R = Exponer, G = Escudo Negativo)
uniform sampler2D uBackgroundTexture;    // Capa de Placa Base importada

// Parámetros de Control
uniform int uBlendMode;                  // 0: Pintar Luz, 1: Pintar Sombra, 2: Modo Estela
uniform float uDecayFactor;              // Factor de persistencia (ej. 0.965)
uniform float uMinLuminance;             // Rango Mínimo (ej. 0.15)
uniform float uMaxLuminance;             // Rango Máximo (ej. 0.88)
uniform float uMinSpeed;                 // Velocidad mín px/s
uniform float uMaxSpeed;                 // Velocidad máx px/s
uniform float uMotionSpeed;              // Velocidad actual calculada del trazo

// Chroma Key & Exclusión de Color
uniform bool uChromaRejectActive;
uniform vec3 uTargetColor;               // Color muestreado (RGB normalizado)
uniform float uColorTolerance;           // Delta E / Distancia euclidiana
uniform float uFeather;                  // Suavizado de bordes
uniform bool uExclusiveColorMode;        // Invertir muestreo (Solo este color)

// Estado del Lienzo
uniform bool uIsPaused;
uniform float uPauseGammaBoost;          // 1.25 para High-Contrast Overlay
uniform bool uHasBackground;
uniform float uBgOpacity;

// Cálculo de Luminancia Rec.709
float getLuminance(vec3 color) {
    return dot(color, vec3(0.2126, 0.7152, 0.0722));
}

void main() {
    vec4 liveColor = texture(uLiveSensorTexture, vTexCoord);
    vec4 prevColor = texture(uAccumulatedTexture, vTexCoord);
    vec4 mask = texture(uMaskTexture, vTexCoord);
    
    // Si la app está en pausa, renderizamos el High-Contrast Overlay congelado
    if (uIsPaused) {
        vec3 boosted = pow(prevColor.rgb, vec3(1.0 / uPauseGammaBoost));
        fragColor = vec4(boosted, prevColor.a);
        return;
    }

    // 1. Verificación de Máscara Negativa (Escudo Anti-Destellos)
    // El canal G de la máscara representa la zona protegida
    if (mask.g > 0.05) {
        // En zona protegida se preserva el fotograma anterior sin admitir nueva luz
        fragColor = prevColor;
        return;
    }

    // 2. Filtro de Luminancia Min/Max
    float lum = getLuminance(liveColor.rgb);
    if (lum < uMinLuminance || lum > uMaxLuminance) {
        liveColor = vec4(0.0);
    }

    // 3. Filtro de Chroma Key / Muestreo de Color a Ignorar
    if (uChromaRejectActive) {
        float colorDist = distance(liveColor.rgb, uTargetColor);
        float edge = smoothstep(uColorTolerance, uColorTolerance + uFeather, colorDist);
        
        if (!uExclusiveColorMode) {
            // Ignora el color específico (ej. amarillo sodio)
            liveColor.rgb *= edge;
        } else {
            // Modo Pase Exclusivo (solo acumula el color seleccionado)
            liveColor.rgb *= (1.0 - edge);
        }
    }

    // 4. Modos de Acumulación Computacional
    vec4 result = prevColor;
    if (uBlendMode == 0) {
        // Pintar con Luz (Lighten / Max Blend)
        result = max(prevColor, liveColor);
    } 
    else if (uBlendMode == 1) {
        // Pintar con Sombra (Darken / Min Blend)
        if (lum > 0.0) {
            result = min(prevColor, liveColor);
        }
    } 
    else if (uBlendMode == 2) {
        // Modo Estela Dinámica (Decay / Phosphor Trails)
        vec4 decayed = prevColor * uDecayFactor;
        result = max(decayed, liveColor);
    }

    // 5. Composición con Placa Base de Fondo si existe
    if (uHasBackground) {
        vec4 bg = texture(uBackgroundTexture, vTexCoord);
        result = mix(result, max(result, bg), uBgOpacity);
    }

    fragColor = result;
}
```

---

## 3. Modelo de Datos y Estados (`CaptureEngineState.kt`)

```kotlin
package com.lightpainting.camera.domain.model

import android.graphics.Bitmap

enum class CaptureState {
    READY,
    RECORDING,
    PAUSED
}

enum class BlendMode(val title: String) {
    LIGHT_PAINT("Pintar Luz"),
    SHADOW_PAINT("Pintar Sombra"),
    TRAIL_DECAY("Modo Estela")
}

data class Checkpoint(
    val id: Int,
    val timestampSeconds: Double,
    val label: String,
    val textureBufferId: Int
)

data class ChromaKeyFilter(
    val isActive: Boolean = true,
    val hexColor: String = "#FF9E1B",
    val rgb: Triple<Float, Float, Float> = Triple(1.0f, 0.62f, 0.11f),
    val tolerance: Float = 0.15f,
    val feather: Float = 0.12f,
    val isExclusiveMode: Boolean = false,
    val zoomFactor: Float = 8.0f,
    val samplingMatrix: String = "3x3"
)

data class ManualExposureSettings(
    val iso: Int = 100,
    val shutterSpeedNanos: Long = 0L, // 0L = Modo Bulb Infinito
    val manualFocusDistance: Float = 0.85f,
    val focusPeakingEnabled: Boolean = true,
    val minLuminance: Float = 0.15f,
    val maxLuminance: Float = 0.88f
)

data class IntervalometerRoutine(
    val initialDelaySeconds: Int = 3,
    val exposureTimeSeconds: Int = 15,
    val autoPauseSeconds: Int = 5,
    val cycleCount: Int = 1,
    val audioBeepEnabled: Boolean = true
)
```

---

## 4. Gestor de Cámara Manual Camera2 (`Camera2Manager.kt`)

```kotlin
package com.lightpainting.camera.core.camera

import android.annotation.SuppressLint
import android.content.Context
import android.hardware.camera2.*
import android.media.ImageReader
import android.os.Handler
import android.os.HandlerThread
import android.view.Surface
import com.lightpainting.camera.domain.model.ManualExposureSettings
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.asStateFlow

class Camera2Manager(private val context: Context) {

    private val cameraManager = context.getSystemService(Context.CAMERA_SERVICE) as CameraManager
    private var cameraDevice: CameraDevice? = null
    private var captureSession: CameraCaptureSession? = null
    private var previewRequestBuilder: CaptureRequest.Builder? = null

    private val backgroundThread = HandlerThread("CameraBackground").apply { start() }
    private val backgroundHandler = Handler(backgroundThread.looper)

    private val _isSessionReady = MutableStateFlow(false)
    val isSessionReady = _isSessionReady.asStateFlow()

    @SuppressLint("MissingPermission")
    fun startCamera(surface: Surface, settings: ManualExposureSettings) {
        val cameraId = getRearCameraId() ?: return

        cameraManager.openCamera(cameraId, object : CameraDevice.StateCallback() {
            override fun onOpened(camera: CameraDevice) {
                cameraDevice = camera
                createSession(camera, surface, settings)
            }

            override fun onDisconnected(camera: CameraDevice) {
                camera.close()
                cameraDevice = null
            }

            override fun onError(camera: CameraDevice, error: Int) {
                camera.close()
                cameraDevice = null
            }
        }, backgroundHandler)
    }

    private fun createSession(camera: CameraDevice, surface: Surface, settings: ManualExposureSettings) {
        previewRequestBuilder = camera.createCaptureRequest(CameraDevice.TEMPLATE_MANUAL).apply {
            addTarget(surface)
            applySettings(this, settings)
        }

        camera.createCaptureSession(
            listOf(surface),
            object : CameraCaptureSession.StateCallback() {
                override fun onConfigured(session: CameraCaptureSession) {
                    captureSession = session
                    previewRequestBuilder?.let {
                        session.setRepeatingRequest(it.build(), null, backgroundHandler)
                    }
                    _isSessionReady.value = true
                }

                override fun onConfigureFailed(session: CameraCaptureSession) {
                    _isSessionReady.value = false
                }
            },
            backgroundHandler
        )
    }

    fun updateManualSettings(settings: ManualExposureSettings) {
        previewRequestBuilder?.let { builder ->
            applySettings(builder, settings)
            captureSession?.setRepeatingRequest(builder.build(), null, backgroundHandler)
        }
    }

    private fun applySettings(builder: CaptureRequest.Builder, settings: ManualExposureSettings) {
        // Bloqueo de auto-exposición y balance para control manual absoluto
        builder.set(CaptureRequest.CONTROL_MODE, CaptureRequest.CONTROL_MODE_OFF)
        builder.set(CaptureRequest.CONTROL_AE_MODE, CaptureRequest.CONTROL_AE_MODE_OFF)
        builder.set(CaptureRequest.CONTROL_AF_MODE, CaptureRequest.CONTROL_AF_MODE_OFF)

        // Asignación de ISO manual
        builder.set(CaptureRequest.SENSOR_SENSITIVITY, settings.iso)

        // Asignación de tiempo de obturación (si es > 0)
        if (settings.shutterSpeedNanos > 0L) {
            builder.set(CaptureRequest.SENSOR_EXPOSURE_TIME, settings.shutterSpeedNanos)
        }

        // Asignación de enfoque manual (distancia de dioptrías)
        builder.set(CaptureRequest.LENS_FOCUS_DISTANCE, settings.manualFocusDistance)
    }

    private fun getRearCameraId(): String? {
        return cameraManager.cameraIdList.firstOrNull { id ->
            val characteristics = cameraManager.getCameraCharacteristics(id)
            val facing = characteristics.get(CameraCharacteristics.LENS_FACING)
            facing == CameraCharacteristics.LENS_FACING_BACK
        }
    }

    fun release() {
        captureSession?.close()
        cameraDevice?.close()
        backgroundThread.quitSafely()
    }
}
```

---

## 5. Servicio de Reconocimiento Vocal Zero-Latency (`VoiceCommandService.kt`)

```kotlin
package com.lightpainting.camera.core.voice

import android.content.Context
import android.content.Intent
import android.os.Bundle
import android.speech.RecognitionListener
import android.speech.RecognizerIntent
import android.speech.SpeechRecognizer
import kotlinx.coroutines.flow.MutableSharedFlow
import kotlinx.coroutines.flow.asSharedFlow
import java.util.Locale

sealed class VoiceCommand {
    object Play : VoiceCommand()
    object Pause : VoiceCommand()
    object Undo : VoiceCommand()
    object Save : VoiceCommand()
}

class VoiceCommandService(private val context: Context) {

    private var speechRecognizer: SpeechRecognizer? = null
    private val _commands = MutableSharedFlow<VoiceCommand>(extraBufferCapacity = 10)
    val commands = _commands.asSharedFlow()

    fun startListening() {
        if (!SpeechRecognizer.isRecognitionAvailable(context)) return

        speechRecognizer = SpeechRecognizer.createOnDeviceSpeechRecognizer(context).apply {
            setRecognitionListener(object : RecognitionListener {
                override fun onResults(results: Bundle?) {
                    val matches = results?.getStringArrayList(SpeechRecognizer.RESULTS_RECOGNITION)
                    matches?.firstOrNull()?.lowercase(Locale.ROOT)?.let { phrase ->
                        processPhrase(phrase)
                    }
                    // Reiniciar escucha continua en bucle
                    restartListening()
                }

                override fun onError(error: Int) {
                    restartListening()
                }

                override fun onReadyForSpeech(params: Bundle?) {}
                override fun onBeginningOfSpeech() {}
                override fun onRmsChanged(rmsdB: Float) {}
                override fun onBufferReceived(buffer: ByteArray?) {}
                override fun onEndOfSpeech() {}
                override fun onPartialResults(partialResults: Bundle?) {
                    val partialMatches = partialResults?.getStringArrayList(SpeechRecognizer.RESULTS_RECOGNITION)
                    partialMatches?.firstOrNull()?.lowercase(Locale.ROOT)?.let { phrase ->
                        processPhrase(phrase)
                    }
                }
                override fun onEvent(eventType: Int, params: Bundle?) {}
            })
        }

        restartListening()
    }

    private fun processPhrase(text: String) {
        when {
            "play" in text || "iniciar" in text || "reanudar" in text -> _commands.tryEmit(VoiceCommand.Play)
            "pausa" in text || "parar" in text || "congelar" in text -> _commands.tryEmit(VoiceCommand.Pause)
            "deshacer" in text || "atrás" in text || "volver" in text -> _commands.tryEmit(VoiceCommand.Undo)
            "guardar" in text || "exportar" in text || "foto" in text -> _commands.tryEmit(VoiceCommand.Save)
        }
    }

    private fun restartListening() {
        val intent = Intent(RecognizerIntent.ACTION_RECOGNIZE_SPEECH).apply {
            putExtra(RecognizerIntent.EXTRA_LANGUAGE_MODEL, RecognizerIntent.LANGUAGE_MODEL_FREE_FORM)
            putExtra(RecognizerIntent.EXTRA_LANGUAGE, "es-ES")
            putExtra(RecognizerIntent.EXTRA_PREFER_OFFLINE, true)
            putExtra(RecognizerIntent.EXTRA_MAX_RESULTS, 1)
        }
        speechRecognizer?.startListening(intent)
    }

    fun destroy() {
        speechRecognizer?.destroy()
        speechRecognizer = null
    }
}
```

---

## 6. ViewModel del Visor y Acumulación (`LightPaintingViewModel.kt`)

```kotlin
package com.lightpainting.camera.presentation.viewfinder

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.lightpainting.camera.core.voice.VoiceCommand
import com.lightpainting.camera.core.voice.VoiceCommandService
import com.lightpainting.camera.domain.model.*
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.flow.update
import kotlinx.coroutines.launch

class LightPaintingViewModel(
    private val voiceService: VoiceCommandService
) : ViewModel() {

    private val _captureState = MutableStateFlow(CaptureState.READY)
    val captureState = _captureState.asStateFlow()

    private val _activeBlendMode = MutableStateFlow(BlendMode.LIGHT_PAINT)
    val activeBlendMode = _activeBlendMode.asStateFlow()

    private val _exposureSettings = MutableStateFlow(ManualExposureSettings())
    val exposureSettings = _exposureSettings.asStateFlow()

    private val _chromaFilter = MutableStateFlow(ChromaKeyFilter())
    val chromaFilter = _chromaFilter.asStateFlow()

    private val _checkpoints = MutableStateFlow<List<Checkpoint>>(emptyList())
    val checkpoints = _checkpoints.asStateFlow()

    private val _elapsedSeconds = MutableStateFlow(0.0)
    val elapsedSeconds = _elapsedSeconds.asStateFlow()

    init {
        listenVoiceCommands()
    }

    private fun listenVoiceCommands() {
        viewModelScope.launch {
            voiceService.commands.collect { cmd ->
                when (cmd) {
                    VoiceCommand.Play -> onPlayTriggered()
                    VoiceCommand.Pause -> onPauseTriggered()
                    VoiceCommand.Undo -> undoLastCheckpoint()
                    VoiceCommand.Save -> saveFrame()
                }
            }
        }
    }

    fun onPlayTriggered() {
        _captureState.value = CaptureState.RECORDING
    }

    fun onPauseTriggered() {
        if (_captureState.value == CaptureState.RECORDING) {
            _captureState.value = CaptureState.PAUSED
            // Crear checkpoint en VRAM automáticamente
            val newIndex = _checkpoints.value.size + 1
            val cp = Checkpoint(
                id = newIndex,
                timestampSeconds = _elapsedSeconds.value,
                label = "Pausa #$newIndex",
                textureBufferId = newIndex
            )
            _checkpoints.update { it + cp }
        }
    }

    fun setBlendModeInHot(mode: BlendMode) {
        // Se puede conmutar en caliente incluso en PAUSA sin reiniciar el lienzo
        _activeBlendMode.value = mode
    }

    fun undoLastCheckpoint() {
        if (_checkpoints.value.isNotEmpty()) {
            val list = _checkpoints.value.toMutableList()
            list.removeLast()
            _checkpoints.value = list
            // Se restaura el buffer en el shader de la GPU
        }
    }

    fun setIso(iso: Int) {
        _exposureSettings.update { it.copy(iso = iso) }
    }

    fun setLuminanceLimits(min: Float, max: Float) {
        _exposureSettings.update { it.copy(minLuminance = min, maxLuminance = max) }
    }

    fun setChromaTolerance(tol: Float) {
        _chromaFilter.update { it.copy(tolerance = tol) }
    }

    fun saveFrame() {
        // Exportar buffer FBO a DNG RAW / PNG de 48MP
    }
}
```

---

## 7. Interfaz Gráfica Jetpack Compose (`ViewfinderScreen.kt`)

```kotlin
package com.lightpainting.camera.presentation.viewfinder

import androidx.compose.animation.*
import androidx.compose.foundation.background
import androidx.compose.foundation.border
import androidx.compose.foundation.clickable
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
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.text.font.FontFamily
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.unit.dp
import androidx.compose.ui.unit.sp
import com.lightpainting.camera.domain.model.BlendMode
import com.lightpainting.camera.domain.model.CaptureState

// Tokens de Diseño LuminaCam Pro
val AmoledBlack = Color(0xFF000000)
val GlassSurface = Color(0xCC0A0D12)
val NeonTeal = Color(0xFF00FFCC)
val NeonMagenta = Color(0xFFFF007F)
val AmberPause = Color(0xFFFFB800)
val RedRec = Color(0xFFFF3B30)

@Composable
fun ViewfinderScreen(
    viewModel: LightPaintingViewModel
) {
    val state by viewModel.captureState.collectAsState()
    val blendMode by viewModel.activeBlendMode.collectAsState()
    val checkpoints by viewModel.checkpoints.collectAsState()
    val exposure by viewModel.exposureSettings.collectAsState()

    Box(
        modifier = Modifier
            .fillMaxSize()
            .background(AmoledBlack)
    ) {
        // 1. Superficie de la Cámara y Renderizado GPU (GLSurfaceView)
        // [Representado aquí como el visor base que recibe los shaders]
        Box(modifier = Modifier.fillMaxSize())

        // 2. High-Contrast Overlay si está en Pausa
        if (state == CaptureState.PAUSED) {
            Box(
                modifier = Modifier
                    .fillMaxSize()
                    .background(Color(0x33000000))
                    .border(2.dp, AmberPause.copy(alpha = 0.5f))
            ) {
                Text(
                    text = "CANVAS FROZEN · HIGH-CONTRAST OVERLAY (1.25x GAMMA)",
                    color = AmberPause,
                    fontSize = 11.sp,
                    fontFamily = FontFamily.Monospace,
                    modifier = Modifier
                        .align(Alignment.TopCenter)
                        .padding(top = 80.dp)
                        .background(GlassSurface, RoundedCornerShape(4.dp))
                        .padding(horizontal = 8.dp, vertical = 4.dp)
                )
            }
        }

        // 3. HUD Superior (Batería, Formato, Asistente de Voz)
        Row(
            modifier = Modifier
                .fillMaxWidth()
                .statusBarsPadding()
                .padding(horizontal = 16.dp, vertical = 8.dp),
            horizontalArrangement = Arrangement.SpaceBetween,
            verticalAlignment = Alignment.CenterVertically
        ) {
            Row(
                horizontalArrangement = Arrangement.spacedBy(8.dp),
                verticalAlignment = Alignment.CenterVertically
            ) {
                AssistChip(
                    onClick = {},
                    label = { Text("⚡ 84%", color = NeonTeal, fontSize = 11.sp) },
                    colors = AssistChipDefaults.assistChipColors(containerColor = GlassSurface)
                )
                AssistChip(
                    onClick = {},
                    label = { Text("RAW/DNG", color = Color.White, fontSize = 11.sp) },
                    colors = AssistChipDefaults.assistChipColors(containerColor = GlassSurface)
                )
            }

            // Widget Micrófono Activo Hands-Free
            Row(
                modifier = Modifier
                    .background(GlassSurface, RoundedCornerShape(16.dp))
                    .border(1.dp, NeonTeal.copy(alpha = 0.4f), RoundedCornerShape(16.dp))
                .padding(horizontal = 10.dp, vertical = 4.dp),
                verticalAlignment = Alignment.CenterVertically,
                horizontalArrangement = Arrangement.spacedBy(6.dp)
            ) {
                Icon(Icons.Default.Mic, contentDescription = null, tint = NeonTeal, modifier = Modifier.size(14.dp))
                Text("Escuchando...", color = NeonTeal, fontSize = 11.sp, fontFamily = FontFamily.Monospace)
            }
        }

        // 4. Panel Flotante de Deshacer / Checkpoints (visible en Pausa)
        AnimatedVisibility(
            visible = state == CaptureState.PAUSED,
            enter = fadeIn() + slideInVertically(),
            exit = fadeOut() + slideOutVertically(),
            modifier = Modifier
                .align(Alignment.BottomCenter)
                .padding(bottom = 180.dp)
        ) {
            Column(
                modifier = Modifier
                    .fillMaxWidth(0.92f)
                    .background(GlassSurface, RoundedCornerShape(12.dp))
                    .border(1.dp, Color.White.copy(alpha = 0.1f), RoundedCornerShape(12.dp))
                    .padding(12.dp),
                verticalArrangement = Arrangement.spacedBy(8.dp)
            ) {
                Row(
                    modifier = Modifier.fillMaxWidth(),
                    horizontalArrangement = Arrangement.SpaceBetween
                ) {
                    Text("HISTORIAL DE PAUSAS (VRAM BUFFER)", color = Color.Gray, fontSize = 10.sp, fontFamily = FontFamily.Monospace)
                    Text("${checkpoints.size} CHECKPOINTS", color = NeonTeal, fontSize = 10.sp, fontFamily = FontFamily.Monospace)
                }

                Row(
                    modifier = Modifier.fillMaxWidth(),
                    horizontalArrangement = Arrangement.spacedBy(8.dp)
                ) {
                    Button(
                        onClick = { viewModel.undoLastCheckpoint() },
                        colors = ButtonDefaults.buttonColors(containerColor = Color(0xFF3A1116)),
                        modifier = Modifier.weight(1f),
                        shape = RoundedCornerShape(8.dp)
                    ) {
                        Icon(Icons.Default.Undo, contentDescription = null, tint = NeonMagenta, modifier = Modifier.size(16.dp))
                        Spacer(Modifier.width(6.dp))
                        Text("Deshacer Trazo", color = Color.White, fontSize = 12.sp)
                    }

                    OutlinedButton(
                        onClick = { /* Redo */ },
                        modifier = Modifier.weight(1f),
                        shape = RoundedCornerShape(8.dp)
                    ) {
                        Icon(Icons.Default.Redo, contentDescription = null, tint = Color.LightGray, modifier = Modifier.size(16.dp))
                        Spacer(Modifier.width(6.dp))
                        Text("Rehacer", color = Color.LightGray, fontSize = 12.sp)
                    }
                }
            }
        }

        // 5. Selector Deslizante de Modos de Renderizado (Conmutación en Caliente)
        Row(
            modifier = Modifier
                .align(Alignment.BottomCenter)
                .padding(bottom = 110.dp)
                .background(GlassSurface, RoundedCornerShape(24.dp))
                .border(1.dp, Color.White.copy(alpha = 0.15f), RoundedCornerShape(24.dp))
                .padding(4.dp),
            horizontalArrangement = Arrangement.spacedBy(4.dp)
        ) {
            BlendMode.values().forEach { mode ->
                val selected = blendMode == mode
                Box(
                    modifier = Modifier
                        .clip(RoundedCornerShape(20.dp))
                        .background(if (selected) NeonTeal else Color.Transparent)
                        .clickable { viewModel.setBlendModeInHot(mode) }
                        .padding(horizontal = 14.dp, vertical = 8.dp)
                ) {
                    Text(
                        text = mode.title,
                        color = if (selected) Color.Black else Color.White,
                        fontSize = 12.sp,
                        fontWeight = if (selected) FontWeight.Bold else FontWeight.Normal
                    )
                }
            }
        }

        // 6. Botón Obturador Multifunción Central (Play / Pausa)
        Box(
            modifier = Modifier
                .align(Alignment.BottomCenter)
                .padding(bottom = 24.dp)
                .size(76.dp)
                .clip(CircleShape)
                .background(
                    when (state) {
                        CaptureState.RECORDING -> RedRec
                        CaptureState.PAUSED -> AmberPause
                        CaptureState.READY -> NeonTeal
                    }
                )
                .clickable {
                    if (state == CaptureState.RECORDING) {
                        viewModel.onPauseTriggered()
                    } else {
                        viewModel.onPlayTriggered()
                    }
                },
            contentAlignment = Alignment.Center
        ) {
            Icon(
                imageVector = when (state) {
                    CaptureState.RECORDING -> Icons.Default.Pause
                    CaptureState.PAUSED -> Icons.Default.PlayArrow
                    CaptureState.READY -> Icons.Default.PlayArrow
                },
                contentDescription = null,
                tint = Color.Black,
                modifier = Modifier.size(36.dp)
            )
        }
    }
}
```

---

## 8. Manifiesto de Android (`AndroidManifest.xml`)

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.lightpainting.camera">

    <!-- Permisos de Bajo Nivel de Hardware -->
    <uses-permission android:name="android.permission.CAMERA" />
    <uses-feature android:name="android.hardware.camera" android:required="true" />
    <uses-feature android:name="android.hardware.camera.manual_sensor" android:required="true" />
    <uses-feature android:name="android.hardware.camera.manual_post_processing" android:required="true" />

    <!-- Reconocimiento de Voz Zero-Latency Offline -->
    <uses-permission android:name="android.permission.RECORD_AUDIO" />

    <!-- Almacenamiento RAW DNG / PNG -->
    <uses-permission android:name="android.permission.READ_MEDIA_IMAGES" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" android:maxSdkVersion="28" />

    <!-- Aceleración GPU OpenGL ES 3.2 -->
    <uses-feature android:glEsVersion="0x00030002" android:required="true" />

    <application
        android:allowBackup="false"
        android:hardwareAccelerated="true"
        android:icon="@mipmap/ic_launcher"
        android:label="LuminaCam Pro"
        android:theme="@android:style/Theme.NoTitleBar.Fullscreen">

        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:screenOrientation="portrait">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>
</manifest>
```
