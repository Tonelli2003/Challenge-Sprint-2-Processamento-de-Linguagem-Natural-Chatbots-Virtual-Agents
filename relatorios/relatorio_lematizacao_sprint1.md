# Relatorio de Lematizacao Comparativo
## Sprint 1 | PLN Forzy
Data: 2026-05-09

## Metricas (amostra 300 tokens)

| Abordagem | Vocabulario | Reducao |
|---|---|---|
| Sem transformacao | 287 | - |
| Stemming RSLP | 281 | 2.09% |
| Lematizacao (adotado) | 285 | 0.7% |

## Exemplos de Transformacao

| Token | Lema | Metodo |
|---|---|---|
| `lubrificantes` | `lubrificante` | lematizacao_pt |
| `não-lineares` | `não-linear` | lematizacao_pt |
| `capacitores` | `capacitor` | lematizacao_pt |
| `industriais` | `industrial` | lematizacao_pt |
| `Motores` | `motor` | lematizacao_pt |
| `equivalentes` | `equivalente` | lematizacao_pt |
| `enrolamentos` | `enrolamento` | lematizacao_pt |
| `motores` | `motor` | lematizacao_pt |
| `Polos` | `polo` | lematizacao_pt |
| `Valores` | `valor` | lematizacao_pt |

## Decisao Final
Lematizacao por regras morfologicas. Stemming RSLP descartado.
