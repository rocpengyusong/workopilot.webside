# Core Features Detailed Explanation

## 1. Companion Audit Workbench (Browser Plugin)

Auditors' daily work no longer faces our system backend, but continues to use SAP, Yonyou, or OA.

- **Auto Activation**: The plugin automatically recognizes the current URL (e.g., travel reimbursement page) and wakes up intelligently.
- **Traffic Light Mechanism**:
    - 🟢 **PASS**: Risk detection passed, suggested to release.
    - 🔴 **BLOCK**: Blocking risk found (e.g., invoice header error), submission prohibited.
    - 🟡 **WARN**: Warning risk found (e.g., accommodation fee slightly exceeded), manual review required.
- **Evidence Tracing**: Click on risk items to directly pop up a comparison chart of **"Invoice Slice vs Reported Data"**, making AI's judgment basis clear at a glance.

## 2. Smart Audit Strategy Studio

The "Brain" construction factory of the system, designed for strategy administrators.

### Rule Configuration IDE
A low-code configuration environment similar to VSCode:
- **Prompt Engineering**: Directly write natural language instructions, supporting variable insertion (e.g., `{{invoice_amt}}`).
- **Dynamic Data Source**: Support checking **Attachment Classification Whitelist** (e.g., only extract "Hotel Bill"), precisely controlling AI attention.
- **Tool Mounting**: Mount **MCP Tools** for rules (e.g., exchange rate query, enterprise business query).

### Full-Link Simulation Sandbox
Perform 100% real simulation testing before rule release:
- **Simulated Input**: Upload real attachment images, edit simulated document JSON.
- **Perspective Running**: View AI's Chain of Thought (CoT), Token consumption, and intermediate variable snapshots.

## 3. Hybrid Policy Engine

Turn the enterprise's "dead regulations" into "living data".

- **Structured Parameters**: For rigid indicators (e.g., Beijing, Level M3, limit 800 yuan), AI automatically extracts them as structured data for precise mathematical comparison.
- **RAG Semantic Retrieval**: For flexible principles (e.g., whether it meets the necessity of business entertainment), use vector retrieval technology to find original text basis from PDF policy documents.

## 4. Capabilities & Integration Center (Capabilities Hub)

- **Attachment Classification Center**: Support OCR+LLM or pure Vision Model routes, supporting structured splitting of multi-page contracts (Cover/Body/Signature Page).
- **Tool Ecosystem**:
    - **Native Tools**: System built-in high-performance C# methods.
    - **MCP Services**: Compatible with Model Context Protocol, easily connecting to external Python/Node.js services.
