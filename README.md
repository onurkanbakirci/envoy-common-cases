# Envoy Proxy Load Balancer Demo

This project demonstrates a simple load balancing setup using Envoy Proxy with Docker Compose. It includes two backend services that Envoy distributes traffic between using a round-robin load balancing strategy.

## Architecture

```
Client Request → Envoy Proxy (Port 10000) → Backend Services
                     ↓
               ┌─────────────┐
               │   Envoy     │
               │ Load Balancer│
               └─────────────┘
                     ↓
            ┌─────────┴─────────┐
            ↓                   ↓
    ┌─────────────┐     ┌─────────────┐
    │  Service-1  │     │  Service-2  │
    │ (Port 5678) │     │ (Port 5678) │
    └─────────────┘     └─────────────┘
```

## Components

### Envoy Proxy
- **Admin Interface**: Accessible on port `9901`
- **Proxy Interface**: Accessible on port `10000`
- **Load Balancing**: Round-robin distribution between backend services
- **Configuration**: Located in `edge/envoy.yaml`

### Backend Services
- **Service-1**: HTTP echo service responding with "hello world! service-1"
- **Service-2**: HTTP echo service responding with "hello world! service-2"
- **Image**: `hashicorp/http-echo:latest`
- **Internal Port**: `5678`

## Prerequisites

- Docker
- Docker Compose

## Quick Start

1. **Clone and navigate to the project**:
   ```bash
   cd envoy/edge
   ```

2. **Start the services**:
   ```bash
   docker-compose up -d
   ```

3. **Test the load balancer**:
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

4. **Check Envoy admin interface**:
   ```bash
   curl http://localhost:9901/stats
   ```

## Configuration Details

### Envoy Configuration (`edge/envoy.yaml`)

- **Admin Interface**: Binds to `0.0.0.0:9901` for management and monitoring
- **HTTP Listener**: Accepts traffic on `0.0.0.0:10000`
- **Cluster Configuration**: 
  - Name: `envoy_backend_services`
  - Load balancing policy: Round-robin
  - Health check timeout: 1 second
  - Service discovery: Strict DNS

### Docker Compose Configuration (`edge/docker-compose.yaml`)

- **Network**: All services communicate via `envoy-network` bridge network
- **Dependencies**: Envoy starts after both backend services are ready
- **Volume Mounting**: Envoy configuration is mounted from the host

## Monitoring and Administration

### Envoy Admin Interface
Access the admin interface at `http://localhost:9901` to view:
- `/stats` - Runtime statistics
- `/clusters` - Cluster status and health
- `/config_dump` - Current configuration
- `/server_info` - Server information

### Useful Admin Endpoints
```bash
# View cluster status
curl http://localhost:9901/clusters

# View runtime stats
curl http://localhost:9901/stats

# View configuration
curl http://localhost:9901/config_dump
```

## Testing Load Balancing

You can verify the round-robin load balancing behavior:

```bash
# Script to test multiple requests
for i in {1..6}; do
  echo "Request $i:"
  curl -s http://localhost:10000
  echo ""
done
```

## Stopping the Services

```bash
docker-compose down
```

To also remove the network:
```bash
docker-compose down --volumes --remove-orphans
```

## Customization

### Adding More Backend Services

1. Add new service to `docker-compose.yaml`:
   ```yaml
   service-3:
     container_name: service-3
     image: hashicorp/http-echo:latest
     networks:
       - envoy-network
     environment:
       - ECHO_TEXT="hello world! service-3"
   ```

2. Update the cluster endpoints in `envoy.yaml`:
   ```yaml
   - endpoint:
       address:
         socket_address:
           address: service-3
           port_value: 5678
   ```

### Changing Load Balancing Strategy

Modify the `lb_policy` in `envoy.yaml`:
- `round_robin` (default)
- `least_request`
- `random`
- `ring_hash`

## Troubleshooting

### Common Issues

1. **Port conflicts**: Ensure ports 9901 and 10000 are not in use
2. **Service startup order**: Envoy depends on backend services being available
3. **Network connectivity**: All services must be on the same Docker network

### Logs

View logs for specific services:
```bash
docker-compose logs envoy
docker-compose logs service-1
docker-compose logs service-2
```

## Learn More

- [Envoy Proxy Documentation](https://www.envoyproxy.io/docs)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Load Balancing Concepts](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/upstream/load_balancing/load_balancing)
