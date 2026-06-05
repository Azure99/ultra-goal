# Ultra-Goal

[English](./README.md) | [中文](./README_cn.md)

> A multi-agent orchestration Skill designed for agents such as Codex and Claude Code, used to continuously advance long-running, large-scale engineering tasks in unattended mode.

## Installation

Ask the Agent to install the Ultra-Goal Skill directly:

> Install this skill: <https://github.com/Azure99/ultra-goal/tree/main/skills/ultra-goal>

You can also install it through the `skills` CLI:

```bash
npx skills add Azure99/ultra-goal
```

## Design Background

Over the past year, spec-driven workflows such as Superpowers, GSD, and OpenSpec have become widely accepted. The SDD — Spec-Driven Development — idea of “first clarifying the specification and plan, then handing implementation over to an Agent” has also effectively become the de facto standard in AI-assisted development.

However, when actually executing an SDD workflow, there are still three unavoidable pain points:

1. **The decision-making cost is still borne by the user.**
   During the spec / plan writing phase, the user still needs to choose among multiple candidate solutions, such as data structures, API style, migration paths, and so on. At every key fork in the road, the final decision still has to be made by a human.

2. **SDD is inherently ill-suited to extremely long-horizon tasks.**
   For large tasks that need to run continuously for hours or even days, it is almost impossible to exhaustively anticipate every detail before implementation actually begins. The longer the spec, the more likely it is to drift away from real-world constraints. The earlier such deviations accumulate, the higher the cost of later implementation and correction.

3. **Dynamic adjustment is expensive.**
   Once implementation is already underway, if the original spec / plan turns out to be infeasible — for example, because the wrong library was chosen, or the complexity of a module was seriously underestimated — conventional SDD tools often struggle to smoothly roll back, correct, or re-plan. The user usually has to interrupt the session, provide background context again, or even tear everything down and start over.

## Design Philosophy

Ultra-Goal organizes work as a continuously advancing phase loop. Each phase independently completes planning, implementation, acceptance, and commit. After a phase ends, the system reviews the complete roadmap and, based on what has already been delivered, dynamically adds, deletes, merges, splits, or reorders subsequent phases.

Each phase always contains four steps:

1. **Planning**
   Launch 2 sub-Agents to explore independently. Each produces an initial plan. The orchestrator Agent then acts as the final decision-maker, arbitrates between them, and forms the final implementation plan.

2. **Implementation**
   Launch 1 sub-Agent to execute according to the implementation plan and complete the necessary self-tests.

3. **Acceptance**
   Launch 2–3 sub-Agents to review independently. They cross-check, supplement, and challenge one another’s findings. Finally, the orchestrator Agent synthesizes the results and determines whether the phase passes acceptance.

4. **Commit**
   If acceptance passes, create a git commit. If acceptance fails, send the work back to the implementation Agent for rework.

This design differs from traditional SDD tools in several fundamental ways:

* **Decisions no longer depend on human intervention.**
  All trade-offs in plans, as well as disputes raised during the review phase, are evaluated and voted on by multiple sub-Agents, then arbitrated by the orchestrator Agent. They are no longer handed back to the user for a final decision.

* **The plan evolves continuously.**
  The roadmap is reviewed and revised after every phase is completed. All changes are recorded in `roadmap-changelog.md`.

* **The goal can remain appropriately abstract.**
  The user only needs to provide a relatively clear goal. They do not need to design all implementation details in advance before the task begins.

## Suitable Use Cases

**Ultra-Goal is not suitable for small tasks.**

Starting an Ultra-Goal workflow means spending a large number of tokens on planning, review, voting, and arbitration. For local changes such as “add a button” or “fix a typo,” this overhead is usually not worthwhile.

Ultra-Goal is suitable for scenarios such as:

* Large engineering tasks that need to continue for hours or even days, such as major refactors, migrations across technology stacks, or building a complete module from scratch;
* Cases where the user cannot keep watching the screen and wants the task to keep progressing in unattended mode;
* Cases where multiple projects need to move forward in parallel, and the user wants to reduce the ongoing cost of decision-making and supervision.

## Model Selection

* **The orchestrator model should be as capable as possible.**
  It performs well with the `Codex + gpt-5.5 xhigh` combination. The orchestrator Agent is responsible for overall coordination. If the model is not capable enough, the entire workflow can easily go off the rails.

* **Sub-Agents use the orchestrator model by default.**
  If you are cost-sensitive, you can explicitly specify a smaller model for sub-Agents, such as `gpt-5.4`, or even `gpt-5.4 mini`.

## Usage Recommendations

### 1. The goal should be clear, but should not contain too many implementation details

An example of an appropriate goal:

> “Migrate this Express project to Fastify while keeping the existing API behavior unchanged. All existing integration tests and E2E tests must continue to pass.”

An overly vague goal is not suitable:

> “Change the project a bit.”

However, the goal also does not need to be written as concrete implementation steps:

> “Migrate Express to Fastify. First, modify `server.ts` and replace all `app.use` calls with …”

Implementation details like these crowd out the exploration and planning space of the sub-Agents, and may instead reduce the final quality.

### 2. Provide end-to-end acceptance methods whenever possible

Relying only on unit tests is usually not enough. It is entirely possible for a sub-Agent to make all tests pass while the actual functionality can no longer run correctly.

It is recommended to specify end-to-end acceptance methods in the goal, for example:

* **Web applications**: have the acceptance Agents use `playwright-cli` to control a real browser, then perform interactive acceptance and visual acceptance on key user paths.
* **CLI tools**: execute real commands, and verify stdout, exit codes, and generated files.
* **Server-side APIs**: start the real service process, then use `curl` or an HTTP client to verify the behavior of key endpoints.

### 3. Explicit invocation is required

Ultra-Goal’s `SKILL.md` sets `disable-model-invocation: true`, so the model will not automatically apply this workflow. The user must invoke it explicitly.

This is an intentional design choice: Ultra-Goal has a high runtime cost and is not suitable for automatic triggering.
