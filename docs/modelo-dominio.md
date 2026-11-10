# Modelo de domínio das conciliações

- **Estado:** Rascunho. Parte do questionário ao gabinete (#4) e dos spikes (#5, #6). Valida-se na sessão no gabinete (#7) e passa a **Aceite** com os requisitos validados (#8).
- **Ticket:** #9 (épico #1)
- **Requisitos:** CB-01 a CB-06, CT-01 a CT-03, EF-01 a EF-03, GL-01 a GL-09

Marcas: ✅ confirmado pelo gabinete ou decidido com o utilizador · ⚠️ deduzido, por validar · ❓ em aberto.

Há um espelho desta página no Notion, na página principal do Bate Certo ("Modelo de domínio"). **A versão que vale é esta.** Uma mudança entra por PR e depois copia-se para o Notion.

## 1. Para que serve

- Dá nomes comuns às coisas: o que é um movimento, um emparelhamento, um pendente.
- Dá um identificador a cada regra de negócio (R1, R2…). Os testes levam esse identificador no nome (ADR-005, secção 5), e assim sabe-se que testes provam cada regra.
- É a base do ADR-001 (a lógica de negócio fica separada da base de dados e da interface), do ADR-003 (separação dos dados) e do ADR-004 (que dados se guardam).

Não é o esquema da base de dados. Esse faz-se na fundação (E1, #16), a partir daqui.

## 2. O fluxo

```mermaid
flowchart LR
  SAFT[SAF-T de contabilidade] --> IMP[Importação]
  EXT[Extrato bancário] --> IMP
  EF[Ficheiro do e-Fatura] -.-> IMP
  IMP --> NORM[Movimentos e lançamentos normalizados]
  NORM --> EMP[Emparelhamento<br/>o sistema propõe]
  EMP --> REV[Revisão<br/>uma pessoa aceita]
  REV --> TXT[TXT de lançamentos propostos]
  REV --> FALTA[Lista de documentos em falta]
  TXT --> PRI[Primavera<br/>o gabinete importa]
  PRI -.->|SAF-T seguinte| SAFT
  FALTA --> CLI[Cliente do gabinete<br/>por email]
```

1. **Entradas:** o SAF-T de contabilidade, exportado do Primavera, com os lançamentos e o plano de contas ✅. Os extratos dos bancos, por ficheiro (CSV/Excel configurável por banco e MT940) ✅. O ficheiro exportado do e-Fatura, carregado à mão ⚠️ (proposta, por debater no #15).
2. **Normalizar:** cada fonte é convertida no mesmo formato interno: valores em cêntimos, datas e conta.
3. **Emparelhar:** o sistema propõe pares entre o banco e a contabilidade, por níveis (R6).
4. **Rever:** uma pessoa aceita, corrige ou rejeita (R2). O que fica sem par passa a pendente, com uma causa (R11). O que falta lançar (comissões, por exemplo) passa a proposta de lançamento.
5. **Saídas:** um TXT com os lançamentos propostos e aceites, para importar no Primavera (GL-03) ✅, e a lista de documentos em falta por cliente (GL-04) ⚠️.
6. **Regresso:** o gabinete importa o TXT no Primavera. No SAF-T seguinte, esses lançamentos voltam a entrar e ligam-se às propostas que os originaram (R14).

O Bate Certo **nunca escreve no Primavera** (R1): é o gabinete que importa o TXT.

## 3. Entidades

```mermaid
erDiagram
  GABINETE ||--o{ UTILIZADOR : tem
  GABINETE ||--o{ EMPRESA : acompanha
  EMPRESA ||--o{ CONTA_BANCARIA : tem
  EMPRESA ||--o{ REGRA_CLASSIFICACAO : define
  EMPRESA ||--o{ IMPORTACAO : recebe
  EMPRESA ||--o{ EXPORTACAO : recebe
  CONTA_BANCARIA ||--o{ PERIODO : "trabalha-se por mês"
  CONTA_BANCARIA ||--o{ MOVIMENTO : tem
  IMPORTACAO ||--o{ MOVIMENTO : cria
  IMPORTACAO ||--o{ LANCAMENTO : cria
  LANCAMENTO ||--|{ LINHA_LANCAMENTO : tem
  PERIODO ||--o{ EMPARELHAMENTO : "aceite em"
  EMPARELHAMENTO }o--o{ MOVIMENTO : liga
  EMPARELHAMENTO }o--o{ LINHA_LANCAMENTO : liga
  EMPARELHAMENTO |o--o| PROPOSTA_LANCAMENTO : "a diferença gera"
  MOVIMENTO ||--o| PENDENTE : "pode ficar"
  LINHA_LANCAMENTO ||--o| PENDENTE : "pode ficar"
  PROPOSTA_LANCAMENTO |o--o{ PENDENTE : resolve
  PENDENTE ||--o| DOCUMENTO_EM_FALTA : gera
  PROPOSTA_LANCAMENTO }o--o{ EXPORTACAO : "sai em"
  PROPOSTA_LANCAMENTO |o--o| LANCAMENTO : "confirmada por"
```

- Um movimento pode ter vários emparelhamentos ao longo do tempo (os desfeitos ficam no histórico), mas só um **aceite** (R7).
- Um emparelhamento liga movimentos a linhas de lançamento. Pode ter um dos lados vazio quando se anula a si próprio: um movimento e o seu estorno, que somam zero (R6).
- **Todas as entidades pertencem a uma empresa**, direta ou indiretamente, e tudo o que se liga entre si pertence à mesma empresa (R3, R15). O diagrama não mostra todas essas ligações, para se ler melhor.

| Entidade | O que é | O que guarda (principal) | Notas |
|---|---|---|---|
| **Gabinete** | O cliente do Bate Certo: um gabinete de contabilidade. | Nome, utilizadores, empresas. | É a primeira fronteira da separação dos dados (R3). |
| **Utilizador** | Uma pessoa do gabinete: contabilista certificado ou técnico. | Nome, papel, gabinete. | Aceita propostas e fecha períodos. Fica registado quem fez o quê (GL-06). Previstos 3 a 5 em simultâneo (GL-05) ⚠️. |
| **Empresa** | Um cliente do gabinete, com a sua contabilidade. | NIF, nome, plano de contas (do SAF-T). | Cerca de 45 por gabinete (GL-01) ✅. É a segunda fronteira da separação (R3). |
| **Conta bancária** | Uma conta de uma empresa num banco. | Empresa, banco, IBAN, conta contabilística correspondente (subconta da 12). | 2 a 3 por empresa (CB-02) ⚠️. |
| **Período** | Um mês de trabalho de uma conta bancária. | Conta, mês, estado, quem fechou e quando, e a fotografia do fecho: saldo do banco, saldo da contabilidade e pendentes nesse momento. | É a unidade de trabalho (R10). A fotografia não muda depois do fecho. |
| **Importação** | O registo de um ficheiro carregado. | Tipo (SAF-T, extrato, e-Fatura), empresa, conta (nos extratos), quem, quando, impressão digital do ficheiro, intervalo de datas, saldo inicial e final (nos extratos). | Impede importar o mesmo ficheiro duas vezes e permite confirmar a cadeia de saldos (R4). |
| **Movimento** | Uma linha do extrato bancário. | Conta, data, data-valor, valor em cêntimos com sinal, descritivo, referência. | Sinal: positivo é dinheiro que entra, e corresponde a um débito na conta 12. A empresa é a da conta, e a importação tem de ser da mesma empresa e conta. |
| **Lançamento** | Um registo da contabilidade, lido do SAF-T. | Empresa, diário, número do documento, data, linhas. | O Bate Certo só o lê. |
| **Linha de lançamento** | Uma linha de um lançamento. | Conta, lado (débito ou crédito), valor em cêntimos, descrição. | Na bancária, interessam as linhas da conta 12 da conta bancária. |
| **Emparelhamento** | A ligação entre movimentos e linhas de lançamento que representam o mesmo dinheiro. | Empresa, conta, período onde foi aceite, nível (R6), estado, diferença, proposta que explica a diferença, quem aceitou e quando. | O coração do sistema. |
| **Pendente** | Um movimento ou uma linha sem par no fim da revisão. | Empresa, origem, causa (R11), estado. | Pode passar para o mês seguinte. |
| **Regra de classificação** | Uma regra de uma empresa que diz como tratar um tipo de movimento. | Empresa, padrão do descritivo, conta e diário a usar. | Por exemplo, as comissões de um banco (R8). |
| **Proposta de lançamento** | Um lançamento que o Bate Certo sugere: comissões, encargos, prestações, ou a comissão descontada num depósito de TPA. | Empresa, linhas a débito e a crédito, regra que a originou, pendentes ou emparelhamento que resolve, estado, quem aceitou, lançamento do Primavera que a confirmou. | Débito = crédito (R9). Só sai no TXT depois de aceite (R2). Uma proposta pode juntar vários pendentes, por exemplo, todas as comissões do mês ⚠️. |
| **Exportação** | Um TXT gerado para importar no Primavera. | Empresa, propostas incluídas, motivo (primeira exportação ou reexportação), quem gerou, quando. | Um TXT é sempre de uma só empresa, porque o Primavera importa para uma empresa de cada vez. Layout: o da v9 é público; o da v10 ❓ (GL-03). |
| **Documento em falta** | Um item da lista a enviar ao cliente. | Empresa, período, movimento ou fatura de origem, descrição, estado. | Hoje a lista é feita em Word e enviada por email (GL-04) ⚠️. |

A conciliação e-Fatura (EF-01) e a de terceiros (CT-01 a CT-03) ainda não estão modeladas. ⚠️ A hipótese é que usem o mesmo emparelhamento: no e-Fatura, um documento comunicado à AT ↔ um lançamento de compra; nos terceiros, uma fatura ↔ um ou mais pagamentos. Modela-se quando se decidir a ordem dos épicos (#15).

## 4. Regras de negócio

Cada regra tem um identificador que não muda. Se uma regra deixar de valer, marca-se como **retirada** e o número não se reutiliza.

As regras R1 a R12 foram propostas na conversa de 2026-11-04, a partir do questionário e dos spikes. As R13 a R15 nasceram na revisão deste documento. Só o R1 foi decidido de forma explícita ✅. **O merge deste rascunho aprova a estrutura e as regras como hipótese de trabalho, não como decisão final.** Os pontos marcados ⚠️ ou ❓ validam-se na sessão no gabinete (#7), e o documento só passa a Aceite no #8.

### R1. Nunca escreve no Primavera ✅

O Bate Certo não se liga à base de dados nem à API do Primavera para escrever. Entrega ficheiros (o TXT e a lista de faltas), e é o gabinete que os importa ou envia.

- **Porquê:** um erro de escrita no ERP estraga a contabilidade de um cliente. O gabinete já importa ficheiros hoje e prefere esse caminho (GL-03). Decidido a 2026-11-04.
- **Como se prova:** o sistema não tem credenciais nem ligação ao Primavera. Verifica-se na revisão de cada PR, e não por um teste.
- **Requisitos:** CB-01, GL-02, GL-03.

### R2. O sistema propõe, uma pessoa aceita

Nenhum emparelhamento e nenhuma proposta de lançamento passa a "aceite" sem a ação de um utilizador. Fica registado quem aceitou e quando.

- **Porquê:** a responsabilidade pela contabilidade é do contabilista certificado. Um erro automático que ninguém viu chega ao Primavera.
- ⚠️ **Por validar:** os emparelhamentos de nível exato podem ser aceites em lote, com um só clique.
- **Como se prova:** unitário (sem um utilizador, o estado não muda) e ponta a ponta no fluxo de revisão.
- **Requisitos:** GL-06.

### R3. Os dados são separados por empresa e por gabinete

Cada dado pertence a uma empresa, e cada empresa a um gabinete. Um utilizador só vê e altera as empresas do seu gabinete. Uma operação numa empresa nunca lê nem altera os dados de outra.

- **Porquê:** sigilo profissional dos contabilistas (OCC) e RGPD. Uma fuga entre empresas ou entre gabinetes é o erro mais grave possível.
- **Como se prova:** integração com a base de dados real, sempre com três casos: o normal, sem permissão (recusado) e outra empresa (não vê, não altera) (ADR-005, secção 6). A forma técnica decide-se no ADR-003 (#12). A coerência dentro de uma operação é o R15.
- **Requisitos:** GL-01, GL-09.

### R4. As importações ficam registadas e nunca se duplicam

Cada ficheiro importado fica registado, com a sua impressão digital. O mesmo ficheiro carregado duas vezes é recusado.

Quando um extrato se sobrepõe a outro já importado (por exemplo, de 15/01 a 15/02, depois de um de 01/01 a 31/01), só se compara o **intervalo de datas em comum**. Nesse intervalo, os movimentos iguais contam-se, e fica o maior número visto num só ficheiro. Assim, duas transferências de 10 € no mesmo dia, com o mesmo descritivo, continuam a ser duas. Os saldos inicial e final de cada extrato confirmam o resultado: o saldo anterior mais os movimentos tem de dar o saldo final.

- **Porquê:** os extratos de meses seguidos sobrepõem-se muitas vezes. Um movimento duplicado gera uma diferença falsa, e um movimento apagado esconde dinheiro.
- ⚠️ **Por validar:** o que torna dois movimentos "iguais". A proposta é a conta, a data, o valor, o descritivo e a referência, quando o banco a dá. E se todos os formatos de extrato trazem os saldos.
- **Como se prova:** unitário (sobreposição parcial, movimentos repetidos legítimos) e invariante (importar o mesmo conteúdo duas vezes dá sempre o mesmo resultado que importar uma; a cadeia de saldos bate).
- **Requisitos:** CB-02.

### R5. Os valores guardam-se em cêntimos inteiros

Todos os valores em euros guardam-se e calculam-se como números inteiros de cêntimos. Nunca se usam números com vírgula flutuante. Num movimento, o sinal positivo é dinheiro que entra na conta (um débito na conta 12) e o negativo é dinheiro que sai (um crédito).

- **Porquê:** com vírgula flutuante, 0,10 + 0,20 não dá exatamente 0,30. Numa conciliação, um cêntimo de diferença é uma diferença.
- **Como se prova:** invariante (somar e separar valores nunca perde nem ganha cêntimos) e ficheiros de referência na leitura de cada formato.
- **Requisitos:** CB-05 (diferenças de cêntimos).

### R6. O emparelhamento faz-se por níveis

O sistema tenta, por esta ordem:

1. **Exato:** o mesmo valor, a mesma data e um só candidato possível. ⚠️ Pode fazer sentido aceitar uma diferença de um ou dois dias (data-valor).
2. **Provável:** o mesmo valor e a data dentro de uma janela de poucos dias, ou o descritivo ou a referência a ajudar. ⚠️ O tamanho da janela valida-se com o gabinete.
3. **Vários para um:** um movimento corresponde a várias linhas, ou o contrário. Por exemplo, o depósito de um TPA que junta vendas de vários dias, ou um pagamento que liquida várias faturas. Se houver uma diferença (a comissão que o banco já descontou), o emparelhamento só fica completo com uma proposta de lançamento para essa diferença.
4. **Manual:** a pessoa escolhe o par.

Casos especiais:

- **Valores repetidos** (por exemplo, salários iguais): o sistema não escolhe sozinho um par. ⚠️ Quando há o mesmo número de movimentos e de linhas com o mesmo valor no período, propõe-nos como um grupo, para aceitar de uma vez. Se não houver, vão para a escolha manual, com os candidatos à vista.
- **Anulações:** um movimento e o seu estorno (+100 € e −100 €) emparelham-se entre si, sem linha de lançamento, porque somam zero.

Em qualquer emparelhamento aceite: a soma dos movimentos = a soma das linhas + a diferença, e a diferença tem uma proposta de lançamento associada.

- **Porquê:** o gabinete emparelha sobretudo pelo valor e pela data, e concilia à mão os valores repetidos (CB-05). Os TPA são a maior dor (CB-03), e a comissão descontada tem de chegar à contabilidade, senão o saldo da conta 12 nunca bate com o extrato.
- **Como se prova:** unitário por nível, com os limites (fronteira da janela, cêntimos, fim do mês). Invariante: a igualdade acima.
- **Requisitos:** CB-03, CB-04, CB-05.

### R7. Cada movimento tem no máximo um emparelhamento aceite

Um movimento, ou uma linha de lançamento, nunca está em dois emparelhamentos aceites ao mesmo tempo. Pode aparecer em várias sugestões (propostas pelo sistema), mas, quando uma é aceite, as outras caem.

Desfazer um emparelhamento ("despicar") é uma ação explícita e fica no histórico. Para tirar só um movimento de um grupo (por exemplo, um dia errado num depósito de TPA), desfaz-se o grupo e o sistema cria logo um grupo novo com o que resta, pronto a aceitar com um clique (R2). Assim, nenhum grupo aceite fica a violar a igualdade do R6.

- **Porquê:** hoje, um erro num documento de TPA obriga a despicar o documento todo e a refazê-lo à mão (CB-03).
- **Como se prova:** invariante (depois de qualquer sequência de emparelhar e desfazer, nenhum movimento tem dois emparelhamentos aceites) e unitário no desfazer parcial.
- **Requisitos:** CB-03.

### R8. As regras de classificação são de cada empresa

O gabinete define, para cada empresa, como tratar tipos de movimento: por exemplo, "o descritivo contém COMISSAO → conta de gastos X, diário Y". Uma regra só se aplica à empresa onde foi criada.

- **Porquê:** cada empresa tem o seu plano de contas e os seus bancos. Hoje as comissões, os encargos e as prestações lançam-se um a um (CB-04).
- ❓ **Em aberto:** se faz falta ter também regras do gabinete, que todas as empresas herdam.
- **Como se prova:** unitário (aplicar regras) e integração (uma regra de uma empresa não se aplica a outra).
- **Requisitos:** CB-04.

### R9. Em cada lançamento, débito = crédito

Nenhuma proposta de lançamento é aceite nem exportada se o total a débito for diferente do total a crédito.

- **Porquê:** é a regra base das partidas dobradas. Um lançamento desequilibrado é recusado pelo Primavera ou, pior, entra errado.
- **Como se prova:** invariante (toda a proposta gerada balanceia) e ficheiros de referência do TXT, comparados carácter a carácter.
- **Requisitos:** GL-03, CB-04.

### R10. Trabalha-se por empresa, conta e mês, com fecho

A conciliação faz-se por período: uma conta bancária de uma empresa, num mês. Um movimento pertence ao mês da sua data ⚠️ (ou da data-valor: por validar). Um emparelhamento pertence ao período aberto onde é aceite, mesmo que ligue coisas de meses diferentes: um cheque lançado em janeiro e descontado em fevereiro emparelha-se em fevereiro, e janeiro não muda.

Ao fechar um período, guarda-se uma fotografia: o saldo do banco, o saldo da contabilidade e os pendentes desse momento. A fotografia não muda depois. Num período fechado não se aceitam emparelhamentos nem propostas. Se um ficheiro importado trouxer um movimento novo com data de um período fechado, o movimento entra, mas fica um alerta no período e só se trata depois de um utilizador o reabrir. A reabertura fica registada.

- **Porquê:** é assim que o gabinete trabalha (mês a mês, no fecho). Um período já entregue não deve mudar sem se saber.
- ❓ **Em aberto:** quem pode reabrir um período (só o contabilista certificado?). E se o mês serve para todas as empresas: há empresas conciliadas só no trimestre ou só no fim do ano ("depende da empresa", GL-01).
- **Como se prova:** unitário com relógio controlado (fronteira do mês) e integração (num período fechado, as alterações são recusadas e um movimento novo gera alerta).
- **Requisitos:** CB-01, GL-01.

### R11. Cada pendente tem uma causa

No fim da revisão, cada movimento ou linha sem par fica como pendente, com uma causa: falta o documento, falta lançar, diferença de datas (por exemplo, um cheque ainda não descontado), erro, ou outra. Os pendentes por falta de documento alimentam a lista de documentos em falta. Os de "falta lançar" resolvem-se com uma proposta de lançamento, que pode juntar vários pendentes. Os pendentes que passam de mês, ou de ano, continuam na lista até se resolverem.

- **Porquê:** a lista de faltas é trabalho comum às três conciliações (GL-04). As faltas que passam de ano ficam numa lista própria (CB-06).
- ⚠️ **Por validar:** a lista de causas. Ver como o gabinete as chama.
- **Como se prova:** unitário (um pendente sem causa não deixa fechar o período) e ficheiro de referência da lista de faltas.
- **Requisitos:** CB-06, GL-04, EF-01.

### R12. Não se guardam senhas da AT nem dados reais

O Bate Certo nunca guarda credenciais do Portal das Finanças nem do homebanking dos clientes. O código, os testes e os dados de exemplo nunca têm dados reais: são fictícios (gerador de dados do ADR-005) ou da empresa de demonstração do Primavera.

- **Porquê:** sigilo profissional e RGPD. Uma senha guardada é um risco que o gabinete não precisa de correr, porque os ficheiros exportados chegam.
- **Como se prova:** revisão de cada PR, secret scanning e a origem dos dados de teste (ADR-005, secção 7).
- **Requisitos:** EF-03.

### R13. Cada proposta exporta-se uma só vez, salvo pedido explícito

Uma proposta de lançamento que já saiu num TXT não volta a sair noutro, a não ser que um utilizador peça para a reexportar, com o motivo (por exemplo, a importação no Primavera falhou). Cada exportação fica registada.

Se for preciso anular uma proposta que já foi importada no Primavera (por exemplo, depois de reabrir o período), a proposta passa a **anulada** e o sistema propõe um estorno, que segue o caminho normal: aceitar, exportar, importar.

- **Porquê:** importar o mesmo TXT duas vezes no Primavera duplica os lançamentos. É o mesmo princípio do R4, mas na saída. E, como o Bate Certo não escreve no Primavera (R1), a única forma de corrigir um lançamento já importado é outro lançamento.
- **Como se prova:** unitário e invariante (gerar duas exportações seguidas sem pedido de reexportação nunca repete propostas).
- **Requisitos:** GL-03.

### R14. O SAF-T seguinte confirma o que foi exportado ⚠️

Quando chega um SAF-T novo, as linhas que correspondem a propostas exportadas ligam-se a essas propostas, que passam a **confirmadas no Primavera**. Não aparecem como pendentes novos, nem como lançamentos a propor outra vez.

Se o SAF-T novo mudar ou apagar um lançamento que está num emparelhamento aceite, o emparelhamento passa a precisar de revisão, com um alerta. Nunca se desfaz em silêncio.

- **Porquê:** sem isto, cada comissão exportada voltava como uma linha sem par, e o gabinete via o mesmo trabalho duas vezes ou lançava em dobro.
- ⚠️ **Por validar:** como reconhecer no SAF-T um lançamento que veio do TXT (número do documento, diário, descrição?). Depende do layout do TXT na v10.
- **Como se prova:** unitário e ficheiros de referência (um SAF-T de exemplo com as propostas importadas).
- **Requisitos:** GL-03.

### R15. Tudo o que se liga pertence à mesma empresa

Um emparelhamento só liga movimentos e linhas da mesma empresa e da mesma conta. Uma proposta só resolve pendentes da sua empresa. Um TXT só leva propostas de uma empresa. A importação de um extrato tem de ser da mesma empresa que a conta bancária.

- **Porquê:** é a separação dos dados (R3) dentro de cada operação, e o Primavera importa um TXT para uma empresa de cada vez.
- **Como se prova:** unitário (ligar elementos de empresas diferentes é recusado), logo na lógica de negócio, além da integração do R3.
- **Requisitos:** GL-01, GL-03.

## 5. Estados

| Entidade | Estados | Quem muda |
|---|---|---|
| Emparelhamento | sugerido → aceite → desfeito · sugerido → rejeitado · aceite → por rever (R14) | O sistema sugere; uma pessoa aceita, rejeita ou desfaz (R2, R7). |
| Proposta de lançamento | proposta → aceite → exportada → confirmada no Primavera · proposta → rejeitada · exportada ou confirmada → anulada | O sistema propõe; uma pessoa aceita ou anula; a exportação e o SAF-T seguinte mudam o resto (R2, R13, R14). |
| Período | aberto → fechado → reaberto (volta a aberto) | Uma pessoa (R10). |
| Pendente | aberto → resolvido | Resolve-se com um emparelhamento, uma proposta aceite ou uma decisão da pessoa (R11). |
| Documento em falta | em falta → pedido → recebido · em falta → pedido → sem documento | Uma pessoa. "Sem documento" corresponde às despesas não documentadas (CB-06). |

## 6. Ligação aos requisitos

| Requisito | Entidades | Regras |
|---|---|---|
| CB-01 Complementar a conciliação do Primavera | Período | R1, R10 |
| CB-02 Ler extratos | Importação, Movimento | R4 |
| CB-03 Tratar movimentos de TPA | Emparelhamento, Proposta de lançamento | R6 (nível 3), R7 |
| CB-04 Propor lançamentos de comissões e encargos | Regra de classificação, Proposta de lançamento | R6, R8, R9 |
| CB-05 Emparelhar por valor e data | Emparelhamento | R5, R6 |
| CB-06 Movimentos sem documento | Pendente, Documento em falta | R11 |
| CT-01 a CT-03 Terceiros | ⚠️ ainda não modelado | — |
| EF-01 Conferir o e-Fatura | ⚠️ ainda não modelado; Documento em falta | R11 |
| EF-02 Talões pequenos | ⚠️ ainda não modelado | — |
| EF-03 Sem credenciais da AT | — | R12 |
| GL-01, GL-09 Dimensão e onde ficam os dados | Gabinete, Empresa | R3, R10, R15 |
| GL-02 Primavera no servidor do gabinete | — | R1 |
| GL-03 TXT de lançamentos | Proposta de lançamento, Exportação | R1, R9, R13, R14, R15 |
| GL-04 Lista de documentos em falta | Pendente, Documento em falta | R11 |
| GL-05 Utilizadores em simultâneo | Utilizador | — |
| GL-06 Quem resolveu cada diferença | Utilizador | R2 |
| GL-07 Dashboard, GL-08 Acesso dos clientes | Fora deste rascunho | — |

## 7. Por validar na sessão no gabinete (#7)

1. Como se lançam hoje os TPA e porque é que o valor não bate com o extrato (comissão descontada? vendas de vários dias juntas?) (R6, R7).
2. A janela de dias para um emparelhamento exato e provável, e se o mês se conta pela data ou pela data-valor (R6, R10).
3. As causas que usam para um movimento sem par (R11).
4. Se as regras de classificação são de cada empresa ou também do gabinete (R8).
5. Quem pode fechar e reabrir um período, e se há empresas que não se conciliam todos os meses (R10).
6. Se aceitar em lote os emparelhamentos exatos e os grupos de valores repetidos é aceitável (R2, R6).
7. Se as comissões do mês se lançam juntas num só lançamento ou uma a uma (R11).
8. O layout do TXT na v10, e como se reconhece no Primavera um lançamento importado de um TXT (R9, R13, R14).
9. Se os extratos que recebem trazem o saldo inicial e final (R4).

## 8. Fora deste documento

- O esquema da base de dados: na fundação (E1, #16).
- A stack e os nomes no código: ADR-001 (#10).
- A forma técnica da separação dos dados: ADR-003 (#12).
- Que dados se guardam, por quanto tempo e quem os vê: ADR-004 (#13).
