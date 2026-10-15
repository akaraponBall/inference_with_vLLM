# vLLM Benchmark

A Docker Compose-based project for serving and benchmarking Large Language Models (LLMs) using [vLLM](https://github.com/vllm-project/vllm).

The project provides two services:

- **vLLM Server:** Hosts an LLM through an OpenAI-compatible API with NVIDIA GPU acceleration.
- **Benchmark:** Measures inference throughput, latency, and concurrency using `vllm bench serve`.

## Prerequisites

- Docker Engine and Docker Compose
- NVIDIA GPU with compatible drivers
- NVIDIA Container Toolkit

## Configuration

Create a `.env` file in the project directory:

```dotenv
# Model configuration
MODEL_NAME=cyankiwi/LFM2.5-8B-A1B-AWQ-INT4
HF_TOKEN=
VLLM_LOGGING_LEVEL=INFO

# vLLM server
GPU_MEM_UTIL=0.85
MAX_MODEL_LEN=8192

# Benchmark
BENCH_INPUT_LEN=512
BENCH_OUTPUT_LEN=128
BENCH_NUM_PROMPTS=100
BENCH_CONCURRENCY=8
```

Adjust the model name and settings according to your GPU resources.

## Usage

### 1. Start the vLLM server

```bash
docker compose up -d
```

The server runs at `http://localhost:8000`.

Check the server status:

```bash
docker compose ps
```

Verify the API:

```bash
curl http://localhost:8000/v1/models
```

View server logs:

```bash
docker compose logs -f vllm
```

### 2. Run the benchmark

```bash
docker compose run --rm benchmark
```

The benchmark waits for the vLLM service to become healthy before sending requests.

Results are printed directly to the terminal; no result files are saved.

### 3. Test different concurrency levels

Override concurrency without modifying `.env`:

```bash
BENCH_CONCURRENCY=1 docker compose run --rm benchmark
BENCH_CONCURRENCY=4 docker compose run --rm benchmark
BENCH_CONCURRENCY=8 docker compose run --rm benchmark
BENCH_CONCURRENCY=16 docker compose run --rm benchmark
BENCH_CONCURRENCY=32 docker compose run --rm benchmark
```

Only one benchmark should run at a time for comparable measurements.

## Benchmark Metrics

| Metric | Description |
|---|---|
| Request throughput (req/s) | Completed requests per second |
| Output throughput (tok/s) | Generated tokens per second |
| TTFT | Time to First Token |
| TPOT | Time Per Output Token |
| ITL | Inter-Token Latency |
| E2EL | End-to-End Latency |

Increasing concurrency may improve throughput but also increase response latency.

## Stop the Server

```bash
docker compose down
```

Downloaded models remain cached through the mounted Hugging Face directory.

## Notes

- The benchmark runs in a separate container without requiring GPU access.
- The benchmark service uses a Docker Compose profile and does not start during normal `docker compose up -d`.
- A health check ensures that benchmarking begins only after the vLLM server is ready.
- For reproducible benchmarks, use fixed model settings, input/output lengths, and a pinned vLLM Docker image version.

## References

- [vLLM Documentation](https://docs.vllm.ai/)
- [vLLM Benchmark CLI](https://docs.vllm.ai/en/stable/benchmarking/cli/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
