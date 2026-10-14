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
| 001 | Escolha da stack | Por escrever | [#10](https://github.com/RafaelFerraz2002/bate-certo/issues/10) |
| 002 | Alojamento e servidor | Por escrever | [#11](https://github.com/RafaelFerraz2002/bate-certo/issues/11) |
| 003 | Multi-gabinete e multi-empresa | Por escrever | [#12](https://github.com/RafaelFerraz2002/bate-certo/issues/12) |
| 004 | Proteção de dados, RGPD e sigilo profissional | Por escrever | [#13](https://github.com/RafaelFerraz2002/bate-certo/issues/13) |
| 005 | Processo de desenvolvimento e testes | Por escrever | [#14](https://github.com/RafaelFerraz2002/bate-certo/issues/14) |

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
