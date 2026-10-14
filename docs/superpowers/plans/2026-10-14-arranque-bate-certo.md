# Arranque do Bate Certo — Plano de implementação

> **Para agentes:** SUB-SKILL OBRIGATÓRIA: usar superpowers:subagent-driven-development (recomendado) ou superpowers:executing-plans para executar este plano tarefa a tarefa. Os passos usam checkboxes (`- [ ]`).

**Objetivo:** montar a organização do projeto Bate Certo (GitHub, quadro, tickets do M0, Notion e `CLAUDE.md`), pronta para começar o levantamento no gabinete.

**Arquitetura:**
- **GitHub:** o repositório público `RafaelFerraz2002/bate-certo` contém os ADRs, os templates e as Actions, e o Project "Bate Certo" organiza os tickets.
- **Notion:** a página "Bate Certo" dentro de 🗂️ Projetos guarda a visão, o levantamento, os requisitos, o glossário, o diário, o manual e o índice de ADRs.
- **Molde:** tudo copia o projeto CoopHousing (`../CoopHousing/senior-care-platform` e a página Notion `3e7fe701-a459-81c6-b6ea-f3e2e6d6d69d`).

**Ferramentas:** `gh` 2.101 (REST e GraphQL), GitHub Actions, as ferramentas MCP do Notion (`notion-create-pages`, `notion-create-database`, `notion-update-page`, `notion-fetch`) e Python 3 (só para o script que cria os issues, que fica no scratchpad).

**Spec:** `docs/superpowers/specs/2026-10-14-arranque-bate-certo-design.md`

## Restrições globais

- Língua: português de Portugal em documentação, issues, PRs, Notion e commits.
- Repositório `RafaelFerraz2002/bate-certo`, **público** e **sem licença open source** (não criar `LICENSE`).
- **Nunca dados reais** de clientes do gabinete (NIFs, nomes, extratos, SAF-T, valores) no repositório, issues, Notion, seeds ou testes; os exemplos são fictícios ou da empresa de demonstração do Primavera.
- **Nunca segredos** em ficheiros versionados.
- Fluxo: ticket → branch `tipo/#nr-descricao` → PR com `Closes #nr`, título em Conventional Commits → **squash merge feito pelo utilizador**. Nunca push direto para `main` depois da Task 1, nunca force push, nunca merge.
- Descrição dos PRs: só "O que muda" (e um resumo, se ajudar) e `Closes #nr`. Sem secção "Como rever", sem rodapé "Generated with Claude Code" e sem link da sessão (regra do utilizador no CoopHousing, que se sobrepõe ao rodapé por defeito).
- Commits terminam com `Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>`, sem linha `Claude-Session` (mesma regra).
- Estados do quadro: `Backlog`, `Pronto`, `Em curso`, `Em revisão`, `Feito`.
- Labels: `epic`, `feature`, `chore`, `spike`, `docs`, `bug`, `mod:plataforma`, `mod:bancaria`, `mod:terceiros`, `mod:efatura`.
- Prefixos dos requisitos: `CB-` bancária, `CT-` contas correntes de terceiros, `EF-` e-Fatura/AT, `GL-` global.
- O Notion fica sob a página 🗂️ Projetos `3f9fe701-a459-8109-a71a-cdd5a8e4a3a7`.

## Foco da revisão

1. **Numeração dos issues diferente da esperada** (por exemplo, se um PR do Dependabot ocupar um número). Tudo o que referencia `#2`/`#3` deve usar o mapa impresso pelo script da Task 4, não números assumidos. A Task 4 verifica o mapa antes de continuar.
2. **Ruleset a bloquear o merge do PR de organização**, por exigir um check que ainda não existe no `main`. O check obrigatório só se acrescenta na Task 7, depois do merge.
3. **`gh project item-edit --field/--value` sem suporte nesta versão do gh.** A Task 3 testa a sintaxe num item antes de o script a usar em massa, e o script tem um fallback por IDs.
4. **Páginas do Notion criadas no sítio errado** (na raiz do workspace, não em Projetos). A Task 5 verifica o `ancestor-path` de cada página criada.
5. **O script dos issues corrido duas vezes** cria duplicados. O script aborta se o repositório já tiver issues.

---

## Mapa de ficheiros

**No repositório `Contabilidade/bate-certo/`:**
- `CLAUDE.md`: orientação pública do projeto (Task 6)
- `.github/ISSUE_TEMPLATE/ticket.md` e `.github/ISSUE_TEMPLATE/config.yml`: template de ticket (Task 6)
- `.github/pull_request_template.md`: template de PR (Task 6)
- `.github/workflows/pr-title.yml`: validação do título do PR (Task 6)
- `.github/dependabot.yml`: atualizações das Actions (Task 6)
- `docs/adr/README.md`: regras, índice e modelo dos ADRs (Task 6)
- `docs/superpowers/specs/…` e `docs/superpowers/plans/…`: já existem (a Task 1 faz o push)

**Fora do repositório:**
- `Contabilidade/CLAUDE.md`: contexto privado, não versionado (Task 6)
- `<scratchpad>/criar_issues.py`: script descartável (Task 4)

---

### Task 1: Repositório no GitHub

**Ficheiros:** nenhum novo. Faz push do commit local `e08eb21` mais o commit deste plano.

**Interfaces:**
- Consome: o repositório git local em `Contabilidade/bate-certo`, já com o branch `main`.
- Produz: `https://github.com/RafaelFerraz2002/bate-certo` com `main` protegido. As tasks seguintes assumem `OWNER=RafaelFerraz2002` e `REPO=bate-certo`.

- [ ] **Passo 1: Commit do plano**

```bash
cd "/Users/enlt-rferraz/Desktop/Rich baby/Contabilidade/bate-certo"
git add docs/superpowers/plans/2026-10-14-arranque-bate-certo.md
git commit -m "docs: plano de implementação do arranque

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>"
```

- [ ] **Passo 2: Criar o repositório público e fazer push do `main`**

```bash
gh repo create RafaelFerraz2002/bate-certo --public \
  --description "Bate Certo: plataforma de conciliações contabilísticas (bancária, terceiros, e-Fatura)" \
  --source . --remote origin --push
```

Esperado: `✓ Created repository RafaelFerraz2002/bate-certo` e `✓ Pushed commits`.

- [ ] **Passo 3: Configurar o merge (só squash, título do PR como commit) e apagar o branch no merge**

```bash
gh repo edit RafaelFerraz2002/bate-certo \
  --enable-squash-merge --enable-merge-commit=false --enable-rebase-merge=false \
  --delete-branch-on-merge --enable-wiki=false
gh api -X PATCH repos/RafaelFerraz2002/bate-certo \
  -f squash_merge_commit_title=PR_TITLE -f squash_merge_commit_message=PR_BODY >/dev/null
```

- [ ] **Passo 4: Segurança (Dependabot, secret scanning, push protection)**

```bash
gh api -X PUT repos/RafaelFerraz2002/bate-certo/vulnerability-alerts
gh api -X PUT repos/RafaelFerraz2002/bate-certo/automated-security-fixes
gh api -X PATCH repos/RafaelFerraz2002/bate-certo --input - <<'EOF' >/dev/null
{"security_and_analysis":{"secret_scanning":{"status":"enabled"},"secret_scanning_push_protection":{"status":"enabled"}}}
EOF
```

- [ ] **Passo 5: Ruleset no `main` (PR obrigatório, só squash, sem force push nem apagar), sem exceções**

```bash
gh api -X POST repos/RafaelFerraz2002/bate-certo/rulesets --input - <<'EOF'
{
  "name": "main",
  "target": "branch",
  "enforcement": "active",
  "conditions": {"ref_name": {"include": ["~DEFAULT_BRANCH"], "exclude": []}},
  "rules": [
    {"type": "deletion"},
    {"type": "non_fast_forward"},
    {"type": "pull_request", "parameters": {
      "required_approving_review_count": 0,
      "dismiss_stale_reviews_on_push": false,
      "require_code_owner_review": false,
      "require_last_push_approval": false,
      "required_review_thread_resolution": false,
      "allowed_merge_methods": ["squash"]}}
  ]
}
EOF
```

- [ ] **Passo 6: Verificar**

```bash
gh repo view RafaelFerraz2002/bate-certo --json visibility,licenseInfo,squashMergeAllowed,mergeCommitAllowed,rebaseMergeAllowed,deleteBranchOnMerge
gh api repos/RafaelFerraz2002/bate-certo --jq '.security_and_analysis'
gh api repos/RafaelFerraz2002/bate-certo/rulesets --jq '.[].name'
git log --oneline origin/main
```

Esperado:
- `"visibility":"PUBLIC"`, `"licenseInfo":null`, squash `true`, merge e rebase `false`, `deleteBranchOnMerge` `true`;
- `secret_scanning` e `secret_scanning_push_protection` `enabled`;
- ruleset `main`;
- 2 commits no `origin/main`.

---

### Task 2: Labels e milestones

**Interfaces:**
- Consome: o repositório da Task 1.
- Produz: as labels da secção "Restrições globais" e as milestones com estes títulos exatos, usados pelo script da Task 4:
  - `M0 Levantamento e Arquitetura`
  - `M1 Fundação`
  - `M2 Primeira conciliação (piloto)`
  - `M3 Segunda conciliação`
  - `M4 Terceira conciliação`
  - `M5 Comercialização`

- [ ] **Passo 1: Criar ou atualizar as labels (o `--force` atualiza as que já existem por defeito, como `bug`)**

```bash
R=RafaelFerraz2002/bate-certo
gh label create epic           -R $R --force --color 5319e7 --description "Épico (agrupa tickets)"
gh label create feature        -R $R --force --color 0e8a16 --description "Funcionalidade nova"
gh label create chore          -R $R --force --color c5def5 --description "Setup, infraestrutura, manutenção"
gh label create spike          -R $R --force --color fbca04 --description "Investigação / decisão"
gh label create docs           -R $R --force --color 0075ca --description "Documentação / ADR"
gh label create bug            -R $R --force --color d73a4a --description "Erro"
gh label create mod:plataforma -R $R --force --color ededed --description "Módulo plataforma"
gh label create mod:bancaria   -R $R --force --color ededed --description "Conciliação bancária"
gh label create mod:terceiros  -R $R --force --color ededed --description "Contas correntes de clientes/fornecedores"
gh label create mod:efatura    -R $R --force --color ededed --description "Conciliação e-Fatura/AT"
```

- [ ] **Passo 2: Apagar as labels por defeito que não usamos**

```bash
for l in documentation duplicate enhancement "good first issue" "help wanted" invalid question wontfix; do
  gh label delete "$l" -R RafaelFerraz2002/bate-certo --yes 2>/dev/null || true
done
```

- [ ] **Passo 3: Criar as milestones**

```bash
R=repos/RafaelFerraz2002/bate-certo/milestones
gh api $R -f title="M0 Levantamento e Arquitetura" -f description="Levantamento no gabinete, spikes, modelo de domínio e ADRs 001–005" >/dev/null
gh api $R -f title="M1 Fundação" -f description="Scaffolding, CI, staging, autenticação, empresas, importação de dados base" >/dev/null
gh api $R -f title="M2 Primeira conciliação (piloto)" -f description="A conciliação com mais dor, em uso real no gabinete" >/dev/null
gh api $R -f title="M3 Segunda conciliação" -f description="" >/dev/null
gh api $R -f title="M4 Terceira conciliação" -f description="" >/dev/null
gh api $R -f title="M5 Comercialização" -f description="Multi-gabinete, subscrição, onboarding, página pública, rever visibilidade do repositório" >/dev/null
```

- [ ] **Passo 4: Verificar**

```bash
gh label list -R RafaelFerraz2002/bate-certo --json name --jq '.[].name' | sort
gh api repos/RafaelFerraz2002/bate-certo/milestones --jq '.[].title'
```

Esperado: exatamente as 10 labels e as 6 milestones acima.

---

### Task 3: Project "Bate Certo"

**Interfaces:**
- Consome: o repositório da Task 1.
- Produz:
  - o Project do utilizador com o título `Bate Certo` e o número `$PN`, impresso no passo 1 e usado nas Tasks 4–7;
  - o campo `Status` com as opções `Backlog`, `Pronto`, `Em curso`, `Em revisão`, `Feito`;
  - os campos `Prioridade` (`Alta`, `Média`, `Baixa`), `Tamanho` (`S`, `M`, `L`) e `Requisitos` (texto).

- [ ] **Passo 1: Criar o Project e ligá-lo ao repositório**

```bash
gh project create --owner RafaelFerraz2002 --title "Bate Certo" --format json --jq '.number'
# guardar o número impresso:
PN=<número impresso>
gh project link $PN --owner RafaelFerraz2002 --repo RafaelFerraz2002/bate-certo
```

- [ ] **Passo 2: Substituir as opções do Status pelas 5 do CoopHousing**

```bash
PID=$(gh project view $PN --owner RafaelFerraz2002 --format json --jq '.id')
SID=$(gh project field-list $PN --owner RafaelFerraz2002 --format json --jq '.fields[]|select(.name=="Status").id')
gh api graphql -f query='
mutation($f:ID!){ updateProjectV2Field(input:{fieldId:$f, singleSelectOptions:[
 {name:"Backlog",    color:GRAY,   description:""},
 {name:"Pronto",     color:BLUE,   description:"Pronto a começar"},
 {name:"Em curso",   color:YELLOW, description:""},
 {name:"Em revisão", color:PURPLE, description:"PR aberto"},
 {name:"Feito",      color:GREEN,  description:""}]}){ projectV2Field{ ... on ProjectV2SingleSelectField{ options{name} } } } }' -f f=$SID
```

Esperado: JSON com as 5 opções.

- [ ] **Passo 3: Criar os campos Prioridade, Tamanho e Requisitos**

```bash
gh project field-create $PN --owner RafaelFerraz2002 --name Prioridade --data-type SINGLE_SELECT --single-select-options "Alta,Média,Baixa"
gh project field-create $PN --owner RafaelFerraz2002 --name Tamanho    --data-type SINGLE_SELECT --single-select-options "S,M,L"
gh project field-create $PN --owner RafaelFerraz2002 --name Requisitos --data-type TEXT
```

- [ ] **Passo 4: Testar a sintaxe `item-edit --field/--value` (ponto 3 do Foco da revisão)**

```bash
ITEM=$(gh project item-create $PN --owner RafaelFerraz2002 --title "teste-sintaxe" --format json --jq '.id')
gh project item-edit $PN --owner RafaelFerraz2002 --id "$ITEM" --field Status --value "Pronto" \
  && echo SINTAXE_OK || echo SINTAXE_FALHOU
gh project item-delete $PN --owner RafaelFerraz2002 --id "$ITEM"
```

- Se `SINTAXE_OK`, o script da Task 4 usa `--field/--value`.
- Se `SINTAXE_FALHOU`, usa o fallback `--project-id/--field-id/--single-select-option-id`, já incluído no script.

- [ ] **Passo 5: Ativar os workflows nativos do Project (passo manual no browser)**

A API não permite configurá-los. Ou o utilizador os ativa, ou o Claude o faz pelo Claude in Chrome, se o utilizador autorizar. Em `https://github.com/users/RafaelFerraz2002/projects/$PN/workflows`:

| Workflow | Configuração |
|---|---|
| Item added to project | Status → `Backlog` |
| Item closed | Status → `Feito` |
| Pull request merged | Status → `Feito` |
| Pull request linked to issue | Status → `Em revisão` |
| Auto-add sub-issues to project | Ativo |
| Auto-add to project | Repositório `bate-certo`, filtro `is:issue,pr is:open` |

- [ ] **Passo 6: Verificar**

```bash
gh api graphql -f query='query($n:Int!){user(login:"RafaelFerraz2002"){projectV2(number:$n){title fields(first:30){nodes{... on ProjectV2SingleSelectField{name options{name}} ... on ProjectV2Field{name dataType}}} workflows(first:20){nodes{name enabled}}}}}' -F n=$PN
```

Esperado:
- Status com as 5 opções, Prioridade com 3, Tamanho com 3, Requisitos `TEXT`;
- os 5 workflows da tabela `enabled: true`, e `Auto-add to project` também, se o plano do GitHub o permitir.

---

### Task 4: Épicos e tickets do M0

**Ficheiros:** criar `<scratchpad>/criar_issues.py` (descartável, não vai para o repositório).

**Interfaces:**
- Consome: as milestones da Task 2 (títulos exatos), `$PN` da Task 3 e a sintaxe validada na Task 3, passo 4.
- Produz: 20 issues e um mapa `chave → número`, impresso e guardado em `<scratchpad>/issues.json`. Números esperados num repositório vazio:

| Chave | Nº | Chave | Nº |
|---|---|---|---|
| E0 | #1 | T8 | #9 |
| T1 Notion | #2 | ADR1 | #10 |
| T2 Repo | #3 | ADR2 | #11 |
| T3 Guião | #4 | ADR3 | #12 |
| T4 Spike formatos | #5 | ADR4 | #13 |
| T5 Spike mercado | #6 | ADR5 | #14 |
| T6 Sessão | #7 | T14 Ordem | #15 |
| T7 Requisitos | #8 | E1–E5 | #16–#20 |

As Tasks 5–7 usam sempre o mapa, nunca esta tabela.

- [ ] **Passo 1: Escrever o script**

```python
#!/usr/bin/env python3
"""Cria os épicos e os tickets do M0 do Bate Certo. Uso: criar_issues.py <PN> <usar_field_value:0|1> <saida.json>"""
import json, subprocess, sys

OWNER, REPO = "RafaelFerraz2002", "bate-certo"
R = f"{OWNER}/{REPO}"
PN, FIELD_VALUE, OUT = sys.argv[1], sys.argv[2] == "1", sys.argv[3]
M0 = "M0 Levantamento e Arquitetura"

def gh(*args, inp=None):
    return subprocess.run(["gh", *args], input=inp, capture_output=True, text=True, check=True).stdout.strip()

def corpo(contexto, criterios, fora, notas="", requisitos="—"):
    crit = "\n".join(f"- [ ] {c}" for c in criterios)
    return (f"## Contexto\n{contexto}\n\n## Requisitos\n{requisitos}\n\n"
            f"## Critérios de aceitação\n{crit}\n\n## Fora de âmbito\n{fora}\n\n## Notas\n{notas}").strip() + "\n"

SPEC = "`docs/superpowers/specs/2026-10-14-arranque-bate-certo-design.md`"

EPICOS = [
  ("E0", "[E0] Arranque: levantamento e arquitetura", ["epic", "mod:plataforma"], M0,
   corpo("Organizar o projeto, levantar requisitos no gabinete e tomar as decisões de arquitetura antes de qualquer código. Spec: " + SPEC + ".",
         ["Todos os tickets do M0 fechados", "Critério de saída do M0 cumprido (spec, 4.5)"], "Código de produto.")),
]
EPICOS_DEPOIS = [
  ("E1", "[E1] Plataforma: fundação", ["epic", "mod:plataforma"], "M1 Fundação",
   corpo("Scaffolding, CI completo, staging, autenticação, empresas do gabinete e importação de dados base (plano de contas e movimentos).",
         ["Partido em tickets quando o M0 fechar"], "As conciliações.")),
  ("E2", "[E2] Conciliação bancária", ["epic", "mod:bancaria"], None,
   corpo("Extrato bancário vs. movimentos da conta 12 (Depósitos à ordem).",
         ["Requisitos CB-* validados", "Milestone atribuída pelo ticket da ordem dos épicos", "Partido em tickets quando for o próximo"], "Outras conciliações.")),
  ("E3", "[E3] Contas correntes de clientes/fornecedores", ["epic", "mod:terceiros"], None,
   corpo("Conta corrente de terceiros (21x/22x) vs. extratos de fornecedores, faturas vs. recebimentos/pagamentos.",
         ["Requisitos CT-* validados", "Milestone atribuída pelo ticket da ordem dos épicos", "Partido em tickets quando for o próximo"], "Outras conciliações.")),
  ("E4", "[E4] Conciliação e-Fatura/AT", ["epic", "mod:efatura"], None,
   corpo("Faturas comunicadas no e-Fatura vs. documentos lançados na contabilidade: faltas e diferenças de IVA.",
         ["Requisitos EF-* validados", "Milestone atribuída pelo ticket da ordem dos épicos", "Partido em tickets quando for o próximo"], "Outras conciliações.")),
  ("E5", "[E5] Comercialização", ["epic", "mod:plataforma"], "M5 Comercialização",
   corpo("Multi-gabinete a sério, subscrição, onboarding, página pública e revisão da visibilidade do repositório.",
         ["Partido em tickets quando o piloto (M2) estiver em uso"], "—")),
]

TICKETS = [
  ("T1", "Montar o Notion do Bate Certo", ["chore", "mod:plataforma"], "Alta", "M", "Em curso",
   corpo("Página Bate Certo em 🗂️ Projetos, com a estrutura da spec (3.2).",
         ["Páginas Visão e âmbito, Levantamento no gabinete, Diário de aprendizagem, Manual da app, Decisões (ADR) criadas",
          "Bases Requisitos, Glossário e Diário criadas com o esquema do CoopHousing",
          "Glossário com os termos iniciais de contabilidade", "Página principal com links para o quadro, épicos e milestones"],
         "Conteúdo do levantamento (ticket do guião e da sessão).")),
  ("T2", "Organização do repositório: templates, Actions, ADRs e CLAUDE.md", ["chore", "mod:plataforma"], "Alta", "M", "Em curso",
   corpo("Repositório, quadro, labels, milestones, ruleset e segurança já criados. Falta versionar a orientação do projeto (spec 3.3 e 3.4).",
         ["`CLAUDE.md` público e `CLAUDE.md` privado na pasta-mãe", "Templates de ticket e de PR",
          "Action de validação do título do PR em Conventional Commits", "`dependabot.yml` para as Actions",
          "`docs/adr/README.md` com regras, índice e modelo", "Check do título do PR obrigatório no ruleset depois do merge"],
         "CI de lint/testes/build (M1).")),
  ("T3", "Guião do levantamento no gabinete", ["docs", "mod:plataforma"], "Alta", "S", "Backlog",
   corpo("Preparar a sessão no gabinete: o que perguntar, o que observar e o que pedir (spec 4.1).",
         ["Página no Notion com o guião, pronta a imprimir", "Perguntas por conciliação (bancária, terceiros, e-Fatura)",
          "Linha de base (tempos, nº de empresas)", "Regras de sigilo e pergunta sobre acordo de confidencialidade",
          "Pedido de ficheiros da empresa de demonstração do Primavera"],
         "Fazer a sessão (ticket próprio).", "Depende do Notion montado.")),
  ("T4", "Spike: formatos e fontes de dados", ["spike", "mod:plataforma"], "Alta", "M", "Backlog",
   corpo("Saber de onde podem vir os dados das conciliações e em que formato (spec 4.2).",
         ["Extratos: CSV por banco, MT940, CAMT.053", "Open Banking (Enable Banking)", "SAF-T (PT) de contabilidade: estrutura",
          "Exportação do e-Fatura: formato e automatização", "Primavera: exportações, importações aceites, leitura do SQL Server",
          "Página no Notion com recomendação"],
         "Implementar importadores.")),
  ("T5", "Spike: o que já existe e regulamentação", ["spike", "mod:plataforma"], "Média", "M", "Backlog",
   corpo("Perceber onde está o espaço do Bate Certo face ao Primavera e ao mercado (spec 4.3).",
         ["O que o Primavera faz nas três conciliações", "Add-ons e concorrentes (TOConline, Moloni e afins)",
          "Confirmar que conciliação não exige certificação AT", "Obrigações se for vendido", "Página no Notion com conclusões"],
         "Plano de negócio.")),
  ("T6", "Sessão de levantamento no gabinete", ["spike", "mod:plataforma"], "Alta", "M", "Backlog",
   corpo("Ver as três conciliações a serem feitas no Primavera e recolher dores, tempos e restrições. Feito pelo utilizador.",
         ["Sessão feita com o guião", "Notas sem dados reais", "Ficheiros da empresa de demonstração pedidos"],
         "Escrever requisitos (ticket seguinte).", "Depende do guião e dos dois spikes.")),
  ("T7", "Requisitos e relatório do levantamento", ["docs", "mod:plataforma"], "Alta", "L", "Backlog",
   corpo("Passar o que se viu no gabinete para a base Requisitos e escrever o relatório (✅ ⚠️ ❓).",
         ["Requisitos CB-/CT-/EF-/GL- na base", "Relatório na página Levantamento", "Linha de base registada",
          "Validado com o gabinete (coluna Validação)"],
         "Modelo de domínio.")),
  ("T8", "Modelo de domínio das conciliações", ["docs", "mod:plataforma"], "Alta", "M", "Backlog",
   corpo("Entidades e relações: Empresa, Conta, Extrato, Movimento, Regra de emparelhamento, Conciliação, Diferença…",
         ["Diagrama e descrição das entidades", "Ligado aos requisitos que o justificam"], "Esquema físico da base de dados.")),
  ("ADR1", "ADR-001: escolha da stack", ["docs", "spike", "mod:plataforma"], "Alta", "M", "Backlog",
   corpo("Escolher backend, frontend e base de dados, com base no levantamento e no modelo de domínio.",
         ["2–3 opções comparadas (prós, contras, custo)", "Decisão em `docs/adr/001-stack.md`"], "Código.")),
  ("ADR2", "ADR-002: alojamento e servidor", ["docs", "spike", "mod:plataforma"], "Alta", "S", "Backlog",
   corpo("SO do servidor, cloud vs. rede do gabinete, região UE, backups. Por defeito Linux (Debian/Ubuntu LTS) + Docker num fornecedor UE (spec 4.4).",
         ["Opções comparadas com preços verificados", "Decisão em `docs/adr/002-alojamento.md`"], "Deploy (M1).")),
  ("ADR3", "ADR-003: multi-gabinete e multi-empresa", ["docs", "spike", "mod:plataforma"], "Média", "S", "Backlog",
   corpo("Como isolar os dados de cada gabinete e de cada empresa cliente.",
         ["Opções comparadas", "Decisão em `docs/adr/003-multi-empresa.md`"], "Implementação.")),
  ("ADR4", "ADR-004: proteção de dados, RGPD e sigilo profissional", ["docs", "spike", "mod:plataforma"], "Alta", "M", "Backlog",
   corpo("Dados fiscais e bancários de clientes do gabinete; sigilo dos contabilistas certificados (OCC).",
         ["Regras de tratamento, retenção e acesso", "Decisão em `docs/adr/004-protecao-dados.md`"], "—")),
  ("ADR5", "ADR-005: processo de desenvolvimento e testes", ["docs", "spike", "mod:plataforma"], "Média", "S", "Backlog",
   corpo("Reaproveitar o processo do CoopHousing (ADR-004/005 de lá) e adaptar.",
         ["Processo de review (Claude `/code-review` + CodeRabbit?)", "Estratégia de testes", "Decisão em `docs/adr/005-processo-testes.md`"], "—")),
  ("T14", "Ordem dos épicos das conciliações", ["docs", "mod:plataforma"], "Alta", "S", "Backlog",
   corpo("Decidir qual conciliação vem primeiro, pela dor maior encontrada no levantamento.",
         ["E2–E4 renumerados por ordem, se preciso", "Milestones M2–M4 atribuídas aos épicos"], "—")),
]

def cria(chave, titulo, labels, milestone, body):
    args = ["issue", "create", "-R", R, "--title", titulo, "--body-file", "-"]
    for l in labels: args += ["--label", l]
    if milestone: args += ["--milestone", milestone]
    url = gh(*args, inp=body)
    num = int(url.rsplit("/", 1)[1])
    print(f"{chave}: #{num} {titulo}")
    return num, url

def item(url, campos):
    item_id = gh("project", "item-add", PN, "--owner", OWNER, "--url", url, "--format", "json", "--jq", ".id")
    for campo, valor in campos.items():
        if FIELD_VALUE:
            gh("project", "item-edit", PN, "--owner", OWNER, "--id", item_id, "--field", campo, "--value", valor)
        else:
            pid = gh("project", "view", PN, "--owner", OWNER, "--format", "json", "--jq", ".id")
            f = json.loads(gh("project", "field-list", PN, "--owner", OWNER, "--format", "json"))
            fld = next(x for x in f["fields"] if x["name"] == campo)
            opt = next(o["id"] for o in fld["options"] if o["name"] == valor)
            gh("project", "item-edit", "--id", item_id, "--project-id", pid, "--field-id", fld["id"], "--single-select-option-id", opt)

def sub(pai, filho):
    fid = gh("api", f"repos/{R}/issues/{filho}", "--jq", ".id")
    gh("api", "-X", "POST", f"repos/{R}/issues/{pai}/sub_issues", "-F", f"sub_issue_id={fid}")

if gh("issue", "list", "-R", R, "--state", "all", "-L", "1", "--json", "number", "--jq", "length") != "0":
    sys.exit("O repositório já tem issues: abortar para não duplicar.")

mapa = {}
e0, e0_url = cria(*EPICOS[0]); mapa["E0"] = e0
item(e0_url, {"Status": "Em curso", "Prioridade": "Alta", "Tamanho": "L"})
for chave, titulo, labels, prio, tam, status, body in TICKETS:
    n, url = cria(chave, titulo, labels, M0, body + f"\nÉpico: #{e0}\n")
    mapa[chave] = n
    sub(e0, n)
    item(url, {"Status": status, "Prioridade": prio, "Tamanho": tam})
for chave, titulo, labels, ms, body in EPICOS_DEPOIS:
    n, url = cria(chave, titulo, labels, ms, body)
    mapa[chave] = n
    item(url, {"Status": "Backlog"})
json.dump(mapa, open(OUT, "w"), indent=2)
print(json.dumps(mapa))
```

- [ ] **Passo 2: Correr o script**

```bash
python3 -I <scratchpad>/criar_issues.py $PN <1 se SINTAXE_OK, 0 se não> <scratchpad>/issues.json
```

Esperado: 20 linhas `chave: #n título` e o mapa JSON.

- [ ] **Passo 3: Verificar**

```bash
gh issue list -R RafaelFerraz2002/bate-certo --state open -L 50 --json number,title,milestone --jq '.[]|"\(.number) \(.title) [\(.milestone.title // "-")]"'
gh api repos/RafaelFerraz2002/bate-certo/issues/1/sub_issues --jq 'length'
gh project item-list $PN --owner RafaelFerraz2002 --format json --jq '.items|length'
cat <scratchpad>/issues.json
```

Esperado:
- 20 issues;
- 14 sub-issues no E0;
- 20 itens no Project;
- `T1` e `T2` em "Em curso".

Se `T1 ≠ 2` ou `T2 ≠ 3`, usar os números do mapa em tudo o que vem a seguir.

---

### Task 5: Notion (ticket T1)

**Interfaces:**
- Consome: o mapa `issues.json` e `$PN`.
- Produz: os IDs que a Task 6 escreve no `CLAUDE.md` privado:
  - página principal `BC_PAGE`;
  - página Levantamento `BC_LEV`;
  - bases `collection://` Requisitos `BC_REQ`, Glossário `BC_GLO` e Diário `BC_DIA`;
  - página Diário de aprendizagem `BC_APR`;
  - página Manual `BC_MAN`;
  - página Decisões `BC_ADR`.

  Guardá-los em `<scratchpad>/notion.json`.

Carregar antes as ferramentas: `ToolSearch select:mcp__claude_ai_Notion__notion-create-pages,mcp__claude_ai_Notion__notion-create-database,mcp__claude_ai_Notion__notion-update-page`.

- [ ] **Passo 1: Criar a página principal `⚖️ Bate Certo`, filha de 🗂️ Projetos (`3f9fe701-a459-8109-a71a-cdd5a8e4a3a7`)**

Conteúdo (substituir `#E0` e `$PN` pelos valores reais):

```markdown
Plataforma de conciliações contabilísticas (bancária, contas correntes de terceiros, e-Fatura/AT). Primeiro para um gabinete de contabilidade concreto; depois, se ficar boa, para vender; sempre portefólio e aprendizagem.

## Backlog (GitHub)
Os tickets vivem no GitHub; o Notion é a documentação. Cada requisito tem a propriedade **Ticket** com o épico onde vai ser feito.
- Quadro: [GitHub Project — Bate Certo](https://github.com/users/RafaelFerraz2002/projects/$PN)
- Repositório: [bate-certo](https://github.com/RafaelFerraz2002/bate-certo)
- Épicos: [label epic](https://github.com/RafaelFerraz2002/bate-certo/issues?q=label%3Aepic) — começar pelo [E0](https://github.com/RafaelFerraz2002/bate-certo/issues/#E0)
- Fases: [milestones](https://github.com/RafaelFerraz2002/bate-certo/milestones) — M0 Levantamento e Arquitetura → M1 Fundação → M2 Primeira conciliação (piloto) → M3 → M4 → M5 Comercialização

**Forma de trabalhar:** escolher um ticket "Pronto" → branch `tipo/#nr-descricao` → PR com `Closes #nr` → CI verde → review → tu aprovas → merge. Cada épico só é partido em tickets antes de começar.

**Regra de ouro:** nunca dados reais de clientes do gabinete (NIFs, nomes, extratos, SAF-T, valores) no GitHub nem no Notion.
```

Verificar com `notion-fetch` que o `ancestor-path` contém "Projetos".

- [ ] **Passo 2: Criar a subpágina `🎯 Visão e âmbito`**

```markdown
## Propósito
**Bate Certo** começa pelas conciliações. "Bater certo" é o que um contabilista diz quando uma conciliação fecha.

| Para quem | Ordem |
|---|---|
| Um gabinete de contabilidade concreto | 1.º: resolver uma dor real |
| Outros gabinetes / PMEs | 2.º: vender, se ficar boa |
| Portefólio e aprendizagem | Sempre |

## O que sabemos
- O gabinete faz lançamentos e conciliações no **Primavera**.
- Conciliações no âmbito: **bancária**, **contas correntes de clientes/fornecedores**, **e-Fatura/AT**.
- Avança-se feature a feature, com tickets no GitHub.

## O que ainda não sabemos (é o objetivo do M0)
- Que dor o Primavera não resolve.
- Tempo atual por conciliação e número de empresas (linha de base).
- Quem usaria a plataforma.
- Se os dados podem estar na cloud ou têm de ficar na rede do gabinete.
- O que acontece às diferenças: só sugerir, ou gerar lançamentos para o Primavera?

## Critérios de sucesso
- **M0:** requisitos das três conciliações validados pelo gabinete, ADRs 001–005 aceites, ordem dos épicos decidida, linha de base registada.
- **Piloto (M2):** o gabinete usa a primeira conciliação em trabalho real e gasta menos tempo do que na linha de base.

## Fora de âmbito (por agora)
- Outras conciliações (Estado, cartões/TPA, caixa, intercompany).
- Lançamentos contabilísticos no Bate Certo (substituir o Primavera).
```

- [ ] **Passo 3: Criar a subpágina `🔎 Levantamento no gabinete` com a base `Requisitos` inline**

Texto da página:

```markdown
<callout>Níveis de confiança: ✅ Confirmado (vi no ecrã) · ⚠️ Deduzido (precisa de validação) · ❓ Não verificado. O guião, as notas da sessão e o relatório entram aqui nos tickets do guião e dos requisitos.</callout>
```

Esquema da base **Requisitos**, igual ao do CoopHousing com o Módulo adaptado:
- **Requisito:** título
- **ID:** texto (ex.: `CB-01`)
- **Módulo:** seleção — `Global` (gray), `Conciliação bancária` (blue), `Contas correntes de terceiros` (green), `e-Fatura/AT` (orange), `Plataforma` (purple)
- **Tipo:** seleção — `Funcional` (blue), `Regra de negócio` (orange), `Não funcional` (purple), `Dados/Campos` (green), `Permissões` (red)
- **Confiança:** seleção — `✅ Confirmado` (green), `⚠️ Deduzido` (yellow), `❓ Não verificado` (red)
- **Validação:** seleção — `Por validar` (gray), `Validado` (green), `Corrigir` (orange), `Rejeitado` (red)
- **Origem:** seleção — `Levantamento Claude` (blue), `Utilizador` (green), `Gabinete` (yellow)
- **Prioridade:** seleção — `Alta` (red), `Média` (yellow), `Baixa` (gray)
- **Onde foi visto:** texto (ecrã do Primavera ou passo do processo)
- **Detalhe:** texto
- **Notas:** texto
- **Ticket:** URL (descrição "Link do épico/ticket no GitHub")

- [ ] **Passo 4: Criar a subpágina `📚 Diário de aprendizagem` com as bases `Glossário` e `Diário`**

Texto da página:

```markdown
Aprender contabilidade e engenharia enquanto se constrói. No fim de cada ticket fechado: uma entrada no **Diário** e os termos novos no **Glossário**.
```

**Glossário:**
- **Termo:** título
- **Explicação:** texto (descrição "Uma ou duas frases, sem jargão")
- **No nosso projeto:** texto (descrição "Onde/porquê aparece aqui")
- **Área:** seleção — `Contabilidade` (brown), `Fiscal/AT` (red), `Banca` (blue), `Processo` (pink), `Arquitetura` (purple), `Backend` (blue), `Frontend` (green), `Base de dados` (orange), `Qualidade` (yellow), `Infra/Deploy` (gray), `Segurança/RGPD` (red)
- **Ticket:** texto (descrição "Ex.: #4 / ADR-001")

**Diário:**
- **Entrada:** título
- **Ticket:** URL
- **Data:** data
- **O que se fez**, **O que aprendi**, **Dúvidas**, **Termos novos:** texto

- [ ] **Passo 5: Preencher o Glossário com os 30 termos iniciais (Ticket = `#<T1>`)**

| Termo | Área | Explicação | No nosso projeto |
|---|---|---|---|
| Conciliação | Contabilidade | Comparar dois registos do mesmo dinheiro (por exemplo, o banco e a contabilidade) e explicar todas as diferenças até "bater certo". | É o que o Bate Certo faz: as três primeiras são a bancária, a de terceiros e a do e-Fatura. |
| Conciliação bancária | Banca | Comparar o extrato do banco com os movimentos da conta do banco na contabilidade (conta 12). | Primeiro candidato a épico (E2). |
| Extrato bancário | Banca | Lista dos movimentos de uma conta num período, emitida pelo banco. | Uma das duas "metades" da conciliação bancária; chega em PDF, CSV, MT940, CAMT.053 ou Open Banking. |
| Lançamento | Contabilidade | Registo de um facto na contabilidade: data, documento, contas a débito e a crédito, valores. | Feito pelo gabinete no Primavera; o Bate Certo lê-os, não os cria (por agora). |
| Débito e crédito | Contabilidade | Os dois lados de cada lançamento. Numa conta de banco, débito = entra dinheiro, crédito = sai dinheiro. | Saber o lado é essencial para emparelhar um movimento do banco com um lançamento. |
| Partidas dobradas | Contabilidade | Regra base: cada lançamento tem total a débito igual ao total a crédito. | Validação útil ao importar lançamentos. |
| Plano de contas | Contabilidade | Lista organizada das contas usadas pela empresa (ex.: 12 Depósitos à ordem, 21 Clientes). | Importado do SAF-T no M1. |
| SNC | Contabilidade | Sistema de Normalização Contabilística: as regras e o código de contas da contabilidade em Portugal. | Define a numeração das contas que vamos ver (12, 21, 22, 24…). |
| Conta 12 – Depósitos à ordem | Contabilidade | Conta onde se regista o dinheiro em contas bancárias. Normalmente há uma subconta por banco (121, 122…). | É o lado "contabilidade" da conciliação bancária. |
| Conta corrente (de terceiro) | Contabilidade | Histórico de tudo o que se deve a (ou se tem a receber de) um cliente ou fornecedor: faturas, pagamentos, notas de crédito. | Base da conciliação de terceiros (E3). |
| Conta 21 – Clientes / 22 – Fornecedores | Contabilidade | Contas do SNC para o que os clientes devem à empresa (21) e o que a empresa deve aos fornecedores (22). | Lado "contabilidade" da conciliação de terceiros. |
| Razão (extrato de conta) | Contabilidade | Lista de todos os lançamentos de uma conta, com saldo acumulado. | É o que se compara com o extrato do banco ou do fornecedor. |
| Balancete | Contabilidade | Resumo dos saldos de todas as contas num momento. | Serve para confirmar que os dados importados batem com o Primavera. |
| Diário (contabilístico) | Contabilidade | Agrupamento de lançamentos por tipo (bancos, compras, vendas…). Não confundir com o Diário de aprendizagem. | Os lançamentos do SAF-T vêm organizados por diário. |
| Documento de suporte | Contabilidade | O papel ou ficheiro que justifica um lançamento: fatura, recibo, extrato. | Uma diferença de conciliação costuma ser um documento em falta. |
| Movimento pendente (em aberto) | Contabilidade | Movimento que ainda não tem par do outro lado (por exemplo, um cheque emitido que o banco ainda não pagou). | O resultado de uma conciliação é a lista dos pendentes e das diferenças. |
| Diferença de conciliação | Contabilidade | Valor que não bate entre os dois lados e precisa de explicação ou correção (comissão bancária não lançada, erro de valor…). | O que o gabinete tem de resolver; ver "o caminho de volta para o Primavera". |
| Emparelhamento (matching) | Backend | Ligar um movimento de um lado ao seu par do outro lado: por valor, data, referência ou descrição. | O coração técnico do Bate Certo; automatizá-lo é onde se poupa tempo. |
| Fecho do mês (período) | Processo | Fechar a contabilidade de um mês depois de tudo lançado e conciliado. | As conciliações são feitas tipicamente no fecho, por empresa. |
| SAF-T (PT) | Fiscal/AT | Ficheiro XML normalizado, exigido pela AT, com os dados contabilísticos (ou de faturação) de uma empresa. Todos os programas certificados o exportam. | Candidato a fonte principal dos lançamentos (spike de formatos). |
| e-Fatura | Fiscal/AT | Portal da AT onde ficam as faturas comunicadas pelos emitentes, associadas ao NIF do adquirente. | Um dos lados da conciliação e-Fatura (E4). |
| ATCUD | Fiscal/AT | Código único de cada documento fiscal, impresso na fatura. | Chave forte para emparelhar faturas do e-Fatura com lançamentos. |
| NIF | Fiscal/AT | Número de Identificação Fiscal. | Identifica clientes e fornecedores; é dado pessoal: nunca usar NIFs reais. |
| IVA dedutível e liquidado | Fiscal/AT | IVA pago nas compras que a empresa pode recuperar (dedutível) e IVA cobrado nas vendas que tem de entregar ao Estado (liquidado). | As diferenças de IVA são um dos objetivos da conciliação e-Fatura. |
| Contabilista Certificado (CC) | Processo | Profissional inscrito na Ordem dos Contabilistas Certificados (OCC), responsável pela contabilidade dos clientes e obrigado a sigilo. | Utilizador provável do Bate Certo; o sigilo dele define as regras de dados. |
| Gabinete de contabilidade | Processo | Empresa que faz a contabilidade de várias outras empresas (os seus clientes). | O primeiro cliente do Bate Certo. |
| Empresa (no Primavera) | Contabilidade | Cada cliente do gabinete é uma empresa separada no Primavera, com a sua contabilidade. | O Bate Certo tem de lidar com muitas empresas por gabinete. |
| Primavera | Processo | Software de gestão e contabilidade português (ERP), usado pelo gabinete. | O sistema que o Bate Certo complementa: lemos dados dele e, talvez, devolvemos lançamentos. |
| MT940 / CAMT.053 | Banca | Formatos normalizados de extrato bancário: MT940 (SWIFT, texto) e CAMT.053 (ISO 20022, XML). | Alternativas ao CSV de cada banco, a avaliar no spike de formatos. |
| Open Banking (PSD2) | Banca | Diretiva europeia que permite, com autorização, ler os movimentos de uma conta por API através de um prestador autorizado (ex.: Enable Banking). | Possível forma de obter extratos sem ficheiros. |

- [ ] **Passo 6: Criar a subpágina `📖 Manual da app`**

```markdown
Como usar o Bate Certo. Cada funcionalidade só passa a "Feito" no quadro quando a sua página aqui estiver escrita.

*Ainda não há funcionalidades: a primeira chega no M2.*
```

- [ ] **Passo 7: Criar a subpágina `🧭 Decisões (ADR)` (substituir `#ADRn` pelos números do mapa)**

```markdown
Os ADRs vivem no repositório, em [`docs/adr/`](https://github.com/RafaelFerraz2002/bate-certo/tree/main/docs/adr), revistos por PR. Aqui fica só o índice.

| ADR | Título | Estado | Ticket |
|---|---|---|---|
| 001 | Escolha da stack | Por escrever | #ADR1 |
| 002 | Alojamento e servidor | Por escrever | #ADR2 |
| 003 | Multi-gabinete e multi-empresa | Por escrever | #ADR3 |
| 004 | Proteção de dados, RGPD e sigilo profissional | Por escrever | #ADR4 |
| 005 | Processo de desenvolvimento e testes | Por escrever | #ADR5 |
```

Os `#ADRn` ficam como links para `https://github.com/RafaelFerraz2002/bate-certo/issues/<n>`.

- [ ] **Passo 8: Verificar e guardar os IDs**

- Fazer `notion-fetch` da página principal e confirmar as 5 subpáginas e o ancestral "Projetos".
- Fazer `notion-fetch` de cada base e confirmar o esquema.
- Contar as linhas do Glossário com `notion-query-data-sources`: esperado 30.
- Escrever `<scratchpad>/notion.json` com `BC_PAGE`, `BC_LEV`, `BC_REQ`, `BC_APR`, `BC_GLO`, `BC_DIA`, `BC_MAN` e `BC_ADR`.

- [ ] **Passo 9: Passar o T1 para revisão**

```bash
gh project item-edit $PN --owner RafaelFerraz2002 --url https://github.com/RafaelFerraz2002/bate-certo/issues/<T1> --field Status --value "Em revisão"
gh issue comment <T1> -R RafaelFerraz2002/bate-certo --body "Notion montado: <link da página principal>. Falta a tua revisão; fecha o ticket quando estiver bem."
```

Se a sintaxe `--url/--field/--value` falhou na Task 3, usar o fallback por IDs do script.

---

### Task 6: Ficheiros do repositório e `CLAUDE.md` (ticket T2)

**Ficheiros:**
- Criar: `CLAUDE.md`, `.github/ISSUE_TEMPLATE/ticket.md`, `.github/ISSUE_TEMPLATE/config.yml`, `.github/pull_request_template.md`, `.github/workflows/pr-title.yml`, `.github/dependabot.yml`, `docs/adr/README.md`
- Criar fora do repositório: `Desktop/Rich baby/Contabilidade/CLAUDE.md`

**Interfaces:**
- Consome: `issues.json`, `notion.json` e `$PN`.
- Produz: o check `Validar título` (nome do job), exigido no ruleset na Task 7.

- [ ] **Passo 1: Criar o branch**

```bash
cd "/Users/enlt-rferraz/Desktop/Rich baby/Contabilidade/bate-certo"
git switch -c "chore/#<T2>-organizacao-repo"
```

- [ ] **Passo 2: `.github/ISSUE_TEMPLATE/ticket.md` e `config.yml`**

```markdown
---
name: Ticket
about: Funcionalidade, tarefa ou decisão do backlog
title: ""
labels: []
---

## Contexto
<!-- Porquê este ticket existe e que problema resolve. -->

## Requisitos
<!-- IDs da base Requisitos no Notion, ex.: CB-03, EF-01 -->

## Critérios de aceitação
- [ ] 

## Fora de âmbito
<!-- O que este ticket NÃO faz. -->

## Notas
<!-- Nunca dados reais de clientes do gabinete. -->
```

```yaml
blank_issues_enabled: true
```

- [ ] **Passo 3: `.github/pull_request_template.md`**

```markdown
## O que muda
<!-- Lista curta do que muda e porquê. -->

Closes #
```

- [ ] **Passo 4: `.github/workflows/pr-title.yml`**

```yaml
name: Título do PR

on:
  pull_request:
    types: [opened, edited, synchronize, reopened]

permissions:
  pull-requests: read

jobs:
  titulo:
    name: Validar título
    runs-on: ubuntu-latest
    steps:
      - uses: amannn/action-semantic-pull-request@v5.5.3
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          types: |
            feat
            fix
            docs
            chore
            refactor
            test
            ci
            build
            perf
            revert
          requireScope: false
```

- [ ] **Passo 5: `.github/dependabot.yml`**

```yaml
version: 2
updates:
  - package-ecosystem: github-actions
    directory: /
    schedule:
      interval: weekly
    commit-message:
      prefix: ci
```

- [ ] **Passo 6: `docs/adr/README.md` (substituir `<ADRn>` pelos números do mapa)**

````markdown
# Architecture Decision Records (ADR)

Um ADR regista **uma decisão de arquitetura**: o contexto, as opções que foram pesadas, o que se decidiu e as consequências. Serve para que, daqui a meses, se perceba *porquê* o sistema é assim, e não só *como* é.

## Regras

- Um ficheiro por decisão: `NNN-titulo-curto.md`, com numeração sequencial.
- Cada ADR nasce de um ticket no GitHub e entra por PR, como o resto do código.
- Um ADR aceite não se reescreve. Se a decisão mudar, cria-se um ADR novo com estado *Substitui ADR-NNN*, e o antigo passa a *Substituído por ADR-MMM*.
- Apresentar sempre o porquê **e** as alternativas rejeitadas.
- Escritos em português de Portugal. O índice é espelhado na página "Decisões (ADR)" do Notion.

## Índice

| ADR | Título | Estado | Ticket |
|---|---|---|---|
| 001 | Escolha da stack | Por escrever | [#<ADR1>](https://github.com/RafaelFerraz2002/bate-certo/issues/<ADR1>) |
| 002 | Alojamento e servidor | Por escrever | [#<ADR2>](https://github.com/RafaelFerraz2002/bate-certo/issues/<ADR2>) |
| 003 | Multi-gabinete e multi-empresa | Por escrever | [#<ADR3>](https://github.com/RafaelFerraz2002/bate-certo/issues/<ADR3>) |
| 004 | Proteção de dados, RGPD e sigilo profissional | Por escrever | [#<ADR4>](https://github.com/RafaelFerraz2002/bate-certo/issues/<ADR4>) |
| 005 | Processo de desenvolvimento e testes | Por escrever | [#<ADR5>](https://github.com/RafaelFerraz2002/bate-certo/issues/<ADR5>) |

## Modelo

```markdown
# ADR-NNN: Título

- **Estado:** Proposto | Aceite | Substituído por ADR-MMM
- **Data:** AAAA-MM-DD
- **Ticket:** #NN
- **Requisitos:** IDs da base Requisitos (ex.: CB-03)

## Contexto
O problema, as restrições e os critérios de decisão.

## Opções consideradas
As alternativas, com prós e contras.

## Decisão
O que se escolheu e porquê.

## Consequências
O que fica mais fácil, o que fica mais difícil, os riscos e o que fica para outros ADRs.
```
````

- [ ] **Passo 7: `CLAUDE.md` público (substituir `$PN` e `<T*>`)**

````markdown
# Bate Certo

Plataforma de conciliações contabilísticas: **bancária**, **contas correntes de clientes/fornecedores** e **e-Fatura/AT**. Primeiro para um gabinete de contabilidade concreto, que lança e concilia no Primavera; depois, se ficar boa, para vender. Também é portefólio e aprendizagem.

## Onde está cada coisa

- **Spec do arranque:** `docs/superpowers/specs/2026-10-14-arranque-bate-certo-design.md`.
- **Decisões de arquitetura:** `docs/adr/`. Começa pelo `docs/adr/README.md`. **Não contrariar um ADR aceite**; se algo pedir uma mudança, propõe-se um ADR novo.
- **Tickets:** GitHub Issues + Project "Bate Certo" (`gh project view $PN --owner RafaelFerraz2002`). Épicos = issues com a label `epic` (`[E0]`…`[E5]`); as fases são as milestones M0–M5.
- **Requisitos funcionais, glossário, manual e diário:** no Notion. Os IDs dos requisitos (`CB-`, `CT-`, `EF-`, `GL-`) são referidos na secção "Requisitos" de cada ticket.

## Como se trabalha

1. Todo o trabalho nasce de um ticket. Se não houver ticket, cria-se um antes de começar.
2. Branch `tipo/#nr-descricao` (ex.: `feat/#42-importar-extrato`) a partir de `main`, atualizado.
3. Tickets de arquitetura ou funcionalidades novas passam por brainstorming → desenho aprovado → plano, **antes** de escrever código.
4. PR para `main` com `Closes #nr` e título em Conventional Commits (`feat(bancaria): …`, `docs(adr): …`). Entra por **squash merge**. A descrição do PR tem só "O que muda" (e um resumo, se ajudar) e `Closes #nr`. Sem secção "Como rever", sem rodapé "Generated with Claude Code" e sem link da sessão. Os commits também não levam a linha `Claude-Session`.
5. Ao começar um ticket, passa para "Em curso"; ao abrir o PR, para "Em revisão". Só o utilizador aprova o merge.
   Estados possíveis: `Backlog`, `Pronto`, `Em curso`, `Em revisão`, `Feito`.
   ```sh
   gh project item-edit $PN --owner RafaelFerraz2002 --url <url-do-issue> --field Status --value "Em curso"
   ```
6. **Nunca** fazer push direto para `main`, force push, nem merge sem pedido explícito.
7. Uma funcionalidade só está "Feita" quando a sua página do **Manual da app** no Notion estiver escrita.

## Regras que não se quebram

- **Nunca pôr dados reais** de clientes do gabinete (NIFs, nomes, extratos, SAF-T, valores, faturas) no repositório, em seeds, em testes, em issues nem no Notion. O repositório é público. Os dados de exemplo são fictícios ou vêm da empresa de demonstração do Primavera.
- **Nunca pôr segredos** (passwords, chaves, tokens, certificados de Open Banking) em ficheiros versionados. Só em variáveis de ambiente.
- Qualquer funcionalidade que guarde, mostre ou exporte dados fiscais ou bancários segue o ADR-004 (proteção de dados e sigilo profissional), quando existir. Na dúvida, pergunta-se.
- O repositório não tem licença open source, de propósito: o código é do autor.

## Convenções

- **Língua:** documentação, issues, PRs e mensagens ao utilizador em **português de Portugal**. A convenção de nomes no código decide-se com a stack (ADR-001). Até lá, não se escreve código de produto.
- **Decisões técnicas:** apresentar sempre o porquê **e** as alternativas rejeitadas. Escolher pelo que o projeto precisa.
- **Termos de contabilidade:** explicar sem jargão e acrescentar ao Glossário do Notion.
- **Comandos de build, teste e lint:** ainda não existem. São definidos no scaffolding (M1) e acrescentados aqui nessa altura.
````

- [ ] **Passo 8: `CLAUDE.md` privado em `Desktop/Rich baby/Contabilidade/CLAUDE.md` (fora do git; substituir os IDs do `notion.json`)**

````markdown
# Bate Certo: contexto privado (fora do repositório público)

Este ficheiro **não está versionado**: fica na pasta-mãe de `bate-certo/`. O Claude Code lê-o em conjunto com o `CLAUDE.md` do repositório. Aqui fica o que não pode ser público.

## Notion
- Página principal: `BC_PAGE` (dentro de 🗂️ Projetos `3f9fe701-a459-8109-a71a-cdd5a8e4a3a7`)
- Levantamento no gabinete: `BC_LEV`. Base **Requisitos**: `BC_REQ` (ID, Módulo, Tipo, Confiança, Validação, Origem, Prioridade, Onde foi visto, Detalhe, Notas, Ticket)
- **Diário de aprendizagem**: `BC_APR`
  - Glossário: `BC_GLO` (Termo, Explicação, No nosso projeto, Área, Ticket)
  - Diário: `BC_DIA` (Entrada, Ticket, Data, O que se fez, O que aprendi, Dúvidas, Termos novos)
- Manual da app: `BC_MAN` · Decisões (ADR): `BC_ADR`

## No início de cada sessão
O merge é feito pelo utilizador, por isso não sei quando um ticket fechou. Verificar:
```sh
gh pr list -R RafaelFerraz2002/bate-certo --state merged -L 10
gh issue list -R RafaelFerraz2002/bate-certo --state closed -L 10
```
Para cada PR merged ou ticket fechado que ainda não tenha entrada no **Diário**:
1. Criar uma entrada no Diário e acrescentar ao **Glossário** os termos novos (contabilidade e técnicos), com uma explicação curta e sem jargão, e onde aparecem no projeto.
2. Se o ticket mexeu em requisitos, atualizar a propriedade Ticket ou Notas na base Requisitos.
3. Se mexeu num ADR, atualizar o índice na página Decisões (ADR).

Depois disso, propor o próximo ticket do quadro.

## Ao começar cada ticket
- Dizer se vale a pena mudar o **effort** ou o **modelo**: high com Opus para arquitetura, segurança, dados fiscais e multi-empresa; medium ou Sonnet para tickets mecânicos.
- Dizer se uma estratégia com **agentes** compensa (pesquisa em paralelo, implementação + review, etc.) e quanto custa mais ou menos.

## O utilizador e o domínio
- O utilizador está a aprender contabilidade: explicar os termos e não assumir conhecimento do domínio.
- O que o gabinete faz valida-se com o gabinete; não se inventa. Hipóteses ficam marcadas com ⚠️ ou ❓.

## Gabinete de contabilidade (sigilo profissional)
- Os contabilistas certificados estão obrigados a sigilo (OCC). Nunca copiar, fotografar nem transcrever dados reais de clientes do gabinete: nem para o repositório, nem para o Notion, nem para a conversa.
- Ficheiros de exemplo só da **empresa de demonstração do Primavera** ou fictícios.
- Se o utilizador colar dados que pareçam reais (NIFs, nomes de empresas, IBANs, valores de faturas), avisar e não os guardar em lado nenhum.
- Se algum dia houver acesso ao Primavera do gabinete pelo browser: **só leitura**, nunca gravar, lançar, apagar ou fechar períodos; as passwords são sempre escritas pelo utilizador; no fim, relatório do que pode ter sido alterado.
````

- [ ] **Passo 9: Verificar os ficheiros (sem dados reais, YAML válido, sem placeholders por substituir)**

```bash
python3 -I -c "import sys,yaml;[yaml.safe_load(open(f)) for f in sys.argv[1:]];print('YAML_OK')" .github/workflows/pr-title.yml .github/dependabot.yml .github/ISSUE_TEMPLATE/config.yml
grep -rnE '<T[0-9]+>|<ADR[0-9]>|\$PN|BC_[A-Z]{3}' CLAUDE.md docs/adr/README.md ../CLAUDE.md && echo "FALTA SUBSTITUIR" || echo "SEM_PLACEHOLDERS"
git status --short
```

Esperado:
- `YAML_OK` (se faltar o `yaml`, validar com `gh workflow view` depois do push);
- `SEM_PLACEHOLDERS`;
- só os 7 ficheiros novos no `git status` (o `../CLAUDE.md` está fora do repositório).

- [ ] **Passo 10: Commit, push e PR**

```bash
git add CLAUDE.md .github docs/adr
git commit -m "chore: organização do repositório (templates, Actions, ADRs e CLAUDE.md)

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>"
git push -u origin HEAD
gh pr create -R RafaelFerraz2002/bate-certo --base main \
  --title "chore: organização do repositório (templates, Actions, ADRs e CLAUDE.md)" \
  --body "## O que muda
- \`CLAUDE.md\` com a orientação do projeto (fluxo de tickets, regras de dados e sigilo, convenções).
- Templates de ticket e de PR.
- Action que valida o título do PR em Conventional Commits.
- Dependabot para as Actions.
- \`docs/adr/README.md\` com regras, índice e modelo dos ADRs.

Closes #<T2>"
```

- [ ] **Passo 11: Verificar o check no PR**

```bash
gh pr checks --watch -R RafaelFerraz2002/bate-certo
```

Esperado: `Validar título` aprovado. Depois, confirmar que o T2 está em "Em revisão" no quadro (pelo workflow nativo, ou manualmente com `item-edit`). **Parar aqui: o merge é do utilizador.**

---

### Task 7: Fecho (depois do merge do utilizador)

**Interfaces:**
- Consome: o PR da Task 6 com merge feito e `$PN`.
- Produz: o ruleset com o check obrigatório, a memória atualizada e o próximo ticket proposto.

- [ ] **Passo 1: Confirmar o merge**

```bash
gh pr list -R RafaelFerraz2002/bate-certo --state merged -L 5
git switch main && git pull --ff-only
```

- [ ] **Passo 2: Tornar obrigatório o check `Validar título` no ruleset**

```bash
RID=$(gh api repos/RafaelFerraz2002/bate-certo/rulesets --jq '.[]|select(.name=="main").id')
gh api -X PUT repos/RafaelFerraz2002/bate-certo/rulesets/$RID --input - <<'EOF'
{
  "name": "main",
  "target": "branch",
  "enforcement": "active",
  "conditions": {"ref_name": {"include": ["~DEFAULT_BRANCH"], "exclude": []}},
  "rules": [
    {"type": "deletion"},
    {"type": "non_fast_forward"},
    {"type": "pull_request", "parameters": {
      "required_approving_review_count": 0,
      "dismiss_stale_reviews_on_push": false,
      "require_code_owner_review": false,
      "require_last_push_approval": false,
      "required_review_thread_resolution": false,
      "allowed_merge_methods": ["squash"]}},
    {"type": "required_status_checks", "parameters": {
      "strict_required_status_checks_policy": false,
      "required_status_checks": [{"context": "Validar título"}]}}
  ]
}
EOF
gh api repos/RafaelFerraz2002/bate-certo/rulesets/$RID --jq '[.rules[].type]'
```

Esperado: `["deletion","non_fast_forward","pull_request","required_status_checks"]`.

- [ ] **Passo 3: Diário e Glossário (rotina do `CLAUDE.md` privado)**

- Criar uma entrada no Diário "Arranque do projeto" (Ticket = URL do T2), com o que se fez e o que se aprendeu (SNC, conciliação, rulesets, Projects v2).
- Acrescentar ao Glossário os termos técnicos novos que surgiram: `Ruleset`, `Conventional Commits`, `Squash merge`, `Dependabot`, `Secret scanning`.

- [ ] **Passo 4: Atualizar a memória**

Atualizar `projeto-contabilidade.md` na memória com:
- o nome Bate Certo;
- o repositório;
- o número do Project;
- o ID da página Notion;
- o próximo passo.

- [ ] **Passo 5: Propor o próximo ticket**

Com o T1 e o T2 feitos, propor os que não dependem do gabinete e podem correr em paralelo: **guião** (T3), **spike de formatos** (T4) e **spike de mercado** (T5). O ADR-005 (T13 do M0) também não depende do gabinete.
