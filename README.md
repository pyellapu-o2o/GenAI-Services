# GenAI Platform

A modular, scalable platform for developing workflows using generative AI. Built with AWS Lambda, Step Functions, and supporting multiple AI providers.

## Table of Contents

  - [Architecture Overview](https://www.google.com/search?q=%23%EF%B8%8F-architecture-overview)
  - [Core Services](https://www.google.com/search?q=%23-core-services-modular-lambdas)
  - [Business Workflows](https://www.google.com/search?q=%23-business-workflows)
  - [Setup and Deployment](https://www.google.com/search?q=%23%EF%B8%8F-setup-and-deployment)
  - [Testing](https://www.google.com/search?q=%23-testing)
  - [Monitoring and Logging](https://www.google.com/search?q=%23-monitoring-and-logging)
  - [Development Guide](https://www.google.com/search?q=%23-development-guide)
  - [Usage Examples](https://www.google.com/search?q=%23-usage-examples)
  - [Performance Considerations](https://www.google.com/search?q=%23-performance-considerations)
  - [Security](https://www.google.com/search?q=%23-security)
  - [Troubleshooting](https://www.google.com/search?q=%23-troubleshooting)
  - [Support](https://www.google.com/search?q=%23-support)
  - [License](https://www.google.com/search?q=%23-license)
  - [Roadmap](https://www.google.com/search?q=%23-roadmap)

-----

## 🏗️ Architecture Overview

The platform is designed with a modular microservices architecture, orchestrated by AWS Step Functions. This allows for flexible and scalable workflow creation.

```
genai-platform/
├── core-services/           # Generic, reusable microservices
│   ├── llm-chat-service/    # Multi-provider LLM interface
│   ├── data-validator-service/  # LLM-as-judge validation
│   ├── image-generator-service/ # Template-based image generation
│   ├── video-generator-service/ # Video creation interface
│   └── storage-service/         # Multi-cloud storage abstraction
├── workflow-orchestrator/   # Step Functions state machine
├── business-workflows/      # Domain-specific implementations
│   └── fund-factsheet-video/ # Example: Fund factsheet to video
└── shared-layers/           # Common utilities and infrastructure
    ├── aws-infra/           # AWS-specific implementations
    ├── utils/               # Shared utilities
    └── types/               # Common type definitions
```

-----

## 🚀 Core Services (Modular Lambdas)

These are standalone, reusable Lambda functions that perform specific GenAI tasks.

1.  **LLM Chat Service**

      * **Purpose**: Provides a unified interface for multiple AI providers (e.g., Bedrock, VertexAI, OpenAI).
      * **Input**: Prompt, optional file, response type, model configuration.
      * **Output**: Structured response in the requested format.
      * **Example Payload**:
        ```json
        {
          "provider": "bedrock",
          "model_id": "anthropic.claude-v2",
          "prompt": "Extract fund performance data",
          "file_reference": "s3://input-bucket/documents/fund-report.pdf",
          "response_type": "json",
          "temperature": 0.2
        }
        ```

2.  **Data Validator Service**

      * **Purpose**: Implements an "LLM-as-judge" pattern for consensus-based validation using multiple models.
      * **Input**: Data to validate, validation criteria, and model configurations.
      * **Output**: A consensus validation result with a confidence score.

3.  **Image Generator Service**

      * **Purpose**: Renders Jinja2 templates into various output formats.
      * **Input**: Template name, data payload, and output type (e.g., `png`, `jpg`, `html`).
      * **Output**: A rendered image or HTML file.

4.  **Video Generator Service**

      * **Purpose**: Acts as a unified interface for video generation APIs (e.g., Personate, Synthesia).
      * **Input**: Script, visual assets, and configuration details.
      * **Output**: A video job ID and its current status.

5.  **Storage Service**

      * **Purpose**: Provides a multi-cloud storage abstraction layer (e.g., for AWS S3, Google Cloud Storage).
      * **Input**: Content, storage path, and metadata.
      * **Output**: A URL to the stored location.

-----

## 📋 Business Workflows

Business workflows are AWS Step Functions state machines that orchestrate calls to core services to achieve a specific business outcome.

### Fund Factsheet Video Generation

This workflow automates the creation of a video summary from a fund factsheet PDF.

**Steps:**

1.  `extract_factsheet_data`: Extracts data from the PDF using an LLM.
2.  `validate_extracted_data`: Validates the extracted data using multi-model consensus.
3.  `generate_summary_script`: Generates a video script from the validated data.
4.  `generate_summary_slide`: Creates a visual slide to accompany the script.
5.  `generate_thumbnail`: Generates a video thumbnail.
6.  `start_video_generation`: Initiates the video rendering process via an external API.
7.  `check_video_status`: Monitors the rendering progress.
8.  `save_video_and_metadata`: Stores the final video and associated metadata.

-----

## 🛠️ Setup and Deployment

### Prerequisites

  * AWS Account with appropriate permissions
  * Python 3.9+
  * Node.js (for AWS CDK deployment)
  * Docker (for local testing)

### Environment Setup

1.  **Clone the repository:**
    ```bash
    git clone <repository-url>
    cd genai-platform
    ```
2.  **Install dependencies:**
    ```bash
    pip install -r requirements.txt
    ```
3.  **Set up environment variables:**
    ```bash
    cp .env.example .env
    # Edit the .env file with your specific configuration
    ```

### Configuration

Prompts, API keys, and workflow configurations are stored in a DynamoDB table named `VideoPipelineConfig`.

**Example Configuration:**

```json
{
  "pipeline_type": "fund_factsheet_video",
  "config_data": {
    "llm_provider": "bedrock",
    "extraction_model": "anthropic.claude-v2",
    "extraction_prompt": "Extract the following fields from the factsheet...",
    "validation_models": [
      {"provider": "bedrock", "model_id": "claude-v2"},
      {"provider": "vertexai", "model_id": "gemini-pro"}
    ],
    "consensus_threshold": 0.7,
    "output_bucket": "your-video-output-bucket"
  }
}
```

### Deployment

Use the AWS CDK to deploy the stacks.

```bash
# Deploy core services
cd infrastructure
cdk deploy CoreServicesStack

# Deploy business workflow
cdk deploy FundFactsheetWorkflowStack

# Deploy shared layers
cdk deploy SharedLayersStack
```

-----

## 🧪 Testing

### Unit Tests

Run tests for each module and check for code coverage.

```bash
# Test core services
pytest tests/core-services/ --cov=core-services

# Test business workflows
pytest tests/business-workflows/ --cov=business-workflows

# Test shared utilities
pytest tests/shared-layers/ --cov=shared-layers
```

### Integration Tests

Test the end-to-end workflow execution.

```bash
# Test complete workflow
python -m tests.integration.test_fund_factsheet_workflow

# Test with mock services
python -m tests.integration.test_with_mocks
```

### Local Development

Use Docker Compose to spin up a local development environment.

```bash
# Start local environment
docker-compose up -d

# Run a workflow locally
python -m local_runner --workflow fund_factsheet_video --document sample.pdf
```

-----

## 📊 Monitoring and Logging

### CloudWatch Dashboards

  * **GenAI-Workflow-Execution**: Tracks workflow success rates, failures, and execution times.
  * **LLM-Service-Metrics**: Monitors LLM provider performance, latency, and token usage.
  * **Video-Generation-Metrics**: Tracks video rendering times and success rates from third-party APIs.

### Logging Structure

Logs are structured as JSON for easy parsing and analysis.

```json
{
  "level": "INFO",
  "timestamp": "2024-01-15T10:30:00Z",
  "workflow_id": "wf_123",
  "step_name": "extract_factsheet_data",
  "execution_time": 2.5,
  "data_points": 15,
  "model_used": "claude-v2"
}
```

-----

## 🔧 Development Guide

### Adding a New Core Service

1.  Create a new service directory in `core-services/new-service/`.
2.  Implement the Lambda handler with standardized input/output schemas.
3.  Add unit tests in `tests/core-services/test_new_service.py`.
4.  Update the CDK infrastructure in `infrastructure/core_services_stack.py` to deploy the new service.

### Creating a New Business Workflow

1.  Create a new workflow directory in `business-workflows/new-workflow/`.
2.  Implement step classes that extend the `BaseWorkflowStep`.
3.  Define the Step Functions state machine logic.
4.  Add the workflow configuration to the DynamoDB table.
5.  Create end-to-end integration tests for the new workflow.

### Adding a New AI Provider

1.  Implement the provider logic in `shared-layers/llm_providers/new_provider.py`.
2.  Ensure the new class extends the `LLMProvider` base class.
3.  Register the new provider in the `LLMProviderFactory`.
4.  Update the configuration schema to support the new provider's models and settings.

-----

## 🚦 Usage Examples

### Direct Lambda Invocation

You can invoke core services directly for specific tasks.

```python
import boto3
import json

lambda_client = boto3.client('lambda')

# Call the LLM service
response = lambda_client.invoke(
    FunctionName='llm-chat-service',
    Payload=json.dumps({
        "provider": "bedrock",
        "model_id": "anthropic.claude-v2",
        "prompt": "Extract fund performance data",
        "response_type": "json"
    })
)
```

### Starting a Workflow

Start a business workflow by invoking the orchestrator.

```python
# Start the fund factsheet workflow
response = lambda_client.invoke(
    FunctionName='workflow-orchestrator',
    Payload=json.dumps({
        "workflow_type": "fund_factsheet_video",
        "document_id": "doc_123",
        "business_context": {
            "fund_id": "fund_456",
            "user_id": "user_789"
        }
    })
)
```

-----

## 📈 Performance Considerations

  * **Cold Start Mitigation**: Use provisioned concurrency for critical Lambdas, Lambda layers for shared dependencies, and keep package sizes minimal.
  * **Scaling**: Core services are designed for high concurrency (1000+), while workflow steps are set for moderate concurrency (100+). Video generation scaling is limited by external API rate limits.
  * **Cost Optimization**: Use Spot Instances for non-critical workloads, S3 Intelligent-Tiering for storage, and configure CloudWatch log retention policies.

-----

## 🔒 Security

  * **IAM Roles**: Follow the principle of least privilege with service-specific roles and cross-account access policies where necessary.
  * **Data Protection**: Data is encrypted at rest (AWS KMS) and in transit (TLS 1.2+). API keys and other secrets are stored securely in AWS Secrets Manager.
  * **Compliance**: The platform is designed to be GDPR-ready with audit logging and S3 access logging enabled.

-----

## 🆘 Troubleshooting

### Common Issues

  * **Cold starts**: If latency is an issue, enable provisioned concurrency on the affected Lambda function.
  * **Timeout errors**: Increase the Lambda timeout settings in the CDK stack definition.
  * **Permission errors**: Verify that the IAM roles associated with the functions have the necessary permissions.

### Debug Mode

Enable detailed logging for local development and debugging.

```bash
# Enable debug logging
export LOG_LEVEL=DEBUG
export DEBUG_MODE=true

# Run the local runner with detailed logging
python -m local_runner --debug
```

-----

## 📞 Support

  * **Documentation**: [Confluence Link](https://investo2o.atlassian.net/wiki/x/DABUmw)
  * **Slack Channel**: `#genai-video-pipeline`
  * **JIRA Project**: `GENAI`

-----

## 📄 License

This project is licensed under the

-----

## 🎯 Roadmap

  * [ ] Multi-region deployment
  * [ ] Advanced monitoring with Datadog integration
  * [ ] A marketplace for custom image/video templates
  * [ ] Real-time progress updates for workflows (e.g., via WebSockets)
