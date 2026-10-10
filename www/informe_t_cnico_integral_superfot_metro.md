# SuperFotómetro — Informe Técnico Integral de Características y Especificaciones

**Nombre del Proyecto:** SuperFotómetro (anteriormente LuminaCam Pro)  
**Fecha de compilación:** Actual  
**Plataforma de destino:** Android Nativo (Kotlin 2.0+, Jetpack Compose, Camera2 API, OpenGL ES 3.2 GPU Shaders) & Prototipo Python 3  
**Arquitectura visual:** Modo AMOLED Pitch Black (`#000000`), Luminous Teal (`#00FFCC`), Magenta Neón (`#FF007F`), Amber Pause (`#FFB800`)

---

## 1. Visor de Cámara y Dinámica de Exposición en Vivo

1. **Visor Pro a Pantalla Completa (Live View)**:
   - Integración con sensor de cámara nativa con cálculo de fotones y acumulación GPU continua a 60 FPS.
   - Histograma RGB dinámico integrado para monitorizar niveles de exposición sin obstruir el encuadre.
   - Nivel giroscópico y telemetría de batería en tiempo real.

2. **Selector de Lentes Físicos del Móvil**:
   - Conmutación directa y en caliente entre todas las cámaras traseras disponibles en el hardware mediante `CameraCharacteristics.LENS_INFO_AVAILABLE_FOCAL_LENGTHS`:
     - **0.6x (14mm)**: Ultra Gran Angular.
     - **1x (24mm)**: Lente Principal de alta resolución.
     - **3x (75mm)**: Teleobjetivo óptico.
     - **Macro**: Para fotografía de detalle lumínico cercano.

3. **Controles Manuales del Sensor (Estilo DSLR / Modo M)**:
   - **Sensibilidad ISO**: Ajuste continuo y por pasos fijos desde **ISO 50** hasta **ISO 6400**.
   - **Velocidad de Obturación**: Desde velocidades ultrarrápidas (**1/8000s**) hasta largas exposiciones (**30s**) y modo **BULB $\infty$** continuo.
   - **Enfoque Manual & Focus Peaking**: Anillo milimétrico de distancia focal con realce en neón sobre los bordes enfocados.

---

## 2. Motor de Acumulación y Shaders GPU (OpenGL ES 3.2)

1. **Modos de Fusión Dinámicos (Conmutables en Caliente)**:
   - **Pintar con Luz (Lighten / Max Blend)**: $P_{out} = \max(P_{prev}, P_{curr})$. Acumula únicamente las fuentes luminosas que superen los umbrales configurados.
   - **Pintar con Sombra (Darken / Min Blend)**: $P_{out} = \min(P_{prev}, P_{curr})$. Permite crear siluetas oscuras recortadas sobre fondos o trazos iluminados.
   - **Modo Estela Dinámica (Decay / Phosphor Trails)**: $P_{out} = \max(P_{prev} \times \alpha, P_{curr})$. Genera un desvanecimiento exponencial gradual en las estelas lumínicas.

2. **Muestreo por Fotogramas y Modo Estroboscópico**:
   - Presets de cadencia de muestreo: **1:1 (60 fps)**, **1:2 (30 fps)**, **1:4 (15 fps punteado)** y **1:10 (6 fps estroboscópico)**.
   - Intervalo manual continuo de 1 a 30 frames para simular destellos de pulsos lumínicos periódicos en la GPU.

3. **Conmutación de Modos en Caliente**:
   - Los modos de renderizado se pueden alternar tanto en plena grabación como durante el estado de pausa sin reiniciar ni perder el lienzo acumulado.

---

## 3. Estado en Pausa y Superposición en Vivo (High-Contrast Overlay)

1. **Superposición de la Imagen Capturada sobre el Sensor en Vivo**:
   - Al entrar en pausa, el visor mantiene la transmisión de la cámara en vivo mientras proyecta el lienzo acumulado congelado.
   - Realce de contraste gamma ($1.25\times$) para distinguir perfectamente la posición actual del sujeto frente a las líneas previamente pintadas.

2. **Deslizador de Opacidad / Transparencia de Superposición**:
   - Permite regular en tiempo real la mezcla alfa del lienzo en pausa sobre el sensor en directo:
     $$\text{fragColor} = \text{mix}(\text{liveColor.rgb}, \text{boostedPrev}, \text{uPauseOverlayOpacity})$$
   - Desde **0%** (visualización exclusiva de la cámara) hasta **100%** (visualización opaca del lienzo acumulado).

3. **Historial de Pausas y Checkpoints con Deshacer (Undo)**:
   - Cada pausa genera un punto de control (checkpoint) en los buffers de la GPU.
   - Botón **"Deshacer Último Trazo"** para retroceder en la memoria de fotogramas y eliminar trazos erróneos sin arruinar la sesión completa.

---

## 4. Botón Disparador Flotante y Ergonomía del Interfaz

1. **Botón Play/Pausa Móvil y Reubicable (`Draggable Shutter`)**:
   - El disparador central se puede arrastrar libremente por cualquier cuadrante de la pantalla (`pointerInput / detectDragGestures`) para adaptarlo a la posición de la mano o al trípode.
   - Presets de anclaje rápido: **Libre**, **Abajo Izquierda** o **Abajo Derecha**.
   - Bloqueo y desbloqueo de posición mediante indicador de candado.

2. **Paneles Ocultables Mediante Deslizamiento Lateral (`Swipe to Dismiss`)**:
   - Las tarjetas de control inferior (Placa base, Umbrales, Muestreo de frames, Modo Manual, Máscaras) se pueden replegar y ocultar deslizando horizontalmente hacia los lados.
   - Botón de acceso directo a **Visor Inmersivo / Fullscreen** para dejar la pantalla 100% limpia de interfaces y evitar contaminación lumínica.

---

## 5. Gestión de Capa de Fondo (Placa Base)

1. **Importación Flexible de Imagen de Fondo**:
   - Opción para cargar fotografías guía previas (URI / Bitmap de astrofotografía, arquitectura nocturna o bocetos).
   - Interruptor de superposición (`Superponer On/Off`).

2. **Inicio Sin Imagen (Lienzo Limpio)**:
   - Botón dedicado **"Lienzo Limpio"** que permite iniciar sesiones directamente desde negro absoluto sin requerir la carga de ningún archivo base.

---

## 6. Deslizadores Duales (Rangos Mínimos y Máximos)

Todos los deslizadores cuentan con **dos manipuladores (mínimo y máximo simultáneos)** para aislar la captura:

1. **Rango de Luminancia (Lux)**:
   - Filtro de corte inferior para descartar sombras/ruido y corte superior para evitar clipping y zonas sobreexpuestas (ej. `Min 15% — Max 88%`).

2. **Rango de Velocidad del Trazo**:
   - Filtra el movimiento en píxeles por segundo (ej. `5 px/s — 120 px/s`), ignorando fuentes de luz estáticas no deseadas o movimientos fuera de ritmo.

3. **Rango de Tolerancia Cromática ($\Delta E$)**:
   - Ajusta el ancho de banda del espectro de color que debe ser capturado o rechazado.

---

## 7. Muestreo de Color a Ignorar (Cuentagotas & Chroma Key)

1. **Cuentagotas con Zoom Óptico 8x**:
   - Retícula ampliada que permite muestrear píxel a píxel directamente en el visor cualquier color parásito (ej. farolas de vapor de sodio amarillo `#FF9E1B` o luces de emergencia).

2. **Filtro de Exclusión o Pase Exclusivo**:
   - **Exclusión**: Ignora por completo el color muestreado durante la grabación.
   - **Modo Exclusivo**: Invierte el filtro para captar *únicamente* la fuente de luz del color seleccionado.

---

## 8. Herramientas de Máscaras e Inversión de Selección

1. **Máscara de Luz (+)**:
   - Pincel activo para delimitar zonas específicas del encuadre donde se permite la acumulación de fotones.

2. **Escudo Negativo (-)**:
   - Capa de protección contra deslumbramientos directos a la lente, protegiendo áreas sensibles para que mantengan su oscuridad original.

3. **Inversión de Selección (`Invertir Máscara`)**:
   - Botón instantáneo para invertir los canales de la máscara (lo protegido pasa a ser expuesto y viceversa).
   - Ajustes de **calibre del pincel** (8 px a 120 px) y **dureza del trazo**.

---

## 9. Muestreo de Voz Personalizado y Control Manos Libres (Zero-Latency)

1. **Entrenamiento de Fonemas y Calibración Acústica Propia**:
   - Espectro de frecuencias de voz en tiempo real ($20\,\text{Hz} - 16\,\text{kHz}$) y calibración de ruido de fondo ($-48\,\text{dB}$).
   - Registro de 3 muestras vocales por comando para adaptarse al timbre, tono y volumen personal del usuario.

2. **Asignación Libre de Palabras por Función**:
   - Permite personalizar la palabra exacta para cada acción crítica:
     - **Iniciar Trazo** (ej. *"¡FUEGO!"* o *"Play"*).
     - **Pausar / Congelar** (ej. *"¡ALTO!"* o *"Pausa"*).
     - **Deshacer Trazo** (ej. *"¡BORRA!"* o *"Atrás"*).
     - **Invertir Máscara** (ej. *"¡ESCUDO!"*).
     - **Conmutar Lente** (ej. *"¡LENTE!"*).
     - **Guardar Captura** (ej. *"¡FOTO!"*).

3. **Ajustes Individuales de Detección**:
   - Deslizador de tolerancia acústica por comando (80% a 95%).
   - Botón de prueba auditiva y simulador en vivo con tiempo de respuesta de **18 ms** sin conexión a Internet.

---

## 10. Intervalómetro y Automatización de Disparos

1. **Temporizador de Retardo Previo**: Cuenta atrás configurable para posicionarse antes del inicio.
2. **Duración de Exposición Activa**: Tiempo cronometrado de acumulación continua.
3. **Pausas Programadas entre Ciclos**: Intervalos automáticos para cambiar de linterna o herramienta lumínica.
4. **Ciclos Repetitivos y Avisos Sonoros**: Secuencias automatizadas de 1 a $\infty$ con beeps acústicos de sincronización.

---

## 11. Formatos de Salida y Exportación Profesional

1. **Soporte DNG RAW de 14-bit**: Captura de rango dinámico completo sin compresión de cámara.
2. **PNG sin Pérdida**: Renderizado instantáneo de la composición matricial para revisión inmediata.
