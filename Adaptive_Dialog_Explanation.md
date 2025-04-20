
# 🧠 Understanding Adaptive Dialog Schema for Copilot Studio

## What Language is This?

The PasswordHelperAgent.dialog is the YAML code for the agent, written in the **Adaptive Dialog Schema** which is used in **Microsoft Copilot Studio**. This schema is based on the **Bot Framework Composer**.

---

## 🔍 Key Characteristics

- **`kind`**: Specifies the dialog element type (e.g., `Question`, `ConditionGroup`, `SendActivity`, `InvokeFlowAction`).
- **`prompt`, `variable`, `entity`**: Define user interaction and capture input.
- **`InvokeFlowAction`**: Integrates Power Automate Flows into your dialog.
- **`ConditionGroup`**: Adds logic and branching based on user input or conditions.
- **`CancelAllDialogs`**: Terminates the conversation based on logic outcomes.

---

## ⚙️ Used For

- Building **advanced conversational flows** in Microsoft Copilot Studio.
- Implementing **conditional logic** and **multi-step dialogs**.
- **Calling Power Automate flows** for backend tasks like verifying user identity or issuing a Temporary Access Pass.
- **Interacting with Microsoft Graph**, Dataverse, Azure Key Vault, and more.

---

## 💡 Authoring Environment

You can work with this schema using:
- **Microsoft Copilot Studio** (Unified Canvas)
- **Bot Framework Composer** (for advanced editing and debugging)

---

## 🔗 Learn More

- [Microsoft Copilot Studio Documentation](https://learn.microsoft.com/en-us/microsoft-copilot-studio/)
- [Bot Framework Composer](https://learn.microsoft.com/en-us/composer/)
- [Power Virtual Agents Authoring](https://learn.microsoft.com/en-us/power-virtual-agents/)
