# Arranque do Bate Certo: organização e M0 (Levantamento e Arquitetura)

- **Data:** 2026-10-14
- **Estado:** Aprovado no brainstorming, à espera de revisão da spec escrita
- **Âmbito desta spec:** como o projeto se organiza (Notion, GitHub, `CLAUDE.md`) e o plano do M0. **Não** inclui a stack, o alojamento nem o modelo de dados: esses saem do M0, como ADRs.

## 1. Propósito

**Bate Certo** é uma plataforma de contabilidade ("tipo Primavera") que começa pelas **conciliações**. "Bater certo" é o que um contabilista diz quando uma conciliação fecha.

| Para quem | Ordem |
|---|---|
| Um gabinete de contabilidade concreto | 1.º: resolver uma dor real do gabinete |
| Outros gabinetes / PMEs | 2.º: vender, se a plataforma ficar boa |
| Portefólio e aprendizagem | Sempre |

### O que sabemos (dito pelo utilizador)
- O gabinete faz os lançamentos e as conciliações no **Primavera**.
- Interessam três conciliações: **bancária**, **contas correntes de clientes/fornecedores** e **e-Fatura/AT**. As outras (Estado, cartões/TPA, caixa, intercompany) ficam para depois.
- Avança-se feature a feature, com tickets no GitHub e Actions para ajudar no desenvolvimento e no planeamento.
- O utilizador consegue **conversar com o gabinete e ver as conciliações a serem feitas no ecrã**.

### O que ainda não sabemos (é para isto que serve o M0)
- Que dor o Primavera não resolve (volume de empresas, emparelhamento manual, conciliações fora do Primavera, falta de visão por cliente).
- Quanto tempo leva hoje cada conciliação e quantas empresas o gabinete tem.
- Quem usaria a plataforma (contabilista certificado, técnico, cliente do gabinete).
- Se os dados podem estar na cloud ou têm de ficar na rede do gabinete.
- O que acontece às diferenças encontradas: a plataforma só sugere, ou gera lançamentos para importar no Primavera?

### Critérios de sucesso
- **M0:** os requisitos das três conciliações estão validados pelo gabinete, os ADRs 001–005 estão aceites, a ordem dos épicos está decidida e existe a linha de base de tempos.
- **Produto (piloto, M2):** o gabinete usa a primeira conciliação em trabalho real e gasta menos tempo do que na linha de base.

## 2. Abordagem escolhida

**Levantamento primeiro, com a organização montada em paralelo.** Não se escreve código de produto nem se decide a stack antes de ver o gabinete a trabalhar.

Alternativas rejeitadas:
- **Começar já pela conciliação bancária com a stack do CoopHousing:** o risco é construir o que o Primavera já faz, e decisões como "cloud ou rede do gabinete" chegariam tarde.
- **Levar à sessão um protótipo descartável:** fica como plano B, se a primeira conversa for vaga.

## 3. Organização

O modelo é o do projeto CoopHousing, com as adaptações abaixo.

### 3.1 Pastas locais

```
Desktop/Rich baby/Contabilidade/
├─ CLAUDE.md         privado, não versionado (IDs do Notion, regras do gabinete)
└─ bate-certo/       repositório git (público)
```

### 3.2 Notion

Uma página **Bate Certo** dentro de **🗂️ Projetos** (`3f9fe701-a459-8109-a71a-cdd5a8e4a3a7`), ao lado de Coophousing e Dropshipping OS, com:

| Página / base | Conteúdo |
|---|---|
| **Visão e âmbito** | O propósito, os critérios de sucesso e o que está fora de âmbito (secção 1 desta spec) |
| **Levantamento no gabinete** | O guião, as notas da sessão e o relatório, com os níveis ✅ Confirmado / ⚠️ Deduzido / ❓ Não verificado |
| **Requisitos** (base) | Mesmo esquema da base do CoopHousing: ID, Módulo, Confiança, Validação, Origem, Ticket (URL do épico), Notas. Prefixos dos IDs: `CB-` bancária, `CT-` contas correntes de terceiros, `EF-` e-Fatura/AT, `GL-` global |
| **Diário de aprendizagem** | A base **Glossário** (Termo, Explicação, No nosso projeto, Área, Ticket) e a base **Diário** (Entrada, Ticket, Data, O que se fez, O que aprendi, Dúvidas, Termos novos). O glossário começa com cerca de 25 termos de contabilidade explicados sem jargão |
| **Manual da app** | Como usar a plataforma. Começa vazio; uma feature só passa a "Feito" quando a sua página do manual estiver escrita |
| **Decisões (ADR)** | Só o índice, com links para `docs/adr/` no repositório |
| **Backlog** | Links para o quadro, os épicos e as milestones no GitHub |

Os ADRs vivem no repositório porque são revistos por PR e ficam junto ao código. O Notion não os duplica.

### 3.3 GitHub

- **Repositório:** `RafaelFerraz2002/bate-certo`, **público**.
  - **Sem licença open source.** Código público sem licença continua a ser do autor, o que protege a venda futura.
  - A visibilidade é revista antes do M5 (comercialização).
- **Segurança:** Dependabot (alertas e atualizações), secret scanning e push protection.
- **Ruleset no `main`:** sem push direto nem force push. Entra-se por PR com squash merge.
- **Project "Bate Certo":** os mesmos campos do CoopHousing.
  - **Status:** Backlog, Pronto, Em curso, Em revisão, Feito.
  - **Prioridade, Tamanho e Requisitos** (texto com os IDs do Notion).
- **Labels:** `epic`, `feature`, `chore`, `spike`, `docs`, `bug`, `mod:plataforma`, `mod:bancaria`, `mod:terceiros`, `mod:efatura`.
- **Templates:**
  - **Ticket:** Contexto, Requisitos, Critérios de aceitação, Fora de âmbito, Notas.
  - **PR:** O que muda, `Closes #nr`.
- **Actions no M0:**
  - validação do título do PR em Conventional Commits;
  - automações do quadro: issue novo vai para Backlog, PR aberto passa a Em revisão, merge passa a Feito.

  Usam-se os workflows nativos do Project quando chegam, e Actions só para o que eles não fazem. O CI de lint, testes e build entra com o scaffolding (M1).

### 3.4 `CLAUDE.md`

- **Público** (`bate-certo/CLAUDE.md`):
  - o que é o projeto e onde está cada coisa;
  - o fluxo de trabalho: ticket → branch `tipo/#nr-descricao` → PR com `Closes #nr` → squash merge feito pelo utilizador;
  - os estados do quadro, as convenções (pt-PT, Conventional Commits) e as regras que não se quebram.
- **Privado** (`Contabilidade/CLAUDE.md`):
  - os IDs do Notion;
  - a rotina do início de sessão: ver os PRs merged, depois atualizar o Diário, o Glossário e os Requisitos;
  - a recomendação de effort, modelo ou agentes a dar ao começar cada ticket;
  - as regras de sigilo do gabinete (3.5).

### 3.5 Regras que não se quebram

- **Nunca há dados reais** de clientes do gabinete (NIFs, nomes, extratos, SAF-T, valores) no repositório, em issues, seeds, testes ou no Notion. Os dados de exemplo são fictícios ou vêm da **empresa de demonstração do Primavera**.
- **Sigilo profissional:** os contabilistas certificados estão obrigados a sigilo (OCC). Na sessão no gabinete:
  - não se fotografam ecrãs com dados reais;
  - não se levam ficheiros reais;
  - pergunta-se ao gabinete se quer um acordo de confidencialidade.
- **Nunca há segredos** em ficheiros versionados.
- Um ADR aceite não se contraria: propõe-se um ADR novo.

## 4. Plano do M0: Levantamento e Arquitetura

Milestone **M0**, épico **E0 Arranque**. São tickets no GitHub, por esta ordem:

| # | Ticket | Tipo | Quem | Depende de |
|---|---|---|---|---|
| 1 | Montar o Notion (3.2), com o glossário inicial | chore | Claude | — |
| 2 | Montar o repositório, Project, labels, milestones, templates, Actions do M0, ruleset, segurança e `CLAUDE.md` (3.3, 3.4) | chore | Claude | — |
| 3 | Guião do levantamento (ver 4.1) | docs | Claude | 1 |
| 4 | Spike: formatos e fontes de dados (ver 4.2) | spike | Claude | — |
| 5 | Spike: o que já existe e regulamentação (ver 4.3) | spike | Claude | — |
| 6 | Sessão no gabinete | — | Utilizador | 3, 4, 5 |
| 7 | Requisitos na base do Notion + relatório do levantamento (✅ ⚠️ ❓), validado com o gabinete | docs | Claude + utilizador | 6 |
| 8 | Modelo de domínio das conciliações (Empresa, Conta, Extrato, Movimento, Regra de emparelhamento, Conciliação, Diferença…) | docs | Claude + utilizador | 7 |
| 9 | ADR-001: stack | docs | Claude + utilizador | 7, 8 |
| 10 | ADR-002: alojamento e servidor (SO, cloud vs. rede do gabinete, região UE, backups) | docs | Claude + utilizador | 7 |
| 11 | ADR-003: multi-gabinete / multi-empresa (isolamento de dados) | docs | Claude + utilizador | 8 |
| 12 | ADR-004: proteção de dados, RGPD e sigilo profissional | docs | Claude + utilizador | 7, 10 |
| 13 | ADR-005: processo de desenvolvimento e testes (reaproveitar o CoopHousing) | docs | Claude + utilizador | — |
| 14 | Ordem dos épicos E2–E4, pela dor maior | docs | Utilizador | 7 |

Os tickets 1, 2, 4, 5 e 13 não dependem do gabinete e podem correr em paralelo.

### 4.1 Guião do levantamento (ticket 3)

Uma página no Notion, para levar impressa ou no portátil.

**Contexto do gabinete**
- Quantas empresas (clientes) tem.
- Quem faz as conciliações e com que frequência.
- Que versão do Primavera usa e onde está instalado (servidor local ou cloud).

**Por cada conciliação (bancária, terceiros, e-Fatura)**
- Ver o processo no ecrã, passo a passo.
- De onde vêm os dados: extrato em PDF, CSV ou homebanking; extratos de fornecedores; exportação do e-Fatura.
- Onde se perde tempo e o que se faz em Excel.
- O que acontece às diferenças encontradas, e se são lançadas no Primavera à mão.

**Linha de base:** minutos por conciliação, por empresa e por mês, e o número de empresas.

**Utilizadores:** quem usaria a plataforma, e se os clientes do gabinete teriam acesso.

**Restrições**
- Se os dados podem estar na cloud (UE) ou têm de ficar na rede do gabinete.
- Exigências de segurança.

**O que pedir:** exportações da **empresa de demonstração** do Primavera (SAF-T de contabilidade, extratos, mapas de conciliação). Nunca de clientes reais.

**Sigilo:** as regras de 3.5, e a pergunta sobre o acordo de confidencialidade.

**Fecho:** "Se pudesse mudar uma coisa nas conciliações, o que seria?"

### 4.2 Spike: formatos e fontes de dados (ticket 4)

O resultado é uma página no Notion com uma recomendação. Cobre:
- **Extratos bancários:** CSV por banco, MT940, CAMT.053.
- **Open Banking:** Enable Banking, que o utilizador já usa no projeto Firefly.
- **SAF-T (PT) de contabilidade:** estrutura e o que traz.
- **Exportação do e-Fatura:** formato e se é possível automatizá-la.
- **Primavera:**
  - o que exporta;
  - que importações aceita (para o caminho de volta das diferenças);
  - se é viável ler diretamente da sua base de dados SQL Server.

### 4.3 Spike: o que já existe e regulamentação (ticket 5)

O resultado é uma página no Notion. Cobre:
- **O que já existe:**
  - o que o Primavera faz nas três conciliações;
  - add-ons do mercado;
  - concorrentes (TOConline, Moloni e afins);
  - onde fica o espaço para o Bate Certo.
- **Regulamentação:** confirmar que um software de conciliação não precisa de certificação da AT (essa aplica-se ao software de faturação) e que obrigações tem se for vendido.

### 4.4 Sobre o SO do servidor

A decisão fica no ADR-002, porque depende do gabinete.

- **Por defeito:** Linux (Debian ou Ubuntu LTS) com Docker, num fornecedor da UE.
- **A exceção:** se o gabinete exigir que a plataforma corra na rede dele, ao lado do Primavera (normalmente Windows Server), as opções passam a ser Docker em Windows Server ou uma VM Linux local.

### 4.5 Critério de saída do M0

- Os requisitos das três conciliações estão na base Requisitos, validados pelo gabinete.
- O modelo de domínio está escrito.
- Os ADRs 001–005 estão aceites.
- A ordem de E2–E4 está decidida.
- A linha de base está registada.

## 5. Roadmap

| Milestone | Conteúdo | Épico |
|---|---|---|
| **M0 Levantamento e Arquitetura** | Secção 4 | E0 Arranque |
| **M1 Fundação** | Scaffolding, CI completo, staging, autenticação, empresas do gabinete, importação dos dados base (plano de contas e movimentos) | E1 Plataforma |
| **M2 Primeira conciliação (piloto no gabinete)** | A conciliação com mais dor, em uso real | E2 |
| **M3 Segunda conciliação** | | E3 |
| **M4 Terceira conciliação** | | E4 |
| **M5 Comercialização** | Multi-gabinete a sério, subscrição, onboarding, página pública, revisão da visibilidade do repositório | E5 |

- **E2–E4:** *Conciliação bancária*, *Contas correntes de terceiros* e *e-Fatura/AT*. A ordem sai do ticket 14.
- **Partir em tickets:** cada épico só é partido quando estiver para começar.

## 6. Fora de âmbito desta spec

- A stack, o alojamento, o modelo de dados e a autenticação: saem dos tickets 8–12.
- As conciliações além das três escolhidas.
- Lançamentos contabilísticos dentro do Bate Certo, ou seja, substituir o Primavera.
- Código de produto antes do fim do M0.
