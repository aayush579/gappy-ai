# AI PR Review & Release Readiness Assistant (Powered by Lemma SDK)

This project is a complete, end-to-end collaborative workspace built for the **Gappy AI Hackathon** using the **Lemma SDK & Platform**. It implements the **AI PR Review & Release Readiness Assistant**, which automates pull request auditing, handles release verification checks, integrates a human-in-the-loop approval gate, and drafts release notes automatically upon approval.

---

## 💡 The Problem & Purpose

### The Problem
Traditional CI/CD pipelines run automated unit tests and linters, but they fail to catch architectural regressions, security anti-patterns, or memory leaks that require code reasoning. On the other hand, manual code review is slow and often blocks releases.

Furthermore, existing developer automation tools operate in siloed chat interfaces. There is no shared workspace where:
1. An AI agent's audit results are saved in structured tables.
2. A human operator can review the findings, fill out an approval form, and trigger next steps in the same interface.

### The Solution: The PR Assistant
This assistant bridges that gap using **Lemma**:
* **Automated Audit**: Whenever a PR is created or updated, its diff is ingested. An AI Agent analyzes the code, categorizes risks, and writes a detailed code review.
* **Structured State**: Review results, checklist statuses, and generated release notes are stored directly as columns in a structured table (`pr_checks`).
* **Human-in-the-Loop**: The workflow halts at a `FormNode` (Human Approval Gate). The release manager reviews the AI's feedback in the Lemma UI and decides whether to approve.
* **Release Generation**: If approved, the assistant automatically synthesizes the diff into structured, user-facing release notes.

---

## 🛠️ How it Uses the Lemma SDK

The assistant uses the `lemma-sdk` Python client to define and provision the workspace dynamically:

1. **Structured Tables (`pod.tables`)**: 
   Creates a `pr_checks` table with a custom schema (UUID ID, text diffs, checklist status, markdown code reviews, and release notes). This ensures all reviews are stored as permanent structured data, not ephemeral chat messages.
2. **AI Agents (`pod.agents`)**:
   Deploys `pr_reviewer_agent` configured with targeted instructions to audit pull requests, find risks, and draft code reviews.
3. **Multi-Step Workflows (`pod.workflows`)**:
   Defines a graph (`pr_review_workflow`) containing:
   * `AgentNode` (`run_pr_review`): Executes the AI PR audit.
   * `FormNode` (`human_approval`): Displays an interactive form in the Operator UI for the human to approve or reject the release.
   * `DecisionNode` (`routing_decision`): Routes execution based on human approval state.
   * `AgentNode` (`generate_release_notes`): Generates release notes for approved PRs.

---

## 🚀 Running Locally

Follow these steps to run the complete stack on your machine.

### Prerequisites
* Python 3.11+ (with `uv` installed)
* Node.js 18+
* Docker / Podman (running PostgreSQL, Redis, and SuperTokens)

### 1. Set Up and Run the Services
Ensure PostgreSQL, Redis, and SuperTokens containers are running:
```bash
cd lemma-platform/lemma-backend
docker compose up -d
```

### 2. Start the Backend API Server
1. Overrides inside `lemma-platform/lemma-backend/.env`:
   ```env
   FRONTEND_URL=http://localhost:3710
   AUTH_FRONTEND_URL=http://localhost:3710
   ```
2. Run the server:
   ```bash
   cd lemma-platform/lemma-backend
   uv run python standalone_app.py
   # Starts the API server at http://localhost:8711
   ```

### 3. Start the Next.js Frontend
```bash
cd lemma-platform/lemma-frontend
# Ensure it runs on port 3710
npx next dev --port 3710
```

### 4. Bootstrap and Simulate the Workflow
We provide scripts to bootstrap the database resources and run simulated reviews.

1. **Bootstrap the Workspace**:
   Creates the `pr_checks` table, the review agent, and the workflow in your local Lemma database:
   ```bash
   uv run python C:\Users\Aayush\.gemini\antigravity\brain\808c3a22-675a-41ad-9b26-162b044528df/scratch/setup_pr_assistant.py
   ```
2. **Run a Simulated PR Review**:
   Inserts a mock PR with a memory leak diff, triggers the workflow, and performs the audit:
   ```bash
   uv run python C:\Users\Aayush\.gemini\antigravity\brain\808c3a22-675a-41ad-9b26-162b044528df/scratch/simulate_workflow.py
   ```

3. **Interact in the UI**:
   Open [http://localhost:3710](http://localhost:3710), sign in with:
   * **Email**: `dev@gappy.ai`
   * **Password**: `Password123!`
   
   Go to the **PR Review Pod**, inspect the `pr_checks` datastore, and approve/reject pending runs in the Workflow tab!

---

## 📦 How to Deploy Live on GitHub

To publish this project to GitHub:

1. **Initialize a Git Repository** (if not already done at the root workspace):
   ```bash
   git init
   ```
2. **Add a `.gitignore`** to exclude virtual environments, system logs, and private keys:
   ```gitignore
   .venv/
   __pycache__/
   *.log
   .local/
   .env
   node_modules/
   ```
3. **Commit and Push to GitHub**:
   ```bash
   git add .
   git commit -m "feat: complete AI PR Reviewer with Lemma SDK integration"
   git remote add origin https://github.com/<your-username>/<your-repo-name>.git
   git branch -M main
   git push -u origin main
   ```
