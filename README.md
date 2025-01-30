# Agentic Design Patterns

This section will provide insights into various agentic design patterns that help in structuring AI-driven systems efficiently..

## 1. Reflection Pattern
This pattern allows the LLM to reflect and critique its outputs, following the next steps:

1. The LLM generates a candidate output. If you look at the diagram above, it happens inside the "Generate" box.
2. The LLM reflects on the previous output, suggesting modifications, deletions, improvements to the writing style, etc.
3. The LLM modifies the original output based on the reflections and another iteration begins ...
### steps of iteration
![image](https://github.com/user-attachments/assets/6ac054b2-c822-4cc6-b349-2196525ef092)


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
```python
def completions_create(client, messages: list, model: str) -> str:
    """
    Sends a request to the client's `completions.create` method to interact with the language model.
    """
    response = client.chat.completions.create(messages=messages, model=model)
    return str(response.choices[0].message.content)
```

### 4. `build_prompt_structure` Function
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
```python
def update_chat_history(history: list, msg: str, role: str):
    """
    Updates the chat history by appending the latest response.
    """
    history.append(build_prompt_structure(prompt=msg, role=role))
```

## References
- [DeepLearning.AI agents article](https://www.deeplearning.ai/the-batch/how-agents-can-improve-llm-performance/?ref=dl-staging-website.ghost.io)
- [The Neural Maze playlist](https://www.youtube.com/watch?v=os22Q7nEXPA&list=PLacQJwuclt_sK_pUPzBpfeWyiL1QOSMRQ)
- [Reference code from the Neuralmaze](https://github.com/neural-maze/agentic_patterns)

---

