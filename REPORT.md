# Reporte Técnico — CVE-2026-31431 "Copy Fail"

**Estudiante:** Snto666  
**Fecha:** 2026-05-16  

## 1. Bug raíz y ubicación

El bug se encuentra en `crypto/algif_aead.c`, función `_aead_recvmsg()`. En 2017 se introdujo una optimización que usaba `sg_chain()` para encadenar el TX SGL al RX SGL, y luego asignaba `req->src = req->dst`. Esto hace que el mismo scatterlist sea fuente y destino de la operación criptográfica.

## 2. Por qué el write a dst[assoclen + cryptlen] es peligroso

Cuando src == dst, la operación AEAD escribe el tag de autenticación en `dst[assoclen + cryptlen]`. Si ese destino apunta a páginas del page cache de un binario setuid como `/usr/bin/su`, el kernel escribe directamente en memoria del binario con privilegios de kernel, sin ninguna verificación de permisos. Esto permite modificar el comportamiento del binario para obtener root.

## 3. Por qué el exploit es "stealthy"

El exploit opera exclusivamente en el page cache del kernel — la copia en RAM del archivo. El archivo en disco permanece intacto. Herramientas como `sha256sum` o `ls -la` no detectan nada anormal porque leen el disco, no el page cache. Al reiniciar, el page cache se limpia y todo vuelve a la normalidad, sin dejar rastro.

## 4. Conexión con conceptos de clase

- **Page cache:** El kernel mantiene copias de archivos en RAM. CVE-2026-31431 escribe en esta caché sin pasar por el sistema de archivos ni sus permisos.
- **setuid:** El bit setuid en `/usr/bin/su` hace que se ejecute como root. Al corromper su page cache, cualquier usuario obtiene shell root.
- **inodos:** El inodo del archivo en disco no se modifica. Solo la página en RAM cambia, por eso el ataque es invisible al sistema de archivos.
- **chmod:** El exploit no necesita cambiar permisos. Aprovecha los permisos ya existentes del binario setuid para escalar privilegios.

## 5. Lección aprendida

CVE-2026-31431 demuestra cómo decisiones técnicas razonables en aislamiento pueden combinarse para crear vulnerabilidades graves. La optimización de 2017 era lógica: reutilizar el mismo scatterlist ahorra memoria y CPU. Pero combinada con AF_ALG (que permite operaciones crypto sobre archivos del page cache) y binarios setuid, creó un camino directo de escalada de privilegios que funcionó sin modificaciones en Ubuntu, RHEL, Amazon Linux y SUSE durante 9 años. La seguridad debe analizarse de forma sistémica, no componente por componente.
