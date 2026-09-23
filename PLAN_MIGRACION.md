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

*Generado el 23 de septiembre de 2026 en sesión de arquitectura con el usuario. Actualizado con inventario de assets.*
