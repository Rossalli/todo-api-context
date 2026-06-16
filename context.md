# Contexto Técnico — API de Lista de Tarefas

> Artefato gerado a partir da story: *"Como usuário, quero uma API REST para gerenciar minhas tarefas"*

---

## 1. Requisitos Funcionais

| # | Operação | Detalhes |
|---|----------|----------|
| RF-01 | **Criar tarefa** | Título obrigatório; descrição opcional. Retorna a tarefa criada com `id` gerado e `status = pendente`. |
| RF-02 | **Listar tarefas** | Retorna todas as tarefas. Aceita filtro por `status` (`pendente` / `concluída`). |
| RF-03 | **Marcar como concluída** | Atualiza o `status` de uma tarefa existente para `concluída` a partir do `id`. |
| RF-04 | **Remover tarefa** | Exclui uma tarefa existente a partir do `id`. |

### Modelo de dados da tarefa

| Campo | Tipo | Observação |
|-------|------|------------|
| `id` | inteiro ou UUID | Gerado automaticamente na criação |
| `titulo` | string | Obrigatório |
| `descricao` | string | Opcional; pode ser `null` |
| `status` | enum `pendente` / `concluída` | Valor padrão: `pendente` |
| `data_criacao` | datetime (ISO 8601) | Gerado automaticamente na criação |

---

## 2. Restrições Técnicas

- **Linguagem / framework:** Python 3.x + FastAPI.
- **Persistência:** exclusivamente em memória (estrutura nativa Python, ex.: `dict` ou `list`); sem banco de dados nesta versão.
- **Formato de resposta:** JSON em todos os endpoints.
- **Códigos HTTP de erro:**
  - `404 Not Found` para `id` inexistente em qualquer operação que o exija.
  - Demais erros (ex.: campo obrigatório ausente) devem retornar o código semanticamente correto (`422 Unprocessable Entity` via validação automática do FastAPI/Pydantic).
- **Testes:** pytest cobrindo os quatro casos de uso (RF-01 a RF-04); sem restrição de cobertura mínima declarada na story, mas todos os fluxos principais devem ter ao menos um teste.

---

## 3. Decisões em Aberto

| ID | Ponto | Opções mapeadas | Quem decide |
|----|-------|-----------------|-------------|
| DA-01 | **Tipo do `id`** | Inteiro auto-incremental *vs.* UUID v4 | Time / Tech Lead |
| DA-02 | **Nome dos campos na API** | Português (`titulo`, `descricao`) *vs.* inglês (`title`, `description`) | Time |
| DA-03 | **Endpoint de "marcar como concluída"** | `PATCH /tarefas/{id}` com body `{"status": "concluída"}` *vs.* `POST /tarefas/{id}/concluir` (ação explícita) | Time |
| DA-04 | **Filtro de status na listagem** | Query param `?status=pendente` *vs.* endpoints separados (`/tarefas/pendentes`, `/tarefas/concluidas`) | Time |
| DA-05 | **Formato de `data_criacao`** | UTC fixo *vs.* fuso do servidor | Time |

---

## 4. Pontos de Atenção

- **Concorrência:** persistência em memória não é thread-safe; se o servidor FastAPI for iniciado com múltiplos workers (Uvicorn/Gunicorn), cada processo terá seu próprio estado isolado. Deve-se usar um único worker em dev ou garantir lock/shared state se necessário.
- **Perda de dados:** ao reiniciar o processo, todas as tarefas são apagadas. Isso é esperado nesta versão, mas deve ser comunicado claramente ao usuário/produto.
- **Validação de `status`:** o campo deve aceitar apenas os valores `pendente` e `concluída`; qualquer outro valor deve ser rejeitado com erro explícito (`422`).
- **Idempotência:** tentar marcar como concluída uma tarefa já concluída deve ser definido — retornar `200` sem efeito colateral ou `409 Conflict`. Não está especificado na story.
- **Testes:** como o estado vive em memória, os testes precisam de isolamento entre si (setup/teardown do estado global); usar `TestClient` do FastAPI com fixture de reset do repositório.
- **Escopo futuro:** a story menciona "sem banco nesta primeira versão", indicando evolução esperada; projetar o repositório de dados atrás de uma interface/protocolo facilita a troca posterior sem refatoração dos endpoints.
