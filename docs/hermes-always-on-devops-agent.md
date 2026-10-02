# 🤖 My 24/7 AI DevOps Coding Agent

## Hermes + Claude Code + Telegram on a cheap always-on thin client

Hi, I'm **Amir** 👋

I'm a DevOps engineer, and like a lot of people in DevOps, I spend part of my day solving interesting infrastructure problems... and another part doing the same kinds of small repetitive jobs again and again.

A server needs patching. A CI/CD pipeline fails. A script needs a small change. A Terraform module needs an update. A tool I built needs one more feature. Tests need to be run, branches need to be checked, and somebody still has to make sure the code that was generated actually works.

None of those jobs is necessarily difficult on its own. The annoying part is the **constant context switching**.

I kept thinking:

> **What if I had a small AI worker that was always online, understood my repositories, could call a real coding agent, verify the result, manage Git, and report back to me on my phone?**

That is what this setup became.

I use **Hermes** as the agentic AI harness and orchestrator. Hermes is connected to **OpenRouter**, where I use a low-cost **DeepSeek Flash v4** model for the orchestration layer. When real coding work is needed, Hermes delegates that work to the **Claude Code CLI**, which is my main coding agent.

The split is intentional:

> **Claude helps me think through the problem. Claude Code does the main coding. Hermes runs the workflow, performs QA, manages Git, pushes the repo, and keeps me updated through Telegram.**

This is not meant to replace me as an engineer. It is meant to remove the repetitive overhead around the work.

---

# ⚡ The idea in one picture

![My 24/7 AI DevOps architecture](https://raw.githubusercontent.com/Amiri83/hermis/main/docs/images/architecture-overview.svg)

The whole thing is surprisingly simple.

I can discuss an idea with Claude, turn that discussion into a short implementation prompt, send it to Hermes through Telegram, and let the thin client handle the rest.

Hermes can launch Claude Code, wait for it to finish, run its own verification, manage the Git operations, and tell me what happened.

My laptop does not have to stay awake.

---

# 🧠 How I divide the work between Claude, Hermes and Claude Code

This is probably the most important part of the setup.

I **do not** ask one AI agent to do everything.

When I have a new idea, bug, or feature, I normally start by talking it through with **Claude**. I explain what I want, what is currently broken, what constraints matter, and what I do *not* want changed.

Claude helps me turn that into a clear, compact prompt for Hermes.

Then I hand the task over.

### Claude: planning and shaping the task

Claude is where I usually work through the idea first.

That conversation is useful for questions like:

- What exactly should change?
- What should stay untouched?
- What edge case am I forgetting?
- What is the smallest useful test?
- What should count as PASS?
- What instructions does Hermes actually need?

I do not want a giant 5-page prompt. By the time I send the task to Hermes, the thinking should already be distilled into something actionable.

### Hermes: orchestration, QA and Git management

Hermes is my **manager layer**.

Its job is not to out-code Claude Code.

Its job is to keep the job moving.

In my workflow Hermes is responsible for things like:

- checking the current repo and branch
- launching the coding agent
- monitoring the task
- running the relevant tests itself
- verifying that the result actually matches the request
- checking Git status
- committing only after the task passes
- pushing the branch
- reporting PASS / FAIL, issues and next steps back to Telegram

That separation matters to me.

> **The agent that writes the code is not the same layer that decides the job is finished.**

### Claude Code: the main coding agent

When the work gets into the actual codebase, **Claude Code CLI** is my primary coding agent.

That is where I want the heavier coding intelligence spent:

- reading the codebase
- implementing features
- fixing bugs
- changing tests
- debugging failures
- reasoning about implementation details

Hermes stays relatively lightweight and cheap; Claude Code does the expensive coding work only when I actually need it.

---

# 📋 A real prompt from my workflow

Here is a real example.

After discussing the issue with Claude, this was the kind of short prompt I sent to Hermes:

![Claude to Hermes implementation prompt](https://raw.githubusercontent.com/Amiri83/hermis/main/docs/images/claude-to-hermes-prompt.png)

In text form:

```text
On feature/patch-all: after app restart, an analysis left RUNNING is not
marked INTERRUPTED and Analyze/Patch stay disabled. Startup recovery from
faa2ae6 isn't working on a real DB (schema v7, existing runs). Fix + test
with a DB containing a RUNNING analysis before startup.
Full suite + ruff. Commit after PASS, push. Report PASS/FAIL | issues | next.
```

I like this kind of prompt because it is short but still gives Hermes everything it needs:

**where the problem is → what the failure looks like → what must be tested → when Git operations are allowed → what I want reported back.**

---

# 🧪 Hermes does not just trust the coding agent

This is the part that made the setup useful for me rather than just interesting.

After Claude Code finishes, Hermes still has work to do.

Here is a real result from my Telegram workflow:

![Hermes QA and Git management](https://raw.githubusercontent.com/Amiri83/hermis/main/docs/images/hermes-qa-git-result.png)

In that example Hermes exercised the fallback path, brought up the required containers, reran the integration test against the real environment, cleaned things up, and then handled the Git commit.

That is exactly the kind of repetitive workflow I want the orchestrator doing for me.

I want Claude Code focused on implementation.

I want Hermes focused on:

```text
implement
   ↓
verify
   ↓
test
   ↓
inspect
   ↓
commit
   ↓
push
   ↓
report
```

---

# 💬 Telegram is my remote control

Telegram is what makes the whole setup feel different from simply running an AI coding tool on my laptop.

I can be somewhere else and send something like:

```text
Check the latest branch.

Have Claude fix the regression.

Run the targeted tests.

If everything passes, commit and push.

Report PASS/FAIL, issues, and next.
```

Then I can put my phone away.

The actual work is happening on the thin client at home.

I do not need a remote desktop session. I do not need VS Code open. I do not need my laptop to stay awake.

Telegram is just the control surface.

---

# 🖥️ Hardware I used

I deliberately did **not** build an expensive home server for this.

The machine only needs enough resources to run Linux, Hermes, Git, development tools, containers when needed, and the local Claude Code CLI. The heavy LLM computation happens remotely.

I bought a used **Dell Wyse 5070 thin client for about CAD 120**.

My base hardware is:

| Part | My setup | Approx. cost |
|---|---|---:|
| Thin client | Dell Wyse 5070 | CAD 120 |
| CPU | Intel Celeron J4105, 4 cores | included |
| RAM | 4 GB physical RAM | included |
| Internal storage | 32 GB | included |
| Wireless | Linux-compatible USB Wi-Fi adapter/module | CAD 20 |
| External storage | 256 GB external SSD | CAD 60 |
| **Approx. total** | | **CAD 200** |

For roughly **CAD 200**, I ended up with a little always-on Linux box that does exactly what I need.

And because the LLM is not running locally, the Celeron CPU is not really the bottleneck.

---

# 💾 Making 32 GB storage usable

The original 32 GB internal storage was the first real limitation.

Once you start adding Docker, containerd, package caches, repositories, Python environments, logs and DevOps tools, 32 GB disappears quickly.

The **256 GB external SSD** fixed that.

I mounted it at:

```text
/data
```

and use it as the main working storage for the box.

My layout is roughly:

```text
/data
├── docker
├── containerd
├── projects
└── swapfile
```

My normal project path points to the SSD:

```text
~/projects -> /data/projects
```

Docker data lives under:

```text
/data/docker
```

and containerd data lives under:

```text
/data/containerd
```

The rest of the SSD becomes my main storage for repositories, deployments, containers and Hermes-related working data.

The little 32 GB internal drive can mostly worry about Ubuntu.

---

# 🧠 4 GB RAM + 4 GB swap

The Wyse only has **4 GB of physical RAM**.

That is enough for the core workload most of the time, but containers, tests, package managers and agent processes can create temporary memory spikes.

So I added a **4 GB swap file on the external SSD**.

That gives me:

```text
4 GB physical RAM
+ 4 GB swap
----------------
8 GB total memory space available to the OS
```

To be clear: **4 GB of swap is not the same thing as having 8 GB of real RAM**.

It is slower.

But for an inexpensive orchestration server it gives me useful headroom and helps prevent a temporary spike from killing a process.

For this box, that trade-off works.

---

# 📡 Why I added Wi-Fi

I also bought a Linux-compatible wireless adapter for about **CAD 20**.

This was less about performance and more about practicality.

I wanted the Wyse to sit somewhere convenient and stay online 24/7. I did not want the physical location of an Ethernet jack deciding where my agent had to live.

Once I added Wi-Fi, I could place the little box where I wanted, plug it in, and mostly forget about it.

For something like this, **Linux chipset compatibility matters more than flashy Wi-Fi marketing numbers**.

---

# 🐧 Installing Hermes on Ubuntu 26.04

I run the box on **Ubuntu 26.04**.

For Hermes itself, I followed the official installation method from the [Hermes Agent website](https://hermes-agent.nousresearch.com/).

The install command is wonderfully simple:

```bash
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
```

After that, I went through the setup for the pieces I wanted to use.

In my case the important parts were:

1. **Hermes Agent** running on the thin client.
2. **OpenRouter** as the model provider for Hermes.
3. **DeepSeek Flash v4** as my low-cost orchestration model.
4. **Telegram** connected so I can talk to Hermes remotely.
5. **Claude Code CLI** installed and authenticated as the main coding agent.
6. **Git / GitHub access** configured so Hermes can manage the repository workflow.

I intentionally chose a cheaper model for Hermes because Hermes is mostly coordinating the work.

It does not need my strongest coding model burning tokens just to decide that the next step is “run these tests” or “check git status.”

That is one of the main ideas behind my setup:

> **Spend the expensive model on the coding problem. Keep orchestration lightweight.**

The official Hermes site currently documents the same terminal installer and supports messaging integrations such as Telegram.

---

# 🛠️ What lives on the thin client

Over time this little Wyse has basically become a tiny DevOps workstation.

Some of the tools I keep available on it are:

```text
Ubuntu 26.04
Hermes Agent
Claude Code CLI

Git
Docker
containerd

Terraform
AWS CLI
kubectl
Helm

Python
Node.js
```

My repositories live on the external SSD, so Hermes and Claude Code work against normal local Git repositories rather than some temporary throwaway environment.

---

# 🔄 What happens when I actually send a job?

![Task lifecycle](https://raw.githubusercontent.com/Amiri83/hermis/main/docs/images/task-lifecycle.svg)

A normal task looks like this:

1. **I discuss the idea with Claude.**
2. **Claude helps me turn it into a short prompt for Hermes.**
3. **I send it through Telegram.**
4. **Hermes launches Claude Code to implement it.**
5. **Hermes independently runs the appropriate QA/tests.**
6. **If it passes, Hermes handles the commit and push.**
7. **Telegram gives me the final PASS/FAIL, issues and next step.**

This is the workflow I was trying to build from the start:

> **I can start useful coding work without opening my laptop.**

---

# 💡 Why not just run all of this on my laptop?

Because my laptop is a laptop.

It sleeps.

I take it with me.

I reboot it.

I use it for work.

Sometimes it is not connected.

An always-on agent that disappears whenever I close my laptop is not really an always-on agent.

The Wyse solves that problem in the least glamorous way possible:

It just sits there.

Quietly.

Running Linux.

Waiting for Telegram messages.

And that is perfect.

---

# 💰 Why this cheap hardware is enough

The key is understanding where the actual computation happens.

My thin client handles:

- Hermes
- Git repositories
- Git operations
- command execution
- local tests
- containers
- DevOps CLI tools
- persistent storage
- networking
- orchestration

The LLM inference happens remotely.

So I do not need a big GPU machine sitting at home.

I need a **boring, reliable, low-power Linux box that is always available**.

That turns out to be a much cheaper problem.

---

# 🔐 A quick security note

Because this machine can execute commands and push code, I treat it like a real development host.

Things I deliberately do **not** publish include:

```text
Telegram bot tokens
OpenRouter/API keys
GitHub credentials
SSH private keys
PEM files
private repository names
internal IP addresses
service credentials
```

The more useful an agent becomes, the more important basic credential hygiene becomes too.

Do not paste your secrets into a public README or Gist.

---

# 🎯 What I ended up with

For around **CAD 200 in hardware**, I now have a small system that gives me:

- a 24/7 Ubuntu agent host
- Telegram remote control
- Hermes as the orchestration layer
- a cheap model through OpenRouter for routine agent work
- Claude for discussing and shaping tasks
- Claude Code CLI for the main implementation
- independent QA after the coding agent finishes
- automated Git management
- GitHub push/branch workflow
- persistent SSD-backed repositories
- local DevOps tooling
- a workflow that does not depend on my laptop being awake

The architecture is not complicated.

That is one of the things I like most about it.

---

# 🚀 Final thoughts

I did not build this because I wanted an AI experiment sitting in a corner of my house.

I built it because I am a DevOps engineer and I have repetitive work.

Sometimes the right answer to a repetitive task is a Bash script.

Sometimes it is Terraform.

Sometimes it is a small Python tool.

And now, sometimes I can simply describe the problem, let my agent workflow build or fix the tool, verify the result, and give me the branch when it is ready.

My setup has a very simple division of responsibility:

> **I bring the problem. Claude helps shape the solution. Claude Code writes the code. Hermes manages the job, verifies it, handles Git, and reports back to me.**

The Wyse provides the always-on home.

Telegram gives me access from anywhere.

The cloud models provide the intelligence.

A cheap little thin client became my **24/7 AI DevOps coding worker**. 🤖
