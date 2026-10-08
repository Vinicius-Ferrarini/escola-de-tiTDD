# Constitution — Regras persistentes do projeto

- Regra 0: Se encontrar divergência seguir na seguinte ordem: constitution, spec, plan, tests, tasks. Depois seguir o tasks.md até o fim.


1. Rotas exatas : `POST /bilhetes`, `POST /bilhetes/{id}/encerramento`, `POST /bilhetes/{id}/cancelamento`, `GET /bilhetes/ativos`, `GET /bilhetes?placa=`, `GET /relatorios/diario?data=`. Nunca criar rotas, campos ou regras fora do spec.md.
2. Stack: Python 3.11 + FastAPI; persistência em memória (dict); IDs inteiros sequenciais começando em 1.
3. Dependências: somente `fastapi`, `uvicorn`, `pydantic` v2, `pytest` e `httpx`.
4. Dinheiro sempre em centavos inteiros (`int`). Nunca usar float em cálculo nem em resposta.
5. Datas sempre em ISO-8601 com fuso `-03:00`.
6. Parâmetros fixos da variante: `TARIFA_HORA_CENTAVOS=450`, `FRACAO_MINUTOS=15`, `TETO_DIARIO_CENTAVOS=8000`, `TOLERANCIA_MINUTOS=15`. Nenhuma variável de ambiente é obrigatória.
7. O servidor deve subir em `0.0.0.0`, porta **8000**. O projeto deve ter `Containerfile`, `requirements.txt` e `README.md`.
8. Testes com pytest; cada caso do tests.md vira uma função de teste. Nunca alterar teste para fazê-lo passar.
9. Todo o código fica em `app/`, seguindo a PEP 8, exceto pelas respostas da api que deve sempre ser em camelCase formato json