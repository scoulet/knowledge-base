### The new trend of coding agents/vibe coding will push us towards a more **test-driven** approach.

There are many examples of huge refactors failing, where agents touch specific files to make an example case work, but only that one case. It's a bit like an overzealous genie who grants wishes literally, without the full underlying context. 

For me, one way to provide that context is by writing **unit tests**, since they encapsulate the functional and application logic. A second argument is that all studies show LLMs perform better with **"few-shot learning"** (ref tba). If these tests are integrated, the agent should be able to make the requested new feature work while still respecting the existing unit tests.

In conclusion, even if TDD has an initial cost, this cost can be very quickly offset by the enormous productivity gains offered by LLMs.

## ✅ Confirm / Support the idea

- [**Test‑driven development with AI**](https://www.builder.io/blog/test-driven-development-ai)  
  Explains how generative‑AI agents (e.g. Cursor, Claude Code) empower a test‑first workflow—reviving TDD as a default, not a best practice you skip.

- [**Test Driven Development Meets Generative AI**](https://www.btc‑embedded.com/test-driven-development-meets-generative-ai/)  
  A use‑case article showing LLMs generating unit tests from requirements first, then implementation—bridging specification‑as‑code with edge‑case detection.  

- [**Does Few‑Shot Learning Help LLM Performance in Code Synthesis?**](https://arxiv.org/pdf/2406.18181v1)  
  A 2025 arXiv study demonstrating that framing tests as few‑shot examples significantly improves code generation reliability.  

## 🧠 Qualify / Nuance the idea

- [**A Comparative Case Study on the Impact of Test‑Driven Development**](https://arxiv.org/pdf/1711.05082.pdf ) (2017)  
  Shows TDD yields better design and maintainability, but adoption often slows development and requires overcoming cultural resistance.  

- [**Operational Definition and Automated Inference of TDD**](https://csdl.ics.hawaii.edu/techreports/2009/09-01/09-01.pdf) (Hawai‘i study)  
  Developers reported TDD improved productivity, but actual practice often consumed ~16% more time—highlighting the upfront cost of test‑first workflows.  
  


**Tags**:  
#reflexion #llm #agents #tdd #tests #context #codegen #software

