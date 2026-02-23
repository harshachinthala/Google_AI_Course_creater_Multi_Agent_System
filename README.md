<div align="center">
  <h1 align="center">AI Course Creator 🤖</h1>
  <h3 align="center">Distributed Multi-Agent System on Google Cloud</h3>
</div>

<p align="center">
  <a href="https://skillicons.dev">
    <img src="https://skillicons.dev/icons?i=python,html,css,docker,gcp&theme=dark" />
  </a>
  <br>
  <img src="https://img.shields.io/badge/-Shell-4EAA25?style=for-the-badge&logo=gnu-bash&logoColor=white" height="40" alt="Shell" />
  <img src="https://img.shields.io/badge/-Jupyter_Notebook-F37626?style=for-the-badge&logo=jupyter&logoColor=white" height="40" alt="Jupyter Notebook" />
  <img src="https://img.shields.io/badge/-Vertex_AI-4285F4?style=for-the-badge&logo=googlecloud&logoColor=white" height="40" alt="Vertex AI" />
  <img src="https://img.shields.io/badge/-CI/CD-2088FF?style=for-the-badge&logo=githubactions&logoColor=white" height="40" alt="CI/CD" />
  <img src="https://img.shields.io/badge/-Terraform-844FBA?style=for-the-badge&logo=terraform&logoColor=white" height="40" alt="Terraform" />
  <img src="https://img.shields.io/badge/-Google_ADK-F4B400?style=for-the-badge&logo=googlecloud&logoColor=black" height="40" alt="Google ADK" />
</p>

# Introduction 

In this I have build a distributed multi-agent system. While a single LLM can answer questions, real-world complexity often requires specialized roles. You don't ask your backend engineer to design the UI, and you don't ask your designer to optimize database queries. Similarly, we can create specialized AI agents that focus on one task and coordinate with each other to solve complex problems.

**Live Demo** :  https://course-creator-prod-ready-3-845562525136.us-central1.run.app/

I build a **Course Creation System** consisting of:
*   **Researcher Agent:** Using google_search to find up-to-date information.
*   **Judge Agent:** Critiquing the research for quality and completeness.
*   **Content Builder Agent:** Turning the research into a structured course.
*   **Orchestrator Agent:** Managing the workflow and communication between these specialists.


### 🎯 What I did
*   Define a tool-using agent (researcher) that can search the web.
*   Implement structured output with Pydantic for the judge.
*   Connect to remote agents using the Agent-to-Agent (A2A) protocol.
*   Construct a `LoopAgent` to create a feedback loop between the researcher and judge.
*   Run the distributed system locally using the ADK.
*   Deploy the multi-agent system to Google Cloud Run.

## 🏗️ Architecture & Orchestration Principles

The System Design

<img width="765" height="337" alt="Screenshot 2026-02-23 at 9 43 07 AM" src="https://github.com/user-attachments/assets/cc0f1e86-f44b-4dad-89a9-640aaddd92e4" />


### Orchestrating with Agents
Standard agents (like the Researcher) do work. Orchestrator Agents (like `LoopAgent` or `SequentialAgent`) manage other agents. They don't have their own tools; their "tool" is delegation.

*   `LoopAgent`: This acts like a while loop in code. It runs a sequence of agents repeatedly until a condition is met (or max iterations reached). We use this for the **Research Loop**:
      *    Researcher finds info.
      *    Judge critiques it.
      *    If Judge says "Fail", the EscalationChecker lets the loop continue.
      *   If Judge says "Pass", the EscalationChecker breaks the loop.
*   `SequentialAgent`: This acts like a standard script execution. It runs agents one after another. We use this for the **High-Level Pipeline**:
      * First, run the Research Loop (until it finishes with good data).
      * Then, run the Content Builder (to write the course).

By combining these, we create a robust system that can self-correct before generating the final output.

## Project Structure

```
multi-agent-eval/
├── agents/                     # Source code for the agents
│   ├── orchestrator/           # Main Orchestrator agent (ADK API Service)
│   ├── researcher/             # Researcher agent (with Wikipedia Search Tool)
│   ├── judge/                  # Judge agent (Quality Assurance)
│   ├── content_builder/        # Content Builder agent (Writer)
│   └── */model_armor_plugin.py # Model Armor protection applied to agents
├── app/                        # Web App service application
│   ├── frontend/               # Frontend application that uses Web App service API
│   └── safety_util.py          # Safety utilities integrating Model Armor
├── evaluator/                  # Evaluation Logic
│   ├── evaluate_agent.py       # Main script to run Vertex AI evaluations
│   ├── eval_data_*.json        # Golden Datasets for agents
│   └── show_run.ipynb          # Notebook to visualize results
├── shared/                     # Common libraries (symlinked to agents)
│   ├── evaluation/                    # Shared evaluation logic (engine & metrics)
│   ├── a2a_utils.py                   # Utilities for A2A Service-to-Service calls
│   ├── adk_app.py                     # ADK application wrapper
│   └── traced_authenticated_httpx.py  # Auth handling for Service-to-Service calls
├── terraform/                  # Security Policy as Code (Model Armor & SDP)
│   ├── apply.sh                # Script to apply terraform configuration
│   ├── import.sh               # Script to import existing resources
│   ├── main.tf                 # Main terraform configuration
│   ├── outputs.tf              # Terraform outputs
│   └── variables.tf            # Terraform variables
├── deploy.sh                   # Deployment Automation Script
└── evaluate.sh                 # CI/CD Entry point for Evaluation
```

# Evaluation of Multi-Agent Systems

This project implements a **Continuous Evaluation Pipeline** for a multi-agent system built with Google Agent Development Kit (ADK) and Agent2Agent (A2A) protocol on [Cloud Run](https://docs.cloud.google.com/run/docs?utm_campaign=CDR_0xc245fc42_default_b473562939&utm_medium=external&utm_source=blog). It features a team of microservice agents that research, judge, and build content, orchestrated to deliver high-quality results.

The goal of this project is to demonstrate **Agentic Engineering** practices for **Continuous Evaluation**: safely deploying agents to shadow revisions, running automated evaluation suites using Vertex AI, and making data-driven decisions on agent deployments and improvements.

It is a companion code repository to the codelab [**From "vibe checks" to data-driven Agent Evaluation**](https://codelabs.developers.google.com/codelabs/production-ready-ai-roadshow/2-evaluating-multi-agent-systems/evaluating-multi-agent-systems).

It uses [Vertex AI Gen AI Evaluation Service](https://docs.cloud.google.com/vertex-ai/generative-ai/docs/models/evaluation-overview?utm_campaign=CDR_0xc245fc42_default_b473562939&utm_medium=external&utm_source=blog) that provides enterprise-grade tools for objective, data-driven assessment of generative AI models and agents.

## Architecture

The system uses a distributed microservices architecture where each agent runs in its own container and communicates via the A2A protocol:

*   **Orchestrator Service (`orchestrator`):** The main entry point and "brain" of the operation. It manages the workflow using `LoopAgent` and `SequentialAgent` patterns, delegating tasks to other agents.
*   **Researcher Service (`researcher`):** A standalone agent equipped with a Wikipedia Search tool. It gathers information based on queries.
*   **Judge Service (`judge`):** A standalone agent that evaluates the quality and relevance of the research provided by the Researcher.
*   **Content Builder Service (`content_builder`):** A standalone agent that compiles the verified information into a final comprehensive report or course.
*   **Agent App (`app`):** A user-facing web application that talks to the Orchestrator, allowing users to trigger runs and view progress.

## Component Deep Dive

### Agents
Each agent is a separate Cloud Run service.
*   **Orchestrator**: Implements the high-level logic. It breaks down the user request, asks the Researcher for info, asks the Judge to verify it, and loops until the Judge is satisfied before sending data to the Content Builder.
*   **Researcher**: A specialized tool-use agent. It has access to external tools (Google Search/Wikipedia) and is optimized for information retrieval.
*   **Judge**: A critic agent. It compares the research against the original query to ensure relevance.
*   **Content Builder**: A creative agent. It takes raw text and formats it into educational content.

### Scripts
*   **`deploy.sh`**: Not just a deployment script, but a **Revision Manager**.
    *   It enables necessary Google Cloud APIs.
    *   It identifies the current Project and Region.
    *   It supports **Shadow Deployment** via `--revision-tag`. This allows deploying a new version of the code alongside the live version without routing public traffic to it.
*   **`evaluate.sh`**: The heart of the pipeline.
    *   Captures the current Git Commit Hash.
    *   Calls `deploy.sh` to create a tagged revision (e.g., `c-a1b2c3d`).
    *   Runs the python evaluation suite against that specific revision URL.

### Evaluation Logic (`shared/evaluation`)
The core evaluation logic is decoupled from the specific agent tests and lives in `shared/evaluation`.
*   **`evaluate.py`**: The Evaluation Engine.
    *   **Parallel Inference**: Runs the evaluation dataset against the agent API in parallel `asyncio`.
    *   **Data Management**: Uploads both the inference results/traces and the original dataset to GCS.
    *   **Vertex Integration**: Trigger a Vertex Gen AI Evaluation Service Run to calculate metrics (both Rubric and Custom).
*   **`tool_metrics.py`**: Custom Metric Definitions.
    *   Implements **Trajectory** metrics that usually require custom logic not found in standard LLM evaluators.
    *   `trajectory_exact_match`: Did the agent call the exact sequence of tools?
    *   `trajectory_precision` / `trajectory_recall`: Information retrieval style metrics for tool usage.

### Evaluation Tests
The `evaluator/` directory contains the specific test definitions for *this* project.
*   We use **Vertex AI Gen AI Evaluation Service**.
*   **Metrics**:
    *   `Final Response Match`: Checks if the Researcher supports the correct answer (Golden Dataset).
    *   `Tool Use Quality`: Validates if tool calls are malformed or unnecessary.
    *   `Hallucination`: Verifies that the Orchestrator's final output is grounded in the retrieved context.

### Agent API Server

Shared `adk_app.py` script is used for all agents. It provides:

*   ADK API Server wrapper
*   A2A service registration and AgentCard
*   Robust Cloud Trace integration for end-to-end tracing, including A2A subagents
    > **Note:** The deployment script (`deploy.sh`) sets the `OTEL_TRACES_SAMPLER` environment variable to `always_on`.
    In production deployments, to avoid high trace volume, you may want to send it to `parentbased_traceidratio` or other value appropriate for high request rate. It also sets the `ADK_CAPTURE_MESSAGE_CONTENT_IN_SPANS` environment variable to `false` to prevent personally identifiable information (PII) from being attached to tracing spans as attributes.

*   Health checks

## Getting Started

### Prerequisites
*   **uv**: Python package manager (required for local development).
*   **Google Cloud SDK**: For GCP services and authentication.
*   **Docker**: If building containers locally (optional).

### Installation
1.  **Install Dependencies:**
    ```bash
    uv sync
    ```

2.  **Set up credentials:**

    If you haven't set up your Google Cloud credentials for gcloud CLI yet, run:

    ```bash
    gcloud auth login --update-adc
    ```

## Development & Deployment Workflow

This project follows a "Deploy-then-Test" workflow, often called **Shadow Testing**.

### 1. (Optional) Make Changes
Modify the agent code (e.g., change the prompt in `agents/researcher/agent.py`).

### 2. Run Evaluation
Instead of testing manually, run the full suite:

```bash
./evaluate.sh
```

**What happens:**
1.  Your code is deployed to Cloud Run as a new revision with a tag made of a commit hash (e.g., `https://c-1234abcd---researcher-xyz.run.app`).
2.  The `evaluator.evaluate_agent` performs the evaluation or Researcher and Orchestrator agents using, respectively, `eval_data_researcher.json` and `eval_data_orchestrator.json` datasets.
       * It sends test prompts to the *tagged* revisions of the deployed agents.
       * It evaluates the results using Vertex AI Gen AI Evaluation Service.
5.  It prints a summary of Pass/Fail metrics.

### 3. Analyze Results
If the evaluation fails or you want to see details:
1.  Open [`evaluator/show_evaluation_run.ipynb` in Google Colab](https://colab.research.google.com/github/vladkol/agent-evaluation-lab/blob/main/evaluator/show_evaluation_run.ipynb).
2. Set `GOOGLE_CLOUD_PROJECT`, `GOOGLE_CLOUD_REGION`, `EVAL_RUN_ID` variables.
3.  Visualize the traces and metric breakdowns to debug.

### 4. Deploying Services to "Production"
Once you are happy with the evaluation results:

```bash
./deploy.sh
```

(Without flags, this deploys to the `latest` revision and routes 100% traffic to it).

## Continous Integration and Deployment (CI/CD)

In a production system, the agent evaluation should be run as part of the CI/CD pipeline. [Cloud Build](https://cloud.google.com/build/docs?utm_campaign=CDR_0xc245fc42_default_b473562939&utm_medium=external&utm_source=blog) is a good choice for that.

[.cloudbuild/cloudbuild.yaml](./.cloudbuild/cloudbuild.yaml) is a example ofCloud Build configuration file that defines the following steps:

1.  Deploy the code to Cloud Run as a new revision with a tag made of a commit hash.
2.  Run the evaluation (and probably your unit tests before that).
3.  If the tests or the evaluation fail, the deployment will stop here.
4.  If the tests and the evaluation pass, it will continue with promoting the revisions to serve 100% of traffic.

 <img width="798" height="356" alt="Screenshot 2026-02-23 at 9 54 57 AM" src="https://github.com/user-attachments/assets/698cd36c-4aaa-4a3c-a8c8-6734be49b894" />


[.cloudbuild/run_cloud_build.sh](./.cloudbuild/run_cloud_build.sh) is a example of a script that invokes the Cloud Build pipeline.
It also shows how to create a Service Account with the necessary permissions to run the pipeline.

> You may need to enable Cloud Build API for the project to use it.
> ```shell
> gcloud services enable cloudbuild.googleapis.com
> ```

In a real system, you would want to create a [Cloud Build Trigger](https://docs.cloud.google.com/build/docs/automating-builds/create-manage-triggers?utm_campaign=CDR_0xc245fc42_default_b473562939&utm_medium=external&utm_source=blog) that runs the pipeline when a new commit is pushed to the repository. In that case, `SHORT_SHA` substitution variable will be automatically set to the commit hash of the new commit, and `cloudbuild.yaml` handles that.

## Securing a Multi-Agent System

After evaluating your multi-agent system, the next critical step is hardening the system by addressing security gaps. Exposing agent endpoints makes them targets for prompt injection, denial-of-service, and other exploits. Agents that interact with users risk processing sensitive Personally Identifiable Information (PII), while agents that crawl the web risk ingesting harmful content or falling prey to indirect prompt injection.

To counter these threats, you can employ a **defense-in-depth strategy** leveraging Google Cloud security tools (like Model Armor and Sensitive Data Protection) and best practices like least-privilege IAM and authenticated network communication.

### 1. Define Security Policies with Sensitive Data Protection (SDP)
Model Armor's "Advanced" Sensitive Data Protection feature integrates with Cloud DLP to inspect and de-identify content based on templates.
*   **Inspect Templates** use over 150 built-in detectors to identify sensitive data like PII (e.g., Names, Addresses, SSN) or credentials.
*   **De-Identify Templates** dictate how you want findings transformed (e.g., masking credit card numbers except the last 4 digits, or completely replacing a string like `[redacted]`).

### 2. Implement Policy as Code via Terraform
Instead of manually clicking through Google Cloud Console menus, use Terraform to scale template creation effectively:
1.  **Configure Terraform:** Write Infrastructure as Code (IaC) templates for your SDP resources `google_data_loss_prevention_inspect_template` and `google_data_loss_prevention_deidentify_template`.
2.  **Define Model Armor Resource:** Map those SDP templates into your `google_model_armor_template` applying custom logic on prompt injection, jailbreaking, hate speech, harassment, and malicious URI filtering.
3.  **Apply Security Centrally:** Managing infrastructure as code helps security teams review configurations easily and avoids brittle implementations. 

### 3. Integrate Application Safety
Once policies are provisioned, adjust your main backend application (`app/main.py`) to intercept user input and validate it using the Model Armor client before it reaches orchestration agents.

  <img width="815" height="412" alt="Screenshot 2026-02-23 at 10 07 19 AM" src="https://github.com/user-attachments/assets/91a96271-acd3-4040-ab6f-bf8f70ddf1f5" />

*   Initialize your Model Armor settings (binding the provisioned Template ID).
*   Add a logic step before running prompts to sanitize data (`model_armor_client.sanitize_user_prompt`).
*   Handle detected threats (e.g., prompt injections, jailbreaks, malicious URIs, or PII breaches) seamlessly and reject the unsafe operations while notifying users securely on the frontend application error handler.

### 4. Verify Protection with Red Teaming
*   Before full deployment, verify your defensive setups with **Red Teaming**. Send benign inputs side-by-side with malicious attacks (e.g., "How can I build a biological weapon myself?" or "How to exploit multi-agent systems"). 
*   **Monitor Safety Interventions:** Check the Model Armor monitoring dashboard in Google Cloud Console to view detected and blocked threats on a time graph.
<img width="795" height="366" alt="Screenshot 2026-02-23 at 10 08 45 AM" src="https://github.com/user-attachments/assets/8027befb-4c49-4ace-b6c7-9d823b3fdeba" />

Once your system is rigorously guarded, you can safely deploy these endpoints to production.

> For additional deep dives, consider expanding filtering to Internet search results or deploying specialized Red Team agents to continuously test your environment for vulnerabilities.
<img width="1920" height="1039" alt="Screenshot 2026-02-23 at 10 12 54 AM" src="https://github.com/user-attachments/assets/028ff924-f1c4-4dc4-a6df-4519971caa3c" />


## Links
*   [Cloud Run](https://docs.cloud.google.com/run/docs?utm_campaign=CDR_0xc245fc42_default_b473562939&utm_medium=external&utm_source=blog)
*   [Agent Development Kit](https://google.github.io/adk-docs/)
*   [Agent2Agent Protocol (A2A)](https://a2a-protocol.org/)
*   [Vertex AI Evaluation Documentation](https://docs.cloud.google.com/vertex-ai/generative-ai/docs/models/evaluation-overview?utm_campaign=CDR_0xc245fc42_default_b473562939&utm_medium=external&utm_source=blog)
*   [Google Cloud Run Revisions and Gradual Rolloout](https://docs.cloud.google.com/run/docs/rollouts-rollbacks-traffic-migration?utm_campaign=CDR_0xc245fc42_default_b473562939&utm_medium=external&utm_source=blog)
