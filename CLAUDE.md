# Bate Certo

Plataforma de conciliações contabilísticas: **bancária**, **contas correntes de clientes/fornecedores** e **e-Fatura/AT**. Primeiro para um gabinete de contabilidade concreto, que lança e concilia no Primavera; depois, se ficar boa, para vender. Também é portefólio e aprendizagem.

## Onde está cada coisa

- **Spec do arranque:** `docs/superpowers/specs/2026-10-14-arranque-bate-certo-design.md`.
- **Decisões de arquitetura:** `docs/adr/`. Começa pelo `docs/adr/README.md`. **Não contrariar um ADR aceite**; se algo pedir uma mudança, propõe-se um ADR novo.
- **Tickets:** GitHub Issues + Project "Bate Certo" (`gh project view 3 --owner RafaelFerraz2002`). Épicos = issues com a label `epic` (`[E0]`…`[E5]`); as fases são as milestones M0–M5.
- **Requisitos funcionais, glossário, manual e diário:** no Notion. Os IDs dos requisitos (`CB-`, `CT-`, `EF-`, `GL-`) são referidos na secção "Requisitos" de cada ticket.

## Como se trabalha

1. Todo o trabalho nasce de um ticket. Se não houver ticket, cria-se um antes de começar.
2. Branch `tipo/#nr-descricao` (ex.: `feat/#42-importar-extrato`) a partir de `main`, atualizado.
3. Tickets de arquitetura ou funcionalidades novas passam por brainstorming → desenho aprovado → plano, **antes** de escrever código.
4. PR para `main` com `Closes #nr` e título em Conventional Commits (`feat(bancaria): …`, `docs(adr): …`). Entra por **squash merge**. A descrição do PR tem só "O que muda" (e um resumo, se ajudar) e `Closes #nr`. Sem secção "Como rever", sem rodapé "Generated with Claude Code" e sem link da sessão. Os commits também não levam a linha `Claude-Session`.
5. Ao começar um ticket, passa para "Em curso"; ao abrir o PR, para "Em revisão". Só o utilizador aprova o merge.
   Estados possíveis: `Backlog`, `Pronto`, `Em curso`, `Em revisão`, `Feito`.
   ```sh
   gh project item-edit 3 --owner RafaelFerraz2002 --url <url-do-issue> --field Status --value "Em curso"
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
