# Pruebas V0.2

- `npm test`: geometría DXF, ida/vuelta de coordenadas, validación de áreas y sesiones MASTER (6 pruebas superadas en desarrollo).
- Extractor KMZ: probado con ZIP DEFLATE de tres placemarks.
- Los resultados de coordenadas se contrastaron con pyproj/PROJ para controles WGS84 → zonas MAGNA y retorno.
- Pruebas de navegador automatizado: no ejecutadas en este entorno; la política del navegador disponible bloquea la navegación a HTTP local/file. Deben realizarse en Chrome Android, Safari iOS y navegadores de escritorio.
- Pruebas de Netlify Functions + Netlify Blobs reales: **pendientes de desplegar**. No confundir el éxito de pruebas unitarias con una publicación Netlify validada.

## Lista para ensayos en campo

1. En un proyecto Netlify desplegado correctamente, el operador carga sin contraseña; `/.netlify/functions/voladuras` devuelve el objeto oficial (al principio vacío).
2. El MASTER inicia sesión mediante contraseña y publica tres voladuras. Un móvil diferente debe verlas al abrir la URL.
3. Un operador importa DXF local sin modificar la versión oficial. También KML, KMZ y GeoJSON.
4. El operador cambia WGS84 / MAGNA Central / Bogotá Central / Oeste / Este y verifica coordenadas contra control topográfico.
5. El MASTER sustituye el conjunto oficial por un archivo nuevo. El operador visualiza la fecha/versión nueva después de sincronizar.
6. Sin internet, el dispositivo previamente sincronizado muestra datos cacheados y aviso de desactualización.
7. En el borde de los polígonos verificar mensajes de incertidumbre GPS; nunca interpretar como permiso de acceso.
8. Probar recarga de PWA y actualización de service worker con una publicación posterior.
