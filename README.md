# CiscoWise

CiscoWise is a Retrieval-Augmented Generation (RAG) chatbot designed to **validate Cisco device configurations against STIG (Security Technical Implementation Guide) rules** and **guide users through troubleshooting steps**. Inspired by ChatGPT’s multi-turn conversational approach, CiscoWise keeps context across turns and provides step-by-step solutions using chain-of-thought prompting—while only revealing concise, final answers to users.

## Table of Contents

- [Overview](#overview)  
- [Key Features](#key-features)  
- [Architecture](#architecture)  
- [Installation](#installation)  
- [Usage](#usage)  
  - [1. Running CiscoWise](#1-running-ciscowise)  
  - [2. Example Prompts](#2-example-prompts)  
- [Project Structure](#project-structure)  
- [Configuration & Customization](#configuration--customization)  
- [Feedback & Fine-Tuning](#feedback--fine-tuning)  
- [Roadmap](#roadmap)  
- [Disclaimer](#disclaimer)

---

## Overview

**CiscoWise** uses large language model (LLM) capabilities and retrieval-based methods to:

1. **Perform Compliance Checks**: Compare Cisco configuration text files against an Excel spreadsheet of STIG rules.  
2. **Assist with Troubleshooting**: Leverage a stored “troubleshooting steps” document (extracted from a PNG or written manually) to guide users through step-by-step network diagnostics.  
3. **Maintain Conversation Context**: Emulate ChatGPT’s style of multi-turn conversation, remembering user queries and agent responses throughout a session.

At its core, CiscoWise uses a **Chain-of-Thought** strategy internally to reason, but only surfaces the final, concise answer to the user—resulting in more reliable and insightful responses while keeping the conversation streamlined.

---

## Key Features

- **Multi-turn Conversational Flow**: The user can continue asking follow-up questions without losing context—just like ChatGPT.
- **Retrieval-Augmented Generation (RAG)**:  
  - **FAISS Index**: Creates an index from relevant documents (including troubleshooting steps) so the LLM can retrieve context for each new query.  
  - **Compliance Excel**: Reads compliance rules from an `.xls` or `.xlsx` STIG file and compares them against the Cisco config text.  
- **Chain-of-Thought Reasoning**: Encourages the LLM to reason step-by-step internally. Only the final “summarized” conclusion is displayed to the user.
- **User Feedback & Fine-Tuning**: Gathers real-time feedback from users and optionally performs on-the-fly LoRA (Low-Rank Adaptation) fine-tuning to improve performance over time.
- **Open-Ended or “ChatGPT-Style”**: The conversation continues until the user explicitly ends it (e.g., “exit,” “quit”).

---

## Architecture

1. **LLM (DeepSeek AI)**: Serves as the core text generation & reasoning engine.  
2. **SentenceTransformer (MiniLM)**: Embedding model for semantic retrieval via FAISS.  
3. **FAISS**: Vector store that indexes relevant documents (e.g., troubleshooting steps, reference guides) so that each user query can fetch the top-matching context.  
4. **Excel + Regex Compliance**: A custom module loads Excel rows as rules, each having a regex check (“present” or “absent”) against the device config file.  
5. **Chain-of-Thought Prompting**: A system instruction plus conversation history is fed to the LLM, which internally enumerates reasoning steps but surfaces only the final answer.

---

## Installation

1. **Clone the Repository**:
   ```bash
   git clone https://github.com/YourUsername/CiscoWise.git
   cd CiscoWise
   ```

2. **Install Dependencies**:
   ```bash
   pip install faiss-cpu sentence-transformers scikit-learn matplotlib seaborn transformers accelerate \
               pandas openpyxl xlrd
   ```
   - `openpyxl` is typically used for `.xlsx` files.
   - If your STIG file is an older `.xls`, ensure `xlrd` is installed.

3. **(Optional) GPU Setup**:  
   - For larger models or faster inference, configure CUDA if you have a GPU.  
   - Make sure `torch` and `transformers` are installed with GPU support.

---

## Usage

### 1. Running CiscoWise

1. **Prepare Your Files**:
   - **Device Config** (e.g., `PE1-Cisco.txt`)  
   - **STIG Excel** (e.g., `Cisco IOS XE Switch RTR Security Technical Implementation Guide-MAC-3_Sensitive.xls`)  
   - **Troubleshooting Steps** or any other knowledge docs stored in code or separate text files

2. **Run**:
   ```bash
   python cisco_automation_agent.py
   ```
   - The script will initialize the LLM, build a FAISS index, load compliance rules, and start an interactive loop.

3. **Interact**:
   - **User**: Type a query, e.g. “Generate a compliance report for PE1-Cisco.”  
   - **Agent**: Returns the compliance result.  
   - **User**: Continue with “Why did it fail the LLDP check?”  
   - **Agent**: Maintains context and explains the relevant rule.  
   - Type “exit” or “quit” when finished.

### 2. Example Prompts

- **Compliance**:  
  > “Run a compliance report for PE1-Cisco using the STIG.”  

![photo_2025-02-25_16-15-43](https://github.com/user-attachments/assets/3478006d-2a28-4cb7-bf56-a0093ec59d1b)

- **Troubleshooting**:  
  > “I have an issue reaching the app server. Can you troubleshoot?”

  ![photo_2025-02-25_16-16-19](https://github.com/user-attachments/assets/6658f8fd-b40c-4da8-94dd-5195768bb99a)


- **Multi-turn** (relying on conversation memory):  
  > User: “Generate a compliance report.”  
  > Agent: “...”  
  > User: “Show me the lines that caused the BGP check to fail.”  
  > Agent: “...”  
  > User: “How can I fix them?”  
  > Agent: “...”

![photo_2025-02-25_16-16-44](https://github.com/user-attachments/assets/d87e9ca5-dd86-405c-afdc-65cfa70d6e53)

---

## Project Structure

```
CiscoWise/
  ├─ rag_chatbot_conversational.py    # Main Python script with multi-turn logic
  ├─ requirements.txt                 # Dependencies
  ├─ PE1-Cisco.txt                    # Example device config (placeholder)
  ├─ Cisco IOS XE Switch ... .xls     # Excel STIG file
  ├─ feedback_dataset.json            # Storage for user feedback (auto-updated)
  ├─ README.md                        # This README
  └─ ...
```

- **rag_chatbot_conversational.py**:  
  Main entry point. Loads models, sets up conversation loop, handles compliance vs. troubleshooting vs. normal queries.

- **PE1-Cisco.txt**:  
  Sample Cisco config. You can replace with your own.

- **STIG file**:  
  Example Excel containing compliance checks.

---

## Configuration & Customization

- **Paths**: Update `DEVICE_CONFIG_PATH` and `COMPLIANCE_XLS_PATH` in the Python script for your actual file locations.  
- **Prompting Style**: Tweak the chain-of-thought prompts (e.g., the system instruction) to refine how “step-by-step” the model is.  
- **Memory Length**: If you have extremely long conversations, you might want to limit the number of previous turns stored.  
- **Regex Checks**: In the `check_compliance` function, expand beyond simple “present”/“absent” if you have more complex rules or multiple lines.

---

## Feedback & Fine-Tuning

CiscoWise includes a **feedback loop** that asks, “Is this correct? (y/n)” after each answer. If you type “n” and provide a corrected response:

1. Your correction gets stored in `feedback_dataset.json`.  
2. Once enough corrections accumulate, the script triggers a **LoRA fine-tune** step.  
3. This gradually adapts the model to your environment and user style.

To disable or modify this process, remove or edit the `online_fine_tune()` calls in the script.

---

## Roadmap

- **Advanced Config Parsing**: Move beyond raw regex checks into structured parsing of Cisco CLI syntax.  
- **Enhanced STIG Coverage**: Expand compliance rules to handle multiple device types or updated STIG versions.  
- **Caching**: Store embeddings and partial results to improve performance in repeated queries.  
- **UI/Frontend**: Optionally build a web or chat UI on top, so users can interact outside the terminal.

---

## Disclaimer

1. **CiscoWise** is **not** an official Cisco product and is **unaffiliated** with Cisco.  
2. The compliance checks are examples—**verify them** for your real environment.  
3. Chain-of-thought outputs can be creative or speculative; always validate critical decisions with an SME.  
4. Usage of third-party LLMs or additional libraries implies acceptance of their terms & conditions.

---

