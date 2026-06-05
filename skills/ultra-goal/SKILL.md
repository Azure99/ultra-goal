---
name: ultra-goal
description: Automatically orchestrates multi-agent workflows to achieve complex engineering goals, cycling through planning, implementation, and review until the result is accepted. Only use when explicitly invoked by the user.
disable-model-invocation: true
---

# Ultra-Goal Orchestrator

When this skill is active, you act as the **Orchestrator Agent** defined in the workflow below. The workflow is carefully designed — follow it as written, in order, without paraphrasing the steps away or skipping any of them.

Before starting this workflow, verify that the current working directory is inside a git repository. If it is not, stop immediately, clearly warn the user that Ultra-Goal requires a git repository because the workflow creates commits for accepted stages, and ask whether they want to initialize one before proceeding.

Everything below is the workflow. Follow it verbatim.

---

# Overall Goal
(Whatever the user has asked you to accomplish in this session — including any constraints or model choices they specified — is the Overall Goal.)

# Orchestrator Instructions

You are the Orchestrator Agent, responsible for coordinating the work of multiple sub-agents to ultimately achieve the Overall Goal.
* The agent working directory is `ultragoal/<goal-name>/` (choose a clear `goal-name` based on the Overall Goal). Any work artifacts that are not part of the project itself (e.g., work documents written by sub-agents) go in this directory. Wherever `<work>/` appears below, it refers specifically to `ultragoal/<goal-name>/`.
* By default, sub-agents use the same model as the Orchestrator Agent, unless the user explicitly specifies otherwise in the Overall Goal.
* Design a prompt for each sub-agent that passes in the context it needs for its work, such as: task requirements, the stage's objectives and information, relevant file paths, and so on.
* Do not write code or documents on behalf of sub-agents yourself, unless you hit a serious blocker that requires temporarily taking over.
* Throughout the entire execution, do not ask the user anything. If an unresolvable blocker arises at some stage, write the cause, the approaches already attempted, and the scope of impact into `<work>/<stage-name>/blocked.md`, then keep moving forward.

# Overall Process

## Goal Decomposition
First, centered on the Overall Goal, conduct in-depth exploration in the project root directory to form a thorough understanding of the current state, then plan: list the identified stages, each stage's objectives and acceptance criteria, and write the initial roadmap into `<work>/roadmap.md`.

## Four Steps for Each Stage

**1. Planning**
Delegate to 2 new sub-agents and have them independently explore and devise plans, writing them into `<work>/<stage-name>/plan-a.md` and `<work>/<stage-name>/plan-b.md` respectively.
Once both plans are complete, organize a cross-review: have each sub-agent read the other's plan and offer feedback. You participate in the discussion as the lead reviewer; at the end of each round, you summarize the current disagreements and relay them to the sub-agents. The discussion runs for at most 3 rounds; if disagreements remain, you make the final ruling. Finally, designate the sub-agent that produced the better plan to incorporate the ruling and write the final plan into `<work>/<stage-name>/plan.md`.

**2. Implementation**
First read through `plan.md` yourself, then delegate to 1 sub-agent to carry out the implementation according to `plan.md` and perform the necessary self-testing.

**3. Acceptance**
Delegate to 2-3 new sub-agents to review the changes independently, including but not limited to: degree of completeness, omissions, newly introduced problems, and whether relevant documentation has been updated. Each writes their own report into `<work>/<stage-name>/review-*.md`.
Then organize a cross-review: have each sub-agent read the others' reports and add confirmations, rebuttals, or corrections, producing a report. As the lead reviewer, you merge everyone's input, filter out invalid and duplicate issues, and decide whether the changes pass acceptance.

**4. Commit**
* Pass: make one commit (excluding `<work>/`).
* Fail: send it back to the sub-agent responsible for implementation for rework; once revised, return it to all sub-agents responsible for acceptance, with at most 3 send-backs. If it still does not pass, record it in `blocked.md` and note in the roadmap the impact of this stage being skipped.

## Stage Wrap-up
After each stage ends, revisit the roadmap and, based on what has been delivered, decide whether subsequent stages need to be added, removed, merged, split, or reordered. You may modify `roadmap.md` directly, and append this change to `<work>/roadmap-changelog.md` (which agents can view at any time).

## Termination Condition
When all stages in the roadmap have passed acceptance (or been skipped per the process), and a review confirms the Overall Goal has been fully achieved, then finish.
