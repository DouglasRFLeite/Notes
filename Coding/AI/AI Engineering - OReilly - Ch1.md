
Without question, this is the most important sentence on this chapter:

> AI engineering refers to the process of building applications on top of foundation  models.

### What are Foundation Models?

NLP (Natural Language Processing) is not something new. Language Models (as they can also be called) work with **token statistical inference**. That is, if I show you the sentence “My name is…”, the most likely next word is “Mohammed”, because it’s the most common name in the world. 

>[!info]
>Yes, this does mean Language Models are Autocompletes on Steroids.

That said, I didn’t say Language Models work with Word Inference, I said **Token Inference**.

**Tokens** are the basic unit of Language Models. They represent, on average, 0.75 words. The word “cooking”, for example, can be represented by the tokens “cook” and “ing”, each representing a part of the original meaning. 

Now, from Language Models to Large Language Models (LLMs) and Foundation Models lies the huge step we took on these last couple of years.

LLMs are possible due to Self-Supervision, a training technique that reduced cost by millions or even billions of dollars, that’s without going into time. That said, LLMs are just that, very big Language Models. Same idea, just more data and more training.

From Foundation Models, on the other hand, the leap is in variety rather than in power. Foundation Models like GPT and Gemini are impressive mostly due to using **Multimodal Models**, meaning they can use text, image, audio and even videos to help them infere the next “token”. 

Finally, what makes Foundation Models powerful enough to complete disrupt our world as they have is the fact that they are General Purpose, meaning I can use them for coding, you can use it for marketing and Trump can use it for posting on X. All with the exact same model. 

### AI Engineering X ML Engineering X Fullstack Engineering

AI Engineering lies in an awkward middle ground between ML Engineering and Fullstack. That said, people often do AI Engineering without half of the necessary knowledge to do either ML or Fullstack Engineering.

ML concepts are very good to haves, specially when you are trying to take your model from 95% to 99% accuracy (or precision, or F1, or whatever you want). Fullstack Engineering is important when you need to improve your UI, reduce your costs or escale your application. That’s not even getting into DevOps (the book doesn’t).

That said, at the time I’m writing this, tons of people are creating full out integrations apps with n8n or make, enhancing models to working-level with prompt engineering and creating web apps with lovable.

Bottom line is: you can start with very little, but there will never be an end to what you can and should learn to improve.

### The AI Stack

One of the most interesting things I read on this chapter is also something I had seen in a couple places before, the separation between **Application Development, Model Development and Infrastructure**. 

To work with AI is too work in one of these fields, rarely more then one at a time.

When you think of Infrastructure work, you think of GPUs, you think of Cloud, you think of servers and data management. Say maybe you want to work with NVIDIA or AWS, this is what you’d be working with. 

Model Development is the work of OpenAI, Anthropic and Google. This is where you’d need a ton more ML Engineering knowledge. Differently from Infrastructure, this is a bit less impossible to be a Solopreneurs line of work, but still pretty unlikely.

Application Development is actually developing the applications that use, access or rely on Foundation Models. Most of the time when you hear the term AI Engineering (the book and our notes are no different) this is what it means.

### Different Trainings

The last point about this chapter I want to touch on is this. You may hear the term training, when related to Ai Engineering, meaning very different things.

**Pre-Training** is the first and most important step of model training. It’s when you reset and randomize all of the weights and use the whole training dataset to train the model from scratch. This consumes roughly 98% of the whole compute and data resources related to training. In ML Engineering, this is usually just referred to as Training.

**Finetuning** means running another round of training on the already trained model, usually to make it more of a specialist in one subject or to have it know data the original “trainer” couldn’t feed it.

**Post-Training** is, technically, the same as finetuning, but is used when the model developer, instead of the application developer, performs the new rounds of training.

**Prompt Engineering is not Training!** As much as it makes some sense to say you “trained it to do what you wanted”, the term Training in this field means something specific and that doesn’t involve prompt or context engineering. 

---
That’s it, now I can read the next chapter. Thanks past Douglas.

