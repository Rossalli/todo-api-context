# Context — API de Lista de Tarefas

> Origem: Story "API REST para gerenciar tarefas"
> Última atualização: 2025-05-27

---

## 1. Requisitos Funcionais

| # | Operação | Endpoint sugerido | Detalhes |
|---|----------|-------------------|----------|
| RF-01 | Criar tarefa | `POST /tasks` | Título obrigatório; descrição opcional; status inicial = `pending` |
| RF-02 | Listar tarefas | `GET /tasks` | Suporta filtro por status via query param (`?status=pending` ou `?status=done`) |
| RF-03 | Marcar como concluída | `PATCH /tasks/{id}` | Altera status de `pending` → `done` |
| RF-04 | Remover tarefa | `DELETE /tasks/{id}` | Remove permanentemente; retorna 404 se id não existir |

### Modelo de dados — Tarefa

| Campo | Tipo | Obrigatoriedade | Observação |
|-------|------|-----------------|------------|
| `id` | string/int | Gerado pela API | Identificador único |
| `title` | string | Obrigatório | Não pode ser vazio |
| `description` | string | Opcional | Default `null` ou `""` |
| `status` | enum | Gerado pela API | Valores: `pending` \| `done` |
| `created_at` | datetime | Gerado pela API | Timestamp ISO 8601 (UTC) |

---

## 2. Restrições Técnicas

- **Linguagem / Framework:** Python 3.x + FastAPI.
- **Persistência:** Em memória (estrutura nativa Python — `dict` ou `list`); sem banco de dados nesta versão.
- **Formato de resposta:** JSON em todos os endpoints.
- **Códigos HTTP:**
  - `201 Created` → criação bem-sucedida.
  - `200 OK` → leitura, atualização e remoção bem-sucedidas.
  - `404 Not Found` → id inexistente em qualquer operação que referencie `{id}`.
  - `422 Unprocessable Entity` → validação de payload (FastAPI padrão via Pydantic).
- **Testes:** pytest cobrindo obrigatoriamente os quatro casos de uso (criar, listar, concluir, remover).
- **Sem autenticação** nesta versão.

---

## 3. Decisões em Aberto

| # | Decisão | Opções identificadas | Impacto |
|---|---------|----------------------|---------|
| DA-01 | Tipo do `id` | Auto-incremento inteiro × UUID v4 | UUIDs evitam colisão em merges futuros de listas, porém são mais verbosos |
| DA-02 | Comportamento do filtro `?status` sem valor | Retornar todas as tarefas × retornar erro 400 | Define contrato do `GET /tasks` |
| DA-03 | Retorno do `DELETE` | `204 No Content` (sem body) × `200 OK` com objeto removido | Consistência com clientes que esperam confirmação do item deletado |
| DA-04 | Idempotência do `PATCH` | Permitir chamar concluir em tarefa já concluída (no-op) × retornar `409 Conflict` | Comportamento esperado pelo consumidor precisa ser alinhado |
| DA-05 | Valores aceitos para `status` no filtro | Case-sensitive × case-insensitive | Afeta validação no query param |

---

## 4. Pontos de Atenção

- **Concorrência:** Armazenamento em memória sem locks — se houver múltiplos workers (ex.: `--workers N` no Uvicorn), cada processo terá sua própria cópia dos dados. Para esta versão, usar **um único worker** é suficiente; documentar a limitação.
- **Volatilidade dos dados:** Reiniciar o servidor apaga todas as tarefas. Deve estar explícito no README do projeto.
- **Validação de título vazio:** `title: ""` é tecnicamente um string válido para o Pydantic sem restrição adicional — verificar uso de `min_length=1` no schema.
- **Cobertura de testes:** A story exige cobertura dos quatro casos de uso; cenários de erro (id inexistente, payload inválido) não estão explicitados na story, mas são altamente recomendados para garantir os códigos HTTP corretos.
- **Escopo futuro:** A story menciona "primeira versão", sinalizando que persistência real (banco de dados) é esperada em iterações seguintes — manter a camada de repositório isolada para facilitar troca.
