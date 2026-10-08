# Tasks — Decomposição

- [ ] T1 — Estrutura: pasta `app/`, `requirements.txt`, `Containerfile` (porta 8002) e `README.md` (rodar local e em container).
- [ ] T2 — `store.py`: bilhetes em memória, ids sequenciais a partir de 1, `reset()`.
- [ ] T3 — Handlers de erro: todo erro no formato `{"erro": "<codigo>"}`; 422 antes de 409.
- [ ] T4 — UC1 + UC8: abrir bilhete (placa `^[A-Z0-9]{7}$`, `entrada` opcional) e bloquear placa com bilhete aberto.
- [ ] T5 — `pricing.py` (UC2 + UC7): tolerância, frações, arredondamento e teto conforme plan.md.
- [ ] T6 — UC2 + UC5: encerrar e cancelar, com 404/409.
- [ ] T7 — UC3 + UC6: listar ativos e histórico por placa, ordenados por `entrada` decrescente.
- [ ] T8 — UC4: relatório diário com média arredondando 0,5 para cima.
- [ ] T9 — Testes pytest: um teste por linha do tests.md; rodar até todos passarem.