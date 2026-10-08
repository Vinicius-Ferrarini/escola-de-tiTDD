# Tests — Casos de borda por regra de negócio

> [!WARNING]
> Teto: `valor_centavos` nunca passa de **8000**, mesmo que o tempo exceda.

## Valor (UC2 + UC7): tarifa 450/h, fração 15 min (112,5), tolerância 15, teto 8000

| Caso | Minutos | `valor_centavos` esperado |
| --- | --- | --- |
| Zero | 0 | 0 |
| Limite exato da tolerância | 15 | 0 |
| Tolerância +1 (cobra integral, sem descontar) | 16 | 225 |
| Fração exata | 30 | 225 |
| Adjacente +1 min | 31 | 338 |
| Hora cheia | 60 | 450 |
| Caso quebrado | 95 | 788 |
| Logo abaixo do teto | 1065 | 7988 |
| Teto (+1 min) | 1066 | 8000 |
| Muito acima do teto | 2880 | 8000 |

- `valor_centavos` sempre inteiro no JSON (nunca `12.50` nem `1250.0`).
- Segundos residuais são truncados: `entrada` = agora − 15 min, encerrado em seguida → 0.

## Relatório (UC4)

| Bilhetes encerrados no dia (minutos) | `tempo_medio_minutos` esperado |
| --- | --- |
| 30 e 31 (média 30,5) | 31 (arredonda 0,5 para cima) |
| 30, 30 e 31 (média 30,33) | 30 |
| 30, 31 e 31 (média 30,67) | 31 |
| Nenhum bilhete | `total_bilhetes` 0, `faturamento_centavos` 0, `tempo_medio_minutos` 0 |

- Bilhetes abertos, cancelados ou encerrados em outro dia não entram no relatório.
- `data=05/10/2026` → 422 `{"erro": "data_invalida"}`.

## Validação e conflitos (UC1, UC5, UC6, UC8)

| Situação | Esperado |
| --- | --- |
| `POST /bilhetes` sem placa | 422 `placa_invalida` |
| Placa `abc1d23` (minúscula) | 422 `placa_invalida` |
| Placa `ABC1D2` (6 caracteres) | 422 `placa_invalida` |
| `entrada` = `ontem` | 422 `entrada_invalida` |
| Placa com bilhete aberto **e** `entrada` inválida | 422 `entrada_invalida` (422 vem antes de 409) |
| Placa com bilhete aberto | 409 `bilhete_em_aberto` |
| Abrir de novo após encerrar ou cancelar | 201 |
| Encerrar duas vezes | 409 `bilhete_ja_encerrado` |
| Encerrar ou cancelar o id 999 | 404 `bilhete_nao_encontrado` |
| Cancelar bilhete encerrado ou cancelado | 409 `bilhete_nao_aberto` |
| `GET /bilhetes?placa=ZZZ9Z99` (nunca estacionou) | 200 `[]` |

## Ordenação (UC3, UC6)

| Dados | Esperado |
| --- | --- |
| Abertos A (10:00), B (10:30), C (11:00) | `GET /bilhetes/ativos` → C, B, A |
| A encerrado, B cancelado, C aberto (mesma placa) | Ativos → só C; histórico → C, B, A |