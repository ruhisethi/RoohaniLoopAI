Batch Ingestion Service with FastAPI
Overview
This project implements a batch ingestion service using FastAPI. It accepts a list of IDs with an associated priority and processes them in batches asynchronously with rate limiting. The service also exposes APIs to track the status of ingestion and batch processing.

Features
Batch processing: IDs are split into batches of 3.

Priority-based queue: Batches with higher priority (HIGH > MEDIUM > LOW) are processed first.

Rate limiting: Minimum 5-second delay between processing two batches to avoid overloading.

Asynchronous processing: Batches and IDs are processed asynchronously to improve throughput.

In-memory status tracking: Tracks ingestion requests, batch statuses, and overall completion.

Status querying: API to check the status of ingestion and individual batches.

Technology Stack
FastAPI: Python web framework for building APIs.

Asyncio: To handle asynchronous batch processing.

Uvicorn: ASGI server to run FastAPI.

Python 3.9+

API Endpoints
1. POST /ingest
Submit a list of IDs with a priority for ingestion.

Request Body:

json
Copy
{
  "ids": [int, int, ...],    // List of integer IDs, each between 1 and 1,000,000,007
  "priority": "HIGH" | "MEDIUM" | "LOW"
}
Response:

json
Copy
{
  "ingestion_id": "string"   // Unique identifier for this ingestion request
}
Behavior:

Validates that all IDs fall within the specified range.

Splits IDs into batches of size 3.

Creates a unique ingestion ID and unique batch IDs.

Adds batches to a priority queue.

Starts asynchronous batch processing with rate limiting.

2. GET /status/{ingestion_id}
Retrieve the status of an ingestion request.

Path Parameter:

ingestion_id (string): The unique ingestion ID received from /ingest.

Response:

json
Copy
{
  "ingestion_id": "string",
  "status": "yet_to_start" | "triggered" | "completed",
  "batches": [
    {
      "batch_id": "string",
      "ids": [int, int, ...],
      "status": "yet_to_start" | "triggered" | "completed"
    },
    ...
  ]
}
Behavior:

Returns overall ingestion status.

Returns status of each batch and the IDs it contains.

Internal Workflow
Ingestion submission:

Clients submit IDs and priority.

IDs are split into batches.

Each batch is assigned a unique batch ID.

Batches are enqueued based on priority and enqueue time.

Batch processing:

A background async task processes batches from the queue.

At least 5 seconds elapse between batch processing to throttle API calls.

For each ID in a batch, a mock async function simulates an external API call with a 1-second delay.

Batch and ingestion statuses update accordingly.

Status updates:

Batches transition through statuses: yet_to_start → triggered → completed.

Ingestion status is completed only when all batches are completed.

Running the Project Locally
Install dependencies:

bash
Copy
pip install fastapi uvicorn
Run the server:

bash
Copy
uvicorn main:app --reload
API docs available at:

arduino
Copy
http://127.0.0.1:8000/docs
Deployment Instructions (Render.com example)
Place your FastAPI app code in main.py.

Create a requirements.txt including:

nginx
Copy
fastapi
uvicorn
Set the start command in Render to:

nginx
Copy
uvicorn main:app --host 0.0.0.0 --port 10000
Push the code to a Git repository connected to Render.

Deploy your service.

Limitations and Future Improvements
In-memory storage: Currently, ingestion and batch data are stored in memory and lost on service restart. Use persistent storage (e.g., database) for production.

Rate limiting: Fixed delay based rate limiting; consider more robust rate limiters.

Error handling: Limited error handling on batch processing; enhance for real API calls.

Scalability: Single-process in-memory queue may not scale; consider distributed task queues like Celery or RQ.

Security: No authentication or input sanitization beyond ID range validation.

Example Usage
Submit ingestion:

bash
Copy
curl -X POST "http://localhost:8000/ingest" -H "Content-Type: application/json" -d '{
  "ids": [1,2,3,4,5,6,7],
  "priority": "HIGH"
}'
Check status:

bash
Copy
curl "http://localhost:8000/status/<ingestion_id>"
