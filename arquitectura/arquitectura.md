# Arquitectura e Infraestructura del Sistema

## 1. Diagrama de la Arquitectura
El proyecto Evaluario sigue una arquitectura aislada y sin fugas de datos (*Privacy-by-Design*), diseñada para ejecutarse íntegramente en el equipo local del docente sin necesidad de servidores externos ni servicios de pago en la nube.

![Esquema general del sistema](1%20-%20Montaje%20actual.png)

*Archivo editable disponible:* [`Esquema del sistema.drawio`](Esquema%20del%20sistema.drawio)

---

## 2. Flujo de Corrección y Aislamiento

El flujo de trabajo se divide en dos módulos independientes:

1. **Módulo de Redes (Web Offline):** Lee las memorias en PDF entregadas por los alumnos, procesa las capturas de pantalla de terminales mediante OCR en WebAssembly (sin subirlas a internet) y extrae las evidencias de configuración de red (IPs, máscaras, comandos ejecutados).
2. **Módulo de Bases de Datos (Sandbox en Docker):** Levanta un contenedor MySQL aislado con almacenamiento en memoria RAM (`tmpfs`), ejecuta las soluciones de los alumnos y las compara con la solución canónica del profesor.

![Flujo de corrección](2%20-%20Flujo%20de%20correccion.png)

---

## 3. Seguridad y Hardening del Sandbox MySQL

Para evitar riesgos derivados de la ejecución de scripts SQL potencialmente dañinos o mal formateados por parte de los alumnos, el contenedor Docker aplica las siguientes restricciones:

- **Almacenamiento volátil en RAM (`tmpfs`):** El directorio `/var/lib/mysql` reside en la memoria RAM (768 MB). Al reiniciar o detener el contenedor, cualquier cambio destructivo desaparece de inmediato.
- **Sin acceso a ficheros del sistema anfitrión:** `--secure-file-priv=NULL` y `--local-infile=0` impiden la lectura o exportación de archivos mediante sentencias como `LOAD DATA LOCAL INFILE` o `SELECT ... INTO OUTFILE`.
- **Límite de tiempo estricto:** `--max-execution-time=5000` cancela automáticamente cualquier consulta que demore más de 5 segundos, evitando bucles o bloqueos por productos cartesianos descontrolados.
- **Aislamiento de red:** El puerto solo se publica en el bucle local (`127.0.0.1:3307`), siendo inaccesible desde el resto de la red local.

![Seguridad del sandbox](3%20-%20Seguridad%20del%20sandbox.png)

---

## 4. Especificación Técnica de Componentes

| Componente | Función principal | Sistema operativo / Tecnología | Configuración prevista |
|---|---|---|---|
| **Cliente Web Docente** | Gestión de cursos, panel de corrección de memorias y visualización de rúbricas. | Navegador Web (Chrome / Firefox / Edge) · React 19 + TypeScript | SPA empaquetada en un único fichero HTML autónomo. Política estricta de seguridad CSP `connect-src 'none'`. |
| **Motor de OCR Local** | Extracción de texto de capturas de pantalla de terminales en segundo plano. | WebAssembly (Tesseract.js Core) en Web Worker | Preprocesado de imagen con inversión de contraste y reescalado de líneas a ~30 px. |
| **Sandbox de Bases de Datos** | Entorno efímero y seguro para la ejecución de scripts SQL de alumnos. | Linux / Docker Engine (`mysql:8.4`) | Enlace exclusivo a `127.0.0.1:3307`. Datos en memoria volátil `tmpfs` (768 MB). `--secure-file-priv=NULL`, `--local-infile=0` y timeout de 5 segundos. |
| **Motor de Comparación SQL** | Segmentación de sentencias, comparación de estados DDL/DML y generación de casos límite. | Python 3.14 (gestor Astral `uv`) | Librerías: `pymysql`, `smol-toml`. CLI ejecutable localmente sin dependencias globales del sistema. |
| **Almacenamiento Local** | Persistencia de datos de cursos, tareas, rúbricas y copias de seguridad. | IndexedDB nativo + File System Access API | Almacenamiento estructurado en el navegador con opción de copia de seguridad diaria automática en una carpeta local. |
