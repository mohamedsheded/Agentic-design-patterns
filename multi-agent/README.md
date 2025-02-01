# Table of Contents
- [Generic Agent Class](#Agent-Class)
- [Crew Class](#Crew-Class)

# Agent Class
## 1. Workflow of Agent Class which can be multi agents working together in a crew
![WhatsApp Image 2025-02-01 at 12 59 32 PM](https://github.com/user-attachments/assets/37d6a735-584d-4ccd-b574-831ccd0755c7)
## 2. Each agent from agent class is a ReAct Agent 
![CamScanner 02-01-2025 10 39n_2](https://github.com/user-attachments/assets/82a5c5ed-d747-4e73-9608-bea03f398215)

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

# Crew Class
![WhatsApp Image 2025-02-01 at 6 17 07 PM](https://github.com/user-attachments/assets/00f9fc60-c9fc-4514-b7c1-78cb838e41e3)

## Overview

The `Crew` class represents a group of agents working together within a structured execution environment. It allows for managing multiple agents, handling dependencies between them, and executing them in a controlled manner using **topological sorting**.

### Key Features:

- Manages a collection of agents.
- Supports **dependency resolution** and **topological sorting**.
- Provides **context management** using Python's `with` statement.
- Visualizes agent dependencies as a **Directed Acyclic Graph (DAG)**.
- Executes agents in the correct dependency order.

## Class Definition

```python
class Crew:
```

The `Crew` class acts as a container for agents, ensuring they execute in an orderly fashion based on dependencies.

### Attributes:

| Attribute      | Type   | Description                                                                         |
| -------------- | ------ | ----------------------------------------------------------------------------------- |
| `current_crew` | `Crew` | A class-level variable that tracks the active Crew instance in the context manager. |
| `agents`       | `list` | A list of agents registered in this Crew.                                           |

## Methods

### `__init__()`

```python
    def __init__(self):
        self.agents = []
```

This initializes a new Crew instance with an empty list of agents.

### `__enter__()` (Context Manager Entry)

```python
    def __enter__(self):
        Crew.current_crew = self
        return self
```

When a `Crew` instance is used within a `with` statement, this method sets it as the current active crew.

### `__exit__(exc_type, exc_val, exc_tb)` (Context Manager Exit)

```python
    def __exit__(self, exc_type, exc_val, exc_tb):
        Crew.current_crew = None
```

When exiting the `with` block, it resets `current_crew` to `None`, ensuring no lingering references.

### `add_agent(agent)`

```python
    def add_agent(self, agent):
        self.agents.append(agent)
```

Adds an agent to the crew.

**Arguments:**

- `agent`: The agent object to be added.

### `register_agent(agent)` (Static Method)

```python
    @staticmethod
    def register_agent(agent):
        if Crew.current_crew is not None:
            Crew.current_crew.add_agent(agent)
```

Registers an agent with the currently active crew instance.

**Arguments:**

- `agent`: The agent to register.

### `topological_sort()`

```python
    def topological_sort(self):
```

This method sorts the agents **topologically** based on their dependencies, ensuring execution order is maintained.

#### Code Implementation:

```python
    def topological_sort(self):
        in_degree = {agent: len(agent.dependencies) for agent in self.agents}
        queue = deque([agent for agent in self.agents if in_degree[agent] == 0])
        sorted_agents = []

        while queue:
            current_agent = queue.popleft()
            sorted_agents.append(current_agent)

            for dependent in current_agent.dependents:
                in_degree[dependent] -= 1
                if in_degree[dependent] == 0:
                    queue.append(dependent)

        if len(sorted_agents) != len(self.agents):
            raise ValueError("Circular dependencies detected among agents, preventing a valid topological sort")

        return sorted_agents
```

#### Step-by-Step Explanation:

1. **Initialize Dependency Tracking:**
   - Create a dictionary `in_degree` that maps each agent to the number of dependencies it has.
   - This ensures we can track how many dependencies remain for each agent.

2. **Identify Independent Agents:**
   - Use a deque (double-ended queue) to store all agents with `in_degree` of 0 (no dependencies).
   - These agents can be processed immediately.

3. **Process Agents in Order:**
   - Pop an agent from the front of the queue and append it to `sorted_agents`.
   - For each dependent agent, reduce its `in_degree` by 1.
   - If a dependent agent's `in_degree` reaches 0, it is added to the queue for processing.

4. **Detect Cycles:**
   - If `sorted_agents` does not contain all agents, it means a circular dependency exists, raising a `ValueError`.

#### Returns:

- A list of agents sorted in topological order.

#### Raises:

- `ValueError`: If circular dependencies exist.

### `plot()` (Graph Visualization)

```python
    def plot(self):
```

Generates a **Directed Acyclic Graph (DAG)** using Graphviz to visualize agent dependencies.

**Process:**

1. Creates a **Graphviz Digraph**.
2. Adds nodes for each agent.
3. Draws edges from dependencies to dependents.
4. Returns the Graphviz object.

**Returns:**

- `Digraph`: A Graphviz graph object.

### `run()`

```python
    def run(self):
```

Executes all agents in the crew **in topologically sorted order**.

**Process:**

1. Calls `topological_sort()` to determine execution order.
2. Iterates over sorted agents.
3. Calls each agent’s `run()` method.
4. Uses `fancy_print` to log execution.
5. Prints the output in red (using `colorama.Fore.RED`).

##

