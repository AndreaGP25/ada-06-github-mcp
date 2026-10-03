# AI Usage Log — ADA-06

### Entrada 1
* **Etapa:** MCP configuration 
* **Objetivo:** Configurar el GitHub MCP Server oficial en modo de solo lectura con mínimo privilegio.
* **Acción realizada:**
  * Se creó un Fine-Grained Personal Access Token (PAT) en GitHub limitado únicamente al repositorio del proyecto y con permisos `Read-Only` para `Contents`, `Issues` y `Pull Requests`.
  * Se configuró `mcp_config.json` agregando la cabecera `X-MCP-Readonly: true` y limitando los toolsets a `repos, issues, pull_requests`.
* **Decisión humana / Evidencia:**
  * Decidí no almacenar el token en el repositorio ni en variables de entorno versionables.
  * Se ejecutó `/mcp` en el cliente Antigravity y se verificó que el servidor `github-readonly` estuviera operativo exponiendo exclusivamente 22 herramientas de consulta (lectura/búsqueda) sin capacidades de escritura.

---

### Entrada 2
* **Etapa:** Repository analysis (Fase 2 — Repository Understanding)
* **Objetivo:** Inspeccionar la estructura del proyecto y arquitectura mediante GitHub MCP sin usar interfaz web ni copia manual.
* **Acción realizada:**
  * Se solicitó a la IA analizar el repositorio mediante el prompt estandarizado del ADA.
  * La IA ejecutó las herramientas MCP `get_file_contents` y `search_code` para revisar archivos clave (`README.md`, `REQUIREMENTS.md`, `SPEC.md`, `src/services/search_service.py`, etc.).
* **Decisión humana / Evidencia:**
  * Se inspeccionaron las invocaciones MCP realizadas por el agente y se corroboró que la descripción de los componentes principales (como `CustomerSearchService` y `DEFAULT_CUSTOMERS`) coincidía con el código real alojado en la rama `main`.

---

### Entrada 3
* **Etapa:** Issue / PR analysis (Fases 3 y 4 — Issue & Pull Request Analysis)
* **Objetivo:** Analizar el bug de búsqueda insensible a acentos reportado en el Issue #1 y su propuesta en el PR correspondiente.
* **Acción realizada:**
  * La IA utilizó `issue_read` para extraer el problema reportado en el Issue #1 sobre la falta de normalización de diacríticos en `CustomerSearchService`.
  * Posteriormente, la IA utilizó `pull_request_read` para analizar los cambios propuestos en los archivos y las pruebas añadidas.
* **Decisión humana / Evidencia:**
  * Se separaron explícitamente los hechos reportados en el Issue (ej. "maría" no retorna "Maria Garcia") de las inferencias de ingeniería del agente (ej. comportamiento esperado de caracteres especiales como la "ñ" en castellano).
  * Veriqué manualmente que la brecha detectada en las pruebas unitarias existentes se cubriera adecuadamente en la revisión del PR.

---

### Entrada 4
* **Etapa:** Permission / security review 
* **Objetivo:** Validar la frontera del modo Read-Only y analizar los riesgos de seguridad y el *Lockdown Mode*.
* **Acción realizada:**
  * Se solicitó a la IA evaluar sus propias herramientas para comprobar si podía realizar modificaciones o comentarios en GitHub.
* **Decisión humana / Evidencia:**
  * La IA confirmó la imposibilidad de ejecutar acciones de escritura debido a la ausencia de herramientas mutables expuestas.
  * Se elaboró el análisis de riesgos documentando las tres capas de seguridad (Autenticación, Autorización del PAT y Política MCP) y se evaluó la función del *Lockdown Mode* como mitigación de *Prompt Injection* de esfuerzo razonable (*best-effort*).