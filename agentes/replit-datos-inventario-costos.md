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

## Parte 3 — Recetas y costo por producto (food cost)

```
1. Primero revisa si la API de Fudo expone recetas de productos (ingredientes y cantidades
   por producto). Si sí, impórtalas y vincúlalas a los insumos del ERP.
2. Si no, permite importar recetas desde una hoja de Google Sheets / CSV con columnas:
   Producto, Ingrediente, Cantidad, Unidad. (Martín está llenando la hoja
   "Recetas Lapinou — Top 25" en Google Drive.)
3. Calcula el costo de cada producto = suma(cantidad × costo unitario del insumo),
   convirtiendo unidades (g↔kg, ml↔L, pieza).
4. Calcula el recipeCost de las 23 recetas de producción (panadería) igual, y que el
   costo de una sub-receta (ej. biga, masa madre) se use dentro de otras recetas.
5. Al importar ventas de Fudo, descuenta del inventario los insumos según la receta
   (consumo teórico) y registra el movimiento.
6. Agrega al rol `agent` (llave de agentes, solo GET) estas rutas:
   - GET /api/reports/food-cost?startDate=&endDate=  → venta, costo teórico y % por
     producto y por categoría, y lista de productos vendidos sin receta.
   - GET /api/data-quality → los conteos de la tabla del diagnóstico, para que
     los agentes sigan el avance.
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
