# Máquinas de estado · v1

## Bono (`bonds.status`)

| Estado | Significado |
|--------|-------------|
| `emitido` | Registrado por TSE, pendiente de asignación on-chain |
| `activo` | En custodia del dueño actual |
| `en_venta` | Publicado en marketplace |
| `en_escrow` | Reservado durante negociación / escrow |
| `congelado` | Bloqueado por TSE |

## Transferencia (`transfers.status`)

### Flujo P2P (`payment_method`: sinpe | transferencia)

```
solicitada → aceptada → en_escrow → pago_registrado → liberada
                ↘ rechazada / cancelada
```

### Flujo wallet (`payment_method`: wallet)

```
solicitada → aceptada → [DvP on-chain] → liberada
```

No pasa por `en_escrow` off-chain; el pago USDC+bono es atómico en Stellar.

## Métodos de pago en bono (`bonds.payment_methods[]`)

El dueño elige al publicar. El comprador elige uno al ofertar (`transfers.payment_method`).
