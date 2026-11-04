# ADR-005: Processo de desenvolvimento e testes

- **Estado:** Aceite
- **Data:** 2026-11-04
- **Ticket:** [#14](https://github.com/RafaelFerraz2002/bate-certo/issues/14) (épico [#1](https://github.com/RafaelFerraz2002/bate-certo/issues/1))
- **Requisitos:** GL-03, CB-03, CB-04, CB-05

## Contexto

O código vai ser escrito sobretudo pelo Claude, e quem faz o merge não lê todas as linhas. Para fazer o merge com confiança, é preciso saber que o PR respeita os ADRs, que o código é bom, que nenhum dado sai de onde não deve e, acima de tudo, que **os números batem certo**. Este ADR diz como uma mudança chega ao `main` e o que se testa, como e quando.

O processo parte do que foi decidido no CoopHousing (ADR-004 e ADR-005 do `senior-care-platform`) e adapta-o. Lá, o maior risco era uma organização ver os dados de outra, e os testes centravam-se na base de dados. Aqui, o maior risco é uma conciliação errada: um emparelhamento mal feito, uma comissão com o valor errado, ou um TXT que o Primavera recusa ou importa mal.

### Critérios de decisão

| Critério | O que sabemos |
|---|---|
| Equipa | Um programador, dono do repositório e o único a aprovar merges. O código é escrito em grande parte com apoio de IA. |
| Repositório | Público. Um dado real ou um segredo publicado não se apaga de forma fiável. |
| Onde estão os riscos | Na lógica de negócio: emparelhamento, comissões, TPA e partidas dobradas. Na leitura de ficheiros externos (SAF-T, extratos) e no TXT de saída para o Primavera, que é de largura fixa. Na separação dos dados por empresa e por gabinete (sigilo profissional). |
| Quem sabe o resultado certo | O contabilista certificado, não o programador. |
| O Primavera | Corre em Windows, com licença, no servidor do gabinete. Não pode correr no CI. |
| Escala | Um gabinete, 3 a 5 pessoas, cerca de 45 empresas, centenas de movimentos por mês e por empresa. |
| Stack | Ainda não escolhida (ADR-001). Este ADR não fixa ferramentas. |
| Custo | Ferramentas grátis ou incluídas na subscrição do Claude. |

## Opções consideradas

### Integração no `main`

| Opção | Prós | Contras |
|---|---|---|
| **A. Tudo por PR, com squash merge** ✅ | Um commit por ticket, ligado ao issue. O PR guarda o porquê e o review. | Mesmo uma gralha exige ticket, branch e PR. |
| B. Push direto para o `main` | O mais rápido. | Não há momento para o review nem para o CI. Perde-se a ligação ao ticket. |
| C. PR com merge commit ou rebase | Mantém todos os commits do branch. | O `main` fica com commits intermédios e é mais difícil reverter um ticket inteiro. |

### Review

| Opção | Prós | Contras |
|---|---|---|
| **A. `/code-review` do Claude Code + CodeRabbit, com o merge do dono** ✅ | Dois revisores independentes, ambos grátis (o CodeRabbit é grátis em repositórios públicos). Os comentários ficam no PR. | O review por IA pode deixar passar erros ou dar falsos positivos. |
| B. GitHub Action do Claude em cada PR | Corre sozinho. | Exige uma chave de API paga, guardada como segredo do repositório. |
| C. Só o dono | Zero ferramentas. | Quem escreve não é bom revisor do próprio trabalho, sobretudo com código gerado em volume. |

### Onde fica o grosso dos testes

| Opção | Prós | Contras |
|---|---|---|
| **A. Testes unitários da lógica de negócio no centro, com testes de invariantes e de ficheiros de referência; integração com base de dados real para a separação dos dados; poucos testes ponta a ponta** ✅ | Testa onde está o risco: a lógica que decide se bate certo. A lógica é pura (recebe movimentos e lançamentos, devolve emparelhamentos), por isso os testes são rápidos e não dependem da base de dados. | Exige que a lógica de negócio fique separada da base de dados e da interface. É uma restrição para o ADR-001 e para o modelo de domínio (#9). |
| B. Integração com base de dados real no centro (como no CoopHousing) | Apanha erros de consultas e de permissões. | O risco principal aqui não está na base de dados. Testes mais lentos para a parte que mais muda. |
| C. Ponta a ponta em todos os ecrãs | Cobre toda a interface. | CI lento e testes instáveis, para uma app com poucos ecrãs e poucos utilizadores. |

### Como se sabe que o TXT funciona

| Opção | Prós | Contras |
|---|---|---|
| **A. Ficheiros de referência no CI + importação manual na empresa de demonstração antes de cada versão** ✅ | O CI apanha qualquer mudança no TXT. O Primavera confirma que aceita o ficheiro. | A parte manual depende de alguém a fazer. |
| B. Só ficheiros de referência | Tudo automático. | O resultado esperado pode estar errado, e só o Primavera o diria. |
| C. Correr o Primavera no CI | Tudo automático e real. | Impossível: Windows, licença e instalação no gabinete. |

### Cobertura

| Opção | Prós | Contras |
|---|---|---|
| **A. Mínimo de 80% nas linhas alteradas em cada PR** ✅ | Cada mudança traz testes, sem punir código antigo. | Precisa de comparar com o `main`. |
| B. Meta global | Simples. | Incentiva testes só para subir o número. |
| C. Não medir | Zero esforço. | Módulos inteiros sem testes passam sem ninguém reparar. |

## Decisão

**Tudo entra por PR com squash merge, revisto pelo `/code-review` e pelo CodeRabbit, e só o dono faz o merge. O grosso dos testes está na lógica de negócio, com testes de invariantes e de ficheiros de referência. A separação dos dados testa-se com base de dados real, e os testes ponta a ponta são poucos, mas correm nos três browsers. Antes de cada versão, o TXT é importado na empresa de demonstração do Primavera.**

### 1. Fluxo de um ticket

1. O issue passa a **Em curso** e cria-se o branch `tipo/#nr-descricao` a partir do `main` atualizado.
2. Commits livres no branch, com mensagens descritivas.
3. Abre-se o PR para o `main`: título em Conventional Commits e corpo com "O que muda" e `Closes #nr` (como diz o `CLAUDE.md`). O issue passa a **Em revisão**.
4. O CI tem de ficar verde.
5. Review: corre-se o `/code-review`, que deixa os comentários no PR, e o CodeRabbit comenta sozinho.
6. Cada conversa é corrigida, ou respondida com a justificação, e depois marcada como resolvida.
7. O dono decide e faz o squash merge. É o merge que vale como aprovação, porque o GitHub não deixa o autor aprovar o próprio PR. O branch é apagado e o issue fecha.

### 2. Regras do GitHub

| Decisão | Porquê |
|---|---|
| **Ruleset no `main`: só por PR, sem aprovação obrigatória** | Tudo passa por um PR. A aprovação não pode ser obrigatória: o autor não pode aprovar o próprio PR e não há outro programador. *(Já aplicado.)* |
| **Só squash merge e apagar o branch depois do merge** | Um commit por ticket, sem branches mortos. *(Já aplicado.)* |
| **Sem force push nem eliminação do `main`** | O histórico publicado não se reescreve. *(Já aplicado.)* |
| **Conversas resolvidas obrigatórias** | Nenhum comentário do review fica sem resposta. Um comentário errado também se resolve, mas com a justificação escrita. *(Aplicado neste ticket.)* |
| **Checks do CI obrigatórios, que correm sempre** | Nada entra com o build, o lint ou os testes partidos. Um check obrigatório nunca é filtrado por caminhos no gatilho do workflow: um check que não corre fica à espera e bloqueia o merge. Se for preciso poupar tempo, a filtragem faz-se dentro do workflow, e o check reporta sucesso. *(Hoje só o "Validar título". Os outros entram com o CI.)* |

### 3. Definição de pronto

Um ticket está pronto quando:

1. Cumpre todos os critérios de aceitação do issue.
2. Tem os testes da secção 6, e o CI está verde. Um PR sem código não precisa de testes.
3. A documentação afetada está atualizada (ADR, `CLAUDE.md`, Manual da app no Notion para funcionalidades).
4. Não tem dados reais nem segredos.
5. Passou pelo review e tem todas as conversas resolvidas.
6. O dono fez o merge.

### 4. Como se verifica cada coisa

| O que se quer saber | Verificação automática (CI) | Revisão |
|---|---|---|
| Os números batem certo | Testes das regras de negócio, de invariantes e de ficheiros de referência (secção 5). | O dono confirma que cada regra afetada tem o seu teste. |
| ADRs respeitados | Sempre que possível, uma regra de um ADR passa a ser um teste. | `/code-review` e CodeRabbit contra os ADRs aceites. |
| Bom código | Lint, formatação e verificação de tipos (ferramentas no ADR-001). | `/code-review` e CodeRabbit. |
| Dados separados por empresa e por gabinete | Testes de integração com base de dados real (secção 5, nível 3). | Revisor. |
| Sem dados reais nem segredos | Secret scanning com push protection (já ligado). Dados de teste só do gerador ou de ficheiros verificados (secção 7). | Revisor. |
| O Primavera aceita o TXT | Ficheiros de referência no CI. | Importação manual na empresa de demonstração antes de cada versão (secção 8). |

Uma regra que vive num teste não depende de ninguém se lembrar dela.

### 5. Níveis de teste

1. **Regras de negócio (unitários).** A lógica de negócio fica separada da base de dados e da interface, e testa-se sozinha: emparelhamento, comissões, TPA, períodos, partidas dobradas, valores em cêntimos. **É aqui que fica a maior parte dos testes.** Cada regra de negócio do modelo de domínio (#9) tem um identificador, e os seus testes levam esse identificador no nome. Assim, para cada regra, sabe-se que testes a provam.
2. **Invariantes.** Testes que geram centenas de casos ao acaso e verificam o que tem de ser sempre verdade, por exemplo:
   - em cada lançamento proposto, o débito é igual ao crédito;
   - nenhum movimento está em dois emparelhamentos ativos;
   - os emparelhados mais os pendentes somam o total do extrato;
   - importar o mesmo ficheiro duas vezes não duplica movimentos.

   Quando um destes testes encontra uma falha, o caso que a provocou passa a ser um teste fixo.
3. **Integração com base de dados real.** A separação dos dados (ADR-003 e ADR-004): o utilizador da empresa A, ou do gabinete A, não vê nem altera nada da B. Também se testam importações e transações. A base de dados nunca é simulada.
4. **Ficheiros de referência.** A leitura de cada formato (SAF-T, CSV/Excel de cada banco, MT940, CAMT.053, ficheiro do e-Fatura) e a geração do TXT para o Primavera. Para cada um há um ficheiro de entrada e o resultado esperado, guardados no repositório e comparados exatamente, carácter a carácter no caso do TXT. Mudar um resultado esperado é uma decisão explícita no PR, com o motivo.
5. **Ponta a ponta.** Só 1 a 2 fluxos críticos, por exemplo importar → emparelhar → rever → exportar o TXT, e a lista de documentos em falta. Correm em **Chromium, Firefox e WebKit**, e verificam também a acessibilidade (falham em erros graves).

### 6. O que cada PR tem de trazer

| Mudança | Teste exigido |
|---|---|
| Correção de um bug | Um teste que reproduz o bug e falha antes da correção. |
| Regra de negócio nova ou alterada | Testes com o identificador da regra, incluindo os limites (zero, cêntimos, datas na fronteira do mês) e, quando fizer sentido, um invariante. |
| Formato de ficheiro novo ou alterado (entrada ou saída) | Um par de ficheiros de referência. |
| Acesso a dados novo ou alterado | Integração com três casos: o normal, sem permissão (recusado) e outra empresa (não vê, não altera). |
| Ecrã ou fluxo crítico novo ou alterado | O teste ponta a ponta desse fluxo. |
| Só documentação ou configuração sem código | Não precisa. |

**Como se escrevem:**
- Nas regras de negócio e nas correções de bugs, **o teste escreve-se primeiro** e vê-se falhar antes de escrever o código.
- Nunca se simula a base de dados nem a nossa própria lógica. Só se simula o que é externo e não tem substituto local, e apenas na fronteira.
- Cada teste cria os seus dados e não depende da ordem nem do resultado de outro.
- **Tempo controlado:** a lógica recebe a data atual de um relógio injetável, e os testes avançam-no em vez de esperar. É o que permite testar fechos de mês e datas na fronteira.

### 7. Dados de teste

- **Um único gerador de dados fictícios**, com semente fixa, para que os dados sejam sempre os mesmos e uma falha se possa repetir. Gera empresas, contas, extratos, lançamentos, TPA e comissões plausíveis, em português de Portugal.
- **Sempre dois gabinetes e, em cada um, pelo menos duas empresas**, para que "A não vê nada de B" seja sempre testável.
- **Ficheiros da empresa de demonstração do Primavera** só entram no repositório depois de verificados, um a um, para garantir que não trazem nada do gabinete (por exemplo, o nome ou a licença no cabeçalho do SAF-T). Se trouxerem, usa-se o gerador.
- **Nunca dados reais**, nem cópias de ficheiros de clientes, nem dados "anonimizados à mão".

### 8. Verificações manuais

- **Importação no Primavera, antes de cada versão.** O dono importa o TXT gerado pela versão nova na **empresa de demonstração** do Primavera do gabinete, nunca numa empresa real, e confirma que o Primavera aceita o ficheiro e que os lançamentos ficam como esperado. Fica uma checklist curta no repositório (criada com o primeiro exportador).
- **Testes de aceitação com o gabinete.** Para cada funcionalidade, um caso fictício ou da empresa de demonstração em que um contabilista do gabinete confirma o resultado esperado (o que emparelha, o que fica pendente, que lançamentos se propõem). Esse caso passa a ser um teste automático. Quem decide o resultado certo é quem sabe contabilidade.

### 9. CI, testes instáveis e exceções

- **CI de cada PR em 10 minutos, no máximo**, com checks com nome e obrigatórios (os nomes definem-se com o CI).
- **Sem execução noturna por agora:** os testes cabem todos em cada PR, e o Dependabot já verifica as dependências. Volta-se a pensar nisto se o CI passar dos 10 minutos.
- **Testes instáveis:** um teste que falha ao acaso vai para **quarentena** com um ticket, por no máximo 7 dias, e corrige-se ou apaga-se. Não se carrega em "repetir até passar". **Um teste de regra de negócio ou de separação dos dados nunca vai para quarentena:** corrige-se no momento.
- **Exceções a um check** (uma linha que não dá para testar, um resultado de referência atualizado):
  - ficam no próprio PR, sempre com o motivo escrito;
  - **o Claude nunca cria uma exceção por iniciativa própria:** pergunta primeiro ao dono, com o motivo, e a autorização é o merge;
  - não dá para impor isto tecnicamente, porque o GitHub vê o dono e o Claude como a mesma conta, por isso a proteção está na regra, na visibilidade e na revisão;
  - os testes de separação dos dados não admitem exceção.

### 10. Performance

- **Um limite simples, verificado no CI:** importar e emparelhar um ano de extratos fictícios de uma empresa demora poucos segundos. O valor exato fixa-se com o scaffolding.
- *Rejeitado:* teste de carga. Com um gabinete e cerca de 45 empresas, a escala não o justifica. Reavalia-se antes de a app ser oferecida a outros gabinetes.

### 11. Transição

| Peça | Até existir | Quando passa a valer |
|---|---|---|
| CI (lint, testes, cobertura) | Só o check "Validar título". Os PRs só de docs entram sem mais checks. | No scaffolding do épico E1 ([#16](https://github.com/RafaelFerraz2002/bate-certo/issues/16)). |
| CodeRabbit | O review é só o `/code-review`. | Quando o dono instalar a app do CodeRabbit no repositório (é grátis em repositórios públicos). |
| Testes na definição de pronto | Não se aplicam enquanto não houver código. | No scaffolding (E1). |
| Ferramentas de teste | Por escolher. | ADR-001 ([#10](https://github.com/RafaelFerraz2002/bate-certo/issues/10)). |

## Consequências

**Positivas**
- Os testes estão onde está o risco: na lógica que decide se bate certo.
- Cada regra de negócio tem testes com o seu identificador. Quem faz o merge vê que regra mudou e que testes a provam, sem ler o código todo.
- Os testes de invariantes apanham casos em que ninguém pensaria.
- Os ficheiros de referência tornam visível qualquer mudança, mesmo de um espaço, no TXT para o Primavera.
- O resultado esperado vem do contabilista, não de quem escreve o código.

**Negativas e riscos**

| Risco | Mitigação |
|---|---|
| O resultado esperado de um teste estar errado, e o teste confirmar o erro | Testes de aceitação validados por um contabilista; importação na empresa de demonstração antes de cada versão. |
| A importação manual no Primavera ser esquecida | Checklist no repositório; faz parte de cada versão, não de cada PR. |
| Um ficheiro de referência ser atualizado só para o teste passar | Mudar um resultado esperado exige o motivo no PR, e o revisor confirma-o. |
| 80% das linhas alteradas atingido com testes fracos | O revisor verifica os casos; os testes das regras e da separação dos dados existem independentemente da percentagem. |
| Testes ponta a ponta instáveis nos três browsers | Poucos testes, bem escolhidos; esperar por estados da página e não por tempos fixos; quarentena com prazo. |
| O review por IA falhar | O merge do dono continua obrigatório. |

**Decisões que ficam para outros ADRs e tickets**
- ADR-001 ([#10](https://github.com/RafaelFerraz2002/bate-certo/issues/10)): as ferramentas de teste, de lint e de cobertura. Tem de permitir separar a lógica de negócio da base de dados e da interface.
- ADR-003 ([#12](https://github.com/RafaelFerraz2002/bate-certo/issues/12)) e ADR-004 ([#13](https://github.com/RafaelFerraz2002/bate-certo/issues/13)): as regras de separação dos dados que os testes de integração verificam.
- Modelo de domínio ([#9](https://github.com/RafaelFerraz2002/bate-certo/issues/9)): as regras de negócio e os seus identificadores.
- Scaffolding (E1, [#16](https://github.com/RafaelFerraz2002/bate-certo/issues/16)): o CI, os checks obrigatórios, o gerador de dados fictícios, o relógio injetável e o limite de performance.
