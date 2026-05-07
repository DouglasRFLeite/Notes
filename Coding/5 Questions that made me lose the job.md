
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








