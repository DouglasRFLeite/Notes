
# Revisão pré-entrevista — Active Recall

> Regras: responder sem consultar a explicação anterior, sem Google, sem IDE.
> Rascunho feio é o objetivo. Se travar, escreve "não sei" e segue.
> Tempo sugerido: 8-10 min por tópico.

---

## 1) Idempotência

**O que é (1-2 frases):**
Idempotência é a característica de um software ou aplicação que recebendo N chamados/requests iguais não os executa múltiplas vezes, mas gera apenas um resultado, o mesmo de ter recebido um único request.

**Que problema resolve:**
Pensando em verbos HTTP, a idempotência se mostra mais necessária para lidar com POST requests. 
Se eu envio um POST para criar o usuário Douglas múltiplas vezes (considerando que "Douglas" não seja uma chave única) a aplicação naturalmente cria múltiplos usuários Douglas. Para criação de usuários pode não ser um problema, mas será pra criação de pedidos, transações bancárias, registro de movimentações, etc. 
A implementação da idempotência é necessária para resolver esse tipo de problema não apenas para quando o usuário clica em "Criar" múltiplas vezes, mas ainda mais em especial para quando há retry ou o cliente envia, por outro motivo, a mesma requisição múltiplas vezes.

**Como funciona / como se implementa:**
Existem algumas formas de se implementar que dependem do caso de uso. No exemplo do usuário, bem como outros, faz sentido que um uniqueID (pode ser chamado de deduplicationID) seja enviado do cliente e que o servidor meramente rejeite criar uma segunda entrada no banco com esse ID. 
Em outros casos o uniqueID pode ser um UUID criado com o único propósito de servir para deduplicação. 

**Quando usar / quando não usar:**
É necessário implementar o cuidado com a idempotência toda vez que múltiplas repetições de um mesmo request podem causar algum dano à aplicação ou ao mundo real (e.g., transações de banco, criação de pedidos). Em casos em que isso não faça diferença, pode não ser necessário.

**Pegadinha ou trade-off:**
?

---

## 2) SAGA

**O que é (1-2 frases):**
SAGA é uma estratégia para fazer algo semelhante a um rollback para quando um processo depende de mais de uma transação. Depende da existência de um "transação inversa" que desfaça cada etapa do processo.

**Que problema resolve:**
Usemos como exemplo uma compra num e-commerce: Criação do Pedido -> Pagamento -> Reserva do Item -> Envio
Uma falha na etapa de envio não é resolvida por um simples rollback na transação de Envio. Pelo contrário, é necessário "desfazer" as últimas transações também. O rollback apenas desfaria a transação que falhou.

**Como funciona / como se implementa:**
Para resolver esse problema, a SAGA determina a necessidade de "contramedidas":
Criação de Pedido - Remoção do Pedido
Pagamento - Extorno 
Reserva do Item - Liberação do Item
Envio - Cancelamento/Devolução
Caso haja uma falha na etapa de Envio, as outras transações serão "desfeitas" pela respectiva contramedida.

**Quando usar / quando não usar:**
A estratégia SAGA deve ser usada para processos 2PC-Two Partition Commit, ou seja, processos que a conclusão é dividida em duas ou mais partes que teriam que ser desfeitas individualmente.
Não é necessário usar SAGA quando a transação é única, resolvida inteiramente em uma única transação que possa ser desfeita de forma mais simples com rollback.

**Pegadinha ou trade-off:**
As contramedidas não são um rollback, então via de regra "sobram" efeitos colaterais da transação inicial como um email enviado ou um registro na conta bancária do usuário.

---

## 3) Circuit Breaker

**O que é (1-2 frases):**
Circuit Breaker é um mecanismo utilizado como wrapper de requests feitos pela aplicação. Funciona semelhante a um circuito elétrico, interrompendo temporariamente tentativas de conexão que estejam falhando e atrapalhando a performance do resto do sistema.

**Que problema resolve:**
Em geral incluimos estratégias de retry e time-out em requests para que não se tornem bloqueantes. Contudo, mesmo com essas estratégias, se as requests são muito comuns, muitas threads podem ficar bloqueadas esperando o time-out quando o serviço externo está constantemente falhando. Isso, pareado com os retrys, pode se tornar um blocker na aplicação.

**Como funciona / como se implementa:**
O Circuit Breaker funciona de forma semelhante a circuito elétrico. Quando percebe que a chamada de API está falhando com frequência ele interrompe o funcionamento daquela chamada, retornando falha automática sem bloquear a thread (estádo OPEN). Após algum tempo, o Circuit Breaker restaura parcialmente o funcionamento das chamadas, permitindo a execução de uma parcela delas e analisando o resultado (estado SEMI-CLOSED). Quando percebe que o sistema externo voltou a funcionar, o Circuit Breaker restaura o funcionamento da chamada (estado CLOSED).

**Quando usar / quando não usar:**
O Circuit Breaker faz sentido quando uma chamada de API a um sistema externo é feita com frequência pela nossa aplicação, algo que poderia causar um grande problema de performance caso o serviço externo caia.
Talvez não haja necessidade de usar uma estratégia como essa para chamadas de API pouco frequentes, ou nas quais o time-out já é naturalmente muito curto.

**Pegadinha ou trade-off:**
É necessário saber falhar a chamada. O resto da aplicação, quem "trigga" a chamada de API deve estar preparada para a falha criada pelo Circuit Breaker.

---

## 4) Kafka vs outros MQ

**O que é (1-2 frases):**
O Kafka difere dos MQs tradicionais pois não é exatamente um MQ broker, e sim um sistema de logging. Em geral, as transações no Kafka são armazenadas como logs, não necessariamente transportadas como pacotes.

**Que problema resolve:**
Todo sistema de mensageria vem com o propósito de atender a necessidade de comunicação assíncrona, mais resiliente e consistente (em troca, em geral, de um gasto maior de recursos).

**Como funciona / como se implementa:**
O Kafka adiciona todas as mensagens em uma espécie de arquivo de logs, de onde as outras aplicações podem ler e onde elas identificam a mudança por um offset. Portanto, nem sempre as mensagens são removidas, apresentando uma maior facilidade de entrega para múltiplos ouvintes e também de "replay" do stream de mensagens.
As outras MQs em geral funcionam mais como uma fila de fato, em que a mensagem entra e é retirada pelo ouvinte quando a recebe.

**Quando usar / quando não usar:**
Kafka faz mais sentido quando as mensagens serão lidas por múltiplos ouvintes ou quando pode ser necessário revisitar o stream de mensagens anteriores.
Para casos em que apenas um ouvinte lerá cada tópico e retirará obrigatoriamente a mensagem da fila, bem como casos em que não será necessário revisitar as mensagens antigas, o Kafka notoriamente trás uma complexidade maior, fazendo mais sentido utilizar MQs tradicionais. 

**Pegadinha ou trade-off:**
Não é 100% correto chamar o Kafka de uma MessageQueue, porque na verdade ele funciona como um logging system.

---

## 5) Arquitetura N-Tier

**O que é (1-2 frases):**


**Que problema resolve:**


**Como funciona / como se implementa:**


**Quando usar / quando não usar:**


**Pegadinha ou trade-off:**


---

## Auto-avaliação rápida (preencher DEPOIS de responder tudo)

Pra cada tópico, marca: 🟢 firme | 🟡 sei o básico | 🔴 travei

- [ ] Idempotência:
- [ ] SAGA:
- [ ] Circuit Breaker:
- [ ] Kafka vs MQ:
- [ ] N-Tier:

**Onde travei mais:**


**O que quero aprofundar com o Claude:**