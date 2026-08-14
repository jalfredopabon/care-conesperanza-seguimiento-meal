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
