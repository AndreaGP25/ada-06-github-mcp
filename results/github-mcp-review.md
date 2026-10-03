# GitHub MCP Engineering Review — ADA-06

## 1. Repository
* **Owner / Repo:** `AndreaGP25/ADA-05_Spec_Driven_Feature`
* **Visibility:** Public
* **Branch / reference analyzed:** `main` (base) y `fix/diacritics-accent-insensitive-search` (PR #2, commit `83ee319`)

## 2. MCP Connection
* **Server:** `github-readonly` (SSE: `https://api.githubcopilot.com/mcp/`)
* **Mode:** READ-ONLY (`X-MCP-Readonly: true`)
* **Toolsets:**
  - `repos`
  - `issues`
  - `pull_requests`

## 3. Repository Understanding
* **Purpose:** Aplicación CLI en Python para búsqueda en memoria de clientes por nombre o correo, bajo Spec-Driven Development.
* **Architecture summary:** Arquitectura en capas limpia: Dominio (`models.py`, `exceptions.py`), Repositorio (`customer_repository.py`), Servicio (`search_service.py`) y CLI (`cli.py`).
* **Important files:** `SPEC.md`, `ARCHITECTURE.md`, `REQUIREMENTS.md`, `src/services/search_service.py`, `src/entrypoints/cli.py`.
* **Tests:** Unitarios e integración con `pytest` y `pytest-cov` (meta de cobertura $\ge 90\%$).

## 4. Issue Analysis
* **Issue:** Issue #1 — *"Búsqueda no encuentra registros cuando el término incluye tildes o caracteres acentuados"*.
* **Facts from Issue:** `CustomerSearchService` solo usa `.lower()`. Buscar "maría" o "lucía" no devuelve resultados aunque existen "Maria Garcia" y "Lucia Fernandez".
* **Ambiguities:** No precisa el alcance diacrítico (¿solo tildes agudas o también `ñ`/diéresis?) ni fija criterios de aceptación formales.
* **Related requirements/spec:** `REQUIREMENTS.md` (`FR-02`), `SPEC.md` (`Search Rules`, `AC-02`).
* **Related code:** `src/services/search_service.py` (`search()`).
* **Related tests:** En `main` no existían pruebas con caracteres acentuados.

## 5. Pull Request Review
* **PR:** PR #2 — *"fix: normalizar tildes y diacriticos en busqueda de clientes"*.
* **Changed behavior:** Añade `_normalize()` con `unicodedata.normalize('NFKD')` para remover marcas diacríticas en la consulta y en cada registro.
* **Changed files:** `src/services/search_service.py`, `README.md`, `tests/unit/test_search_service.py`, `tests/integration/test_interface.py`.
* **Tests:** 3 pruebas unitarias y 1 prueba de integración CLI añadidas.
* **Observations:** Alcance limpio y puntual. Omite actualizar `SPEC.md` y `docs/traceability.md`.
* **Risks:** Normalización en caliente de cada registro en cada consulta (`NFR-01`); imperceptible con 12 registros, pero ineficiente a gran escala.
* **Questions:** `NFKD` convierte `ñ` en `n` (ej. "pena" coincide con "Peña"). ¿Es la regla de negocio esperada?
* **Potential defects:** Validar longitud antes de normalizar: una consulta con solo marcas combinantes (ej. `"\u0301\u0301"`) pasa la validación ($\ge 2$), se normaliza a `""` y retorna **todos los registros del catálogo**.

## 6. Traceability
`Issue #1` $\longrightarrow$ `FR-02 / AC-02` $\longrightarrow$ `PR #2` $\longrightarrow$ `search_service.py (_normalize)` $\longrightarrow$ `test_search_service.py & test_interface.py`

## 7. Permission Review
* **Authentication:** GitHub Personal Access Token (PAT).
* **GitHub token permissions:** Lectura exclusiva sobre repositorios, issues y pull requests.
* **MCP read-only:** `True` (encabezado HTTP forzado).
* **Enabled toolsets:** `repos, issues, pull_requests`.
* **Write capabilities exposed:** Ninguna (0 herramientas de escritura).

## 8. Security Notes
* **Credential exposure:** PAT local en `mcp_config.json`; proteger permisos del archivo y no versionar.
* **Prompt injection:** Datos externos de GitHub leídos de forma pasiva sin ejecutar comandos embebidos.
* **Excess permissions:** Mínimo privilegio cumplido (sin acceso a CI/CD, secretos ni escritura).
* **Repository scope:** Consultas acotadas a `AndreaGP25/ADA-05_Spec_Driven_Feature`.

## 9. Human Review
* **What did you verify yourself?:** Validé que SPEC.md y traceability.md permanecieron intactos en main y en la rama del PR.
* **What AI conclusions did you reject or modify?:** Se reclasificó el caso de `ñ` como pregunta de producto en lugar de defecto, y el riesgo de rendimiento se acotó a escalabilidad futura.

## 10. Conclusion
* **What did MCP add to the engineering workflow?:** Inspección integral y trazabilidad en tiempo real (Issue $\rightarrow$ Spec $\rightarrow$ PR $\rightarrow$ Pruebas) dentro del entorno de desarrollo, con garantías estrictas de solo lectura y sin fricción de clonado manual.