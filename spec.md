---
# Spec — Zona Azul Digital


## Casos de uso

### UC1 — Abrir bilhete
`POST /bilhetes` — body `{"placa": "ABC1D23"}` (7 caracteres alfanuméricos, maiúsculos)
→ **201** `{"id": 1, "placa": "ABC1D23", "entrada": "<ISO-8601 com fuso -03:00>", "status": "aberto"}`


- O body aceita **`entrada` opcional** (ISO-8601 com fuso): quando presente, o bilhete abre naquele instante em vez de "agora".
- Único valor obrigatório é placa.
- Critérios de aceite:
	- Quando `placa` estiver ausente ou fora de `^[A-Z0-9]{7}$`, retornar 422 `{"erro": "placa_invalida"}`.
	- - Dada a placa ABC1D23 com bilhete aberto, quando faço POST /bilhetes de novo, então recebo 409. 
	

### UC2 — Encerrar bilhete
`POST /bilhetes/{id}/encerramento` → **200**:
```json
{"id": 1, "placa": "ABC1D23", "entrada": "...", "saida": "...",
 "minutos": 95, "valor_centavos": 1250}
```
Regras de valor:
- Cobra-se por fração de `FRACAO_MINUTOS` minutos, **arredondando para cima**
  (fração exata cobra 1 fração; 1 minuto a mais já cobra a fração seguinte);
- hora cheia = `TARIFA_HORA_CENTAVOS`; valor da fração = tarifa ÷ (60 ÷ `FRACAO_MINUTOS`);
- aplica-se o **teto diário**: `valor_centavos` nunca supera `TETO_DIARIO_CENTAVOS`;
- valor sempre em **centavos, inteiro** — a API nunca retorna ponto flutuante.[^por-que-centavos]
- Critérios de aceite:
	- Dada uma permanência de 1066 min (72 frações = 8100), quando encerro, então `valor_centavos` é 8000 (teto).

### UC3 — Listar ativos
`GET /bilhetes/ativos` 
→ **200** com array dos bilhetes abertos, mais recentes primeiro.
- Critérios de aceite:
	- Dados os bilhetes abertos A (10:00), B (10:30) e C (11:00), quando faço GET /bilhetes/ativos, então recebo 200 na ordem C, B, A.
	
	
### UC4 — Relatório diário
`GET /relatorios/diario?data=AAAA-MM-DD` 
→ **200**:{"data": "2026-10-05", "total_bilhetes": 12, "faturamento_centavos": 8400, "tempo_medio_minutos": 47}

`tempo_medio_minutos` considera apenas bilhetes encerrados no dia, arredondando
**0,5 para cima**.
- Critérios de aceite:
	- Listar todos os bilhetes encerrados do dia
	- Quando `data` não estiver no formato AAAA-MM-DD, retornar 422 `{"erro": "data_invalida"}`.


### UC5 — Cancelar bilhete
`POST /bilhetes/{id}/cancelamento` → **200** com `status: "cancelado"`.
Só bilhetes **abertos** podem ser cancelados — sem cobrança (não gera `saida`
nem `valor_centavos`).
- Critérios de aceite:
	- Quando faço cancelamento, devo receber código 200 e não ser possivel listar novamente


### UC6 — Histórico por placa
`GET /bilhetes?placa=ABC1D23` → **200** com array de **todos** os bilhetes da
placa (qualquer status), mais recentes primeiro. Placa que nunca estacionou →
array vazio.
- Critérios de aceite:
	- Quando a placa nunca estacionou, retornar 200 com array vazio [].

### UC7 — Tolerância gratuita
Os primeiros `TOLERANCIA_MINUTOS` de um bilhete são **grátis**: duração ≤
tolerância → `valor_centavos: 0`. Passou da tolerância (mesmo por 1 minuto) →
cobra **integral desde o primeiro minuto** — a tolerância **não** é descontada.
- Critérios de aceite:
	- Com 15 min, `valor_centavos` é 0; com 16 min, é 225 (cobrado desde o primeiro minuto).

### UC8 — Uma vaga por placa
`POST /bilhetes` para placa que já tem bilhete **aberto** → **409**
`{"erro": "bilhete_em_aberto"}`. Após encerrar ou cancelar, a placa volta a
poder abrir.
- Critérios de aceite:
	- Quando faço POST /bilhetes de novo com uma placa que está ativa, retornar 409 `{"erro": "bilhete_em_aberto"}`.