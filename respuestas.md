# Preguntas finales — Deber 6

### 1. ¿Por qué dos especialistas y no uno solo? ¿Qué mejora y qué costo agrega?
* **Por qué:** Porque dividir responsabilidades especializa las instrucciones y evita sobrecargar el contexto del modelo con reglas contradictorias (por ejemplo, separar el análisis de seguridad estricto de la orquestación general).
* **Mejora:** Mayor precisión, mejor aislamiento de tareas y cumplimiento de restricciones de seguridad.
* **Costo:** Agrega complejidad de coordinación y mayor overhead en la gestión de archivos de configuración.

*(Vista general del flujo del equipo y agentes especialistas):*
![Flujo del equipo](evidencias/parte1_flujo.png)

---

### 2. ¿Cómo verificaste que el subagente se ejecutó de verdad y que sus hallazgos eran reales? ¿Qué harías si fallara?
* **Verificación:** Revisando el output estructurado en la consola con los hallazgos detallados sobre `usuarios.py`, y comprobando que el archivo original no fue alterado indebidamente.
* **Si fallara:** Revisaría la sintaxis del archivo TOML de configuración y los permisos de ejecución.

*(Evidencia de la invocación y ejecución real del subagente):*
![Subagente ejecutándose](evidencias/parte2_subagente.png)

![Llamada al subagente](evidencias/parte2_llamada.png)

---

### 3. ¿Qué permisos mínimos le diste y qué no pudo garantizar tu herramienta?
* **Permisos mínimos:** Se configuró el subagente para utilizar exclusivamente operaciones de lectura y búsqueda (`workspace-read`), deshabilitando la escritura y ejecución directa.
* **Lo que no pudo garantizar:** El control automático de permisos dinámicos entre los modos de la sesión, requiriendo directrices estrictas en el TOML.

---

### 4. ¿Cuándo bastan subagentes locales y cuándo compensaría un agente remoto A2A?
* **Subagentes locales:** Bastan cuando la tarea se ejecuta dentro del mismo entorno de desarrollo y workspace de forma síncrona.
* **Agente remoto A2A:** Compensaría cuando se requiere integrar sistemas desacoplados que corren en servidores externos o arquitecturas distribuidas.

*(Diseño conceptual de la comunicación A2A):*
![Diseño A2A](evidencias/parte3_a2a.png)

---

### 5. ¿Cómo manejaría tu cliente `input-required` o un fallo de la Task?
* **`input-required`:** El orquestador local pausaría el flujo de trabajo para solicitar al usuario los datos o aprobaciones faltantes antes de continuar.
* **Fallo de la Task (`failed`):** El cliente capturaría la excepción, evaluaría un reintento o escalaría un reporte para intervención humana.

---

### 6. ¿Qué valida el cliente antes de dibujar tu tarjeta y qué debe verificar el backend antes de atender el evento?
* **El cliente valida:** Que el esquema UI recibido coincida con los componentes permitidos y que los datos estén sanitizados.
* **El backend valida:** La autenticación y autorización del usuario que emite el evento y la integridad de los parámetros.

*(Diseño de la interfaz generativa A2UI):*
![Interfaz A2UI](evidencias/parte4_interfaz.png)

---

### 7. ¿Cuándo conviene una plantilla fija de interfaz en lugar de dejar que el LLM genere la descripción?
Conviene una plantilla fija cuando la estructura visual es crítica, repetitiva o requiere un diseño estricto y predecible (como formularios de pago o paneles de control) donde un componente generado dinámicamente por el LLM podría introducir inconsistencias visuales.