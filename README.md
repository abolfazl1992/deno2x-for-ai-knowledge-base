# Deno 2.x AI Knowledge Base 🦕💡

## 🤔 What's This All About?

I put this together because, like many of you, I noticed that most AI models (like ChatGPT, Gemini, etc.) are still stuck in the past, generating code for **Deno 1.x**. Since Deno 2.x introduced a lot of significant (and awesome!) breaking changes, that old code just doesn't work anymore.

This repository contains a single, comprehensive markdown file (`DENO_2_KNOWLEDGE_BASE.md`) that acts as a brain-dump of all the critical information about Deno 2.x. The goal is to give our AI assistants the knowledge they need to be truly helpful Deno 2 developers.

It covers:
*   The high-level strategy behind Deno 2 (especially Node/npm compatibility).
*   **Crucial breaking changes** from v1 to v2.
*   Every major **CLI flag change** (`deno bundle`, `deno cache`, `--unstable`, etc.).
*   A detailed list of **removed/renamed APIs** (`Deno.run`, `Deno.copy`, RIDs) and their modern replacements.
*   New best practices for a Deno 2.x world.

## 🚀 How to Use This with Your AI

This is the easy part! To get your AI assistant to generate correct Deno 2.x code, you can provide it with this knowledge base as context.

Here are a few ways to do it:

#### 1. The Easy Way: Copy & Paste into the Prompt

Just copy the entire content of the `DENO_2_KNOWLEDGE_BASE.md` file and paste it at the beginning of your prompt before you ask your question.

**Example Prompt:**

> ```
> [PASTE THE ENTIRE CONTENT OF DENO_2_KNOWLEDGE_BASE.MD HERE]
>
> ---
>
> Now, using the knowledge provided above, please write a simple Deno 2 web server using Deno.serve() that responds with "Hello, Deno 2!".
> ```

#### 2. The Attachment Method

If your AI service allows file uploads (like Gemini Advanced or the ChatGPT desktop app), just attach the `DENO_2_KNOWLEDGE_BASE.md` file to your prompt.

#### 3. The Pro Way: System Instructions

For services that have a "System Prompt" or "Custom Instructions" feature, you can paste the knowledge base content there. This way, the AI will have the context for your entire conversation without you needing to paste it every time.

## 🙏 Want to Help?

This knowledge base was created from the official docs, but things change! If you spot something that's missing, outdated, or just plain wrong, please feel free to:

*   Open an `Issue`.
*   Submit a `Pull Request`.

Any and all contributions are welcome. Let's make our AI helpers smarter, together!

## 📄 License

This project is shared under the MIT License. Feel free to use it however you see fit. See the `LICENSE` file for more details.