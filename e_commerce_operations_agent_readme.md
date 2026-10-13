# Autonomous E-Commerce Operations & Escalation AI Agent

An automated end-to-end business operations agent built on **n8n**, integrating **Airtable**, **Supabase Vector Store (RAG)**, **Google Gemini**, and **Gmail**. The system automates operational decision-making, escalates courier delays, checks company policy compliance, and dispatches executive summaries directly to stakeholders.

---

## 📌 Architecture & Data Flow

```text
[Manual/Scheduled Trigger]
           │
           ▼
[Airtable: customer_orders] ──► Fetch live transactional orders
           │
           ▼
[Code Node (JavaScript)]    ──► Batch multiple order items into a unified payload
           │
           ▼
[n8n AI Agent (Tools Agent)]
   ├── Model: Google Gemini Chat Model (gemini-1.5-flash)
   └── Tool: Supabase Vector Store (store_policy_lookup)
           │     └── Embeddings: Google Gemini Embeddings (text-embedding-004)
           │
           ▼
[Gmail Node]                 ──► Dispatch executive operations escalation report
```

---

## 🚀 Key Features

- **Automated Order Ingestion:** Ingests live transactional records from Airtable (Customer details, Tracking ID, Courier, Transit Days, Complaint).
- **RAG-Powered Policy Verification:** Queries an enterprise policy vector database in Supabase (`documents` table) using Gemini embeddings to evaluate return, damage, and delivery SLAs.
- **Autonomous Incident Classification:**
  - Evaluates delivery delays (`days_in_transit > 5`) and triggers priority flags for underperforming couriers.
  - Verifies damage complaints against unboxing video policies.
  - Generates professional customer reply drafts for standard tracking queries.
- **Automated Reporting:** Generates an executive-grade operational briefing and emails it instantly via Gmail.

---

## 🛠️ Tech Stack

- **Workflow Automation:** n8n
- **Database / CRM:** Airtable
- **Vector Database (Knowledge Base):** Supabase (PostgreSQL with `pgvector`)
- **LLM & Embeddings:** Google Gemini (`gemini-1.5-flash`, `text-embedding-004`)
- **Notification Action:** Gmail API / OAuth2

---

## 📋 Table Schema (Airtable: `customer_orders`)

| Field Name | Type | Description |
| :--- | :--- | :--- |
| `order_id` | Single line text | Unique Order Identifier (e.g., `VR-101`) |
| `customer_name` | Single line text | Customer full name |
| `customer_email` | Email | Customer contact email |
| `order_status` | Single select | Status (`In Transit`, `Delayed`, `Delivered`) |
| `courier_name` | Single select | Delivery partner (`Trax`, `Leopards`) |
| `tracking_id` | Single line text | Courier tracking number |
| `days_in_transit` | Number | Total elapsed transit duration |
| `order_value` | Number | Order value in PKR |
| `customer_complaint` | Long text | Customer inquiry or issue logged |

---

## ⚙️ Setup & Deployment

### 1. Supabase Policy Vector Store
Ensure the `documents` table has vector search enabled with Gemini embeddings for return and transit policies:
```sql
CREATE EXTENSION IF NOT EXISTS vector;

CREATE TABLE documents (
  id BIGSERIAL PRIMARY KEY,
  content TEXT,
  metadata JSONB,
  embedding VECTOR(768)
);
```

### 2. Airtable Setup
- Create base and populate the `customer_orders` table with active operational data.
- Generate an Airtable Personal Access Token (PAT) with `data.records:read` permissions.

### 3. n8n Configuration
1. Import `workflow.json` into n8n.
2. Link your credentials:
   - **Airtable Personal Access Token account**
   - **Google Gemini API** (for chat model & embeddings)
   - **Supabase Vector Store**
   - **Gmail OAuth2**
3. Configure the **AI Agent System Message** for strict SLA and priority logic.
4. Execute the workflow to test automated reporting.

---

## 📊 Sample Output (Dispatched Executive Report)

```text
[URGENT] Daily Operations & Escalation Report - E-Commerce Ops

1. Executive Summary:
- Total Orders Scanned: 4
- Critical Courier Escalations: 2 (Trax)
- Damaged Parcel Investigations: 1 (Leopards)
- Standard Inquiries: 1

2. Order Breakdown & Decisions:
- VR-101 (Hamza Khan): Normal transit window. Tracking link draft prepared.
- VR-102 (Ayesha Malik): SLA breach (6 days in transit). Escalated to Trax operations lead.
- VR-103 (Bilal Ahmed): Damaged item report. Unboxing video proof requested per 24-hr policy.
- VR-104 (Zainab Tariq): Extreme delay (8 days). Refund review initiated, Trax investigation opened.

3. Logistics Alert:
- Courier 'Trax' shows recurring transit bottlenecks (>5 days SLA violations). Immediate logistics partner review recommended.
```