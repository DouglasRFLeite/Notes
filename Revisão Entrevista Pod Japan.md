
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
Não é necessário usar SAGA quando a transação é única

**Pegadinha ou trade-off:**


---

## 3) Circuit Breaker

**O que é (1-2 frases):**


**Que problema resolve:**


**Como funciona / como se implementa:**


**Quando usar / quando não usar:**


**Pegadinha ou trade-off:**


---

## 4) Kafka vs outros MQ

**O que é (1-2 frases):**


**Que problema resolve:**


**Como funciona / como se implementa:**


**Quando usar / quando não usar:**


**Pegadinha ou trade-off:**


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