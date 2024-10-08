# LLM Benchmark Application

## Table of Contents
1. [Introduction](#introduction)
2. [Features](#features)
3. [Prerequisites](#prerequisites)
4. [Local Development Setup](#local-development-setup)
5. [Docker Compose Setup](#docker-compose-setup)
6. [Kubernetes Deployment](#kubernetes-deployment)
7. [Configuration](#configuration)
8. [Usage](#usage)
9. [Implementation Details](#implementation-details)
10. [Troubleshooting](#troubleshooting)

## Introduction

The LLM Benchmark application is designed to benchmark and simulate metrics for Large Language Models (LLMs). It consists of two main components: a metric benchmarking service and a metric simulation service. The application uses PostgreSQL for data storage and Redis for caching and message queuing.

## Features

- Simulates performance data for multiple LLMs
- Ranks LLMs based on various metrics
- Exposes rankings via an API endpoint
- Deployable as microservices in a Kubernetes cluster


## Prerequisites

- Python 3.9+
- Postgres
- Redis
- Docker and Docker Compose
- Kubernetes cluster (for Kubernetes deployment)
- Helm 3+ (for Kubernetes deployment)

## Local Development Setup

1. Clone the repository:
   ```
   git clone https://github.com/kampkelly/llm-benchmark.git
   cd llm-benchmark
   ```

2. Create a virtual environment and activate it:
   ```
   python -m venv venv
   source venv/bin/activate  # On Windows, use `venv\Scripts\activate`
   ```

3. Install dependencies:
   `pip install -r requirements.txt` or `make install`
<br>

4. Set up your `.env` file with the necessary environment variables:
   ```
   POSTGRES_USER=
   POSTGRES_HOST=localhost
   POSTGRES_PORT=5432
   POSTGRES_PASSWORD=postgres
   POSTGRES_DB=llm_benchmark
   API_KEY=1234
   SCHEDULE_INTERVAL=1
   REDIS_HOST=localhost
   REDIS_PORT=6379
   SEED=3
   ```

5. Run database migrations:
   `alembic upgrade head` or `make run-migrations`
<br>
6. Start the services:
    Metric simulator
   `uvicorn metric_simulator.main:app --reload` or `make start-simulator`

   Metric benchmark (In a separate terminal):
   `uvicorn metric_benchmark.main:app --reload --port 8001` or `make start-benchmark`

## Docker Compose Setup

1. Ensure Docker and Docker Compose are installed on your system.

2. Build and start the services:
   ```
   docker-compose up --build
   ```

This will start the metric benchmark service, metric simulator service, PostgreSQL, and Redis.
You can the access the rankings api (more details below) on port `8001`


## Kubernetes Deployment

1. Ensure you have a Kubernetes cluster set up and `kubectl` configured to communicate with your cluster.

2. Install Helm if you haven't already:
   ```
   curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
   ```
  or `brew install helm`


3. Navigate to the Helm chart directory:
   ```
   cd llm-benchmark-chart
   ```

4. Create/update .env file as described earlier
<br>


5. Run script to start
`./deploy.sh`
This updates dependencies, copies some needed environment variables and then installs helm using the environment variables

## Configuration

The application can be configured using environment variables. The main configuration options are:

- `POSTGRES_USER`: PostgreSQL username
- `POSTGRES_HOST`: PostgreSQL host
- `POSTGRES_PORT`: PostgreSQL port
- `POSTGRES_PASSWORD`: PostgreSQL password
- `POSTGRES_DB`: PostgreSQL database name
- `API_KEY`: API key for authentication
- `SCHEDULE_INTERVAL`: Interval for scheduled tasks
- `REDIS_HOST`: Redis host
- `REDIS_PORT`: Redis port
- `SEED`: Seed for random number generation (optional)

## Usage

Once the application is running, you can interact with it using the provided APIs. The main endpoints are:

##### Metric Benchmark Rankings API: `http://localhost:8001/api/v1/benchmarks/rankings`

This API provides an endpoint to fetch the rankings of various Language Learning Models (LLMs) based on their benchmark results. The rankings are calculated using simulation metrics such as Time to First Token (TTFT), Tokens Per Second (TPS), End-to-End Request Latency (e2e_latency), and Requests Per Second (RPS).

##### Endpoint

- **GET** `/api/v1/benchmarks/rankings`
Accessing this endpoint requires an api-key sent in the request headers under `x-api-key`. The value of this api keyh can be set in the .env file `API_KEY=1234`

##### Response

The response will be a JSON object containing the rankings of LLMs for the specified metric. The structure of the response is as follows:

```json
{
    "data": [
        {
            "ttft": [
                {
                    "llm_name": "Claude 3.5 Sonnet",
                    "mean_value": 1.05
                },
                {
                    "llm_name": "GPT-4o",
                    "mean_value": 1.03
                },
                ...
            ]
        },
        {
            "tps": [
                {
                    "llm_name": "Claude 3.5 Sonnet",
                    "mean_value": 80.45
                },
                ...
            ]
        },
    ]
}
```

### Implementation Details
The metric simulation makes use of a randomizer which generates random values from a uniform distribution. It optionally makes use of a seed whose default seed value is 3. The response attained from this seed value (3) can be found in response.json file. To change the seed, please update the value in the readme.

Some initial data for llms and metrics (not data points) are seeded into the database the first time the server boots up. This allows for easy testing and generation of data points for the metrics.

On calling the rankings API, itakes approximately 60 - 80 milliseconds to return a response. The result is then cached for faster retrieval further reducing the latency to less than 10 milliseconds The expiry of this cache is controlled by the repeated job that regenerates the metrics and clears the cache every x minutes. To configure x minutes, update the `SCHEDULE_INTERVAL` in .env file

## Troubleshooting

Common issues and their solutions:

1. Database connection errors:
   - Ensure PostgreSQL is running and accessible.
   - Check the `POSTGRES_*` environment variables.

2. Redis connection errors:
   - Ensure Redis is running and accessible.
   - Check the `REDIS_*` environment variables.
