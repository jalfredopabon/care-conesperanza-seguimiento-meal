# Plan de Migración y Reescritura — ConEsperanza Landing Storytelling

> **Propósito de este documento:** Guía completa para cualquier agente que retome este proyecto.
> Contiene el diagnóstico del estado actual, la arquitectura objetivo y el plan de ejecución paso a paso.
> **No ejecutar ningún paso sin leer todo el documento primero.**

---

## 0. Contexto del Proyecto

- **Proyecto:** Landing Page Ligera de Storytelling — Consorcio Humanitario ConEsperanza.
- **Repositorio:** `care-conesperanza-seguimiento-meal`
- **Sitio en vivo:** https://jalfredopabon.github.io/care-conesperanza-seguimiento-meal/
- **Ruta local:** `C:\Users\JosePabon\Music\proyecto_seguimiento_poa\02_LANDING_STORYTELLING\proyecto_app_ligera\index.html`
- **Archivo:** Monolito único `index.html` (~209 KB, 2.554 líneas). **No crear archivos separados.**
- **Deploy:** Doble rama (`main` → `gh-pages`). Ver protocolo al final de este documento.

### Socias implementadoras
| ID | Nombre | Color institucional |
|---|---|---|
| `todos` | Consorcio ConEsperanza (general) | Azul OCHA `#418FDE` |
| `care` | CARE Colombia | Naranja `#F37021` |
| `irc` | IRC — International Rescue Committee | Amarillo `#FFC325` |
| `mercy_corps` | Mercy Corps | Carmesí `#E31837` |
| `stc` | Save the Children | Rojo `#DA291C` |

---

## 1. Diagnóstico del Estado Actual (Auditoría)

### Métricas del archivo actual
- **Total de líneas:** 2.554
- **Tamaño:** ~209 KB
- **CSS:** líneas 4 a 1.013 (1.009 líneas)
- **HTML body:** líneas ~1.013 a 1.390
- **JavaScript:** líneas 1.390 a 2.551 (1.161 líneas)
- **Variables CSS:** 29 custom properties definidas
- **Funciones JS:** 26 funciones
- **Secciones:** p1 a p6 (6 secciones completas)
- **Socias con data completa:** todos, care, irc, mercy_corps, stc

### Qué funciona bien
- Las 6 secciones (#p1 al #p6) están maquetadas y son visualmente sólidas.
- El cambio de socia (selectOrg()) renderiza el contenido correcto de cada una de las 5 socias.
- El tema claro/oscuro fue corregido (fix en sesión actual): usa toggleCurrentTheme() en escritorio y móvil.
- Las 29 variables CSS manejan correctamente los temas por atributo data-theme.
- La ambientación de color institucional por socia funciona con data-active-org.
- El nav móvil (bottom tabs) y el drawer están operativos.
- Hay gráficas funcionales (tendencias, donut, divergentes, burbujas, índice).

### Qué no funciona: Problema Principal — i18n incompleto

El sistema de idioma tiene dos funciones desconectadas que se "pelean":

1. `setLanguage(lang)` → llama `applyTranslations()` → solo traduce 30 claves superficiales
   (eyebrows, H1, H2, párrafo lead). NO traduce el contenido profundo.
2. `selectOrg(orgId)` → llama `renderOrgContent()` → IGNORA el idioma, inyecta HTML en español.

Resultado: Si estás en inglés y cambias de socia, todo vuelve al español.

#### Contenido que NO se traduce actualmente:
| Sección | Qué falta traducir |
|---|---|
| `#p2` Despliegue territorial | Etiquetas de los 4 KPIs territoriales |
| `#p3` Sectores de respuesta | Títulos y descripciones de las 4 tarjetas sectoriales |
| `#p4` Voces de la comunidad | Las 4 citas/testimonios + roles |
| `#p5` Rendición de cuentas (AAP) | Callout, líneas confidenciales |
| `#p6` Aprendizaje y adaptación | Las 2 tarjetas en paralelo |
| Footer | Las 4 columnas completas |
| Socias CARE, IRC, Mercy Corps, STC | Todo su contenido narrativo específico |

### Ajustes visuales pendientes (a aplicar durante migración)
- Tamaños de H1, H2, H3 y paddings (revisar con usuario).
- Modificaciones visuales en tarjetas sectoriales de #p3 (tamaños y jerarquía interna).

### Deuda técnica
| Deuda | Detalle |
|---|---|
| 26 funciones JS dispersas | Sin orden lógico, algunas duplicadas |
| CSS de 1.009 líneas | Sin tokens tipográficos centralizados |
| JS de 1.161 líneas | Datos y lógica mezclados |
| TRANSLATIONS + PARTNERS_DATA separados | Deben fusionarse en un objeto |
| renderOrgContent + applyTranslations | Trabajo superpuesto, deben unificarse |

---

## 2. Arquitectura Objetivo

### Principio: "Single Source of Truth"
Un cambio en un lugar → se refleja en toda la página.

### Estructura interna del nuevo index.html

```
index.html
│
├── <style>          ← TODO EL CSS
│    ├── Tokens/variables (tipografía, espaciado, colores de tema)
│    ├── Estilos del header, secciones, tarjetas, footer
│    └── Responsive / media queries
│
├── <body>           ← ESQUELETO HTML MÍNIMO (contenedores vacíos)
│    ├── Header (botones de idioma, tema, Power BI)
│    ├── Subbarra de socias (escritorio)
│    ├── <section id="p1"> ... <section id="p6"> (vacíos, JS los llena)
│    ├── Footer (vacío, JS lo llena)
│    ├── Drawer móvil
│    └── Bottom nav móvil
│
└── <script>         ← TODO EL JS, EN ESTE ORDEN EXACTO
      ├── Bloque A: MASTER_DATA       ← Todos los textos (ES/EN/FR × 5 socias)
      ├── Bloque B: AppState          ← { org, lang, theme }
      ├── Bloque C: renderApp()       ← Motor único de render
      └── Bloque D: Eventos           ← Clic → setState → renderApp
```

### Estructura de MASTER_DATA

```javascript
const MASTER_DATA = {
  todos: {
    ES: { hero: {...}, p2: {...}, p3: {...}, p4: {...}, p5: {...}, p6: {...}, footer: {...} },
    EN: { hero: {...}, p2: {...}, p3: {...}, p4: {...}, p5: {...}, p6: {...}, footer: {...} },
    FR: { hero: {...}, p2: {...}, p3: {...}, p4: {...}, p5: {...}, p6: {...}, footer: {...} }
  },
  care:        { ES: {...}, EN: null, FR: null },  // null = fallback a ES
  irc:         { ES: {...}, EN: null, FR: null },
  mercy_corps: { ES: {...}, EN: null, FR: null },
  stc:         { ES: {...}, EN: null, FR: null }
};
```

Fallback automático: si MASTER_DATA[org][lang] es null, usa MASTER_DATA[org]['ES'].

### AppState y motor único

```javascript
const AppState = { org: 'todos', lang: 'ES', theme: 'light' };

function setState(patch) {
  Object.assign(AppState, patch);
  try { localStorage.setItem('app-state', JSON.stringify(AppState)); } catch(e) {}
  renderApp();
}

function renderApp() {
  const data = MASTER_DATA[AppState.org][AppState.lang]
            || MASTER_DATA[AppState.org]['ES'];
  // Pinta: header global, 6 secciones, footer, drawer móvil
}
```

Cualquier cambio usa el mismo patrón:
- Cambio de idioma: setState({ lang: 'EN' })
- Cambio de socia: setState({ org: 'care' })
- Cambio de tema: setState({ theme: 'dark' })

### Cómo añadir un nuevo idioma (portugués, ejemplo)
1. Añadir PT: {...} dentro de MASTER_DATA.todos (y socias si aplica).
2. Añadir <button data-lang="PT">PT</button> en el header.
3. Nada más. El motor lo toma automáticamente.

---

## 3. Plan de Ejecución — Pasos en Orden

### FASE 0: Preparación
- [ ] Leer este documento completo.
- [ ] Verificar `git status` limpio en `main`.
- [ ] Crear backup: `git tag backup-pre-migracion`.

### FASE 1: Nuevo CSS (tokens primero)
- [ ] Sección de tokens tipográficos:
      --text-display, --text-h1, --text-h2, --text-h3, --text-body, --text-small
      --space-section, --space-card, --space-inner
      --radius-card, --radius-btn
- [ ] Ajustes visuales (REVISAR CON USUARIO antes de escribir):
      - Tamaños de H1, H2, H3
      - Modificaciones en tarjetas de #p3
- [ ] Migrar estilos que funcionan: colores institucionales, subbarra, header, ambientación por socia.
- [ ] Mantener reglas data-theme="light/dark" y data-active-org="[org]".

### FASE 2: Esqueleto HTML
- [ ] Header: logo/marca, botones ES/EN/FR, botón tema, botón Power BI.
- [ ] Subbarra de socias (escritorio).
- [ ] Contenedores vacíos #p1 a #p6.
- [ ] Footer (vacío).
- [ ] Drawer móvil (idioma + tema + Power BI).
- [ ] Bottom nav móvil (socias).

### FASE 3: MASTER_DATA (los datos)
- [ ] Migrar todos → ES (del PARTNERS_DATA actual).
- [ ] Escribir todos → EN (expandir desde TRANSLATIONS.EN actual).
- [ ] Escribir todos → FR (expandir desde TRANSLATIONS.FR actual).
- [ ] Migrar care → ES, irc → ES, mercy_corps → ES, stc → ES.
- [ ] Configurar EN: null, FR: null en socias para activar fallback.

### FASE 4: Motor de render
- [ ] Implementar AppState.
- [ ] Implementar setState(patch).
- [ ] Implementar renderApp() con fallback automático.
- [ ] Renderizar: header global, 6 secciones, footer, drawer.

### FASE 5: Eventos
- [ ] Clic en idioma → setState({ lang })
- [ ] Clic en socia → setState({ org })
- [ ] Clic en tema → setState({ theme })
- [ ] Persistencia en localStorage.
- [ ] Restaurar estado al cargar.

### FASE 6: Gráficas
- [ ] Migrar funciones: drawTrend, drawDiverge, drawBubble, drawDonut, drawIndex, drawRev.
- [ ] Conectar renderAll() dentro de renderApp().

### FASE 7: Pruebas y deploy
- [ ] Cambio de idioma en todas las socias.
- [ ] Cambio de tema en todas las vistas.
- [ ] Nav móvil (drawer y bottom tabs).
- [ ] Persistencia (cerrar y reabrir).
- [ ] Deploy (ver Protocolo abajo).

---

## 4. Reglas de Diseño Obligatorias (Preservar del original)

1. "No-Line" Rule: No usar bordes 1px solid para separar secciones. Usar cambios de background-color.
2. Sentence case en español: títulos en minúsculas (01. Panorama general, no 01. Panorama General).
3. Fecha oficial: 30 de septiembre de 2026.
4. Redacción paraguas en "Todos": no amarrar la narrativa a un solo departamento.
5. 4 tarjetas fijas en #p3: VBG, Protección, Salud, MPCA. Nunca eliminar ni reducir.
6. 4 testimonios en #p4: uno por sector. Nunca reducir.
7. No crear archivos HTML separados por socia. Todo en index.html.

---

## 5. Protocolo de Deploy Git

```bash
# 1. Commit en main
git -C "C:\Users\JosePabon\Music\proyecto_seguimiento_poa\02_LANDING_STORYTELLING\proyecto_app_ligera" checkout main
git -C "C:\Users\JosePabon\Music\proyecto_seguimiento_poa\02_LANDING_STORYTELLING\proyecto_app_ligera" add index.html
git -C "C:\Users\JosePabon\Music\proyecto_seguimiento_poa\02_LANDING_STORYTELLING\proyecto_app_ligera" commit -m "feat: descripcion del cambio"
git -C "C:\Users\JosePabon\Music\proyecto_seguimiento_poa\02_LANDING_STORYTELLING\proyecto_app_ligera" push origin main

# 2. Merge y push a gh-pages (sitio en vivo)
git -C "C:\Users\JosePabon\Music\proyecto_seguimiento_poa\02_LANDING_STORYTELLING\proyecto_app_ligera" checkout gh-pages
git -C "C:\Users\JosePabon\Music\proyecto_seguimiento_poa\02_LANDING_STORYTELLING\proyecto_app_ligera" merge main
git -C "C:\Users\JosePabon\Music\proyecto_seguimiento_poa\02_LANDING_STORYTELLING\proyecto_app_ligera" push origin gh-pages

# 3. Regresar a main
git -C "C:\Users\JosePabon\Music\proyecto_seguimiento_poa\02_LANDING_STORYTELLING\proyecto_app_ligera" checkout main
```

---

## 6. Protocolo de Diálogo con el Usuario

1. NUNCA modificar código sin explicar diagnóstico y esperar "procede".
2. Un ajuste a la vez.
3. Al finalizar siempre concluir con: **Listo para que revises.**

---


---

## 7. Inventario de Assets (Imágenes, Logos, Fuentes)

> **Regla crítica:** Todos los archivos de imagen y SVG deben permanecer en la **misma carpeta** que `index.html`.
> Las rutas son relativas. El nuevo HTML los llama exactamente igual que el actual. **No mover ni renombrar estos archivos.**

### Imágenes fotográficas (.jpg)

| Archivo | Sección | Descripción |
|---|---|---|
| `img_section_1.jpg` | `#p1` Hero | Fotografía vertical — persona mirando al horizonte en territorio comunitario |
| `img_section_3.jpg` | `#p2` Despliegue territorial | Presencia humanitaria y trabajo territorial en zonas comunitarias |
| `img_section_5.jpg` | `#p5` Rendición de cuentas (AAP) | Monitoreo y evaluación en campo con KoboToolbox |
| `footer.jpg` / `footer.jpeg` | Footer | Ilustración ConEsperanza Colombia (con fallback automático: si no carga .jpg, carga .jpeg) |

### Logos de socias (.svg)

| Archivo | Socia | Dónde aparece |
|---|---|---|
| `onu_logo.svg` | Todos (Consorcio / OCHA) | Subbarra escritorio + bottom nav móvil |
| `care_logo.svg` | CARE Colombia | Subbarra escritorio + bottom nav móvil |
| `irc_logo.svg` | IRC | Subbarra escritorio + bottom nav móvil |
| `mercy_corps.svg` | Mercy Corps | Subbarra escritorio + bottom nav móvil |
| `stc_logo.svg` | Save the Children | Subbarra escritorio + bottom nav móvil |

Nota: cada logo aparece dos veces en el HTML (una en escritorio, una en móvil). El nuevo HTML debe mantener esta estructura.

### Fuentes (incrustadas en base64 — sin dependencia de internet)

| Fuente | Rol | Cómo está cargada |
|---|---|---|
| **Besley** | Serif editorial — titulares H1, H2, displays | `@font-face` con base64 dentro del `<style>` |
| **Inter** | Sans-serif funcional — cuerpo de texto, botones, etiquetas | `@font-face` con base64 dentro del `<style>` |

> Las fuentes están **completamente incrustadas** en el HTML. No dependen de Google Fonts ni de conexión a internet.
> Esto es intencional para garantizar carga instantánea en zonas rurales con conectividad limitada.
> Al migrar, los bloques `@font-face` se copian íntegros al inicio del nuevo `<style>`.

### Íconos

| Tipo | Qué son | Cómo están implementados |
|---|---|---|
| Íconos del drawer móvil | 🌐 (idioma) y 🌓 (tema) | Emojis Unicode — no requieren librería |
| Ícono Power BI | Gráfico de barras amarillo | SVG inline directo en el HTML |
| Íconos de navegación | Hamburguesa, chevrons, etc. | SVG inline directo en el HTML |

> No hay dependencias externas de íconos (no Font Awesome, Lucide, Heroicons, etc.).
> Todo está incrustado. Al migrar, se copian los SVGs inline tal cual están.



---

## 8. Guía Técnica de Implementación para Gemini

> Esta sección es el manual de ejecución. Contiene los patrones exactos de CSS, HTML y JS que debes usar.
> **No improvises nombres ni estructuras.** Si algo no está aquí, pregunta antes de inventar.

---

### 8.1. CSS — Variables y Tokens

#### Cómo está hoy (preservar exactamente estos nombres y valores)
El CSS usa `data-theme` en el elemento `:root` para manejar temas. Estos son los tokens existentes:

```css
/* TEMA CLARO (default) */
:root, :root[data-theme="light"] {
  --ink:         #050A0E;     /* Texto principal */
  --ink-soft:    #3D3D3D;     /* Texto secundario */
  --ink-muted:   #717171;     /* Texto terciario / etiquetas */
  --canvas:      #F8FAFC;     /* Fondo principal de página */
  --canvas-muted:#F1F5F9;     /* Fondo de secciones alternas */
  --surface-soft:#E2E8F0;     /* Fondo de tarjetas */
  --hairline:    #E2E8F0;     /* Borde divisor principal */
  --hairline-soft:#EDF2F7;    /* Borde divisor suave */
  --on-ink:      #FFFFFF;     /* Texto sobre fondos oscuros */
  --on-ink-muted:#A6A6A6;
  --c-up:        #2E6B38;     /* Color positivo (charts) */
  --c-down:      #C0392B;     /* Color negativo (charts) */
  --c-accent:    #6E120B;     /* Acento */
  --grid:        #E2E8F0;     /* Líneas de grilla de charts */
  --axis:        #9A9A94;     /* Ejes de charts */
  --font-d: "Besley", Georgia, serif;      /* Display / Titulares */
  --font-b: "Inter", -apple-system, sans-serif; /* Body / Funcional */
  --wrap:   1240px;           /* Max-width del contenedor */
}

/* TEMA OSCURO */
:root[data-theme="dark"] {
  --ink:         #F3F4F6;
  --ink-soft:    #D1D5DB;
  --ink-muted:   #9CA3AF;
  --canvas:      #14171A;
  --canvas-muted:#1B1E23;
  --surface-soft:#23272E;
  --hairline:    #2C313A;
  --hairline-soft:#22262E;
  --on-ink:      #14171A;
  --c-up:        #4EAF61;
  --c-down:      #E74C3C;
  --c-accent:    #D96A61;
  --grid:        #2C313A;
  --axis:        #6B7280;
}
```

#### Tokens NUEVOS que debes agregar (tipografía y espaciado centralizado)
Agrégalos dentro de `:root` en la sección de variables:

```css
:root {
  /* --- TIPOGRAFÍA CENTRALIZADA --- */
  --text-display: 3.25rem;   /* Hero H1 */
  --text-h1:      2.25rem;   /* H1 de secciones */
  --text-h2:      1.65rem;   /* H2 subtítulos */
  --text-h3:      1.1rem;    /* H3 tarjetas */
  --text-body:    1rem;      /* Cuerpo de texto */
  --text-small:   0.875rem;  /* Etiquetas y badges */
  --text-micro:   0.75rem;   /* Meta info */

  /* --- ESPACIADO CENTRALIZADO --- */
  --space-section: 72px;     /* Padding vertical entre secciones */
  --space-card:    24px;     /* Padding interno de tarjetas */
  --space-inner:   16px;     /* Espaciado interno menor */

  /* --- BORDES Y RADIOS --- */
  --radius-card:   14px;     /* Border-radius de tarjetas */
  --radius-btn:    8px;      /* Border-radius de botones */
  --radius-pill:   999px;    /* Border-radius de pills/badges */
}
```

#### Regla "No-Line": Cómo separar secciones SIN bordes
**PROHIBIDO:** `border: 1px solid` para separar secciones de contenido.
**CORRECTO:** Usar cambio de `background-color` entre secciones:

```css
/* Alternancia de fondo entre secciones */
.page { background: var(--canvas); }
.page.alt { background: var(--canvas-muted); }
```

La **única excepción** permitida para bordes visibles es el `border-top` de la subbarra de socias, que usa `--subbar-border`.

---

### 8.2. CSS — Ambientación de Socias (NO modificar)

El sistema de color institucional funciona con dos atributos en el `<html>`:
- `data-theme="light"` o `"dark"`
- `data-active-org="todos"` | `"care"` | `"irc"` | `"mercy_corps"` | `"stc"`

La combinación de ambos define las variables de la subbarra:

```css
/* Ejemplo de patrón (copiar exactamente del original, NO reescribir): */
:root[data-theme="light"][data-active-org="todos"] {
  --subbar-bg:        #F0F6FA;
  --subbar-border:    rgba(65, 143, 222, 0.20);
  --subbar-bg-mobile: rgba(240, 246, 250, 0.94);
}
:root[data-theme="light"][data-active-org="care"] {
  --subbar-bg:        #FFF7ED;
  --subbar-border:    rgba(243, 112, 33, 0.22);
  --subbar-bg-mobile: rgba(255, 247, 237, 0.94);
}
/* ... idem para irc, mercy_corps, stc en light y dark */
```

Estos bloques ya existen en el CSS actual. **Cópialos tal cual al nuevo `<style>`.**

La subbarra los consume así:
```css
.org-subbar-desktop {
  background:  var(--subbar-bg,     var(--canvas-muted));
  border-top:  1px solid var(--subbar-border, var(--hairline));
  border-bottom: 1px solid var(--subbar-border, var(--hairline));
  /* Transición fluida al cambiar de socia: */
  transition: background .4s ease, border-color .4s ease;
}
```

---

### 8.3. HTML — Esqueleto Body (contenedores vacíos que JS llena)

El `<body>` debe tener **solo la estructura**, sin contenido de texto. El JS lo inyecta.

```html
<body>
  <!-- HEADER -->
  <header class="top-bar">
    <div class="header-main-bar">
      <div class="wrap-bar">
        <!-- Marca (izquierda) -->
        <a href="#" class="brand-identity">
          <div class="brand-meta">
            <div class="brand-title">ConEsperanza</div>
            <div class="brand-sub"><!-- JS inyecta el texto según idioma --></div>
          </div>
        </a>
        <!-- Controles escritorio (>= 900px) -->
        <div class="top-controls-desktop">
          <div class="lang-segment">
            <button class="lang-btn active" data-lang="ES" onclick="setState({lang:'ES'})">ES</button>
            <button class="lang-btn" data-lang="EN" onclick="setState({lang:'EN'})">EN</button>
            <button class="lang-btn" data-lang="FR" onclick="setState({lang:'FR'})">FR</button>
          </div>
          <button class="theme-toggle-btn" id="themeBtn" onclick="setState({theme: AppState.theme==='dark'?'light':'dark'})" aria-label="Cambiar tema">◑</button>
          <button class="pbi-btn-desktop" id="pbiBtn" onclick="openPowerBI()">
            <!-- SVG inline del ícono de Power BI aquí -->
            <span id="pbiBtnLabel">Power BI</span>
          </button>
        </div>
        <!-- Botón menú móvil (< 900px) -->
        <button class="menu-btn-mobile" onclick="toggleDrawer(true)">
          <!-- SVG hamburguesa inline -->
        </button>
      </div>
    </div>
    <!-- Subbarra de socias (escritorio) -->
    <div class="org-subbar-desktop">
      <div class="subbar-inner">
        <span class="subbar-label" id="subbarLabel"><!-- JS inyecta --></span>
        <!-- Tabs de socias: el JS las genera, o van en HTML estático -->
        <button class="org-tab active" data-org="todos" onclick="setState({org:'todos'})">
          <img src="onu_logo.svg" alt="Todos" class="org-icon">
          <span>Todos</span>
        </button>
        <!-- ... care, irc, mercy_corps, stc igual -->
      </div>
    </div>
  </header>

  <!-- SECCIONES PRINCIPALES (vacías, JS las llena) -->
  <main>
    <section id="p1" class="page"></section>
    <section id="p2" class="page alt"></section>
    <section id="p3" class="page"></section>
    <section id="p4" class="page alt"></section>
    <section id="p5" class="page"></section>
    <section id="p6" class="page alt"></section>
  </main>

  <!-- FOOTER (vacío, JS lo llena) -->
  <footer class="site-footer" id="site-footer"></footer>

  <!-- DRAWER MÓVIL -->
  <div id="drawerBackdrop" class="drawer-backdrop" onclick="toggleDrawer(false)"></div>
  <div id="mobileDrawer" class="mobile-drawer">
    <!-- Contenido del drawer: idioma, tema, Power BI -->
    <!-- JS inyecta los labels según idioma -->
  </div>

  <!-- BOTTOM NAV MÓVIL -->
  <nav class="bottom-nav-mobile">
    <!-- 5 tabs de socias para móvil -->
  </nav>
</body>
```

---

### 8.4. JavaScript — AppState (el corazón del sistema)

```javascript
/* ============================================================
   BLOQUE B: ESTADO GLOBAL DE LA APP
   ============================================================ */
const AppState = {
  org:   'todos',  // 'todos' | 'care' | 'irc' | 'mercy_corps' | 'stc'
  lang:  'ES',     // 'ES' | 'EN' | 'FR'
  theme: 'light'   // 'light' | 'dark'
};

function setState(patch) {
  Object.assign(AppState, patch);
  // Aplicar tema al DOM si cambió
  if (patch.theme !== undefined) {
    document.documentElement.setAttribute('data-theme', AppState.theme);
    try { localStorage.setItem('ce-theme', AppState.theme); } catch(e) {}
  }
  // Aplicar org al DOM si cambió
  if (patch.org !== undefined) {
    document.documentElement.setAttribute('data-active-org', AppState.org);
    try { localStorage.setItem('ce-org', AppState.org); } catch(e) {}
  }
  // Guardar idioma
  if (patch.lang !== undefined) {
    try { localStorage.setItem('ce-lang', AppState.lang); } catch(e) {}
  }
  renderApp();
}

// Restaurar estado guardado al cargar
(function restoreState() {
  try {
    const savedTheme = localStorage.getItem('ce-theme');
    const savedOrg   = localStorage.getItem('ce-org');
    const savedLang  = localStorage.getItem('ce-lang');
    if (savedTheme) AppState.theme = savedTheme;
    if (savedOrg && MASTER_DATA[savedOrg]) AppState.org = savedOrg;
    if (savedLang && ['ES','EN','FR'].includes(savedLang)) AppState.lang = savedLang;
  } catch(e) {}
  document.documentElement.setAttribute('data-theme', AppState.theme);
  document.documentElement.setAttribute('data-active-org', AppState.org);
})();
```

**Nota importante:** Las claves de localStorage cambian de `mide-theme`, `conesperanza-lang`, `selected-org` a `ce-theme`, `ce-lang`, `ce-org` para evitar conflictos con la versión anterior.

---

### 8.5. JavaScript — MASTER_DATA (estructura exacta)

```javascript
/* ============================================================
   BLOQUE A: DATOS MAESTROS
   Estructura: MASTER_DATA[org][lang][seccion][campo]
   ============================================================ */
const MASTER_DATA = {

  todos: {
    ES: {
      // Header global (estos campos afectan a elementos fuera de las secciones)
      global: {
        brandSub:     "Reporte de Consorcio",
        subbarFilter: "Filtrar por Organización:",
        orgTodos:     "Todos",
        pbiBtnText:   "Power BI",
        // Drawer móvil
        drawerTitle:     "Ajustes & Acceso",
        drawerLangLabel: "Idioma",
        drawerLangSub:   "Preferencia de idioma",
        drawerThemeLabel:"Tema Visual",
        drawerThemeSub:  "Alternar modo claro / oscuro",
        drawerPbiBtn:    "Abrir Power BI",
      },
      // Sección 1 (#p1 — Hero)
      hero: {
        title: "Seguimiento al POA 2026",
        lead:  "Monitoreo continuo...",
        kpisHTML: `<div class="t">...</div>`, // HTML de los KPIs
        imgSrc: "img_section_1.jpg",
        imgAlt: "Persona en territorio comunitario"
      },
      // Sección 2 (#p2 — Despliegue territorial)
      p2: {
        eyebrow: "Despliegue territorial",
        title:   "Presencia humanitaria en zonas de alta vulnerabilidad",
        lead:    "...",
        kpisHTML: `...`,  // HTML de los 4 KPIs territoriales
        imgSrc: "img_section_3.jpg",
        imgAlt: "Trabajo territorial en zonas comunitarias"
      },
      // Sección 3 (#p3 — Sectores de respuesta) — SIEMPRE 4 tarjetas
      p3: {
        eyebrow:  "Sectores de respuesta",
        title:    "Cuatro líneas de acción...",
        lead:     "...",
        recsHTML: `...`   // HTML de las 4 tarjetas sectoriales (.rec)
      },
      // Sección 4 (#p4 — Voces de la comunidad) — SIEMPRE 4 testimonios
      p4: {
        eyebrow:    "Voces de la comunidad",
        title:      "Evidencia cualitativa...",
        lead:       "...",
        quotesHTML: `...` // HTML de los 4 blockquotes
      },
      // Sección 5 (#p5 — AAP)
      p5: {
        eyebrow:    "Rendición de cuentas (AAP)",
        title:      "Mecanismos de escucha...",
        lead:       "...",
        callout:    "...", // Texto del bloque callout
        imgSrc:     "img_section_5.jpg",
        imgAlt:     "Monitoreo con KoboToolbox"
      },
      // Sección 6 (#p6 — Aprendizaje)
      p6: {
        eyebrow:   "Aprendizaje y adaptación",
        title:     "Lecciones aprendidas...",
        lead:      "...",
        cardsHTML: `...`  // HTML de las 2 tarjetas en split
      },
      // Footer
      footer: {
        col1Title:    "ConEsperanza",
        col1Body:     "...",
        col2Title:    "Navegación",
        col2Links:    [...],  // Array de {label, href}
        col3Title:    "Socias implementadoras",
        col4Title:    "Datos del reporte",
        col4Date:     "Corte: 30 de septiembre de 2026",
        col4PbiLabel: "Abrir informe Power BI"
      }
    },
    EN: { /* misma estructura, textos en inglés */ },
    FR: { /* misma estructura, textos en francés */ }
  },

  care: {
    ES: { /* misma estructura, contenido específico de CARE */ },
    EN: null,  // fallback a ES automáticamente
    FR: null
  },

  irc:         { ES: { /* ... */ }, EN: null, FR: null },
  mercy_corps: { ES: { /* ... */ }, EN: null, FR: null },
  stc:         { ES: { /* ... */ }, EN: null, FR: null }
};
```

---

### 8.6. JavaScript — renderApp() (motor único)

```javascript
/* ============================================================
   BLOQUE C: MOTOR ÚNICO DE RENDER
   ============================================================ */
function renderApp() {
  const { org, lang } = AppState;

  // Fallback: si el idioma no existe para esa socia, usar ES
  const data = (MASTER_DATA[org][lang]) || MASTER_DATA[org]['ES'];
  const g    = data.global;

  // --- HEADER GLOBAL ---
  const brandSub = document.querySelector('.brand-sub');
  if (brandSub) brandSub.textContent = g.brandSub;

  const subbarLabel = document.querySelector('#subbarLabel');
  if (subbarLabel) subbarLabel.textContent = g.subbarFilter;

  const pbiBtnLabel = document.querySelector('#pbiBtnLabel');
  if (pbiBtnLabel) pbiBtnLabel.textContent = g.pbiBtnText;

  // Botones de idioma: marcar activo
  document.querySelectorAll('.lang-btn').forEach(btn => {
    btn.classList.toggle('active', btn.dataset.lang === lang);
  });

  // Tabs de socias: marcar activo
  document.querySelectorAll('[data-org]').forEach(tab => {
    tab.classList.toggle('active', tab.dataset.org === org);
  });

  // Drawer móvil
  const drawerTitle = document.querySelector('#mobileDrawer .drawer-title');
  if (drawerTitle) drawerTitle.textContent = g.drawerTitle;
  // ... resto del drawer

  // --- SECCIÓN 1: HERO (#p1) ---
  const p1 = document.querySelector('#p1');
  if (p1) {
    p1.querySelector('h1').textContent         = data.hero.title;
    p1.querySelector('p.lead').textContent     = data.hero.lead;
    p1.querySelector('.tease').innerHTML       = data.hero.kpisHTML;
    p1.querySelector('.hero-img').src          = data.hero.imgSrc;
    p1.querySelector('.hero-img').alt          = data.hero.imgAlt;
  }

  // --- SECCIÓN 2: TERRITORIAL (#p2) ---
  const p2 = document.querySelector('#p2');
  if (p2) {
    p2.querySelector('.eyebrow').textContent   = data.p2.eyebrow;
    p2.querySelector('h2').textContent         = data.p2.title;
    p2.querySelector('p.lead').textContent     = data.p2.lead;
    p2.querySelector('.tease').innerHTML       = data.p2.kpisHTML;
  }

  // --- SECCIÓN 3: SECTORES (#p3) — SIEMPRE 4 tarjetas ---
  const p3 = document.querySelector('#p3');
  if (p3) {
    p3.querySelector('.eyebrow').textContent   = data.p3.eyebrow;
    p3.querySelector('h2').textContent         = data.p3.title;
    p3.querySelector('p.lead').textContent     = data.p3.lead;
    p3.querySelector('.recs').innerHTML        = data.p3.recsHTML;
  }

  // --- SECCIÓN 4: VOCES (#p4) — SIEMPRE 4 testimonios ---
  const p4 = document.querySelector('#p4');
  if (p4) {
    p4.querySelector('.eyebrow').textContent   = data.p4.eyebrow;
    p4.querySelector('h2').textContent         = data.p4.title;
    p4.querySelector('.quotes-box').innerHTML  = data.p4.quotesHTML;
  }

  // --- SECCIÓN 5: AAP (#p5) ---
  const p5 = document.querySelector('#p5');
  if (p5) {
    p5.querySelector('.eyebrow').textContent   = data.p5.eyebrow;
    p5.querySelector('h2').textContent         = data.p5.title;
    p5.querySelector('p.lead').textContent     = data.p5.lead;
    p5.querySelector('.callout p').textContent = data.p5.callout;
  }

  // --- SECCIÓN 6: APRENDIZAJE (#p6) ---
  const p6 = document.querySelector('#p6');
  if (p6) {
    p6.querySelector('.eyebrow').textContent   = data.p6.eyebrow;
    p6.querySelector('h2').textContent         = data.p6.title;
    p6.querySelector('.split').innerHTML       = data.p6.cardsHTML;
  }

  // --- FOOTER ---
  renderFooter(data.footer);

  // --- RE-RENDERIZAR GRÁFICAS ---
  if (typeof renderAll === 'function') requestAnimationFrame(renderAll);

  // Transición suave
  document.querySelector('main').style.opacity = '1';
}
```

---

### 8.7. JavaScript — Funciones auxiliares críticas

```javascript
// Abrir/cerrar drawer móvil
function toggleDrawer(open) {
  const drawer   = document.getElementById('mobileDrawer');
  const backdrop = document.getElementById('drawerBackdrop');
  const willOpen = open !== undefined ? open : !drawer.classList.contains('open');
  drawer.classList.toggle('open', willOpen);
  backdrop.classList.toggle('open', willOpen);
}

// Power BI (placeholder)
function openPowerBI() {
  alert('Redirigiendo al informe consolidado de Power BI - Consorcio Humanitario');
}
```

---

### 8.8. Qué NO hacer (Reglas de Gemini)

| ❌ PROHIBIDO | ✅ CORRECTO |
|---|---|
| Crear archivos `.js`, `.css` separados | Todo dentro de `index.html` |
| Usar `addEventListener` en el botón de tema | Usar `onclick="setState({theme:...})"` |
| Llamar `renderOrgContent()` o `applyTranslations()` | Solo llamar `setState()` o `renderApp()` |
| Usar `border: 1px solid` para separar secciones | Usar cambio de `background-color` |
| Inventar nombres de variables CSS | Usar los `--` definidos en §8.1 |
| Inventar nombres de clases HTML | Respetar las clases existentes del original |
| Crear más de 1 bloque `<script>` | Un solo `<script>` al final del `<body>` |
| Usar librerías externas (jQuery, Bootstrap, etc.) | Solo JS vanilla |
| Usar Google Fonts via `<link>` | Las fuentes ya están en base64 en el `<style>` |
| Hardcodear textos en el HTML body | Todo texto va en `MASTER_DATA` |
| Eliminar el fallback a ES en socias | El fallback `|| MASTER_DATA[org]['ES']` es obligatorio |

---

### 8.9. Orden de ejecución al iniciar la página

Al cargar la página, el JS debe ejecutarse en este orden exacto:

1. Definir `MASTER_DATA` (los datos).
2. Definir `AppState` con valores default.
3. Definir todas las funciones: `setState`, `renderApp`, `renderFooter`, `toggleDrawer`, `openPowerBI`, funciones de gráficas.
4. Llamar `restoreState()` para leer localStorage.
5. Llamar `renderApp()` una sola vez para pintar el estado inicial.
6. Llamar `renderAll()` para inicializar las gráficas.

```javascript
// Al final del <script>, después de todas las definiciones:
restoreState();
renderApp();
```

---

### 8.10. Breakpoints y responsive

```css
/* Móvil: < 900px — se ocultan controles de escritorio */
@media (max-width: 900px) {
  .top-controls-desktop { display: none; }
  .menu-btn-mobile       { display: flex; }
  .org-subbar-desktop    { display: none; }
  .bottom-nav-mobile     { display: flex; }
}

/* Escritorio: >= 900px */
@media (min-width: 901px) {
  .menu-btn-mobile    { display: none; }
  .bottom-nav-mobile  { display: none; }
  .org-subbar-desktop { display: flex; }
}
```

---

*Sección técnica añadida el 23 de septiembre de 2026 para facilitar la ejecución por Gemini.*
