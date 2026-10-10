# SuperFotómetro — Paquete Maestro Completo para Descarga & Compilación Offline

Este paquete contiene la estructura integral de carpetas, código fuente, assets y la carpeta de imágenes de previsualización para **SuperFotómetro**.

---

## 📂 1. Estructura Completa del Proyecto (`SuperFotometro.zip`)

```text
SuperFotometro/
├── settings.gradle.kts
├── build.gradle.kts
├── gradle.properties
├── README.md
├── DESIGN.md
├── superfotometro_pydroid.py               <-- Script ejecutable en móvil Android (Pydroid 3)
│
├── imagenes_preview_jpg/                   <-- Carpeta con imágenes JPG de prueba (eliminar cuando desees)
│   ├── 01_visor_limpio_boceto.jpg          <-- Visor limpio con deslizador mariposa y disparador
│   ├── 02_visor_pestanas_laterales.jpg     <-- Visor despejado con pestañas de colores en bordes
│   ├── 03_ia_sujeto_todo_menos_yo.jpg      <-- Panel MediaPipe IA con Inpainting en tiempo real
│   ├── 04_suite_10_modulos_completa.jpg    <-- Vista de ingeniería con los 10 módulos desplegados
│   ├── 05_boceto_original_camara.jpg       <-- Boceto a mano alzada de la cámara retro
│   ├── 06_boceto_pantalla_limpia.jpg       <-- Boceto a mano alzada de la distribución de pantalla
│   └── 07_muestra_caligrafia_pluma.jpg     <-- Muestra de escritura manual para tipografía
│
└── app/
    ├── build.gradle.kts
    ├── proguard-rules.pro
    └── src/
        └── main/
            ├── AndroidManifest.xml
            ├── res/
            │   ├── raw/
            │   │   ├── accumulate_frag.glsl        <-- Fragment shader OpenGL ES 3.2 (Luz, Sombra, Estela)
            │   │   └── inpainting_blend.glsl       <-- Shader de segmentación y supresión de sujeto
            │   ├── font/
            │   │   └── pen_handwritten.xml         <-- Tipografía manuscrita a pluma
            │   ├── values/
            │   │   ├── strings.xml
            │   │   ├── colors.xml
            │   │   └── themes.xml
            │   └── drawable/
            │       ├── ic_shutter_sketch.xml       <-- Icono disparador vectorizado fiel al boceto
            │       └── ic_butterfly_slider.xml     <-- Cursor de mariposa (>|<) vectorizado
            └── java/
                └── com/
                    └── superfotometro/
                        └── camera/
                            ├── MainActivity.kt
                            ├── core/
                            │   ├── SuperFotometroPipeline.kt    <-- CameraX + OpenCV HSV + MediaPipe
                            │   ├── SuperFotometroHardwareManager.kt
                            │   └── VoiceTriggerService.kt
                            ├── domain/
                            │   └── SuperFotometroModels.kt
                            └── ui/
                                ├── theme/
                                │   ├── Color.kt
                                │   ├── Theme.kt
                                │   └── Type.kt
                                └── screens/
                                    ├── ViewfinderDespejadoScreen.kt
                                    ├── IaSujetoInpaintingModal.kt
                                    └── ModulesDocksOverlay.kt
```

---

## 📸 2. Contenido de la Carpeta de Imágenes (`imagenes_preview_jpg/`)

Esta carpeta se incluye con las imágenes renderizadas y bocetos originales para verificar la estética visual:
1. **`01_visor_limpio_boceto.jpg`**: Captura del visor 100% limpio con el disparador circular grabado y la barra con cursor mariposa.
2. **`02_visor_pestanas_laterales.jpg`**: Captura del visor despejado con las 9 pestañas de colores replegadas en los bordes.
3. **`03_ia_sujeto_todo_menos_yo.jpg`**: Captura del panel de IA Sujeto abierto mostrando el cuerpo del fotógrafo enmascarado y sustituido por el fondo.
4. **`04_suite_10_modulos_completa.jpg`**: Captura de la ingeniería completa con los 10 módulos de captura e histogramas.
5. **`05_boceto_original_camara.jpg`**: Archivo de referencia del dibujo a bolígrafo de la cámara rangefinder retro.
6. **`06_boceto_pantalla_limpia.jpg`**: Archivo de referencia del dibujo a pluma de la distribución perimetral.
7. **`07_muestra_caligrafia_pluma.jpg`**: Archivo de referencia de la caligrafía manual técnica.

*(Nota: Puedes eliminar esta carpeta en cualquier momento sin afectar al código de la aplicación).*

---

## 💻 3. Archivos Clave del Núcleo

### `app/src/main/res/raw/inpainting_blend.glsl`
```glsl
#version 320 es
precision highp float;

in vec2 vTexCoord;
out vec4 fragColor;

uniform sampler2D uLiveSensor;         // Cámara en vivo (CameraX)
uniform sampler2D uAccumulatedCanvas;  // Lienzo acumulado en VRAM FBO
uniform sampler2D uCleanBackground;    // Fondo limpio para Inpainting
uniform sampler2D uAiMaskTexture;      // Máscara MediaPipe (Silueta del sujeto)

uniform int uIdentifyMode;             // 0: Normal, 1: Solo Yo, 2: Todo Menos Yo (Inpainting)
uniform int uBlendMode;                // 0: Luz, 1: Sombra, 2: Estela
uniform float uDecay;                  // Factor de desvanecimiento
uniform float uPauseAlpha;             // Deslizador de mariposa (0.0 a 1.0)
uniform bool uIsPaused;

void main() {
    vec4 live = texture(uLiveSensor, vTexCoord);
    vec4 prev = texture(uAccumulatedCanvas, vTexCoord);
    vec4 bg = texture(uCleanBackground, vTexCoord);
    float personMask = texture(uAiMaskTexture, vTexCoord).r;

    // 1. MODO EN PAUSA (SUPERPOSICIÓN CON CURSOR DE MARIPOSA)
    if (uIsPaused) {
        vec3 boosted = pow(prev.rgb, vec3(1.0 / 1.25));
        vec3 blended = mix(live.rgb, boosted, uPauseAlpha);
        fragColor = vec4(blended, 1.0);
        return;
    }

    // 2. MODO 9.4: "TODO MENOS YO" (Inpainting del operador)
    if (uIdentifyMode == 2) {
        if (personMask > 0.5) {
            // El cuerpo del fotógrafo se reemplaza por el fotograma de fondo limpio
            live = bg;
        }
    } else if (uIdentifyMode == 1) { // MODO 9.2: "SOLO YO"
        if (personMask < 0.5) {
            live = vec4(0.0, 0.0, 0.0, 1.0);
        }
    }

    // 3. ACUMULACIÓN DE LUZ
    vec4 res = prev;
    if (uBlendMode == 0) {
        res = max(prev, live);
    } else if (uBlendMode == 1) {
        res = min(prev, live);
    } else if (uBlendMode == 2) {
        res = max(prev * uDecay, live);
    }

    fragColor = res;
}
```

---

## 🚀 4. Instrucciones para Descarga y Uso

1. Puedes descargar el archivo ZIP directamente o explorar cada archivo individual.
2. Si utilizas un ordenador con Android Studio: abre la carpeta descomprimida y ejecuta `Build > Build APK(s)`.
3. Si estás desde tu móvil Android: abre `superfotometro_pydroid.py` en la app **Pydroid 3** para ejecutarlo al instante con sensor táctil simulado.
