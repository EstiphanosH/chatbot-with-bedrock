# Customer Support Chatbot with Amazon Bedrock Flows

[![AWS Cloud](https://shields.io)](https://amazon.com)
[![Amazon Bedrock](https://shields.io)](https://amazon.combedrock/)
[![Python](https://shields.io)](https://python.org)

An end-to-end serverless customer support automation workflow orchestrated via **Amazon Bedrock Flows** and powered by **Amazon Nova** foundation models. This application demonstrates advanced conversational classification, autonomous tool execution via Bedrock Agents, and automated LLM-as-a-judge quality evaluations.

---

## 🏗️ Architecture Overview

The system classifies, gates, and routes incoming customer inquiries into three isolated operational paths:

```text
                       ┌──────────────────────┐
                       │    FlowInputNode     │
                       └──────────┬───────────┘
                                  │ (User Message)
                       ┌──────────▼───────────┐
                       │  Routing Classifier  │
                       └──────────┬───────────┘
                                  │
                    ┌─────────────┼─────────────┐
                    │          Router           │
                    ├───────────────────────────┤
                    │             │             │
                    ▼             ▼             ▼
              [BUG_REPORT] [PLATFORM_QUESTION] [OTHER_REQUEST]
                    │             │             │
        ┌───────────▼───┐  ┌──────▼────────┐    │
        │  BUG REPORT   │  │  Prompt Node  │    │
        │    HANDLER    │  │ (Embedded FAQ)│    │
        └───────┬───────┘  └──────┬────────┘    │
                │                 │             │
         ┌──────▼───────┐  ┌──────▼────────┐  ┌─▼─────────────┐
         │  FlowOutput  │  │ FlowOutput_2  │  │ FlowOutput_3  │
         │ (Bug Ticket) │  │ (FAQ Answer)  │  │ (Escalation)  │
         └──────────────┘  └───────────────┘  └───────────────┘
```

1. **Bug Reports:** Handled by an Amazon Bedrock Agent via AgentCore layout. The agent dynamically collects three mandatory parameters (Description, Steps to Reproduce, Environment) via multi-turn conversation and invokes an AWS Lambda function to persist the ticket into Amazon DynamoDB.
2. **Platform Questions:** Handled directly within a Flow Prompt Node. An authoritative e-commerce FAQ (`online_shop_faq.md`) is injected into the context window at inference time for deterministic, hallucination-free generation.
3. **Other Requests:** Acts as a safety fallback, politely redirecting the customer to a physical human support phone line.

---

## 📦 Project Structure

```bash
├── evidence                         # Local environment variable configuration
├── .gitignore                    # Git file exclusion targets
├── agentcore_config.json         # Managed AgentCore deployment definition mappings
├── chat.py                       # CLI application testing terminal wrapper loop
├── cleanup_agentcore.py          # Utility script for programmatic asset teardown
├── cloudformation-testing.yaml   # Infrastructure for automated evaluations
├── cloudformation-tool.yaml      # Ticket store backend (DynamoDB & Lambda)
├── create_bug_report.py          # Lambda handler executing the ticket insertion
├── create_harness.py             # Automation entry script provisioning the Bedrock core
├── generate-eval-dataset.py     # Evaluation data compiler creating validation JSONL
├── harness-tests-template.json   # Base blueprint for configuring system assertions
├── harness-tests.json            # Final active evaluation test configurations
├── online_shop_faq.md            # Authoritative store support database source
├── README.md                     # Project documentation overview
├── requirements.txt              # Project library dependencies index
├── setup_gateway.py              # Proxy API Gateway provisioning connection link
└── system_prompt.txt             # Primary agent guardrails text configuration
```

---

## ⚙️ Prerequisites & Dependencies

* An active **AWS Account** deployed in **`us-east-1`** (North Virginia).
* Model access granted in the Amazon Bedrock Console for **Amazon Nova Models** (or equivalent LLMs).
* AWS CLI configured with local administrator or engineering credentials.
* **Python 3.9+** runtime environment equipped with the dependencies configured inside `requirements.txt`:

```bash
pip install -r requirements.txt
```

---

## 🚀 Execution & Implementation Steps

### Step 1: Provision System Core Infrastructure
Run the cloudformation stack along with deployment scripts to construct the database schema and provision parameters.

```bash
# 1. Deploy the background DynamoDB tool table 
aws cloudformation deploy \
  --template-file cloudformation-tool.yaml \
  --stack-name bug-report-tool-stack \
  --capabilities CAPABILITY_IAM \
  --region us-east-1

# 2. Deploy your agent architecture definition harness 
python create_harness.py

# 3. Establish routing endpoints via api gateway linking
python setup_gateway.py
```

### Step 2: Build and Configure the Bedrock Flow
Navigate to the Amazon Bedrock Console under **Orchestration > Flows** to build your canvas layout. Keep these platform constraints in mind:

* **Strict Conditions:** Bedrock Flow condition nodes rely on exact text evaluations. Enclose your conditional matching variables within parentheses (e.g., `(category == "PLATFORM_QUESTION")`) and ensure strings match your classifier output precisely.
* **Agent Slots:** Enable the "User Input" feature under Advanced Settings inside your Agent configuration node. This allows the model to ask follow-up questions if fields are missing.
* **Context Windows:** Embed the `online_shop_faq.md` markup text cleanly between `<faq_context>` boundaries inside the Prompt node to constrain Nova Pro's answer mapping.

### Step 3: Automated Testing & Model Evaluation
To test your application programmatically against your `harness-tests.json` file suite, execute the valuation processing pipeline:

```bash
# Execute evaluation matrix tests and export performance metrics dataset
python generate-eval-dataset.py
```
Import the generated JSONL dataset output directly into **Amazon Bedrock Evaluations** to execute automated LLM-as-a-judge accuracy metrics.

### Step 4: Local Client Verification
To verify the performance of the chatbot workflow interface locally within your command terminal window interface runtime:

```bash
python chat.py
```

---

## 🧹 Infrastructure Cleanup

To prevent ongoing AWS infrastructure costs from data storage or idle configurations, execute the cleanup stack script:

```bash
python cleanup_agentcore.py

aws cloudformation delete-stack --stack-name bug-report-tool-stack --region us-east-1
```

---

## 🛠️ Technology Stack

* **[Amazon Bedrock Flows](https://amazon.com):** Visual orchestration engine for generative logic.
* **[Amazon Bedrock Agents](https://amazon.com):** Action execution layer mapping intents to Python scripts.
* **[Amazon Bedrock Evaluations](https://amazon.com):** Algorithmic model response monitoring and verification.
* **[AWS Lambda](https://amazon.comlambda/):** Serverless computing runtime executing business tools.
* **[Amazon DynamoDB](https://amazon.comdynamodb/):** NoSQL persistence table serving as an issue ticket tracking repository.
