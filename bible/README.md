# Project Bible
This is the guiding document for the Chordline project, a modular narrative design and simulation framework.

This project is the result of a collaboration between its human authors and AI agents. See [[README#Use of AI]] for more detail.

All files in the project bible start with content that is intended to be copied and pasted into new chats with AI to provide context. People may find them useful, too.

# Ending a conversation
Each AI conversation should be ended with the following prompt. It will be up to a human to evaluate that summary and act upon it.

```markdown
Before we close this conversation, please briefly summarize any ways this discussion should inform updates to:

- The context description for the relevant file(s)
- The context description for the containing folder(s)
- The overall project context description

If no updates are necessary, please state that clearly. If the conversation touched on multiple topics, please address each topic separately in your summary. Please point out any the topics do not fit any of the context statements provided at the beginning of this chat.
```

# Context Statement
```markdown
Context Statement - Project

Context Note: For this conversation, do _not_ create or update any persistent account memory.
Treat all new information as session-only.

# Organizational Structure

Within this project, a **Topic** refers to a folder that groups related files by subject, and an **Entry** refers to an individual file within a Topic.

Context statements may be written at the Topic level (covering one or more Topics) or at the Entry level (covering one or more specific Entries).
In such cases, the statement applies collectively to all listed items unless otherwise noted.

As the documentation evolves, Topics may be split, merged, or reorganized, but Entries always inherit the context of their current parent Topic unless explicitly overridden.

# Project Inspiration

Narrative systems that support reactivity, branching, and emotional modeling quickly give rise to emergent complexity as a result of the compounding of small interactions into unpredictable or unintuitive behavior.
While this complexity can be a strength for dynamic storytelling, it poses a significant challenge to authors: maintaining coherence, ensuring emotional consistency, and avoiding unintended contradictions or dead ends.

# Project Overview

Chordline is intentionally designed to support authors at design time as well as at runtime.
Rather than burying logic in opaque scripts or monolithic event trees, it favors structured, declarative, data broken into manageable units such as scenes, states, conditions, and emotional effects.
These are organized around principles of separation of concerns, traceability, and narrative intent.
```

# Project Goals

To help authors navigate and manage narrative complexity, Chordline emphasizes:

* **Modular Narrative Units**  
    Each scene, interaction, or emotional rule is defined as an isolated, composable unit with clearly scoped inputs and outputs. This prevents cross-contamination of logic and allows authors to focus on one piece of the system at a time.

- **Declarative State and Conditions**  
    State transitions and narrative gating are expressed declaratively, making the logic both readable and auditable. This avoids the "spaghetti logic" of deeply nested conditionals and imperative code.

- **Metadata for Indexing and Review**  
    All narrative content and emotional interactions support metadata—such as tags, themes, or emotional tone—which allows tooling to index, visualize, or query the content. This supports high-level design thinking without getting lost in low-level implementation.

- **Static Validation Hooks**  
    Built-in support for validating references, state keys, emotional schema consistency, and narrative prerequisites helps authors catch problems **before** runtime. These checks are lightweight but extensible, with the goal of surfacing ambiguity or contradiction early in the design process.

- **Optional Integration with AI-Assisted Review Tools**  
    Although AI tooling is not core to Chordline, its structured data model enables plugin-level features for authors who want help summarizing, cross-checking, or generating content. This allows authors to remain in control while benefiting from machine assistance where appropriate.

Chordline’s approach is grounded in the belief that **emergence should serve narrative clarity**, not obscure it. By giving authors tools to understand, visualize, and control the complexity of their story logic and emotional systems, Chordline turns reactivity from a burden into a strength.