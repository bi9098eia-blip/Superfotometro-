---
name: LuminaCam Pro
colors:
  surface: '#131313'
  surface-dim: '#131313'
  surface-bright: '#393939'
  surface-container-lowest: '#0e0e0e'
  surface-container-low: '#1b1b1b'
  surface-container: '#1f1f1f'
  surface-container-high: '#2a2a2a'
  surface-container-highest: '#353535'
  on-surface: '#e2e2e2'
  on-surface-variant: '#b9cbc2'
  inverse-surface: '#e2e2e2'
  inverse-on-surface: '#303030'
  outline: '#83958d'
  outline-variant: '#3a4a44'
  surface-tint: '#00e0b3'
  primary: '#fdfffc'
  on-primary: '#00382b'
  primary-container: '#00ffcc'
  on-primary-container: '#00725a'
  inverse-primary: '#006b54'
  secondary: '#ffb1c4'
  on-secondary: '#65002e'
  secondary-container: '#ff4a8d'
  on-secondary-container: '#590028'
  tertiary: '#fffeff'
  on-tertiary: '#412d00'
  tertiary-container: '#ffdda5'
  on-tertiary-container: '#835d00'
  error: '#ffb4ab'
  on-error: '#690005'
  error-container: '#93000a'
  on-error-container: '#ffdad6'
  primary-fixed: '#24ffcd'
  primary-fixed-dim: '#00e0b3'
  on-primary-fixed: '#002118'
  on-primary-fixed-variant: '#00513f'
  secondary-fixed: '#ffd9e1'
  secondary-fixed-dim: '#ffb1c4'
  on-secondary-fixed: '#3f001a'
  on-secondary-fixed-variant: '#8f0044'
  tertiary-fixed: '#ffdea8'
  tertiary-fixed-dim: '#ffba20'
  on-tertiary-fixed: '#271900'
  on-tertiary-fixed-variant: '#5e4200'
  background: '#131313'
  on-background: '#e2e2e2'
  surface-variant: '#353535'
typography:
  headline-lg:
    fontFamily: Space Grotesk
    fontSize: 32px
    fontWeight: '700'
    lineHeight: 38px
    letterSpacing: -0.02em
  headline-lg-mobile:
    fontFamily: Space Grotesk
    fontSize: 26px
    fontWeight: '700'
    lineHeight: 32px
    letterSpacing: -0.01em
  headline-md:
    fontFamily: Space Grotesk
    fontSize: 22px
    fontWeight: '600'
    lineHeight: 28px
    letterSpacing: -0.01em
  headline-sm:
    fontFamily: Space Grotesk
    fontSize: 18px
    fontWeight: '600'
    lineHeight: 24px
  body-lg:
    fontFamily: Space Grotesk
    fontSize: 16px
    fontWeight: '400'
    lineHeight: 22px
  body-md:
    fontFamily: Space Grotesk
    fontSize: 14px
    fontWeight: '400'
    lineHeight: 20px
  body-sm:
    fontFamily: Space Grotesk
    fontSize: 12px
    fontWeight: '400'
    lineHeight: 16px
  metric-xl:
    fontFamily: JetBrains Mono
    fontSize: 28px
    fontWeight: '700'
    lineHeight: 32px
    letterSpacing: -0.03em
  metric-lg:
    fontFamily: JetBrains Mono
    fontSize: 20px
    fontWeight: '600'
    lineHeight: 24px
    letterSpacing: -0.02em
  metric-md:
    fontFamily: JetBrains Mono
    fontSize: 14px
    fontWeight: '500'
    lineHeight: 18px
    letterSpacing: -0.01em
  label-md:
    fontFamily: JetBrains Mono
    fontSize: 12px
    fontWeight: '500'
    lineHeight: 16px
    letterSpacing: 0.04em
  label-sm:
    fontFamily: JetBrains Mono
    fontSize: 10px
    fontWeight: '600'
    lineHeight: 12px
    letterSpacing: 0.08em
rounded:
  sm: 0.25rem
  DEFAULT: 0.5rem
  md: 0.75rem
  lg: 1rem
  xl: 1.5rem
  full: 9999px
spacing:
  gutter: 0.75rem
  margin: 1rem
  space-xs: 0.25rem
  space-sm: 0.5rem
  space-md: 0.75rem
  space-lg: 1.25rem
  space-xl: 2rem
---

## Brand & Style

This design system is tailored for an elite-tier night and long-exposure light painting capture tool on Android. Designed for low-light fieldwork, astrophotography, and kinetic light sculpture, the aesthetic fuses precision instrument design with an evocative darkroom ambiance.

### Emotional Signature & Posture
- **Surgical Precision:** Every control, read-out, and reticle feels calibrated, immediate, and free of visual noise.
- **Atmospheric Immersion:** The interface acts as a minimal HUD hovering above the sensor stream, minimizing retinal fatigue during midnight shooting sessions.
- **Tactile Mastery:** Interactions echo physical high-end camera rigs—tactile dial steppings, deliberate safety lock triggers, and luminescent feedback loops.

### Visual Style
- **AMOLED Glassmorphism:** Pure `#000000` canvas to extinguish display pixels entirely, overlaid with smoked semi-transparent glass plates (`rgba(10, 13, 18, 0.85)` to `rgba(18, 22, 30, 0.92)`) featuring fine specular borders.
- **Instrument HUD Optics:** High-contrast phosphor-inspired accents—Luminous Teal and Neon Magenta—paired with technical telemetry indicators to ensure instant legibility against long-exposure light trails.
- **Night-Vision Ergonomics:** Dimmed ambient states preserve night-adjusted vision, flaring into neon clarity only under active manipulation.

## Colors

The palette is engineered around sensor efficiency, OLED energy conservation, and optical contrast in total darkness.

### Core Chromatic Roles
- **Primary (`#00FFCC` - Luminous Teal):** Active capture states, laser focal reticles, focused slider thumbs, and primary shutter readouts. Emits a calibrated `0 0 12px rgba(0, 255, 204, 0.45)` neon glow when energized.
- **Secondary (`#FF007F` - Neon Magenta):** Light trails tracking path, trace-mode overlays, bulb lock indicators, and secondary vector guides. Emits a `0 0 12px rgba(255, 0, 127, 0.40)` glow on activation.
- **Warning / Hold (`#FFB800` - Amber Gold):** Exposure pause, thermal sensor throttling, clipping alerts, and ND filter warnings.
- **Critical / Recording (`#FF3B30` - Signal Red):** Live sensor integration, active RAW buffer writes, bulb shutter active timer, and tally lamp indicators.

### Surfaces & Glass Overlays
- **Canvas Base:** `#000000` (True pitch black, 0% OLED power drain).
- **Surface Dark (HUD Shelves & Docks):** `rgba(10, 13, 18, 0.85)` with backdrop filter blur (24px).
- **Surface High (Flyout Panels & Modals):** `rgba(18, 22, 30, 0.92)` with backdrop filter blur (32px).
- **Surface Scrim (Expanded Viewports):** `rgba(0, 0, 0, 0.60)`.
- **Specular Hairline Border:** `rgba(255, 255, 255, 0.12)`.
- **Active Structural Border:** `rgba(0, 255, 204, 0.40)`.

### Monochromatic Hierarchy
- **Text & Glyph Bright:** `#FFFFFF` (100% opacity, primary technical readouts).
- **Text & Glyph Standard:** `rgba(255, 255, 255, 0.70)` (Secondary labels, inactive metrics).
- **Text & Glyph Dim:** `rgba(255, 255, 255, 0.38)` (Scale divisions, disabled states, grid lines).

## Typography

Typography establishes an absolute separation between narrative/operational controls and mathematical telemetry.

### Font Pairing Logic
- **Primary Interface UI (`Space Grotesk`):** Modern, geometric, and high-tech without feeling sterile. Used for modal headers, mode descriptions, tooltips, and system notifications.
- **Camera Telemetry & Data Readouts (`JetBrains Mono`):** Fixed-pitch tabular figures. Crucial for live-updating numbers (ISO, shutter speed, Kelvin temperature, EV compensation, live histogram brackets, timer intervals) so layout shift never occurs during exposure.

### Typographic Rules
- Set all `label-sm` and `label-md` tokens to full uppercase for HUD status indications (e.g., `RAW`, `AF-C`, `LIVE BLEND`, `F/1.8`, `ISO 64`).
- Metric tokens enforce fixed numeric widths via OpenType `tnum` (tabular figures) across all device scales.

## Layout & Spacing

The viewfinder prioritizes an unobstructed 100% active sensor framing zone. Controls live in floating optical control decks optimized for single-hand portrait holding and two-hand tripod landscape clamping.

### Layout Model
- **No-Occlusion Safe Area:** Viewfinder area preserves 16:9, 4:3, or 1:1 image boxes without intrusive chrome. HUD bars anchor to the display perimeter with Android display-cutout and navigation inset avoidance.
- **Top Control Strip:** Thin 48px glass bar for passive status (Battery, Thermal, FPS, Storage, Gyro horizon).
- **Lower Control Cockpit:** Multi-tiered glass cluster housing quick-dial parameters, secondary shutter trigger, and expanded thumb-accessible exposure drawer.
- **Side Adjustment Ribbon (Landscape) / Thumb Arc (Portrait):** Vertical or radial slide-rails with 44dp minimum hit-targets for darkroom manipulation.

### Responsive Behaviors
- **Mobile Portrait:** Single-column layout. Quick-access bar docked at the bottom immediately above the OS navigation bar.
- **Mobile Landscape / Tripod Clamped:** Symmetrical dual-dock layout. Left edge houses histogram, focus peaking toggles, and Kelvin/tint; right edge houses shutter trigger, EV wheel, and bulb mode locks.
- **Foldable / Tablet:** Modular multi-window instrumentation. Viewfinder maintains center focus, while telemetry, RGB histogram, and light vector maps break out into docked floating cards.

## Elevation & Depth

In a pure black environment, traditional drop shadows disappear. Depth is articulated exclusively through glass translucence, specular edge highlights, and luminous light emissions.

### The Z-Layer Optical System
- **Level 0 (Sensor Feed Canvas):** Pure black backing `#000000` with the raw camera texture stream.
- **Level 1 (Submerged Reticles & Overlays):** Reticle lines, rule of thirds, light accumulation vectors. Zero background, `rgba(255, 255, 255, 0.25)` strokes, zero blur.
- **Level 2 (HUD Cockpits & Control Ribbons):** Floating glass surfaces (`rgba(10, 13, 18, 0.85)`), 20–24px backdrop blur, 1px perimeter border of `rgba(255, 255, 255, 0.12)`.
- **Level 3 (Interactive Drawers & Expanded Dials):** Surface High (`rgba(18, 22, 30, 0.92)`), 32px backdrop blur, bordered by `rgba(255, 255, 255, 0.18)` with subtle directional light tint from the top.
- **Level 4 (Modal Alarms, Calibrations & Long-Press Menus):** Opaque dark tinted plate (`rgba(8, 10, 14, 0.98)`), framed by high-contrast primary accent hairline (`rgba(0, 255, 204, 0.5)`).

### Neon Bloom & Active Illumination
- Active elements do not elevate using shadows; they illuminate. A toggle or dial thumb glowing with Luminous Teal casts an optical drop-shadow of `0 0 16px rgba(0, 255, 204, 0.40)`.
- Active recording tally emits an intermittent pulse `0 0 20px rgba(255, 59, 48, 0.60)`.

## Shapes

The geometric architecture balances surgical optics and tactile usability. It avoids excessively round toy-like curves, adopting an intentional 8px (`0.5rem`) radius baseline for standard components, extending to 16px (`1rem`) for larger floating HUD cards.

### Corner Radius Allocation
- **HUD Plates, Cards, and Trays (`rounded-lg` / 16px):** Outer glass decks containing parameters and histogram widgets.
- **Buttons, Steppers, and Pill Chips (`rounded-md` / 8px):** Tactile toggles, parameter chips, and input controls.
- **Primary Shutter Trigger:** Perfect circle (`rounded-full`), double-concentric ring layout.
- **Telemetry Indicators & Focus Loupe:** Razor-sharp inner corners (`rounded-none` or 2px) to denote calibrated digital instruments.

## Components

### 1. Primary Shutter & Bulb Trigger
- **Visuals:** 72dp concentric action unit. Outer ring: 2px border in `rgba(255, 255, 255, 0.3)`. Inner disc: pure `#000000` with an inset 56dp core.
- **States:**
  - *Standby:* Inner core has a solid 2px outline of Luminous Teal (`#00FFCC`).
  - *Light Painting Active:* Core fills with Signal Red (`#FF3B30`) with an expanding pulse animation.
  - *Bulb Locked:* A micro Neon Magenta (`#FF007F`) slide-lock icon clicks into place at the apex of the outer ring.

### 2. Pro HUD Sliders (ISO, Shutter Speed, Focus)
- **Visuals:** Thin 2dp track in `rgba(255, 255, 255, 0.15)`. Active range line colored in Luminous Teal.
- **Thumb:** 18dp rounded rect (`rounded-xs` / 4px) filled with `#000000`, framed with 2px solid `#00FFCC`, producing a distinct teal aura when dragged.
- **Tick Marks & Scales:** Rendered with `JetBrains Mono` at `label-sm` along the track with vertical micro-ticks (`1dp x 6dp`) in `rgba(255, 255, 255, 0.25)`.
- **Interaction:** Tap anywhere on a metric chip to expand the slider inline via a spring transition; long-press activates 1/3-stop fine-tuning step mode.

### 3. Parameter Chips (EV, Kelvin, Shutter, ISO)
- **Resting:** Smoked pill (`rgba(10, 13, 18, 0.70)`), border 1px `rgba(255, 255, 255, 0.12)`. Text: standard white, tabular monospace metrics.
- **Active / Expanded:** Glows with primary teal border (`rgba(0, 255, 204, 0.8)`), text switches to `#00FFCC`.

### 4. Interactive Histogram & Exposure Clipping
- **Base:** Translucent deck (`rgba(10, 13, 18, 0.85)`) embedded in the upper HUD.
- **Readout:** Real-time RGB overlay curves with `0.6` alpha blending.
- **Clipping Warning:** Zebra patterns for high-exposure blowout; under-exposure shadows trigger dynamic Amber Gold (`#FFB800`) boundary brackets.

### 5. Reticles and Light Vector Guides
- **Focus Box:** Minimalist 48dp crosshair frame with open center.
  - *Searching:* `rgba(255, 255, 255, 0.40)`.
  - *Locked:* Snaps down to 36dp in `#00FFCC` with short haptic click.
  - *Manual Focus Assist (Peaking):* Neon Magenta (`#FF007F`) fringe highlight overlay over high-contrast edges in the live viewfinder.

### 6. Toggle Buttons & Safety Switches
- **Tripod Mode / Live Composite Toggle:** Compact 36dp square glass toggle (`rounded-md`).
- **Inactive:** Icon rendered in `rgba(255, 255, 255, 0.4)`.
- **Engaged:** Icon illuminates in `#00FFCC` with a subtle inner glow (`rgba(0, 255, 204, 0.15)` background wash).