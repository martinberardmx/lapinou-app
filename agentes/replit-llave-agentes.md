# Instrucción para el Agente de Replit — Llave de solo lectura para agentes de IA

Copia y pega todo el bloque de abajo en el Agente de Replit del proyecto del ERP (cafe-lapinou).

---

Necesito que agregues acceso de **solo lectura** a la API para mis agentes de IA (n8n),
sin romper el login actual por sesión.

1. Crea un secreto de Replit llamado `AGENT_API_KEY` (yo pondré el valor; genera uno
   aleatorio de 40+ caracteres y muéstramelo una sola vez si no existe).
2. Agrega un middleware de autenticación: si la petición trae el header
   `X-Agent-Key` y coincide con `AGENT_API_KEY` (comparación en tiempo constante),
   trátala como autenticada con rol `agent`.
3. El rol `agent` solo puede hacer **GET** y solo a estas rutas
   (cualquier otra cosa → 403):
   - `/api/reports/daily`, `/api/reports/monthly`
   - `/api/dashboard/summary`, `/api/dashboard/top-products`,
     `/api/dashboard/sales-trend`, `/api/dashboard/sales-by-category`
   - `/api/sales`
   - `/api/inventory`, `/api/inventory/low-stock`
   - `/api/production/items`, `/api/production/orders`, `/api/production/lots`
   - `/api/requisitions/suggestions`, `/api/requisitions/alerts`
   - `/api/suppliers`, `/api/suppliers/price-compare`
   - `/api/products`, `/api/categories`, `/api/expenses`
   - `/api/tasks/alerts`, `/api/quality/alerts`, `/api/maintenance/summary`
   - `/api/calendar/upcoming`, `/api/calendar/alerts`
   - `/api/fudo/status`
4. Nunca expongas datos de empleados sensibles (salarios, teléfonos, direcciones)
   al rol `agent`.
5. Registra en logs cada petición del rol `agent` (ruta y hora, sin la llave).
6. Documenta en un archivo `API_AGENTES.md` cada ruta permitida con sus
   parámetros (por ejemplo `date`, `startDate`, `endDate`) y un ejemplo de respuesta.
7. Prueba con curl que:
   - `GET /api/reports/daily` con la llave → 200
   - sin llave → 401
   - `POST` a cualquier ruta con la llave → 403
