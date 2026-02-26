# MagicPrompt

**Structured response traits for ChatGPT, Claude, Gemini, and open-source LLMs.**

MagicPrompt is a system prompt that makes AI responses more accurate, structured, and honest. It enforces a Plan-Check-Ground workflow: the model plans before responding, checks its own work at each step, and grounds claims in evidence rather than guessing.

Created in 2023 by [Ryan Niemann](https://github.com/ryanniemann). Originally built for ChatGPT's custom traits field, v3.0 expands to a family of three prompts optimized for different models and interfaces.

## What It Does

When you use MagicPrompt, every response follows a consistent structure:

1. **Plan** — The model restates your request, builds a checklist, and asks clarifying questions (only when they'd meaningfully change the answer).
2. **Respond and Check** — The model delivers a response, then reviews it for logic errors, contradictions, gaps, and unsupported claims.
3. **Revise** — Issues found during the check are addressed in a revised response — not cosmetic edits, but substantive improvements.
4. **Ground** — Claims are tied to evidence. When the model is uncertain, it says so instead of guessing.

The result: fewer hallucinations, fewer assumptions, and responses you can actually trust.

## Which Version Should I Use?

| Version | Best For | Where to Paste It | Character Count |
|---------|----------|-------------------|-----------------|
| [**Compact**](prompts/compact-chatgpt.md) | ChatGPT | Settings → Personalization → "What traits should ChatGPT have?" | ~1,434 |
| [**Standard**](prompts/standard-claude-gemini.md) | Claude, Gemini | Claude Profile → Personal Preferences or Projects (system prompt), Gemini Gems, or start of conversation | ~3,663 |
| [**Extended**](prompts/extended-opensource.md) | Llama, Mistral, Mixtral, Phi, etc. | System prompt in Ollama, LM Studio, text-generation-webui, or API calls | ~6,945 |

**Not sure?** Start with the one that matches the model you use most. If you use multiple models, use the matching variant for each — they produce the same workflow but are tuned for how each model behaves.

### Why three versions?

Different models have different failure modes. ChatGPT tends to overformat and overuse em dashes. Claude tends to be so helpful it answers its own clarifying questions. Open-source models often ignore pause instructions entirely. Each variant is tuned for its target model's specific tendencies.

The three versions also reflect different character budgets. ChatGPT's traits field has a hard 1,500-character limit. Claude and Gemini support longer system prompts. Open-source models have no limit and benefit from explicit examples that larger models don't need.

## The PCG Framework

MagicPrompt is built on three principles, abbreviated as **PCG**:

| Principle | What It Means | What It Prevents |
|-----------|---------------|------------------|
| **Plan** | Understand the request before responding. Restate it, clarify ambiguities, decompose complex tasks. | Charging ahead on wrong assumptions. Producing a plausible but misaligned response. |
| **Check** | Validate reasoning at each step. Flag contradictions, gaps, and unsupported claims. | Internal inconsistencies. Confident-sounding answers built on shaky logic. |
| **Ground** | Base claims in evidence. Cite sources. Say "I don't know" when uncertain. | Hallucinations. Plausible-sounding fabrications. Guesses presented as facts. |

## How It Works In Practice

### Before MagicPrompt

**You:** "Help me restructure my engineering team."

**AI:** *Immediately produces a 2,000-word reorganization plan based on assumptions about your team size, tech stack, company stage, and goals — none of which it asked about.*

### After MagicPrompt

**You:** "Help me restructure my engineering team."

**AI:**

> Here's how I'm interpreting your request:
>
> **Checklist:**
> 1. Current team structure and pain points
> 2. Target structure recommendation
> 3. Transition plan
> 4. Role definitions
> 5. Risks and mitigations
>
> A couple of questions before I start:
> 1. How large is the team currently, and how is it organized (by function, product, or something else)?
> 2. What's driving the restructure — scaling challenges, delivery speed, or something else?
>
> Once you answer these, I'll tailor my response accordingly.

The AI asks *before* assuming. The response that follows is grounded in your actual situation, not a generic template.

## Key Design Decisions

**The model asks questions only when they matter.** MagicPrompt doesn't force a fixed number of questions. It instructs the model to ask only when the answer would meaningfully change the response. Simple questions get direct answers. Complex or ambiguous requests get targeted clarification first.

**Two-pass responses catch real issues.** The Considerations step between Response and Revised Response isn't cosmetic — the prompt explicitly instructs the model to identify substantive issues and address them, not to manufacture superficial changes when none exist.

**Uncertainty is a feature, not a failure.** Rather than guessing confidently (the default behavior of most LLMs), MagicPrompt instructs the model to flag low-confidence areas, distinguish between well-supported and uncertain claims, and say "I don't know" when it genuinely doesn't.

**Formatting scales to content.** A two-sentence answer doesn't get headers and bullet points. A multi-section analysis does. The prompt prevents over-formatting on simple queries and under-formatting on complex ones.

## Installation

### ChatGPT (Compact)

1. Open [ChatGPT](https://chat.openai.com)
2. Go to **Settings → Personalization → Customize ChatGPT**
3. In the "What traits should ChatGPT have?" field, paste the contents of [`prompts/compact-chatgpt.md`](prompts/compact-chatgpt.md) (everything below the "Copy" line)
4. Save

### Claude (Standard)

1. Open [Claude](https://claude.ai)
2. Create or open a **Project**
3. In the project's **system prompt** field, paste the contents of [`prompts/standard-claude-gemini.md`](prompts/standard-claude-gemini.md) (everything below the "Copy" line)

Alternatively, paste it at the start of any conversation.

### Gemini (Standard)

1. Open [Gemini](https://gemini.google.com)
2. Create a **Gem**
3. In the instructions field, paste the contents of [`prompts/standard-claude-gemini.md`](prompts/standard-claude-gemini.md) (everything below the "Copy" line)

### Open-Source Models (Extended)

Paste the contents of [`prompts/extended-opensource.md`](prompts/extended-opensource.md) (everything below the "Copy" line) as the **system prompt** in your interface:

- **Ollama:** Use the `SYSTEM` field in your Modelfile, or pass via `--system` flag
- **LM Studio:** Paste into the System Prompt field in the chat settings
- **text-generation-webui:** Paste into the "Context" or "System message" field
- **API calls:** Include as the `system` role message

## Known Limitations

**The two-pass workflow adds overhead.** Every response goes through Response → Considerations → Revised Response. For quick factual questions, this produces a better answer but takes longer. The Standard and Extended versions mitigate this by routing simple queries to a more direct path.

**Clarification pauses aren't guaranteed.** When MagicPrompt tells a model to stop and wait for your answers, most models comply most of the time — but not always. Claude may occasionally answer its own questions. Open-source models may ignore the pause instruction entirely. The Extended version uses an Assumptions Block as a fallback: even if the model doesn't pause, you can see and correct its assumptions on your next message.

**Self-review can't catch reasoning errors it doesn't recognize.** The Considerations step reliably catches formatting issues, missing context, and factual gaps. It's weaker at catching logical reasoning errors — a known limitation of all current LLMs. For high-stakes decisions, treat the model's self-review as a starting point, not a substitute for your own judgment.

**The Compact version has no simple-query bypass.** Because of ChatGPT's 1,500-character limit, the Compact version always runs the full structured workflow. If you ask a simple factual question, the model should ask zero clarifying questions and keep the checklist minimal, but the structure is still present. The Standard and Extended versions handle this more gracefully by routing simple queries directly to a response.

## Version History

### v3.0 (Current)

- Expanded from a single ChatGPT prompt to a family of three model-optimized variants
- Redefined PCG from "Prompt Completion Guidelines" to the **Plan-Check-Ground** mnemonic — every letter now maps to a concrete behavioral instruction
- Replaced rigid "exactly three clarifying questions" with impact-based calibration — the model asks only when the answer would meaningfully change the response
- Redesigned the pause mechanism per model: hard stop (ChatGPT), conditional pause with explicit prohibitions (Claude/Gemini), Assumptions Block fallback (open-source)
- Added "meaningfully differ" quality gate — the Revised Response must substantively address issues, not repeat the Response with minor additions
- Grouped Considerations by function: Verify → Gaps → Improve
- Added workflow routing for Standard/Extended: simple queries skip directly to a response
- Added formatting fallback for non-Markdown environments (Extended)
- Added few-shot examples for open-source models (Extended)
- Added capability declaration for models without web search (Extended)

### v2.2.3

- Original single-prompt version optimized for ChatGPT's traits field
- Fixed three-question rule, Order 1/Order 2 workflow, PCG as "Prompt Completion Guidelines"

## License

[MIT](LICENSE)

## Contributing

Feedback, testing results across different models, and suggested improvements are welcome. Open an issue or submit a pull request.

If you test MagicPrompt on a model not covered here and have observations about what works or breaks, that's especially valuable.
