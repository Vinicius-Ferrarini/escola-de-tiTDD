---
# Spec — Sistema de Reservas de Quadras


## Casos de uso

### UC1 — Abrir bilhete
`POST /bilhetes` — body `{"placa": "ABC1D23"}` (7 caracteres alfanuméricos, maiúsculos)
→ **201** `{"id": 1, "placa": "ABC1D23", "entrada": "<ISO-8601 com fuso -03:00>", "status": "aberto"}`


- O body aceita **`entrada` opcional** (ISO-8601 com fuso): quando presente, o bilhete abre naquele instante em vez de "agora".
- Único valor obrigatório é placa.
- Critérios de aceite:
	- Quando `placa` for diferente de 3 letras , 1 número , 1 letra , 2 letras,REGEX `^[A-Z]{3}[0-9][A-Z][0-9]{2}$`, retornar erro 409
	- Não permitir abrir 2 bilhetes por 
	

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
	- Dada uma permanência de 300 min (20 frações = 5000), quando encerro, então o valor é 5000. Esse é o caso de bater exatamente no teto

### UC3 — Listar ativos
`GET /bilhetes/ativos` 
→ **200** com array dos bilhetes abertos, mais recentes primeiro.
- Critérios de aceite:
	- Dados os bilhetes abertos A (entrada 10:00), B (10:30) e C (11:00)
	
	
### UC4 — Relatório diário
`GET /relatorios/diario?data=AAAA-MM-DD` 
→ **200**:{"data": "2026-10-05", "total_bilhetes": 12, "faturamento_centavos": 8400, "tempo_medio_minutos": 47}

`tempo_medio_minutos` considera apenas bilhetes encerrados no dia, arredondando
**0,5 para cima**.
- Critérios de aceite:
	- Listar todos os bilhetes encerrados do dia
	- Se digitar data errada retornar erro 404


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
	- Se tentar puxar histórico de alguma placa que nunca estacionou deve retornar 409

### UC7 — Tolerância gratuita
Os primeiros `TOLERANCIA_MINUTOS` de um bilhete são **grátis**: duração ≤
tolerância → `valor_centavos: 0`. Passou da tolerância (mesmo por 1 minuto) →
cobra **integral desde o primeiro minuto** — a tolerância **não** é descontada.
- Critérios de aceite:
	- Quando duração ≤ tolerância , deve retornar grátis

### UC8 — Uma vaga por placa
`POST /bilhetes` para placa que já tem bilhete **aberto** → **409**
`{"erro": "bilhete_em_aberto"}`. Após encerrar ou cancelar, a placa volta a
poder abrir.
- Critérios de aceite:
	- Quando tento fazer novamente o `POST /bilhetes` com uma placa que está ativa retornar erro 400