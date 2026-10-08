# Plan — Arquitetura e decisões técnicas

## Stack

- **Python 3.11 + FastAPI + uvicorn**, pydantic v2. Justificativa: pouco código para um contrato REST pequeno e TestClient pronto para testar.
- Persistência **em memória** (dict de bilhetes + contador de id), porque o enunciado não exige banco e a suíte sobe o serviço do zero.

## Estrutura de arquivos

```
app/main.py         # rotas e handlers de erro {"erro": ...}
app/pricing.py      # cálculo de valor (tolerância, fração, teto)
app/store.py        # bilhetes em memória, ids sequenciais, reset()
tests/test_api.py   # um teste por caso do tests.md
requirements.txt    # fastapi, uvicorn, pydantic, pytest, httpx
Containerfile       # imagem do serviço
README.md           # como rodar local, em container e como testar
```

## Decisões

1. **Dinheiro em centavos `int`**, porque float acumula erro (0.1 + 0.2 != 0.3) e o contrato exige `valor_centavos` inteiro.
2. **Minutos truncados**: `minutos = int((saida - entrada).total_seconds() // 60)`. Motivo: segundos residuais não podem empurrar 15 min para 16 min.
3. **Cálculo do valor**, nesta ordem:
   - se `minutos <= 15` (tolerância), o valor é 0;
   - senão `fracoes = (minutos + 14) // 15`, ou seja, o ceil sobre os minutos inteiros, sem descontar a tolerância;
   - `valor = (fracoes * 450 + 3) // 4`, que é o ceil de 112,5 × frações. Justificativa: multiplicar antes de dividir mantém tudo inteiro, e arredondar para cima segue a regra do enunciado;
   - `valor = min(valor, 8000)` (teto).
4. **Placa**: regex `^[A-Z0-9]{7}$`, porque o contrato pede 7 alfanuméricos maiúsculos. Placa ausente ou fora do formato retorna 422 `placa_invalida`.
5. **Erros**: substituir o handler `RequestValidationError` do FastAPI para responder 422 com `{"erro": ...}`. A validação de formato roda antes de qualquer regra 409.
6. **Datas**: `entrada` é opcional e parseada com `datetime.fromisoformat`; sem fuso ou inválida retorna 422 `entrada_invalida`. Toda data sai em ISO-8601 com fuso `-03:00`. A `saida` é o relógio atual em `-03:00`.
7. **Status** em português: `aberto`, `encerrado`, `cancelado`. Cancelar não gera `saida` nem `valor_centavos`.
8. **Ordenação** (ativos e histórico): `entrada` decrescente e, em empate, `id` decrescente.
9. **Relatório**: considera os bilhetes encerrados cuja `saida` cai na data (fuso -03:00). `tempo_medio_minutos = (2*soma + n) // (2*n)`, que arredonda 0,5 para cima; sem bilhetes, o resultado é 0. Data fora de `AAAA-MM-DD` retorna 422 `data_invalida`.
10. **Porta 8080 dentro do container**, porque a suíte executa `docker run -p 8002:8080` (contrato: `porta_interna` 8080, `PORTA_SERVICO` 8002).
11. **`GET /healthz`** → 200 `{"status": "ok"}`, porque a suíte aguarda essa rota antes de começar.
12. **Lock global** em abrir, encerrar e cancelar, porque requisições simultâneas não podem duplicar id nem abrir duas vezes a mesma placa.
13. **Versões fixas** no `requirements.txt` e lint com `ruff`, pois o SDLC do código gerado é avaliado.

## Containerfile

```dockerfile
FROM python:3.11-slim
WORKDIR /srv
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY app/ app/
EXPOSE 8080
CMD ["sh", "-c", "uvicorn app.main:app --host 0.0.0.0 --port ${PORT:-8080}"]
```