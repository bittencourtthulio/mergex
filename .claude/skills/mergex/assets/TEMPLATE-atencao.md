# TEMPLATE — classificação da atenção humana (E3)

Grave em `docs/entregas/<trabalho_id>/ATENCAO.md`. Substitua todos os
marcadores `{{assim}}`. A ordem das faixas é fixa: o revisor lê de cima para
baixo e pode parar quando quiser. Apague este cabeçalho ao gravar.

---

# Onde gastar atenção — {{trabalho_id}}

Branch: `{{branch}}` → `{{branch_base}}`
Data: {{AAAA-MM-DD}}

**{{n}} arquivos — {{x}} olho obrigatório, {{y}} leitura rápida, {{z}} dispensável**

A classificação é derivada de evidência registrada, nunca de sensação.
Tamanho de diff não é critério.

## OLHO OBRIGATÓRIO — ler linha a linha

| Arquivo | Mudança | Tamanho | Por quê |
|---|---|---|---|
| `{{caminho}}` | {{A/M/D/R}} | {{+n/-n}} | {{critério (O1..O8) e a evidência que o confirma}} |

## LEITURA RÁPIDA — conferir intenção, não implementação

| Arquivo | Mudança | Tamanho | Por quê |
|---|---|---|---|
| `{{caminho}}` | {{A/M/D/R}} | {{+n/-n}} | {{critério (L1..L3) e a evidência}} |

## DISPENSÁVEL — a máquina já provou

| Arquivo | Mudança | Tamanho | Por quê |
|---|---|---|---|
| `{{caminho}}` | {{A/M/D/R}} | {{+n/-n}} | {{critério (D1..D3) e a evidência}} |

## Fontes consultadas

- {{fonte — o que respondeu}}

## Fontes ausentes

- {{fonte que não existe — que dimensão ficou sem avaliação}}

{{fonte ausente não vira suposição: o revisor precisa saber que a dimensão não
foi avaliada, não que foi avaliada e deu negativo}}
