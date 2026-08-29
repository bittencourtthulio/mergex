# mergex

Leva o trabalho feito até o repositório e até o revisor humano, com a branch nascendo junto com o trabalho e o diff chegando já classificado por onde a atenção vale a pena.

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="assets/mergex-fluxo-dark.svg">
  <img alt="A branch nasce quando o trabalho começa e recebe um commit por task concluída; o portão de prontidão barra o que não está pronto; os artefatos das skills irmãs — plano, causa raiz, raio de impacto, roteiro manual e convenções — alimentam a entrega; as saídas são o pull request, o pacote de QA e o registro da entrega, e o revisor humano recebe o diff já dividido em olho obrigatório, leitura rápida e dispensável; a revisão e o merge ficam fora do fluxo automático, como ação manual." src="assets/mergex-fluxo.svg">
</picture>

## O problema: o último metro

Um plano impecável termina com código escrito. E aí ele para.

O pull request chega ao revisor com 600 linhas, título de uma frase e nenhum contexto. O revisor não participou do planejamento, não leu a causa raiz, não sabe o raio de impacto e não faz ideia de qual daqueles 40 arquivos é o que pode derrubar a folha de pagamento. Então ele faz a única coisa possível: lê tudo por igual — e, lendo tudo por igual, a mudança de uma linha na regra fiscal passa com a mesma atenção que o arquivo de teste de 300 linhas. Todo o rigor do método se perde no último metro, exatamente onde ele deveria ser convertido em confiança.

Numa software house com QA e suporte separados do desenvolvimento, essa fronteira é o ponto mais caro do fluxo. O QA recebe "testa aí" e abre o código para descobrir o que mudou. O suporte espera alguém traduzir a entrega para o cliente.

E há o problema de vários desenvolvedores no mesmo projeto: sem branch nascendo junto com o trabalho, tudo acontece na branch em que o dev estava — normalmente a principal.

A `mergex` cuida desse metro final. Cria a branch quando o trabalho começa, commita cada task concluída, sobe a branch e abre o pull request. E costura os artefatos que as outras skills já produziram na descrição da entrega e no pacote do QA.

Ela **não gera conteúdo novo** sobre a mudança. Tudo que escreve vem de arquivo existente: plano, causa raiz, raio de impacto, roteiro manual, convenções do repositório. Insumo ausente vira aviso do que falta, nunca invenção.

## Como funciona

Nove etapas. As oito primeiras são automáticas; a nona só roda quando você a chama.

| Etapa | Quando | O que faz |
|---|---|---|
| **E0** Abertura | No início do trabalho | Cria a branch antes da primeira linha de código. Árvore suja: para e avisa. Branch que já existe: retoma nela |
| **E1** Commit por task | A cada task que fecha | Um commit por task, com a intenção na mensagem. Varredura de segredo antes de cada um |
| **E2** Portão de prontidão | Ao fim da execução | Dez verificações. `PRONTO` ou `BLOQUEADO` com o que falta. Nunca maquia |
| **E3** Atenção humana | Depois do portão | Classifica o diff nas três faixas, arquivo por arquivo, com a evidência de cada um |
| **E4** Descrição do PR | Depois da classificação | Monta a descrição a partir dos artefatos. Cabe em uma tela |
| **E5** Pacote de QA | Antes do push | Documento executável por quem não programa |
| **E6** Push | Depois do portão e do pacote | Sobe a branch. Nunca forçado, nunca na principal |
| **E7** Abertura do PR | Depois do push | Abre o PR pela ferramenta do serviço. Ausente, grava em arquivo e informa |
| **E8** Registro | Ao fim | Grava o registro da entrega, com frontmatter, para o painel de operação |
| **E9** Revisão e merge | **Só quando você chama** | Lista os PRs abertos, ordena por impacto e conduz um merge por vez |

## As três faixas de atenção humana

O revisor humano é recurso caro e finito. O trabalho da `mergex` é gastar esse recurso onde ele muda o resultado, e não gastá-lo onde a máquina já provou.

| Faixa | O que o revisor faz | Quando um arquivo cai aqui |
|---|---|---|
| **OLHO OBRIGATÓRIO** | Lê linha a linha | Zona de risco declarada; regra de negócio ou cálculo; migração de banco, qualquer uma; autenticação, autorização, dado pessoal; contrato público (rota, payload, evento, retorno); código sem cobertura antes e depois; efeito irreversível; raio ALTO |
| **LEITURA RÁPIDA** | Confere a intenção, não a implementação | Coberto por caracterização que continua passando; camada isolada com cobertura existente; código novo em arquivo novo com os dois testes verdes |
| **DISPENSÁVEL** | Não abre — a máquina já provou | Teste que só acrescenta caso; alteração mecânica coberta por regressão verde; arquivo gerado automaticamente, quando declarado como tal |

A classificação é **derivada de evidência registrada** — zonas de risco, raio, cobertura, tipo de mudança. Nunca de sensação. Na dúvida entre duas faixas, sobe para a mais rigorosa e diz por quê.

**Tamanho de diff não é critério.** Um arquivo de uma linha em zona de risco fiscal é OLHO OBRIGATÓRIO; um lockfile de 4.000 linhas é DISPENSÁVEL. É exatamente essa inversão que a leitura por diff não consegue fazer.

## O comando de revisão: por que ele é o único manual

Todo o resto do fluxo é automático — as skills irmãs acionam a `mergex` sozinhas e ela entrega sem intervenção. `/mergex-revisar` é a única exceção do ecossistema.

Ele não é encadeado por nenhum fluxo, não é sugerido ao fim de um trabalho, não aparece como próximo passo em relatório nenhum, e só roda quando você o chama pelo nome. **Integrar código é decisão humana.**

Quando você chama, ele:

- lista os PRs abertos e reúne o estado de cada um: raio, faixas de atenção, integração contínua, conflito;
- destaca no topo os PRs que tocam o mesmo arquivo;
- ordena **do menor para o maior impacto** — cada merge fácil que entra reduz a superfície do próximo, e adiar o difícil não o piora, enquanto adiar o fácil sim;
- conduz um PR por vez, com confirmação explícita de cada um.

Ele **nunca resolve conflito**. Relata onde está, quais arquivos e trechos, e o que cada lado pretendia segundo a mensagem de commit e o plano de cada trabalho — essa análise é o que ele tem de mais útil. A resolução é humana. E nunca faz merge com integração vermelha, de PR em rascunho, em lote, ou de PR com arquivos em olho obrigatório sem que você confirme que os revisou.

## O pacote de QA: por que existe numa casa com QA separado

Quando o QA não é quem escreveu o código, "testa aí" custa uma tarde: ele abre o diff, tenta inferir o que mudou, e testa o que conseguiu adivinhar.

O `QA-PACOTE.md` tem um critério de qualidade só, e é duro: **o QA não deve precisar ler código para trabalhar.** Ele traz o que mudou em linguagem de produto, o roteiro completo dentro do próprio arquivo, o que observar de colateral nas telas vizinhas, dado de teste fictício, o ambiente e como chegar ao estado inicial, e um critério de aprovação binário.

Em trabalho vindo da `runx`, ele aponta ainda o relatório de uso: o texto que o suporte devolve ao cliente depois da aprovação.

## Instalação

As skills ficam **apenas** em `.claude/skills/` nos dois harnesses — o OpenCode lê `.claude/skills/*/SKILL.md` nativamente, e duplicar a árvore causaria colisão de nome. Só os comandos existem nas duas pastas.

### Claude Code

Instalação no projeto:

```bash
git clone https://github.com/bittencourtthulio/mergex.git /tmp/mergex
mkdir -p .claude/skills .claude/commands
cp -r /tmp/mergex/.claude/skills/mergex .claude/skills/
cp /tmp/mergex/.claude/commands/mergex*.md .claude/commands/
cp /tmp/mergex/AGENTS.md .
```

Para usar em todos os projetos, troque `.claude/` por `~/.claude/`.

### OpenCode

A skill vai no mesmo lugar; muda só a pasta dos comandos:

```bash
git clone https://github.com/bittencourtthulio/mergex.git /tmp/mergex
mkdir -p .claude/skills .opencode/commands
cp -r /tmp/mergex/.claude/skills/mergex .claude/skills/
cp /tmp/mergex/.opencode/commands/mergex*.md .opencode/commands/
cp /tmp/mergex/AGENTS.md .
```

O `AGENTS.md` na raiz instrui o agente a acionar a skill nos pontos certos, em qualquer harness.

### Verificação

```bash
ls .claude/skills/mergex/SKILL.md
ls .claude/commands/mergex*.md    # ou .opencode/commands/
```

## Primeiros passos

1. **Comece um trabalho.** Ao iniciar a implementação de uma feature (`sprintx`) ou de uma ocorrência (`runx`), a `mergex` abre a branch sozinha. Para chamá-la à mão: `/mergex-abrir`.
2. **Implemente.** Cada task que fecha com a suíte verde vira um commit.
3. **Confira antes de entregar:** `/mergex-check` roda o portão e diz o que falta.
4. **Entregue:** `/mergex` roda o fluxo completo — classificação, descrição, pacote de QA, push e PR.
5. **Quando você quiser revisar:** `/mergex-revisar`. Só quando você quiser.

| Comando | O que faz |
|---|---|
| `/mergex` | Fluxo automático completo a partir do estado atual |
| `/mergex-abrir` | Abre a branch do trabalho (E0) |
| `/mergex-check` | Portão de prontidão (E2) |
| `/mergex-atencao` | Classifica o diff nas três faixas (E3) |
| `/mergex-pr` | Descrição, push e abertura do PR (E4, E6, E7) |
| `/mergex-qa` | Pacote para o QA (E5) |
| `/mergex-revisar` | Revisão e merge — manual, só por chamada explícita |

Exemplos de saída real em [`exemplos/`](exemplos/): uma correção de regra de cálculo fiscal vinda da `runx`, em projeto sob modo legado com raio ALTO.

## Integração com o ecossistema Expx

| Skill | O que faz | Como se integra |
|---|---|---|
| [`sprintx`](https://github.com/bittencourtthulio/sprintx) | Planeja e executa features novas (F1–F6) | E0 no início da F6, E1 a cada task, E2–E8 ao fim — [patch](docs/integracao/patch-sprintx.md) |
| [`runx`](https://github.com/bittencourtthulio/runx) | Trata ocorrências de produção (E1–E5) | E0 no início do fix, E1 a cada task, E2–E8 entre o QA e o relatório — [patch](docs/integracao/patch-runx.md) |
| `legadox` | Camada modificadora para projetos legados | Sem acionamento próprio: raio, caracterização, reversão e dívida alimentam o portão, a classificação e o PR — [patch](docs/integracao/patch-legadox.md) |
| `stackx` | Descobre o dialeto técnico do repositório | Convenções de branch, commit e teste vêm do `CONVENCOES.md` — [patch](docs/integracao/patch-stackx.md) |

Os patches em [`docs/integracao/`](docs/integracao/) são prompts prontos para colar no repositório de cada skill irmã. Nenhum deles altera comportamento quando a `mergex` não está instalada.

**A ausência da `mergex` nunca quebra o fluxo das outras skills**, e a ausência de qualquer irmã nunca quebra o fluxo da `mergex`: insumo que não existe vira aviso do que falta.

## O que a mergex NÃO faz

- **Não aprova o próprio trabalho.** Prepara a revisão; não a dispensa nem a substitui.
- **Não faz merge no fluxo automático.** A integração é sempre decisão humana.
- **Não cria release, não publica, não faz deploy.**
- **Não altera código para caber na entrega.** Trabalho incompleto é barrado, nunca maquiado.
- **Não fecha a ocorrência.** A `mergex` entrega; quem fecha é a `runx`.

> ### Não previne colisão entre desenvolvedores
>
> A `mergex` não avisa que outro trabalho planeja tocar o mesmo arquivo, não reserva arquivo e não resolve conflito. **A branch isola o trabalho; ela não impede que duas pessoas alterem o mesmo código.** Quem resolve conflito é quem revisa o pull request, com contexto humano.
>
> É por isso que duas coisas são tratadas como prioritárias na skill: a **mensagem de commit por task**, que é o que permite entender a intenção de cada mudança durante um merge difícil, e a **lista de arquivos alterados em destaque no PR**, que é o que permite ao revisor perceber sobreposição com outro PR aberto sem ferramenta nenhuma.

## Licença

MIT. Veja [LICENSE](LICENSE).

## Como contribuir

Abra uma issue descrevendo o caso concreto — o repositório, a etapa e o que aconteceu — antes de abrir um PR grande.

Contribuições que preservam as regras invioláveis da skill são bem-vindas. As que as flexibilizam precisam do caso de uso que as justifica: elas existem porque cada uma resolve um jeito específico de o último metro dar errado. Em particular, `/mergex-revisar` não vira automático.
