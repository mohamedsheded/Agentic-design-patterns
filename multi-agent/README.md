# Agent Class Documentation

## Overview

The `Agent` class represents an AI agent designed to work within a multi-agent system. Each agent can:
- Process a specific task with dependencies.
- Collaborate with other agents to share context.
- Utilize tools for task execution.
- Generate meaningful responses using an LLM (Large Language Model).

This class is part of a multi-agent framework and integrates with `ReactAgent` for response generation, `Crew` for agent registration, and `Tool` for extending capabilities.

---

## Features

- **Task Execution:** Each agent is assigned a task and generates an appropriate response.
- **Context Handling:** Receives and shares context with other agents.
- **Dependency Management:** Supports hierarchical task execution with dependencies and dependents.
- **Customizable LLM:** Uses an LLM of choice for response generation.
- **Integration with Crew:** Automatically registers itself in an active `Crew` context.
- **Operator Overloading for Dependencies:** Supports intuitive chaining using `>>` and `<<` operators.

---

## Class Attributes & Arguments

### **Attributes**
| Attribute               | Type               | Description |
|-------------------------|--------------------|-------------|
| `name`                 | `str`              | The name of the agent. |
| `backstory`            | `str`              | A background description for the agent. |
| `task_description`     | `str`              | The specific task assigned to the agent. |
| `task_expected_output` | `str` (optional)   | The expected output format or content of the task. Defaults to an empty string. |
| `react_agent`          | `ReactAgent`       | An instance of `ReactAgent` for response generation. |
| `dependencies`         | `list[Agent]`      | A list of agents that this agent depends on. |
| `dependents`           | `list[Agent]`      | A list of agents that depend on this agent. |
| `context`              | `str`              | Context information accumulated from other agents. |

### **Arguments (Constructor Parameters)**
| Parameter            | Type                 | Default Value | Description |
|----------------------|----------------------|---------------|-------------|
| `name`              | `str`                 | Required      | Name of the agent. |
| `backstory`         | `str`                 | Required      | Background information for the agent. |
| `task_description`  | `str`                 | Required      | Description of the task assigned to the agent. |
| `task_expected_output` | `str` (optional)  | `""`         | Expected output format of the task. |
| `tools`             | `list[Tool]` (optional) | `None`       | A list of tools available for the agent. |
| `llm`               | `str` (optional)      | `"llama-3.1-70b-versatile"` | The LLM model to use for response generation. |

---

## Operator Overloading for Dependencies

The `Agent` class supports the use of `>>` and `<<` operators for easy dependency management.

- `agent_A >> agent_B`: Makes `agent_B` dependent on `agent_A` (i.e., `agent_B` requires output from `agent_A`).
- `agent_B << agent_A`: Equivalent to `agent_A >> agent_B`.

Example:
```python
agent_A = Agent(name="Agent A", backstory="Data Collector", task_description="Collect data from source A")
agent_B = Agent(name="Agent B", backstory="Data Processor", task_description="Process data collected by Agent A")

agent_A >> agent_B  # Now agent_B depends on agent_A
```

---

## Methods

### **1. `__init__()` - Constructor**
Initializes an agent with a name, task, dependencies, and response generation system.

### **2. `__repr__()`**
Returns the agent's name as its string representation.

### **3. `add_dependency(other: Agent | list[Agent])`**
Adds an agent (or a list of agents) as a dependency.
- If a single agent is provided, it is added to `dependencies`, and a back-reference is added to `dependents` of the other agent.
- If a list of agents is provided, the same logic applies to each agent in the list.
- Raises a `TypeError` if the input is not an instance or list of `Agent`.

### **4. `add_dependent(other: Agent | list[Agent])`**
Adds an agent (or a list of agents) as a dependent.
- If a single agent is provided, it is added to `dependents`, and a back-reference is added to `dependencies` of the other agent.
- If a list of agents is provided, the same logic applies to each agent in the list.
- Raises a `TypeError` if the input is not an instance or list of `Agent`.

### **5. `receive_context(input_data: str)`**
Stores context information from other agents.
- Appends the received input data to the agent’s `context` attribute.
- Formats the received context in a structured manner.

### **6. `create_prompt()`**
Generates a structured prompt based on the agent’s task, context, and expected output.
- Encapsulates the task, expected output, and received context in XML-like tags.
- Ensures the agent follows the expected response format.

### **7. `run()`**
Executes the agent’s task by generating a response via `ReactAgent` and passing the output to its dependents.
- Calls `create_prompt()` to format the task input.
- Uses `react_agent.run(user_msg=msg)` to generate a response.
- Passes the generated response to all dependent agents via `receive_context()`.
- Returns the generated output.

---

## Example Usage

```python

# Create two agents
agent_A = Agent(
    name="DataCollector",
    backstory="Gathers financial data from online sources.",
    task_description="Scrape the latest stock market prices."
)

agent_B = Agent(
    name="DataAnalyzer",
    backstory="Analyzes financial trends based on stock data.",
    task_description="Perform statistical analysis on collected data."
)

# Establish dependency
agent_A >> agent_B  # Agent B depends on Agent A

# Run the first agent
output_A = agent_A.run()

# Run the second agent after receiving context
output_B = agent_B.run()
```

