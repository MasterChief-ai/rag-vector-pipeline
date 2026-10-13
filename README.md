# rag-vector-pipeline - RAG Document Vectorization Pipeline

## Overview
This repository contains the `rag wdf.json` workflow file, built for the n8n automation platform. This workflow serves as the foundational data ingestion layer for a Retrieval-Augmented Generation (RAG) system. 

The purpose of this automation is to take a static document stored in Google Drive and transform it into a searchable, mathematical format (vectors) so that Large Language Models (LLMs) can "read" and query the document to answer user questions. 

## Core Tools and Integrations
This workflow orchestrates several specialized tools to process the data:
* **n8n:** The visual automation platform managing the data flow.
* **Google Drive:** The source repository where the raw document is stored.
* **LangChain Integration:** Utilized within n8n to handle complex AI data operations, including document loading and text splitting.
* **Cohere:** The embedding model provider used to convert human-readable text into vector arrays.
* **Pinecone:** The vector database used to store and index the generated embeddings for fast retrieval.

## Detailed Workflow Breakdown

The system executes a linear process to ingest, process, and store the document.

### 1. Manual Execution
* **Trigger (`n8n-nodes-base.manualTrigger`):** This workflow does not run on a schedule or via a webhook. It is designed to be triggered manually by clicking the "Execute workflow" button within the n8n interface. This is typical for batch processing or updating a knowledge base on demand.

### 2. Document Acquisition
* **File Download (`n8n-nodes-base.googleDrive`):** Once triggered, the workflow connects to Google Drive via OAuth2. It specifically targets and downloads a file named `basic-documents-47-en.pdf` using its unique file ID (`16C8v8VoRlyGhQj1VsiEGf53rLOnsdYDm`).

### 3. Data Processing and Vectorization
The final step occurs within the Pinecone Vector Store node, which relies on three sub-components connected via LangChain data links:
* **Data Loading (`@n8n/n8n-nodes-langchain.documentDefaultDataLoader`):** The downloaded PDF is passed into this node as binary data. It extracts the raw text from the file.
* **Text Splitting (`@n8n/n8n-nodes-langchain.textSplitterRecursiveCharacterTextSplitter`):** LLMs cannot process massive documents all at once. This node breaks the extracted text down into smaller, overlapping semantic chunks (using recursive character splitting) to maintain context.
* **Embedding Generation (`@n8n/n8n-nodes-langchain.embeddingsCohere`):** Each text chunk is sent to the Cohere API. Specifically, it uses the `embed-english-v3.0` model to convert the text chunks into high-dimensional vector embeddings.

### 4. Vector Storage
* **Database Insertion (`@n8n/n8n-nodes-langchain.vectorStorePinecone`):** Finally, the generated vector embeddings, along with their associated text chunks, are pushed to a Pinecone vector database. The workflow is configured to insert these vectors into an index named `whoonindex` under the specific namespace `who`.

## Installation and Deployment

To utilize this workflow, you must import it into an active n8n instance and configure your API credentials.

### 1. Importing the Workflow
1. Log into your n8n workspace.
2. Create a new workflow.
3. Open the `rag wdf.json` file, copy all the JSON text, and paste it directly onto the n8n canvas.

### 2. Configuring Credentials
You will need to authorize n8n to connect to the external services. Open the respective nodes and supply your credentials:
* **Google Drive:** Open the "Download file" node and connect your Google account using OAuth2. Ensure the connected account has access to the target file.
* **Cohere:** Open the "Embeddings Cohere" node and input your Cohere API key.
* **Pinecone:** Open the "Pinecone Vector Store" node and input your Pinecone API key and environment details.

### 3. Customization
If you wish to use this workflow for your own documents, you must update the target file and database locations:
1. **Google Drive Node:** Update the File ID in the "Download file" node to point to the specific PDF you want to process.
2. **Pinecone Node:** Update the `Pinecone Index` and `Pinecone Namespace` fields to match your Pinecone project architecture.