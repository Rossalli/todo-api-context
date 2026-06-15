# context.md — API de Lista de Tarefas

> Artefato gerado a partir da story. Serve como referência técnica para implementação e revisão.

---

## 1. Requisitos Funcionais

| # | Operação | Detalhes |
|---|----------|----------|
| RF-01 | **Criar tarefa** | Recebe `título` (obrigatório) e `descrição` (opcional). Retorna a tarefa criada com `id` gerado, `status` inicial `pendente` e `data_criacao`. |
| RF-02 | **Listar tarefas** | Retorna todas as tarefas. Aceita filtro opcional por `status` (`pendente` \| `concluída`). |
| RF-03 | **Marcar como concluída** | Atualiza o `status` de uma tarefa existente para `concluída` pelo seu `id`. |
| RF-04 | **Remover tarefa** | Exclui permanentemente uma tarefa pelo `id`. |

### Modelo de dados — Tarefa

| Campo | Tipo | Obrigatório | Observação |
|-------|------|-------------|------------|
| `id` | string / int | — | Gerado pelo sistema na criação |
| `titulo` | string | ✅ | Não pode ser vazio |
| `descricao` | string | ❌ | Pode ser `null` |
| `status` | enum | — | Valores: `pendente`, `concluída` |
| `data_criacao` | datetime | — | Definida pelo sistema na criação |

---

## 2. Restrições Técnicas

| Categoria | Decisão |
|-----------|---------|
| **Linguagem** | Python (versão mínima não especificada na story — ver Decisões em Aberto) |
| **Framework web** | FastAPI |
| **Persistência** | Em memória (`dict` ou `list` em variável de módulo); sem banco de dados nesta versão |
| **Formato de resposta** | JSON em todos os endpoints |
| **Códigos HTTP de erro** | `404 Not Found` para `id` inexistente; demais erros devem usar código semântico adequado (ex.: `422` para payload inválido, nativo do FastAPI/Pydantic) |
| **Testes** | `pytest`; cobertura obrigatória dos quatro casos de uso (RF-01 a RF-04) |
| **Banco de dados** | Explicitamente fora do escopo desta versão |

---

## 3. Decisões em Aberto

| ID | Questão | Impacto | Sugestão de encaminhamento |
|----|---------|---------|---------------------------|
| DA-01 | **Versão mínima do Python** | Pode afetar sintaxe de type hints e recursos do FastAPI | Alinhar com o time; sugerir ≥ 3.11 |
| DA-02 | **Tipo do `id`** | `int` sequencial (simples, mas sensível a concorrência) vs `UUID` (mais robusto) | Definir antes de implementar o modelo Pydantic |
| DA-03 | **Operação de "marcar como concluída"** | `PATCH /tasks/{id}` (atualização parcial) vs `PUT /tasks/{id}` (substituição completa) | Preferir `PATCH` por semântica REST; confirmar com PO se edição de outros campos também será necessária |
| DA-04 | **Endpoint de remoção retorna corpo?** | `204 No Content` (sem corpo) vs `200 OK` com a tarefa removida | Definir contrato para facilitar integração de clientes |
| DA-05 | **Filtro de listagem via query param** | Nome e valores exatos do parâmetro (ex.: `?status=pendente` vs `?filter=pending`) | Padronizar antes de gerar a collection/documentação |
| DA-06 | **Autenticação / autorização** | Story não menciona; API está aberta | Confirmar se é fora de escopo ou pós-MVP |
| DA-07 | **Concorrência na persistência em memória** | Múltiplas requisições simultâneas podem corromper o estado | Avaliar uso de `asyncio.Lock` ou aceitar limitação como conhecida para esta versão |

---

## 4. Pontos de Atenção

| # | Ponto | Risco |
|---|-------|-------|
| PA-01 | **Persistência volátil** | Todos os dados são perdidos ao reiniciar o processo; qualquer teste de carga ou demo longa pode gerar confusão se não documentado claramente. |
| PA-02 | **Validação do `título`** | FastAPI/Pydantic rejeita campo ausente, mas string vazia `""` passa validação padrão. É necessário adicionar `min_length=1` explicitamente no schema. |
| PA-03 | **Enum de `status`** | Usar `enum.Enum` do Python (ou `Literal` do Pydantic) garante contrato estrito; aceitar string livre abre margem a inconsistências. |
| PA-04 | **Cobertura de testes** | A story exige cobertura dos quatro casos de uso, mas não define cobertura de cenários negativos (ex.: criar sem título, buscar id inexistente). Recomendado incluir para garantir robustez. |
| PA-05 | **Documentação automática** | FastAPI gera `/docs` (Swagger) e `/redoc` por padrão; definir `title`, `version` e `description` na instância `FastAPI()` evita docs genéricas. |
| PA-06 | **Geração do `id`** | Em ambiente de produção com múltiplas instâncias, `id` sequencial em memória não é seguro. Documentar essa limitação no README. |
