# Agentic Design Patterns

This section provides insights into various agentic design patterns that help structure AI-driven systems efficiently.

## Table of Contents
- [Agentic Design Patterns](#agentic-design-patterns)
  - [1. Reflection Pattern](#1-reflection-pattern)
  - [2. Tool Call Pattern](#2-tool-call-pattern)
  - [3. ReAct "Planning" Pattern](#3-react-planning-pattern)
  - [4. Multi-Agent Pattern](#4-multi-agent-pattern)
- [Useful Utils to Construct an Agentic Pattern](#useful-utils-to-construct-an-agentic-pattern)
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
    - [10. `extract_tag_content` Function](#10-extract_tag_content-function)
- [References](#references)

## 1. Reflection Pattern
[Reflection agent class](https://github.com/mohamedsheded/Agentic-design-patterns/tree/main/reflection-agent)

This pattern allows the LLM to reflect and critique its outputs, following these steps:

1. The LLM generates a candidate output.
2. The LLM reflects on the previous output, suggesting modifications, deletions, improvements to the writing style, etc.
3. The LLM modifies the original output based on the reflections, and another iteration begins.

### Steps of Iteration
![image](https://github.com/user-attachments/assets/6ac054b2-c822-4cc6-b349-2196525ef092)

## 2. Tool Call Pattern
[Tool call agent Class](https://github.com/mohamedsheded/Agentic-design-patterns/blob/main/tool_call-agent/README.md)

![image](https://github.com/user-attachments/assets/d148e782-61ab-4055-b158-391f8b81a132)

## 3. ReAct "Planning" Pattern
[Planning agent Class](https://github.com/mohamedsheded/Agentic-design-patterns/blob/main/planning-agent/README.md)

1. ![CamScanner 02-01-2025 10 39n_2](https://github.com/user-attachments/assets/bfda4c6d-04e7-44fc-893b-34cd8fb4fcf1)
2. ![CamScanner 02-01-2025 10 39n_1](https://github.com/user-attachments/assets/985f8c04-ab10-4fea-adb0-4942a88069b6)

## 4. Multi-Agent Pattern
[Multi-agent class](https://github.com/mohamedsheded/Agentic-design-patterns/blob/main/multi-agent/README.md)

![WhatsApp Image 2025-02-01 at 12 59 32 PM](https://github.com/user-attachments/assets/041c2897-b6db-45e0-8a03-f9e4e6f44fd4)

---

# Useful Utils to Construct an Agentic Pattern

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
- 
### 10. `extract_tag_content` Function
This function extracts all content enclosed by specified tags (e.g., `<thought>`, `<response>`, etc.) from a given text.
```python
import re
from dataclasses import dataclass

@dataclass
class TagContentResult:
    """
    A data class to represent the result of extracting tag content.

    Attributes:
        content (List[str]): A list of strings containing the content found between the specified tags.
        found (bool): A flag indicating whether any content was found for the given tag.
    """
    content: list[str]
    found: bool

def extract_tag_content(text: str, tag: str) -> TagContentResult:
    """
    Extracts all content enclosed by specified tags (e.g., <thought>, <response>, etc.).

    Parameters:
        text (str): The input string containing multiple potential tags.
        tag (str): The name of the tag to search for (e.g., 'thought', 'response').

    Returns:
        TagContentResult: A dataclass instance containing extracted content and a found flag.
    """
    # Build the regex pattern dynamically to find multiple occurrences of the tag
    tag_pattern = rf"<{tag}>(.*?)</{tag}>"

    # Use findall to capture all content between the specified tag
    matched_contents = re.findall(tag_pattern, text, re.DOTALL)

    # Return the dataclass instance with the result
    return TagContentResult(
        content=[content.strip() for content in matched_contents],
        found=bool(matched_contents),
    )
```
- **Purpose**: This function allows easy extraction of structured data enclosed within specific tags.
- **How it Works**:
  - Uses regex to find and extract content within a specified tag.
  - Returns a structured result using a dataclass with a `content` list and a `found` flag.
- **Use Case**: Useful for parsing structured text, such as extracting insights from AI model responses formatted with XML-like tags.

---

# References
- [DeepLearning.AI agents article](https://www.deeplearning.ai/the-batch/how-agents-can-improve-llm-performance/?ref=dl-staging-website.ghost.io)
- [The Neural Maze playlist](https://www.youtube.com/watch?v=os22Q7nEXPA&list=PLacQJwuclt_sK_pUPzBpfeWyiL1QOSMRQ)
- [Reference code from the Neuralmaze](https://github.com/neural-maze/agentic_patterns)

