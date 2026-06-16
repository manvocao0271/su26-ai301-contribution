# Contribution [#287]: [Adapter: Amp]

**Contribution Number:** [1]  
**Student:** [Man Cao]  
**Issue:** [https://github.com/orthogonalhq/nous-core/issues/287]  
**Status:** [Phase I] [Complete]

---

## Why I Chose This Issue

I chose this issue because I found Nous to be quite interesting to learn from as a "good first issue". Nous is an AI personal assistant that lives on a user's local machine to help with daily tasks. I am developing my career as an AI engineer so I believe tackling this issue is a perfect fit for me. I also saw the issues page being relatively new and fresh on comments. It is likely that this issue will be assigned or at least easily reviewed since there is a low number of participants. This also makes the issue less complex in addition to the "acceptance criteria" that they attached to the issue.

---

## Understanding the Issue

### Problem Description

[In your own words, what's broken or missing?]

### Expected Behavior

[What should happen?]

### Current Behavior

[What actually happens?]

### Affected Components

[Which parts of the codebase are involved?]

---

## Reproduction Process

### Environment Setup

First time trying to open with dev containers had errors. Node.js and pnpm wasn't installed and the configuration files were not correctly set. Had to change @devcontainer.json with the feature configuration settings.

### Steps to Reproduce

1. pnpm install & pnpm build, run web interface
2. review localhost:4317
3. A majority of the project is still in development (e.g. Dashboard, Org Chart, Inbox, etc.)

### Reproduction Evidence

- **Commit showing reproduction:** [https://github.com/manvocao0271/nous-core/tree/fix-issue-agentadapter-amp]
- **Screenshots/logs:** [If applicable]
- **My findings:** [Still have trouble with the CLI and the desktop app interface. Many of the features for the agent are still in production. Will talk with maintainer to get the latest production-ready code for further testing.]

---

## Solution Approach

### Analysis

[Project list of coding agents is missing a specific implemention from Amp Code. Needs an adapter file to wire SDK hook.]

### Proposed Solution

[Refer to already implemented adapters for other coding agents. Refer to Amp's own website for instructions how to set up implementation for local client. Use the current file structure for claude or codex when implementing the adapter for Amp.]

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** [The project is missing an AgentAdapter for Amp (a coding agent).]

**Match:** [This is similar to the other AgentAdapers like Jetbrains, Gemini CLI, OpenClaw, Alibaba Qwen, etc.]

**Plan:** [Step-by-step implementation plan]
1. [Create file self/subcortex/coding-agents/src/amp-adapter.ts for Amp. Refer to similar files like claude-adapter.ts and codex-adapter.ts]
2. [File should contain a Amp Agent SDK adapter that wraps Amp Agent SDK. Should run the coding agent task with functions like runAmpAgent, importAmpSdk, buildSdkHooks, extractFinalResponse]
3. [Use pnpm test to validate input/output against Zod schemas. Write to self/subcortex/coding-agents/src/__tests__/]

**Implement:** [Link to your branch/commits as you work]

**Review:** [Self-review checklist - does it follow the project's contribution guidelines?]

**Evaluate:** [How will you verify it works?]

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
