# Interview

This is an interview for someone who has read every note in this course and done its labs. It starts with easy questions so the candidate settles in, then goes deeper, one stage at a time. The last stages have no single right answer. They test judgement: what to build first, what to accept, and how to defend a choice.

## How to run it

- **Time.** About 90 minutes. Stages 1 and 2 take 15 minutes together. Most of the time goes to stages 4 to 7.
- **Order.** Go through the stages in order. Inside a stage, pick the questions that suit the conversation. You do not need to ask them all.
- **Follow-ups matter more than first answers.** Each question lists follow-ups. A candidate who repeats a note's wording does well on the first answer. The follow-ups show whether they can use the idea somewhere the notes did not go.
- **Ask for the example.** Whenever the candidate states a rule, ask "show me where that broke, or would break, in the assistant". The course gives them a running example, the broker assistant, so they always have one.
- **Push back once.** Disagree with a good answer once, politely. Notice whether they hold a correct position, change it for a better reason, or give way to pressure.
- **"I don't know" is a good answer** when followed by how they would find out.

Each question has three parts:

- **Question:** what you ask.
- **Listen for:** what a strong answer contains. It is a guide, not a checklist to read out.
- **Follow-ups:** where to go next.

Red flags are listed at the end of each stage.

---

## Stage 1: warm-up (5 minutes)

The aim is to get the candidate talking about their own work. Do not score these heavily.

**1.1 Question:** Which note or lab changed how you think about AI systems the most, and why?

- **Listen for:** a specific moment, not a summary. For example, the moment in note 3 when a tool result passing through the model became model output again, or note 7 where "the model resisted" turned out not to be a control.
- **Follow-ups:** What did you believe before? What would you tell a colleague who still believes it?

**1.2 Question:** In two or three sentences, what is the assistant in this course, and what can it do by the end?

- **Listen for:** an investor's trading assistant that reads quotes, holdings, news and documents, remembers preferences, uses outside servers and agents, and places orders only after confirmation and code checks.
- **Follow-ups:** Which single power makes it dangerous? (Placing orders, and to a lesser degree reading private holdings and documents.)

**1.3 Question:** Which lab surprised you, where the result was not what you expected?

- **Listen for:** a real observation, such as a model that obeyed a hidden instruction in one run and refused in the next, or a test that passed for the wrong reason.
- **Follow-ups:** What did you change after you saw it?

---

## Stage 2: the core rule (10 minutes)

These check that the course's main idea is understood and not just remembered.

**2.1 Question:** The course says "the model proposes, code decides". Explain that to someone who has never built with a model.

- **Listen for:** a model's output is a suggestion, like an order typed by a client. Code that does not depend on the model checks it against facts the code holds (symbol, lot size, fat-finger limit, live price, holdings) before anything happens.
- **Follow-ups:** What can code not check? (Intent, and facts it does not have. That is why tools come in note 3.) Is a system prompt that says "never place orders above ₹1 lakh" a control? Why not?

**2.2 Question:** What is the difference between a jailbreak and a prompt injection?

- **Listen for:** a jailbreak attacks the model provider's safety training, and the user is usually the attacker. A prompt injection attacks our application through content the model reads, and the attacker is often a third party while the user is the victim.
- **Follow-ups:** Which one is your problem as the builder of the assistant? Can one attack be both?

**2.3 Question:** Name a hard control and a soft control from the course, and say when a soft control is still worth having.

- **Listen for:** hard means enforced by code and true every time (holdings taken from the session, a typed memory field the model cannot write). Soft means a classifier, a model judge or an instruction, which is right most of the time. Soft controls are worth having for detection, for harm in text where no hard rule fits, and as a second layer. They are never the only thing between an attacker and an order.
- **Follow-ups:** How do you know a soft control is working? (Measured misses and false alarms, note 10a.)

**2.4 Question:** Where does the identity of "my" come from when Ravi says "show my holdings"?

- **Listen for:** from the logged-in session or token, never from an argument the model supplies. The tool the model sees takes no account ID at all.
- **Follow-ups:** Why is removing the argument better than validating it?

**Red flags in stage 2:** calls instructions or a better model a control; thinks injection can be fixed by filtering keywords; cannot say where identity comes from.

---

## Stage 3: walk through the labs (15 minutes)

Now ask about what they built and saw. These show whether the labs were done or only read.

**3.1 Question:** In note 4, confirmation is tied to the exact order. Walk me through how you implemented that, and what happens if the model changes the quantity after Ravi says yes.

- **Listen for:** the confirmation is bound to the order's fields (a hash or the exact typed order). Any change invalidates it and needs a new confirmation.
- **Follow-ups:** The note also says a confirmation is not a safety check. What went wrong without the pending-sell check? (The same shares could be sold twice, each sell confirmed.)

**3.2 Question:** Tell me about your evaluation set from note 5. How did you know a security test was really testing something?

- **Listen for:** "test the test": run it against a broken version and see it fail. Security cases must pass on every run, not on average. Plain code tests for the hard rules, model evals for behaviour.
- **Follow-ups:** Who can edit the evaluation set, and why does that matter? (Whoever edits the tests changes the policy.) How many runs do you need before a pass rate means something?

**3.3 Question:** In the MCP lab, the vendor changed its tool description after you reviewed it. What did your code do?

- **Listen for:** the reviewed tool definitions are pinned (for example by a hash), and the agent refuses to start if they change. What reaches the model is what the server publishes, not what its documentation says.
- **Follow-ups:** With two servers, how can one shadow the other? What did you do about it? (Name tools after their server, route by a reviewed table, pass closed values.)

**3.4 Question:** Show me how Mallory got a premium report in note 7a with no injection at all.

- **Listen for:** one index for all documents, ranked first and filtered later or not at all. It is IDOR through search. The fix is principals on every document and a filter on the reader's principals from the session before ranking.
- **Follow-ups:** A document's permissions change after it was indexed. What else must change? (Embeddings, caches, summaries and anything made from the document.)

**3.5 Question:** In note 8, how did Mallory's post end up in Ravi's memory, and what design stopped it?

- **Listen for:** the model could write to memory, and it read Mallory's post. The fix is typed memory fields written only by the investor through the app, not by the model.
- **Follow-ups:** Is a summary of an old conversation memory? (Note 8a: yes in effect, and it is untrusted.)

**3.6 Question:** What did you change when you rebuilt the assistant with a framework in note 11?

- **Listen for:** a map of where each hand-written control now lives, and which ones the framework dropped or changed by default. Runtime-loaded prompts and skills treated as pinned, reviewed dependencies.
- **Follow-ups:** Which framework default surprised you the most?

**Red flags in stage 3:** answers only in OWASP IDs; cannot describe a single concrete result; says every lab "just worked".

---

## Stage 4: apply it to something new (20 minutes)

Give the candidate systems the course did not cover. Strong candidates carry the same ideas over without being told which note applies.

**4.1 Question:** A support team wants an assistant that reads customer emails and can issue refunds up to ₹5,000. Review the design.

- **Listen for:**
  - The lethal trifecta: customer emails are untrusted content, account data is private, and a refund is a way out (money leaves).
  - The model returns a closed choice (refund or not, from a list of reasons). Code builds the refund from the ticket's own data, not from the email text.
  - The refund amount and account come from the order record, not the email.
  - A daily limit per customer and per agent, and logging.
- **Follow-ups:** The business says ₹5,000 is small, so no human check is needed. Agree or disagree? (Look at many small refunds, and the attacker's ability to try many times.) What eval cases would you write first?

**4.2 Question:** A coding agent has read access to the company's repos and can open pull requests. What is the worst thing an issue comment can make it do?

- **Listen for:** an issue comment is untrusted content. The agent can read secrets in repos and can write to a pull request, so it could leak secrets into a PR, a link, or a commit. It could also add a backdoor that a tired reviewer approves.
- **Follow-ups:** Which of the six patterns from note 10b fits here? Which of the three trifecta legs do you remove?

**4.3 Question:** A hospital wants a RAG assistant over patient notes for doctors. What from note 7a carries over directly, and what is new?

- **Listen for:** principals per document and filter before ranking carry over. New: relationships change all the time (a patient moves ward), break-the-glass access with review, and shared spaces such as a ward handover where the answer is read by several people.
- **Follow-ups:** A doctor asks for "a summary of today's patients" in a shared handover channel. Whose permissions do you search with?

**4.4 Question:** Your company wants to let customers' own AI apps connect to your API through an MCP server. Where do you start?

- **Listen for:** this is a public API (deep dive 6a). OAuth tokens issued for this server and checked on every call, the client taken from the token, small scopes that expire, typed arguments and no shell, only the fields a task needs, and every call logged.
- **Follow-ups:** A customer's AI app is compromised. What can the attacker do with their token, and how fast can you stop it?

**4.5 Question:** A team wants an agent that runs every morning with nobody watching and can sell shares within limits the investor set. What changes compared with the assistant in Part 1?

- **Listen for:** approval moves to setup time as a typed mandate created in the app, with limits and an expiry. The plan is fixed at setup; the model returns a closed label. A short-lived token per run. Only listed sources can start a run. Notifications, a circuit breaker and a kill switch replace the person watching.
- **Follow-ups:** The attacker can now try once a day, forever. What stops that? (Run caps, alerts on unusual labels.)

**Red flags in stage 4:** starts with a vendor product name; adds a guardrail model as the main control; never asks where the data and the identity come from.

---

## Stage 5: design deep dive (20 minutes)

Pick one problem and stay on it. Let the candidate lead. Interrupt with the follow-ups as they go.

**5.1 Problem:** Design the full threat model for the broker assistant as it is at the end of the course, as if a review board meets tomorrow.

- **Listen for:**
  - A data-flow picture: investor, app, model provider, tools, MCP servers, retrieval index, memory store, outside agents, the order system.
  - Trust boundaries drawn where data crosses between owners, including the model provider and outside agents.
  - For each high risk: the control, whether it is hard or soft, and how it is tested.
  - The model provider treated as a third party: storage off, minimum data sent, a scoped key with a spending limit.
  - Fail-closed choices: what happens when the price service or the permission check is down.
  - Monitoring for an agent acting outside its job (orders after news reads, unusual tool sequences).
- **Follow-ups:**
  - Which three risks would you fix first if you had one quarter? Defend the order.
  - Where is the system weakest today, honestly?
  - Which control would you remove if it cost too much, and what would you watch instead?

**5.2 Problem:** Design how values carry their source through the assistant, so an order field that came from news is rejected (note 10b, step 3).

- **Listen for:** labels on every value, not one flag per source. Labels survive copying, summarising and the model passing them on. Code checks the label where it matters (order fields), and the confirmation shows where each part came from.
- **Follow-ups:**
  - The model rewrites a number from a news article as a "new" number. How does the label survive? (It does not, if the model can freely produce values. So order fields come only from typed sources the code fills, or the whole model output is labelled with its least trusted input.)
  - The attacker cannot change the quantity but can flip a condition, such as "sell if the price falls below X". What do you do?
  - What does this cost in developer effort, and how do you get teams to use it?

**5.3 Problem:** Design delegated access for an outside tax agent that needs to read Ravi's trades (notes 12 and 13).

- **Listen for:** a token with two identities, the user (`sub`) and the agent (`act`); read-only scope for trades; short-lived and revocable; the broker's API checks the token, not the words in the request. Tokens never enter the model's context. Send the minimum data, never forward Ravi's token to the tax agent. Its answer is untrusted: typed fields, checked against holdings, shown as suggestions.
- **Follow-ups:**
  - The tax agent later asks for a scope to place sell orders "to save Ravi time". How do you handle the request?
  - The tax agent's Agent Card changes its signing key. What happens in your system?

**Red flags in stage 5:** a long list of controls with no priorities; no mention of testing or monitoring; cannot say what is still weak.

---

## Stage 6: incidents and pressure (15 minutes)

These are about acting under uncertainty. Give the facts in pieces, and add the next fact only after the candidate responds.

**6.1 Scenario:** At 10:05 a trader reports that the assistant placed a sell order they say they never confirmed.

- Fact 2: the audit log shows a confirmation event for that order.
- Fact 3: the investor had pasted a PDF contract note into the chat two minutes earlier.
- **Listen for:** first contain (can more orders like this happen right now?), then gather evidence from the logs of model calls, tool calls and orders (note 15), then work out the path. Questions such as: was the confirmation bound to the exact order shown? Did text in the PDF shape what was shown? Was the confirmation UI built from code, or from streamed model text (note 8a)?
- **Follow-ups:** Who do you tell, and when? What test do you add so this cannot happen silently again?

**6.2 Scenario:** The model provider announces that the alias you use now points to a new model version, starting tonight.

- **Listen for:** you should already pin an exact model snapshot, not an alias (deep dive 14a). Any model change goes through evals, then shadow traffic, then a canary, with a way back.
- **Follow-ups:** The new model scores better overall but fails one security case one time in fifty. Ship or not?

**6.3 Scenario:** A researcher emails you a working injection: a hidden instruction in a public news post makes the assistant recommend a small stock to every investor who asks about the sector.

- **Listen for:** no order is placed, but it is market manipulation through the assistant's reply. Reproduce it, turn it into an eval case, decide whether to pause the news feature, and treat output harm (stock tips, invented prices) as real harm (note 10a).
- **Follow-ups:** The product team says the model "usually" ignores it, so it is low priority. How do you respond, with numbers?

**6.4 Pushback:** A product leader says: "Every new tool needs a security review, and it slows us down by two weeks. Remove it."

- **Listen for:** neither yes nor no at once. Size the review to the risk: tools that only read public data get a light check, tools with private data or a way out get the full review. Turn the review into reusable building blocks (typed tools, pinned servers, a permission-aware search service) so most tools need little review.
- **Follow-ups:** What would you measure to show the review is worth its cost?

**Red flags in stage 6:** jumps to a fix before containing; blames the model; agrees with any pushback to keep the peace, or refuses every pushback.

---

## Stage 7: limits and open questions (10 minutes)

The course says some problems are not solved. A strong candidate knows where the edges are and has an opinion.

**7.1 Question:** Is prompt injection a solvable problem? What would change your mind?

- **Listen for:** not solvable in the model alone today. Instructions and data share one channel, and adaptive attacks keep beating defences tested only on fixed attacks. The answer is system design that limits what a successful injection can do. A good answer names what evidence would change the view.
- **Follow-ups:** If a vendor shows a 99% block rate, what questions do you ask? (Against which attacks, adaptive or fixed, with what false alarm rate, measured how.)

**7.2 Question:** The six patterns in note 10b all give up some capability. When would you choose capability over safety, and how would you make that choice visible?

- **Listen for:** a choice tied to how bad the worst outcome is and whether it can be undone. Accepted risks written down with an owner, an expiry and monitoring, not hidden.
- **Follow-ups:** Give an example from the assistant where you would accept a risk.

**7.3 Question:** Monitoring is the backstop in many notes. What can monitoring not catch?

- **Listen for:** one quiet action that looks normal (a single small leak), harm in a reply that is never acted on, slow attacks spread over many sessions, and anything not logged. Monitoring that records client data is itself a store to protect (deep dive 14a).
- **Follow-ups:** How do you test that an alert would actually fire?

**7.4 Question:** What would you add to this course, and why?

- **Listen for:** a topic tied to a transferable security idea, with a reason it is missing, not a list of products.
- **Follow-ups:** Which note would you cut or merge?

---

## Stage 8: close (5 minutes)

**8.1 Question:** If you joined and found an AI feature already live with none of these controls, what would you do in your first two weeks?

- **Listen for:** learn what it can do and to whose data before changing anything. Find the trifecta and the ways out, then cut the most dangerous power first with the smallest change. Add logging if it is missing. Talk to the owning team before writing a report.
- **Follow-ups:** What would you not do in the first two weeks?

**8.2 Question:** What questions do you have for us?

- **Listen for:** questions about how decisions are made, who owns the model risk, and what is already in production. These show how they think about the work.

---

## Scoring guide

Score each stage from 1 to 4. Weight stages 4 to 7 twice as much as stages 1 to 3.

| Score | What it looks like |
| --- | --- |
| 1 | Repeats rules from the notes but cannot apply them to a new system |
| 2 | Applies the right idea when prompted, misses priorities or testing |
| 3 | Carries ideas to new systems without prompts; names hard and soft controls; says how each is tested; knows what is still weak |
| 4 | Everything in 3, plus sets clear priorities with reasons, holds a correct position under pushback, changes it for a better argument, and makes trade-offs visible to people outside security |

Across the whole interview, the strongest signals are:

- **Starts from the data and the identity.** Asks where each value comes from and who it belongs to before naming any control.
- **Separates proposing from deciding.** Never puts a model, an instruction or a classifier between an attacker and an irreversible action.
- **Tests everything it claims.** Every control comes with a way to show it works, and a way to show the test works.
- **Knows the edges.** Says plainly what is unsolved, and designs so that a failure is small.
- **Brings people along.** Explains a risk and a trade-off in plain words, and offers a way to move forward, not only a no.
