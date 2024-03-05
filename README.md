# RAG App Root Repository

## Overview

This repository serves as the root for a multi-service application designed to automate the extraction, processing, and querying of PDF documents from the Reserve Bank of India's notifications. It utilizes Docker and Docker Compose to orchestrate multiple services, including a PDF extractor, text embedding generator, and a query service, alongside Redis for caching. 

## Getting Started

### Prerequisites

- Docker and Docker Compose installed on your system.
- Git for cloning the repository and its submodules.

### Clone the Repository

To clone the repository and all associated submodules in a single command, use:

```sh
git clone --recurse-submodules <repository-url>
```

Replace `<repository-url>` with the actual URL of this root repository. This command ensures that you retrieve the entire project structure, including each of the service submodules.

## Configuration
Before running the services, ensure you create `.env` files for each service at the individual repository level, according to the specific needs of each service. These `.env` files contain environment variables critical for the operation of the services, such as API keys, file paths, and configuration settings.

### PDF Extractor Service

- **Location**: `./pdf-extractor/.env`
- **Contents**: Must include `CHROMEDRIVER_PATH`, `PDF_STORAGE_PATH`, `START_YEAR`, and `YEAR_RANGE`.

### Embedding Generator Service
- **Location**: `./embedding-gen/.env`
- **Contents**: Must include `START_PIPELINE`.
### **Important Note**: 
The Embedding Generator Service's indexing takes about 15 minutes due to the large FastEmbed model download. This model ensures embeddings have a dimension of 1024, vital for search accuracy and document retrieval efficiency.


### Query Service
- **Location**: `./query-service/.env`
- **Contents**: Must include `COHERE_API_KEY`.

**NOTE:** For the query service, ensure your `.env` file includes the following environment variables, crucial for its operation:

```
COHERE_API_KEY=<cohere_api_key>
FAISS_INDEX_PATH=/usr/src/app/data/indices/combined_index.index
FAISS_METADATA_PATH=/usr/src/app/data/indices/metadata.json
REDIS_URL=redis://redis:6379
```
## Running the Services
Use Docker Compose to orchestrate the services. The docker-compose.yml file references all submodules and services, ensuring they are networked and started in the correct sequence.

Start the PDF Extractor Service: This service should run first to ensure that PDFs are extracted and available for processing.

```sh
docker-compose build pdf-extractor
docker-compose up pdf-extractor
```
Start the Embedding Generator Service: After PDF extraction, process the documents to generate embeddings.

```sh
docker-compose build embedding-gen
docker-compose up embedding-gen
```
Start the Fullstack Application: With embeddings generated and PDFs extracted, the query service can now provide endpoint access to search and summarize documents and this would be used directly by the fullstack application with proper UI and server implementation.

### Note: Please enter the API key in the UI, so that it can use `https://openai.com/` instead of the server api to stream the data. You can skip the signing in process but don't skip the browser api input.

```sh
docker-compose build --no-cache fullstack-app
docker-compose up fullstack-app
```
Ensure that the .env files for each service are correctly configured before starting them.

### API Usage

Once the application is running, you can send queries to the /query endpoint using a tool like curl or Postman. Here is an example using curl:

```sh
curl -X 'POST' \
  'http://localhost:8000/query' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "queries": ["<your query here>"]
}'
```

Replace "<your query here>" with your actual query.

## Docker Compose
The `docker-compose.yml` file at the root directory orchestrates the startup, networking, and volume management of the services. It defines the necessary volumes for data persistence and a bridge network for inter-service communication.

### Volumes:
- `dataset_extraction`: Shared volume for PDF data and embeddings.
- `redis_data`: Persistent storage for Redis cache.
### Networks:
- `global_network`: Ensures that all containers can communicate with each other and with Redis.
## Additional Notes
The services are designed to be run sequentially to ensure data flow from extraction to querying.
Adjustments to `.env` files or Docker Compose configurations may be necessary based on your specific deployment needs or changes in external API keys or paths.
This setup provides a streamlined process for deploying a comprehensive document handling and querying system using modern containerization and microservice architecture.