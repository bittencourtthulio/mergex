# E3 — CLASSIFICAÇÃO DA ATENÇÃO HUMANA

Você está no E3, o coração da skill. Aqui você pega o diff inteiro e diz, **arquivo por arquivo**, onde o revisor humano precisa gastar atenção e onde não precisa.

O revisor é recurso caro e finito. Um diff de 40 arquivos sem classificação faz uma de duas coisas, as duas ruins: ou ele lê tudo por igual e não sobra energia para o que importa, ou ele bate o olho em tudo por igual e o que importa passa.

**A classificação é derivada de evidência registrada** — raio, zonas de risco, cobertura de teste, tipo de mudança, artefatos das skills irmãs. **Nunca de sensação, nunca de tamanho** (regra 8).

## Pré-requisitos verificáveis

- O E2 devolveu `PRONTO`. Se devolveu `BLOQUEADO`, o E3 não roda.
- Existe um diff para classificar. Sem versionador, classifique os arquivos declarados nas tasks — a lógica é a mesma.

## Passo 1 — Levantar o diff

```
git diff --name-status <branch-base>...HEAD
```

Use `...` (três pontos): você quer o que **esta branch** fez desde que divergiu da base, não as diferenças acumuladas da base desde então.

Para cada arquivo, guarde: caminho, tipo de mudança (`A` adicionado, `M` modificado, `D` removido, `R` renomeado) e o tamanho (`git diff --numstat`) — o tamanho entra no relatório como **informação**, nunca como critério.

Nenhum arquivo do diff fica de fora da classificação. Todo arquivo cai em **exatamente uma** faixa.

## Passo 2 — Reunir as evidências

Antes de classificar qualquer coisa, carregue as fontes. Cada uma responde a uma pergunta diferente:

| Fonte | Pergunta que responde | Onde |
|---|---|---|
| `PERFIL.md` da legadox | Este arquivo está em zona de risco declarada? | `docs/legado/PERFIL.md` |
| Raio de impacto | Este arquivo veio de raio ALTO, MÉDIO ou BAIXO? | artefato de raio da legadox |
| Testes de caracterização | Este arquivo está congelado por caracterização que continua passando? | artefatos da legadox |
| Plano de reversão | Este arquivo tem efeito irreversível declarado? | artefato de reversão da legadox |
| `tasks.md` | Que task tocou este arquivo? Quais testes a cobrem? | sprintx/runx |
| `01-CAUSA-RAIZ.md` | Este arquivo é onde a causa foi comprovada? | runx |
| Relatório de cobertura | Este arquivo tinha e tem cobertura de teste? | ferramenta do repositório |
| `CONVENCOES.md` da stackx | Que arquivos este repositório gera automaticamente? | `docs/stack/CONVENCOES.md` |

**Fonte ausente não vira suposição.** Se não existe `PERFIL.md`, você não sabe se o arquivo está em zona de risco — e então os outros critérios decidem. Registre a fonte ausente como aviso na saída: o revisor precisa saber que aquela dimensão não foi avaliada.

## Passo 3 — Classificar, arquivo por arquivo

Aplique **nesta ordem**. Pare no primeiro critério que bater: a ordem já é a da rigidez.

### OLHO OBRIGATÓRIO — o revisor lê linha a linha

Basta **um** destes:

| # | Critério | Evidência que o confirma |
|---|---|---|
| O1 | Arquivo em zona de risco declarada | O caminho casa com uma zona listada em `docs/legado/PERFIL.md` |
| O2 | Mudança de regra de negócio ou de cálculo | A task é `tipo: regra-de-calculo`, ou o diff altera fórmula, alíquota, arredondamento, condição de negócio |
| O3 | Migração de banco, **qualquer uma** | Arquivo em pasta de migração, ou DDL no diff (`CREATE`, `ALTER`, `DROP`, `RENAME`) |
| O4 | Autenticação, autorização, dado pessoal | Login, sessão, token, permissão, papel, CPF/CNPJ, e-mail, endereço, dado de saúde ou financeiro |
| O5 | Alteração de contrato público | Rota, assinatura de endpoint, payload, evento publicado ou consumido, formato de retorno, contrato de integração |
| O6 | Código sem cobertura de teste antes **e** depois | O relatório de cobertura não cobre as linhas alteradas, e nenhuma task declara teste sobre elas |
| O7 | Efeito irreversível declarado no plano de reversão | O plano de reversão da legadox diz que este arquivo produz efeito que o versionador não desfaz |
| O8 | Veio de raio ALTO | O raio da legadox classifica este arquivo como ALTO |

### LEITURA RÁPIDA — o revisor confere intenção, não implementação

Nenhum critério de OLHO OBRIGATÓRIO bateu, e **um** destes bate:

| # | Critério | Evidência que o confirma |
|---|---|---|
| L1 | Mudança coberta por teste de caracterização que continua passando | A caracterização existe, cobre estas linhas, e a suíte está verde |
| L2 | Alteração em camada isolada com cobertura existente | O arquivo já tinha cobertura antes, e a camada não é atravessada por contrato público |
| L3 | Código novo em arquivo novo, com os dois testes verdes | `A` no diff, a task declara `teste_integracao` e `teste_funcional`, suíte verde |

### DISPENSÁVEL — a máquina já provou

Nenhum critério acima bateu, e **um** destes bate:

| # | Critério | Evidência que o confirma |
|---|---|---|
| D1 | Arquivo de teste que só acrescenta caso | O arquivo é de teste, e o diff só adiciona casos — não altera nem remove asserção existente |
| D2 | Alteração mecânica coberta por teste de regressão verde | Renomeação, movimentação, formatação, troca de import — com o teste de regressão do trabalho verde sobre ela |
| D3 | Arquivo gerado automaticamente, **quando declarado como tal** | O `CONVENCOES.md` da stackx ou o próprio arquivo o declara gerado (lockfile, snapshot, cliente de API gerado, migração gerada por ORM que só reflete o modelo) |

### A regra do desempate

**Na dúvida entre duas faixas, sobe para a mais rigorosa e diz por quê.** Escreva a dúvida na justificativa: "subiu para OLHO OBRIGATÓRIO porque a cobertura destas linhas não pôde ser confirmada".

Um arquivo que não bate em nenhum critério de nenhuma faixa vai para **OLHO OBRIGATÓRIO** por padrão. Ausência de evidência é ausência de prova, e o padrão é o rigor.

### O que NÃO é critério

- **Tamanho do diff.** Nem para subir, nem para descer. Um arquivo de uma linha em zona de risco é OLHO OBRIGATÓRIO; um arquivo de 900 linhas de snapshot gerado é DISPENSÁVEL.
- **Confiança no autor**, humano ou máquina.
- **Pressa da entrega.**
- **Quantidade de arquivos já em OLHO OBRIGATÓRIO.** A faixa não tem cota. Se o trabalho inteiro é de risco, o trabalho inteiro é OLHO OBRIGATÓRIO — e o revisor precisa saber disso antes de abrir o diff.

## Passo 4 — Escrever a justificativa

Cada arquivo leva **uma linha** de justificativa que nomeia o critério e a evidência. A justificativa é o que torna a classificação auditável: sem ela, é opinião.

Correto:

```
src/fiscal/calculo_icms_st.py — OLHO OBRIGATÓRIO
  O1: zona de risco "fiscal/" declarada em docs/legado/PERFIL.md; O2: altera a
  base de cálculo (T-01.02); O8: raio ALTO
```

Errado — não nomeia evidência, é sensação:

```
src/fiscal/calculo_icms_st.py — OLHO OBRIGATÓRIO
  parece arriscado, melhor olhar com atenção
```

## Exemplos de classificação

### Correto

**`src/fiscal/calculo_icms_st.py` — 1 linha alterada → OLHO OBRIGATÓRIO**
O caminho está na zona de risco `fiscal/` do `PERFIL.md` (O1) e a linha alterada é a base de cálculo (O2). **O tamanho não rebaixa.** Uma linha é exatamente onde um erro fiscal se esconde: `base - desconto` virando `base + desconto` são dois caracteres e uma autuação.

**`tests/fiscal/test_icms_st.py` — 60 linhas adicionadas → DISPENSÁVEL**
Arquivo de teste, o diff só acrescenta casos, nenhuma asserção existente foi alterada ou removida (D1). O revisor não precisa ler: se o teste estivesse errado, a suíte não estaria verde sobre o comportamento novo. **Confirme que nada foi removido** — teste apagado nunca é DISPENSÁVEL.

**`src/relatorios/exportador.py` — 120 linhas refatoradas → LEITURA RÁPIDA**
Refatoração coberta por teste de caracterização que continua passando (L1). O revisor confere a intenção — "extraíram o formatador para uma classe" — sem ler linha a linha, porque o comportamento está congelado por teste.

### Incorreto

**Rebaixar por tamanho.** "`calculo_icms_st.py` mudou só uma linha, então LEITURA RÁPIDA." Viola a regra 8 e a 9. O critério é a zona e a natureza da mudança.

**Subir por tamanho.** "`package-lock.json` tem 4.000 linhas, então OLHO OBRIGATÓRIO." Arquivo gerado declarado é DISPENSÁVEL (D3). Fazer o revisor encarar 4.000 linhas de lockfile é o desperdício exato que a skill existe para evitar. (Mudança de *dependência* é assunto do arquivo de manifesto, não do lockfile.)

**Classificar por sensação.** "Esse arquivo eu não conheço, então OLHO OBRIGATÓRIO." O resultado até pode estar certo — o padrão sem evidência é OLHO OBRIGATÓRIO — mas a justificativa tem que dizer isso: "sem evidência de cobertura; classificado no padrão".

**Deixar arquivo sem faixa.** Todo arquivo do diff é classificado. Um arquivo esquecido é um arquivo que ninguém revisou.

**Confundir migração gerada com migração.** Uma migração de banco é sempre O3, **mesmo gerada por ORM**, quando cria, altera ou remove estrutura ou dado. D3 só vale para arquivo gerado que **reflete** algo já revisado em outro lugar e não tem efeito próprio no banco.

## Passo 5 — Gravar a saída

Grave `docs/entregas/<trabalho_id>/ATENCAO.md` a partir de `assets/TEMPLATE-atencao.md`.

A ordem é fixa: **OLHO OBRIGATÓRIO primeiro**, depois LEITURA RÁPIDA, depois DISPENSÁVEL. O revisor lê de cima para baixo e pode parar quando quiser — o mais importante já passou.

Estrutura:

1. Uma linha de resumo: `N arquivos — X olho obrigatório, Y leitura rápida, Z dispensável`.
2. As três seções, cada arquivo com caminho, tipo de mudança, tamanho e a justificativa com o critério.
3. **Fontes consultadas e fontes ausentes** — o revisor precisa saber que dimensão não foi avaliada.

Registre no `ENTREGA.md` a contagem por faixa (`atencao.olho_obrigatorio`, `atencao.leitura_rapida`, `atencao.dispensavel`).

Esta saída vai inteira para a seção **"Onde eu quero seu olho"** da descrição do PR (E4), e alimenta a ordenação do E9.

## Critério de saída

- [ ] Todo arquivo do diff está em exatamente uma faixa.
- [ ] Toda classificação nomeia pelo menos um critério (`O1`..`O8`, `L1`..`L3`, `D1`..`D3`).
- [ ] Nenhuma justificativa é baseada em tamanho de diff.
- [ ] Arquivos sem evidência suficiente estão em OLHO OBRIGATÓRIO, com a razão declarada.
- [ ] As fontes ausentes estão listadas.
- [ ] `ATENCAO.md` gravado e as contagens estão no `ENTREGA.md`.

## Quando falha

| Situação | O que fazer |
|---|---|
| `PERFIL.md` ausente | Zona de risco não avaliada; registre como fonte ausente; os outros critérios decidem |
| Sem relatório de cobertura | O6 não pode ser descartado: arquivo sem teste declarado em task nenhuma vai para OLHO OBRIGATÓRIO |
| Arquivo não bate em nenhum critério | OLHO OBRIGATÓRIO por padrão, com a razão declarada |
| Diff vazio | Não há entrega. Relate e encerre: o E2 deveria ter barrado |
| Sem versionador | Classifique os arquivos declarados nas tasks, com a mesma tabela; registre que a base foi o plano, não o diff |
| Arquivo removido (`D`) | Classifique pelo que ele era: teste removido é OLHO OBRIGATÓRIO (perda de cobertura), nunca DISPENSÁVEL |
