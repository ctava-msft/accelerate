# Accelerate

Accelerate is a generic application and AI workflow design for routing user requests to deterministic application logic or probabilistic AI planning. It separates intent detection, task selection, execution, resource retrieval, business logic, and response generation so that each part can evolve independently.

The design is platform-neutral. It can be used in a Teams app, web app, API, background service, or agent, with local or remote tools and any supported language model (LM) SDK or API.

## Design goals

- Use deterministic code where correctness and repeatability matter.
- Use probabilistic AI where interpretation, planning, or generation adds value.
- Keep model decisions separate from tool execution and business rules.
- Give tools narrow contracts with validated inputs and outputs.
- Apply authentication, authorization, observability, and error handling at every boundary.
- Allow a workflow to combine deterministic and probabilistic steps.

## Workflow

```mermaid
flowchart TD
    entry[Agent<br/>Entry Point] --> determineIntent[Determine<br/>Intent/Task]
    determineIntent <--> determineTask[[Determine Task:<br/>Make LM Call<br/><br/>SDK or API]]
    determineIntent --> switchTask[Switch on<br/>Task]
    switchTask --> taskType{Deterministic or<br/>Probabilistic<br/>Task?}

    taskType -- Deterministic --> deterministic[Execute<br/>Deterministic Task<br/>Step(s)]
    taskType -- Probabilistic --> planTask[[Plan Task:<br/>Make LM Call<br/><br/>SDK or API]]

    deterministic --> executeSteps
    planTask --> executeSteps["Execute<br/>Task Step(s) through<br/>[Local | Remote]<br/>Tool Calls"]

    executeSteps --> fetchDecision{Fetch<br/>Resources?}
    fetchDecision -- Yes --> fetchResources[Fetch Resources]
    fetchResources --> businessLogic[Execute Business<br/>Logic]
    businessLogic --> summarize[Summarize Output]
    fetchDecision -- No --> summarize

    summarize --> exit[Agent<br/>Exit Point]
```

The source for the agent workflow is also available in [`aks.mmd`](./aks.mmd). Additional application and channel diagrams are in [`app.mmd`](./app.mmd) and [`teams.mmd`](./teams.mmd).

## Workflow stages

1. **Entry point** receives a request from a user, application, API, or event.
2. **Intent determination** identifies the requested task. An LM may help interpret natural language, but the resulting intent must map to a supported task.
3. **Task routing** selects a known workflow and decides whether its execution is deterministic, probabilistic, or hybrid.
4. **Planning and execution** runs predefined steps or asks an LM to propose a plan, then invokes approved local or remote tools.
5. **Resource retrieval** obtains any additional data required to complete the request.
6. **Business logic** validates data and applies domain rules in code.
7. **Output summarization** converts the result into a response suitable for the calling channel.

## Choosing deterministic or probabilistic execution

The decision should be based on the nature and risk of the task, not on whether an LM is available.

### Choose deterministic execution

Use a deterministic, or static, workflow when:

- The inputs, steps, and expected outputs are known.
- The same valid input should produce the same result.
- Rules can be expressed clearly in code, configuration, or a state machine.
- The task changes data, moves money, grants access, deploys software, or performs another high-impact action.
- Regulatory, audit, safety, or contractual requirements demand explainable and reproducible behavior.
- Exact calculations, strict validation, low latency, or predictable cost are important.
- A failure must be handled through an explicit, tested recovery path.

Examples include validating a form, calculating a price, checking authorization, updating a record, provisioning an approved resource, or calling a known API sequence.

### Choose probabilistic execution

Use a probabilistic, or dynamic, workflow when:

- The request is ambiguous, conversational, or expressed in unstructured language.
- The task requires classification, extraction, summarization, drafting, or semantic comparison.
- The correct sequence of steps depends on context that cannot be fully enumerated in advance.
- More than one acceptable answer or plan may exist.
- A human can review the result, or an incorrect result has limited and reversible impact.
- The value of flexibility outweighs the added latency, cost, and variability of an LM call.

Examples include interpreting user intent, summarizing documents, drafting content, selecting relevant knowledge, or proposing a plan from an approved set of tools.

### Decision criteria

| Criterion | Prefer deterministic | Prefer probabilistic |
| --- | --- | --- |
| Input | Structured and validated | Ambiguous or unstructured |
| Output | Exact schema or single correct answer | Multiple acceptable answers |
| Process | Known and enumerable | Context-dependent or open-ended |
| Impact of error | High, irreversible, or regulated | Low, reversible, or reviewable |
| Reproducibility | Required | Helpful but not required |
| Explainability | Exact rule trace required | Rationale and evidence are sufficient |
| Latency and cost | Must be tightly bounded | Additional LM latency and cost are acceptable |
| Test strategy | Assertions against exact results | Evaluations, quality thresholds, and human review |

When the criteria conflict, choose the safer deterministic option for the action itself and use AI only to interpret, recommend, or draft.

## Prefer hybrid workflows

Many useful workflows are hybrid:

1. Use an LM to interpret the request or propose a plan.
2. Validate the intent and plan against an allowlist of supported tasks and tools.
3. Require deterministic checks for identity, authorization, policy, inputs, and preconditions.
4. Execute side effects through deterministic application code.
5. Validate tool outputs before passing them to another step.
6. Use an LM to summarize the verified result.

An LM should not directly bypass business rules or security controls. High-impact or irreversible actions should require explicit user confirmation or human approval before deterministic execution.

## Operational guidance

- Treat model output as untrusted input and validate it against a schema.
- Limit available tools and permissions to those required for the selected task.
- Set timeouts, retry policies, and idempotency controls for remote calls.
- Record the selected intent, workflow version, tool calls, outcomes, and correlation identifiers.
- Do not log secrets, credentials, or unnecessary personal data.
- Test deterministic paths with conventional unit and integration tests.
- Evaluate probabilistic paths with representative datasets, quality thresholds, adversarial inputs, and regression checks.
- Define safe failure behavior; uncertainty should trigger clarification or escalation rather than an unsupported action.

