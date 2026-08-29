# Exemplo — classificação da atenção humana

Caso: correção de regra de cálculo fiscal, vinda da `runx`, em projeto sob modo legado com raio **ALTO**. Saída da etapa E3, gravada em `docs/entregas/OC-2026-0184-icms-st-base-desconto/ATENCAO.md`.

---

# Onde gastar atenção — OC-2026-0184-icms-st-base-desconto

Branch: `fix/OC-2026-0184-icms-st-base-desconto` → `main`
Data: 2026-08-29

**9 arquivos — 4 olho obrigatório, 2 leitura rápida, 3 dispensável**

A classificação é derivada de evidência registrada, nunca de sensação.
Tamanho de diff não é critério.

## OLHO OBRIGATÓRIO — ler linha a linha

| Arquivo | Mudança | Tamanho | Por quê |
|---|---|---|---|
| `src/fiscal/calculo_icms_st.py` | M | +1/-1 | **O1**: caminho casa com a zona de risco `fiscal/` do `PERFIL.md`. **O2**: a linha alterada é a base de cálculo (T-01.02). **O8**: raio ALTO. Uma linha é exatamente onde um erro fiscal se esconde — `base - desconto` contra `base + desconto` são dois caracteres e uma autuação |
| `src/fiscal/base_calculo.py` | M | +14/-6 | **O1**: zona de risco `fiscal/`. **O8**: raio ALTO |
| `migrations/0042_ajusta_precisao_base_st.sql` | A | +18/-0 | **O3**: migração de banco — qualquer uma. **O7**: o plano de reversão declara a aplicação como efeito que o versionador não desfaz |
| `src/relatorios/exportador_nfe.py` | M | +7/-3 | **O5**: altera o campo de base no XML da NF-e, contrato público com a SEFAZ |

## LEITURA RÁPIDA — conferir intenção, não implementação

| Arquivo | Mudança | Tamanho | Por quê |
|---|---|---|---|
| `src/fiscal/formatacao/moeda.py` | M | +9/-4 | **L1**: coberto pela caracterização `test_congela_base_st_sem_desconto`, que continua passando. O comportamento está congelado; basta conferir a intenção da refatoração. Mesmo estando sob `src/fiscal/`, a zona de risco declarada é `src/fiscal/` para cálculo — a subpasta `formatacao/` não está na tabela de zonas, e a caracterização cobre estas linhas |
| `tests/fiscal/test_base_calculo.py` | M | +6/-4 | **L2**: altera asserção existente para o valor correto. Alteração de asserção não é acréscimo de caso, então não é D1: exige conferência de intenção — o revisor confirma que o novo valor esperado é o certo |

## DISPENSÁVEL — a máquina já provou

| Arquivo | Mudança | Tamanho | Por quê |
|---|---|---|---|
| `tests/fiscal/test_icms_st_desconto_incondicional.py` | A | +84/-0 | **D1**: arquivo de teste novo, só acrescenta casos; nenhuma asserção existente alterada ou removida |
| `tests/caracterizacao/test_congela_base_st_sem_desconto.py` | A | +52/-0 | **D1**: caracterização nova, só acrescenta casos |
| `tests/relatorios/test_exportador_nfe.py` | M | +21/-0 | **D1**: só acrescenta caso — confirmado que nenhuma linha foi removida (`-0`) |

## Fontes consultadas

- `docs/legado/PERFIL.md` — zonas de risco declaradas: `fiscal`, `folha`, `integracao-sefaz`
- Registro de raio da legadox — `FAIXA: ALTO`, zonas tocadas `fiscal` e `relatorios`
- Plano de reversão — efeito irreversível declarado para `migrations/0042_*`
- Testes de caracterização — 2 registrados, ambos passando
- `sprint-01/tasks.md` — 4 tasks, cobertura declarada por arquivo
- Relatório de cobertura — `make coverage`, 2026-08-29
- `docs/stack/CONVENCOES.md` — arquivos gerados: `poetry.lock`, `src/api/gerado/`

## Fontes ausentes

Nenhuma.

---

## Nota de leitura — por que a classificação ficou assim

Três decisões deste diff merecem atenção, porque são as que mais costumam sair erradas:

**O arquivo de uma linha não foi rebaixado.** `calculo_icms_st.py` tem `+1/-1` e é o arquivo **mais** crítico da entrega. Tamanho não é critério: a zona de risco e a natureza da mudança são.

**O lockfile não apareceu.** `poetry.lock` não foi tocado neste trabalho. Se tivesse sido, iria para DISPENSÁVEL por D3 — declarado como gerado no `CONVENCOES.md` —, apesar de ser o maior diff do repositório.

**A migração não foi tratada como arquivo gerado.** Ela é gerada pelo ORM, mas migração de banco tem efeito próprio no banco: é O3, OLHO OBRIGATÓRIO. D3 só vale para arquivo gerado que *reflete* algo já revisado em outro lugar.
