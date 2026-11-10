# Evaluario · Ingeniería de Pre-Corrección Local para FP

> **Proyecto Intermodular de ASIR (Curso 2026-2027)**  
> **Centro:** I.E.S. Triana (Sevilla)  
> **Alumno:** Sergio  
> **Web de Lanzamiento (GitHub Pages):** [https://piratapacorro.github.io/evaluario-lanzamiento/](https://piratapacorro.github.io/evaluario-lanzamiento/)  
> **Tablero Kanban en Jira:** [https://evaluario.atlassian.net](https://evaluario.atlassian.net)

---

## 1. Nombre del proyecto
**Evaluario**: Plataforma de pre-corrección determinista de prácticas de Redes y Bases de Datos para profesorado de Formación Profesional.

---

## 2. Descripción
Evaluario es una herramienta de ingeniería docente diseñada para automatizar la pre-evaluación de prácticas técnicas en los ciclos formativos de Informática (2.º de ASIR para servicios de red y 1.º para bases de datos). El sistema inspecciona las entregas de los estudiantes (memorias en PDF con capturas de pantalla de máquinas virtuales y scripts `.sql`), verifica cada requisito técnico y propone un borrador de calificación explicado punto por punto, manteniendo siempre al profesor como único decisor de la nota final.

---

## 3. Justificación
En el ciclo de ASIR, la corrección manual de prácticas de redes (configuración de DHCP, subredes, enrutamiento y servicios en VirtualBox) obliga al profesorado a revisar cientos de capturas de terminal para verificar direcciones IP, máscaras, puertas de enlace y ficheros de configuración. De igual modo, en bases de datos es necesario ejecutar y comprobar decenas de scripts SQL por cada tarea.

El uso de modelos de inteligencia artificial comercial en la nube no es viable en un centro educativo por tres motivos críticos:
1. **Privacidad y cumplimiento legal:** El RGPD y el Reglamento Europeo de IA (EU AI Act, Anexo III 3.b) clasifican la evaluación educativa como de Alto Riesgo e impiden enviar datos identificativos o trabajos de alumnos a servidores externos.
2. **Reproducibilidad:** Los modelos generativos sufren alucinaciones y pueden puntuar de forma distinta dos entregas idénticas.
3. **Coste operativo:** Las APIs de pago por token generan un gasto continuo inasumible.

Evaluario resuelve esta necesidad mediante un motor 100% local, determinista, sin conexión y de coste cero bajo arquitectura *Privacy-by-Design*.

---

## 4. Objetivos
- **Reducción superior al 65% en tiempos de corrección:** Transformar horas de lectura repetitiva en una revisión ágil sobre evidencias ya localizadas.
- **Privacidad radical (*Privacy-by-Design*):** Garantizar mediante cabecera de seguridad `connect-src 'none'` que ningún archivo, nombre ni captura de los alumnos abandone el ordenador del docente.
- **Ejecución aislada de código hostil:** Evaluar scripts SQL dentro de un contenedor Docker MySQL 8.4 cuyos datos residen en memoria RAM volátil (`tmpfs`), sin acceso al disco anfitrión.
- **Detección ética de indicios de copia:** Localizar huellas de máquina compartidas entre dos alumnos (direcciones MAC y marcas de tiempo exactas al segundo) filtrando las constantes por defecto de VirtualBox.
- **Trazabilidad completa:** Vincular cada punto propuesto con la página y captura exacta del PDF o con el conjunto de filas devueltas en la base de datos.

---

## 5. Tecnologías utilizadas
- **Infraestructura y Contenedores:** Docker Desktop, Docker Compose, Linux / Debian.
- **Bases de Datos y Sandbox:** MySQL 8.4 Server confinado a `127.0.0.1:3307` sobre volumen `tmpfs` en RAM (768 MB).
- **Motor Backend y Scripting:** Python 3.14 gestionado con Astral `uv`, `pymysql`, `smol-toml`.
- **Aplicación Web Offline-First:** React 19, TypeScript, Vite 8 (`vite-plugin-singlefile`), Radix Themes.
- **Visión Artificial y OCR Local:** Tesseract.js (WebAssembly SIMD) ejecutado en Web Worker con preprocesado de contraste y escalado a 30 px.
- **Persistencia y Cifrado:** IndexedDB nativo, File System Access API (copias automáticas locales) y Web Crypto API (`AES-256-GCM` + `PBKDF2-SHA-256` con 600.000 iteraciones).
- **Control de Versiones y Gestión:** Git, GitHub, GitHub Pages, Jira Software (Kanban).

---

## 6. Funcionalidades principales
1. **Lector Óptico de Memorias de Redes (ASIR / SMR):**
   - Descompresión en memoria del archivo ZIP descargado de Google Classroom.
   - Preprocesamiento de capturas oscuras de terminal y lectura OCR sin internet.
   - Hoja de lectura óptica interactiva con burbujas de estado (confirmado, ausente, por revisar) y salto directo a la página de la evidencia.
2. **Corrector Determinista SQL (DAM / DAW / ASIR):**
   - Separador propio con soporte de `DELIMITER` para procedimientos y triggers.
   - Comparación semántica de esquemas DDL (tablas, tipos compatibles, PK, FK, UNIQUE, CHECK).
   - Evaluación de consultas SELECT y modificaciones DML partiendo siempre del estado correcto de cada ejercicio.
   - Generación automática de datos sintéticos en los valores límite (`datos_extra.sql`) para detectar consultas que aciertan por casualidad.
3. **Calibración por Grupos y Detector Anti-Copia:**
   - Agrupación de alumnos que cometen exactamente el mismo fallo para calibrar la rúbrica (`rubrica.toml`) con una sola respuesta.
   - Aviso privado para el docente cuando dos memorias comparten huellas de red exclusivas.

---

## 7. Planificación (Fases del Proyecto en Jira)
El proyecto se estructura en 6 épicas en el tablero de Jira (`EVA`) desde septiembre de 2026 hasta finales de mayo de 2027:

| Fase / Épica | Periodo | Estado | Tareas principales |
|---|---|---|---|
| **1. Elección de la idea del proyecto** | 22/09/2026 - 03/11/2026 | Finalizado | Lluvia de ideas de ASIR, búsqueda de información técnica y entrega del Anteproyecto. |
| **2. Planificación y organización** | 03/11/2026 - 15/12/2026 | En curso | Configuración del tablero Kanban en Jira, repositorio en GitHub, Landing Page en GitHub Pages y diagramas técnicos. |
| **3. Infraestructura segura en Docker** | 16/12/2026 - 31/01/2027 | Pendiente | Contenedor MySQL 8.4 en RAM (`tmpfs`), bloqueo de ficheros (`--secure-file-priv=NULL`) y aislamiento de red. |
| **4. Desarrollo del motor de corrección** | 01/02/2027 - 20/03/2027 | Pendiente | Script en Python para evaluar ejercicios SQL y comprobador web de memorias de redes con OCR. |
| **5. Pruebas de campo y mejoras** | 21/03/2027 - 30/04/2027 | Pendiente | Pruebas con prácticas reales de clase y ajuste del detector de indicios de copia entre alumnos. |
| **6. Memoria final y defensa FCT** | 01/05/2027 - 28/05/2027 | Pendiente | Redacción de la memoria final del proyecto, diapositivas y preparación de la defensa ante el tribunal. |

---

## 8. Estado actual del proyecto
El proyecto se encuentra en la **Fase 3 del curso (Planificación y Organización)**. El núcleo funcional de Evaluario cuenta ya con el sandbox de MySQL operativo, el lector óptico funcional y una batería de pruebas automatizadas en verde para garantizar la fiabilidad antes de las pruebas piloto con profesorado real.

---

## 9. Diseño técnico
Los diagramas de infraestructura, flujos de datos, archivos editables y prototipos de interfaz están disponibles en este repositorio:
- [Especificación de Arquitectura e Infraestructura (Markdown)](arquitectura/arquitectura.md)
- [Visor Web de Arquitectura e Infraestructura (HTML)](https://piratapacorro.github.io/evaluario-lanzamiento/arquitectura/arquitectura.html)
- [Diagrama General del Montaje Actual (PNG)](arquitectura/1%20-%20Montaje%20actual.png)
- [Diagrama del Flujo de Corrección (PNG)](arquitectura/2%20-%20Flujo%20de%20correccion.png)
- [Diagrama de Seguridad del Sandbox Docker (PNG)](arquitectura/3%20-%20Seguridad%20del%20sandbox.png)
- [Archivo Editable de Diagramas (Draw.io)](arquitectura/Esquema%20del%20sistema.drawio)
- [Prototipos de Interfaz de Usuario (Markdown)](prototipos/prototipos.md)
- [Visor Web de Prototipos de Interfaz (HTML)](https://piratapacorro.github.io/evaluario-lanzamiento/prototipos/prototipos.html)

---

## 10. Enlaces del proyecto
- **Landing Page Oficial de Lanzamiento (GitHub Pages):** [https://piratapacorro.github.io/evaluario-lanzamiento/](https://piratapacorro.github.io/evaluario-lanzamiento/)
- **Tablero Kanban en Jira (Proyecto EVA):** [https://evaluario.atlassian.net](https://evaluario.atlassian.net)
- **Repositorio Público del Proyecto:** [https://github.com/Piratapacorro/evaluario-lanzamiento](https://github.com/Piratapacorro/evaluario-lanzamiento)
