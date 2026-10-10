---
name: SuperFotómetro
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
  signal-record: '#FF3B30'
  surface-glass-shelf: rgba(10, 13, 18, 0.85)
  surface-glass-modal: rgba(18, 22, 30, 0.92)
  specular-hairline: rgba(255, 255, 255, 0.12)
  text-dim: rgba(255, 255, 255, 0.38)
  text-standard: rgba(255, 255, 255, 0.70)
  text-bright: '#FFFFFF'
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

This design system delivers an elite, professional-grade telemetry HUD engineered for long-exposure photography, light painting, and low-light astrophotography. Built specifically for demanding night conditions, the aesthetic fuses surgical laboratory instrumentation with deep-night field ergonomics.

### Personality & Posture
- **Surgical Precision:** Zero non-functional decoration. Visual hierarchy mirrors calibrated scientific devices and mechanical rangefinders.
- **Optoelectronic Darkroom Immersion:** Interfaces rest on total pitch black to prevent retinal fatigue and preserve dark-adapted vision during hours of midnight exposure work.
- **Tactile Telemetry:** Every rotary slider, stop-lock, and shutter threshold evokes physical precision apparatuses, with tactile haptic and visual neon blooms confirming state locks.

### Design Movement
- **AMOLED Glassmorphism & Cybernetic Instrument HUD:** Pure `#000000` canvas backdrops eliminate panel light leak on OLED hardware. Smoked, frosted-glass decks float over live sensor viewfinders, bounded by specular hairline borders and illuminated with luminous phosphors.

## Colors

The color palette is engineered around sensor efficiency, OLED energy preservation, and extreme chromatic legibility in dark environments.

### Key Roles
- **Primary (`#00FFCC` - Luminous Teal):** Focal locks, active exposure scrubbers, primary shutter confirmation rings, and operational values. Generates an energetic neon presence.
- **Secondary (`#FF007F` - Neon Magenta):** Light painting path guides, bulb lock mechanisms, exposure peaking fringes, and vector trails.
- **Tertiary (`#FFB800` - Amber Pause):** Exposure hold, clipping warnings, thermal limits, and ND filtration calculation alerts.
- **Neutral (`#000000` - AMOLED Pitch Black):** Base structural canvas, ensuring OLED pixels turn off completely to eliminate stray light in field sessions.

### Named Technical Colors
- **`signal-record` (`#FF3B30`):** Live sensor integration, tally lamp pulses, active bulb timer counts, and RAW write queues.
- **Surface Glass Variants:** Smoked translucent volumes (`rgba(10, 13, 18, 0.85)` to `rgba(18, 22, 30, 0.92)`) paired with `specular-hairline` boundaries (`rgba(255, 255, 255, 0.12)`) create readable instrument banks without occluding framing feeds.

## Typography

Typography strictly distinguishes narrative controls from operational telemetry.

### Font Pairing Logic
- **Space Grotesk:** Primary interface typography. Its architectural proportions and clean humanist geometry supply legibility across tooltips, parameter drawers, modal titles, and exposure modes.
- **JetBrains Mono:** Dedicated telemetry and readout typography. Provides fixed-pitch tabular alignments (`tnum`) for fast-updating numerical data (Kelvin scales, ISO, EV steps, shutter speeds, and live histograms), eliminating jitter and layout shift during dynamic live captures.

### Formatting Rules
- Render all `label-sm` and `label-md` tokens in uppercase to match aerospace and hardware camera standards (`RAW`, `BULB`, `AF-C`, `ISO 100`, `1/250S`).
- Use tabular numerals across all numerical telemetry counters.

## Layout & Spacing

The layout model optimizes interaction for one-handed portrait shooting and two-handed clamped landscape rig orientations, isolating the viewfinder from control occlusion.

### Grid & Composition
- **Unobstructed Viewfinder Frame:** Center framing preserves complete 3:2, 4:3, or 16:9 viewports.
- **Top Sensor Ribbon:** 48px floating strip reserved for secondary telemetry: battery capacity, internal sensor temperature, live FPS, storage remaining, and artificial horizon.
- **Lower Control Cockpit:** Tiered dock clustered at the thumb zone housing parameter pills, shutter trigger, and the secondary exposure wheel.

### Responsive Reflow Rules
- **Portrait:** Vertical layout anchors parameter chips immediately above the shutter ring, utilizing `space-md` gaps.
- **Landscape / Rig Mount:** Cockpit divides into dual edge anchors. Left deck carries the histogram, focus peaking, and WB controls; right deck handles the shutter unit, bulb lock, and EV scroller.
- **Foldable / Tablet:** Expanded multi-column HUD surfaces live RGB wave monitors, light vector maps, and curve editors as detached dockable panes.

## Elevation & Depth

Standard ambient drops disappear against total `#000000` backings. Hierarchy is conveyed via material density, specular rim lighting, and luminous color emission.

### Optical Z-Tiers
- **Layer 0 (Raw Frame Canvas):** Pure `#000000` backing underneath the camera feed.
- **Layer 1 (Submerged Reticles & Guides):** Overlaid composition matrices, light paths, and spot-meter circles with zero blur and subtle translucent strokes (`rgba(255, 255, 255, 0.25)`).
- **Layer 2 (Floating Glass Decks):** Control trays and docks using `surface-glass-shelf` (`rgba(10, 13, 18, 0.85)`), 24px backdrop blur, and 1px borders of `specular-hairline` (`rgba(255, 255, 255, 0.12)`).
- **Layer 3 (Modals & Flyouts):** `surface-glass-modal` (`rgba(18, 22, 30, 0.92)`), 32px backdrop blur, framed with subtle top-lit directional specular highlights (`rgba(255, 255, 255, 0.18)`).
- **Layer 4 (Critical Alerts & Alarms):** Heavy smoked surfaces bounded by sharp `rgba(0, 255, 204, 0.5)` or `rgba(255, 59, 48, 0.6)` active hairlines.

### Neon Emission & Bloom
Active states illuminate instead of casting drop shadows:
- Energized toggles, selected chips, and active thumb targets project an optical glow of `0 0 14px rgba(0, 255, 204, 0.45)`.
- Active bulb recording and integration tally pulses emit `0 0 20px rgba(255, 59, 48, 0.60)`.

## Shapes

The geometric architecture balances mechanical precision with comfortable touch-target ergonomics.

### Corner Application
- **HUD Plates & Floating Cards:** 1rem (`rounded-lg`) corner radii keep large glass containers modern without dominating the angular reticles inside.
- **Interactive Controls & Parameter Chips:** 0.5rem (`rounded`) corner radii provide distinct boundaries for interactive items.
- **Shutter Unit & Quick-Action Triggers:** Continuous circular geometries (`rounded-full`) for tactile differentiation during shooting.
- **Instrument Tick Marks & Level Reticles:** 0px sharp corners to reinforce their role as calibrated measurement tools.

## Components

### Buttons & Shutter Actuators
- **Primary Shutter:** 72dp concentric circle. Outer ring carries a 2px stroke in `rgba(255, 255, 255, 0.3)`. The inner core (56dp) defaults to pitch black with a 2px Luminous Teal (`#00FFCC`) border. During light capture, it transitions to Signal Red (`#FF3B30`) with a rhythmic neon pulse.
- **Bulb Lock Switch:** Radial slider positioned atop the shutter ring. Snapping into position illuminates a Neon Magenta (`#FF007F`) padlock indicator.
- **HUD Glass Buttons:** Smoked plate with 1px `specular-hairline` border, 0.5rem radius, and minimum 44dp hit zones.

### Chips & Parameter Toggles
- **Telemetry Chips (ISO, Shutter, EV, K):** Dark glass pills (`rgba(10, 13, 18, 0.70)`) displaying uppercase monospace values. Tap states activate a 1px Luminous Teal border and glow.
- **Status Badges:** Compact labels rendered with `label-sm` in tabular uppercase (`RAW`, `LIVE BLEND`, `ND 1000`).

### Sliders & Steppers
- **Pro Exposure Sliders:** 2dp track in `rgba(255, 255, 255, 0.15)` with an active fill in `#00FFCC`. The thumb is an 18dp rounded rectangle with a pitch black core and 2px neon border, casting a teal halo when engaged. Tick marks use `JetBrains Mono` at `label-sm`.

### Input Fields & Value Pickers
- **Manual Input Dials:** Numeric stepper inputs set on smoked glass blocks with centered tabular digits (`metric-md`). Flanked by `+` and `-` micro-triggers with instantaneous haptic response.

### Cards & Telemetry Trays
- **Histogram Tray:** Floating top-docked glass panel (`rgba(10, 13, 18, 0.85)`) hosting real-time RGB curves. Clipping points illuminate Amber Gold (`#FFB800`) boundary flags.
- **Sensor Metadata Sheet:** Bottom-docked card using `rounded-lg` containing technical EXIF data, focal distance readouts, and estimated noise thresholds.

### Reticles & Viewfinder Overlays
- **Focus Reticle:** Dynamic 48dp crosshair frame. Remains `rgba(255, 255, 255, 0.40)` while autofocus hunts, and contracts to 36dp with a solid `#00FFCC` stroke upon lock.
- **Focus Peaking:** Sharp 1px Neon Magenta (`#FF007F`) edge highlight accenting high-contrast planes in manual focus mode.