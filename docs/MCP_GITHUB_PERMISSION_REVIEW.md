# MCP GitHub Permission Review — ADA-06

## 1. Tabla de Evaluación de Capacidades y Permisos

| Capability | Needed | Granted | Control | Decision |
| :--- | :--- | :--- | :--- | :--- |
| **Read repository contents** | Yes | Yes | PAT Contents: Read | Keep |
| **Read Issues** | Yes | Yes | PAT Issues: Read | Keep |
| **Read Pull Requests** | Yes | Yes | PAT Pull requests: Read | Keep |
| **Create / Edit Issue** | No | No | MCP Read-Only (`X-MCP-Readonly: true`) | Block |
| **Comment on PR** | No | No | MCP Read-Only (`X-MCP-Readonly: true`) | Block |
| **Push files / Branch** | No | No | MCP Read-Only + PAT Contents: Read-only | Block |
| **Other repositories** | No | No | Fine-grained PAT (Selected Repo Only) | Block |


## 2. Definición de Capas de Seguridad 

* **Capas de Autenticación (Authentication):** Identifica la identidad del cliente ante GitHub mediante el Fine-Grained Personal Access Token (PAT) enviado en la cabecera `Authorization: Bearer <TOKEN>`.
* **Capas de Autorización (Authorization - GitHub Level):** Define qué acciones tiene permitido ejecutar el PAT dentro de la API REST de GitHub. En este ADA, el PAT solo tiene permisos `Read-Only` para un único repositorio seleccionado.
* **Exposición / Política de Herramientas MCP (MCP Tool Exposure & Policy):** El servidor MCP filtra y elimina la exposición de herramientas mutables (escritura/edición) al modelo de lenguaje a través de la cabecera `X-MCP-Readonly: true` y restringe el alcance mediante `X-MCP-Toolsets: repos, issues, pull_requests`.


## 3. Análisis de Riesgos de Seguridad

| Riesgo | Ejemplo | Mitigación |
| :--- | :--- | :--- |
| **Over-permissioned token** | Un token con acceso a todos los repositorios de la cuenta o con permisos de escritura (*write*). | Uso de Fine-grained PAT limitado explícitamente a un único repositorio y asignando únicamente permisos *Read-only*. |
| **Credential leakage** | Incluir el PAT en un commit, captura de pantalla, informe o en el archivo `AI_USAGE_LOG.md`. | No versionar ni publicar credenciales en el repositorio; en caso de filtración accidental, revocar el token inmediatamente en GitHub. |
| **Prompt injection** | Un Issue o Pull Request malicioso creado por un tercero que contenga instrucciones para manipular las respuestas de la IA. | Tratar todo el contenido proveniente de GitHub (Issues, PRs, código) como datos no confiables y mantener el servidor en modo *read-only*. |
| **Unnecessary tools** | Habilitar toolsets innecesarios como `actions`, `security_events` o `admin`. | Restringir los toolsets exclusivamente a `repos`, `issues` y `pull_requests`. |
| **Scope confusion** | Que el agente lea o analice por error un repositorio diferente al asignado. | Especificar explícitamente la tupla `OWNER/REPOSITORY` en los prompts enviados al agente. |


## 4. Análisis de Lockdown Mode

### ¿Qué es el Lockdown Mode?
El **Lockdown Mode** es un mecanismo de protección del servidor MCP de GitHub diseñado como una medida de mitigación *best-effort* contra ataques de **Prompt Injection** indirecto. Cuando está activado, el servidor limita o filtra el contenido devuelto por la API para reducir la probabilidad de que instrucciones maliciosas incrustadas en Issues o Pull Requests manipulen la conducta del agente.

### ¿Por qué NO sustituye una frontera de autorización?
* **Mitigación Best-Effort:** No garantiza al 100% que una instrucción maliciosa pase desapercibida o no engañe al LLM.
* **Falta de aislamiento real:** El análisis de texto de un LLM siempre es propenso a evasiones si el modelo procesa la entrada como contexto activo.
* **Separación de capas:** La verdadera frontera de seguridad radica en la **Autorización** (los permisos *Read-Only* del Fine-Grained PAT y la restricción de herramientas mutables mediante `X-MCP-Readonly: true`). Sin permisos de escritura en la API de GitHub, el impacto de un Prompt Injection exitoso se reduce drásticamente, ya que la IA no tiene herramientas disponibles para modificar el repositorio ni ejecutar acciones destructivas.