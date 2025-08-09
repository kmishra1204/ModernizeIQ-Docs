# Tutorial: LLaMA-Factory

LLaMA-Factory is a comprehensive toolkit designed to **simplify** the *efficient fine-tuning* of over 100 large language models. It provides a structured workflow from managing training settings and preparing diverse datasets to running various fine-tuning methods and evaluating model performance. Users can interact with it via a **command-line interface** or a *graphical web interface* for training, inference, and evaluation.


**Source Repository:** [https://github.com/hiyouga/LLaMA-Factory.git](https://github.com/hiyouga/LLaMA-Factory.git)

```mermaid
flowchart TD
    A0["Hyperparameter Management (HParams)
"]
    A1["Model Loading & Adapters
"]
    A2["Data Processing Pipeline
"]
    A3["Training Workflows (Stages)
"]
    A4["Chat/Inference Engine
"]
    A5["Command Line Interface (CLI)
"]
    A6["Web User Interface (WebUI)
"]
    A7["Evaluation Framework
"]
    A5 -- "Configures" --> A0
    A6 -- "Configures" --> A0
    A0 -- "Directs loading" --> A1
    A0 -- "Directs processing" --> A2
    A0 -- "Guides workflow" --> A3
    A0 -- "Sets parameters" --> A4
    A0 -- "Defines parameters" --> A7
    A1 -- "Provides model" --> A3
    A1 -- "Provides model" --> A4
    A1 -- "Provides model" --> A7
    A2 -- "Provides data" --> A3
    A2 -- "Prepares input" --> A4
    A2 -- "Provides data" --> A7
    A3 -- "Loads/Adapts model" --> A1
    A3 -- "Loads/Processes data" --> A2
    A5 -- "Launches training" --> A3
    A6 -- "Launches training" --> A3
    A4 -- "Interacts with" --> A1
    A4 -- "Processes input with" --> A2
    A5 -- "Initiates chat" --> A4
    A6 -- "Powers interaction" --> A4
    A5 -- "Initiates evaluation" --> A7
    A5 -- "Exports model" --> A1
    A6 -- "Triggers export" --> A1
    A7 -- "Loads model" --> A1
    A7 -- "Loads datasets" --> A2
```

## Chapters

1. [Command Line Interface (CLI)
](01_command_line_interface__cli__.md)
2. [Web User Interface (WebUI)
](02_web_user_interface__webui__.md)
3. [Hyperparameter Management (HParams)
](03_hyperparameter_management__hparams__.md)
4. [Data Processing Pipeline
](04_data_processing_pipeline_.md)
5. [Model Loading & Adapters
](05_model_loading___adapters_.md)
6. [Training Workflows (Stages)
](06_training_workflows__stages__.md)
7. [Evaluation Framework
](07_evaluation_framework_.md)
8. [Chat/Inference Engine
](08_chat_inference_engine_.md)


---

Built by [Codalytix.com](Codalytix.com)
