# BlastMap Web V0.2 — áreas oficiales MASTER + capas personales

Aplicación web progresiva para ver en teléfono o PC las áreas de voladuras de una mina. Las geometrías **oficiales** se publican por un MASTER autenticado; operadores que abren el enlace reciben automáticamente la última publicación sin necesidad de contraseña. Cada operador puede importar archivos **personales** y cambiar el sistema de coordenadas de lectura, sin modificar las áreas oficiales.

## Funcionalidad realizada

- Visor canvas de polígonos, cuadrícula MAGNA Bogotá, zoom táctil/ratón, búsqueda, medición, GPS del navegador y simulación de posiciones para ensayos.
- Importación local de **DXF ASCII** (LWPOLYLINE/POLYLINE cerradas, con arcos bulge), **KML**, **KMZ** y **GeoJSON** (Polygon/MultiPolygon sin huecos). Entidades abiertas, huecos y geometrías no compatibles se advierten/omiten, **nunca se rellena un contorno abierto**.
- Selección por nombre, sin códigos EPSG visibles, de **WGS84, MAGNA Central, MAGNA Bogotá Central, MAGNA Oeste, MAGNA Este, MAGNA Oeste-Oeste y MAGNA Este-Este**. MAGNA Central y MAGNA Bogotá Central son alias del mismo origen (EPSG:3116). Otros códigos internos: EPSG:3115 (Oeste), EPSG:3117 (Este), EPSG:3114 (Oeste-Oeste), EPSG:3118 (Este-Este). WGS84 EPSG:4326. Los operadores seleccionan sistema de **origen DXF** en Importar; seleccionan sistema de **visualización** en Ajustes (no altera áreas).
- La geometría normalizada se almacena internamente en MAGNA Bogotá Central (metros) y se preserva Z sin conversión vertical. KML/KMZ/GeoJSON geográficos siempre usan [longitud,latitud], WGS84.
- Operador: fuentes personales en IndexedDB. MASTER: elige entre esas fuentes y publica el **conjunto completo** oficial, que reemplaza al anterior.
- Backend Netlify Functions y Netlify Blobs site-scoped, autenticación de MASTER con hash scrypt y cookie firmada HttpOnly + SameSite Strict; comprobación Origin para POST; bloqueo de escritura desde vistas previas si Netlify expone URL canonical. Respaldo de la publicación anterior y comprobación básica de versión optimista (no transaccional).
- GET oficial sin credenciales, lectura cacheada para consulta offline en IndexedDB. Refresco al abrir, al regresar a la pestaña o cada **120 segundos** mientras la pestaña está visible (no hay push en tiempo real).
- PWA con service worker **sin cachear la API**, con shell básico offline. El mapa es un **plano local vectorial**, no incluye ortofotos ni mapas satelitales.

## Lo que NO está incluido

SHP, GPX, GeoPDF, GeoTIFF, DXF binario, bloques INSERT, HATCH/SPLINE/REGION, polígonos con huecos, sincronización en segundo plano con la pantalla apagada, publicación automática de archivos importados, roles adicionales, aprobación de trabajos de voladura, ni mapas base satelitales. Añadirlos después de pruebas en campo. Los navegadores pueden borrar datos offline, respaldar siempre.

## Implementar en tu sitio Netlify existente

**IMPORTANTE: Netlify Drop (arrastrar el HTML o ZIP a la página de Drop) solamente publica archivos estáticos. NO instala ni despliega Netlify Functions ni Blobs para este flujo.** Para que funcione MASTER + sincronización central, debes desplegar **la carpeta completa del proyecto** mediante un repositorio Git conectado a Netlify o Netlify CLI con build.

### Método recomendado: repositorio Git + Netlify

1. Descomprime `BlastMap_Web_V0_2.zip` y sube su contenido como raíz de un repositorio privado de GitHub/GitLab; mantén `public/`, `netlify/functions/`, `package.json` y `netlify.toml`.
2. En Netlify, crea un sitio desde ese repositorio o cambia la fuente de despliegue del sitio existente a Git. En la configuración de build usa **Publish directory:** `public`; **Functions directory:** `netlify/functions`; Node 22. `netlify.toml` ya incluye estas rutas. Instala las dependencias npm; Netlify realiza la instalación durante el build cuando detecta el proyecto Node.
3. Genera los valores secretos en un equipo con Node 20+:
   `node scripts/create-master-secret.mjs "TuClavePersonalLargaDe14OMasCaracteres"`
   Usa una contraseña única; la indicada es solo un marcador. La salida produce `BLASTMAP_MASTER_HASH` y `BLASTMAP_SESSION_SECRET`. **NO subas los valores a Git ni los compartas.** La contraseña pasada como argumento podría quedar en el historial del terminal; elimina la entrada sensible o ejecuta el comando en un entorno controlado.
4. En Netlify → Site configuration → Environment variables, crea esas dos variables en un scope que **incluya Functions**, no en `netlify.toml` ni en `public/`. Si utilizas dominio personalizado, agrega opcionalmente `BLASTMAP_PUBLIC_ORIGIN` con el origen completo del dominio administrador (por ejemplo, `https://mapas.ejemplo.com`). Si no, usa el origen principal que Netlify expone como `URL`.
5. Vuelve a desplegar el sitio para que las Functions reciban las variables. Verifica desde la URL pública `/.netlify/functions/master` (respuesta JSON con `configured:true`) y `/.netlify/functions/voladuras` (al principio, `areas:[]`). **Si estas rutas devuelven 404 o HTML, las funciones no están desplegadas; subir solamente `public/` no es suficiente.**
6. En el mismo enlace de BlastMap, abre la pestaña **MASTER**, inicia sesión, importa un DXF/KML/KMZ/GeoJSON desde **Importar**, regresa a MASTER, marca qué fuentes publicar y pulsa **Publicar conjunto oficial**. El MASTER reemplaza la publicación completa.
7. Abre el mismo enlace en otro celular, sin iniciar sesión. Las áreas oficiales deberían cargarse al abrir. Si la pantalla está activa, verifica una actualización después de refrescar o en el ciclo de 120 segundos. Las importaciones personales siguen siendo locales.

### Alternativa con Netlify CLI

Desde la raíz del proyecto, con Node.js, conexión a internet, acceso al sitio Netlify y credenciales configuradas:

```bash
npm install
npx netlify-cli login
npx netlify-cli link
npx netlify-cli deploy --build --prod
```

Si ya está asociado a un sitio, `link` permite seleccionar el sitio existente. No compartas tokens de Netlify. Asegura que la publicación realmente incluya las funciones antes de usar el MASTER.

## Pruebas locales

```bash
npm test
npx netlify-cli dev
```

`netlify dev` simula Functions en `localhost` y necesita acceso Netlify/Blobs para almacenamiento central. La interfaz sola puede abrirse mediante un servidor HTTP estático para probar importaciones, pero la publicación MASTER no funciona si no hay Functions.

Archivos de demostración en `public/`:

- `EJEMPLO_3_VOLADURAS_EPSG3116.dxf` — geometría CAD en MAGNA Bogotá Central.
- `EJEMPLO_3_VOLADURAS_WGS84.kml` — tres polígonos geográficos.
- `EJEMPLO_3_VOLADURAS_WGS84.kmz` — el KML anterior comprimido.
- `EJEMPLO_3_VOLADURAS_WGS84.geojson` — mismos polígonos como GeoJSON.

## Reglas de operación y privacidad

- **El enlace operador es público:** cualquier persona con la URL puede consultar las áreas oficiales. No publiques datos sensibles sin autorización de la mina.
- El MASTER tiene contraseña y los operadores no pueden publicar, pero pueden importar capas personales y cambiar la visualización local.
- Únicamente se consideran oficiales las áreas publicadas por MASTER. Los archivos locales no se envían al servidor salvo que el MASTER los elija y publique.
- El importador no puede determinar con certeza el CRS de un DXF: es fundamental seleccionarlo correctamente y comprobar al menos un par de puntos de control antes de usar GPS.
- El GPS de un teléfono puede errar varios metros: un valor dentro/cerca del borde no acredita condiciones de acceso o seguridad. No utilizar el visor para autorización de trabajos de voladura, evacuaciones o control de exclusiones.
- Los operadores desconectados ven el último conjunto oficial guardado (puede estar obsoleto). La interfaz avisa cuando no puede sincronizar.
- Netlify Blobs es almacenamiento objeto y no una base de datos transaccional. Para producción crítica con múltiples MASTER concurrentes se recomienda una base de datos transaccional, revisión/aprobación y auditoría de cambios. V0.2 presupone un único MASTER publicando a la vez.
- Los mapas de fondo satelital y sus licencias no están incluidos. La capa vectorial de la aplicación y las áreas cacheadas pueden mostrarse offline después de una carga previa; no está garantizado almacenamiento permanente en navegador.

## Carpetas

- `public/` — cliente web, mapas, importador, PWA y archivos de prueba.
- `netlify/functions/master.mjs` — inicio/cierre de sesión MASTER.
- `netlify/functions/voladuras.mjs` — lectura pública y publicación autenticada.
- `netlify/functions/security.mjs` — criptografía, sesiones, protección de origen.
- `netlify/functions/validation.mjs` — validación de polígonos normalizados.
- `scripts/create-master-secret.mjs` — generar secretos por instalación.
- `tests/` — pruebas automáticas de geometría y autenticación.

**Estado:** proyecto fuente e importadores probados localmente. Requiere despliegue real en Netlify, ajuste de secretos y validación de GPS + sincronización entre dispositivos y DXF mineros auténticos. No se entrega un sitio público publicado ni se afirma haberlo probado en Netlify.
