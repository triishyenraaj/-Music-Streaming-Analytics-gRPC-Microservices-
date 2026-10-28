# 🎵 Music Streaming Analytics (gRPC Microservices)

A modular **gRPC-based microservices system** that simulates a **music streaming analytics pipeline**, similar to real-world platforms like Spotify or YouTube Music. This project demonstrates distributed processing, inter-service communication, and containerized deployment using Docker.

---

## 🧩 High-Level Architecture

This system consists of **three gRPC services** and a client orchestrator:

| Service | Description | Port |
|----------|--------------|------|
| **MapReduceService** | Aggregates raw streaming data (song plays) into per-song counts (like total plays per artist/song). | `50051` |
| **UserBehaviorService** | Analyzes user-level statistics such as total listening time and favorite artist. | `50053` |
| **RecommendationService** | Uses play counts and user behavior to generate personalized recommendations and trending lists. | `50055` |
| **Client** | Orchestrates calls to all services, prints analytics results, and saves performance metrics. | — |

All services communicate using **Protocol Buffers (protobuf)** definitions found under `grpc/generated/`.

---

## ⚙️ Pipeline Overview

```
Client → MapReduceService → UserBehaviorService → RecommendationService → Client
```

1. **Client loads dataset** (`data/stream_data.csv`).
2. Sends the data to **MapReduceService**, which:
   - Counts how many times each song was played (`artist - song_id`).
   - Returns aggregated play counts and metrics.
3. Sends the same dataset to **UserBehaviorService**, which:
   - Calculates total listening time per user.
   - Determines each user’s top artist.
   - Lists top 5 most active users.
4. Sends results to **RecommendationService**, which:
   - Identifies trending songs (global top 5).
   - Recommends songs not from a user’s top artist.
5. **Client** aggregates all results and writes detailed JSON metrics to `results/run_grpc_metrics.json`.

---

## 🧠 Key Features

| Category | Details |
|-----------|----------|
| **Concurrency** | Each gRPC service uses `ThreadPoolExecutor` for parallel processing. |
| **Metrics & Timing** | Each service records processing time and outputs JSON metrics. |
| **Isolation** | Each service runs independently and communicates via defined protobuf schemas. |
| **Containerization** | All services are Dockerized and orchestrated using `docker-compose`. |
| **Scalable** | Can be easily extended for new analytics or recommendation models. |

---

## 🧭 Running Locally (Without Docker)

You can also run all services manually using Python, simulating the distributed setup.

### **Step 1: Install Dependencies**

Ensure you have Python 3.8+ installed. Then run:

```bash
pip install -r requirements.txt
```

This installs required packages such as `grpcio`, `protobuf`, and `grpcio-tools`.

### **Step 2: Generate gRPC Code from .proto File**

If not already generated, create the Python gRPC bindings using:

```bash
python generate_proto.py
```

This script compiles `music_service.proto` into `music_service_pb2.py` and `music_service_pb2_grpc.py` under `grpc/generated/`.

### **Step 3: Start All gRPC Services**

Each service must be started in a **separate terminal** to simulate a distributed microservices environment.

#### 🧮 Terminal 1 — MapReduce Service
```bash
cd grpc/server
python mapreduce_stream_service.py
```
Expected Output:
```
[MapReduce] gRPC server started on port 50051
```

#### 👥 Terminal 2 — UserBehavior Service
```bash
cd grpc/server
python user_behavior_service.py
```
Expected Output:
```
[UserBehavior] gRPC server started on port 50053
```

#### 🎧 Terminal 3 — Recommendation Service
```bash
cd grpc/server
python recommendation_service.py
```
Expected Output:
```
[Recommendation] gRPC server started on port 50055
```

### **Step 4: Run the Client**

Once all services are running, open a **new terminal** and execute:

```bash
cd  grpc/client
python client.py
```

This client will:
- Load streaming data from `data/stream_data.csv`
- Send it sequentially to all 3 services
- Print detailed analytics to the terminal
- Save the results to `results/run_grpc_metrics.json`

### **Step 5: View Results**

After completion, open:
```
results/run_grpc_metrics.json
```
This file contains complete performance timings and analysis output for all services.

---

## 📊 Output Metrics

Each service writes runtime metrics to `/tmp/` (inside container) and the client aggregates them into a final summary JSON.

### Example Service Metrics:

```json
{
  "processing_time": 0.342,
  "count_keys": 142,
  "num_users": 55,
  "num_trending": 5
}
```

### Final Aggregated Output (`results/run_grpc_metrics.json`):

```json
{
  "workflow": "Client → MapReduce → UserBehavior → Recommendation → Client",
  "performance": {
    "mapreduce_time": 0.34,
    "userbehavior_time": 0.28,
    "recommendation_time": 0.19,
    "total_workflow_time": 0.81
  },
  "mapreduce_results": {
    "top_songs": {"Artist1 - SongA": 53, "Artist2 - SongB": 41}
  },
  "userbehavior_results": {
    "top_users": ["U1", "U2", "U3"]
  },
  "recommendation_results": {
    "trending_songs": ["Artist2 - SongB", "Artist3 - SongC"]
  }
}
```

## 🐳 Docker Setup

Each service has its own Dockerfile, defined in the `docker/` folder, and the setup is orchestrated via `docker-compose.grpc.yml`.

### **Run the system with Docker Compose:**

```bash
docker-compose -f docker/docker-compose.grpc.yml up --build
```

This command starts the following containers:
- **grpc-mapreduce** (port `50051`)
- **grpc-userbehavior** (port `50053`)
- **grpc-recommendation** (port `50055`)

All containers are connected on the shared `grpc-network` bridge.

### **Run client locally:**

Once the services are running, execute:

```bash
cd grpc/client
python client.py
```

---

## 📁 Folder Structure

```
cst352-main/
├── data/
│   └── stream_data.csv
├── docker/
│   ├── Dockerfile.grpc.mapreduce
│   ├── Dockerfile.grpc.userbehavior
│   ├── Dockerfile.grpc.recommendation
│   └── docker-compose.grpc.yml
├── grpc/
│   ├── client/
│   │   └── main_client.py
│   ├── server/
│   │   ├── mapreduce_server.py
│   │   ├── userbehavior_server.py
│   │   └── recommendation_server.py
│   ├── proto/
│   │   └── music_service.proto
│   └── generated/
│       ├── music_service_pb2.py
│       └── music_service_pb2_grpc.py
├── services/
├── requirements.txt
└── generate_proto.py
```

---

## 📊 Output Metrics

Each service writes runtime metrics to `/tmp/` (inside container) and the client aggregates them into a final summary JSON.

### Example Service Metrics:

```json
{
  "processing_time": 0.342,
  "count_keys": 142,
  "num_users": 55,
  "num_trending": 5
}
```

### Final Aggregated Output (`results/run_grpc_metrics.json`):

```json
{
  "workflow": "Client → MapReduce → UserBehavior → Recommendation → Client",
  "performance": {
    "mapreduce_time": 0.34,
    "userbehavior_time": 0.28,
    "recommendation_time": 0.19,
    "total_workflow_time": 0.81
  },
  "mapreduce_results": {
    "top_songs": {"Artist1 - SongA": 53, "Artist2 - SongB": 41}
  },
  "userbehavior_results": {
    "top_users": ["U1", "U2", "U3"]
  },
  "recommendation_results": {
    "trending_songs": ["Artist2 - SongB", "Artist3 - SongC"]
  }
}
```

---

## 🔍 Evaluation Summary

| Aspect | Rating | Comment |
|--------|---------|----------|
| **Architecture** | ⭐⭐⭐⭐☆ (4.5/5) | Well-structured microservice pattern with clear gRPC communication. |
| **Code Quality** | ⭐⭐⭐⭐☆ | Modular, clean, and uses concurrency effectively. |
| **Scalability** | ⭐⭐⭐⭐☆ | Each service is independent and easily deployable. |
| **Error Handling** | ⭐⭐⭐☆ | Could improve error propagation from gRPC layers. |
| **Documentation** | ⭐⭐⭐ | Functional; this README provides the missing overview. |

---

## 🚀 Future Improvements

- Add a REST gateway or frontend dashboard for visualization.
- Use persistent storage (e.g., PostgreSQL or Redis) for historical analytics.
- Implement advanced recommendation models using ML libraries.
- Add Prometheus/Grafana for live performance monitoring.

---

## 📦 Requirements

To run locally without Docker:

```bash
pip install -r requirements.txt
python generate_proto.py
python grpc/server/mapreduce_server.py
python grpc/server/userbehavior_server.py
python grpc/server/recommendation_server.py
python grpc/client/main_client.py
```

---

## 🧾 License

This project is for **academic and demonstration purposes** under the CST352 course. All rights reserved by the original author(s).

---

