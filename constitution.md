# Constitution — Regras persistentes do projeto

- Regra 0: Se encontrar divergência seguir na seguinte ordem: constitution, spec, plan, tests, tasks. Depois seguir o tasks.md até o fim.


1. Rotas exatas : `POST /bilhetes`, `POST /bilhetes/{id}/encerramento`, `POST /bilhetes/{id}/cancelamento`, `GET /bilhetes/ativos`, `GET /bilhetes?placa=`, `GET /relatorios/diario?data=`. Nunca criar rotas, campos ou regras fora do spec.md.
2. Stack: Python 3.11 + FastAPI; persistência em memória (dict); IDs inteiros sequenciais começando em 1.
3. Dependências: somente `fastapi`, `uvicorn`, `pydantic` v2, `pytest`, `httpx` e `ruff`, com versões fixas no `requirements.txt`.
4. Dinheiro sempre em centavos inteiros (`int`). Nunca usar float em cálculo nem em resposta.
5. Datas sempre em ISO-8601 com fuso `-03:00`.
6. Parâmetros fixos da variante: `TARIFA_HORA_CENTAVOS=450`, `FRACAO_MINUTOS=15`, `TETO_DIARIO_CENTAVOS=8000`, `TOLERANCIA_MINUTOS=15`. Nenhuma variável de ambiente é obrigatória.
7. Dentro do container, o servidor deve escutar em `0.0.0.0`, porta **8080** (a env `PORT` só sobrescreve em testes locais). A suíte executa `docker run -p 8002:8080`, então a API é acessada em `http://localhost:8002`. O projeto deve ter `Containerfile`, `requirements.txt` e `README.md`.
8. Testes com pytest; cada caso do tests.md vira uma função de teste. Nunca alterar teste para fazê-lo passar.
9. Todo o código fica em `app/`, seguindo a PEP 8. Campos JSON em snake_case, exatamente como no spec.md.
10. Todo erro deve retornar o corpo `{"erro": "<codigo>"}`: 422 para formato inválido (placa, entrada, data), 404 para bilhete inexistente, 409 para conflito de estado. O handler padrão do FastAPI (`{"detail": ...}`) deve ser substituído.
11. Validação de formato (422) sempre vem antes de regra de negócio (409). O exemplo `{"id": 7, "valor": 12.50}` contradiz o contrato e nunca deve ser seguido.
12. `GET /healthz` deve sempre responder 200 `{"status": "ok"}`; a suíte espera por ele antes de testar. É a única rota permitida além das do spec.md.
13. Abrir, encerrar e cancelar devem usar um `threading.Lock`: requisições simultâneas nunca podem gerar id duplicado nem dois bilhetes abertos para a mesma placa.