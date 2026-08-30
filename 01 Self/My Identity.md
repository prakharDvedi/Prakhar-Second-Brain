
> Last meaningfully updated: August 2026  
> This is a living document, not a permanent definition of me.

## The short version

I'm **Prakhar Dwivedi**, a student and software engineer in the process of becoming much better at building things that actually work in the real world.

The direction I've increasingly chosen for myself is software, particularly backend systems, AI agents, developer infrastructure, and products where engineering quality actually matters.

I like **building things, breaking things, understanding why they broke, and then making them harder to break again**.

I enjoy learning broadly. Not just programming, but systems, economics, history, science, language, technology, culture, infrastructure, business, fashion, watches, aviation, perfumes, finance, and strange niches that help me understand how the world works.

A recurring theme in my life is **upward movement**. I like improving myself, but I also genuinely enjoy helping other people learn, build, earn more, and become more capable.

---

# Where I Am Right Now

I'm in the strange transition period between being a college student and becoming a professional engineer.

I'm currently working with **Pear Protocol / Pear Foundation** as an **AI & Web3 intern**, primarily around Agent Pear. 

This has become one of the most important parts of my engineering education because I'm working on production systems rather than isolated college projects.

My work has involved things like:

- AI agent reliability
    
- backend systems
    
- trading execution flows
    
- validation and safety checks
    
- observability and Langfuse
    
- Telegram systems
    
- knowledge retrieval
    
- debugging production incidents
    
- race conditions and distributed-system behavior
    
- agent response correctness
    
- Web3 infrastructure
    

I've shipped dozens of PRs and gradually moved from simply fixing assigned problems toward investigating failures, finding issues myself, questioning architecture, and thinking about what happens after code reaches production.

That change matters more to me than the raw PR count.

---

# What Kind of Engineer Am I Becoming?

I don't think I want to be a pure frontend engineer.

I also don't particularly want to become a pure ML researcher whose entire universe consists of benchmarks and papers.

The intersection I find most interesting is:

**Software Engineering × Backend Systems × Applied AI**

Especially:

- AI agents
    
- agentic workflows
    
- production AI systems
    
- backend architecture
    
- reliability
    
- observability
    
- AI security
    
- RAG and retrieval systems
    
- developer tooling
    
- distributed systems
    
- real-world failure modes
    
- infrastructure around models rather than only the models themselves
    

I like problems where the interesting question isn't:

> "Can the model do this?"

but:

> "Can we make the entire system do this reliably, repeatedly, safely, and truthfully?"

That's a much harder problem.

And therefore naturally the one humans have decided to make my problem.

---

# How I Think

## Evidence before confidence

I dislike calling something a bug just because it looks suspicious.

My instinct is increasingly:

1. reproduce it,
    
2. trace the path,
    
3. collect evidence,
    
4. attempt to disprove my own hypothesis,
    
5. distinguish facts from interpretations,
    
6. only then claim a root cause.
    

I would rather say **"I don't know yet"** than confidently invent an explanation.

---

## Understand the system, not just the line of code

When something fails, I usually want to understand:

- What happened before it?
    
- What state existed?
    
- Which service owned that state?
    
- What assumptions were made?
    
- What happened asynchronously?
    
- What happens on retry?
    
- What happens under concurrency?
    
- What happens when an external dependency behaves differently?
    
- What happens in production that doesn't happen locally?
    

I increasingly think engineering is mostly the study of **interactions between things that individually appear correct**.

---

## I like adversarial thinking

When reviewing something, I naturally ask:

- How can this fail?
    
- How can this assumption be violated?
    
- Can I construct a counterexample?
    
- Is this fixing the actual problem or hiding the symptom?
    
- Are we protecting working behavior?
    
- What is the blast radius?
    
- Is this complexity actually justified?
    

I don't want code review to be approval theatre.

---

# My Engineering Style

Some principles I've developed:

### Read before writing

Understand the existing behavior before editing it.

### Minimal diffs

Fix the actual problem without casually redesigning neighboring systems.

### Protect working behavior

A fix that creates three new regressions isn't much of a fix.

### Production evidence matters

Logs, traces, metrics and reproducible behavior beat intuition.

### Dry-run dangerous things

Especially anything touching production users, money, databases or infrastructure.

### Separate facts from hypotheses

A suspected root cause is not a confirmed root cause.

### Communicate uncertainty early

During incidents, silence while investigating can be worse than saying:

> Here's what we know, here's what we suspect, and here's what I'm checking.

### Simplicity has value

Not every problem deserves another monitoring service, abstraction, framework, queue, dashboard and ceremonial YAML file.

Sometimes checking the damn logs is enough.

---

# What I Enjoy

## Building

I like taking something that doesn't exist and making it real.

Projects are one of the main ways I learn.

I've built things across:

- AI
    
- healthcare
    
- agents
    
- web applications
    
- backend systems
    
- real-time systems
    
- Web3
    
- experimental interactive projects
    

One major project is **MedBud**, an AI-assisted healthcare bill auditing product that analyzes medical bills and supporting documents to identify questionable charges, pricing anomalies, duplicates, insurance mismatches and other issues.

I took it to the **Best Possible AI Hackathon finale at AltSpace Dharamshala**.

The important next step for projects like this is increasingly obvious to me:

**stop polishing demos forever and find real users.**

---

## Breaking

I genuinely enjoy finding edge cases.

There's something satisfying about discovering:

> "This works perfectly, assuming reality politely behaves exactly like the test fixture."

Production rarely extends that courtesy.

---

## Teaching

I like explaining things.

Teaching forces me to understand something clearly enough that I can remove the jargon without removing the substance.

I think **good education creates leverage**.

Someone who understands how things work has far more freedom than someone who only knows what buttons to press.

---

## Learning broadly

I don't want my knowledge graph to contain only programming.

I want to understand the world surrounding technology.

Things I'm interested in learning about include:

- computing
    
- AI
    
- economics
    
- companies
    
- markets
    
- finance
    
- infrastructure
    
- transportation
    
- cities
    
- aviation
    
- manufacturing
    
- geopolitics
    
- modern history
    
- science
    
- psychology
    
- language
    
- architecture
    
- design
    
- fashion
    
- watches
    
- perfumes
    
- media
    
- internet culture
    
- business models
    
- weird industries
    
- systems most people use without understanding
    

I like explanations that connect:

**history → incentives → technology → culture → current reality**

instead of dumping isolated facts into my brain.

---

# Career Direction

My broad goal isn't simply:

> get a software job.

I want to become the kind of engineer who can be dropped into an unfamiliar system, understand it quickly, find important problems, and ship reliable solutions.

I'm particularly drawn toward environments where:

- the team is small enough that ownership matters,
    
- I can interact with strong engineers,
    
- things move quickly,
    
- my work reaches real users,
    
- I can see the consequences of engineering decisions,
    
- I am trusted with increasingly difficult problems.
    

Prestige and compensation matter, obviously. Pretending otherwise is fashionable nonsense.

But I also care heavily about:

- learning rate
    
- ownership
    
- people
    
- engineering culture
    
- responsibility
    
- interesting problems
    

A prestigious logo attached to boring work isn't automatically a better outcome.

---

# Things I'm Actively Becoming Better At

## Technical

- backend engineering
    
- distributed systems
    
- system design
    
- production debugging
    
- AI agent architecture
    
- evaluation
    
- observability
    
- reliability engineering
    
- security
    
- Web3 infrastructure
    
- DevOps
    
- databases
    
- concurrency
    
- software architecture
    

## Fundamentals

I continue working on:

- DSA
    
- competitive programming
    
- computer science fundamentals
    
- understanding systems beneath frameworks
    

I don't want AI tools to make me someone who can produce code without understanding it.

---

## Professional

I'm learning to:

- communicate technical issues concisely,
    
- give status updates earlier,
    
- disagree constructively,
    
- distinguish severity from novelty,
    
- know when a fix is worth the complexity,
    
- investigate independently,
    
- ask better questions,
    
- think about users rather than only code.
    

---

# Tools and Technologies I Currently Work With

Some of my recurring stack:

- TypeScript
    
- JavaScript
    
- Next.js
    
- Node.js
    
- FastAPI
    
- PostgreSQL
    
- MongoDB
    
- Redis
    
- Docker
    
- GitHub Actions
    
- AWS
    
- Supabase
    
- LangChain
    
- LangGraph
    
- Langfuse
    
- WebSockets
    
- Hugging Face
    
- LLM APIs
    
- Git / GitHub
    

But tools are temporary.

I care more about understanding the underlying systems than becoming emotionally attached to whatever JavaScript framework humanity invents next Tuesday.

---

# My Relationship With AI

I don't see AI as something that should replace thinking.

I increasingly see it as a **cognitive multiplier**.

Used badly, it allows someone to produce enormous quantities of mediocre work.

Used well, it can help someone:

- explore faster,
    
- test assumptions,
    
- review code adversarially,
    
- research unfamiliar systems,
    
- automate repetitive work,
    
- maintain context,
    
- prototype ideas,
    
- learn faster.
    

I want to become exceptionally good at working **with AI without becoming intellectually dependent on it**.

That distinction matters.

---

# Why I'm Building This Second Brain

My brain is good at curiosity.

It is considerably less impressive at reliably storing thousands of disconnected thoughts forever.

So this vault exists to externalize memory.

I want it to become a place where I can store:

- what I learn,
    
- what I believe,
    
- what I'm questioning,
    
- projects,
    
- engineering lessons,
    
- mistakes,
    
- ideas,
    
- people,
    
- books,
    
- papers,
    
- interesting concepts,
    
- career decisions,
    
- personal reflections,
    
- long-term goals.
    

I don't want it to become a graveyard containing 4,000 beautifully tagged notes that I never read again.

The system should help me **think**, not give me another hobby involving folder structures.

---

# Things I Value

### Competence

Being genuinely useful.

### Curiosity

Understanding things beyond what is immediately necessary.

### Independence

Being capable of figuring things out without constantly waiting for instructions.

### Upward mobility

For myself and for people around me.

### Truth over appearance

I'd rather know something is broken than successfully pretend it isn't.

### Building

Ideas become much more interesting after they collide with reality.

### Teaching

Knowledge compounds when shared.

### Ownership

If something is mine, I want to care about the outcome rather than merely completing the assigned ticket.

---

# Some Contradictions In Me

I'm ambitious, but I don't want my entire life to become career optimization.

I care about money, but I also care strongly about enjoying the people and environment I work with.

I like moving quickly, but I increasingly care about correctness.

I enjoy using AI heavily, but I'm suspicious of becoming dependent on it.

I like broad curiosity, but that can turn into distraction.

I like building new things, but the harder skill is often maintaining what already exists.

I want independence, but some of my fastest growth has come from being around people better than me.

I like complexity when understanding it.

I dislike complexity when someone created it unnecessarily.

These aren't problems that need to disappear.

They're tensions I need to manage.

---

# Things I Need To Watch

## Curiosity becoming fragmentation

Being interested in everything can easily become becoming excellent at nothing.

Depth still matters.

## Shipping becoming activity

PR count, commits, projects and hours are proxies.

The actual question is:

> What became better because I worked on it?

## AI replacing struggle

Some amount of intellectual friction is necessary for learning.

If AI solves every difficult thought immediately, I lose something important.

## Overengineering

I naturally see possible failure modes.

That is useful.

It can also lead to fixing hypothetical problems that don't justify the complexity.

## Chasing opportunities

More opportunities are not automatically better.

Compounding in a good environment can beat constantly restarting somewhere new.

---

# Current Major Arcs

## 1. Become a genuinely strong software engineer

Not "student who knows frameworks."

Someone who understands systems.

## 2. Get very good at applied AI engineering

Especially agents, reliability, evaluation, observability and security.

## 3. Build products people actually use

Move beyond hackathon demos toward users, feedback and revenue.

## 4. Develop broad world knowledge

Build a connected understanding of technology, economics, culture and systems.

## 5. Become financially independent

Build useful skills, increase my earning power, and understand money rather than merely earning it.

## 6. Teach and create

Share what I understand and eventually produce things with a longer lifespan than another GitHub repository.

## 7. Write

I want writing to remain part of my life, including working on a book rather than limiting all my output to code and Discord messages.

---

# A Possible Personal Thesis

I enjoy **building systems, understanding systems, and helping people gain leverage through knowledge and technology**.

Engineering is currently the strongest expression of that.

But engineering itself probably isn't the final identity.

The deeper pattern is:

**understand → build → test → teach → improve**

---

# Reminder To Future Me

Don't confuse your current job with your identity.

Don't confuse productivity with progress.

Don't collect knowledge just to feel knowledgeable.

Don't let AI remove your ability to think independently.

Keep building things that encounter reality.

Keep finding people who are better than you.

Keep teaching what you learn.

Stay curious enough to change direction when the evidence changes.

And periodically rewrite this file.

If this document looks exactly the same five years from now, something probably went wrong.