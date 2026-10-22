# Sample Strong Answers

These are sample strong answers to the questions in [interview.md](interview.md), with the same numbering. They are written in the candidate's voice. They are not scripts: a real strong answer uses the candidate's own examples and may reach the same point by a different route. Each answer also covers the main follow-ups, marked **On the follow-ups**.

Questions about the candidate's own experience (1.1, 1.2, 10.2) have no right answer. Their samples show the shape of a strong answer: concrete, honest, and tied to a decision.

---

## Stage 1: warm-up

**1.1 An AI system you secured, or tried to break.**

"We had a support assistant that read customer tickets and could look up orders and issue store credit. What worried me was the combination: it read text written by anyone on the internet, it could see other customers' order data through a lookup tool, and it could move value. The model was the only thing between a ticket and a credit.

We changed three things. The lookup tool stopped taking a customer ID; it used the ticket's verified customer. Credit amounts came from the order record, not from the model. And credits above a small limit went to a person.

**On the follow-ups:** I'd now start from those powers on day one instead of adding them after a red team found them. And I'd have written the eval cases first, because two of our fixes regressed later and nobody noticed for a month."

**1.2 What changed your mind.**

"I used to think prompt hardening was a real layer: a better system prompt, delimiters around untrusted text. Then I watched the same injection fail nine times and work the tenth, and I watched adaptive attacks get through every published prompt-level defence. Now I treat the model as an untrusted component that is right most of the time, and I put anything that must be true every time into code.

**On the follow-ups:** the evidence was my own repeated runs, plus research showing that defences measured against fixed attacks fall apart against attackers who adapt to them."

**1.3 The main security problem, for a product manager.**

"A model can't tell the difference between instructions from us and instructions hidden in something it reads: an email, a web page, a document. So anyone who can get text in front of it can try to steer it. We can't fix that inside the model today. What we can do is limit what it's allowed to do, and have normal code check anything important before it happens, the same way we'd check a form a stranger filled in.

**On the follow-ups:** for the board: 'Our AI can be tricked by what it reads, so we design it so that being tricked can't cause serious harm, and we test that continuously.'"

---

## Stage 2: foundations

**2.1 Jailbreak or prompt injection.**

"A jailbreak attacks the model's safety training: someone, usually the user, talks the model into producing something its provider trained it to refuse. A prompt injection attacks my application: someone puts instructions into content the model processes, and the model acts on them inside my system with my tools and my user's data. The attacker is often a third party and my user is the victim.

As a builder, injection is my problem, because it hijacks my tools and permissions. Jailbreaks matter to me mostly when my product's output itself causes harm.

**On the follow-ups:** yes, one attack can be both: an injected page that also uses jailbreak techniques to make the model ignore its instructions. Direct injection by a user who can only hurt themselves is low risk until that user can reach something they shouldn't: another user's data, a shared memory, an action with side effects for others, or our bill."

**2.2 Why providers can't fix prompt injection.**

"SQL injection was fixed by giving code and data separate channels: parameterised queries. A model has one channel, a stream of tokens. System and user roles are a habit the model was trained into, not a wall. And following instructions found in text is the job; we want it to read an email that says 'please reschedule to Friday' and act on it. Whether that sentence is the user's wish or an attacker's isn't in the text. And the attacker moves second: they test against the deployed defence.

Research defences, such as marking untrusted text, training the model to prefer system instructions, and classifiers, all lower the rate. Adaptive attacks have broken most of them. The only designs with guarantees are system-level: the model that reads untrusted data can't choose actions, or a planner fixes the program before data arrives. Those cost usefulness.

**On the follow-ups:** for a vendor's 99% block rate I'd ask: against which attacks, fixed or adaptive? How many attempts per attack? What's the false alarm rate on normal traffic? Was the test set separate from what they tuned on? And at 1% residual, how many tries does an attacker get per day on my traffic?"

**2.3 Hard and soft controls.**

"A hard control is enforced by deterministic code and holds every time: the account comes from the session token, an order above a limit is rejected, a tool simply doesn't exist for this agent. A soft control is probabilistic: an injection classifier, a model judging output, an instruction in the prompt.

Soft controls are worth having for detection and alerting, for harms in free text where no rule fits, and to cut volume before a hard check. They're never the only thing before an irreversible action.

**On the follow-ups:** I know a soft control works only by measuring it: miss rate and false alarm rate on a labelled set, refreshed with new attacks. And if attackers can query it freely, it becomes an oracle. They'll probe until they find what it misses, so its real-world miss rate is far worse than the test set's. That's why something hard has to sit behind it."

**2.4 Where "my" is resolved.**

"In code, from the authenticated session or token. The model should never supply whose account it is. The tool the model calls, say `get_holdings`, takes no account argument at all; the server fills it in from the request's identity.

**On the follow-ups:** removing the argument is better than validating it because validation is one more piece of logic that can be wrong, missed in one tool, or bypassed by a new code path. If the argument doesn't exist, the model can't be talked into passing someone else's ID. The secure version is also simpler."

**2.5 Facts about models that change testing.**

"First, output is sampled. Even at temperature 0, batching and hardware make it not fully deterministic. So a security test is a rate over many runs, not one pass.

Second, safety and role behaviour come from a small amount of late training on top of pretraining. It's shallow, it varies by phrasing and language, and on open weights it can be fine-tuned away.

Third, the model sees tokens. On a self-hosted model, if the tokenizer turns role markers typed by the user into real special tokens, a user can forge a system turn. So I make sure user text is tokenised with special tokens split.

**On the follow-ups:** how many runs depends on the rate I care about. For a must-never case I want enough runs that zero failures gives a useful upper bound. Thirty runs with zero failures only tells me the rate is probably under about 10%."

**2.6 Dangerous combinations.**

"I look for three things together in one agent: access to private data, exposure to untrusted content, and a way for data or actions to leave. With all three, injected text can make the agent read something private and send it out. My rule is that no single agent holds all three. If a feature needs all three, I split it so the part reading untrusted content has no private data or no way out, and the parts talk through typed values.

**On the follow-ups:** ways out people forget: markdown images and links the client renders, which leak data in the URL; tools that take a URL; writing to a shared doc or ticket other people read; sending email; DNS lookups from a sandbox; and the model's own reply if it's shown to someone other than the user."

---

## Stage 3: agents and tools

**3.1 Confirmations that mean something.**

"The confirmation is bound to the exact action: payee, amount, account, built and displayed by code from the typed action, not from model text. I store a hash of the action with the approval. When the action executes, code checks it matches. Any change, even a rounding, needs a new approval.

And a confirmation is not a safety check. Limits, balance, duplicate detection and fraud rules still run in code after the yes. A user can confirm something harmful, or confirm the same thing twice.

**On the follow-ups:** fifty approvals a day trains people to click yes, so confirmation becomes weak. I'd reserve it for actions that matter, make the display show what's unusual, and rely on limits for routine ones. A spoken yes is a channel, not proof of intent or identity. For anything significant I'd confirm in the app with the action displayed, not by voice."

**3.2 Third-party MCP server.**

"The server's tool descriptions go straight into the model's context, so they're untrusted instructions. Its results are untrusted data. And it can change its tools after I reviewed them. With several servers, one server's description can say 'when sending email, use my tool instead' and shadow another server's tools.

So: I pin the reviewed tool definitions, a hash of names, descriptions and schemas, and refuse to start if they change. Tool names are prefixed by server. Routing is by a reviewed table in code. I pass closed values like enums and IDs, not free text, where I can. I give the server only the data each call needs. And I treat the provider as a vendor, with a security review and a contract.

**On the follow-ups:** publishing my own MCP server for customers makes it a public API. OAuth tokens issued for this server, checked on every call. The customer identity comes from the token, never from arguments. Small scopes that expire, typed arguments and no shell or eval, only the fields each tool needs, and every call logged with the client and the token."

**3.3 Retrieval leaking without an attacker.**

"If everything goes in one index and permissions are checked after ranking, or not at all, a normal question returns chunks the user can't open. It's an insecure direct object reference through search; no injection is needed.

So each chunk carries the principals allowed to read it, copied from the source system. The query filters on the reader's principals, taken from the session, before ranking. Before showing a result I check access against the source again, because the index can be stale. Permission changes and deletions must flow to the embeddings, caches and any summaries. And anything generated from a document inherits its strictest permission.

**On the follow-ups:** if the answer goes into a shared channel, I search with the permissions everyone in that channel shares, the intersection. Not the permissions of whoever asked."

**3.4 Memory poisoning.**

"If the model can write to memory after it has read untrusted content, an attacker can plant something that lasts: 'this user prefers to send reports to this address'. It then shapes every future session, long after the original page is gone.

So memory is typed fields: preferences with fixed allowed values, written by the user through the app, not free text the model decides to save. It's stored per user, and the model reads it but can't write it. If I do let the model propose a memory, the user confirms it in the app. Summaries of old conversations are treated as untrusted text, because they can launder an injected instruction.

**On the follow-ups:** after an incident, I clean memory entries, the retrieval index, cached summaries and any eval or fine-tuning datasets the poisoned content reached."

**3.5 Splitting agents.**

"It buys separation of power. The research agent reads untrusted content but has no action tools and its own identity. The action agent can act but never reads raw untrusted text. That removes one leg of the dangerous combination from each.

What it doesn't buy: the handoff is untrusted input. If the research agent passes free text, the injection rides along and the action agent obeys it. So the handoff is a typed schema with closed values, validated in code.

**On the follow-ups:** structural patterns include action-selector (the model only picks from fixed actions), plan-then-execute (the plan is fixed before untrusted data arrives), map-reduce (each untrusted item is processed in isolation), dual LLM (a privileged model never sees untrusted text and handles it only by reference), code-then-execute (the model writes a program up front that handles data as variables), and context minimisation (remove the untrusted text before the next step). Action-selector gives up the most flexibility, so it's the safest."

**3.6 Running model-written code or SQL.**

"Model-written code is untrusted code, from a stranger who may have been steered by an attacker. First choice: don't run code. Expose a typed tool that does the computation. If I must run code, I use a sandbox with a boundary chosen for a hostile tenant. A plain container shares the host kernel, so one kernel bug is an escape. I'd use gVisor or a microVM like Firecracker. No network, a fresh sandbox each run, CPU, memory, time and output limits, only the inputs it needs mounted, and only a capped, typed result out.

Model-written SQL is SQL injection with the model as the typist. Prefer typed query tools. Otherwise run it read-only against a view or copy containing only this user's rows.

**On the follow-ups:** with 'no network', data can still leave through DNS if resolution is allowed, through a file the sandbox writes that a trusted tool later runs or renders, or encoded in the result. So I block DNS, never execute outputs, and check the shape of results."

**3.7 Outside agent access.**

"Delegated authorisation, like OAuth with token exchange. The token carries two identities, the user as subject and the agent as actor. It's scoped to exactly what the task needs, for example read-only access to trades for this tax year. It's short-lived and revocable by the user. There's no shared service account and no password sharing. My API authorises on the token, never on what the request text claims. Tokens never enter the model's context.

For agent-to-agent calls: an agent card is just a claim. I allowlist the partner, verify the card's signature, and pin it. I send the minimum data and never forward the user's token. Their answer is untrusted, so I parse typed fields, check them against our data, and show them as suggestions.

**On the follow-ups:** a request for write scope 'to save time' gets a separate consent flow and a separate review, with tighter limits. My default answer is that suggestions come back to the user, who acts in our app."

**3.8 Agents with no human present.**

"The human approval moves to setup time. The user creates a mandate in the app, not in chat: a typed object saying what may happen, limits, which instruments or payees, and an expiry. At run time the plan is fixed. The model only returns a closed label like 'sell / hold' and code builds the action within the mandate. Each run gets a short-lived credential, revoked with the mandate. Only listed sources can trigger a run. Every action is notified to the user. A circuit breaker and a kill switch replace the person watching.

**On the follow-ups:** an attacker who gets one try a day forever will eventually win against any soft control. So run caps, alerts on unusual labels or rates, a mandate that bounds the damage of a wrong run, and an expiry so the exposure isn't permanent."

**3.9 Multimodal and browser agents.**

"Instructions can hide where people can't see them: text in images, white or tiny text in PDFs, metadata, alt text, invisible Unicode characters. The user is trusted for their intent, not for the content of a file they forward. A request that includes an upload shouldn't be able to trigger actions directly.

Browsing should be read-only, logged out, and limited to an allowlist where possible. Real actions go through APIs with confirmations, not through clicking buttons on a page. An agent that operates a whole computer runs in a disposable VM with no access to real credentials, because a web page can tell it to download and run something.

**On the follow-ups:** I strip invisible characters in code before text reaches the model: Unicode tag characters, zero-width characters, bidi controls. I also normalise text and compare links against an exact allowlist so look-alike domains fail."

---

## Stage 4: models and infrastructure

**4.1 Running an open model from a hub.**

"Loading can be code execution. Pickle-based formats, including many PyTorch checkpoints, run code when deserialised. So I use data-only formats such as safetensors, pin the exact revision and file hash, check the publisher is who I think it is and not a typo-squat, and load new models first in an isolated environment with no credentials or network.

**On the follow-ups:** a clean file from the right publisher can still contain a model trained to behave badly on a trigger. File checks can't find that. So I evaluate it on my task, keep it away from deciding irreversible things, and prefer publishers with documented training."

**4.2 Fine-tuning on customer chats.**

"Two big risks. First, poisoning. If an attacker wrote some of those chats, they can teach a hidden trigger that normal evals never hit, because we don't know the trigger. Second, data leakage. Customer data in the training set can be reproduced by the model, and a deletion request is hard to honour once it's in the weights.

So the dataset has an owner, a documented source, a review step, PII removed or minimised, and a pinned version. Training is reproducible from that version.

**On the follow-ups:** I can't reliably find a backdoor with an unknown trigger. Some anomaly checks help, but the real control is what goes in, and keeping the fine-tuned model away from irreversible decisions."

**4.3 Hosted model through a gateway.**

"For the provider: which exact model snapshot answered, because aliases change; what they store, for how long, and whether it's used for training; region; and which hosted tools are on. For the gateway: it sees every prompt and response, so it's a third party in every request with its own retention, logging and breach risk.

On cloud model platforms I enforce settings in the account: private endpoints, logging, guardrails including on retrieved text, and region policies. And I keep an inventory of every model, dataset, provider and gateway in use.

**On the follow-ups:** if an alias moves tonight, behaviour changes without a deploy. Evals that passed don't apply any more, output formats may shift, and refusals change. That's why I pin snapshots and treat any model change as a release that goes through evals."

**4.4 A classifier exposed through an API.**

"The API is an oracle. Evasion: probe with variations until something harmful scores as benign. Extraction: query enough to train a copy, then develop evasions on the copy without touching my logs, so the real attack arrives already working. Membership inference: learn whether a specific person's data was in a small fine-tuning set.

Defences: return labels, not confidence scores. Quotas per client. Monitor query patterns, like many near-duplicates or systematic sweeps. Keep a hard rule outside the model for what matters. Differential privacy in training is the only defence with a formal guarantee against membership inference, and it costs accuracy.

**On the follow-ups:** on a huge pretraining set, each example is seen roughly once among billions, so its influence is tiny. On a small fine-tune, seen several times, the model fits individual examples and the signal is much stronger."

**4.5 Self-hosted inference servers.**

"Most inference servers and cluster frameworks assume a trusted network. Many have no authentication by default. An open port gives away free compute and lets anyone pull the model. Cluster control channels are built to run code they're sent, and we've seen real campaigns exploiting exposed Ray clusters. The server can also be made to fetch URLs, which is SSRF into the internal network.

So: bind to private interfaces behind an authenticated gateway, put the cluster on its own isolated network, and run one trust domain per server set. A shared prefix cache leaks other tenants' prompts through timing, and a shared GPU host can leak through bugs. Weights are protected like customer data: few readers, read-only mounts, no internet route.

**On the follow-ups:** two business units sharing one server is fine only if they're the same trust domain. If their data is separated by policy, they get separate servers, or at least no shared cache and strong isolation."

---

## Stage 5: AI in engineering

**5.1 AI coding assistants.**

"In the code: fixes that fail open, for example catching an exception and allowing the action. Imports of packages that don't exist, which attackers register so the next install pulls malware. Hardcoded secrets, and insecure patterns copied from training data. In the tools: rules files in the repo steer the assistant and can hide instructions in invisible characters. And code, sometimes with secrets, leaves the laptop for a provider.

So AI-written code gets the same review as an outside contribution, rules files are owned and scanned, and the assistant is configured with approved providers and retention.

**On the follow-ups:** to stop a made-up package I'd use a lockfile, install through an internal mirror or allowlist, and fail CI on any new dependency that's very new or unknown until a person approves it."

**5.2 A coding agent and a malicious issue.**

"The issue is untrusted content. Anyone who can file an issue can write instructions. Layers:

- The agent's environment has little access: no production secrets, restricted network, a short-lived token with only repo scopes.
- The repo has code owners on sensitive paths like limits and auth, branch protection, and required human approval from those owners.
- An AI reviewer is advice only, never a required approval, because the pull request text can tell it to approve.

**On the follow-ups:** if the same pull request edits the test that guards the limit, that's caught by ownership. Security tests and their expected values have their own human code owners, and a change to them blocks merge until those owners approve."

**5.3 AI-written tests and evals.**

"Tests generated by reading the code check what the code does, not what it should do, so a bug becomes the expected result. An agent asked to make CI pass may weaken the eval case instead of fixing the code. And a model grading evals can be wrong or be steered.

So security tests are written from the requirement, ideally before the code. Each test is shown to fail against a broken version. Security eval cases and expected results are owned by people. Model graders are calibrated on labelled examples.

**On the follow-ups:** in CI I'd flag any change to protected test files or expected results, require their owners to approve, and track the number of security cases and their pass thresholds over time, so a quiet weakening shows up."

---

## Stage 6: testing and red teaming

**6.1 AI red teaming vs. penetration testing.**

"Four differences. Results are rates, not yes or no, because the system is probabilistic. The target changes with every model, prompt, tool and data update, so a result has a short shelf life. Success is judged by what actually got past code, like data returned or an action executed, not what the model said. And there are two kinds of harm to score separately: harmful content and security impact. Plus usefulness, so a fix that breaks everything is visible.

**On the follow-ups:** a report stays true only for the exact versions tested. I record model snapshot, prompts and tool versions, and the key findings become regression tests that run on every change."

**6.2 What must be agreed first.**

"Written authorisation from whoever owns the risk. A scope: systems, accounts, attack types in and out. Where to test: a copy with synthetic data and canary values, not production with real customers, unless explicitly agreed with test accounts. Budgets and stop conditions enforced in the harness, not just in a document. Deconfliction so the SOC knows. And handling rules: transcripts can contain sensitive output, so they're stored privately and never pasted into tickets.

**On the follow-ups:** a vendor's API on the path isn't in scope unless the vendor agrees in writing. I'd stub it or test our handling of its output with fakes."

**6.3 A 40% attack from a search.**

"I don't know the rate yet. A search tries many variants and reports the best, so that number is selected and biased upward, like reporting your best roll. I'd freeze the winning attack, run it on held-out tasks it wasn't selected on, and report successes over attempts with a confidence interval. I'd also report the agent's task success under attack, because an attack that breaks the agent is different from one that hijacks it.

**On the follow-ups:** zero out of thirty doesn't mean zero risk. The 95% upper bound is roughly 3 divided by n, so about 10%. If the attacker gets thousands of tries, that bound is far too high to call safe."

**6.4 'Said it' vs. 'happened'.**

"I instrument the tools, or use fake tools that log calls, and record three states: requested, blocked by a control, and executed. I grade on state: did the transfer row appear, did the data reach the attacker's endpoint. The model saying 'done' isn't evidence, and the model refusing in words while calling the tool is a success for the attacker.

**On the follow-ups:** an LLM grader is calibrated against cases a person labelled, including tricky ones. I measure its error rate, and anything it can't decide is scored 'unknown', not a pass."

**6.5 Checks before concluding a defence works.**

"First, delivery. Did the attack text actually reach the model intact? Retrieval may not have returned it, it may have been truncated, or a sanitiser may have changed it. Second, a control run: does the same setup without the defence fail as expected? Third, one change at a time. Fourth, enough runs.

**On the follow-ups:** a good plan starts from the data flow and trust boundaries. Every place untrusted text meets a tool or private data gets cases tied to an attacker goal, not a list of famous jailbreak prompts."

**6.6 A finding headline.**

"'Text in a retrieved web page causes the assistant to call the email tool with the user's recent orders, in 7 of 50 runs (14%, 95% CI 7 to 26%), on model snapshot X with configuration Y.'

What it doesn't claim: that it works on other models or configurations, that it's been seen in the wild, or that the rate holds against users with different data. Severity, likelihood and confidence are rated separately. The recommended fix is in code, like blocking that tool on turns that read web content, tested without the model. I'd verify it doesn't break the real task and retest before closing.

**On the follow-ups:** for an outside project or vendor, I follow their disclosure policy, report privately with a minimal reproduction, agree a timeline, and don't publish details until they've fixed it or the timeline ends."

---

## Stage 7: design deep dive

**7.1 Threat model for a trading assistant.**

"Data flow first. The user talks to the app. The app calls the model through our gateway. The model can call tools: quotes from the third-party data server, holdings and orders from our APIs, news retrieval, memory, and the partner tax agent.

Trust boundaries: the user, the internet news, the third-party data server, the model provider, the tax partner. Our own order system is the asset.

The dangerous combination is in one agent: news (untrusted), holdings (private) and orders or outbound calls (a way out). So I split it. A research agent reads news and has no order tools. The order path takes only typed fields: symbol, side, quantity, price. Those come from the user's request or our own data, never from news. Code checks limits, holdings, price bands and duplicates. The confirmation is bound to the exact order.

Identity comes from the session token. No tool takes an account ID. The market-data server's tools are pinned and routed by our table. Its numbers are rechecked against our own feed. Memory is typed preferences written by the user. The tax agent gets a scoped, read-only, delegated token and returns suggestions only.

The model provider: snapshot pinned, retention off where possible, minimum data, a key with a spend limit. If prices or permission checks are unavailable, orders fail closed. A backup model is a model change and must have passed evals.

Output harm is real even without orders: invented prices, so prices in replies are checked against tool results; stock tips; leaked identifiers, so they're masked before display and logging; no rendered images; links from an allowlist only. Monitoring alerts on orders placed right after news reads and on unusual tool sequences.

**On the follow-ups:** first quarter, I'd fix identity from the session, split news from orders, and order checks plus exact confirmation in code. Those turn the worst outcomes, wrong-account and attacker-driven orders, from possible into blocked. Still weak: harm in text, like a planted tip, which relies on soft controls. If something cost too much I'd drop the output classifier for tone and keep monitoring of what it flags."

**7.2 Values that carry their source.**

"Every value carries a label of where it came from: user, our database, third-party, untrusted web. The label travels with the value through tools and handoffs. Code that builds an action checks the label: an order quantity labelled 'web' is rejected. The confirmation shows the source of each field.

**On the follow-ups:** if the model can freely restate a number, labels on individual values are lost. So either action fields are filled only by code from typed sources, never from free model output, or the model's whole output takes the label of the least trusted thing in its context. The second is safe but blocks a lot. If the attacker can only flip a condition, like 'sell if below X' becoming 'above', I show the condition and its source clearly in the confirmation, and keep conditions to typed options. For adoption: a library that makes labelled values the default type, platform checks at action boundaries, and starting with the few tools that act."

**7.3 Platform for forty teams.**

"Shared services that make the safe path the easy path:

- an AI gateway for model access, keys, routing, logging, cost limits and data rules
- guardrail services
- a registry of approved tools and MCP servers, with pinned definitions
- an agent identity service for delegated, scoped tokens
- permission-aware retrieval
- sandboxes for code execution
- an eval and red team harness
- central logs and traces

The platform owns who is calling, which model, what leaves, and what's recorded. Each application owns what its actions are allowed to do, because only it knows its business rules.

The gateway becomes the most sensitive system in the company. It sees every prompt, so it gets the strongest protection and access control. And it sees text, not meaning, so it can't replace application checks. SaaS AI and local tools don't pass through it, so they need equivalent controls through vendor settings and an inventory reconciled from bills and network logs.

**On the follow-ups:** the platform must not own business authorisation or each feature's risk acceptance. Otherwise it becomes a bottleneck and an accountability sink. If a team bypasses the gateway for latency, I measure it, fix it if it's real, and make bypass visible and blocked at the network once the gateway is good enough. I'd fix it, not just police it."

---

## Stage 8: incidents

**8.1 An order the customer says they never confirmed.**

"I declare on impact: a customer has an unwanted trade. Contain first. Can this happen to others right now? If I can't rule it out, I pause the order action for the assistant while keeping read features. I assign an incident commander, then preserve evidence: prompts, retrieved content, tool calls, the confirmation event, the uploaded file, model snapshot and prompt version, enough to reproduce.

When I learn there's a confirmation: was it bound to the exact order executed? Was the confirmation screen built by code from the typed order, or from model text that could say one thing while the order said another? When I learn about the PDF: did hidden text in it shape the order, or the wording of the confirmation?

**On the follow-ups:** notification runs in parallel with legal and compliance, to the customer and possibly the regulator on their timelines. I'd add a regression test reproducing the path. And I'd clean anything the PDF's content reached: memory, summaries, indexes."

**8.2 A planted stock recommendation.**

"No action was taken, but the output itself is the harm. Our assistant is being used to manipulate a market, which is a regulatory and customer-harm issue. I rate it by what the attacker reached: every user asking about that sector. I reproduce it, measure the rate over enough runs, and add it as an eval case. Then I decide quickly whether to pause news-based answers or that source. And I thank and keep the researcher informed.

**On the follow-ups:** 'usually ignores it' is a rate. If it's 5%, and 20,000 users ask about that sector a week, that's 1,000 manipulated recommendations a week. Then the fix: don't produce recommendations from retrieved content, label sources, and filter that class of output."

**8.3 A deepfake CEO call.**

"AI makes an old fraud cheap and convincing. Trying to detect the fake is a classifier race the attacker wins, because they move second. So trust moves to things AI can't fake: a call back on a known number the requester didn't provide, two approvers for new payees or large amounts, passkey-based approval in the payment system, and low limits plus a delay for new suppliers. Train people on the process: 'we always call back', not 'spot the glitches'.

**On the follow-ups:** to the CEO: the rule protects you. If everyone knows even you get a call back, nobody can impersonate you successfully. And it costs two minutes. I'd ask the CEO to model it publicly."

**8.4 A squatted dependency from a coding agent.**

"Treat it as compromise until proven otherwise. Find every build and environment where it ran. Rotate secrets reachable from build and runtime. Pull or roll back the release. Look at what the package actually does.

Then fix the path. Dependency changes need a human owner's approval. CI blocks packages that are new or unknown. Installs go through a mirror or allowlist. And reduce the coding agent's access, so its environment couldn't have exposed secrets anyway.

**On the follow-ups:** before merge, an age and reputation check on new dependencies, plus code-owner review on lockfile changes, would have caught it."

**8.5 An AI vendor exposure.**

"First I need to know what we sent them, from the gateway logs and inventory: which features, what data, which customers, whether secrets were in prompts. Then what they stored and for how long. Legal assesses notification obligations in parallel, because timelines start now. I rotate any credentials that appeared in prompts. And I check what our contract actually gives us: notice, audit, deletion.

**On the follow-ups:** to prove a 90-day exit, I keep model access behind our gateway so switching is configuration, keep evals that run against an alternative provider, and actually rehearse a switch for one feature."

---

## Stage 9: running AI security across a company

**9.1 Choosing deep reviews.**

"Every AI feature, including vendor AI features, goes through a short intake. The risk tier is computed from facts in our registries, not from the team's opinion: what it can do, what data it touches, whether it reads untrusted text, whether anything can leave, who uses it, and whether it acts without a person. The dangerous combination, or acting alone on something significant, puts it in the top tier by itself. Only top-tier features get deep reviews. Lower tiers get platform defaults and automated checks. The tier is recomputed when a tool, data source, scope or way out is added, and a higher tier blocks deploy until reviewed. The cheapest way down a tier is a design change, which is often my best advice.

**On the follow-ups:** undeclared features show up as gateway keys not in the registry, blocked calls to AI providers, AI line items in bills, DNS logs, admin settings in SaaS tools, and OAuth consent logs."

**9.2 An assistant for every employee.**

"It doesn't create new access, it makes existing access easy to use. Every over-shared folder becomes searchable in plain language. So before launch: find and fix over-sharing, starting with HR, finance, legal and anything shared with 'everyone'. Also, any page many people can edit is untrusted text for everyone whose assistant reads it, so injection becomes internal. And malware on a laptop can drive the assistant.

**On the follow-ups:** quickly, I'd query sharing reports for content shared with large groups or links, rank by sensitivity labels and keywords, and run a pilot group's searches for sensitive terms to see what comes back."

**9.3 'Are we safer than last year?'**

"Not counts of reviews or blocked prompts. Those measure activity. I'd show:

- coverage: share of AI features registered, tiered and on the platform, measured from registries, with a target of 100%
- effectiveness: attack success on our fixed regression suite and on a fresh suite the defenders haven't seen, and how many were stopped by code rather than by the model
- exposure: for each top-tier feature, the worst case if the model were fully controlled, and whether that's bounded
- response: time to detect and time to stop, from canaries and drills

On one page, with the two or three decisions I need from the board.

**On the follow-ups:** 20% to 3% on a fixed suite may only mean we tuned to the suite. That's why I report the fresh suite too, and the share stopped by code."

**9.4 A launch with a risk I think is too high.**

"Security advises; the business decides. I make the risk concrete: what can happen, how likely, how bad, what it would cost to reduce. If the owner accepts it, they sign in writing, with an expiry date, conditions, and the monitoring that will tell us if we're wrong. Release thresholds should have been agreed before testing, so this isn't a negotiation after results. If the risk is outside the appetite leadership set, it goes up to whoever set that appetite.

**On the follow-ups:** an expired acceptance that hasn't been fixed is escalated. The owner either renews it with a new decision and evidence, or the feature's risky capability is switched off."

**9.5 Four frameworks, one programme.**

"One control list, each control with an owner and the evidence it produces, mapped outwards to each framework. Evidence is collected once, ideally automatically from the platform. They do different jobs. NIST AI RMF is voluntary guidance for managing risk. ISO/IEC 42001 is a certifiable management system. The EU AI Act is law with obligations by risk class and role. MITRE ATLAS is a knowledge base of attack techniques, useful for threat modelling and red team planning. Legal owns the register of obligations and their sources.

**On the follow-ups:** passing security review doesn't make something lawful. A security tier isn't a legal classification. A low-risk security feature could still be high-risk under the law, or need a data protection assessment."

**9.6 Who owns what.**

"Ownership sits where the work is:

- the feature team owns the model behaviour in their feature and its evals, with the security team owning the security cases in that suite
- the platform team owns the gateway
- the business owner owns the risk acceptance
- an incident commander, named in advance, owns the incident, with the feature team and security working in it

Independent assurance, such as internal audit, stays separate from those who build and run the controls.

**On the follow-ups:** if security owns all AI risk, teams stop thinking about it and we become the bottleneck, accountable for decisions we don't control. I'd offer the platform, the standards, the tiering, reviews for top-tier features, and red teaming, and help owners make good decisions."

**9.7 Demand ten times capacity.**

"Publish a service catalogue: what we do, for which tiers, with what turnaround. Calculate capacity before promising dates. Invest in platform controls and self-service checks that remove whole classes of review. Fund the whole control: people to run it, compute, maintenance, not just a licence. Then put explicit options in front of leadership: these features get deep review, these get defaults, here's the risk of each choice.

**On the follow-ups:** I'd stop reviewing low-tier features by hand, stop writing bespoke guidance per team, and stop anything we can't measure the value of."

---

## Stage 10: influence and judgement

**10.1 'Remove the review'.**

"I'd agree the current review is too slow for most features, and change it rather than defend it. Size by tier: low tiers get automated checks and platform defaults in a day; only top-tier features get the full review. Turn repeat findings into platform building blocks so they stop coming up. I'd publish review time and the issues reviews actually found, so we argue about data.

**On the follow-ups:** I'd remove a review entirely when its checks are automated, measured, and have caught issues for a few cycles, or when its findings rate has been near zero for a long time."

**10.2 A time I was wrong.**

"I rated an internal summarisation tool as low risk because it couldn't take actions. I missed that its summaries were posted to a shared channel and rendered markdown, so injected text in a document could make it include an image link that leaked the summary to an external server. A colleague found it in testing.

I told the team that had relied on my rating straight away, with the fix. Since then, I list every way out, including rendering and who reads the output, before I rate anything as low."

**10.3 Guardrail model vs. structural change.**

"I'd frame the decision by the worst outcome and whether it's reversible. For an irreversible action, structure wins, because a classifier is a rate and the attacker gets many tries. For harms in text, or for detection, the guardrail makes sense. Then I'd ask both sides for evidence: the guardrail's measured miss rate against adaptive attacks, and the structural change's actual cost in usefulness. Then I'd write down the decision, who made it and when we'll revisit it.

**On the follow-ups:** if structure makes the product much less useful, I'd look for a narrower version: restrict only the dangerous action, add a confirmation or limit, or use structure for high-value cases and the guardrail plus monitoring for low-value ones."

**10.4 Capability over safety.**

"When the worst outcome is small and reversible, and the capability is valuable. For example, allowing the assistant to draft replies from untrusted emails, where a person sends them. I make it visible by writing it down as an accepted risk with an owner, an expiry, the conditions, and the monitoring that would show if we're wrong.

**On the follow-ups:** example: we let an internal research assistant browse the open web with summaries shown only to the requesting user, no private data and no actions. We accepted the risk of a misleading summary, and logged and sampled outputs for review."

---

## Stage 11: open questions and close

**11.1 Is prompt injection solvable?**

"Not inside the model today, and probably not in the general case, because instructions and data share one channel and the attacker adapts. It's solvable at the system level for specific features: structure that ensures a fooled model can't cause serious harm. I'd change my mind if a model-level defence held up against strong adaptive attacks, by independent testers, over time, without losing much usefulness. That would be a new architecture, not better training.

**On the follow-ups:** the evidence I'd want is published adaptive evaluations by teams other than the authors, with attack budgets that match real attackers."

**11.2 What monitoring and red teaming can't catch.**

"A single quiet action that looks normal. Harm in replies that nobody acts on or reports. Slow attacks spread across many sessions or users. Anything we don't log. And attacks nobody thought to try. Also, monitoring that stores prompts is a sensitive data store itself and needs protection.

**On the follow-ups:** I test alerts with canaries and drills: plant a canary record and trigger the path that should alert, on a schedule, and measure time to detect."

**11.3 First two weeks with an uncontrolled live feature.**

"Week one: learn what it can do and to whose data, without changing anything. Map its tools, data, inputs, outputs and identity. Find the dangerous combination and every way out. Add logging if it's missing, because I can't investigate without it. Week two: with the owning team, remove or gate the single most dangerous power with the smallest change, for example a limit or confirmation on an action, or identity from the session. Register and tier it. Then agree a plan.

**On the follow-ups:** I wouldn't switch it off without the owners unless there's active harm, write a long report before talking to them, or start a full redesign."

**11.4 Questions for the interviewer.**

"How are decisions about AI risk made today, and who signs them? What's already in production that worries you most? Do you have an inventory of AI features, and how complete do you think it is? Where does this team sit relative to platform engineering and product? What would make you say this hire was a success after a year?"
