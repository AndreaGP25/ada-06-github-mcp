# MCP Tool Inventory — ADA-06

## Servidor Configurado
- **Nombre:** `github-readonly`
- **Transporte:** SSE (`https://api.githubcopilot.com/mcp/`)
- **Total de herramientas detectadas:** 22
- **Modo:** Read-Only

## Tabla de Herramientas (Inventory)

| Tool / Capacidad | Toolset | Read/Write | Uso en el ADA | Riesgo |
| :--- | :--- | :--- | :--- | :--- |
| `get_file_contents` | Repositories & Code | Read | Leer el código fuente y archivos de documentación | Bajo |
| `search_repositories`, `search_code`, `list_branches`, `get_commit`, `list_commits`, `search_commits` | Repositories & Code | Read | Navegar por la estructura, historial de commits y cambios del repositorio | Bajo |
| `get_latest_release`, `get_release_by_tag`, `list_releases`, `get_tag`, `list_tags`, `list_repository_collaborators` | Repositories & Code | Read | Consultar información suplementaria de versiones y colaboradores | Bajo |
| `issue_read`, `list_issues`, `search_issues`, `list_issue_fields`, `list_issue_types`, `get_label` | Issues | Read | Consultar los requerimientos, bug reports y detalles de las Issues | Medio: contenido no confiable (riesgo de Prompt Injection) |
| `pull_request_read`, `list_pull_requests`, `search_pull_requests` | Pull Requests | Read | Inspeccionar el código modificado, los cambios propuestos y comentarios del PR | Medio: contenido no confiable (riesgo de Prompt Injection) |
| **Write tools** (ej. `create_issue`, `update_file`, `create_pull_request`) | N/A | No disponibles | Ninguna disponible ni expuesta por el servidor MCP | Alto si estuvieran habilitadas (Bloqueadas por Read-Only) |