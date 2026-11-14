
| Agente | Misión | Herramientas autorizadas | Entrada | Salida | Límites |
|---|---|---|---|---|---|
| Orquestador | repartir la revisión y consolidar | ninguna propia (solo delega) | ruta del archivo | tabla con hallazgos y casos de prueba | no modifica archivos |
| Especialista 1 — Seguridad | buscar claves escritas en el código y entradas sin validar | leer, buscar | ruta del archivo | lista: línea, problema, severidad | solo ese archivo; no escribe |
| Especialista 2 — Pruebas | proponer 3 casos de prueba que faltan | leer | ruta del archivo | lista: nombre del caso y qué verifica | solo propone; no escribe pruebas |

## Flujo del equipo
Patrón utilizado: **Orquestador + Paralelo** (los especialistas analizan el archivo al mismo tiempo sin depender el uno del otro).

1. **Inicio:** El usuario entrega la ruta de `usuarios.py` al **Orquestador**.
2. **Delegación paralela:** El Orquestador le envía simultáneamente el archivo a:
   - **Especialista 1 (Seguridad)**
   - **Especialista 2 (Pruebas)**
3. **Consolidación:** El Orquestador recibe los resultados de ambos y arma una tabla unificada.
4. 🛑 **Checkpoint humano (Human-in-the-loop):** El proceso se detiene para que una persona revise y apruebe el resultado antes de proceder a la ejecución.

## Criterios de validación humana (Checkpoint)
- **Qué mira la persona antes de aprobar:** 
  - Verifica que los hallazgos de seguridad reportados por el Especialista 1 existan realmente en las líneas indicadas de `usuarios.py` (evitando falsos positivos).
  - Revisa que los 3 casos de prueba propuestos por el Especialista 2 tengan sentido lógico para la aplicación.
- **Qué la haría rechazar:** 
  - Un hallazgo inventado o un número de línea erróneo que no coincide con el código.
  - Casos de prueba genéricos que no evalúan el comportamiento real del archivo.

## Detección de fallos y manejo de excepciones
- **Cómo detectas que un especialista falló:** 
  - Devuelve una salida vacía.
  - Alucina o inventa líneas de código que no existen en `usuarios.py`.
  - Intenta modificar el archivo violando sus límites de permisos.
- **Qué hace el orquestador:** 
  - Detecta la anomalía en la estructura de la respuesta o el error de límite.
  - Reintenta la solicitud corrigiendo el prompt (pide una corrección específica) o, si el fallo persiste, detiene el flujo y lanza una alerta a la persona (checkpoint humano) para que intervenga.

## A2A

### Secuencia de comunicación A2A:
1. **Descubrimiento:** El orquestador local consulta la URL del agente remoto para obtener su `Agent Card` (usualmente mediante el endpoint `/.well-known/agent-card.json`).

```
{
  "name": "agente-pruebas-remoto",
  "description": "Agente autónomo externo experto en generar casos de prueba para código Python",
  "version": "1.0.0",
  "supportedInterfaces": [{
    "url": "https://agente-pruebas.ejemplo/a2a",
    "protocolBinding": "JSONRPC",
    "protocolVersion": "1.0"
  }],
  "capabilities": { "streaming": true },
  "skills": [{ 
    "id": "proponer-pruebas", 
    "name": "Generación de Casos de Prueba", 
    "description": "Propone casos de prueba faltantes para un archivo de código", 
    "tags": ["testing", "qa"] 
  }],
  "securitySchemes": { "auth": "Bearer token o mTLS requerido" },
  "defaultInputModes": ["text/plain"],
  "defaultOutputModes": ["text/plain"]
}
```


> Simplificación didáctica; no es JSON conforme a una versión específica de A2A.


2. **Validación:** El cliente verifica que el agente cumpla con la skill necesaria, el protocolo de red compatible y los esquemas de seguridad.
3. **Envío de solicitud:** Se despacha la petición al endpoint del agente remoto especificando la tarea y el contexto.
4. **Ciclo de vida de la tarea:** 
   - El agente remoto recibe la orden y la coloca en estado `submitted`.
   - Comienza el procesamiento (`working`).
   - Al finalizar, emite un resultado o artefacto y pasa a estado `completed`.
5. **Recepción:** El orquestador local procesa la respuesta devuelta por el agente remoto para continuar con el flujo del equipo.

## A2UI

### 1. Propósito de la interfaz generativa en el equipo
Permitir que el orquestador y los subagentes no solo devuelvan texto plano en la consola, sino que dinámicamente rendericen componentes visuales interactivos (tarjetas, tablas de severidad, botones de aprobación y gráficos) según el contexto de la tarea ejecutada.

### 2. Ciclo de vida y flujo de renderizado (A2UI)
1. **Intención de UI:** El agente detecta que el resultado de una tarea (por ejemplo, los hallazgos de seguridad de `usuarios.py`) requiere una visualización estructurada.
2. **Generación del Schema:** El agente emite un payload estructurado en formato JSON (UI Schema) que describe los componentes a renderizar (ej. `Card`, `Table`, `Button`).
3. **Validación y Sanitización:** El cliente o contenedor de la aplicación valida que el esquema cumpla con los componentes permitidos por motivos de seguridad.
4. **Renderizado en Cliente:** La interfaz del usuario interpreta el esquema y genera nativamente los elementos visuales interactivos (como la tabla de hallazgos con severidades resaltadas).
5. **Acción de Usuario:** El usuario interactúa con un componente (ej. hace clic en "Aplicar parche sugerido" o "Descartar falso positivo"), enviando un evento de vuelta al agente.

### 3. Componentes interactivos clave
* **Tabla de Hallazgos:** Muestra las columnas de línea, problema y severidad con etiquetas codificadas por colores (Crítica/Alta en rojo, Media/Baja en amarillo/verde).
* **Botones de Checkpoint (Humano en el bucle):** Permiten pausar la ejecución para que el usuario apruebe o rechace la intervención propuesta antes de que el orquestador continúe.