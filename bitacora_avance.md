# Bitácora de Avance y Flujo de Despliegue Git

🔍 **Explicación de la estructura del repositorio:**

Tu repositorio en GitHub utiliza dos ramas (brazos / "branches"):

- **`main`**: La rama principal donde guardas los cambios de código del proyecto.
- **`gh-pages`**: La rama especial desde la cual GitHub Pages publica el sitio en vivo en el enlace: [https://jalfredopabon.github.io/care-conesperanza-seguimiento-meal/](https://jalfredopabon.github.io/care-conesperanza-seguimiento-meal/).

**El problema identificado:**  
Cuando hacíamos un `git push` a `main`, el sitio en vivo no se actualizaba automáticamente porque GitHub Pages está configurado para leer directamente de la rama `gh-pages`, la cual se quedaba desactualizada sin los nuevos commits de `main`.

---

🛠️ **Solución y Procedimiento Estándar para Despliegue:**

Cada vez que se realicen cambios en la rama `main` y se requiera actualizar el sitio web en vivo, cualquier agente o desarrollador debe seguir estos comandos:

```bash
# 1. Hacer commit y push en la rama main
git checkout main
git add .
git commit -m "descripción de los cambios"
git push origin main

# 2. Sincronizar y publicar en la rama gh-pages (Sitio en vivo)
git checkout gh-pages
git merge main
git push origin gh-pages

# 3. Regresar a la rama main de trabajo
git checkout main
```

---

📌 **Pendiente de revisar: Análisis del Gráfico de Líneas (Sección 2)**

📊 **Análisis de los 5 Ejemplos de Gráficos de Líneas:**

- **Imagen 1 (Processing Time - Curvas Suaves Orgánicas + Filtros Superiores):**  
  Lo destacable: Selector de rango/filtro superior muy limpio en cápsulas (All, 1 year, 3 months, 1 month, 1 week), líneas con curvas suavizadas (smooth splines) muy fluidas, y leyenda inferior con promedios (Avg. 10.23 ms).

- **Imagen 2 (Sales Performance - Curvas suaves en dorado con Glow sutil + Tooltip flotante oscuro):**  
  Lo destacable: El punto activo resaltado en dorado con un Tooltip flotante en negro elegante (Jan 17, 2026 - $9,200) y la línea secundaria punteada.

- **Imagen 3 (Rigetti Computing - Línea de área limpia con marcador de pico):**  
  Lo destacable: Identificador de punto máximo destacado (High 3,921W) con una línea de guía vertical que baja hasta el eje X.

- **Imagen 4 (Income Analytics - Curvas de trazado delgado con indicador de clic y tarjeta flotante):**  
  Lo destacable: Tarjeta flotante blanca de detalle con porcentaje de variación en verde ($150.00 ▲ 30%).

- **Imagen 5 (Overview - Tarjetas de métrica integradas con barra/línea inferior):**  
  Lo destacable: Integración de la cifra numérica directamente sobre los puntos de control del gráfico.

💡 **Propuesta de Combinación de Buenas Prácticas:**

Podemos extraer lo mejor de estos 5 ejemplos para aplicarlo a nuestro gráfico "Ritmo de Atención Mensual":

1. **Filtros Superiores Integrados (Inspirado en Imagen 1 y 2):**  
   Colocar un selector discreto en cápsulas en la esquina superior derecha del gráfico para filtrar la serie por rango: `[Todos | Últimos 3 Meses | Último Mes]`.

2. **Líneas con Curvas Suaves Spline (Inspirado en Imagen 1 y 4):**  
   Reemplazar los trazos rectos/angulares por curvas suaves redondeadas (líneas Bezier fluidas), manteniendo la línea sólida azul para el período actual y la línea punteada gris para el período anterior.

3. **Tarjeta Tooltip Flotante Negra (Inspirado en Imagen 2 y 4):**  
   Al pasar el cursor sobre cualquier punto, mostrar un contenedor oscuro redondeado con el dato exacto de la semana y el porcentaje de cambio (`▲ +28%`).

4. **Indicador de Valor Máximo / Pico (Inspirado en Imagen 3):**  
   Destacar el punto con la atención más alta alcanzada en el período con una pequeña etiqueta o pin tenue.

