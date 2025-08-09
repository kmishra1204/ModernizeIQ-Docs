# Chapter 1: Command Line Interface (CLI)

Welcome to LLaMA-Factory! This tutorial will guide you through the exciting world of large language models, starting with the very first step: how to tell your computer what you want LLaMA-Factory to do.

## What is the Command Line Interface (CLI)?

Imagine you're driving a highly advanced car. You *could* use a complex array of buttons, switches, and touchscreens. Or, you could just tell the car exactly what to do, like "Car, drive to the nearest gas station" or "Car, activate sport mode."

The **Command Line Interface (CLI)** is like telling your computer exactly what to do using text commands. Instead of clicking buttons in a graphical window, you type instructions directly into a special program called a "terminal" or "console."

### Why use CLI for LLaMA-Factory?

LLaMA-Factory is a powerful tool for working with large language models. It can do many complex things, like:

*   **Training a new model**: Teaching a model to learn from new data.
*   **Evaluating performance**: Checking how well a model understands things.
*   **Starting an API server**: Making your model accessible to other programs.
*   **Launching a graphical web interface**: Running a user-friendly visual tool.

The CLI is the "main console" for all these operations. It acts as a central dispatcher. When you type a command, the CLI understands it and routes your request to the correct part of LLaMA-Factory to carry out the task.

Let's say you want to **train a new language model**. This is a central, powerful operation in LLaMA-Factory. How do you tell your computer to start this complex process? With the CLI, you type a simple command, and LLaMA-Factory springs into action!

## Your First Steps with CLI

To use the CLI, you first need to open your computer's terminal program.

*   **Windows**: Search for "Command Prompt" or "PowerShell."
*   **macOS**: Search for "Terminal" or "iTerm."
*   **Linux**: Look for "Terminal" in your applications menu.

Once open, you'll see a text-based window, often with a blinking cursor, waiting for your input.

### LLaMA-Factory's Commands

LLaMA-Factory provides two main ways to start its commands: `llamafactory-cli` (the full name) and a shorter, handier version, `lmf`. Both do exactly the same thing!

Let's try a very simple command: asking LLaMA-Factory for its version.

```bash
lmf version
```

**What happens?**

You'll see a friendly welcome message and the current version of LLaMA-Factory printed directly in your terminal, like this:

```
----------------------------------------------------------
| Welcome to LLaMA Factory, version 0.X.X                |
|                                                        |
| Project page: https://github.com/hiyouga/LLaMA-Factory |
----------------------------------------------------------
```

This confirms LLaMA-Factory is installed and ready to receive commands!

### Getting Help with Commands

The CLI is designed to help you. If you ever forget what commands are available or what options a specific command has, you can ask for help.

To see a list of all general commands:

```bash
lmf help
```

**What happens?**

The terminal will display a list of common LLaMA-Factory operations and how to use them:

```
----------------------------------------------------------------------
| Usage:                                                             |
|   llamafactory-cli api -h: launch an OpenAI-style API server       |
|   llamafactory-cli chat -h: launch a chat interface in CLI         |
|   llamafactory-cli eval -h: evaluate models                        |
|   llamafactory-cli export -h: merge LoRA adapters and export model |
|   llamafactory-cli train -h: train models                          |
|   llamafactory-cli webchat -h: launch a chat interface in Web UI   |
|   llamafactory-cli webui: launch LlamaBoard                        |
|   llamafactory-cli version: show version info                      |
----------------------------------------------------------------------
```

Notice the `-h` (or `--help`) option after each command. This is how you get specific help for that particular operation. For example, to see all the options for the `train` command (which is how we train models):

```bash
lmf train -h
```

**What happens?**

The terminal will now show a detailed list of all the different settings and options you can use when you run the `train` command. This is where you would learn how to specify your model, your dataset, and many other [Hyperparameter Management (HParams)](03_hyperparameter_management__hparams__.md) settings.

## How the CLI Works Under the Hood

You've seen how to use the CLI, but how does it actually work? Let's peel back the layers!

### The "Dispatcher" Analogy

Think of the LLaMA-Factory CLI (`lmf` or `llamafactory-cli`) as a smart dispatcher for a big company. When you type a command like `lmf train ...`, you're essentially calling the dispatcher and saying, "I want to train a model, and here are the details!"

The dispatcher then looks at your request ("train"), knows which department handles training (the `run_exp` function in LLaMA-Factory's training module), and sends your detailed instructions to that department.

Here's a simplified sequence of what happens:

```mermaid
sequenceDiagram
    participant User
    participant Terminal
    participant LLaMA-Factory CLI as llamafactory.cli:main
    participant Specific Function as e.g., run_exp()

    User->>Terminal: Types `lmf train --model llama2 ...`
    Terminal->>LLaMA-Factory CLI: Executes the `llamafactory.cli:main` program
    LLaMA-Factory CLI->>LLaMA-Factory CLI: Identifies "train" as the command
    LLaMA-Factory CLI->>Specific Function: Calls the `run_exp()` function
    Note over Specific Function: (with all the options like `--model llama2`)
    Specific Function-->>LLaMA-Factory CLI: Starts the training process
    LLaMA-Factory CLI-->>Terminal: Relays output/progress
    Terminal-->>User: Displays training output
```

### The Code Behind the Commands

Two key files in LLaMA-Factory make this happen:

1.  **`setup.py`**: This file tells your computer how to install and set up LLaMA-Factory. Crucially, it defines how the `lmf` and `llamafactory-cli` commands become available in your terminal.

    Let's look at a small part of `setup.py`:

    ```python
    # File: setup.py
    # ... (other code) ...

    def get_console_scripts() -> list[str]:
        # This list tells your system what commands to create
        console_scripts = ["llamafactory-cli = llamafactory.cli:main"]
        # We also create a shorter command for convenience!
        # When you type 'lmf', it also points to the same main function.
        console_scripts.append("lmf = llamafactory.cli:main")
        return console_scripts

    def main():
        setup(
            name="llamafactory",
            # ... other setup details ...
            # This is where the commands are registered!
            entry_points={"console_scripts": get_console_scripts()},
            # ...
        )

    if __name__ == "__main__":
        main()
    ```

    **Explanation:** This code snippet essentially creates a shortcut. When you type `lmf` or `llamafactory-cli` into your terminal, your system knows to run the `main` function located inside the `cli.py` file within the `llamafactory` source code. This `main` function is our "dispatcher."

2.  **`src/llamafactory/cli.py`**: This is the actual "dispatcher" code. It reads your command and decides which LLaMA-Factory function to call.

    ```python
    # File: src\llamafactory\cli.py
    import sys
    # These imports are like connecting to different "departments"
    from .train.tuner import run_exp
    from .webui.interface import run_web_ui

    # This dictionary maps each command (like "train") to the function
    # that actually performs that action.
    COMMAND_MAP = {
        "api": ..., # Calls the function to start the API server
        "chat": ..., # Calls the function for the CLI chat interface
        "eval": ..., # Calls the function for model evaluation
        "export": ..., # Calls the function to export models
        "train": run_exp, # This maps "train" to our training function!
        "webchat": ..., # Calls the function for the web chat interface
        "webui": run_web_ui, # This maps "webui" to the web interface function!
        "version": ..., # Displays LLaMA-Factory version
        "help": ..., # Displays the usage instructions
    }

    def main():
        # This line grabs the command you typed (e.g., 'train', 'webui')
        command = sys.argv.pop(1) if len(sys.argv) > 1 else "help"

        # If your command is in our map, run the corresponding function!
        if command in COMMAND_MAP:
            COMMAND_MAP[command]()
        else:
            print(f"Unknown command: {command}.\n{USAGE}")

    if __name__ == "__main__":
        main()
    ```

    **Explanation:**
    *   The `COMMAND_MAP` is the heart of the dispatcher. It's a lookup table.
    *   When you type `lmf train`, the `main` function in `cli.py` reads `train` as your command.
    *   It then finds `"train"` in the `COMMAND_MAP` and sees that it's linked to `run_exp`.
    *   Finally, it calls the `run_exp()` function, which is located in LLaMA-Factory's training module. This is the function that handles all the complex logic for [Training Workflows (Stages)](06_training_workflows__stages__.md).
    *   Similarly, if you type `lmf webui`, it calls `run_web_ui()`, which starts the [Web User Interface (WebUI)](02_web_user_interface__webui__.md). Other commands like `eval` and `chat` also dispatch to their respective functions, which you'll learn about in [Evaluation Framework](07_evaluation_framework_.md) and [Chat/Inference Engine](08_chat_inference_engine_.md).

## Conclusion

The Command Line Interface (CLI) is your direct line of communication with LLaMA-Factory. It allows you to issue powerful commands like `train`, `eval`, or `webui` directly from your terminal, acting as the central dispatcher for all LLaMA-Factory's operations. You now understand how to use these commands and have a basic grasp of how LLaMA-Factory processes them internally.

While the CLI is incredibly powerful for direct control, sometimes a more visual and interactive approach is preferred. In the next chapter, we'll explore LLaMA-Factory's graphical interface: the [Web User Interface (WebUI)](02_web_user_interface__webui__.md).

---

Built by [Codalytix.com](Codalytix.com)
