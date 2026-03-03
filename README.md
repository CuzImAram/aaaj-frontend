# Agent-as-a-Judge: Frontend

This repository contains the web-based user interface for the **[Agent-as-a-Judge](https://github.com/CuzImAram/agent-as-a-judge)** framework. It provides a visual platform to interact with the Pairwise Comparison System, allowing researchers to evaluate RAG responses through a structured, multi-agent pipeline.

## ✨ Key Features

* **Dual Input Modes**: Manually populate query, reference, and response fields or upload JSON files directly in the **CrowdRAG-25** schema.


* **Interactive Citation Tracking**: The interface automatically detects bracketed citations (e.g., `[1]`). Valid citations are highlighted in **blue** and act as hyperlinks to the source text; hallucinated or non-existent citations are flagged in **red**.


* **Context Visualization**: Toggle between "List View" and "Raw Text" for reference passages to better analyze how the LLM grounded its answer.


* **Granular Analysis Reports**: Once the backend agents finish processing, the UI displays detailed "Critique Reports" for every utility dimension, highlighting "Key Issues" like factual errors or missing information.


* **Adjudication Dashboard**: View the final head-to-head verdict, including the overall winner and a logical explanation for which model performed better in each specific dimension.

## 🚀 Getting Started

### Prerequisites

* [Node.js](https://nodejs.org/)
* A running **Backend instance** from the **[Agent-as-a-Judge-Backend](https://github.com/CuzImAram/aaaj-backend)**.



### Installation

1. **Clone the repository**:
```bash
git clone https://github.com/CuzImAram/aaaj-frontend.git
cd aaaj-frontend

```


2. **Install dependencies**:
```bash
npm install

```


3. **Environment Setup**:
Create a `.env` file in the root directory and add your n8n webhook URL:
```env
VITE_N8N_WEBHOOK_URL=http://your-n8n-instance/webhook/your-endpoint

```



### Execution

Run the development server:

```bash
npm run dev

```

The application will be accessible at `http://localhost:5173`.
