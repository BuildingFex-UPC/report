# Buenas Prácticas — Análisis contra guías oficiales

Análisis de cumplimiento de las prácticas del proyecto **BuildingFex** (backend .NET, app administrador Kotlin/Compose y app residente Flutter) evaluado contra dos guías oficiales:

1. **Apple Developer — UI Design Dos and Don'ts** (Interactivity, Readability, Graphics, Clarity)
   <https://developer.apple.com/design/tips/#readability>
2. **Android Developer — Core App Quality Guidelines** (calidad mínima de toda app Android)
   <https://developer.android.com/docs/quality-guidelines/core-app-quality?hl=es-419>

> Fecha del análisis: 16/11/2026 · Basado en evidencia verificada en el código (no en suposiciones).

---

## 1. Lo que se cumple ✅

### 1.1 Android Core Quality → *User experience*

| Criterio (ID) | Evidencia en el código |
|---|---|
| **Consistent_UX / Material design components** | Ambas apps usan Material 3: admin `MaterialTheme` + `Switch`/`SectionCard`; residente `useMaterial3: true` (`lib/core/ui.dart:26`). |
| **Theme_Support (claro/oscuro)** | Admin: `Theme.kt` define `darkColorScheme`/`lightColorScheme` con `isSystemInDarkTheme()` (`Theme.kt:41`). Residente: `darkTheme` + `themeMode: ThemeMode.system` (`main.dart:46-48`) con `buildTheme(brightness:)` y `bfCardColor(Brightness)`. |
| **State_Preservation** | ViewModels con `StateFlow` sobreviven rotación; el residente recarga al volver a la pestaña (`common_areas_screen.dart:25-28`) en vez de perder datos. |
| **App_Startup_Time** (<2 s o indicador) | `FullScreenLoading` + `CircularProgressIndicator` en dashboard, reservas, incidencias y áreas comunes. |
| **Back_Button_Nav / Back_Gesture_Nav** | Sin botones "back" custom en pantalla; navegación estándar (NavigationBar del admin, rutas de Flutter). |
| **Minimize_Permissions** | Manifest del residente: solo `INTERNET` (único permiso solicitado). |
| **Target_SDK / Compile_SDK** | `targetSdk = 37`, `compileSdk` configurado, `minSdk = 29` (`app/build.gradle.kts:8-15`). |
| **Touch_Target_Size (48 dp)** | Componentes M3 por defecto (`AppOutlinedButton`, `TextButton`, FABs). |
| **Content_Description** | Iconos con `contentDescription = string(...)` localizado en Incidents, Information, Imports y Shell. |

### 1.2 Android → *Visual quality*

| Criterio (ID) | Evidencia |
|---|---|
| **Graphic_Quality (sin pixelación)** | Íconos vectoriales M3 (`Icons.*`); fotos recortadas con `clip(RoundedCornerShape)`. |
| **Line_Length (45-75 chars)** — parcial | Textos en `Column` con ancho limitado y `bodySmall`; tarjetas con `weight(1f)` que envuelve el texto. |

### 1.3 Apple → *Readability / Clarity*

| Criterio | Evidencia |
|---|---|
| **Text Size ≥ 11 pt** | Mínimos: `bodySmall` (12 sp) en admin, `fontSize: 13` en residente. |
| **Contrast** | Paleta con `onSurfaceVariant` sobre fondo claro; esquemas M3 que cumplen 4.5:1 por construcción; `bfWarning = #ED6C02` sobre blanco ≈ 3.3:1 (apto para chips/texto grande). |
| **Spacing (sin solapes)** | Componentes `VerticalGap(n)` y `Arrangement.spacedBy(...)` sistemáticos en ambas apps. |
| **Organization (controles junto al contenido)** | Switch de mantenimiento junto a su label en el diálogo; Editar/Eliminar debajo de cada fila. |
| **Alignment** | `Row(verticalAlignment = CenterVertically)` y `crossAxisAlignment: CrossAxisAlignment.start` consistentes. |
| **Hit Targets 44 pt** | `TextButton`, `IconButton` y `Switch` de M3 superan 44×44 pt. |

### 1.4 Funcionalidad (ambas guías)

- **Códigos de error de negocio** (`SPACE_UNDER_MAINTENANCE`, `RESERVATION_OVERLAP`…) → el estado del sistema es visible (Apple *Clarity* / Android *Consistent_UX*).
- **i18n es/en completo** → legibilidad en el idioma del usuario.
- **Botón "No disponible" deshabilitado** en vez de fallo silencioso → feedback claro al usuario.
- **Tests automatizados en verde**: backend 30 (xUnit), admin 21 (JUnit), residente 48 (flutter_test, incl. 10 de accesibilidad/tema/contraste) + `flutter analyze` sin issues.

---

## 2. Brechas detectadas ⚠️ / ❌

| Guía | Criterio (ID) | Severidad | Hallazgo | Estado |
|---|---|---|---|---|
| Android | `Network_Security_Traffic` (todo el tráfico por SSL) | 🔴 Alta | `AndroidManifest.xml:18` del admin declara **`usesCleartextTraffic="true"`** → permite HTTP en claro. | ✅ Resuelto (17/11/2026) |
| Android | `Network_Security_Configuration` | 🔴 Alta | No existía `networkSecurityConfig` declarado en el admin. | ✅ Resuelto: `res/xml/network_security_config.xml` con `cleartextTrafficPermitted="false"` + referencia en el manifest |
| Android | Accesibilidad (`Content_Description`) | 🔴 Alta | **Flutter: 0 usos de `Semantics` / `semanticLabel` / `Tooltip`** en todo `lib/` → lectores de pantalla no anuncian chips ("En mantenimiento"), iconos de costo/capacidad ni el estado del botón Reservar. | ✅ Resuelto: `Semantics` en `StatusChip` y `AppOutlinedActionButton` (`enabled`), etiqueta compuesta por tarjeta de espacio, `tooltip` en los `IconButton` de cerrar |
| Android | `Theme_Support` (oscuro en todo el contenido) | 🟡 Media | Residente: `MaterialApp` sin `darkTheme:`/`themeMode:` (`main.dart:43`) → solo tema claro; colores fijos (`Colors.white` en cards, `bfSurface`) no reaccionaban al modo oscuro. | ✅ Resuelto: `darkTheme` + `themeMode: ThemeMode.system`, `buildTheme(brightness:)`, `bfCardColor(Brightness)` en cards y bottom sheets (el QR queda blanco a propósito, para escanear) |
| Android | `Visual_Contrast` (4.5:1 en texto pequeño) | 🟡 Media | `bfWarning #ED6C02` sobre blanco ≈ 3.1:1 y ≈ 4.4:1 sobre su propio tinte del chip → no apto para texto de 12-18 sp. | ✅ Resuelto: tonos `bfWarningText`/`bfSuccessText` según `Brightness` (≈5.5:1 / ≈5.5:1 en claro, ≈7:1 / ≈6:1 en oscuro) aplicados en `StatusChip`, KPIs y `_StatCard` + test de contraste automático |
| Android | `Content_Description` (todo ítem no-TextView) | 🟡 Media | Íconos decorativos con `contentDescription = null` (`AppShell.kt:148,166,173`, `PhotoPickerField.kt:136`); aceptable solo si son decorativos — verificar con Accessibility Scanner. | ⏳ Pendiente (verificación manual) |
| Apple | *High Resolution* (@2x/@3x) | 🟢 Baja | Fotos de perfil/espacios llegan en data-URL; falta verificar escalado en pantallas Retina altas. | ⏳ Pendiente |
| Apple | *Distortion* (aspect ratio) | 🟢 Baja | Se buscó `ContentScale`/`fit` explícito en `AppImage`. | ✅ Verificado: `AppImage.kt:35` usa `ContentScale.Crop` por defecto y ningún caller lo sobreescribe; `ContentScale.Fit` (logo/splash en `MainActivity.kt:91`, `AppShell.kt:224`) no distorsiona. |
| Android | `StrictMode_Compliance` / `Rendering_Performance` (60 fps) | 🟢 Baja | Sin evidencia de StrictMode habilitado ni tests de rendimiento. |
| Android | `Orientation_Transitions` / `Display_State_Parity` | 🟢 Baja | Sin tests de rotación (el plan de testing de Android pide rotar ≥3 veces por pantalla). |
| Android | `T-Line_Length` | 🟢 Baja | Sin verificación de 45-75 caracteres en bloques largos (descripción de incidencias, reglas de área). |

---

## 3. Plan de remediación sugerido

| # | Acción | Impacto | Esfuerzo | Estado |
|---|---|---|---|---|
| 1 | Eliminar `usesCleartextTraffic="true"` y agregar `networkSecurityConfig` con `base-config cleartextTrafficPermitted="false"` | Seguridad | Bajo | ✅ Hecho |
| 2 | Agregar `Semantics`/`semanticLabel` a chips, iconos y botones clave del residente | Accesibilidad | Medio | ✅ Hecho |
| 3 | Implementar `darkTheme` + `themeMode` en el `MaterialApp` del residente y colores que respondan a `Brightness` | UX | Medio | ✅ Hecho |
| 4 | Oscurecer `bfWarning` para texto pequeño (o usarlo solo en badges ≥18 sp) | Contraste | Bajo | ✅ Hecho: tonos por `Brightness` en chips/KPIs + tests de contraste WCAG |
| 5 | Verificar `ContentScale` en `AppImage` del admin | Imágenes | Bajo | ✅ Verificado: `Crop` por defecto, sin overrides (sin cambio de código) |
| 6 | Correr Accessibility Scanner + tests de rotación en ambas apps | Validación | Bajo | 🟡 Parcial: 11 tests automáticos de accesibilidad/tema/contraste agregados (`test/a11y_theme_test.dart`); falta la corrida manual con dispositivo (checklist abajo) |

> **Verificación de los ítems 1-6 (17/11/2026):** admin `gradlew test assembleDebug` → `BUILD SUCCESSFUL` (21 tests); residente `flutter analyze` → *No issues found* + `flutter test` → **48/48** en verde (38 previos + 10 nuevos en `test/a11y_theme_test.dart`). Sin regresiones.

### 3.1 Checklist manual pendiente (ítem 6)

Ejecutar en un dispositivo/emulador con **Accessibility Scanner** (Google Play) y rotación de pantalla:

1. **Accessibility Scanner** sobre las pantallas principales de ambas apps (dashboard, áreas comunes, reservas, incidencias, ajustes) → cero advertencias de *contrast* y de *touch target size*.
2. **TalkBack** (Android): recorrer una tarjeta de espacio → debe leer el nombre, "En mantenimiento" (si aplica), reglas, costo y el estado del botón Reservar en un solo foco; el chip debe anunciar su etiqueta completa.
3. **Rotación**: girar ≥3 veces por pantalla (dashboard, editor de reserva, hoja de reserva) y verificar que no se pierda el formulario en edición ni haya parpadeos.
4. **Modo oscuro**: activar el tema del sistema y revisar dashboard, finanzas, incidencias y la hoja de reserva (el QR debe seguir en fondo blanco para escanear).

---

## 4. Referencias

- Apple Developer — *UI Design Dos and Don'ts*: <https://developer.apple.com/design/tips/#readability>
- Apple Human Interface Guidelines — Typography / Color / Layout: <https://developer.apple.com/design/human-interface-guidelines/>
- Android — Core App Quality Guidelines: <https://developer.android.com/docs/quality-guidelines/core-app-quality?hl=es-419>
- Android — Adaptive App Quality: <https://developer.android.com/docs/quality-guidelines/adaptive-app-quality>
- Material Design 3 — Color contrast: <https://m3.material.io/foundations/designing/color-contrast>
- Material Design 3 — Target sizes: <https://m3.material.io/foundations/designing/structure#dab862b1-e042-4c40-b680-b484b9f077f6>
