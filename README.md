# Envoy Proxy Load Balancer

This repository demonstrates how to use Envoy Proxy as a load balancer and API Gateway. The setup includes an Envoy proxy that distributes incoming HTTP requests across multiple backend services using round-robin load balancing.

## Architecture

```
Client Request → Envoy Proxy (Port 10000) → Backend Services
                     ↓
               ┌─────────────┐
               │   Envoy     │
               │Load Balancer│
               └─────────────┘
                     ↓
            ┌─────────┴─────────┐
            ↓                   ↓
    ┌─────────────┐     ┌─────────────┐
    │  Service-1  │     │  Service-2  │
    │ (Port 5678) │     │ (Port 5678) │
    └─────────────┘     └─────────────┘
```

## Quick Start

1. **Start the services**:
   ```bash
   docker-compose up -d
   ```

2. **Test the load balancer**:
   ```bash
   # Make multiple requests to see round-robin load balancing
   curl http://localhost:10000
   curl http://localhost:10000
   curl http://localhost:10000
   ```

   You should see alternating responses:
   ```
   hello world! service-1
   hello world! service-2
   hello world! service-1
   ```

3. **Check Envoy admin interface**:
   ```bash
   curl http://localhost:9901/stats
   ```

4. **Stop the services**:
   ```bash
   docker-compose down
   ```

## Advanced Usage

### Testing Load Balancing Behavior

You can verify the round-robin load balancing with this script:

```bash
# Script to test multiple requests
for i in {1..6}; do
  echo "Request $i:"
  curl -s http://localhost:10000
  echo ""
done
```

### Cleanup

To stop services and remove networks:
```bash
docker-compose down --volumes --remove-orphans
```