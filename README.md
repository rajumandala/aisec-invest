## What these notes are

These notes explain how AI applications are built and how they are attacked, one concept at a time. They follow one running example: an **AI order assistant inside a broker's trading app**.

An investor types in plain English, for example "buy 10 Reliance at 2900 limit" or "sell half my Infosys". The assistant turns that into real broker actions. It starts as a single model call. Note by note, it learns to check orders, look up prices and holdings, read market news and screenshots, remember preferences, filter its own replies and place orders after the investor confirms. It then works with agents run by other companies, using only the powers the investor has granted. The course ends with a review of the whole system and the rules a broker must follow.

Each note adds one idea to the same assistant. Each note ends with a problem that the next note solves. The full order is in [Syllabus](00.%20Syllabus.md).

The course has two parts. **Part 1** is the core course: read it in order, and it tells the whole story. **Part 2** is a set of optional deep dives in the [deep-dives](deep-dives) folder, for readers who build or run such systems. Skip them on a first read.

Three appendix notes map the course to the OWASP Top 10 lists, one category at a time: [A1. OWASP Top 10 for LLM Applications](A1.%20OWASP%20Top%2010%20for%20LLM%20Applications.md), [A2. OWASP Top 10 for Agentic Applications](A2.%20OWASP%20Top%2010%20for%20Agentic%20Applications.md) and [A3. OWASP MCP Top 10](A3.%20OWASP%20MCP%20Top%2010.md).

## The market ideas these notes build on

| AI idea | What it is like in a broker's system |
|---|---|
| The model's output is untrusted | An order typed by a client. The broker never trusts it as it is |
| Code makes the decision | The RMS (risk management system) checks every order, wherever it came from |
| Tool calling | The quote and holdings APIs |
| Human approval | The order confirmation screen, for that exact order |
| Indirect prompt injection | A pump-and-dump post that the assistant reads |
| Delegated authorisation | A DDPI: the investor grants the broker one limited power, not full control of the demat account |
| Another company's agent | A third-party research report. Read it, check it, and never let it place an order |

## People in the examples

| Name | Role |
|---|---|
| Asha, Ravi | Investors who use the assistant, each with their own trading account |
| Mallory | An attacker who has an ordinary account and can post on forums, blogs and social media |
| ExampleTax | A tax firm that runs an outside tax agent |

Prices, holdings and companies in the examples are made up unless a note says otherwise.

After the course, [Part S: AI in the SDLC](https://github.com/rajumandala/adlc) looks at the team that builds the assistant using AI to write, review and test its code. [Part M: Model Supply Chain](https://github.com/rajumandala/ai-supplychain) then looks at the models it depends on. [Part P: Running AI Security at a Company](https://github.com/rajumandala/aisec-program) takes the controls from one assistant to every AI feature in the company. [Part R: AI Red Teaming](https://github.com/rajumandala/ai-redteam) attacks AI systems the way an attacker would, and measures and reports what it finds.
