# ADR-010 — Docker y dependencias

## Estado
Aceptado.

## Decisión
PHP, Composer, Node y MySQL se ejecutan dentro de contenedores. `vendor` y `node_modules` se gestionan como volúmenes con nombre.

## Consecuencia
El entorno del evaluador no depende de instalaciones locales y el compose mantiene la base MySQL vacía antes de ejecutar las migraciones.
