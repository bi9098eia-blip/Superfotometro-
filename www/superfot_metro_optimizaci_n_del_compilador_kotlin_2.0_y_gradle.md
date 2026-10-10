# SuperFotómetro — Optimización Extrema del Compilador Kotlin 2.0, R8 y Gradle Daemon

Esta configuración optimiza al máximo el **compilador de Kotlin (K2 compiler)**, el **plugin de Jetpack Compose Compiler**, el motor de optimización de bytecode **R8/ProGuard** y los flags del **Gradle Daemon** para obtener:
1. **Compilación incremental ultrarrápida** (hasta un 65% más rápida gracias al compilador K2 nativo de Kotlin 2.0+).
2. **Optimización en tiempo de compilación para Jetpack Compose** (Strong Skipping Mode & Stability Configuration para evitar recomposiciones innecesarias).
3. **Optimización de memoria y concurrencia** con JVM 17/21 y procesamiento en paralelo de tareas de dexing.
4. **Reducción de tamaño del APK (R8 Full Mode)** y optimización de llamadas nativas NDK / OpenGL ES 3.2.

---

## ⚙️ 1. `gradle.properties` (Aceleración de Compilación & Memoria JVM)

Este archivo configura la máquina virtual de compilación, activa el compilador K2 y permite la ejecución paralela sin bloqueos:

```properties
# ==============================================================================
# GRADLE DAEMON & OPTIMIZACIÓN JVM COMPILADOR
# ==============================================================================
# Asignación óptima de RAM para el compilador Kotlin y tareas de compilación (ajustable de 2GB a 4GB)
org.gradle.jvmargs=-Xmx4096m -XX:+UseParallelGC -XX:MaxMetaspaceSize=1024m -Dfile.encoding=UTF-8

# Compilación concurrente de módulos y deshabilitación de configuración no reactiva
org.gradle.parallel=true
org.gradle.caching=true
org.gradle.vfs.watch=true
org.gradle.configuration-cache=true

# AndroidX & Jetifier (Jetifier desactivado para eliminar sobrecoste en Kotlin 2.0)
android.useAndroidX=true
android.enableJetifier=false

# Optimización R8: Modo completo para eliminar código muerto y optimizar llamadas inline
android.enableR8.fullMode=true

# ==============================================================================
# OPTIMIZACIONES DEL COMPILADOR KOTLIN 2.0 (K2)
# ==============================================================================
# Fuerza el nuevo motor frontend de compilación ultrarrápido K2 de Kotlin 2.0
kotlin.experimental.tryK2=true
kotlin.incremental=true
kotlin.incremental.useClasspathSnapshot=true
kotlin.daemon.jvmargs=-Xmx2048m -XX:+UseParallelGC

# Reportes de estabilidad de Compose Compiler para verificar recomposiciones estables
kotlin.composeCompiler.reportsDestination=build/compose_reports
kotlin.composeCompiler.metricsDestination=build/compose_metrics
```

---

## ⚡ 2. `app/build.gradle.kts` (Compilador Kotlin 2.0 & Jetpack Compose Compiler)

Configuración directa con flags de optimización agresivos para el compilador de Kotlin (`freeCompilerArgs`) y compatibilidad total con C++ / OpenGL ES 3.2:

```kotlin
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
    alias(libs.plugins.kotlin.compose) // Plugin oficial de Compose Compiler de Kotlin 2.0
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

        // Empaquetado optimizado de binarios nativos para procesadores móviles reales (64-bit prioritario)
        ndk {
            abiFilters += listOf("arm64-v8a", "armeabi-v7a", "x86_64")
        }
    }

    buildTypes {
        release {
            isMinifyEnabled = true
            isShrinkResources = true
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
            signingConfig = signingConfigs.getByName("debug") // Facilita la instalación offline sin configurar firmas manuales
        }
        debug {
            isMinifyEnabled = false
            applicationIdSuffix = ".debug"
            isDebuggable = true
        }
    }

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }

    kotlin {
        jvmToolchain(17)
    }

    // ==========================================================================
    // PARÁMETROS DEL COMPILADOR KOTLIN 2.0 (K2 COMPILER ARGS)
    // ==========================================================================
    kotlinOptions {
        jvmTarget = "17"
        freeCompilerArgs = listOf(
            "-Xbackend-threads=4",              // Compilación paralela en 4 hilos de CPU
            "-opt-in=kotlin.RequiresOptIn",      // Habilita APIs experimentales sin warnings
            "-opt-in=androidx.compose.material3.ExperimentalMaterial3Api",
            "-Xjvm-default=all",                 // Emite métodos default en bytecode de interfaces Java 8+
            "-Xcontext-receivers",               // Habilita context receivers de Kotlin
            "-Xskip-prerelease-check"
        )
    }

    buildFeatures {
        compose = true
    }

    packaging {
        resources {
            excludes += listOf(
                "/META-INF/{AL2.0,LGPL2.1}",
                "/META-INF/LICENSE.md",
                "/META-INF/LICENSE-notice.md"
            )
        }
    }
}

// Configuración avanzada del compilador de Jetpack Compose en Kotlin 2.0
composeCompiler {
    enableStrongSkippingMode = true // Salta recomposiciones si los parámetros no han cambiado (Strong Skipping)
    includeSourceInformation = false // Reduce el tamaño del binario eliminando metadata innecesaria en release
}

dependencies {
    implementation(platform(libs.androidx.compose.bom))
    implementation(libs.androidx.core.ktx)
    implementation(libs.androidx.lifecycle.runtime.ktx)
    implementation(libs.androidx.lifecycle.viewmodel.compose)
    implementation(libs.androidx.activity.compose)
    implementation(libs.androidx.ui)
    implementation(libs.androidx.ui.graphics)
    implementation(libs.androidx.material3)
    implementation(libs.androidx.material.icons.extended)

    // Camera2 y Renderizado Nativo OpenGL
    implementation(libs.androidx.camera.core)
    implementation(libs.androidx.camera.camera2)
    implementation(libs.androidx.camera.lifecycle)
    implementation(libs.androidx.camera.view)
    implementation(libs.kotlinx.coroutines.android)
}
```

---

## 🛡️ 3. `app/proguard-rules.pro` (Optimización R8 & Preservación de Shaders OpenGL)

Evita que el optimizador de código elimine funciones de la GPU OpenGL ES 3.2 o clases del Camera2 Manager al minificar:

```proguard
# ==============================================================================
# REGLAS PROGUARD / R8 COMPILER PARA SUPERFOTÓMETRO
# ==============================================================================

# Preservar modelos de datos y enums de estado
-keep class com.superfotometro.camera.domain.** { *; }

# Preservar llamadas nativas y reflection del hardware Camera2
-keepclassmembers class * extends android.hardware.camera2.CameraDevice$StateCallback { *; }
-keepclassmembers class * extends android.hardware.camera2.CameraCaptureSession$StateCallback { *; }

# Optimización Compose Compiler: Preservar clases estables
-keepclassmembers class * {
    @androidx.compose.runtime.Composable *;
}

# Optimización agresiva de llamadas a logs en versión de producción
-assumenosideeffects class android.util.Log {
    public static boolean isLoggable(java.lang.String, int);
    public static int v(...);
    public static int d(...);
}
```

---

## 🚀 4. Comandos de Compilación Ultrarrápida

Para compilar con el máximo rendimiento desde la terminal o consola:

```bash
# Compilar APK con K2 a máxima velocidad y caché activa:
./gradlew assembleDebug --parallel --build-cache --configure-on-demand

# Generar APK optimizado y minificado con R8:
./gradlew assembleRelease --parallel --build-cache
```
