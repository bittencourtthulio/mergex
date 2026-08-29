# Patch de integração — sprintx × mergex

Prompt pronto para colar no repositório da skill `sprintx`. Ele altera a `sprintx` para acionar a `mergex` nos pontos corretos e para expor os artefatos que ela consome.

Trabalhe de forma autônoma até o fim: não faça perguntas, não peça autorização para editar arquivos, não pare no meio.

---

## PARTE 1 — O CONTRATO

A `mergex` é a skill do ecossistema Expx que cuida do versionamento e da entrega: cria a branch quando o trabalho começa, commita cada task concluída, verifica a prontidão, classifica o diff por atenção humana, monta a descrição do pull request e o pacote do QA, sobe a branch e abre o PR.

A `sprintx` planeja e executa features novas em seis fases (F1–F6). O código é escrito na **F6 EXECUÇÃO**. É lá, e só lá, que a `mergex` entra.

### As três regras deste contrato

**1. A `sprintx` aciona, a `mergex` executa.** A `sprintx` não versiona nada por conta própria: ela chama a `mergex` no ponto certo e devolve o controle. Nenhum comando de versionamento novo entra na `sprintx`.

**2. Ausência da `mergex` nunca quebra a `sprintx`.** Toda a integração é condicional à presença de `.claude/skills/mergex/SKILL.md`. Sem ela, a F6 roda exatamente como hoje: executa as tasks, marca os status, roda a suíte, entrega o relatório final. Nenhuma branch, nenhum commit, nenhum erro, nenhum aviso ruidoso.

**3. O comando de revisão NUNCA é encadeado.** A `sprintx` não menciona, não sugere e não aciona `/mergex-revisar` em lugar nenhum — nem no relatório final da F6, nem como próximo passo, nem como dica. Integrar código é decisão humana.

### Os pontos de acionamento

| Momento na sprintx | Etapa da mergex |
|---|---|
| Início da F6, depois de ler o `ORQUESTRADOR.md` e **antes da primeira task** | **E0** — abre `feature/<slug>` |
| Ao fechar **cada** task (passo 8 do TDD: status `concluida`, suíte verde) | **E1** — um commit para aquela task |
| Fim da F6 (todas as tasks executadas, ou nada mais executável) | **E2 → E8** — portão, classificação, PR, pacote de QA, push, abertura, registro |

---

## PARTE 2 — O QUE ALTERAR

### 2.1 `references/06-execucao.md` — três inserções

**(a) Nos pré-requisitos, antes do Passo 1.** Acrescente:

> ### Abertura do trabalho no repositório
>
> Se `.claude/skills/mergex/SKILL.md` existir, acione a **etapa E0 da `mergex`** antes da primeira task, depois de ler o `ORQUESTRADOR.md`. Ela verifica se o repositório é versionado, exige árvore limpa, determina a branch base e cria `feature/<slug>` — retomando a branch se ela já existir.
>
> A ordem importa: a árvore precisa estar limpa quando a branch é criada, e depois da primeira task ela não estará.
>
> Se a `mergex` avisar que há alteração não commitada pendente, **pare a F6** e repasse o aviso: não se começa a executar por cima de trabalho não salvo de alguém.
>
> Sem a `mergex` instalada, siga direto para o Passo 1.

**(b) No Passo 2, imediatamente depois do passo 8 do TDD** (o que marca `status: concluida`). Acrescente como passo 9:

> 9. Se `.claude/skills/mergex/SKILL.md` existir, acione a **etapa E1 da `mergex`** para esta task. Ela commita **apenas os arquivos declarados** em `arquivos`, com a mensagem que traz o objetivo e os testes da task, depois de varrer o diff em busca de segredo, credencial e dado real de cliente.
>
> A ordem importa: o commit registra a task **já marcada** como concluída, e é o `tasks.md` atualizado que dá à mensagem o objetivo e os testes.
>
> Se a `mergex` abortar o commit por suspeita de segredo, **não contorne**: a task fica sem commit, o aviso vai para o relatório final, e o portão de prontidão vai barrá-la depois.
>
> Task marcada `bloqueada` não gera commit.

**(c) No Passo 3 (relatório de encerramento), antes de montar o relatório.** Acrescente:

> ### Entrega
>
> Se `.claude/skills/mergex/SKILL.md` existir, acione as **etapas E2 a E8 da `mergex`**, nesta ordem: portão de prontidão, classificação da atenção humana, descrição do pull request, pacote de QA, push, abertura do PR e registro da entrega.
>
> Se o portão devolver `BLOQUEADO`, **inclua no relatório final o que ele apontou** e não tente contornar: a `mergex` barra e explica, nunca maquia. Achado de auditoria ALTA em aberto na F5 faz o portão barrar.
>
> Acrescente ao relatório final uma seção **Entrega** com: a branch, a quantidade de commits, o resultado do portão, a contagem das três faixas de atenção, e a URL do pull request (ou o caminho de `PR.md`, quando a ferramenta do serviço não estiver disponível).
>
> **Não sugira o merge e não mencione `/mergex-revisar`.** A revisão e a integração são manuais e só rodam quando o desenvolvedor as chama pelo nome.

### 2.2 `SKILL.md` — duas alterações

**(a) Na tabela "Fases → arquivos da skill"**, na linha da F6, acrescente à coluna de roteiro:

> `references/06-execucao.md` — aciona a `mergex` (E0, E1, E2–E8) quando ela estiver instalada

**(b) Numa seção nova, ao fim, antes das regras invioláveis:**

> ## Entrega no repositório
>
> A `sprintx` termina com o código escrito e os testes verdes. Levar isso até o repositório e até o revisor é trabalho da [`mergex`](https://github.com/bittencourtthulio/mergex): ela abre a branch no início da F6, commita cada task que fecha, e ao fim monta a entrega — portão de prontidão, classificação do diff por atenção humana, descrição do pull request e pacote para o QA.
>
> A integração é condicional: sem a `mergex` instalada, a F6 roda exatamente como sempre.

### 2.3 O que NÃO alterar

- **Nenhuma regra inviolável da `sprintx`.** Nenhuma delas muda.
- **A máquina de estados.** A `mergex` não é uma fase e não entra na tabela de detecção de fase.
- **O contrato da task.** Nenhum campo novo. A `mergex` lê o que já existe.
- **As fases F1 a F5.** Antes da F6 não há código escrito.
- **O `expx-schema v1`.** A `mergex` grava seu próprio `kind: entrega` em `docs/entregas/`, fora das pastas da `sprintx`.
- **A única escrita da `mergex` em artefato da `sprintx`** é uma linha no `ORQUESTRADOR.md` registrando a branch. Nada mais.

---

## PARTE 3 — VERIFICAÇÃO

1. Grep por `mergex` em toda a skill: as menções aparecem **apenas** em `references/06-execucao.md` e no `SKILL.md`, nos pontos descritos.
2. Grep por `mergex-revisar`, `revisar`, `merge` e `integrar`: **nenhuma** menção ao comando de revisão em nenhum arquivo. Ele não pode ser encadeado nem sugerido.
3. Toda menção à `mergex` está condicionada à existência de `.claude/skills/mergex/SKILL.md`.
4. Simule a F6 **sem** a `mergex` instalada: a fase roda de ponta a ponta, executa as tasks, entrega o relatório final, e nenhum passo falha nem gera aviso ruidoso.
5. Simule a F6 **com** a `mergex`: E0 antes da primeira task, E1 depois de cada `status: concluida`, E2–E8 antes do relatório final.
6. Simule árvore suja no início da F6: a F6 para e repassa o aviso, sem criar branch e sem executar task.
7. Simule o portão devolvendo `BLOQUEADO`: o relatório final traz o que falta e nada é contornado.
8. Confirme que nenhuma regra inviolável da `sprintx` foi alterada, e que nenhum campo do contrato da task foi acrescentado.
9. Grep por caminho absoluto nos trechos acrescentados.

---

## ENTREGA

Ao terminar, mostre:

- o diff de cada arquivo alterado;
- o resultado das verificações 1 a 9;
- confirmação explícita de que `/mergex-revisar` não é mencionado em lugar nenhum.
