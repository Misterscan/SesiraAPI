# Agents and workflows

Agents and workflows solve different problems.

An agent lets the model choose the next tool or final answer. Its loop is bounded by `max_steps`; every decision must match a strict runtime schema, every tool argument is validated, and every result becomes the next observation.

A workflow has an application-defined dependency graph. Deterministic function steps and model prompt steps can be mixed. Dependencies are checked before execution, cycles/missing dependencies fail explicitly, and retry counts are bounded.

Tool handlers should be small capability boundaries. Validate authorization inside the handler using the supplied context; a valid schema is not authorization. Avoid passing shell or unrestricted file/network access to agents.

Workflow `when` functions receive the complete state and return a boolean. Step handlers receive `(payload, state)`. A single dependency supplies its value directly; multiple dependencies supply an object keyed by dependency ID.
