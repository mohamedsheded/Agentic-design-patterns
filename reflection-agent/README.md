# ReflectionAgent

## Overview

The `ReflectionAgent` is an AI-powered agent designed to generate responses and iteratively refine them through a reflection process. It utilizes a language model to create responses, critique them, and improve interactions dynamically.

This iterative reflection mechanism ensures that responses are not only generated but also evaluated and improved upon, making the interaction more accurate, informative, and refined.

## Features

- Generates responses based on user input.
- Reflects on generated responses to provide iterative improvements.
- Uses a predefined number of reflection cycles (`n_steps`) to enhance output quality.
- Limits chat history to optimize model context and avoid excessive computational load.
- Supports verbosity control to print intermediate steps.

## Class and Methods

### `ReflectionAgent`

This class orchestrates the entire generate-reflect process, interacting with the language model through the `Groq` client.

#### **Attributes**

- `model (str)`: Specifies the model used for generating responses (default: `deepseek-r1-distill-llama-70b`).
- `client (Groq)`: Instance of `Groq` to interface with the language model.

#### **Methods**

### `__init__(self, model: str = "deepseek-r1-distill-llama-70b")`

Initializes the agent with the specified language model and sets up the `Groq` client.

### `_request_completion(self, history: list, verbose: int = 0, log_title: str = "COMPLETION", log_color: str = "")`

Handles model completion requests.

**Arguments:**

- `history (list)`: List of messages forming the conversation history.
- `verbose (int)`: Controls verbosity (default `0` means no output).
- `log_title (str)`: Title for logging.
- `log_color (str)`: Color for terminal output.

**Returns:**

- `str`: Model-generated response.

### `generate(self, generation_history: list, verbose: int = 0) -> str`

Generates a response based on user input.

**Arguments:**

- `generation_history (list)`: List of messages forming the generation history.
- `verbose (int)`: Controls verbosity level.

**Returns:**

- `str`: The generated response.

### `reflect(self, reflection_history: list, verbose: int = 0) -> str`

Reflects on the generated response, critiquing and improving it.

**Arguments:**

- `reflection_history (list)`: List of messages forming the reflection history.
- `verbose (int)`: Controls verbosity level.

**Returns:**

- `str`: The critique or refinement of the response.

### `run(self, user_msg: str, generation_system_prompt: str = "", reflection_system_prompt: str = "", n_steps: int = 3, verbose: int = 0) -> str`

Runs the agent through iterative response refinement cycles.

**Arguments:**

- `user_msg (str)`: The initial user message.
- `generation_system_prompt (str)`: System prompt guiding response generation.
- `reflection_system_prompt (str)`: System prompt guiding reflection.
- `n_steps (int)`: Number of generate-reflect cycles (default `3`).
- `verbose (int)`: Controls verbosity level.

**Returns:**

- `str`: The final response after all refinement steps.

### Reflection Process

The `ReflectionAgent` operates in an iterative manner:

1. **Generation:** The model generates an initial response.
2. **Reflection:** The response is critiqued to identify improvements.
3. **Iteration:** The cycle repeats up to `n_steps` times, refining the response.
4. **Termination:** If `<OK>` is found in the critique, the cycle stops early.

