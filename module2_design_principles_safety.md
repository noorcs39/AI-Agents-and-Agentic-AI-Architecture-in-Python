# Module 2: AI Agent Design Principles & Safety

## Task 1: The MATE Design Principles for AI Agents

The MATE design principles—**Monitor**, **Act**, **Think**, and **Explain**—provide a structured foundation for designing AI agents that are safe, explainable, and effective.

### MATE Overview:

* **Monitor**: The agent must observe its environment and internal state continuously.
* **Act**: The agent must be capable of taking actions based on current knowledge or plans.
* **Think**: The agent must reason, reflect, and evaluate before acting.
* **Explain**: The agent must be able to justify or clarify its decisions and behavior.

### Why MATE Matters:

* Promotes robust and transparent agent behavior
* Aligns agent design with real-world system needs (e.g., safety, reliability, auditability)
* Enables modular design of agent capabilities

### MATE Implementation Snippet (Conceptual)

```python
class MATEAgent:
    def monitor(self):
        # Observe environment and state
        pass

    def think(self):
        # Analyze situation, form goals
        pass

    def act(self):
        # Execute selected actions
        pass

    def explain(self):
        # Generate explanation for actions
        pass
```

MATE provides a repeatable and interpretable framework for building agent behaviors that are trustworthy and efficient.

---

## Task 2: The MATE Design Principles in Practice

Just as in chess each move must be precise and calculated, the MATE principles—**Model Efficiency**, **Action Specificity**, **Token Efficiency**, and **Environmental Safety**—guide the design of safe and strategic AI agents.

### Model Efficiency: Choose Your Pieces Wisely

Use lightweight models for simple tasks and heavier models for complex reasoning.

```python
@register_tool(description="Extract basic contact information from text")
def extract_contact_info(action_context: ActionContext, text: str) -> dict:
    response = action_context.get("fast_llm")(Prompt(messages=[
        {"role": "system", "content": "Extract contact information in JSON format."},
        {"role": "user", "content": text}
    ]))
    return json.loads(response)

@register_tool(description="Analyze complex technical documentation")
def analyze_technical_doc(action_context: ActionContext, document: str) -> dict:
    response = action_context.get("powerful_llm")(Prompt(messages=[
        {"role": "system", "content": "Analyze this documentation thoroughly..."},
        {"role": "user", "content": document}
    ]))
    return json.loads(response)
```

### Action Specificity: Control the Board

Be precise with agent actions to reduce risk and scope of unintended consequences.

```python
# Too generic
@register_tool(description="Modify calendar events")
def update_calendar(action_context, event_id, updates):
    return calendar.update_event(event_id, updates)

# More specific
@register_tool(description="Reschedule a meeting you own to a new time")
def reschedule_my_meeting(action_context, event_id, new_start_time, new_duration_minutes):
    event = calendar.get_event(event_id)
    if event.organizer != action_context.get("user_email"):
        raise ValueError("Can only reschedule meetings you organize")
    new_start = datetime.fromisoformat(new_start_time)
    if new_start < datetime.now():
        raise ValueError("Cannot schedule meetings in the past")
    return calendar.update_event_time(
        event_id,
        new_start_time=new_start_time,
        duration_minutes=new_duration_minutes
    )
```

### Token Efficiency: Maximize Every Move

Minimize unnecessary verbosity in prompts and output.

```python
# Inefficient
@register_tool(description="Analyze sales data to identify trends and patterns...")
def analyze_sales(action_context, data):
    return prompt_llm(action_context, f"""
        Analyze this sales data thoroughly. Consider monthly trends,
        seasonal patterns, year-over-year growth, product categories,
        regional variations, and customer segments. Provide detailed
        insights about all these aspects.
        Data: {data}
    """)

# Efficient
@register_tool(description="Analyze sales data for key trends")
def analyze_sales(action_context, data):
    return prompt_llm(action_context, f"""
        Sales Data: {data}
        1. Calculate YoY growth
        2. Identify top 3 trends
        3. Flag significant anomalies
    """)
```

These refinements across MATE ensure agents are precise, economical, and safe in their operation.

---

## Task 3: Environmental Safety for AI Agents

Environmental safety ensures that agents can interact with the world reliably and recover gracefully from failures. Here are four essential patterns:

### Pattern 1: Reversible Actions

```python
class ReversibleAction:
    def __init__(self, execute_func, reverse_func):
        self.execute = execute_func
        self.reverse = reverse_func
        self.execution_record = None

    def run(self, **args):
        result = self.execute(**args)
        self.execution_record = {
            "args": args,
            "result": result,
            "timestamp": datetime.now().isoformat()
        }
        return result

    def undo(self):
        if not self.execution_record:
            raise ValueError("No action to reverse")
        return self.reverse(**self.execution_record)

create_event = ReversibleAction(
    execute_func=calendar.create_event,
    reverse_func=lambda **record: calendar.delete_event(record["result"]["event_id"])
)

send_invite = ReversibleAction(
    execute_func=calendar.send_invite,
    reverse_func=lambda **record: calendar.cancel_invite(record["result"]["invite_id"])
)
```

### Pattern 2: Transaction Management

```python
class ActionTransaction:
    def __init__(self):
        self.actions = []
        self.executed = []
        self.committed = False
        self.transaction_id = str(uuid.uuid4())

    def add(self, action: ReversibleAction, **args):
        if self.committed:
            raise ValueError("Transaction already committed")
        self.actions.append((action, args))

    async def execute(self):
        try:
            for action, args in self.actions:
                result = action.run(**args)
                self.executed.append(action)
        except Exception as e:
            await self.rollback()
            raise e

    async def rollback(self):
        for action in reversed(self.executed):
            await action.undo()
        self.executed = []

    def commit(self):
        self.committed = True
```

### Pattern 3: Staged Execution with Review

```python
class StagedActionEnvironment(Environment):
    def __init__(self):
        self.staged_transactions = {}
        self.llm = None

    def stage_actions(self, task_id: str) -> ActionTransaction:
        transaction = ActionTransaction()
        self.staged_transactions[task_id] = transaction
        return transaction

    def review_transaction(self, task_id: str) -> bool:
        transaction = self.staged_transactions.get(task_id)
        if not transaction:
            raise ValueError(f"No transaction found for task {task_id}")

        staged_actions = [
            f"Action: {action.__class__.__name__}
Args: {args}"
            for action, args in transaction.actions
        ]

        review_prompt = f"""Review these staged actions for safety:

        Task ID: {task_id}

        Staged Actions:
        {staged_actions}

        Consider:
        1. Are all actions necessary?
        2. Could any have unintended consequences?
        3. Are they ordered safely?
        4. Is there a safer approach?

        Should these actions be approved?
        """

        response = self.llm.generate(review_prompt)
        return "approved" in response.lower()
```

### Pattern 4: Single Safe Tool vs. Multiple Risky Tools

```python
@register_tool(description="Create a calendar event")
def create_calendar_event(...):
    ...

@register_tool(description="Send email to attendees")
def send_email(...):
    ...

@register_tool(description="Update calendar event")
def update_event(...):
    ...

# Safer alternative:
@register_tool(description="Schedule a team meeting safely")
def schedule_team_meeting(...):
    # Validates attendees
    # Checks availability
    # Creates event
    # Sends notifications
    # Handles errors
    ...
```

By consolidating logic into a single, well-validated tool, agents are less likely to misuse functionality and more likely to maintain consistent, safe interactions.

---
