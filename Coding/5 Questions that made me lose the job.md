
## Idempotency

> *What would happen if your application got multiple similar requests?*
> The question was framed for an application I had built/worked on as I talked about it.

Idempotency is the property of an application - or, rather, an API Endpoint - that guarantees that if the application receives N equal requests the result will be the same as if it had received a single one.

Example:

PUT {userName: "Douglas"} -> If the user doesn't exist, create it. If it does, update it's name to "Douglas". That means that if I repeat this call once or 1000 times, the result will be one user in the database with the name "Douglas"

POST {userName: "Douglas"} -> Create an user with the name "Douglas". If I run this once I'll have 1 Douglas. If I run it 1000 times I'll have a thousand.

The first one is idempotent, the second is not. 

Having a thousand Douglas in your application is kind of scary, but not nearly as much as delivering two fridges to someone's home because they double-clicked your button or having an API call retry three times and add an extra $2000 bucks to someone's account.

We can achieve idempotency with a few different implementations. 
- Simplest is to have userName be an unique key and add that as a constraint to the database. That way if you try to create two of those the database would crash
- For more editing-like requests, one could create an UUID for each request and block multiple occurrences of that on the server side. This would be called a deduplication ID. 

---
## SAGA

> *"What would you do if the first step of the transaction succeeded but any of the later ones didn't?"*
> Same as the last one, framed in one application I was working on.

SAGA, as much as it looks like, is not an acronym, but the name of a strategy used to orchestrate multiple steps in a transaction. 

Example:
**Store Purchase Transaction** 

CREATE ORDER -> PAYMENT -> INVENTORY RESERVATION -> SHIPMENT -> COMPLETE

This is an example of a multi-step transaction. It's only complete if all of the steps are successful. But what do I do if any of the later ones fail? There's no native way to just rollback everything once I have charged the user's credit card and blocked the item from being bought in the inventory.

**2PC - Two-Phase Commit**

A more legacy way to handle this issue would be the 2PC, a strategy where you have an orchestrator that askes every process to start and let them know when they are ready to complete. Only once everyone is ready the orchestrator let's everyone commit and concludes the full request. 

If anyone fails, everyone just rollback and nothing is actually done in the real world. Sounds good.

But if any of those take longer, we have 5 processes stuck, 5 tables locked and waiting. Other than that, this stablishes a SPOF (Single Point of Failure) for the whole system. Worked in the past, doesn't work anymore. 

**Compensation Task**

The solution is to make things asynchronous. So we need some way to be able to revert them after they are done, something we can trigger later once we find that a later step failed. 

Those are the compensation tasks.

| TASK                  | COMPENSATORY TASK    |
| --------------------- | -------------------- |
| CREATE ORDER          | DELETE ORDER         |
| PAYMENT               | DEVOLUTION           |
| INVENTORY RESERVATION | CLEAR RESERVATION    |
| SHIPMENT              | SCHEDULE CANCELATION |
So what happens is, if I fail at SHIPMENT, I can revert all of the earlier steps using their compensatory tasks. It's not a full rollback, the application did charge the customer, they did receive an email, that can't just be undone, but at least they won't be charged for a fridge they didn't receive.

**Choreography x Orchestrator**

There are to ways to implement this. 

The first one "trusts" the steps themselves to communicate with each other. The Payment system is triggered by a success message from the CreateOrder system. A PaymentSuccess message is sent by the Payment system to the Inventory system. If the Inventory fails they send an InventoryFail message to Payment, who requests the devolution from the bank and then messages the CreateOrder system to cancel the order.

Obviously when I say "message" you should read that it adds a message to a queue or topic to be read by the other end. It's all async after all.

This first implementation is the choreography. It works well for a smaller number of actors and a small number of possible interactions. Once we start having multiple possibilities it makes more sense to give the decision power to an **orchestrator**.

An orchestrator would manage all of the steps instead of leaving it to the actors themselves. The CreateOrderSuccess message would be sent to the Orchestrator, and then it would trigger the PaymentSystem, and so on. On the InventoryFail, it would trigger the Orchestrator who would simultaneously trigger the compensations from Payment and CreateOrder. 

Yes, this can stablish the SPOF issue again. But unlike the 2PC Strategy, it's able to do things in a non-blocking way without turning everything into a choreographic spaghetti. 

---
## Circuit Breaker (and the "Resilience Stack")

> *"How do you add resilience to your API Calls?"*
> At the time, the only thing I could think of where the `@CircuitBreaker` I saw annotated all over THD's API Calls, so I said that, but I didn't really know what it meant.

A Circuit Breaker is something you wrap around your most frequent API Calls to make it more resilient. So this block will be split into what this is (because I think it has the fancier and less clear name) and how it fits into the rest of the "Resilience Stack".

**Circuit Breaker Definition**

The idea of the Circuit Breaker comes from electricity: if you open (or break) an electric circuit, it will stop flowing with energy.

Imagine you have an API call to Open Weather (just an example). It usually takes around 5ms, but sometimes takes up to 200ms. You set a timeout for 500ms just to be safe. Than the service goes down. Every time you try and reach that API your will wait for 500ms before realizing it fails.

Now imagine you call them every 100ms. At one point, all of your available threads are waiting for Open Weather to never respond. Your system crashes. 

The Circuit Breaker stops that flow by saying "Oh, that's down, let me stop trying". It works in 3 stages:

CLOSED -> Everything works normally. If >50% (or any other threshold) of the calls fail, it opens the circuit
OPEN -> Open circuit, the API Call is not even attempted anymore. The client just receives an instant error message
HALF-OPEN -> After some time, the Circuit Breaker lets a few messages pass and test what happens. Depending on the situation, it goes back to either OPEN or CLOSED

In the end, this is a good way not to leave your threads stuck trying to hit a down system. But this isn't enough, is it? It's just part of the:

**Resilience Stack**

This is what really answers the question of *"How do you add resilience to your API Calls?"*:

1. **Timeout** - An API Call without a timeout is just a bug. It shouldn't even exist. Whenever you try to hit another endpoint, anything can happen, and you never want to leave your application hanging waiting for the other one. Add a timeout, a limit amount of time for your app to wait
2. **Retry** - Another one that sounds pretty obvious but needs saying. If you fail calling an API once, you might as well try again. And again. Maybe not the fourth time. Add a retry policy to your API Calls.
3. **Circuit Breaker** - This is where our friend comes in. Timeouts and Retries are awesome, but if the other system is down, they could still leave you hanging for a long time. 
4. **Bulkhead** - This is another one I hadn't heard much about before. Delegate a thread pool for each service: if you have 5 threads on service A and 10 in service B, even if service A gets stuck for any reason, B will continue to run as usual. 
5. **Fallback** - The last resource. If nothing works, control what it means to go wrong: null, exception, default value, user message, you name it. But name it.

They don't look so frightening one by one, but together they do real work to add resilience to your API Calls. 

---
## Apache Kafka vs. Message Queues

> *"Why did you choose to use Kafka instead of any other Message Queue?"*
> This one took my sleep away for a couple days...

Baffle yourself: Apache Kafka is not a Message Queue. I hope you knew that one before anyone asked you that question in an interview. I didn't. 

The default architecture of a Message Queue is a structure where things come in and things come out FIFO (First-In-First-Out), just as a real queue, as a real line. It works well to deliver messages between two parties, specially if you'll never need those messages again. 

You can also use Kafka for things like that, but it's different. Kafka is actually a distributed commit log, an event streaming platform. Instead of just delivering things, it writes every event into a sort of log file where they can be read from. 

Reading happens with offset, so multiple parties can get that same message. It also doesn't remove the message from the log as soon as it's read, rather, it has an expiration date for every piece of data. Because of that, you can natively replay all of the communication that ran through Kafka over a certain time. 

Both look similar and fulfil similar purposes, but Kafka makes more sense if you need it to do what it does best: deliver the message to multiple people and keep it stored for longer. If you just want it delivered from A to B, traditional MQs will serve you better and with less unnecessary complexity.

**Bonus: SQS**

I don't want to go to deep here, but on that interview I threw the name "SQS" there just so they though I knew something about it. I realized later I didn't. 

AWS SQS is closer to a standard MQ but with a few differences. Two I think are the most interesting are the inability to push a message (it's just polled) and the fact that it's not read exactly-once by default, but at-least-once by default, because of the way it only deletes the message when the consumer asks it too. If the first consumer takes to long to ack the SQS, a second consumer may actually find the message there.

It's also interesting how we could pair SQS with SNS to get something closer to what Kafka does to communicate multiple parties. But I don't want to go so deep into that now. 




