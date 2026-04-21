# Behavior-Driven Development (BDD)

[中文版](./README.zh.md)

## Why BDD?

In traditional software development, developers tend to think in terms of "I need to implement this module" or "this class needs to handle this data." The result: code ships, features run, but none of it maps to the actual *behavior* that users, product managers, and business stakeholders care about. Tests get written, but they end up testing the wrong things. Miscommunication is constant, and rework is inevitable.

BDD (Behavior-Driven Development) was created to solve this problem: **the developer, the product team, and the business are never actually talking about the same thing.**

## Core Idea

> Software development should be driven by *what behavior the system should exhibit in different scenarios*, not by *what code needs to be written*.

BDD shifts the focus from "technical implementation" to "observable behavior."

Instead of telling the team "we need a user login module," everyone collaborates around a shared description:

> When a registered user enters the correct username and password, the system should log them in and redirect to the home page.  
> When they enter the wrong password three times, the system should lock the account and prompt them to contact support.

This isn't a technical spec or code — it's a description of *behavior* that anyone on the team can read and understand.

## BDD vs TDD

BDD is an evolution of TDD that lifts the thinking from "code level" to "behavior level":

| | TDD | BDD |
|---|---|---|
| Core question | Does this function's input/output match the test? | Does this feature behave as the business expects in real scenarios? |
| Test granularity | Unit tests | Scenarios and behaviors |
| Language | Code | Near-natural language (Given-When-Then) |
| Audience | Developers | Product, dev, and QA — everyone |

BDD still writes tests first, but those tests are expressed as behavior scenarios. Test names use `should ...` or `when ... then ...` phrasing.

## Given-When-Then

BDD describes scenarios in three parts:

- **Given** (precondition): the current state of the system
- **When** (action): what the user or system does
- **Then** (outcome): what behavior the system should exhibit

```gherkin
Given the user is registered and the account is active
When the user enters the wrong password three times
Then the account is locked and a contact-support message is shown
```

Mapped to code:

```typescript
describe('User Login', () => {
  describe('when the user enters the wrong password three times', () => {
    it('should lock the account and return a contact-support message', async () => {
      ...
    });
  });
});
```

## Why BDD Matters

1. **Shared language**: Product, dev, and QA use the same vocabulary — no more "I thought you meant X."
2. **Living documentation**: Scenarios are both test cases and always-up-to-date requirements docs.
3. **Earlier misalignment detection**: Describing behavior upfront surfaces disagreements between product, dev, and QA before any code is written.

## Why BDD Is Better Than TDD When Using Coding Agents

TDD was designed for human developers who hold business intent in their heads. Coding agents don't have that context — and that asymmetry makes BDD strictly superior in an AI-assisted workflow.

**1. Agents implement to spec, not to intent**

When you hand an agent a TDD unit test, it will write the minimal code to make that specific test pass — nothing more. If the test was written around implementation details rather than behavior, the agent produces code that satisfies the test but misses the real requirement. BDD scenarios written at the contract boundary leave no room for this drift.

**2. Agents naturally over-engineer without behavioral constraints**

Given an open prompt, agents add options, abstractions, and edge-case handling you didn't ask for. A failing BDD scenario acts as a hard stop: "implement only what makes *this observable behavior* pass." TDD unit tests don't constrain scope the same way.

**3. TDD tests written by agents are self-fulfilling**

If you ask an agent to write tests *after* implementing, it will write tests that match what it built — not what was required. This is the classic TDD trap, and agents fall into it reliably. BDD scenarios should be specified by humans (or agreed upon) before the agent writes a single line of implementation.

**4. Scenarios are human-reviewable without reading code**

`describe('when user enters wrong password 3 times') → it('should lock the account')` is verifiable by anyone. A mock-heavy unit test written by an agent requires understanding the implementation to judge whether it's testing the right thing. BDD keeps the review surface human-scale.

**5. Contract-level tests survive agent refactoring**

Agents refactor aggressively. Unit tests tied to internal methods break every time an agent restructures internals — even when behavior is unchanged. BDD tests at public contract boundaries stay green through any internal restructuring, giving agents the freedom to refactor without test churn.

**In short:** TDD requires the developer to hold intent in their head and transfer it faithfully into tests. With agents, that transfer is unreliable. BDD externalizes intent into human-readable scenarios first — making the agent's job unambiguous and the output verifiable.

## Contents

- [`SKILL.md`](./SKILL.md) — BDD development workflow (Red-Green-Refactor, contract-first, scenario-driven)
- [`testing-anti-patterns.md`](./testing-anti-patterns.md) — Testing anti-patterns reference (7 common mistakes with Gate Functions)
