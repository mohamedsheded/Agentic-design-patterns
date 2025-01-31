# Agentic Design Patterns

This section will provide insights into various agentic design patterns that help in structuring AI-driven systems efficiently..

## 1. Reflection Pattern
This pattern allows the LLM to reflect and critique its outputs, following the next steps:

1. The LLM generates a candidate output. If you look at the diagram above, it happens inside the "Generate" box.
2. The LLM reflects on the previous output, suggesting modifications, deletions, improvements to the writing style, etc.
3. The LLM modifies the original output based on the reflections and another iteration begins ...
### steps of iteration
![image](https://github.com/user-attachments/assets/6ac054b2-c822-4cc6-b349-2196525ef092)

## 2. Tool Call Pattern


---

# Useful Utils to Construct an Agentic Pattern

## Table of Contents
- [Features](#features)
  - [1. ChatHistory](#1-chathistory)
  - [2. FixedFirstChatHistory](#2-fixedfirstchathistory)
- [Additional Functionality](#additional-functionality)
  - [3. `completions_create` Function](#3-completions_create-function)
  - [4. `build_prompt_structure` Function](#4-build_prompt_structure-function)
  - [5. `update_chat_history` Function](#5-update_chat_history-function)
  - [6. `get_fn_signature` Function](#6-get_fn_signature-function)
  - [7. `Tool` Class](#7-tool-class)
  - [8. `tool` Decorator](#8-tool-decorator)
  - [9. `validate_arguments` Function](#9-validate_arguments-function)
- [References](#references)

## Features
### 1. ChatHistory
- Stores messages in a list with a configurable maximum length.
- When the limit is reached, removes the oldest message before adding a new one.
- Useful for maintaining a rolling conversation history without exceeding memory constraints.

#### Usage
```python
from chat_history import ChatHistory

history = ChatHistory(total_length=3)
history.append("Hello")
history.append("How are you?")
history.append("I am fine.")
history.append("What about you?")
print(history)  # Output: ['How are you?', 'I am fine.', 'What about you?']
```

#### Key Differences
| Feature                | `ChatHistory` | `FixedFirstChatHistory` |
|------------------------|--------------|-------------------------|
| Maximum messages      | Yes, removes the oldest message when full | Yes, but keeps the first message fixed |
| Message removal logic | Removes the **first** message (`pop(0)`) | Removes the **second** message (`pop(1)`) |
| Use case             | General-purpose chat history storage | Keeping an important first message fixed |

### 2. FixedFirstChatHistory
- Extends `ChatHistory` but **keeps the first message fixed**.
- When the limit is reached, removes the **second** message instead of the first.
- Ideal for scenarios where an initial system prompt or introduction must persist across exchanges.

#### Usage
```python
from chat_history import FixedFirstChatHistory

history = FixedFirstChatHistory(["Welcome!"], total_length=3)
history.append("How are you?")
history.append("I am fine.")
history.append("What about you?")
print(history)  # Output: ['Welcome!', 'I am fine.', 'What about you?']
```

## Additional Functionality
### 3. `completions_create` Function
This function is responsible for sending a structured chat history to a language model and retrieving a generated response.
```python
def completions_create(client, messages: list, model: str) -> str:
    """
    Sends a request to the client's `completions.create` method to interact with the language model.
    """
    response = client.chat.completions.create(messages=messages, model=model)
    return str(response.choices[0].message.content)
```

### 4. `build_prompt_structure` Function
This function constructs a structured prompt with role-based message tagging, ensuring clear contextual understanding.
```python
def build_prompt_structure(prompt: str, role: str, tag: str = "") -> dict:
    """
    Builds a structured prompt that includes the role and content.
    """
    if tag:
        prompt = f"<{tag}>{prompt}</{tag}>"
    return {"role": role, "content": prompt}
```

### 5. `update_chat_history` Function
This function updates the chat history by appending new messages in a structured manner.
```python
def update_chat_history(history: list, msg: str, role: str):
    """
    Updates the chat history by appending the latest response.
    """
    history.append(build_prompt_structure(prompt=msg, role=role))
```

### 6. `get_fn_signature` Function
This function extracts metadata from a function, including its name, description, and parameter types, to generate its signature.
```python
def get_fn_signature(fn: Callable) -> dict:
    """
    Generates the signature for a given function.
    """
    fn_signature: dict = {
        "name": fn.__name__,
        "description": fn.__doc__,
        "parameters": {"properties": {}},
    }
    schema = {
        k: {"type": v.__name__} for k, v in fn.__annotations__.items() if k != "return"
    }
    fn_signature["parameters"]["properties"] = schema
    return fn_signature
```

### 7. `Tool` Class
This class encapsulates a callable function, storing its metadata and allowing execution with structured arguments.
```python
class Tool:
    """
    A class representing a tool that wraps a callable and its signature.
    """
    def __init__(self, name: str, fn: Callable, fn_signature: str):
        self.name = name
        self.fn = fn
        self.fn_signature = fn_signature

    def __str__(self):
        return self.fn_signature

    def run(self, **kwargs):
        """
        Executes the tool (function) with provided arguments.
        """
        return self.fn(**kwargs)
```

### 8. `tool` Decorator
This decorator function wraps another function inside a `Tool` object, automatically extracting its metadata.
```python
def tool(fn: Callable):
    """
    A decorator that wraps a function into a Tool object.
    """
    def wrapper():
        fn_signature = get_fn_signature(fn)
        return Tool(
            name=fn_signature.get("name"), fn=fn, fn_signature=json.dumps(fn_signature)
        )
    return wrapper()
```
### 9. `validate_arguments` Function
This function validates and converts the input arguments to match the expected types as per the tool's function signature. It ensures that all arguments passed to a function conform to the expected data types.
```python
def validate_arguments(tool_call: dict, tool_signature: dict) -> dict:
    """
    Validates and converts arguments in the input dictionary to match the expected types.

    Args:
        tool_call (dict): A dictionary containing the arguments passed to the tool.
        tool_signature (dict): The expected function signature and parameter types.

    Returns:
        dict: The tool call dictionary with the arguments converted to the correct types if necessary.
    """
    properties = tool_signature["parameters"]["properties"]

    type_mapping = {
        "int": int,
        "str": str,
        "bool": bool,
        "float": float,
    }

    for arg_name, arg_value in tool_call["arguments"].items():
        expected_type = properties[arg_name].get("type")

        if not isinstance(arg_value, type_mapping[expected_type]):
            tool_call["arguments"][arg_name] = type_mapping[expected_type](arg_value)

    return tool_call
```
- **Purpose**: Ensures that arguments passed to a tool conform to their expected data types.
- **How it Works**:
  - Extracts expected types from the function signature.
  - Uses a simple type-mapping dictionary to enforce correct types.
  - Converts arguments to the correct type if they don’t match.
- **Use Case**: Prevents runtime errors due to incorrect data types when calling dynamically loaded tools.


# References
- [DeepLearning.AI agents article](https://www.deeplearning.ai/the-batch/how-agents-can-improve-llm-performance/?ref=dl-staging-website.ghost.io)
- [The Neural Maze playlist](https://www.youtube.com/watch?v=os22Q7nEXPA&list=PLacQJwuclt_sK_pUPzBpfeWyiL1QOSMRQ)
- [Reference code from the Neuralmaze](https://github.com/neural-maze/agentic_patterns)

---

