# Constituição do Projeto — API de Lista de Tarefas

**Versão:** 1.0.0
**Ratificada em:** 2026-06-19
**Última emenda:** 2026-06-19

## Preâmbulo

Este documento estabelece os princípios fundamentais, restrições e processos de governança que regem o desenvolvimento da API de Lista de Tarefas. Toda especificação, plano técnico ou tarefa de implementação derivada deve estar em conformidade com os artigos aqui descritos. Em caso de conflito entre esta constituição e qualquer outro artefato (spec, plano, código), a constituição prevalece — emendas devem ser explícitas e versionadas.

---

## Artigo I — Simplicidade Acima de Tudo

A persistência desta versão é **exclusivamente em memória**, usando estruturas nativas do Python (`dict` ou `list`). Não é permitido introduzir banco de dados, ORM, cache externo ou qualquer dependência de persistência nesta fase. Toda solução deve favorecer a implementação mais simples que satisfaça os requisitos funcionais (RF-01 a RF-04).

*Justificativa:* a story original define explicitamente "sem banco de dados nesta versão". Adicionar complexidade não solicitada é uma violação deste princípio.

## Artigo II — Repositório Abstrato (Extensibilidade Futura)

Apesar do Artigo I, a camada de acesso a dados DEVE ser implementada atrás de uma interface/protocolo (ex.: `TarefaRepository` como `Protocol` ou classe abstrata). Os endpoints e a lógica de negócio não podem depender diretamente da estrutura de armazenamento concreta.

*Justificativa:* a story sinaliza evolução futura ("sem banco nesta primeira versão"). Isolar o contrato de persistência evita refatoração dos endpoints quando uma troca de storage ocorrer.

## Artigo III — Contrato de API em Português

Os nomes dos campos expostos pela API seguem a nomenclatura em **português**, consistente com o domínio de negócio: `id`, `titulo`, `descricao`, `status`, `data_criacao`. Os valores aceitos do campo `status` são exclusivamente `"pendente"` e `"concluída"`, tanto na entrada quanto na saída.

*Ratifica DA-02 → português.*
*Justificativa:* domínio e usuários falam português; manter o vocabulário do RF original reduz tradução mental e mantém linguagem ubíqua entre story, código e contrato.

## Artigo IV — Identificadores como UUID v4

O campo `id` de cada tarefa é um **UUID v4**, gerado no momento da criação e representado como string na serialização JSON.

*Ratifica DA-01 → UUID v4.*
*Justificativa:* evita colisões e suposição de ordenação por parte do cliente, independe de contador global mutável (relevante dado o Artigo I) e facilita eventual migração para storage distribuído.

## Artigo V — Ações Explícitas para Transições de Estado

A conclusão de uma tarefa é exposta como ação explícita: `POST /tarefas/{id}/concluir`. Não é permitido um endpoint genérico de atualização parcial (`PATCH`) que aceite qualquer campo nesta versão.

*Ratifica DA-03 → ação explícita.*
*Justificativa:* expressa a intenção de negócio sem ambiguidade, evita estados inválidos enviados por engano e simplifica a validação.

## Artigo VI — Filtros via Query Parameters

A listagem de tarefas (RF-02) aceita filtro por status exclusivamente via query parameter: `GET /tarefas?status=pendente`. Não são criados endpoints dedicados por status.

*Ratifica DA-04 → query param.*
*Justificativa:* mantém um único recurso (`/tarefas`) e segue convenção REST, evitando explosão de rotas.

## Artigo VII — Data e Hora em UTC

O campo `data_criacao` é sempre gerado e serializado em **UTC**, formato ISO 8601 com sufixo `Z` (ex.: `2026-06-19T14:32:00Z`). Conversão para fuso local é responsabilidade do cliente.

*Ratifica DA-05 → UTC fixo.*
*Justificativa:* evita ambiguidade de fuso horário no servidor e é prática padrão para APIs.

## Artigo VIII — Idempotência da Conclusão

Marcar como concluída uma tarefa já concluída é uma operação **idempotente**: retorna `200 OK` com a tarefa inalterada, sem erro e sem efeito colateral adicional.

*Resolve o ponto de atenção "Idempotência" do contexto técnico.*
*Justificativa:* alinhado à semântica REST para ações que representam estado-alvo; evita que o cliente precise checar o estado atual antes de agir.

## Artigo IX — Erros Semânticos e Previsíveis

- `404 Not Found` sempre que um `id` informado não corresponder a nenhuma tarefa existente, em qualquer operação.
- `422 Unprocessable Entity` para qualquer violação de validação de schema, incluindo título ausente/vazio e `status` fora do enum (`pendente`, `concluída`).
- Toda resposta de erro segue o formato padrão do FastAPI (`{"detail": ...}`), sem exposição de stack traces.

## Artigo X — Testabilidade Obrigatória

Todo requisito funcional (RF-01 a RF-04) deve ter ao menos um teste automatizado com `pytest` e `TestClient` do FastAPI. Os testes são independentes entre si: o estado em memória é resetado via fixture antes de cada teste, sem ordem implícita de execução.

## Artigo XI — Transparência sobre Efemeridade dos Dados

Esta versão **não garante persistência entre reinicializações do processo**. É uma decisão de produto desta fase, não um defeito, e deve estar documentada de forma visível (README, descrição do OpenAPI/Swagger) para qualquer consumidor da API.

## Artigo XII — Execução em Processo Único

Em desenvolvimento e nesta versão do produto, a API DEVE ser executada com um único worker (`uvicorn ... --workers 1`). Múltiplos workers não compartilham o estado em memória, gerando inconsistência visível ao usuário. Escalabilidade horizontal exige primeiro substituir o repositório (Artigo II) por um backend compartilhado.

---

## Restrições Tecnológicas (Não Negociáveis)

| Camada | Escolha | Observação |
|---|---|---|
| Linguagem | Python 3.x | — |
| Framework web | FastAPI | Validação via Pydantic |
| Persistência | Estrutura nativa em memória | `dict`/`list`, atrás de repositório (Artigo II) |
| Testes | pytest + `TestClient` | Isolamento obrigatório (Artigo X) |
| Execução | Uvicorn, single worker | Artigo XII |

## Processo de Governança

1. Esta constituição é a fonte de verdade para decisões de design já tomadas. Specs, planos técnicos e tarefas de implementação não podem contradizê-la silenciosamente.
2. Toda emenda (mudança de um Artigo) exige: (a) registro explícito do motivo, (b) incremento de versão semântica, (c) atualização da data de última emenda.
3. Mudanças que afetam contrato de API já implementado (ex.: trocar `id` de UUID para inteiro) são tratadas como **breaking changes** e exigem versionamento explícito (ex.: `/v2`).
4. Pontos não cobertos por nenhum Artigo permanecem como decisão de implementação do time, desde que não violem os Artigos I e II.

---

*Todas as decisões em aberto originais (DA-01 a DA-05) foram ratificadas nesta versão (Artigos III a VII). Qualquer revisão deve ocorrer via emenda formal, não por desvio silencioso durante a implementação.*
