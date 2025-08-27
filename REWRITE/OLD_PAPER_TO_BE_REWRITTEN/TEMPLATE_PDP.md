# The Persona-Driven Planning (PDP) Framework
A Reusable Template for LLM-Assisted Project Development

### **1. Core Philosophy**

This framework treats project planning as a structured, five-stage assembly line. A raw project concept is passed through a series of specialist LLM personas, each performing a single, well-defined function. The human's role is not to do the work, but to act as the architect, editor, and final quality gate at each stage. This methodology mitigates common LLM failures (sycophancy, lack of context) and ensures a project is architecturally sound and strategically aligned before implementation begins.

### **2. Prerequisites: The Concept Phase**

Before beginning this framework, you must have the following "Concept" artifacts prepared. These documents define the "what" of your project.

1.  **`app_summary.md`**: A plain-language description of the project, its purpose, target user, and core value proposition.
2.  **`visual_mockup`**: A visual representation of the final product.
    *   **For Web Apps:** A static `mockup.html` file.
    *   **For CLI Apps:** A text file showing sample command inputs and the desired terminal output.
    *   **For Automation Scripts:** A diagram or text file describing the "before" and "after" state of the files or systems it will operate on.
3.  **`feature_list.md` or `.json`**: A comprehensive, un-prioritized "brain dump" of all desired features for the final, fully-realized version of the project.

---

## The Five-Stage Planning Workflow

### **Stage 1: The Strategic Blueprint**

*   **Goal:** To establish the high-level project strategy and make the most critical, foundational architectural decisions with clear justification.
*   **Assigned Persona:** **The Staff Software Engineer**
*   **Inputs:** `app_summary.md`, `visual_mockup`, `feature_list`, `BRIEF_PROFILE.md` (your developer profile).
*   **Output:** `DOCUMENT_01_STRATEGIC_BLUEPRINT.md`

#### **Core Process:**
1.  Initiate a new session with the LLM.
2.  Provide all input documents and the full persona prompt for the **Staff Software Engineer**.
3.  The persona will analyze the project and identify the most critical decisions (e.g., database, framework, core algorithm).
4.  It will generate a simulated "Expert Debate" to explore the trade-offs of the most critical decision from multiple perspectives.
5.  It will conclude with a final, justified recommendation based on the debate and your developer profile.

#### **Human Decision-Maker's Role:**
*   Critically evaluate the debate. Does it feel balanced and realistic?
*   Review the final recommendation. Does the justification align with your project goals and constraints?
*   **Make the final decision.** You are the ultimate authority. Approve the recommendation or choose an alternative based on the evidence presented.

---

### **Stage 2: The Technical Foundation**

*   **Goal:** To translate the approved strategic decisions into a concrete, unambiguous technical specification.
*   **Assigned Persona:** **The Technical Foundation Architect**
*   **Inputs:** `DOCUMENT_01_STRATEGIC_BLUEPRINT.md` (and all concept documents).
*   **Output:** `DOCUMENT_02_TECHNICAL_FOUNDATION.md`

#### **Core Process:**
1.  Start a new session with the LLM.
2.  Provide the approved `DOCUMENT_01_STRATEGIC_BLUEPRINT.md` and the **Technical Foundation Architect** persona prompt.
3.  The persona will convert the high-level strategies into definitive technical contracts, including API schemas, data models, dependencies, and setup instructions.

#### **Human Decision-Maker's Role:**
*   Verify that all strategic decisions from Stage 1 have been correctly and completely implemented in the technical specs.
*   Ensure there is no ambiguity. The document should contain decisions, not options.
*   Approve the final technical foundation before proceeding.

---

### **Stage 3: The MVP Prioritization**

*   **Goal:** To ruthlessly define the scope of the Minimum Viable Product (MVP) to ensure rapid delivery of core value and prevent scope creep.
*   **Assigned Persona:** **The MVP Prioritization Strategist**
*   **Inputs:** `DOCUMENT_02_TECHNICAL_FOUNDATION.md`, `feature_list`.
*   **Output:** `DOCUMENT_03_MVP_PRIORITIZATION.md`

#### **Core Process:**
1.  Start a new session.
2.  Provide the `DOCUMENT_02_TECHNICAL_FOUNDATION.md`, the original `feature_list`, and the **MVP Prioritization Strategist** persona.
3.  The persona will classify all features into tiers (`Must Have`, `Should Have`, `Could Have`, `Won't Have`) and define the MVP success criteria.

#### **Human Decision-Maker's Role:**
*   Review the feature classifications. Be honest and disciplined. Does the "Must Have" list truly represent the absolute minimum for the app to be functional?
*   Approve the final MVP scope. This is your defense against adding "just one more thing" later.

---

### **Stage 4: The Development Execution Plan**

*   **Goal:** To convert the blueprint and MVP scope into a step-by-step, actionable implementation plan.
*   **Assigned Persona:** **The Development Execution Planner**
*   **Inputs:** `DOCUMENT_01`, `DOCUMENT_02`, `DOCUMENT_03`.
*   **Output:** `DOCUMENT_04_DEVELOPMENT_EXECUTION.md`

#### **Core Process:**
1.  Start a new session.
2.  Provide the first three planning documents and the **Development Execution Planner** persona.
3.  The persona will create a detailed plan broken down by milestones, with task checklists, workflow instructions, and a testing strategy.

#### **Human Decision-Maker's Role:**
*   Read through the entire plan. Does it feel logical and sequential?
*   Assess the milestones. Are they realistic?
*   Approve the final execution plan. This will become your guide for the implementation phase.

---

### **Stage 5: The Project Readiness Audit (The QA Loop)**

*   **Goal:** To perform a final, comprehensive quality assurance check on the entire plan, identify inconsistencies, and give the ultimate "Go / No-Go" signal for development.
*   **Assigned Persona:** **The Project Readiness Auditor**
*   **Inputs:** `DOCUMENT_01`, `DOCUMENT_02`, `DOCUMENT_03`, `DOCUMENT_04`, and all Concept Phase documents.
*   **Output:** `DOCUMENT_05_PROJECT_READINESS.md`

#### **Core Process: The QA Loop**
1.  Start a new session. It is **highly recommended** to use a different LLM model or service for this stage to get a truly "second opinion."
2.  Provide all four planning documents, all concept documents, your developer profile, and the **Project Readiness Auditor** persona.
3.  The persona will conduct a full audit, score the plan's readiness, and provide a list of "Green Light," "Yellow Light," and "Red Light" items.
4.  **If the result is anything other than a perfect "GREEN LIGHT":**
    *   Take the feedback (the "Yellow" and "Red" items).
    *   Go back to the specific stage (e.g., Stage 2 for a technical spec issue) and the corresponding persona.
    *   Provide the auditor's feedback and instruct the original persona to generate a revised version of its document.
    *   **Repeat this audit process (Stage 5) with the revised documents.**
5.  Continue this loop until the Project Readiness Auditor gives a definitive "✅ **GREEN LIGHT: Proceed with Development**."

#### **Human Decision-Maker's Role:**
*   You are the judge. You initiate the loop, provide the feedback to the specialist personas, and decide when a revised document is ready for re-auditing.
*   You make the final call to exit the planning phase and begin implementation once the "GREEN LIGHT" is achieved.