# Interview

An interview for AI security. It starts with easy questions so the candidate settles in, then goes deeper one stage at a time. The early stages check knowledge. The later stages have no single right answer: they test judgement, priorities, and how the candidate brings other people along.

## How to run it

- **Time.** About three hours, best split into two sessions: stages 1 to 6 (knowledge and depth), then stages 7 to 11 (design, incidents, programme and judgement). If time is short, cut from stages 2 to 4, never from stages 7 to 10.
- **Order.** Go through the stages in order. Inside a stage, pick the questions that suit the conversation.
- **Follow-ups matter more than first answers.** A prepared candidate answers the first question well. The follow-ups show whether they can reason past what they have read.
- **Ask for the example.** Whenever the candidate states a rule, ask "where have you seen that break, or where would it break?"
- **Push back once.** Disagree with a good answer once, politely. Notice whether they hold a correct position, change it for a better reason, or give way to pressure.
- **"I don't know" is a good answer** when followed by how they would find out.

Each question has:

- **Question:** what you ask.
- **Listen for:** what a strong answer contains. A guide, not a checklist to read out.
- **Follow-ups:** where to go next.

Red flags are listed at the end of each stage.

---

## Stage 1: warm-up (10 minutes)

Get the candidate talking about their own work. Do not score these heavily.

**1.1 Question:** Tell me about an AI system you secured, or tried to break. What did it do, and what worried you most about it?

- **Listen for:** a concrete system, what it could read and what it could do, and a specific worry tied to that power, not a generic list of AI risks.
- **Follow-ups:** What did you change because of that worry? What would you do differently now?

**1.2 Question:** What changed your mind about AI security in the last year or two?

- **Listen for:** a real shift in belief, for example "I used to think a better system prompt helped, now I treat the model as untrusted".
- **Follow-ups:** What evidence changed it?

**1.3 Question:** How would you explain the main security problem with LLM applications to a product manager in two minutes?

- **Listen for:** plain words, no jargon. The model follows instructions it finds in any text it reads, so anything it can read can steer what it does. So we limit what it can do and check its output in code.
- **Follow-ups:** Now explain it to the board in thirty seconds.

---

## Stage 2: foundations (15 minutes)

**2.1 Question:** What is the difference between a jailbreak and a prompt injection? Which one is your problem as the builder of an application?

- **Listen for:** a jailbreak attacks the model's safety training, usually by the user. A prompt injection attacks the application through content the model reads, often by a third party, with the user as the victim. Injection is the builder's problem.
- **Follow-ups:** Can one attack be both? Is direct injection by the user a security problem if the user can only hurt themselves?

**2.2 Question:** Why can't the model providers just fix prompt injection?

- **Listen for:** SQL injection was fixed by separating code from data. A model has one stream of tokens; system and user roles are trained habits, not a boundary. Following instructions in text is the model's job, intent is not in the text, and the attacker moves second. Research defences lower the rate; adaptive attacks keep breaking them. Guarantees come only from system design that limits what a fooled model can do, at a cost in usefulness.
- **Follow-ups:** A vendor shows a 99% block rate. What do you ask? (Fixed or adaptive attacks, how many tries, false alarm rate, measured how.)

**2.3 Question:** What is the difference between a hard control and a soft control? Give an example of each in an LLM application.

- **Listen for:** hard means enforced by code and true every time (the account comes from the session, not from the model; an order limit in code). Soft means a classifier, a model judge or an instruction, right most of the time. Soft controls are useful for detection and for harms where no rule fits, never as the only thing before an irreversible action.
- **Follow-ups:** How do you know a soft control is working? An attacker can query your classifier as often as they like. What does that do to it?

**2.4 Question:** A user says "show me my account". Where should the word "my" be resolved?

- **Listen for:** from the authenticated session or token, in code. The tool the model calls should not take an account ID at all.
- **Follow-ups:** Why is removing the argument better than validating it?

**2.5 Question:** Which facts about how models work change how you test them?

- **Listen for:** output is sampled, and even temperature 0 is not fully deterministic, so security results are rates, not pass/fail. Safety training is shallow and late, and can be removed from open weights. The model reads tokens, so role markers in user text can be dangerous on a self-hosted model whose tokenizer parses them.
- **Follow-ups:** How many runs do you need before a pass rate means something?

**2.6 Question:** Walk me through the "lethal trifecta" or an equivalent way you reason about dangerous combinations.

- **Listen for:** private data, untrusted content and a way for data or actions to leave. An agent with all three can be made to leak or act. Remove one leg per agent, or split the work.
- **Follow-ups:** Name three "ways out" people forget. (Rendered images and links, tool calls with URLs, writes others can read, outbound email, DNS from a sandbox.)

**Red flags in stage 2:** treats instructions or a better model as a control; thinks injection can be filtered with keywords; cannot say where identity comes from; treats one passing run as proof.

---

## Stage 3: agents and tools (25 minutes)

**3.1 Question:** An assistant can place payments after the user confirms. How do you make the confirmation mean something?

- **Listen for:** the confirmation is bound to the exact action (amount, payee, account), built from code and not from model text, and any change needs a new confirmation. A confirmation is not a safety check: limits, duplicates and balance checks still run in code.
- **Follow-ups:** The user approves fifty prompts a day. Is the confirmation still a control? What about a spoken "yes" on a voice channel?

**3.2 Question:** Your agent uses a third-party MCP server. What are the risks, and what do you do?

- **Listen for:** tool descriptions and results are untrusted text; the server can change its tools after review; one server's description can shadow another server's tools. Pin reviewed tool definitions and refuse to start if they change, name tools by server, route by a reviewed table, pass only closed values, and treat the server as a vendor.
- **Follow-ups:** Now you publish your own MCP server for customers' AI apps. What changes? (It is a public API: OAuth, client from the token, small expiring scopes, typed arguments, no shell, minimal fields, every call logged.)

**3.3 Question:** You build retrieval over internal documents with different permissions. How does it leak, even with no attacker text?

- **Listen for:** one index, ranked then filtered (or never filtered), returns what the user cannot open: IDOR through search. Each chunk carries its principals; filter on the reader's principals from the session before ranking; check against the source. Permission changes and deletions must reach embeddings, caches and summaries; anything derived from a document inherits its strictest permission.
- **Follow-ups:** The answer is posted in a shared channel. Whose permissions do you search with?

**3.4 Question:** The agent has long-term memory. How does it get poisoned, and how do you design it?

- **Listen for:** if the model can write to memory after reading untrusted text, an attacker can plant a lasting instruction or preference. Memory as typed fields written by the user through the app, separate per user, never free text the model writes. Summaries of old conversations are untrusted too.
- **Follow-ups:** After an incident, what must be cleaned besides the logs?

**3.5 Question:** You split an agent into a research agent and an action agent. What does that buy you, and what does it not?

- **Listen for:** separate tool lists and identities, so the agent reading untrusted text cannot act. But the handoff between them is untrusted input: pass typed, closed values, not free text that the action agent obeys.
- **Follow-ups:** Name design patterns that limit injection by structure. (Action selector, plan then execute, map-reduce, dual LLM, code then execute, context minimisation.) Which gives up the most usefulness?

**3.6 Question:** The model writes code or SQL that you run. How do you contain it?

- **Listen for:** model-written code is untrusted code. Prefer a typed tool; otherwise a sandbox with a strong boundary (gVisor or a microVM over a plain container, since a container shares the host kernel), no network, a fresh sandbox every run, resource limits, and only a capped result out. Model-written SQL is SQL injection with the model as the typist: read-only, only this user's rows.
- **Follow-ups:** "No network" is set. How can data still leave? (DNS, a trusted tool that runs a file the sandbox wrote, results that carry data.)

**3.7 Question:** An outside agent needs to read some of a user's data. How do you give it access?

- **Listen for:** delegated, scoped, short-lived, revocable tokens carrying both the user and the agent identity; no shared service account and no passwords; the API checks the token, not the request text; tokens never enter the model's context. For agent-to-agent calls: an agent card is a claim, so allowlist, verify the signature and pin it; send the minimum; treat the answer as untrusted.
- **Follow-ups:** The partner agent asks for a write scope "to save the user time". What do you do?

**3.8 Question:** The business wants an agent that acts with no human present, for example every morning. What changes?

- **Listen for:** approval moves to setup time: a typed mandate with limits and an expiry, created in the app, not in chat. The plan is fixed at setup; the model returns a closed label and code builds the action. Short-lived per-run credentials, run caps, only listed triggers, notifications, a circuit breaker and a kill switch.
- **Follow-ups:** The attacker can now try every day, forever. What limits that?

**3.9 Question:** The agent reads screenshots, PDFs and web pages, or operates a browser or a whole computer. What is new?

- **Listen for:** instructions hide where humans cannot see them (images, white text, metadata, invisible Unicode). The user is trusted for their intent, not for the content of a file they forward. Browsing read-only, logged out and allowlisted; actions go through APIs, not clicks; a computer-use agent runs in a disposable VM.
- **Follow-ups:** How do you strip text the user cannot see before it reaches the model?

**Red flags in stage 3:** trusts tool descriptions because the vendor is reputable; filters search results after ranking; lets the model write memory freely; runs model code in a plain container on a shared host; gives an agent a shared admin token.

---

## Stage 4: models and infrastructure (20 minutes)

**4.1 Question:** The team wants to run an open model from a public hub. What can go wrong before it answers a single question?

- **Listen for:** some formats (pickle-based) run code on load. Use data-only formats such as safetensors, pin the revision and file hash, check the publisher, load new models in isolation.
- **Follow-ups:** The file is clean and from the right publisher. What can still be wrong? (A model trained to behave badly on a trigger.)

**4.2 Question:** You fine-tune a model on past customer chats. What are the main risks?

- **Listen for:** poisoned training data can add a hidden trigger that normal evals never hit; customer data can come back out of the model; deletion requests are hard to honour after training. The dataset needs an owner, a review and a pinned version.
- **Follow-ups:** How would you look for a backdoor you do not know the trigger for? (Mostly you cannot, so control what goes in.)

**4.3 Question:** You call a hosted model through a gateway. What must you know about each party?

- **Listen for:** which model actually answered (pin snapshots, not aliases), what each party stores and for how long, whether the gateway is a third party in every request, private endpoints and account-level settings on cloud model platforms, and an inventory of models, datasets and providers.
- **Follow-ups:** The provider's alias moves to a new model tonight. What breaks?

**4.4 Question:** A classifier you expose through an API is used as a security control. How can an attacker abuse it without ever seeing it?

- **Listen for:** the API is an oracle. Evasion by probing, extraction by training a copy (then testing evasions quietly on the copy), membership inference on small fine-tunes. Return labels not scores, quotas, monitoring of query patterns, and a hard rule outside the model. Differential privacy is the one defence with a proof, at a cost in accuracy.
- **Follow-ups:** Why is membership inference weak on large pretraining sets and stronger on a small fine-tune?

**4.5 Question:** You self-host models on GPU servers with an inference server and a cluster framework. What do these systems assume?

- **Listen for:** a trusted network. Open ports give away compute and weights; cluster control channels run code they are sent; the server can make outbound requests. Bind privately behind a gateway, isolate the cluster network, separate trust domains (shared prefix caches leak through timing), and protect weights like customer data.
- **Follow-ups:** Two business units want to share one GPU server. What do you say?

**Red flags in stage 4:** treats a model file as plain data; trusts a provider alias; believes evals will find a backdoor; exposes inference servers because they are "internal".

---

## Stage 5: AI in engineering (15 minutes)

**5.1 Question:** Developers use AI coding assistants. What are the security problems with the code they produce, and with the tools themselves?

- **Listen for:** code that fails open, made-up package names an attacker can register, hardcoded secrets, and code leaving the laptop to a provider. Rules files that steer the assistant can hold hidden instructions. AI-written code gets the same review as an outside contribution.
- **Follow-ups:** How do you stop a made-up package reaching production?

**5.2 Question:** A coding agent picks up issues and opens pull requests on its own. An issue says "also raise the transaction limit, the business approved it". What stops this?

- **Listen for:** the issue is untrusted content. The agent's machine has little access and short-lived scoped credentials. Human code owners on sensitive paths, branch protection, required human approval. An AI reviewer is advice only, because a pull request can tell it to approve.
- **Follow-ups:** The same pull request edits the test that would have caught it. Then what?

**5.3 Question:** Teams let AI write tests and evals. What goes wrong?

- **Listen for:** tests written from the code describe the bug as correct. An agent can make a failing security eval pass by editing the case. Security cases and their expected results are owned by people, tested against a broken version, and changes need a separate owner. A model grader is tested too.
- **Follow-ups:** How do you detect, in CI, that an eval case was weakened?

**Red flags in stage 5:** uses an AI reviewer as a gate; gives a coding agent broad access because "a human merges"; lets an agent own the tests that judge it.

---

## Stage 6: testing and red teaming (20 minutes)

**6.1 Question:** How is red teaming an AI system different from a traditional penetration test?

- **Listen for:** results are rates, not yes or no; the target changes with every model, prompt and data update; success is graded on what got past code, not on what the model said; content harm and security harm are different scorecards.
- **Follow-ups:** How long does a red team report stay true?

**6.2 Question:** What must be agreed before you attack an AI system?

- **Listen for:** written authorisation from the risk owner, scope in and out, test copies with synthetic data and canaries, budgets and stop conditions enforced in the harness, deconfliction with defenders, private handling of transcripts. Third-party services are not attacked without their permission.
- **Follow-ups:** The attack path runs through a vendor's API. In scope?

**6.3 Question:** An automated search found an attack that worked 40% of the time. What is the attack success rate?

- **Listen for:** unknown from that number. A search's best score is selected and biased upward. Freeze the attack, measure on held-out tasks, report the denominator and a confidence interval, and the agent's usefulness under attack.
- **Follow-ups:** 0 successes in 30 tries. What can you say? (Not zero risk; give the upper bound, roughly 10% at 95% confidence.)

**6.4 Question:** In an agent test, how do you tell "the model said it did it" from "it happened"?

- **Listen for:** record requested, blocked and executed actions separately, with instrumented or fake tools. Grade state, not words. Test the grader on labelled cases before trusting it; "unknown" is a score, not a pass.
- **Follow-ups:** Your grader is an LLM. How do you calibrate it?

**6.5 Question:** Your attack fails. Before you conclude the defence works, what do you check?

- **Listen for:** that the attack text actually reached the model (delivery), that the control case behaves as expected, that one thing changed at a time, and enough runs to matter.
- **Follow-ups:** What makes a good plan for what to test? (Start from the data flow and trust boundaries, not a jailbreak list.)

**6.6 Question:** Write the headline of a finding as you would in a report, and say what it does not claim.

- **Listen for:** claims only what the evidence supports: the effect, the rate with bounds, the conditions. Severity, likelihood and confidence kept separate. The fix in code, tested without a model, not breaking the real task, and retested before closing.
- **Follow-ups:** How do you report a finding to an open-source project or a vendor?

**Red flags in stage 6:** reports a best-of-search score as a rate; no written authorisation; grades the model's words; says "we found nothing, so it is safe".

---

## Stage 7: design deep dive (25 minutes)

Pick one problem and stay on it. Let the candidate lead; interrupt with the follow-ups.

**7.1 Problem:** Threat model an AI assistant for a retail trading app. It reads market news and the user's holdings, remembers preferences, uses a third-party market-data server, talks to a partner tax agent, and can place orders after confirmation.

- **Listen for:**
  - A data flow: user, app, model provider and gateway, tools, outside servers, retrieval index, memory, partner agent, order system.
  - Trust boundaries where data crosses owners, including the model provider.
  - The dangerous combinations, and which agent loses which leg.
  - For each high risk: the control, hard or soft, and how it is tested.
  - Fail-closed behaviour when a dependency is down; failover to a backup model treated as a model change.
  - Monitoring for an agent acting outside its job, such as orders right after reading news.
  - Output harm with no action: invented prices, stock tips, leaked identifiers, links that leak data when rendered.
- **Follow-ups:**
  - Which three risks do you fix first with one quarter of effort? Defend the order.
  - Where is it weakest after your design?
  - Which control would you drop if it cost too much, and what would you watch instead?

**7.2 Problem:** Design how values carry their source through an agent, so an action field that came from untrusted text is rejected.

- **Listen for:** labels on values, not one flag per source; labels survive copying and summaries; code checks labels where actions are built; the confirmation shows the source of each part.
- **Follow-ups:**
  - The model restates a number from an article as its own. How does the label survive? (Action fields come only from typed sources code fills, or the whole output takes its least trusted input's label.)
  - The attacker can only flip a condition, not a value. Then what?
  - What does this cost developers, and how do you get teams to adopt it?

**7.3 Problem:** Design an AI security platform for a company with forty teams building AI features.

- **Listen for:** shared services: AI gateway, guardrails, tool/MCP registry, agent identity, permission-aware retrieval, sandboxes, evals, logging. The platform owns who calls, which model, what leaves and what is recorded; the application owns what each action may do. The gateway becomes the most sensitive system and sees text, not meaning. SaaS and local AI need equal controls and an inventory. Adoption by making the safe path easy.
- **Follow-ups:**
  - What must the platform not own?
  - A team bypasses the gateway because it adds 200 ms. What do you do?

**Red flags in stage 7:** a long list with no priorities; no testing or monitoring; cannot name what is still weak; everything in the platform, or nothing.

---

## Stage 8: incidents (20 minutes)

Give the facts in pieces. Add the next fact only after the candidate responds.

**8.1 Scenario:** A customer says the assistant placed an order they never confirmed.

- Fact 2: the audit log shows a confirmation.
- Fact 3: the customer had uploaded a PDF two minutes earlier.
- **Listen for:** declare on impact, not on how the model's text looks. Contain first: can it happen again right now? Assign command. Preserve evidence: prompts, tool calls, actions, the file, model snapshot and prompt version, enough to reproduce. Then the path: was the confirmation bound to the exact action shown, built from code or from model text, and did the PDF shape it?
- **Follow-ups:** Who is notified, and when? What test is added? What must be cleaned (memory, indexes, summaries)?

**8.2 Scenario:** A researcher reports that a hidden instruction in a public web page makes your assistant recommend a small stock to everyone asking about its sector.

- **Listen for:** no action taken, but it is market manipulation through your output. Rate by what the attacker reached. Reproduce, measure the rate, add an eval case, decide whether to pause the feature, thank the researcher.
- **Follow-ups:** Product says the model "usually" ignores it. Answer with numbers.

**8.3 Scenario:** The finance head gets a video call from the CEO asking for an urgent transfer to a new supplier. Voice and face are right.

- **Listen for:** AI makes old fraud cheaper. Detecting fakes is a classifier against an attacker who moves second, so trust what AI cannot fake: a call back on a channel the requester did not choose, two approvers, passkeys, low limits for new payees. Train on the process, not on spotting fakes.
- **Follow-ups:** The CEO finds the call-back rule embarrassing. What do you say?

**8.4 Scenario:** A coding agent's pull request merged last week. Today you learn it added a dependency registered three days before.

- **Listen for:** likely a squatted made-up package. Find where it ran, rotate secrets build and runtime could reach, pull the release. Then fix the path: dependency allowlist or age check, owners on dependency changes, less access for the agent.
- **Follow-ups:** How would you have caught it before merge?

**8.5 Scenario:** A widely used AI vendor discloses that customer prompts were exposed for two weeks.

- **Listen for:** know what you sent them (inventory and logging), what they stored, which data was personal or regulated, legal and notification timelines in parallel, rotate anything secret that went in prompts, and review the contract terms you actually have.
- **Follow-ups:** How do you prove you can leave this vendor in 90 days?

**Red flags in stage 8:** fixes before containing; blames the model; trusts the face on the call; no evidence preserved.

---

## Stage 9: running AI security across a company (25 minutes)

**9.1 Question:** You have time for only a few deep reviews a quarter. How do you decide which AI features get one?

- **Listen for:** a short intake for every feature, vendor AI included, and a risk tier computed from facts: what it can do, what data it touches, what untrusted text it reads, whether anything can leave, who uses it, whether it acts alone. Recompute on every new tool, source, scope or way out. A design change is the cheapest way down a tier.
- **Follow-ups:** How do you find the features nobody declared? (Gateway keys, blocked provider calls, bills, DNS logs, SaaS settings, consent logs.)

**9.2 Question:** The company wants to switch on an AI assistant for every employee, connected to email, chat and documents. What do you do first?

- **Listen for:** it makes old over-sharing easy to find, so fix permissions before launch. Pages many people can edit are untrusted text for every reader. Malware on a laptop can use the company's AI tools.
- **Follow-ups:** How would you find the worst over-shared content quickly?

**9.3 Question:** The board asks: "Are we safer than last year?" What do you show?

- **Listen for:** not counts of reviews or blocked prompts. Coverage from registries (target 100%), effectiveness on a fresh attack suite as well as a fixed one, attacks stopped by code rather than by the model, worst-case exposure if the model were fully controlled, time to detect and stop from drills. One page, with the decisions the board must make.
- **Follow-ups:** Attack success fell from 20% to 3%. Why might that mean nothing?

**9.4 Question:** A business owner wants to launch with a risk you think is too high. What happens?

- **Listen for:** security does not own the business decision. The owner signs the remaining risk in writing, with an expiry and monitoring. Release thresholds agreed before testing. Risks outside appetite go to whoever set the appetite.
- **Follow-ups:** The acceptance expires and nothing changed. Then what?

**9.5 Question:** How do you handle NIST AI RMF, ISO/IEC 42001, the EU AI Act and MITRE ATLAS without four separate projects?

- **Listen for:** one control list with owners and evidence, mapped outwards. Evidence collected once from the platform. Each framework does a different job: guidance, a certifiable management system, law, an attack knowledge base. Legal owns obligations. Compliance is not security: a certificate covers a management system, not a model.
- **Follow-ups:** Is a feature lawful because it passed security review?

**9.6 Question:** Who owns what? Give owners for a feature's model behaviour, the gateway, the feature's evals, the risk acceptance, and the incident.

- **Listen for:** ownership where the work is: feature team for behaviour and evals (security owns the security cases), platform team for the gateway, business owner for the risk, an incident commander named before the crisis. Independent assurance stays separate.
- **Follow-ups:** You are asked to own all AI risk. Why say no, and what do you offer instead?

**9.7 Question:** How do you resource and fund this work when demand is ten times your capacity?

- **Listen for:** a service catalogue, capacity calculated before promising dates, platform controls that remove repeated reviews, funding the whole control (people and running cost, not just the licence), and making leadership choose between explicit options.
- **Follow-ups:** What do you stop doing?

**Red flags in stage 9:** measures activity, not outcomes; wants security to own every decision; treats a framework mapping as proof of safety; no plan for undeclared AI.

---

## Stage 10: influence and judgement (15 minutes)

**10.1 Question:** A product leader says every AI feature review slows them by two weeks and wants it removed. What do you do?

- **Listen for:** neither yes nor no at once. Size reviews to the risk tier, turn repeated findings into platform building blocks, measure review time and findings, and offer a faster path for low tiers.
- **Follow-ups:** What would convince you to remove a review entirely?

**10.2 Question:** Tell me about a time you were wrong about a security risk. How did you find out?

- **Listen for:** a real example, owned plainly, and what changed in how they decide.
- **Follow-ups:** How did you tell the people who had acted on your advice?

**10.3 Question:** Two engineers disagree on whether to use a guardrail model or a structural design change. How do you settle it?

- **Listen for:** frame the decision by worst outcome and reversibility, ask for measured evidence, prefer structure for irreversible actions and soft controls for detection, write the decision and who made it.
- **Follow-ups:** What if the structural change makes the product much less useful?

**10.4 Question:** When would you choose capability over safety, and how do you make that choice visible?

- **Listen for:** tied to how bad and how reversible the worst outcome is; accepted risks written with an owner, expiry and monitoring; never hidden.
- **Follow-ups:** Give a real example.

**Red flags in stage 10:** cannot name a mistake; wins by authority, not evidence; says no without a way forward.

---

## Stage 11: open questions and close (10 minutes)

**11.1 Question:** Is prompt injection solvable? What would change your mind?

- **Listen for:** not in the model alone today; system design limits the damage. Names evidence that would change the view.

**11.2 Question:** What can neither monitoring nor red teaming catch?

- **Listen for:** one quiet action that looks normal, harm in replies nobody acts on, slow attacks across sessions, anything unlogged, attacks nobody thought of. Monitoring that stores customer data is itself a store to protect.
- **Follow-ups:** How do you test that an alert would fire?

**11.3 Question:** You join and find an AI feature live with none of these controls. What do you do in your first two weeks?

- **Listen for:** learn what it can do and to whose data first; find the dangerous combination and the ways out; remove the most dangerous power with the smallest change; add logging; register and tier it; talk to the owning team before writing a report.
- **Follow-ups:** What would you not do in the first two weeks?

**11.4 Question:** What questions do you have for us?

- **Listen for:** questions about how decisions are made, who owns AI risk, and what is already in production.

---

## Scoring guide

Score each stage from 1 to 4. Weight stages 7 to 10 twice as much as the others.

| Score | What it looks like |
| --- | --- |
| 1 | Repeats rules but cannot apply them to a new system |
| 2 | Applies the right idea when prompted; misses priorities, testing or links between areas |
| 3 | Carries ideas to new systems unprompted; separates hard and soft controls; says how each is tested and measured; knows what is still weak |
| 4 | Everything in 3, plus clear priorities with reasons, holds a correct position under pushback, changes it for a better argument, and makes trade-offs and ownership clear to people outside security |

A candidate strong in stage 9 but weak in stages 2 to 6 knows programme language without the system underneath. A candidate strong on systems but weak in stage 9 can secure one feature but not yet many. Note which.

The strongest signals across the interview:

- **Starts from the data and the identity.** Asks where each value comes from and whose it is before naming a control.
- **Separates proposing from deciding.** Never puts a model, an instruction or a classifier alone between an attacker and an irreversible action.
- **Measures, not asserts.** Every control has a test, every rate has a denominator and uncertainty, and every test is tested.
- **Knows the edges.** Says plainly what is unsolved, and designs so failures are small.
- **Brings people along.** Explains risk in plain words, puts decisions with their owners, and offers a way forward, not only a no.
