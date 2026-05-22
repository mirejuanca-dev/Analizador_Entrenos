# HR Analyzer Pro
## Manual de usuario y referencia técnica
**Versión actual · Mayo 2026**

---

## Índice

1. [¿Qué es HR Analyzer Pro?](#1-qué-es-hr-analyzer-pro)
2. [Arquitectura técnica](#2-arquitectura-técnica)
3. [Pestaña Importar](#3-pestaña-importar)
4. [Pestaña Análisis](#4-pestaña-análisis)
5. [Pestaña Comparar](#5-pestaña-comparar)
6. [Pestaña Calendario](#6-pestaña-calendario)
7. [Pestaña Perfil y VO₂max](#7-pestaña-perfil-y-vo₂max)
8. [Exportación e importación de datos](#8-exportación-e-importación-de-datos)
9. [Métricas y fórmulas](#9-métricas-y-fórmulas)
10. [Gráficas y visualizaciones](#10-gráficas-y-visualizaciones)
11. [Privacidad y seguridad de datos](#11-privacidad-y-seguridad-de-datos)

---

## 1. ¿Qué es HR Analyzer Pro?

HR Analyzer Pro es un dashboard de análisis de frecuencia cardíaca (FC) orientado al deportista de resistencia. Permite cargar, visualizar y comparar sesiones de entrenamiento exportadas desde dispositivos como **Amazfit/Zepp** o **Apple Watch**, sin necesidad de cuenta, servidor ni conexión permanente a internet.

**Tipos de actividad soportados:**
- 🚵 MTB / Mountain Bike
- 🥾 Hiking / Trail
- 🏋️ WOD / CrossFit
- 🏃 Running

**Formatos de archivo soportados:** GPX · TCX · CSV

---

## 2. Arquitectura técnica

| Característica | Detalle |
|---|---|
| Tecnología | HTML5 + CSS3 + JavaScript puro (sin frameworks) |
| Almacenamiento | `localStorage` del navegador (100% local) |
| Gráficas | Canvas API nativa del navegador |
| Exportación PDF | jsPDF 2.5.1 (CDN cloudflare) |
| Procesado de archivos | FileReader API + DOMParser XML |
| Instalación | Sin instalación. Funciona desde GitHub Pages |
| Offline | Funciona sin conexión una vez cargada (caché Safari) |
| Privacidad | Cero datos enviados a servidores externos |

La aplicación es un único archivo `.html` autocontenido. Todo el CSS, JavaScript y lógica de análisis reside dentro de ese archivo, sin dependencias externas salvo jsPDF para la generación de informes PDF.

---

## 3. Pestaña Importar

### Carga de archivos
Admite arrastre (*drag & drop*) o selección múltiple. El procesado es secuencial mediante una cola interna para evitar conflictos con archivos múltiples.

**Detección automática del tipo de actividad:**
- Por el campo `Sport` del TCX (biking, running, hiking…)
- Por palabras clave en el nombre del archivo (mtb, trail, wod, run…)
- Si no se detecta automáticamente, abre el modal de clasificación manual

### Filtros de búsqueda
La barra de filtros permite combinar:
- **Texto libre** — busca en el nombre de la actividad
- **Rango de fechas** — desde / hasta (campo `date` nativo del navegador)
- **Rango de FC máxima** — filtra por el pico de FC de cada sesión
- **Chips de tipo** — TODAS · MTB · HIKING · WOD · RUNNING

Todos los filtros se aplican simultáneamente y en tiempo real.

### Copia de seguridad
| Función | Formato | Uso |
|---|---|---|
| Exportar JSON | `.json` | Backup completo restaurable |
| Exportar CSV | `.csv` | Compatible con Numbers / Excel |
| Restaurar historial | `.json` | Recupera actividades exportadas |

En iOS/iPadOS la exportación usa la **Web Share API** (Safari 15+), que abre el menú compartir nativo y permite guardar directamente en la app Archivos con la extensión correcta. En escritorio usa descarga directa mediante `<a download>`.

---

## 4. Pestaña Análisis

Se activa al tocar cualquier actividad de la lista. Muestra el análisis completo de la sesión seleccionada.

### Métricas principales (5 tarjetas)

| Métrica | Descripción |
|---|---|
| FC Máxima | Pico absoluto de la sesión en bpm y % sobre FCmáx real |
| FC Media | Media aritmética de todos los registros |
| FC Mínima | Valor mínimo registrado |
| Duración | Minutos totales y número de puntos de datos |
| Recuperación 1 min | Caída de FC desde el pico hasta 8% después del array |

### Alertas automáticas
La app genera alertas contextuales según el % de FCmáx alcanzado:
- ≥ 95% → ⚡ Esfuerzo máximo, recuperación 48h recomendada
- ≥ 85% → ℹ️ Zona anaeróbica
- < 85% → ✅ Intensidad controlada, sesión aeróbica

### Gráfica de FC en el tiempo
Dibujada con **Canvas 2D API**. Incluye:
- Bandas de color de fondo por zona cardíaca (Z1–Z5)
- Línea discontinua roja en FCmáx real del perfil
- Sombreado azul bajo la curva
- **Marcadores de picos detectados** superpuestos (ver sección siguiente)
- Eje X en minutos, eje Y en bpm

El muestreo se reduce automáticamente a máximo 400 puntos para rendimiento óptimo en móvil.

### Tiempo en zonas
Dos visualizaciones complementarias:
- **Barras horizontales** con porcentaje y minutos por zona
- **Gráfico de barras verticales** (Canvas) mostrando la distribución

### Histograma de FC
Agrupa todos los registros en intervalos de 5 bpm. Cada barra recibe el color de la zona cardíaca a la que pertenece ese rango. Permite visualizar la distribución real del esfuerzo a lo largo de la sesión.

### ⚡ Picos de FC detectados

**Algoritmo de detección en tres pasos:**

1. Encuentra todas las rachas continuas de muestras ≥ umbral definido
2. Fusiona rachas separadas menos de N segundos (evita fragmentar un mismo esfuerzo)
3. Calcula para cada pico: FC máxima, FC media, inicio/fin en minutos, duración exacta y zona

**Controles ajustables:**
- **Umbral** (defecto 150 bpm) — FC mínima para considerar un pico
- **Separación mínima** (defecto 60 s) — distancia mínima entre picos distintos

**Para cada pico se muestra:**
- Ranking (🥇🥈🥉…) con color de zona
- FC pico y FC media del período
- Minuto de inicio y fin exactos
- Duración en segundos o minutos
- Calificación de intensidad: Máxima · Alta · Moderada · Controlada
- Barra proporcional comparando duración entre picos

**Resumen:** número total de picos · tiempo total en picos · pico más largo.

Los picos también quedan marcados visualmente en la gráfica principal de FC.

### Estadísticas detalladas
Tabla con 8 parámetros: FC Máxima, FC Media, FC Mínima, Percentil 95, Recuperación, TRIMP, Zona principal y comparativa FCmáx real vs teórica.

### Exportar PDF
Genera un informe A4 con:
- Cabecera con branding y fecha
- Título y metadatos de la actividad
- Grid de 6 métricas clave
- Barras de tiempo en zonas
- Tabla de estadísticas detalladas
- Imagen del gráfico de FC incrustada

En iOS abre el Share Sheet nativo; en escritorio descarga directamente.

---

## 5. Pestaña Comparar

Permite seleccionar 2 o más actividades y generar un análisis comparativo modular.

### Módulos disponibles (8)

| Módulo | Descripción |
|---|---|
| 📊 Resumen FC | Barras de FC máx, media y mínima por actividad |
| 🎯 Tiempo en zonas | Distribución Z1–Z5 lado a lado |
| 📈 Curvas FC | Superposición de las curvas en el tiempo (normalizado al 100%) |
| ⚡ Carga TRIMP | Barras de carga TRIMP con tarjetas de clasificación |
| 💪 Esfuerzo relativo | Índice de esfuerzo ponderado por zona |
| 📅 Progresión | Evolución temporal de FC media y máxima |
| ❤️ Recuperación | Caída de FC post-pico con rating |
| 📋 Tabla completa | Todas las métricas en tabla comparativa |

Todos los módulos son seleccionables individualmente antes de generar el análisis.

---

## 6. Pestaña Calendario

### Vista mensual
Cada día con actividad muestra:
- **Punto de color** según intensidad (TRIMP acumulado del día)
- **Valor TRIMP** numérico
- Al tocar el día → detalle de actividades con nombre, duración, FC máx y TRIMP

**Escala de colores:**
| Color | TRIMP | Intensidad |
|---|---|---|
| 🔵 Azul | < 30 | Baja |
| 🟢 Verde | 30–60 | Moderada |
| 🟡 Amarillo | 60–100 | Media |
| 🟠 Naranja | 100–150 | Alta |
| 🔴 Rojo | > 150 | Máxima |

### 🧠 Detección de sobreentrenamiento (ATL/CTL)

**Métricas calculadas:**

| Métrica | Fórmula | Interpretación |
|---|---|---|
| ATL (carga aguda) | EMA 7 días del TRIMP diario | Fatiga reciente |
| CTL (carga crónica) | EMA 42 días del TRIMP diario | Forma física base |
| Ratio ATL/CTL | ATL ÷ CTL | Balance carga/forma |
| TSB (Training Stress Balance) | CTL − ATL | Estado de forma del día |

Las EMAs (medias móviles exponenciales) usan constantes estándar: k₇ = 2/(7+1) y k₄₂ = 2/(42+1).

**Niveles de alerta:**
- 🟢 Óptimo — ratio ATL/CTL entre 0.7 y 1.15
- 🟠 Carga elevada — ratio 1.15–1.30 (zona de supercompensación)
- ⚠️ Riesgo sobreentrenamiento — ratio > 1.30
- 🟡 Desentrenamiento — ratio < 0.70

Mini-gráfico de barras con los últimos 42 días de TRIMP diario.

---

## 7. Pestaña Perfil y VO₂max

### Datos del deportista
- Edad, FC reposo, FC máxima real, Género, Nombre
- Género afecta: constante TRIMP (1.92 hombres / 1.67 mujeres) y tabla de clasificación Cooper

### Zonas cardíacas (método Karvonen)
Calculadas sobre la **FC de Reserva** (FCmáx − FC reposo), más preciso que el porcentaje directo de FCmáx para deportistas con FC basal baja.

| Zona | % FC Reserva | Nombre |
|---|---|---|
| Z1 | 0–50% | Recuperación |
| Z2 | 50–65% | Aeróbico base |
| Z3 | 65–75% | Aeróbico umbral |
| Z4 | 75–90% | Anaeróbico |
| Z5 | 90–100%+ | VO₂max / Pico |

### 🫁 VO₂max — Estimación multicritero

El VO₂max se estima combinando 4 métodos independientes mediante **media ponderada**. Cada método recibe un peso según la calidad y cantidad de datos disponibles.

#### Método 1 — Uth-Sørensen
```
VO₂max = 15 × (FCmáx / FC reposo)
```
Solo usa datos de perfil. Siempre disponible. **Peso: 1** (bajo).

#### Método 2 — FC Submáxima Z2
Analiza los tramos estables de cada sesión en Zona 2 (aeróbico base). Calcula la FC media en Z2 y la proyecta a FCmáx usando METs estimados por tipo de actividad (MTB: 7, Running: 8, Hiking: 5, WOD: 7).
```
VO₂max ≈ (FCmáx / FC_Z2_media) × (METs × 3.5)
```
Requiere sesiones ≥ 20 min con suficientes muestras en Z2. **Peso: hasta 6** (2 por sesión, máx 3).

#### Método 3 — Ratio TRIMP/FC
Deportistas con mayor VO₂max producen más trabajo a menor FC relativa. Calcula el índice de eficiencia cardíaca sobre todas las sesiones disponibles.
```
Índice = (TRIMP_medio / FC_reserva_relativa) × 0.18
```
Requiere mínimo 3 sesiones. **Peso: hasta 5.**

#### Método 4 — Recuperación HRR1
La caída de FC en el primer minuto post-pico es un predictor validado de VO₂max (Heyward 2010).
```
VO₂max ≈ 18.3 + 0.89 × HRR1
```
Donde HRR1 es la caída media de FC en el primer minuto. **Peso: hasta 6.**

#### Combinación final
```
VO₂max_combinado = Σ(método_i × peso_i) / Σ(peso_i)
```
La confianza del resultado (Alta / Media / Baja) se calcula como promedio ponderado de las confianzas individuales.

#### Clasificación Cooper
Tabla por edad (13–19, 20–29, 30–39, 40–49, 50–59, 60+) y género con 6 niveles: Bajo · Mejorable · Aceptable · Bueno · Excelente · Superior.

#### Gráfica de evolución temporal
Un punto por sesión con VO₂max estimable. Incluye:
- Bandas de color por nivel Cooper como fondo
- Línea de tendencia (regresión lineal simple)
- Indicador de tendencia ↗ ↘ → con variación total en ml/kg/min

---

## 8. Exportación e importación de datos

| Formato | Función | Compatibilidad |
|---|---|---|
| `.json` | Backup completo con todos los datos raw | Restaurar en HR Analyzer Pro |
| `.csv` | Resumen de métricas por actividad | Numbers, Excel, Google Sheets |
| `.pdf` | Informe visual por actividad | Cualquier visor PDF |

**Flujo de exportación en iOS:**
1. Web Share API → menú compartir nativo de iOS
2. El usuario elige "Guardar en Archivos" → nombre y extensión correctos automáticamente
3. Fallback para iOS antiguo: modal con copia al portapapeles

---

## 9. Métricas y fórmulas

### TRIMP (Training Impulse · Banister)
```
TRIMP = Σ [ Δt × r × e^(b×r) ]
```
- **Δt** = duración de cada muestra en minutos
- **r** = FC de reserva relativa = (FC_muestra − FC_reposo) / (FCmáx − FC_reposo)
- **b** = 1.92 hombres / 1.67 mujeres

El factor exponencial `e^(b×r)` penaliza de forma no lineal las intensidades altas. Una muestra a 160 bpm acumula aproximadamente 3× más TRIMP que una a 90 bpm de igual duración.

### Índice de esfuerzo
```
Esfuerzo = Σ [(tiempo_zona_i / tiempo_total) × (i+1) × 100]
```
Media ponderada del número de zona por el tiempo en ella. Z5 contribuye 5× más que Z1.

### Recuperación cardíaca (HRR1)
La app estima HRR1 tomando la FC en el punto donde se alcanza el pico y comparándola con la FC al 8% restante del array de datos. No requiere timestamp preciso.

### Percentil 95
Calculado ordenando todo el array de FC y tomando el valor en la posición 95%. Representa la FC sostenida alta de la sesión, más útil que el máximo absoluto para comparar intensidades.

### VO₂max estimado (perfil rápido)
```
VO₂max ≈ 15 × (FCmáx / FC_reposo)    [Uth-Sørensen]
```
Mostrado en la tarjeta del perfil cardiovascular como valor rápido de referencia.

---

## 10. Gráficas y visualizaciones

Todas las gráficas se generan con la **Canvas 2D API** nativa del navegador, sin librerías externas. Esto garantiza rendimiento en móvil y funcionamiento offline.

### Características comunes
- El canvas se redimensiona al ancho del contenedor en cada renderizado
- Cuadrícula horizontal con etiquetas de eje
- Padding interno: top 20px · right 20px · bottom 30px · left 40px
- Fuente: `system-ui` (tipografía del sistema operativo)

### Gráfica de FC en el tiempo
- Línea azul `#0070f3` sobre fondo con bandas de zona
- Área de relleno semitransparente bajo la curva
- Línea discontinua roja: FCmáx real del perfil
- Marcadores de picos: sombreado vertical + etiqueta de bpm pico
- Reducción de muestreo automática a ≤ 400 puntos

### Barras de zona (análisis)
- Una barra por zona con color propio
- Porcentaje encima de cada barra
- Etiqueta Z1–Z5 debajo

### Histograma de FC
- Agrupación en cubos de 5 bpm
- Color de cada barra según la zona cardíaca del rango
- Etiquetas en el eje X cada 3 cubos

### Gráficas de comparación
- **Barras agrupadas:** hasta 3 métricas por actividad con leyenda
- **Curvas superpuestas:** normalización al 100% del tiempo (eje X relativo)
- **Progresión temporal:** doble línea (FC media + FC máxima) con puntos y valores
- **Barras de zona comparativas:** agrupadas por actividad

### Gráfica VO₂max evolución
- Puntos con color del tipo de actividad
- Área de relleno bajo la curva
- Bandas horizontales de clasificación Cooper como fondo
- Regresión lineal simple como línea de tendencia discontinua
- Indicador de tendencia con variación total

### Gráfica carga 42 días (calendario)
- Una barra por día con color según intensidad TRIMP
- Etiquetas en eje X cada 7 días
- Líneas de cuadrícula horizontales en 0, 50% y 100% del máximo

---

## 11. Privacidad y seguridad de datos

- **Procesado 100% local:** ningún archivo ni dato sale del dispositivo
- **Sin cuenta ni registro:** la app no tiene backend
- **localStorage:** aislado por origen (URL). Solo el código de tu URL puede leerlo. Ninguna otra web, app ni proceso tiene acceso
- **GitHub:** solo sirve el archivo HTML estático. No tiene acceso ni visibilidad de tus datos
- **Backup en tus manos:** exporta el JSON regularmente para tener una copia de seguridad independiente del dispositivo

> El único riesgo real de pérdida de datos es borrar el historial del navegador o desinstalar Safari. El JSON exportado es tu copia de seguridad.

---

*HR Analyzer Pro · Desarrollado con Claude (Anthropic) · Mayo 2026*
