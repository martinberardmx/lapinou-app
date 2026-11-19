# Agentes de Café Lapinou

Sistema de agentes de IA que trabajan para Café Lapinou.

## Arquitectura

```
Martín (WhatsApp, número de administración)
        │
        ▼
   n8n Cloud ──► Agente Director (Claude)
                    │  lee la Memoria en Google Drive
                    ├── Marketing y redes
                    ├── Finanzas y control de gestión
                    ├── Operaciones (compras, inventario, producción)
                    ├── Clientes y lealtad (CRM)
                    └── Expansión
```

- **Cerebro:** Claude (API de Anthropic), un prompt por agente en esta carpeta.
- **Orquestación:** n8n Cloud (horarios, webhooks, credenciales).
- **Memoria compartida:** Google Drive → carpeta `🐰 Cerebro Lapinou (Agentes)`.
- **Canal con Martín:** WhatsApp vía Twilio, número separado del bot de clientes.

## Fuentes de datos

| Sistema | Uso | Conexión |
|---|---|---|
| Fudo | Punto de venta | API (vía ERP) |
| ERP propio (Replit) | Gestión interna | API (por confirmar) |
| PWA de lealtad (`index.html`) | Pasaporte, visitas, drops | Firebase |
| Contabilidad | Excel → Google Sheets | Google Sheets |
| Inbursa / Banorte | Bancos | Estados de cuenta en Drive |
| Mercado Pago | Cobros | API |

## Fases

1. **Base:** memoria de empresa + Director con resumen diario (8am) por WhatsApp. ← en curso
2. Operaciones (producción de panadería, compras, costeo de recetas)
3. Marketing y redes
4. Finanzas y control de gestión
5. Clientes y lealtad
6. Expansión

## Regla de oro

Nada sale hacia afuera (publicar, pagar, escribir a terceros) sin aprobación de Martín.
