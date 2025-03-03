# N8NNexus - AI-Powered Workflow Generator

N8NNexus is an intelligent automation tool that creates n8n workflows based on natural language input. By leveraging OpenAI models and a knowledge base of workflow templates stored in Pinecone, it can generate functional n8n workflows tailored to your requirements.

## Overview

The system consists of two main components:

1.  **Primary Workflow (aigen)**: Handles user input through a chat interface and passes requirements to the subflow
2.  **Secondary Workflow (AgentCreater\_Sub)**: Performs the complex workflow generation, using Pinecone to retrieve similar templates and OpenAI to craft a customized solution

## Features

*   **Natural Language Input**: Describe your desired workflow in plain English
*   **Template-Based Learning**: Leverages a database of 500+ sample workflows for reference
*   **AI-Powered Generation**: Uses GPT-4o and other OpenAI models for workflow creation
*   **Immediate Deployment**: Returns a usable n8n workflow link upon completion

## Prerequisites

*   n8n account (self-hosted or cloud)
*   OpenAI API key
*   Pinecone API key
*   Google Colab access (for data preparation)

## Flow Architecture

![Primary Flow Architecture Image](Screenshot/1.png)

![Secondary Flow Architecture Image](Screenshot/2.png)

The image shows the complete flow from input trigger through embedding, template matching, workflow generation, and validation.
## Setup Instructions

### 1\. Create Required API Accounts and Keys

#### OpenAI API Key

1.  Sign up at [https://platform.openai.com/](https://platform.openai.com/)
2.  Navigate to API keys section
3.  Create a new secret key and save it securely
4.  Ensure you have access to GPT-4o models and embeddings models

#### Pinecone API Key

1.  Create an account on [https://www.pinecone.io/](https://www.pinecone.io/)
2.  Create a new project
3.  Select the "Starter" plan (or higher)
4.  Choose "us-east-1" as your region
5.  Get your API key from the dashboard

#### n8n Account

1.  Set up either a cloud n8n instance at [https://app.n8n.cloud/](https://app.n8n.cloud/) or self-host
2.  For self-hosting, follow the installation guide at [https://docs.n8n.io/hosting/](https://docs.n8n.io/hosting/)
3.  Set up Admin user for API access

### 2\. Import Workflows to n8n

1.  Download the JSON workflow files from the repository:
    *   `AgentCreator_Primary.json` (Primary workflow)
    *   `AgentCreater_Secondary.json` (Secondary workflow)
2.  Import workflows:
    *   Navigate to Workflows section in your n8n instance
    *   Click "Import from File"
    *   Select the downloaded JSON files one by one
    *   Save each workflow after import
3.  Set up credentials:
    *   Go to Settings → Credentials
    *   Create OpenAI credentials:
        *   Name: "OpenAi account"
        *   API Key: Your OpenAI API key
    *   Create Pinecone credentials:
        *   Name: "PineconeApi account"
        *   API Key: Your Pinecone API key
        *   Environment: "us-east-1"
    *   Create n8n API credentials (if needed for programmatic workflow creation)

### 3\. Prepare and Upload Workflow Templates to Pinecone

1.  **Create Workflows Repository**:
    *   Create a folder named `n8n Workflows` in your Google Drive
    *   Fill it with n8n workflow export files in TXT or JSON format
    *   You can export workflows from n8n by clicking the "Export" button in workflow editor
2.  **Prepare the Colab Environment**:
    *   Open the provided Colab notebook (`n8nNexus_Pinecone_Uploader.ipynb`)
    *   Click Runtime → Change runtime type → Select GPU for faster processing
    *   Install required packages:
        
        python
        
        `!pip install pinecone tiktoken`
        
3.  **Mount Google Drive**:
    *   Run the cell to mount your Google Drive:
        
        python
        
        `from google.colab import drive drive.mount('/content/drive')`
        
4.  **Configure API Keys**:
    *   Update the keys and index name in the notebook:
        
        python

        `PINECONE_API_KEY = "YOUR-PINECONE-API-KEY" PINECONE_REGION = "us-east-1" OPENAI_API_KEY = "YOUR-OPENAI-API-KEY" INDEX_NAME = "n8n-templates"`
        
5.  **Create and Configure Pinecone Index**:
    *   The notebook will create a new Pinecone index if it doesn't exist
    *   Ensure the dimension is set to 3072 (matching text-embedding-3-large)
    *   The index uses cosine similarity for best matching
6.  **Process and Upload Workflows**:
    *   The notebook processes each workflow file:
        *   Generates a descriptive title
        *   Creates a TLDR summary
        *   Chunks text if needed (based on token count)
        *   Creates embeddings using OpenAI
        *   Uploads vectors to Pinecone with metadata
    *   Progress is saved in checkpoints to allow resuming if needed
7.  **Run the Entire Notebook**:
    *   Execute all cells to process and upload your workflow templates
    *   The process may take time depending on the number of workflows

### 4\. Configure the Workflows in n8n

#### Primary Workflow (aigen)

1.  Open the imported primary workflow
2.  Configure the Chat Trigger node:
    *   Enable "Public" if you want it accessible without authentication
    *   Set any other options like custom appearance
3.  Configure the OpenAI Chat Model node:
    *   Ensure it's using the correct OpenAI credentials
    *   Select GPT-4o-mini or GPT-4o as the model
    *   Set temperature and other parameters as needed
4.  Configure the Create AI Agent node:
    *   Connect it to the "AgentCreater\_Sub" workflow
    *   Ensure the workflowId parameter points to the correct workflow
5.  Save and activate the workflow

#### Secondary Workflow (AgentCreater\_Sub)

1.  Open the imported secondary workflow
2.  Configure all OpenAI nodes:
    *   Ensure they're using the correct OpenAI credentials
    *   Check that they're using the proper models
3.  Configure the Pinecone Vector Store nodes:
    *   Set the correct Pinecone index name ("n8n-templates")
    *   Confirm they're using the correct Pinecone credentials
    *   Check embeddings dimension matches OpenAI's text-embedding-3-large (3072)
4.  Configure the n8n node:
    *   Set up with the proper n8n API credentials
    *   Ensure it has permission to create workflows
5.  Save and activate the workflow

### 5\. Run the System

1.  Access the chat interface:
    *   Navigate to your n8n instance
    *   Go to Chat (if using cloud n8n) or access the webhook URL (if self-hosted)
2.  Describe your desired workflow in natural language:
    *   Be specific about the inputs, processes, and outputs you want
    *   Include information about any external services to integrate
3.  Review the generated workflow:
    *   The system will process your request
    *   It will return a link to the newly created workflow
    *   You can then open, modify, and activate this workflow

## How It Works

1.  **Input Processing**:
    *   User request is processed by the OpenAI model
    *   Keywords are extracted to search for similar workflows
2.  **Template Retrieval**:
    *   The system finds the most relevant workflow templates in Pinecone
    *   It combines multiple chunks if the template is large
3.  **Workflow Generation**:
    *   A structured plan is created based on the requirements
    *   The plan is enhanced with JSON snippets from example workflows
    *   A complete n8n workflow JSON is generated
4.  **Validation and Deployment**:
    *   The generated JSON is validated for correctness
    *   The workflow is created in your n8n instance
    *   A link to the new workflow is returned

## Example Usage

Input:



`Create an agent that monitors a Slack channel for specific keywords, then uses OpenAI to summarize the conversation and sends a daily report via email.`

Output:


`I've created a workflow based on your requirements! You can access it here: https://your-n8n-instance.com/workflow/ABC123 This workflow: 1. Monitors a Slack channel for specified keywords 2. Collects and processes conversation history 3. Uses OpenAI to generate concise summaries 4. Sends a formatted daily report via email`

## Troubleshooting

### Common Issues

1.  **Pinecone Connection Errors**:
    *   Verify your API key is correct
    *   Ensure the region matches your Pinecone project
    *   Check that the index exists and has the correct dimension
2.  **OpenAI API Errors**:
    *   Verify your API key
    *   Check usage limits on your OpenAI account
    *   Ensure you have access to the models being used
3.  **Workflow Generation Issues**:
    *   Look for error messages in n8n execution history
    *   Verify the JSON structure in the Validate JSON node
    *   Check that all nodes in the workflow are properly connected
4.  **Empty Results in Pinecone**:
    *   Ensure your workflows folder contains valid n8n workflow exports
    *   Check the processed\_files list in the checkpoint file
    *   Verify embeddings were created successfully

## Contributing

Contributions are welcome! Please consider:

*   Adding more template workflows to improve the knowledge base
*   Enhancing the workflow generation process
*   Improving documentation and examples

## License

This project is licensed under the MIT License - see the LICENSE file for details.
