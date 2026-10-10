# SuperFotómetro — Paquete Oficial de Fuentes y Estilos Tipográficos para Android (Jetpack Compose & XML)

Este paquete contiene todos los archivos, configuraciones y recursos listos para integrar la **tipografía caligráfica manuscrita a pluma/bolígrafo** y los **estilos tipográficos oficiales** en el proyecto Android de **SuperFotómetro** (Kotlin 2.0 / Jetpack Compose / Material 3).

---

## 📂 1. Estructura de Carpetas Tipográficas en Android

```text
SuperFotometro/
└── app/
    └── src/
        └── main/
            ├── res/
            │   ├── font/
            │   │   ├── pen_handwritten.xml             <-- Definición de fuente descargable o local
            │   │   └── architects_daughter.ttf         <-- Archivo binario TTF alternativo local
            │   └── values/
            │       ├── font_certs.xml                  <-- Certificados de Google Fonts
            │       └── type_styles.xml                 <-- Estilos XML clásicos de respaldo
            └── java/
                └── com/
                    └── superfotometro/
                        └── camera/
                            └── ui/
                                └── theme/
                                    ├── Type.kt         <-- Typography oficial Jetpack Compose
                                    └── Color.kt        <-- Tokens de color de tinta a pluma
```

---

## 🖋️ 2. Fuentes Seleccionadas y Mapeo con la Caligrafía Manual

Basado en la muestra de escritura manual a bolígrafo (trazos rápidos en mayúsculas, números sueltos y estilo de libreta técnica), la suite utiliza:

1. **`Architects Daughter` / `Kalam`**: Caligrafía de rotulación manual para títulos, etiquetas de telemetría y numerales del visor.
2. **`JetBrains Mono` / `Space Grotesk`**: Tipografía monoespaciada para telemetría binaria de alta precisión, códigos de tiempo y exposición.

---

## 📄 3. Configuración de Fuentes Descargables (`res/font/pen_handwritten.xml`)

Permite a Android descargar la fuente oficial directamente de Google Play Services o compilarla localmente:

```xml
<?xml version="1.0" encoding="utf-8"?>
<font-family xmlns:app="http://schemas.android.com/apk/res-auto"
    app:fontProviderAuthority="com.google.android.gms.fonts"
    app:fontProviderPackage="com.google.android.gms"
    app:fontProviderQuery="name=Architects+Daughter&amp;weight=400"
    app:fontProviderCerts="@array/com_google_android_gms_fonts_certs">
</font-family>
```

---

## 📄 4. Certificados de Google Fonts (`res/values/font_certs.xml`)

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <array name="com_google_android_gms_fonts_certs">
        <item>@array/com_google_android_gms_fonts_certs_dev</item>
        <item>@array/com_google_android_gms_fonts_certs_prod</item>
    </array>
    <string-array name="com_google_android_gms_fonts_certs_dev">
        <item>
            MIIEqDCCA5CgAwIBAgIJANWFuGx90071MA0GCSqGSIb3DQEBBAUAMIGUMQswCQYDVQQGEwJVUzET
            MBEGA1UECBMKQ2FsaWZvcm5pYTEWMBQGA1UEBxMNTW91bnRhaW4gVmlldzEQMA4GA1UEChMHR29v
            Z2xlIDEUMBIGA1UECxMLQW5kcm9pZFRlYW0xEzARBgNVBAMTCkFuZHJvaWRUZWFtMSEwHwYJKoZI
            hvcNAQkBFhJhbmRyb2lkQGFuZHJvaWQuY29tMB4XDDA4MDIyOTIxMjQzNloXDTE4MDIyNzIxMjQz
            ...
        </item>
    </string-array>
    <string-array name="com_google_android_gms_fonts_certs_prod">
        <item>
            MIIEqDCCA5CgAwIBAgIJANWFuGx90071MA0GCSqGSIb3DQEBBAUAMIGUMQswCQYDVQQGEwJVUzET
            MBEGA1UECBMKQ2FsaWZvcm5pYTEWMBQGA1UEBxMNTW91bnRhaW4gVmlldzEQMA4GA1UEChMHR29v
            Z2xlIDEUMBIGA1UECxMLQW5kcm9pZFRlYW0xEzARBgNVBAMTCkFuZHJvaWRUZWFtMSEwHwYJKoZI
            hvcNAQkBFhJhbmRyb2lkQGFuZHJvaWQuY29tMB4XDDA4MDIyOTIxMjQzNloXDTE4MDIyNzIxMjQz
            ...
        </item>
    </string-array>
</resources>
```

---

## 🎨 5. Tokens de Tipografía en Jetpack Compose (`ui/theme/Type.kt`)

Ubicación: `app/src/main/java/com/superfotometro/camera/ui/theme/Type.kt`

```kotlin
package com.superfotometro.camera.ui.theme

import androidx.compose.material3.Typography
import androidx.compose.ui.text.TextStyle
import androidx.compose.ui.text.font.Font
import androidx.compose.ui.text.font.FontFamily
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.text.googlefonts.GoogleFont
import androidx.compose.ui.text.googlefonts.Font as GoogleComposeFont
import androidx.compose.ui.unit.sp
import com.superfotometro.camera.R

// Proveedor de Google Fonts oficial para Compose
val fontProvider = GoogleFont.Provider(
    providerAuthority = "com.google.android.gms.fonts",
    providerPackage = "com.google.android.gms",
    certificates = R.array.com_google_android_gms_fonts_certs
)

// Familia 1: Caligrafía Manuscrita a Pluma / Cuaderno de Diseño
val PenHandwrittenFontName = GoogleFont("Architects Daughter")
val PenHandwrittenFontFamily = FontFamily(
    GoogleComposeFont(googleFont = PenHandwrittenFontName, fontProvider = fontProvider)
)

// Familia 2: Fuente Manuscrita Secundaria / Boli Cuaderno Nocturno
val KalamFontName = GoogleFont("Kalam")
val KalamFontFamily = FontFamily(
    GoogleComposeFont(googleFont = KalamFontName, fontProvider = fontProvider, weight = FontWeight.Bold)
)

// Familia 3: Telemetría Técnica Monoespaciada
val JetBrainsMonoFontName = GoogleFont("JetBrains Mono")
val MonoTechFontFamily = FontFamily(
    GoogleComposeFont(googleFont = JetBrainsMonoFontName, fontProvider = fontProvider)
)

// Tipografía Oficial SuperFotómetro para Material 3
val SuperFotometroTypography = Typography(
    // Títulos de visor ("SUPERFOTÓMETRO", "CALIGRAFÍA A MANO ALZADA")
    displayLarge = TextStyle(
        fontFamily = PenHandwrittenFontFamily,
        fontWeight = FontWeight.Bold,
        fontSize = 24.sp,
        letterSpacing = 1.2.sp
    ),
    displayMedium = TextStyle(
        fontFamily = PenHandwrittenFontFamily,
        fontWeight = FontWeight.Bold,
        fontSize = 18.sp,
        letterSpacing = 1.0.sp
    ),
    // Cabeceras de paneles ("ENFOQUE: MÁSCARA PEAKING MANUAL", "CONTROL DE SUPERPOSICIÓN")
    titleLarge = TextStyle(
        fontFamily = PenHandwrittenFontFamily,
        fontWeight = FontWeight.Bold,
        fontSize = 16.sp,
        letterSpacing = 0.8.sp
    ),
    titleMedium = TextStyle(
        fontFamily = PenHandwrittenFontFamily,
        fontWeight = FontWeight.Bold,
        fontSize = 14.sp
    ),
    // Numerales y telemetría de enfoque ("0.6x", "1.0x 24mm PRIME", "88.5%")
    labelLarge = TextStyle(
        fontFamily = PenHandwrittenFontFamily,
        fontWeight = FontWeight.Bold,
        fontSize = 13.sp,
        letterSpacing = 0.5.sp
    ),
    labelMedium = TextStyle(
        fontFamily = MonoTechFontFamily,
        fontWeight = FontWeight.Medium,
        fontSize = 11.sp,
        letterSpacing = 0.6.sp
    ),
    labelSmall = TextStyle(
        fontFamily = MonoTechFontFamily,
        fontWeight = FontWeight.Bold,
        fontSize = 9.sp,
        letterSpacing = 0.4.sp
    ),
    // Cuerpo explicativo y descripciones de algoritmos
    bodyLarge = TextStyle(
        fontFamily = PenHandwrittenFontFamily,
        fontWeight = FontWeight.Normal,
        fontSize = 13.sp,
        lineHeight = 18.sp
    ),
    bodyMedium = TextStyle(
        fontFamily = PenHandwrittenFontFamily,
        fontWeight = FontWeight.Normal,
        fontSize = 11.sp,
        lineHeight = 15.sp
    )
)
```

---

## 🎨 6. Tokens de Color para Tinta y Cuaderno (`ui/theme/Color.kt`)

Ubicación: `app/src/main/java/com/superfotometro/camera/ui/theme/Color.kt`

```kotlin
package com.superfotometro.camera.ui.theme

import androidx.compose.ui.graphics.Color

// Paleta Oficial "Cuaderno Técnico & Tinta a Pluma"
val InkBlueprintDark = Color(0xFF04080D)        // Fondo de papel oscuro de cuaderno
val InkPaperSurface = Color(0xFF090E14)         // Superficie de contenedores de tinta
val InkPaperBorder = Color(0x3300FFCC)          // Borde con doble pasada de pluma

// Tintas Vivas de Pluma Estilográfica
val NeonTealInk = Color(0xFF00FFCC)            // Tinta primaria cian luminoso
val NeonCyanInk = Color(0xFF00E5FF)            // Tinta de resaltado secundario
val NeonMagentaInk = Color(0xFFFF007F)         // Tinta magenta de foco peaking
val AmberPauseInk = Color(0xFFFFB800)          // Tinta ámbar de alerta y pausa
val RedRecInk = Color(0xFFFF3B30)              // Tinta roja de exposición activa
val WhiteInk = Color(0xFFFFFFFF)               // Trazos sueltos de luz blanca
```

---

## 🛠️ 7. Dependencias Requeridas en `app/build.gradle.kts`

Asegúrate de tener esta dependencia incluida para el soporte nativo de Google Fonts en Compose:

```kotlin
dependencies {
    // Soporte oficial de Google Fonts en Compose sin empaquetar archivos pesados
    implementation("androidx.compose.ui:ui-text-google-fonts:1.7.5")
    
    // Core Compose Material 3
    implementation("androidx.compose.material3:material3:1.3.1")
}
```

---

## 💡 8. Ejemplo de Uso en Componentes Compose

```kotlin
// Aplicar la tipografía manuscrita a un texto o etiqueta
Text(
    text = "ENFOQUE MANUAL PEAKING // CALIBRE 35mm",
    style = SuperFotometroTypography.titleMedium,
    color = NeonTealInk
)

// Numerales con estilo caligráfico de cuaderno
Text(
    text = "SENSIBILIDAD 88.5 %",
    style = SuperFotometroTypography.labelLarge,
    color = WhiteInk
)
```
