# Generador de imágenes personalizadas V4.1

Aplicación web estática (HTML5, CSS3 y JavaScript con Canvas API y Web Share API) para generar, previsualizar en tiempo real, descargar y compartir imágenes personalizadas en formato PNG de `1080 × 1080` píxeles con procesamiento íntegramente local en el dispositivo.

---

## 1. Documentación técnica breve

### Arquitectura y separación de responsabilidades

- `index.html`: estructura HTML5 semántica y adaptable a móviles con los 7 controles principales (vista previa en `<canvas>`, selector de fotografía, campo de nombre obligatorio, control de tamaño/zoom, botón de restablecimiento, descarga PNG y compartir).
- `src/styles.css`: hoja de estilos CSS3 móvil primero, sin desplazamiento horizontal, con áreas táctiles mínimas de `46–48 px`, `touch-action: none` sobre el lienzo y estilos de foco accesibles.
- `src/templates/portraitTemplate.js`: módulo de plantilla gráfica inmutable (`PORTRAIT_TEMPLATE` con `Object.freeze`) basado en el diseño oficial (`public/assets/aistudio/file_00000000ce3881f680c1ad6669ecc7e1_496de046.png`). Conserva intactos todos los textos, colores, logotipos, fecha, hora y ubicación fijos, exponiendo únicamente el recorte circular para la fotografía y la pastilla amarilla para el nombre.
- `src/engine/canvasEngine.js`: motor de personalización independiente de la plantilla. Cachea el fondo fijo en un lienzo auxiliar para mantener intactos los elementos gráficos y acelerar el arrastre en móviles; aplica recorte circular, interpolación de alta calidad (`imageSmoothingQuality = 'high'`), límites de desplazamiento sin huecos, ajuste automático de tipografía con normalización `NFC`, restablecimiento de posición/escala y exportación PNG con nombres únicos (`<nombre>_<timestamp>_<secuencia>.png`).
- `src/main.js`: controlador de interfaz. Conecta entradas de usuario, protección contra cargas asíncronas fuera de orden, gestos táctiles de arrastre y pellizco (`Pointer Events`), descarga directa y compartición nativa mediante `navigator.share` / `navigator.canShare({ files })`.

### Privacidad y requisitos sin servidor

- Todo el procesamiento de fotografías y nombres ocurre en memoria dentro del navegador del usuario.
- No utiliza bases de datos, autenticación, Firebase ni servicios externos de procesamiento.

---

## 2. Instrucciones de publicación (sitio estático independiente)

> Nota: No se ha realizado ninguna publicación ni despliegue automático. Ejecutar los pasos siguientes únicamente cuando se autorice la publicación.

### Compilación del paquete estático

1. Instalar dependencias de compilación (si aún no están instaladas):
   ```bash
   npm install
   ```
2. Generar la carpeta estática `dist/` (con rutas relativas `./assets/...` compatibles tanto con raíz como con subdirectorios):
   ```bash
   npm run build
   ```

### Opción A: Netlify

1. Crear un sitio nuevo en Netlify conectado al repositorio o mediante carga manual de la carpeta `dist/`.
2. Si se conecta por repositorio, configurar:
   - **Build command:** `npm run build`
   - **Publish directory:** `dist`
3. Verificar que el sitio sirva por HTTPS (necesario para habilitar la Web Share API en navegadores móviles).

### Opción B: GitHub Pages

1. Publicar el código en un repositorio independiente de GitHub.
2. Ejecutar `npm run build` y publicar el contenido de `dist/` en la rama `gh-pages` o mediante un flujo de GitHub Actions con `actions/deploy-pages` apuntando a `./dist`.
3. Confirmar el acceso por HTTPS en `https://<usuario>.github.io/<repositorio>/`.

---

## 3. Informe de pruebas

### Verificaciones técnicas ejecutadas localmente

| Prueba | Comando / Alcance | Resultado |
| --- | --- | --- |
| Validación estática (`tsc --noEmit`) | `npm run lint` | Aprobado sin errores |
| Compilación estática (`vite build`) | `npm run build` (generación de `dist/` con rutas relativas) | Aprobado sin errores |
| Inmutabilidad y dimensiones de plantilla | `node --test tests/engine.test.mjs` (`PORTRAIT_TEMPLATE` `1080 × 1080`) | Aprobado |
| Normalización de nombres y acentos (`NFC`) | `node --test tests/engine.test.mjs` (`sanitizeFileNamePart`) | Aprobado |
| Ajuste automático de fuente para nombres largos | `node --test tests/engine.test.mjs` (`computeFittedFontSize`) | Aprobado |
| Límites de arrastre y restablecimiento | `node --test tests/engine.test.mjs` (`clampPanOffset`, `resetPhotoTransform`) | Aprobado |
| Bloqueo sin foto/nombre y nombres únicos PNG | `node --test tests/engine.test.mjs` (`isComplete`, `buildUniqueFileName`, `exportPngBlob`) | Aprobado |

### Alcance pendiente de prueba manual en dispositivo físico

- La hoja nativa de compartir (`navigator.share({ files })`) y la apertura directa de la galería dependen del sistema operativo y del contexto HTTPS en dispositivo móvil Android; se recomienda validar el flujo completo sobre la URL HTTPS de prueba en Android Chrome tras esta entrega.
