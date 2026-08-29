# Exemplo — descrição de pull request

Caso: correção de regra de cálculo fiscal, vinda da `runx`, em projeto sob modo legado com raio **ALTO**. Saída da etapa E4.

---

O ICMS-ST passa a excluir o desconto incondicional da base de cálculo. Hoje ele inclui, e toda nota com desconto sai com imposto a maior — o cliente vem pagando a diferença desde a migração de outubro. A correção vale só para notas emitidas a partir de agora; o passivo já emitido é assunto de outra ocorrência.

## Arquivos alterados

```
migrations/
  A 0042_ajusta_precisao_base_st.sql
src/fiscal/
  M calculo_icms_st.py
  M base_calculo.py
src/relatorios/
  M exportador_nfe.py
src/fiscal/formatacao/
  M moeda.py
tests/caracterizacao/
  A test_congela_base_st_sem_desconto.py
tests/fiscal/
  A test_icms_st_desconto_incondicional.py
  M test_base_calculo.py
tests/relatorios/
  M test_exportador_nfe.py
```

Total: 9 arquivos.

## Origem

**OC-2026-0184** — `regra-de-calculo` — runx.

Relato do cliente: *"As notas com desconto estão saindo com ICMS-ST maior do que o contador calcula. Ele conferiu três notas de agosto e as três deram diferença."*

## Causa raiz

`calcular_base_st()` monta a base somando o valor bruto dos itens e aplicando o MVA, **sem subtrair o desconto incondicional**. A subtração existia até a migração de outubro de 2025, quando o rateio por item foi reescrito e a linha se perdeu — nenhum teste cobria o caminho com desconto, então nada acusou.

Comprovado com a nota 118.402 do relato: base calculada R$ 1.000,00 contra R$ 900,00 esperados, com desconto de R$ 100,00.

Detalhe: [01-CAUSA-RAIZ.md](../docs/manutencao/OC-2026-0184-icms-st-base-desconto/01-CAUSA-RAIZ.md)

## Raio de impacto

> **ALTO**

**Sinais:**
- toca a zona de risco `fiscal/`, declarada no `PERFIL.md`
- altera valor monetário gravado em documento fiscal emitido
- o valor alterado é consumido por 3 outros módulos (relatório de faturamento, exportador de NF-e, segunda via de boleto)
- exige migração para ajustar a precisão decimal da coluna de base

**Zonas tocadas:** `fiscal`, `relatorios`

Aprovação humana de raio ALTO registrada em 2026-08-27.

## O que foi testado

- **T-01.01** — regressão: reproduz a base inflada com desconto incondicional; falhava antes do fix com R$ 1.000,00 contra R$ 900,00 esperados
- **T-01.02** — integração: emite a nota fim a fim e confere a base gravada; funcional: desconto de 10% sobre item de R$ 100,00
- **T-01.03** — integração: exporta a NF-e e confere o campo de base no XML; funcional: confere o arredondamento em 2 casas
- **T-01.04** — integração: relatório de faturamento com nota com desconto; funcional: soma da coluna de imposto

Suíte: **verde**, 1.284 testes — `make test`

## O que foi congelado

Dois testes de caracterização escritos antes do fix:

- **base de ST sem desconto** — congela o cálculo atual para notas **sem** desconto incondicional, que está correto e não pode mudar. É o que garante que a correção não vaza para o caminho que já funcionava.
- **arredondamento para cima na terceira casa** — congela um **comportamento errado preservado de propósito**: o sistema arredonda a base para cima na terceira casa decimal, contra a regra fiscal, que manda arredondar na segunda. Mudar isso agora alteraria o valor de **todas** as notas, com desconto ou sem, e está fora do escopo desta ocorrência. Foi registrado em `DIVIDA.md` como ocorrência a abrir.

## Como reverter

Reverter os commits desta branch restaura o cálculo anterior. A migração `0042` precisa ser revertida à parte, com a migração reversa incluída no mesmo arquivo.

**Efeitos que o versionador NÃO desfaz:**

| Efeito | Como desfazer |
|---|---|
| Migração `0042` aplicada (precisão decimal da coluna `base_st`) | migração reversa; sem perda de dado, a precisão só aumenta |
| Notas emitidas depois do deploy, já com a base correta | não se desfaz, e não se deve: são as notas certas |

## Roteiro de teste manual

[QA-PACOTE.md](QA-PACOTE.exemplo.md) — 5 casos.

Colaterais a observar: relatório de faturamento mensal, segunda via de boleto e exportação de NF-e — os três leem o mesmo valor de base.

## Onde eu quero seu olho

**9 arquivos — 4 olho obrigatório, 2 leitura rápida, 3 dispensável.**

### OLHO OBRIGATÓRIO — ler linha a linha

- `src/fiscal/calculo_icms_st.py` — O1: zona de risco `fiscal/`; O2: altera a base de cálculo (T-01.02); O8: raio ALTO. **1 linha alterada**
- `src/fiscal/base_calculo.py` — O1: zona de risco `fiscal/`; O8: raio ALTO
- `migrations/0042_ajusta_precisao_base_st.sql` — O3: migração de banco; O7: efeito declarado como irreversível pelo versionador
- `src/relatorios/exportador_nfe.py` — O5: altera o campo de base no XML da NF-e, que é contrato público com a SEFAZ

### LEITURA RÁPIDA — conferir intenção, não implementação

- `src/fiscal/formatacao/moeda.py` — L1: coberto pela caracterização de arredondamento, que continua passando
- `tests/fiscal/test_base_calculo.py` — L2: altera asserção existente para o valor correto; a mudança de asserção precisa de conferência de intenção, não de implementação

### DISPENSÁVEL — a máquina já provou

- `tests/fiscal/test_icms_st_desconto_incondicional.py` — D1: arquivo de teste novo, só acrescenta casos
- `tests/caracterizacao/test_congela_base_st_sem_desconto.py` — D1: caracterização nova, só acrescenta casos
- `tests/relatorios/test_exportador_nfe.py` — D1: só acrescenta caso; nenhuma asserção existente alterada

**Fontes ausentes:** nenhuma.

## Fora de escopo

Do `DIVIDA.md`:

- **arredondamento na terceira casa** — o sistema arredonda para cima onde a regra fiscal manda arredondar na segunda. Congelado por caracterização nesta entrega; corrigir mudaria o valor de todas as notas. Ocorrência a abrir.
- **notas já emitidas com base inflada** — o passivo desde outubro de 2025 não é tocado aqui. Exige decisão fiscal sobre carta de correção.
