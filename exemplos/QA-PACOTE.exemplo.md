# Exemplo — pacote para o QA

Caso: correção de regra de cálculo fiscal, vinda da `runx`. Saída da etapa E5, gravada em `docs/entregas/OC-2026-0184-icms-st-base-desconto/QA-PACOTE.md`.

Note que nada abaixo exige abrir código: nenhum nome de arquivo, função ou tabela, e nenhum jargão de método.

---

# Pacote de QA — Imposto ST em notas com desconto

## 1. O que mudou

Quando uma nota tem desconto, o imposto ST passa a ser calculado sobre o valor **já com o desconto aplicado**.

Antes, o sistema calculava o imposto sobre o valor cheio, ignorando o desconto — e o imposto saía mais alto do que deveria. Era o que o contador do cliente vinha apontando.

Notas **sem** desconto não mudam em nada: continuam com exatamente o mesmo imposto de antes.

## 2. Roteiro de teste

### Caso 1 — Nota com desconto: o imposto tem que baixar

**O que fazer:**
1. Entre em Vendas → Nova nota.
2. Escolha um cliente de outro estado (a lista mostra o estado ao lado do nome).
3. Adicione um produto que tenha substituição tributária — a tela mostra a marca "ST" ao lado do nome do produto.
4. Coloque quantidade 1 e valor unitário 100,00.
5. No campo Desconto, coloque 10,00.
6. Clique em Calcular impostos.

**O que deve acontecer:**
O campo Base ST mostra **90,00** (o valor com o desconto já descontado).
O campo Valor ST mostra **16,20**.

**O que indicaria falha:**
Base ST mostrando 100,00, ou Valor ST mostrando 18,00. É o comportamento antigo, que era o problema.

---

### Caso 2 — Nota sem desconto: nada pode mudar

**O que fazer:**
1. Repita o Caso 1, mas deixe o campo Desconto em branco (ou 0,00).
2. Clique em Calcular impostos.

**O que deve acontecer:**
O campo Base ST mostra **100,00** e o Valor ST mostra **18,00**.

**O que indicaria falha:**
Qualquer valor diferente. Este caso existe para garantir que a correção não afetou quem já estava certo — é o teste mais importante da lista.

---

### Caso 3 — Desconto maior, para conferir que a conta acompanha

**O que fazer:**
1. Repita o Caso 1, mas coloque 50,00 no campo Desconto.
2. Clique em Calcular impostos.

**O que deve acontecer:**
Base ST mostra **50,00** e Valor ST mostra **9,00**.

**O que indicaria falha:**
Base ST mostrando 100,00, ou um valor que não seja exatamente a metade do Caso 2.

---

### Caso 4 — Vários itens, com desconto em um só

**O que fazer:**
1. Crie uma nota para cliente de outro estado.
2. Adicione dois produtos com ST: um de 100,00 e outro de 200,00.
3. Coloque desconto de 10,00 **apenas no item de 100,00**.
4. Clique em Calcular impostos.

**O que deve acontecer:**
A Base ST total mostra **290,00** — o desconto sai só do item que o recebeu, e o outro item continua inteiro.

**O que indicaria falha:**
Base ST de 300,00 (o desconto foi ignorado) ou de 280,00 (o desconto foi aplicado duas vezes, ou espalhado pelos dois itens).

---

### Caso 5 — A nota emitida tem que sair com o valor novo

**O que fazer:**
1. Refaça o Caso 1 e emita a nota de verdade (botão Emitir).
2. Espere a confirmação de autorização.
3. Abra o Espelho da nota e depois baixe o arquivo XML pelo botão Baixar XML.

**O que deve acontecer:**
O Espelho mostra Base ST de 90,00 e Valor ST de 16,20 — os mesmos números da tela de cálculo.
O arquivo XML abre normalmente, e a nota fica com situação **Autorizada**.

**O que indicaria falha:**
Valores diferentes entre a tela e o espelho; nota rejeitada pela SEFAZ; ou erro ao baixar o XML.

## 3. O que observar de colateral

Estes três lugares leem o mesmo valor de imposto que foi alterado. Confira cada um depois de emitir a nota do Caso 5:

- **Relatório de faturamento mensal** (Relatórios → Faturamento). Rode o relatório do mês corrente e confira se a coluna de imposto ST bateu com a soma das notas. Emita antes uma nota sem desconto e outra com, para ver as duas na mesma listagem.
- **Segunda via de boleto** (Financeiro → Boletos → 2ª via). Se a nota do Caso 5 gerou boleto, o valor total do boleto tem que bater com o total da nota.
- **Exportação de NF-e** (Fiscal → Exportar XML do período). Exporte o período e confira que o arquivo da nota do Caso 5 sai com os mesmos valores do espelho.

## 4. Dado de teste sugerido

Use dados fictícios. **Não use cliente, CNPJ ou nota reais.**

- **Cliente:** um cadastro de teste, com estado **diferente** do estado da empresa (é o que faz a nota ter ST). Se não houver, crie um com nome "Cliente Teste ST" e um CNPJ fictício.
- **Produto:** um cadastro com substituição tributária marcada e MVA preenchido. A tela de produto mostra "ST" quando está certo.
- **Valores:** os do roteiro (100,00 com desconto de 10,00) foram escolhidos porque a conta dá números redondos e fáceis de conferir de cabeça.

Se não houver cliente de outro estado cadastrado, dá para criar um pela tela de Clientes; nenhum outro preparo é necessário.

## 5. Ambiente e estado inicial

**Onde testar:** ambiente de homologação. O acesso é o mesmo que você usa para os outros testes; se precisar de usuário novo, peça ao time de infraestrutura — a senha não fica escrita aqui.

**O que precisa existir antes:**
- um cliente cadastrado com estado diferente do da empresa;
- pelo menos um produto com substituição tributária e MVA preenchido;
- seu usuário com permissão de emitir nota.

**O que precisa estar ligado:**
- a integração com a SEFAZ de homologação (necessária só para o Caso 5; os casos 1 a 4 são só cálculo de tela).

## 6. Critério de aprovação

**APROVADO quando:** os cinco casos passam com exatamente os valores indicados, **e** os três lugares da seção 3 mostram os mesmos valores da nota.

**REPROVADO quando:** qualquer caso mostra valor diferente do indicado; ou o Caso 2 (nota sem desconto) mudou de comportamento; ou algum dos três lugares da seção 3 mostra valor diferente do da nota; ou a nota do Caso 5 é rejeitada.

## 7. O que NÃO faz parte desta entrega

- **Notas já emitidas antes desta correção.** As notas com desconto emitidas desde outubro do ano passado continuam com o imposto antigo. Corrigir isso depende de decisão fiscal e será tratado separadamente. **Não reprove por isso.**
- **O arredondamento na terceira casa.** O sistema arredonda para cima na terceira casa decimal, e isso está diferente da regra fiscal. É um problema conhecido, ficou de fora de propósito, e será tratado em outra ocorrência. **Não reprove por isso.**
- **Outros impostos.** Só o ST foi alterado. ICMS próprio, IPI, PIS e COFINS não foram tocados.

---

Depois da aprovação, o texto que o suporte devolve ao cliente é o relatório de uso, em `docs/relatorios/<data>-OC-2026-0184-icms-st-base-desconto/uso.md`.

Ele ainda não existe: é escrito no encerramento da ocorrência, que acontece depois desta aprovação.
