# Sialia Project

github: https://github.com/feiyang3cat/sialia


## Client-side Solution: smart/heavey-client-side solution 
(my personal preference, also impacted by temporal designs)
1. compression on the client side directly 
   a. leverage CPU of client-side
   b. saves bandwidth
   c. makes network faster
2. client-side doing the majority of the work 
   a. client-side sending to S3 directly, 
   b. uploading the offsets to the server


## Server-side Solution
![./image.jpeg](image.jpeg)

### (A) Performance Related
#### Major Changes that Brought the Most Impact

| Area    | Improvement | Changes                                                                                                                                   |
| ------- | ----------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| Memory  | 50%         | Changed data structure in S3 and Postgres to reduce redundant buffer for input/output manipulation (input/output take 99% of the payload) |
| Latency | 80%         | 1. Sub-batches and parallelization for uploading<br>2. Compression for input/output (changed from gzip to zstd after submission)          |

#### Minor Optimizations
1. golang-json -> `go-json` (both latency and memory) -> asked cc to implement thte change
2. logging -> `zerolog` (both latency and memory) -> asked cc to implement the change
3. connection pooling for database (latency, but very minor) -> asked cc to implement the change

#### Tools for Performance Analysis
1. tracing (open-tracing + OpenObserve) -> for latency analysis 
2. profiling (pprof) -> for memory analysis
3. profiling (perf) -> for CPU analysis (not yet used, mainly io-bound latency)

### (B) Code Organization
1. structure:  -> changed by me
- moving into internal package (normal golang project structure)
- split into multiple files/functions:
  -  main.go, s3.go, postgres.go, etc. (high-level single responsibility principle/small modules)
  -  internal functions are more atomic and reusable

2. dependencies: -> asked cc to implement the change
- use dependency injection to manage initialization of singletons: fx
- logging framework: zerolog

3. easier commands and tools: -> asked cc to implement the change
-  make bench-record (bench, recording, frontend analysis)

### (C) Working Pipeline -> designed by me, asked cc to implement the change
- change code
- `make test` to ensure correctness
- `make bench-record` to record the benchmark and performance metrics
- `make show-br` to show the benchmark and performance metrics
  

### Major Dependencies of the server-side solution
- Introduced by me:
	github.com/goccy/go-json v0.10.5 -> fastest JSON library compatible with golang-json
	github.com/rs/zerolog v1.34.0 -> structured logging library
	go.uber.org/fx v1.24.0 -> Dependency injection library for managing singletons
- Existing dependencies:
	github.com/aws/aws-sdk-go-v2 v1.41.2 -> S3 client
	github.com/jackc/pgx/v5 v5.7.5 -> PostgreSQL client
	github.com/go-chi/chi/v5 v5.1.0 -> HTTP router
	go.opentelemetry.io/otel v1.42.0 -> OpenTelemetry library for tracing
	github.com/joho/godotenv v1.5.1 -> load environment variables from .env file
