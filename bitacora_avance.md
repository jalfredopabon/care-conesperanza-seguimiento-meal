# Bitácora de Avance y Manual de Arquitectura — ConEsperanza Landing Storytelling

> **Documento maestro para el equipo de desarrollo y próximos agentes.**  
> **Proyecto:** Landing Page Ligera de Storytelling y Rendición de Cuentas — Consorcio Humanitario ConEsperanza.  
> **Fondo:** Fondo Humanitario para Colombia y LAC (OCHA / RHPF LAC).  
> **Socias Implementadoras:** CARE Colombia (Líder de Consorcio), IRC (International Rescue Committee), Mercy Corps, Save the Children.  
> **Corte Oficial de Datos:** 30 de septiembre de 2026.  
> **Sitio en vivo:** [https://jalfredopabon.github.io/care-conesperanza-seguimiento-meal/](https://jalfredopabon.github.io/care-conesperanza-seguimiento-meal/)

---

## 1. Arquitectura Técnica del Proyecto

* **Archivo único (`index.html`):** Toda la aplicación está construida en una arquitectura Single Page Application (SPA) pura en HTML5, CSS3 nativo y JavaScript vanilla (aprox. 210 KB). **No crear archivos HTML separados por organización**; todo el cambio de vistas se realiza dinámicamente en el DOM sin recargas de página.
* **Cero dependencias externas:** No requiere frameworks pesados, Node, React ni Vite; esto garantiza que cargue al instante incluso con conexiones satelitales o intermitentes en zonas rurales.
* **Internacionalización (i18n):** Sistema completo en 3 idiomas (`ES`, `EN`, `FR`) controlado por el diccionario `TRANSLATIONS` y persistido en `localStorage['conesperanza-lang']`.
* **Modo Oscuro / Claro:** Basado en tokens CSS (`--canvas`, `--ink`, `--surface-soft`, etc.) con persistencia en `localStorage['mide-theme']`.

---

## 2. Sistema de Selección Multi-Socia y Ambientación Dinámica

### 2.1. Selector de Organización
* **Entidades configuradas:** `todos` (Consorcio general), `care` (CARE Colombia), `irc` (IRC), `mercy_corps` (Mercy Corps), `stc` (Save the Children).
* **Controlador JS:** Función `selectOrg(orgId)`. Al invocarse:
  1. Aplica el atributo `document.documentElement.setAttribute('data-active-org', orgId)`.
  2. Actualiza las clases `.active` de los botones en escritorio y de las cápsulas en móvil.
  3. Guarda la selección en `localStorage['selected-org']`.
  4. Llama a `renderOrgContent(orgId)` con una transición fade de 100ms que reemplaza los contenidos del diccionario `PARTNERS_DATA`.

### 2.2. Ambientación Visual Institucional (Luxury Subtle Ambient Tint)
Tanto la barra de escritorio (`.org-subbar-desktop`) como la barra flotante móvil (`.bottom-nav-mobile`) responden automáticamente al atributo `data-active-org` con una transición fluida (`0.4s`):

| Socia | Color Base | Relleno Sutil (`--subbar-bg`) | Bordes Simétricos (`--subbar-border`) | Personalidad Visual |
| :--- | :--- | :--- | :--- | :--- |
| **Todos** | Azul OCHA | `#E0F0FA` | `rgba(65, 143, 222, 0.35)` | Institucional, fresco y paraguas humanitario. |
| **CARE** | Naranja CARE | `#FFDBC4` | `rgba(243, 112, 33, 0.35)` | Cálido, acogedor y enfocado en dignidad y género. |
| **IRC** | Amarillo IRC (`#FFC325`) | `#FEF7CD` | `rgba(234, 179, 8, 0.35)` | Amarillo solar limpio, luminoso y sin tintes beige. |
| **Mercy Corps** | Carmesí Mercy (`#E31837`) | `#FFE4E8` | `rgba(227, 24, 55, 0.35)` | Rosa carmesí sutil y técnico para respuesta rápida. |
| **Save the Children** | Rojo STC (`#DA291C`) | `#FEE4E2` | `rgba(218, 41, 28, 0.35)` | Rubí tenue limpio enfocado en niñez y protección. |

* **Simetría de bordes (Opción A):** La subbarra de escritorio cuenta con `border-top` y `border-bottom` coordinados exactamente con el mismo color institucional tenue (`--subbar-border`), enmarcando la barra como una cinta elegante.
* **Comportamiento en móvil:** En pantallas menores a 900px, el encabezado superior mantiene un borde neutro, mientras que la barra flotante inferior hereda el tinte y el borde institucional de la socia seleccionada, expandiendo la cápsula activa con el logo a color.

---

## 3. Estructura de las 6 Secciones Institucionales

Cada una de las páginas (`todos` y las específicas de cada socia) sigue estrictamente el siguiente esquema de 6 secciones con títulos en minúsculas (*sentence case*):

### `01. Panorama general` (`#p1`)
* **Propósito:** Impacto macro y claim principal.
* **Ajuste de encaje (Hero Fit):** En escritorio, esta sección encaja con precisión en el 100% de la altura de la pantalla (`height: calc(100svh - var(--top-bar-h))`), permitiendo ver el título, la narrativa, las métricas y la fotografía vertical completa sin requerir scroll vertical.
* **Métricas macro (Consorcio):**
  * `31.868` — Meta participantes únicos.
  * `14.673` — Alcance conjunto a la fecha.
  * `59.453` — Atenciones totales proyectadas.
  * `4 Socias` — Consorcio ConEsperanza.
* **Enfoque en páginas de socias:** Titulares y 4 KPIs adaptados a la meta específica de cada socia (e.g., atenciones VBG en CARE, transferencias y salud en IRC, kits de emergencia en Mercy Corps, espacios protectores en STC).

### `02. Despliegue territorial` (`#p2`)
* **Propósito:** Presencia geográfica y cobertura en terreno.
* **Texto paraguas:** Distribución geográfica en departamentos y municipios focalizados (zonas rurales y de difícil acceso con mayores brechas de atención humanitaria).
* **Componentes:**
  * Barra horizontal integrada de 4 KPIs territoriales.
  * Fotografía de caminata de campo y trabajo comunitario.
* **Enfoque en páginas de socias:** Detalla los municipios exactos asignados a cada socia, sus rutas de acceso y el número de brigadas desplegadas.

### `03. Sectores de respuesta` (`#p3`)
* **Propósito:** Cobertura sectorial del marco lógico de OCHA.
* **Estructura obligatoria:** Cuadrícula 2x2 con **4 tarjetas fijas**:
  1. `01 VBG`: Prevención, rutas de atención y mitigación de violencia basada en género.
  2. `02 Protección`: Protección general, gestión de casos, niñez y kits de dignidad.
  3. `03 Salud`: Salud primaria, consultas integrales y atención en salud sexual/reproductiva.
  4. `04 MPCA`: Asistencia monetaria multipropósito y transferencias humanitarias en efectivo.
* **Enfoque en páginas de socias:** Cada socia mantiene las 4 tarjetas, pero detalla el paquete de servicios e insumos específicos que entrega en cada uno de estos sectores.

### `04. Voces de la comunidad` (`#p4`)
* **Propósito:** Evidencia cualitativa y dignidad de la atención.
* **Estructura obligatoria:** **4 citas / testimonios (*blockquotes*) con borde vertical**, una por cada sector de respuesta:
  * Testimonio VBG (dignidad y acompañamiento psicosocial).
  * Testimonio Protección (seguridad familiar y niñez).
  * Testimonio Salud (acceso oportuno a medicamentos y consultas).
  * Testimonio Transferencias Monetarias (decisión y autonomía comunitaria).
* **Enfoque en páginas de socias:** Citas textuales reales de participantes atendidos directamente por los equipos de campo de esa socia.

### `05. Rendición de cuentas (AAP)` (`#p5`)
* **Propósito:** Mecanismos de retroalimentación y quejas/peticiones (Accountability to Affected Populations).
* **Componentes:**
  * Fotografía de monitoreo y encuestas de campo mediante KoboToolbox.
  * Tarjeta destacada (*callout body*) con líneas telefónicas confidenciales, buzones comunitarios y porcentaje de casos respondidos y cerrados dentro de los plazos establecidos.
* **Enfoque en páginas de socias:** Mención de los canales específicos que opera cada socia en sus puntos de atención.

### `06. Aprendizaje y adaptación` (`#p6`)
* **Propósito:** Adaptación a dinámicas de seguridad y proyección futura.
* **Estructura obligatoria:** Dos tarjetas en paralelo (*split layout*):
  1. `Adaptación Operativa`: Ajustes implementados ante bloqueos de vías, alertas del SAT/Defensoría y flexibilización de brigadas móviles.
  2. `Puente hacia la Fase 2`: Sostenibilidad de las acciones, articulación con donantes y fortalecimiento de liderazgos comunitarios.
* **Enfoque en páginas de socias:** Lecciones operativas y prioridades de continuidad de cada organización.

---

## 4. Encabezado y Pie de Página Institucional

### Encabezado (Top Bar)
* Marca tipográfica: **ConEsperanza** arriba en negrita, **REPORTE DE CONSORCIO** abajo en `#717171` (sin distintivos "H" ni puntos verdes que agreguen ruido).
* Controles: Selector de idioma (`ES | EN | FR`), alternador de tema claro/oscuro y botón directo simplificado `[ 📊 Power BI ]`.

### Pie de Página (Fórmula A)
Organizado en 4 columnas equilibradas:
1. **Identidad:** Resumen del Consorcio ConEsperanza bajo el Fondo OCHA / RHPF LAC.
2. **Navegación:** Enlaces ancla directos a las 6 secciones (`01. Panorama general` ... `06. Aprendizaje y adaptación`) con tipografía en minúsculas.
3. **Socias implementadoras:** Enlaces interactivos que al hacer clic seleccionan automáticamente la socia y actualizan la vista sin recarga (`javascript:selectOrg('care')`, etc.).
4. **Datos del reporte:** Mención a OCHA Colombia, fecha de corte oficial (**30 de septiembre de 2026**) y botón de acceso al tablero completo de Power BI.

---

## 5. Protocolo de Trabajo Obligatorio para Próximos Agentes

1. **Protocolo de Diálogo y Aprobación (Paso 0):**
   * **NUNCA** modificar código de golpe ante requerimientos de diseño o estructura.
   * Explicar primero el diagnóstico, plantear alternativas claras y esperar la confirmación explícita del usuario (`"procede"`).
2. **Respeto a la Estructura Existente:**
   * No eliminar tarjetas en la Sección 3 (siempre deben existir las 4 tarjetas sectoriales).
   * No reducir las citas en la Sección 4 (siempre deben ser 4 citas verticales).
   * No crear archivos HTML sueltos; cualquier nueva socia o contenido debe agregarse dentro del objeto `PARTNERS_DATA` en `index.html`.
3. **Reglas Tipográficas y Textuales:**
   * En español, los títulos y subtítulos van en **minúsculas / sentence case** (ejemplo: `01. Panorama general`, nunca `01. Panorama General`).
   * La fecha oficial es **30 de septiembre de 2026** (septiembre no tiene 31 días).
   * No amarrar la narrativa general de "Todos" a un solo departamento (usar redacción paraguas para zonas priorizadas de Colombia).
4. **Flujo de Despliegue Git (Doble Rama):**
   * Para que los cambios se reflejen en la web pública de GitHub Pages, es **indispensable** sincronizar `main` y `gh-pages`:

```bash
# 1. Commit en main
git -C "<ruta_repo>" checkout main
git -C "<ruta_repo>" add index.html bitacora_avance.md
git -C "<ruta_repo>" commit -m "feat: descripción clara"
git -C "<ruta_repo>" push origin main

# 2. Merge y push a gh-pages (Sitio en vivo)
git -C "<ruta_repo>" checkout gh-pages
git -C "<ruta_repo>" merge main
git -C "<ruta_repo>" push origin gh-pages

# 3. Retornar a main
git -C "<ruta_repo>" checkout main
```

5. **Notificación al Usuario:**
   * Al finalizar cualquier cambio o despliegue, concluir siempre con la frase de control:
   > **Listo para que revises.**
