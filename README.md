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
                    │       Router             |
                    |___________________________|
                    │             │             │
                    ▼             ▼             ▼
              [BUG_REPORT] [PLATFORM_QUESTION] [OTHER_REQUEST]
                    │             │             │
        ┌───────────▼───┐  ┌──────▼────────┐    │
        │ BUG REPORT 
          HANDLER │        │ Prompt Node   │    │
        │ (Tool Use)    │  │ (Embedded FAQ)│    │
        └───────┬───────┘  └──────┬────────┘    │
                │                 │             │
         ┌──────▼───────┐  ┌──────▼────────┐  ┌─▼─────────────┐
         │ FlowOutput │    │ FlowOutput_2  │  │ FlowOutput_3  │
         │ (Bug Ticket) │  │ (FAQ Answer)  │  │ (Escalation)  │
         └──────────────┘  └───────────────┘  └───────────────┘
```

1. **Bug Reports:** Handled by an Amazon Bedrock Agent. The agent dynamically collects three mandatory parameters (Description, Steps to Reproduce, Environment) via multi-turn conversation and invokes an AWS Lambda function to persist the ticket into Amazon DynamoDB.
2. **Platform Questions:** Handled directly within a Flow Prompt Node. An authoritative e-commerce FAQ (`online_shop_faq.md`) is injected into the context window at inference time for deterministic, hallucination-free generation.
3. **Other Requests:** Acts as a safety fallback. Prompts the agent to politely redirect the customer to a physical human support phone line.

---

## 📦 Project Structure

```bash
├── cloudformation-testing.yaml   # Infrastructure for automated flow evaluations
├── cloudformation-tool.yaml      # Infrastructure for the DynamoDB/Lambda ticketing tool
├── create_bug_report.py          # Lambda handler executing the DynamoDB ticket storage
├── flow-tests-template.json      # Test suite development template
├── generate-eval-dataset.py     # Script executing programmatic tests to output evaluation JSONL
├── docs/
│   ├── testing.md               # Step-by-step automated evaluation runtime guide
│   └── tools-setup.md           # Step-by-step setup guide for the bug reporting ecosystem
└── solution/
    └── README.md                # Reference solution flow definition artifacts and diagrams
```

---

## ⚙️ Prerequisites & Dependencies

* An active **AWS Account** deployed in **`us-east-1`** (North Virginia).
* Model access granted in the Amazon Bedrock Console for **Amazon Nova Models** (or equivalent LLMs).
* AWS CLI configured with local administrator or engineering credentials.
* **Python 3.9+** runtime environment equipped with the latest `boto3` library version.

---

## 🚀 Execution & Implementation Steps

### Step 1: Deploy Tooling Infrastructure
Deploy the CloudFormation stack to provision the backend ticket store (DynamoDB table) and the invocation runtime (AWS Lambda function).

```bash
aws cloudformation deploy \
  --template-file cloudformation-tool.yaml \
  --stack-name bug-report-tool-stack \
  --capabilities CAPABILITY_IAM \
  --region us-east-1
```
*Follow the complete provisioning deep-dive inside [Tools Setup](docs/tools-setup.md).*

### Step 2: Build and Configure the Bedrock Flow
Navigate to the Amazon Bedrock Console under **Orchestration > Flows** to build your canvas layout. Keep these platform constraints in mind:

* **Strict Conditions:** Bedrock Flow condition nodes rely on exact text evaluations. Enclose your conditional matching variables within parentheses (e.g., `(category == "PLATFORM_QUESTION")`) and ensure strings match your classifier output precisely.
* **Agent Slots:** Enable the "User Input" feature under Advanced Settings inside your Agent configuration node. This allows the model to ask follow-up questions if reproduction fields are missing.
* **Context Windows:** Embed the `online_shop_faq.md` markup text cleanly between `<faq_context>` boundaries inside the Prompt node to constrain Nova Pro's answer mapping.
* **Dedicated Sink Mapping:** A single Output node cannot merge multi-branch inputs. Provision an explicit `FlowOutputNode` termination point for every unique resolution track.

### Step 3: Automated Testing & Model Evaluation
Manual playground chat verification is non-scalable. Transition to our programmatic execution pipeline to benchmark accuracy:

1. Populate your test interactions into `flow-tests-template.json`.
2. Execute the evaluation engine to query your active flow endpoint and format results:
   ```bash
   python generate-eval-dataset.py
   ```
3. Import the output dataset file into **Amazon Bedrock Evaluations** to execute automated LLM-as-a-judge accuracy metrics.
*Follow the testing instructions inside [Testing and Evaluation Docs](docs/testing.md).*

---

## 🧹 Infrastructure Cleanup

To prevent ongoing AWS infrastructure costs from data storage or idle configurations, tear down all active environments when finished:

```bash
aws cloudformation delete-stack --stack-name bug-report-tool-stack --region us-east-1
aws cloudformation delete-stack --stack-name bug-report-testing-stack --region us-east-1
```

---

## 🛠️ Technology Stack

* **[Amazon Bedrock Flows](https://amazon.com):** Visual orchestration engine for generative logic.
* **[Amazon Bedrock Agents](https://amazon.com):** Action execution layer mapping intents to Python scripts.
* **[Amazon Bedrock Evaluations](https://amazon.com):** Algorithmic model response monitoring and verification.
* **[AWS Lambda](https://amazon.comlambda/):** Serverless computing runtime executing business tools.
* **[Amazon DynamoDB](https://amazon.comdynamodb/):** NoSQL persistence table serving as an issue ticket tracking repository.
