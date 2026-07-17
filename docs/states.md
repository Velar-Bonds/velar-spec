# Máquinas de estado · v1

> Las transiciones legales están codificadas de forma verificable en
> [`schemas/state-machine.json`](../schemas/state-machine.json). Este documento
> es la versión legible. Los enums viven en `schemas/bond-status.json` y
> `schemas/transfer-status.json`.

## Bono (`bonds.status`)

| Estado | Significado |
|--------|-------------|
| `emitido` | Registrado por el TSE, pendiente de asignación on-chain |
| `pendiente` | Solicitud de bono en revisión (flujo bond-request) |
| `aprobado` | Solicitud aprobada, lista para emitirse/activarse |
| `activo` | En custodia del dueño actual |
| `en_venta` | Publicado en el marketplace |
| `en_escrow` | Reservado durante negociación / escrow |
| `transferido` | Movido a un nuevo dueño (resultado de una transferencia) |
| `congelado` | Bloqueado por el TSE |
| `cancelado` | Anulado (terminal) |
| `rechazado` | Solicitud rechazada (terminal) |

**No transferibles:** `en_escrow`, `cancelado`, `congelado`, `pendiente`, `rechazado`.

### Transiciones

```
emitido    → activo | cancelado
pendiente  → aprobado | rechazado
aprobado   → activo
activo     → en_venta | en_escrow | congelado | cancelado
en_venta   → en_escrow | activo | congelado
en_escrow  → transferido | activo | congelado
transferido→ activo
congelado  → activo
```

## Transferencia (`transfers.status`)

| Estado | Significado |
|--------|-------------|
| `solicitada` | El comprador inició la solicitud de compra/recompra |
| `contraoferta` | Se envió una contraoferta durante la negociación |
| `aceptada` | El dueño aceptó la intención de compra |
| `en_escrow` | Token bloqueado en la canasta/escrow on-chain |
| `pago_registrado` | Pago físico registrado, pendiente de validación |
| `pago_validado` | Pago confirmado; escrow liberable |
| `liberada` | Token liberado al comprador; transferencia completa (terminal) |
| `rechazada` | Rechazada por el dueño o el sistema (terminal) |
| `cancelada` | Cancelada antes de la validación final (terminal) |

### Flujo P2P (`payment_method`: sinpe | transferencia)

```
solicitada → aceptada → en_escrow → pago_registrado → pago_validado → liberada
     ↘ contraoferta ↗            ↘ rechazada / cancelada
```

### Flujo wallet (`payment_method`: wallet)

```
solicitada → aceptada → [DvP atómico USDC+bono on-chain] → liberada
```

No pasa por `en_escrow` off-chain; el pago USDC + el bono se mueven de forma
atómica en Stellar.

## Métodos de pago en bono (`bonds.payment_methods[]`)

El dueño elige al publicar (ver `schemas/payment-method.json`). El comprador
elige uno al ofertar (`transfers.payment_method`).
