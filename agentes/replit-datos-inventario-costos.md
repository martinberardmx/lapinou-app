# Instrucción para el Agente de Replit — Arreglar inventario y costos

Diagnóstico hecho el 2026-11-20 con la API de agentes (solo lectura):

| Problema | Cifra |
|---|---|
| Ingredientes de Fudo sin vincular al inventario del ERP | 276 de 339 (aviso en cada sincronización) |
| Insumos del inventario con costo unitario $0 | 132 de 189 |
| Insumos sin punto de reorden | 106 de 189 |
| Productos activos sin costo | 293 de 310 |
| Productos activos sin receta | 300 de 310 |
| Recetas de producción (panadería) sin costo calculado | 23 de 23 |
| Gastos importados de Fudo con monto $0 | 36 de 43 |
| Ventas de abril-mayo (importación CSV) sin producto vinculado | 994 líneas |

Por eso el consumo de inventario sale en 0 y no se puede calcular el food cost.

---

Copia y pega en el Agente de Replit **una parte a la vez**. Antes de escribir datos,
el agente debe mostrarte qué va a cambiar.

## Parte 1 — Vincular ingredientes de Fudo con el inventario

```
Cada sincronización con Fudo avisa "276 ingredientes de Fudo sin vincular al inventario".
1. Crea una pantalla "Vincular ingredientes" que liste los ingredientes de Fudo sin vincular
   y, para cada uno, proponga el insumo del inventario más parecido (por nombre normalizado:
   sin acentos, minúsculas, sin unidades) con un % de similitud.
2. Permite: aceptar la propuesta, elegir otro insumo, o "crear insumo nuevo" con los datos
   de Fudo (nombre, unidad, costo).
3. Botón "Aceptar todas las propuestas ≥ 90%".
4. Al vincular, recalcula el stock con las compras de Fudo ya importadas de ese ingrediente.
5. NO borres ni fusiones insumos automáticamente.
```

## Parte 2 — Costos de insumos

```
132 de 189 insumos tienen unitCost = 0.
1. Si Fudo trae costo de ingrediente o precio de compra, usa el costo de la última compra
   para cada insumo vinculado (y guarda el historial de costos por fecha).
2. Para los que sigan en 0, agrega un filtro "Sin costo" en Inventario para capturarlos a mano.
3. Revisa la importación de gastos desde Fudo: 36 de 43 gastos llegan con monto 0
   (ej. "Otros — Tiendita", comprobante "Fudo #406"). Corrige el mapeo del campo de monto
   y reimporta los gastos afectados sin duplicarlos.
```

## Parte 3 — Recetas y costo por producto (food cost) — FUENTE: FUDO

Decisión (2026-11-22): las recetas, fichas técnicas y costos de ingredientes se capturan
SOLO en Fudo. El ERP los lee; no se capturan dos veces. En Fudo cada producto tiene
receta con ingrediente, cantidad neta, merma, cantidad bruta y costo (ej. Capuccino francés
= $15.25), y las sub-recetas están en "Fichas técnicas".

```
Antes de cambiar código, respóndeme:
1. ¿La sincronización con Fudo importa hoy recetas (ingredientes y cantidades por producto)
   y el costo de cada ingrediente? Muéstrame qué campos de la API de Fudo lees.
2. ¿La API de Fudo expone las recetas de productos, las fichas técnicas y el costo de
   ingredientes? Haz una consulta de prueba al producto "Capuccino francés"
   (en Fudo su receta suma $15.25) y muéstrame la respuesta.

Si la API sí las expone:
3. Haz que la sincronización importe recetas, fichas técnicas (sub-recetas) y costos de
   ingredientes desde Fudo como fuente de verdad; usa la cantidad BRUTA para el costo
   y el consumo. Vincula cada ingrediente de Fudo con su insumo del inventario (Parte 1).
4. No sobrescribas las 17 recetas capturadas a mano en el ERP sin mostrarme antes la
   diferencia contra Fudo.
5. Al importar ventas, descuenta inventario según la receta (consumo teórico).
6. Agrega al rol `agent` (llave de agentes, solo GET):
   - GET /api/reports/food-cost?startDate=&endDate=  → venta, costo teórico y % por
     producto y por categoría, y lista de productos vendidos sin receta.
   - GET /api/data-quality → conteos del diagnóstico para seguir el avance.

Si la API NO las expone: dímelo y propón la alternativa (por ejemplo, exportación de
Fudo a Excel/CSV que se suba a Google Drive y el ERP importe).
```

## Parte 4 — Merma de panadería

```
1. Agrega "Registrar merma": producto o lote, cantidad, motivo (sobrante del día,
   caducado, quemado/defecto, consumo personal, degustación), fecha y quién.
2. Al cierre del día, en Producción, permite marcar el sobrante de cada lote del día
   en un solo paso.
3. Agrega al rol `agent` la ruta GET /api/reports/waste?startDate=&endDate=
   (cantidad y costo de merma por producto y motivo).
```

## Parte 5 (opcional) — Ventas antiguas sin producto

```
Hay 994 líneas de venta de abril-mayo 2026 (importación CSV) con productId = null.
Vincúlalas al producto por nombre normalizado cuando el match sea exacto; muéstrame
las que no tengan match antes de cambiar nada.
```
