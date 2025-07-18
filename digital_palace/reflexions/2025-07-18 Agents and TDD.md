### The new trend of coding agents/vibe coding will push us towards a more **test-driven** approach.

There are many examples of huge refactors failing, where agents touch specific files to make an example case work, but only that one case. It's a bit like an overzealous genie who grants wishes literally, without the full underlying context. 

For me, one way to provide that context is by writing **unit tests**, since they encapsulate the functional and application logic. A second argument is that all studies show LLMs perform better with **"few-shot learning"** (ref tba). If these tests are integrated, the agent should be able to make the requested new feature work while still respecting the existing unit tests.

In conclusion, even if TDD has an initial cost, this cost can be very quickly offset by the enormous productivity gains offered by LLMs.