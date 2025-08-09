# Chapter 2: Web User Interface (WebUI)

In the [Command Line Interface (CLI)](01_command_line_interface__cli__.md) chapter, you learned how to talk to LLaMA-Factory by typing text commands into a terminal. It's like sending text messages to your computer to get things done. While powerful, sometimes seeing is believing, and clicking buttons can be much simpler!

This is where the **Web User Interface (WebUI)** comes in.

## What is the Web User Interface (WebUI)?

Imagine LLaMA-Factory is a super-powerful car. The CLI is like driving it by typing precise instructions into a black box. The WebUI, on the other hand, is your car's **dashboard**. It's a visual, easy-to-use control panel with gauges, buttons, and switches that lets you manage everything without remembering complex commands.

The WebUI provides a **graphical dashboard** for LLaMA-Factory. It runs right in your everyday web browser (like Chrome, Firefox, or Edge), turning complex configurations into simple clicks and selections.

### Why use WebUI for LLaMA-Factory?

The WebUI makes working with LLaMA-Factory much more intuitive, especially for beginners or when you want a quick overview. It solves several problems:

*   **No Code Needed**: You don't need to write a single line of code to configure your models, datasets, or [Hyperparameter Management (HParams)](03_hyperparameter_management__hparams__.md) settings. Everything is done through dropdowns, text fields, and checkboxes.
*   **Visual Feedback**: It can visualize the training progress in real-time, showing you graphs and numbers as your model learns.
*   **Easy Interaction**: You can directly chat with your loaded models, evaluate their performance, or export them, all from a friendly interface.

Let's say you want to **train a new language model**. Instead of typing a long `lmf train` command with many options, the WebUI lets you:
1.  **Select your model** from a dropdown list (e.g., "Llama-2-7b").
2.  **Choose your dataset** from another list (e.g., "alpaca_zh").
3.  **Adjust training parameters** using sliders and input boxes (e.g., "learning rate", "batch size").
4.  **Click a "Start Training" button**.

This visual approach simplifies the entire process.

## Your First Steps with WebUI

Just like with the CLI, you'll start by opening your terminal (Command Prompt, PowerShell, Terminal, etc.).

To launch the LLaMA-Factory WebUI, you use a very simple CLI command:

```bash
lmf webui
```

**What happens?**

After you run this command, LLaMA-Factory will start setting up the WebUI. You'll see some messages in your terminal, and eventually, it will print a URL, something like this:

```
Visit http://ip:port for Web UI, e.g., http://127.0.0.1:7860
Running on local URL: http://127.0.0.1:7860
```

This `http://127.0.0.1:7860` is the address for your WebUI!

**What to do next?**

1.  **Copy this URL** (e.g., `http://127.0.0.1:7860`).
2.  **Paste it into your web browser's address bar** and press Enter.

Voila! You will now see the LLaMA-Factory WebUI dashboard appear in your browser. It will have different tabs for "Train", "Evaluate & Predict", "Chat", and "Export", allowing you to interact with LLaMA-Factory visually.

## How the WebUI Works Under the Hood

You've seen how easy it is to start the WebUI, but how does LLaMA-Factory make this visual interface appear? Let's take a peek behind the curtain.

### The "Dashboard Builder" Analogy

Think of the `lmf webui` command as telling LLaMA-Factory, "Please set up my car's dashboard and make it available for me to see in my browser!"

Here's a simplified sequence of what happens when you type `lmf webui`:

```mermaid
sequenceDiagram
    participant User
    participant Terminal
    participant LLaMA-Factory CLI as llamafactory.cli:main
    participant WebUI Launcher as src/webui.py:main
    participant Gradio as gradio.Blocks().launch()
    participant Web Browser

    User->>Terminal: Types `lmf webui`
    Terminal->>LLaMA-Factory CLI: Executes `llamafactory.cli:main`
    LLaMA-Factory CLI->>WebUI Launcher: Calls `run_web_ui()` (from src/webui.py)
    WebUI Launcher->>Gradio: `create_ui().queue().launch()`
    Gradio->>Web Browser: Serves the interactive WebUI page
    Web Browser-->>User: Displays the LLaMA-Factory dashboard
    User->>Web Browser: Interacts with UI (e.g., clicks "Train")
    Web Browser->>Gradio: Sends interaction data
    Gradio->>WebUI Launcher: Processes interaction
    WebUI Launcher->>LLaMA-Factory CLI: (Can internally trigger commands like `train`)
```

### The Code Behind the WebUI

As we saw in the previous chapter, `src/llamafactory/cli.py` acts as the dispatcher.

1.  **`src/llamafactory/cli.py`**: The `COMMAND_MAP` tells LLaMA-Factory what to do when you type `webui`.

    ```python
    # File: src\llamafactory\cli.py
    # ...
    from .webui.interface import run_web_ui # Import the WebUI function

    COMMAND_MAP = {
        # ... other commands ...
        "webui": run_web_ui, # This maps "webui" to the run_web_ui function!
        # ...
    }

    def main():
        command = sys.argv.pop(1) if len(sys.argv) > 1 else "help"
        if command in COMMAND_MAP:
            COMMAND_MAP[command]() # Calls run_web_ui() when you type 'lmf webui'
        # ...
    ```

    **Explanation:** When you type `lmf webui`, the `main` function in `cli.py` looks up "webui" in the `COMMAND_MAP`. It finds that "webui" is linked to the `run_web_ui` function. So, it simply calls `run_web_ui()`.

2.  **`src/webui.py`**: This file is often the main entry point for the WebUI when launched directly. Its `main` function is essentially calling `run_web_ui` from the interface module.

    ```python
    # File: src\webui.py
    # ...
    from llamafactory.webui.interface import create_ui # Import the UI creator

    def main():
        # ... setup variables for server name, sharing ...
        print("Visit http://ip:port for Web UI, e.g., http://127.0.0.1:7860")
        create_ui().queue().launch(share=False, server_name="0.0.0.0", inbrowser=True)

    if __name__ == "__main__":
        main()
    ```

    **Explanation:** This `main` function is what gets executed if you were to run `python src/webui.py` directly. It calls `create_ui()` to build the interface and then `launch()` to start the web server that hosts it.

3.  **`src/llamafactory/webui/interface.py`**: This is where the actual graphical interface is defined using a Python library called `Gradio`. Gradio is a popular tool for building simple web interfaces for machine learning models.

    ```python
    # File: src\llamafactory\webui\interface.py
    # ...
    if is_gradio_available(): # Checks if Gradio is installed
        import gradio as gr # Imports the Gradio library

    def create_ui(demo_mode: bool = False) -> "gr.Blocks":
        # ... other setup code ...
        with gr.Blocks(title=f"LLaMA Factory", css=CSS) as demo:
            # Here, different parts of the UI are created:
            # - create_top() creates dropdowns for language, etc.
            # - create_train_tab() builds the training interface
            # - create_eval_tab() for evaluation
            # - create_infer_tab() for chat
            # - create_export_tab() for exporting models
            pass # Simplified: Actual code builds tabs and components

        return demo # Returns the complete Gradio interface

    def run_web_ui() -> None:
        # ... setup variables for server name, sharing ...
        print("Visit http://ip:port for Web UI, e.g., http://127.0.0.1:7860")
        # This is the line that actually creates and launches the web interface!
        create_ui().queue().launch(share=False, server_name="0.0.0.0", inbrowser=True)
    ```

    **Explanation:**
    *   The `create_ui()` function uses `gradio` to build the entire visual layout of the WebUI, including all its tabs and components. Each tab (like "Train", "Evaluate", "Chat") is built by other functions (e.g., `create_train_tab`).
    *   The `run_web_ui()` function (which is called by the CLI dispatcher) then calls `create_ui()` to get the interface and uses `.queue().launch()` to start a web server. This server makes the graphical interface available in your browser at the specified address (like `http://127.0.0.1:7860`).

When you interact with the WebUI (e.g., select a model, click "Start Training"), the Gradio interface translates your clicks and selections into specific actions. These actions often internally trigger the same kind of logic that CLI commands like `lmf train` would, but with all the parameters automatically handled by the WebUI based on your inputs. This means the WebUI also leverages other core LLaMA-Factory components like the [Data Processing Pipeline](04_data_processing_pipeline_.md), [Model Loading & Adapters](05_model_loading___adapters_.md), [Training Workflows (Stages)](06_training_workflows__stages__.md), [Evaluation Framework](07_evaluation_framework_.md), and [Chat/Inference Engine](08_chat_inference_engine_.md).

## Conclusion

The Web User Interface (WebUI) provides a friendly, visual way to interact with LLaMA-Factory. It lets you configure and manage complex tasks like training models, evaluating performance, and chatting with them, all without writing code. You simply launch it with `lmf webui` from your terminal, and then control LLaMA-Factory from your web browser. This abstraction makes LLaMA-Factory much more accessible and user-friendly for many common operations.

In the next chapter, we'll dive deeper into how you manage the many settings and options (called "Hyperparameters") that control LLaMA-Factory's behavior, whether you're using the CLI or the WebUI: [Hyperparameter Management (HParams)](03_hyperparameter_management__hparams__.md).

---

Built by [Codalytix.com](Codalytix.com)
