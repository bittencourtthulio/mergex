# E8 — REGISTRO DA ENTREGA

Você está no E8, a última etapa do fluxo automático. Aqui você fecha `docs/entregas/<trabalho_id>/ENTREGA.md`.

Este arquivo é lido pelo **expx-panel** para mostrar o que aguarda revisão. Prosa não é contrato: o frontmatter é a interface com a máquina, a prosa abaixo dele é para a pessoa.

O `ENTREGA.md` **não substitui** o registro que a runx faz em `docs/relatorios/`. A mergex entrega; a runx fecha a ocorrência (regra 19).

## Quando o arquivo é escrito

Três vezes ao longo do trabalho, sempre no mesmo arquivo:

| Momento | Etapa | Estado |
|---|---|---|
| Abertura da branch | E0 | `aberto` |
| A cada task commitada | E1 | `aberto`, com a lista `commits` crescendo |
| Fim do fluxo | E8 | `entregue` ou `bloqueado` |

## O contrato — `kind: entrega`

Segue o `expx-schema v1`, o mesmo contrato da sprintx e da runx. As regras universais valem todas:

1. O bloco YAML é a primeira coisa do arquivo, delimitado por `---` antes e depois.
2. Toda chave em `snake_case`, minúscula, sem acento.
3. Todo valor de enum em minúscula e sem acento.
4. Datas em ISO `AAAA-MM-DD`, obtidas com `date +%Y-%m-%d` do sistema, **nunca de memória**.
5. Booleanos `true` / `false`, sem aspas.
6. Lista vazia é `[]`, valor ausente é `null`. **NUNCA omita a chave** — o painel diferencia "não se aplica" de "esqueceram de escrever".
7. Campos de texto no YAML são de **uma linha**.
8. `atualizado_em` é reescrito a cada gravação.
9. Nenhum caminho absoluto em nenhum valor.

### Enums próprios deste kind

| Enum | Valores |
|---|---|
| `estado` | `aberto` \| `entregue` \| `bloqueado` |
| `portao` | `pronto` \| `bloqueado` \| `null` (ainda não rodou) |
| `pr_estado` | `rascunho` \| `aberto` \| `merged` \| `fechado` \| `null` |
| `tipo_trabalho` | `feature` \| `ocorrencia` |
| `raio` | `baixo` \| `medio` \| `alto` \| `null` (sem modo legado) |

`expx_tool` fica como a skill de **origem** do trabalho (`sprintx` ou `runx`) — é ela que governa o trabalho. O campo `entregue_por: mergex` diz quem gravou este arquivo.

### O bloco completo

```yaml
---
expx_schema: 1
expx_tool: runx
kind: entrega
trabalho_id: OC-2026-0184-icms-st-base-desconto
entregue_por: mergex
titulo: Corrigir base de calculo do ICMS-ST com desconto incondicional
tipo_trabalho: ocorrencia
tipo_ocorrencia: regra-de-calculo
estado: entregue
versionado: true
branch: fix/OC-2026-0184-icms-st-base-desconto
branch_base: main
commits:
  - task: T-01.01
    commit: a3f19c2
  - task: T-01.02
    commit: 7b2e401
raio: alto
atencao:
  olho_obrigatorio: 3
  leitura_rapida: 2
  dispensavel: 4
portao: pronto
desvios: []
push_feito: true
pr_url: https://github.com/<org>/<repo>/pull/482
pr_estado: rascunho
criado_em: 2026-08-27
atualizado_em: 2026-08-29
entregue_em: 2026-08-29
---
```

### Campo a campo

| Campo | Regra |
|---|---|
| `trabalho_id` | O slug da sprintx ou o `<OC-ID>-<slug>` da runx. O mesmo da pasta de origem |
| `entregue_por` | Sempre `mergex` |
| `tipo_ocorrencia` | O tipo da runx; `null` quando `tipo_trabalho: feature` |
| `estado` | `aberto` no E0; `entregue` quando o fluxo completou; `bloqueado` quando o portão barrou |
| `versionado` | `false` em repositório sem versionador |
| `branch`, `branch_base` | `null` quando `versionado: false` |
| `commits` | Um item por task commitada, na ordem em que fecharam; `[]` sem versionador |
| `raio` | A faixa da legadox; `null` sem modo legado — **nunca invente uma faixa** |
| `atencao` | As três contagens do E3; zeros quando o E3 não rodou |
| `portao` | O resultado do E2 |
| `desvios` | Arquivos alterados fora da lista declarada, detectados no E1 e no E2; `[]` quando não houve |
| `push_feito` | `true` só quando o E6 confirmou que o remoto tem o mesmo commit |
| `pr_url` | A URL devolvida pelo E7; `null` quando o PR não foi aberto — **não é falha** |
| `pr_estado` | `rascunho` na abertura normal; `aberto` quando o QA já aprovou; `merged`/`fechado` quando o E9 ou uma pessoa atualizarem |
| `entregue_em` | A data em que o fluxo completou; `null` enquanto `estado` não for `entregue` |

## A prosa

Abaixo do frontmatter, use `assets/TEMPLATE-ENTREGA.md`. A prosa é para quem abre o arquivo:

1. **Resumo em uma linha** — o que foi entregue e onde está.
2. **Onde está o quê** — links relativos para `PR.md`, `QA-PACOTE.md`, `ATENCAO.md` e para a pasta do trabalho de origem.
3. **Estado da entrega** — portão, push, PR, e o que falta para o merge.
4. **Avisos** — insumos ausentes acumulados pelas etapas: sem raio, sem roteiro manual, sem `DIVIDA.md`, ferramenta de PR ausente. É a lista do que faltou, e é ela que a Parte de entrega apresenta ao usuário.
5. **Desvios**, se houver — arquivos fora da lista declarada.

**O YAML e a prosa andam juntos.** Nunca atualize um sem o outro.

## Entrega ao usuário

Terminado o E8, apresente na tela, curto:

```
mergex — entrega concluída

Trabalho: <trabalho_id>
Branch: <branch> → <branch_base>
Commits: <n> (um por task)
Portão: PRONTO
Atenção humana: <x> olho obrigatório, <y> leitura rápida, <z> dispensável
Push: feito
PR: <url> (rascunho) | não aberto — descrição em docs/entregas/<trabalho_id>/PR.md
Pacote de QA: docs/entregas/<trabalho_id>/QA-PACOTE.md

Avisos: <lista, ou "nenhum">
```

**Não sugira o merge. Não sugira `/mergex-revisar`.** O comando de revisão nunca é encadeado nem oferecido ao fim de um trabalho (regra 16): quem decide revisar e integrar é o desenvolvedor, quando ele quiser, chamando o comando pelo nome.

## Critério de saída

- [ ] `ENTREGA.md` tem frontmatter válido, com o cabeçalho comum e nenhuma chave omitida.
- [ ] `estado` reflete o que aconteceu de verdade.
- [ ] Datas em ISO, obtidas do sistema.
- [ ] Nenhum caminho absoluto.
- [ ] A prosa bate com o YAML.
- [ ] Os avisos acumulados estão listados.
- [ ] Nada foi sugerido sobre merge ou revisão.

## Quando falha

| Situação | O que fazer |
|---|---|
| Portão barrou (E2) | `estado: bloqueado`, `portao: bloqueado`, com o que falta na prosa; o resto fica `null`/`false` |
| Sem versionador | `versionado: false`, `branch`/`branch_base`/`pr_url` `null`, `commits: []`; `estado: entregue` mesmo assim |
| PR não aberto | `pr_url: null`, `pr_estado: null`, aviso na prosa; `estado` continua `entregue` |
| Valor não determinável | `null` ou `[]`, **nunca invente**, nunca omita a chave |
| Arquivo já existe do E0 | Atualize; nunca recrie do zero, nunca apague o histórico de `commits` |
