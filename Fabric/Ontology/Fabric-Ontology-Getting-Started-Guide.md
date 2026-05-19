# Microsoft Fabric Ontology — Customer Getting Started Guide

**Audience:** Fabric administrators, data platform leads, and analytics teams preparing a first Fabric Ontology (preview) deployment.
**Status:** Ontology and Fabric IQ are currently in **public preview**. Expect feature changes; do not use for production workloads with SLAs.
**Goal:** Stand up the tenant, prepare a workspace, and complete an end-to-end ontology walkthrough using the Lakeshore Retail sample.

---

## 1. What you are setting up

**Fabric IQ** is the governed knowledge-graph layer in Microsoft Fabric. Inside Fabric IQ, an **Ontology** defines your business vocabulary — entity types (e.g., Store, Product, SaleEvent), relationships, and rules — and binds them to live data in OneLake, semantic models, and Eventhouse. Once the ontology exists, you expose it to a **Fabric Data Agent** so business users can ask natural-language questions grounded in your real data.

In this walkthrough you will:

1. Confirm licensing and admin roles.
2. Turn on the required tenant settings.
3. Create a Fabric-enabled workspace and a Lakehouse with sample data.
4. Generate the ontology (from a Power BI semantic model **or** directly from OneLake).
5. Enrich it with streaming data, preview it as a graph, and front it with a data agent.

---

## 2. Prerequisites

| Requirement | Who handles it | Notes |
|---|---|---|
| Microsoft Fabric license / trial capacity | Fabric admin | A 60-day trial capacity works for the walkthrough. |
| Fabric administrator role | Tenant admin (Entra) | Required to flip tenant settings. The role is assigned to **individual users**, not security groups. |
| A workspace on Fabric-enabled capacity | Workspace admin | Single workspace used for all walkthrough items. |
| Sample data | The user running the walkthrough | CSVs from the `Ontology samples` GitHub folder. |

---

## 3. Tenant setup (Fabric admin)

### 3.1 Enable Microsoft Fabric on the tenant

If Fabric isn't already enabled:

1. Open the **Fabric admin portal** → **Tenant settings**.
2. Expand **Users can create Fabric items** → enable it (scope to a security group if you want a controlled rollout).
3. Apply. Allow ~15 minutes for propagation.

Reference: *Enable Microsoft Fabric for your organization* — https://learn.microsoft.com/en-us/fabric/admin/fabric-switch

### 3.2 Required tenant settings for Ontology (preview)

In **Admin portal → Tenant settings**, enable the following. Scope each to a security group during pilot:

- **Enable Ontology item (preview)**
- **Users can create Graph (preview)**
- **Users can create and share Data agent item types (preview)**
- **Users can use Copilot and other features powered by Azure OpenAI**
- **Data sent to Azure OpenAI can be processed outside your capacity's geographic region, compliance boundary, or national cloud instance**
- **Data sent to Azure OpenAI can be stored outside your capacity's geographic region, compliance boundary, or national cloud instance**

> The last two are required today because Fabric IQ relies on Azure OpenAI capacity that may be outside your tenant's geo. Confirm this is acceptable to your compliance, data residency, and legal teams before flipping it on. If your tenant has strict residency requirements, pause the rollout and engage your Microsoft account team.

Reference: *Ontology (preview) required tenant settings* — https://learn.microsoft.com/en-us/fabric/iq/ontology/overview-tenant-settings

### 3.3 Recommended governance settings to review

While you're in tenant settings, take a pass through these — they shape how safely the ontology and data agent can be shared:

- **Workspace creation** — restrict to a governance group.
- **Sensitivity labels** — turn on Purview Information Protection so labels carry through to Fabric items.
- **Preview features** — enable for the pilot security group only.

---

## 4. Workspace and sample data setup

Do this once per pilot environment.

### 4.1 Create the workspace

1. In the Fabric portal, **Workspaces → + New workspace**.
2. Name it (e.g., `Ontology-Pilot`).
3. Assign it to a **Fabric-enabled capacity** (trial capacity is fine).
4. Add the pilot team in the appropriate role (Admin / Member / Contributor / Viewer).

### 4.2 Create the Lakehouse and load sample data

1. From your workspace, **+ New item → Lakehouse**. Name it `OntologyDataLH`.
2. Download the sample CSVs from the **Ontology samples** GitHub folder linked in the tutorial (DimStore, DimProducts, FactSales, Freezer, FreezerTelemetry).
3. In the Lakehouse, **Get data → Upload files** and upload:
   - `DimProducts.csv`
   - `DimStore.csv`
   - `FactSales.csv`
   - `Freezer.csv`
   - *(Do **not** upload `FreezerTelemetry.csv` — it goes to Eventhouse later.)*
4. For each uploaded file, choose **… → Load to Tables → New table** and accept defaults.

### 4.3 (Path A only) Create the Power BI semantic model

Skip this section if you plan to build the ontology directly from OneLake.

1. From the Lakehouse ribbon, **New semantic model**.
2. Name it `RetailSalesModel`, leave your tutorial workspace selected, include the four loaded tables.
3. Open the model and confirm relationships look correct before generating the ontology.

---

## 5. Ontology walkthrough — choose your path

Microsoft offers two parallel paths through Tutorial Part 0 → Part 5. **Pick one** based on the customer's starting state.

### Path A — Generate from a Power BI semantic model
Use when the customer already has a well-curated star-schema semantic model that represents their business domain. Fastest path to a working ontology.

### Path B — Build directly from OneLake
Use when there is no semantic model yet, or when the customer wants full control over the ontology design.

Both paths share the same five tutorial steps below; switch the **scenario selector** at the top of each Learn page to your chosen path.

| # | Step | Learn link |
|---|---|---|
| 0 | Introduction & environment setup | https://learn.microsoft.com/en-us/fabric/iq/ontology/tutorial-0-introduction |
| 1 | Create the ontology | https://learn.microsoft.com/en-us/fabric/iq/ontology/tutorial-1-create-ontology |
| 2 | Enrich the ontology (bind streaming data from Eventhouse) | https://learn.microsoft.com/en-us/fabric/iq/ontology/tutorial-2-enrich-ontology |
| 3 | Preview the ontology as a graph | https://learn.microsoft.com/en-us/fabric/iq/ontology/tutorial-3-preview-ontology |
| 4 | Create a Fabric Data Agent on top | https://learn.microsoft.com/en-us/fabric/iq/ontology/tutorial-4-create-data-agent |
| 5 | Conclusion & cleanup | https://learn.microsoft.com/en-us/fabric/iq/ontology/tutorial-5-conclusion |

By the end you should be able to ask the data agent questions like *"What is the top product by revenue across all stores?"* and get a grounded answer.

---

## 6. Going beyond the tutorial

Once the sample works, point the customer at these how-to guides to apply the pattern to their own data:

- Create entity types — https://learn.microsoft.com/en-us/fabric/iq/ontology/how-to-create-entity-types
- Bind data to entity types — https://learn.microsoft.com/en-us/fabric/iq/ontology/how-to-bind-data
- Add relationship types — https://learn.microsoft.com/en-us/fabric/iq/ontology/how-to-create-relationship-types
- Use rules — https://learn.microsoft.com/en-us/fabric/iq/ontology/how-to-use-rules
- Resource links — https://learn.microsoft.com/en-us/fabric/iq/ontology/how-to-use-resource-links
- Ontology MCP server (for agent integrations) — https://learn.microsoft.com/en-us/fabric/iq/ontology/how-to-use-ontology-mcp-server

---

## 7. Reference & training

- **Fabric IQ documentation hub** — https://learn.microsoft.com/en-us/fabric/iq/
- **What is ontology (preview)?** — https://learn.microsoft.com/en-us/fabric/iq/ontology/overview
- **Capacity & usage considerations** — https://learn.microsoft.com/en-us/fabric/iq/ontology/resources-capacity-usage
- **Glossary** — https://learn.microsoft.com/en-us/fabric/iq/ontology/resources-glossary
- **Troubleshooting** — https://learn.microsoft.com/en-us/fabric/iq/ontology/resources-troubleshooting
- **Microsoft Learn training path — Get started with Microsoft Fabric** (includes the "Understand Microsoft Fabric IQ fundamentals" module) — https://learn.microsoft.com/en-us/training/paths/get-started-fabric/
- **Microsoft Fabric documentation root** — https://learn.microsoft.com/en-us/fabric/

---

## 8. Pre-flight checklist

Before the customer kicks off the walkthrough, confirm:

- [ ] Tenant has Fabric enabled (admin switch).
- [ ] Fabric trial capacity or paid SKU is assigned to the pilot workspace.
- [ ] All six Ontology / Data agent / Azure OpenAI tenant settings are enabled (Section 3.2).
- [ ] Compliance / data-residency sign-off on the "data processed/stored outside region" toggles.
- [ ] Pilot workspace created and the pilot team has roles assigned.
- [ ] `OntologyDataLH` Lakehouse exists with the four CSVs loaded as tables.
- [ ] Path decided: **Generate from semantic model** *or* **Build from OneLake**.
- [ ] Owner identified for each of Tutorial Parts 1–5 with a target completion date.

---

*Preview disclaimer: Fabric IQ and Ontology are in public preview. Settings, UI, and behavior may change. Re-check the Learn pages above before each session — they are the source of truth.*
