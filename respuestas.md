# Preguntas finales — Deber 6

### 1. ¿Por qué dos especialistas y no uno solo? ¿Qué mejora y qué costo agrega?
* **Por qué:** Porque dividir responsabilidades especializa las instrucciones y evita sobrecargar el contexto del modelo con reglas contradictorias (por ejemplo, separar el análisis de seguridad estricto de la orquestación general).
* **Mejora:** Mayor precisión, mejor aislamiento de tareas y cumplimiento de restricciones de seguridad (como restringir al revisor de seguridad a solo lectura).
* **Costo:** Agrega complejidad de coordinación y mayor overhead en la gestión de archivos de configuración y llamadas.

### 2. ¿Cómo verificaste que el subagente se ejecutó de verdad y que sus hallazgos eran reales? ¿Qué harías si fallara?
* **Verificación:** Revisando el output estructurado en la consola de Codex con los hallazgos detallados por línea, problema y severidad sobre `usuarios.py`, y comprobando mediante `git status` que el archivo de código original no fue alterado.
* **Si fallara:** Revisaría la sintaxis del archivo TOML de configuración, verificaría los permisos del entorno de ejecución y consultaría los logs de error de la sesión para corregir las restricciones.

### 3. ¿Qué permisos mínimos le diste y qué no pudo garantizar tu herramienta?
* **Permisos mínimos:** Se configuró el subagente para utilizar exclusivamente operaciones de lectura y búsqueda sobre el archivo indicado, deshabilitando la ejecución de código, accesos de red y modificaciones (`workspace-write`).
* **Lo que no pudo garantizar de forma nativa:** El control automático de permisos dinámicos entre los modos de la sesión, lo que requirió adaptar explícitamente las directrices del archivo TOML para evitar bloqueos por herencia de estado `read-only`.

### 4. ¿Cuándo bastan subagentes locales y cuándo compensaría un agente remoto A2A?
* **Subagentes locales:** Bastan cuando la tarea se ejecuta dentro del mismo entorno de desarrollo, compartiendo el mismo workspace y herramientas locales de forma síncrona.
* **Agente remoto A2A:** Compensaría cuando se requiere integrar sistemas desacoplados que corren en servidores de terceros, arquitecturas de microservicios distribuidos o cuando se necesita delegar capacidades especializadas expuestas mediante protocolos de red independientes.

### 5. ¿Cómo manejaría tu cliente `input-required` o un fallo de la Task?
* **`input-required`:** El orquestador local pausaría el flujo de trabajo y levantaría una alerta interactiva o un componente en la interfaz para solicitar al usuario los datos faltantes o credenciales requeridas antes de continuar.
* **Fallo de la Task (`failed`):** El cliente capturaría la excepción devuelta por el agente remoto, evaluaría si el error es transitorio para reintentar la solicitud o escalaría un reporte de fallo para la intervención humana.

### 6. ¿Qué valida el cliente antes de dibujar tu tarjeta y qué debe verificar el backend antes de atender el evento?
* **El cliente valida:** Que el esquema UI recibido coincida con los componentes permitidos, que la estructura JSON sea válida y que los datos estén sanitizados para prevenir inyecciones visuales o fallos de renderizado.
* **El backend valida:** La autenticación y autorización del usuario que emite el evento, la integridad de los parámetros enviados desde la interfaz y que la acción solicitada esté permitida dentro de los límites de seguridad del sistema.

### 7. ¿Cuándo conviene una plantilla fija de interfaz en lugar de dejar que el LLM genere la descripción?
Conviene una plantilla fija cuando la estructura visual es crítica, repetitiva o requiere un diseño estricto y predecible (como formularios de pago, paneles de control corporativos o alertas del sistema) donde un componente generado dinámicamente por el LLM podría introducir inconsistencias visuales o errores de renderizado.