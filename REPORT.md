# Reporte Técnico — CVE-2026-31431 "Copy Fail"
**Estudiante:** Jonathan E. Tito O.
**Fecha:** $(date)

## 1. ¿Cuál es el bug raíz y en qué archivo/función está?

El bug raíz se encuentra en el archivo `crypto/algif_aead.c`, específicamente en la función `_aead_recvmsg()`. En 2017 se introdujo una optimización que encadenaba el TX SGL (scatterlist de transmisión) al final del RX SGL (scatterlist de recepción) usando `sg_chain()`, y luego asignaba `req->src = req->dst`. Esto significa que el mismo scatterlist se usaba tanto como fuente como destino de la operación criptográfica, poniendo páginas del page cache en un scatterlist de escritura.

## 2. ¿Por qué el write a dst[assoclen + cryptlen] es peligroso?

Cuando `req->src == req->dst`, la operación AEAD escribe el tag de autenticación en `dst[assoclen + cryptlen]`. Si ese destino apunta a páginas del page cache de un binario como `/usr/bin/su` (que es setuid-root), el kernel está escribiendo directamente en la memoria caché de ese archivo. Esto permite modificar el comportamiento del binario en memoria sin tocar el disco, lo que es extremadamente peligroso porque permite escalada de privilegios sin dejar rastro en el sistema de archivos.

## 3. ¿Por qué el exploit es "stealthy"?

El exploit es silencioso porque opera exclusivamente en el **page cache** del kernel — la memoria RAM donde el kernel guarda copias de archivos para acceso rápido. El archivo `/usr/bin/su` en disco permanece intacto; solo su versión en memoria es modificada. Por eso herramientas como `sha256sum` aplicadas al archivo en disco no detectan nada anormal, y el archivo se ve normal en el sistema de archivos. Al reiniciar el sistema, el page cache se limpia y el binario vuelve a su estado original.

## 4. Conexión con conceptos de clase

- **Page cache:** El kernel mantiene en RAM copias de archivos para acceso rápido. CVE-2026-31431 escribe directamente en esta caché sin pasar por el sistema de archivos.
- **setuid:** El bit setuid en `/usr/bin/su` hace que el binario se ejecute con los privilegios del dueño (root). Al corromper su page cache, cualquier usuario puede obtener una shell root.
- **inodos:** El inodo del archivo en disco no se modifica — solo la página en memoria. Esto confirma que el ataque es completamente en memoria.
- **chmod:** El exploit no necesita cambiar permisos; aprovecha los permisos ya existentes del binario setuid.

## 5. ¿Qué aprendiste?

CVE-2026-31431 demuestra cómo múltiples decisiones técnicas razonables pueden combinarse para crear una vulnerabilidad grave. La optimización de 2017 era lógica en aislamiento: reutilizar el mismo scatterlist ahorra memoria y ciclos de CPU. Pero al combinarse con la capacidad de AF_ALG de operar sobre archivos del page cache, y con la existencia de binarios setuid, creó un camino directo de escalada de privilegios. Esta vulnerabilidad enseña que la seguridad debe analizarse de forma sistémica, no componente por componente.
