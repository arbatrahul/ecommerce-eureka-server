# Ecommerce Eureka Server

Service discovery server for the Ecommerce Microservices Platform using Netflix Eureka.

## Overview

The Eureka Server provides service discovery and registration for all microservices in the Ecommerce Platform. It acts as a central registry where services can register themselves and discover other services by name rather than hardcoded URLs.

## Features

- ✅ **Service Discovery**: Central registry for all microservices
- ✅ **Service Registration**: Automatic service registration
- ✅ **Health Monitoring**: Health check integration
- ✅ **Load Balancing**: Service instance management
- ✅ **High Availability**: Support for peer-to-peer replication
- ✅ **Dashboard UI**: Web-based service registry dashboard
- ✅ **RESTful API**: Service registry REST API

## Quick Start

### Prerequisites
- Java 17 or higher
- Maven 3.6 or higher

### Running Locally

1. **Build the project**:
   ```bash
   mvn clean package
   ```

2. **Run the application**:
   ```bash
   java -jar target/eureka-server-1.0.0.jar
   ```

3. **Or use Maven**:
   ```bash
   mvn spring-boot:run
   ```

The Eureka Server will start on `http://localhost:8761`

### Accessing the Dashboard

Once running, access the Eureka Dashboard at:
- **URL**: `http://localhost:8761`
- **Default Username**: (if security enabled)
- **Default Password**: (if security enabled)

## Configuration

### Application Configuration (application.yml)

```yaml
server:
  port: 8761

spring:
  application:
    name: eureka-server

eureka:
  instance:
    hostname: localhost
  client:
    register-with-eureka: false
    fetch-registry: false
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

### Standalone Mode (Default)

For standalone Eureka Server:
```yaml
eureka:
  client:
    register-with-eureka: false  # Don't register itself
    fetch-registry: false         # Don't fetch registry
```

### Peer-to-Peer Mode (High Availability)

For multiple Eureka Server instances:
```yaml
# Instance 1
eureka:
  instance:
    hostname: eureka1.example.com
  client:
    service-url:
      defaultZone: http://eureka2.example.com:8761/eureka/

# Instance 2
eureka:
  instance:
    hostname: eureka2.example.com
  client:
    service-url:
      defaultZone: http://eureka1.example.com:8761/eureka/
```

### Security Configuration (Optional)

To enable security on Eureka Server:

```yaml
spring:
  security:
    user:
      name: admin
      password: admin123

eureka:
  client:
    service-url:
      defaultZone: http://admin:admin123@localhost:8761/eureka/
```

## Service Registration

### How Services Register

Microservices register with Eureka by including the Eureka client dependency and configuring:

```yaml
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
    fetch-registry: true
    register-with-eureka: true
  instance:
    prefer-ip-address: true
```

### Registered Services

The following services register with Eureka:

1. **user-service** - Port 8080
2. **product-service** - Port 8082
3. **cart-service** - Port 8083
4. **order-service** - Port 8084
5. **payment-service** - Port 8085
6. **notification-service** - Port 8086
7. **api-gateway** - Port 8080

## API Endpoints

### Eureka REST API

#### Get All Applications
```http
GET /eureka/apps
```

#### Get Application by Name
```http
GET /eureka/apps/{application-name}
```

#### Get Instance by ID
```http
GET /eureka/apps/{application-name}/{instance-id}
```

#### Get All Instances
```http
GET /eureka/instances/{instance-id}
```

### Dashboard Endpoints

#### Eureka Dashboard
```http
GET /
```

#### Eureka Dashboard (Alternative)
```http
GET /eureka
```

## Usage Examples

### Check Eureka Server Status

```bash
curl http://localhost:8761/eureka/apps
```

### Get Registered Services

```bash
curl http://localhost:8761/eureka/apps
```

### Get Specific Service Instances

```bash
curl http://localhost:8761/eureka/apps/user-service
```

### Access Dashboard

Open in browser:
```
http://localhost:8761
```

## Architecture

### Service Discovery Flow

1. **Service Startup** → Microservice starts and registers with Eureka
2. **Heartbeat** → Service sends periodic heartbeats to Eureka
3. **Service Discovery** → Other services query Eureka for service instances
4. **Load Balancing** → Load balancer selects instance from registry
5. **Service Call** → Request routed to selected service instance

### Eureka Server Components

1. **Service Registry**: Stores all registered service instances
2. **Health Monitor**: Monitors service health via heartbeats
3. **Dashboard**: Web UI for viewing registered services
4. **REST API**: Programmatic access to service registry

## High Availability Setup

### Multi-Instance Configuration

For production, run multiple Eureka Server instances:

#### Instance 1 (application-peer1.yml)
```yaml
spring:
  profiles: peer1

eureka:
  instance:
    hostname: eureka1.example.com
  client:
    service-url:
      defaultZone: http://eureka2.example.com:8761/eureka/
```

#### Instance 2 (application-peer2.yml)
```yaml
spring:
  profiles: peer2

eureka:
  instance:
    hostname: eureka2.example.com
  client:
    service-url:
      defaultZone: http://eureka1.example.com:8761/eureka/
```

### Running Multiple Instances

```bash
# Instance 1
java -jar eureka-server.jar --spring.profiles.active=peer1

# Instance 2
java -jar eureka-server.jar --spring.profiles.active=peer2
```

## Testing

### Verify Eureka Server is Running

```bash
curl http://localhost:8761/actuator/health
```

### Check Registered Services

1. Open browser: `http://localhost:8761`
2. View registered applications in dashboard
3. Check service instances and their status

### Test Service Registration

1. Start a microservice with Eureka client
2. Check Eureka dashboard for new registration
3. Verify service appears in registry

## Deployment

### Docker

```bash
# Build image
docker build -t ecommerce/eureka-server .

# Run container
docker run -p 8761:8761 \
  -e SPRING_PROFILES_ACTIVE=docker \
  ecommerce/eureka-server
```

### Docker Compose (Multi-Instance)

```yaml
version: '3.8'
services:
  eureka-server-1:
    image: ecommerce/eureka-server
    ports:
      - "8761:8761"
    environment:
      - SPRING_PROFILES_ACTIVE=peer1
      - EUREKA_INSTANCE_HOSTNAME=eureka1

  eureka-server-2:
    image: ecommerce/eureka-server
    ports:
      - "8762:8761"
    environment:
      - SPRING_PROFILES_ACTIVE=peer2
      - EUREKA_INSTANCE_HOSTNAME=eureka2
```

### Production Considerations

1. **High Availability**:
   - Run at least 2 Eureka Server instances
   - Configure peer-to-peer replication
   - Use load balancer for Eureka endpoints

2. **Security**:
   - Enable authentication
   - Use HTTPS
   - Restrict access to Eureka dashboard

3. **Monitoring**:
   - Monitor Eureka Server health
   - Track registered service count
   - Alert on service registration failures

4. **Performance**:
   - Tune heartbeat intervals
   - Configure eviction timers
   - Optimize registry storage

5. **Network**:
   - Ensure network connectivity between instances
   - Configure firewall rules
   - Use DNS for service discovery

## Troubleshooting

### Common Issues

1. **Services Not Registering**:
   - Verify Eureka Server is running
   - Check service Eureka client configuration
   - Verify network connectivity
   - Check service logs for registration errors

2. **Services Disappearing from Registry**:
   - Check service health endpoints
   - Verify heartbeat configuration
   - Check eviction timer settings
   - Review service logs

3. **Eureka Server Not Starting**:
   - Check port 8761 availability
   - Verify Java version (17+)
   - Check application logs
   - Verify configuration files

4. **Dashboard Not Accessible**:
   - Check if security is enabled
   - Verify port configuration
   - Check firewall rules
   - Verify server is running

5. **Peer Replication Not Working**:
   - Verify peer URLs are correct
   - Check network connectivity
   - Verify hostname resolution
   - Check peer configuration

### Logs

```bash
# Enable debug logging
java -jar target/eureka-server-1.0.0.jar \
  --logging.level.com.netflix.eureka=DEBUG \
  --logging.level.com.netflix.discovery=DEBUG
```

### Health Check

```bash
# Check Eureka Server health
curl http://localhost:8761/actuator/health

# Check registered applications
curl http://localhost:8761/eureka/apps
```

## Configuration Properties

### Key Eureka Server Properties

```yaml
eureka:
  server:
    # Enable self-preservation mode
    enable-self-preservation: true
    # Eviction interval (ms)
    eviction-interval-timer-in-ms: 60000
    # Response cache update interval (ms)
    response-cache-update-interval-ms: 30000
    # Response cache auto-expiration (ms)
    response-cache-auto-expiration-in-seconds: 180
  
  instance:
    # Lease renewal interval (seconds)
    lease-renewal-interval-in-seconds: 30
    # Lease expiration duration (seconds)
    lease-expiration-duration-in-seconds: 90
```

## Dependencies

- Spring Boot 3.2.0
- Spring Cloud Netflix Eureka Server
- Spring Boot Actuator
- Spring Boot Web

## Project Structure

```
src/
├── main/
│   ├── java/org/example/eureka/
│   │   └── EurekaServerApplication.java
│   └── resources/
│       ├── application.yml
│       └── application-peer1.yml (optional)
│       └── application-peer2.yml (optional)
└── test/
```

## Contributing

1. Follow Spring Cloud best practices
2. Test high availability setup
3. Document configuration changes
4. Update service registration examples
5. Handle errors gracefully

## Best Practices

1. **Always Run Multiple Instances**: For production, run at least 2 Eureka Server instances
2. **Enable Security**: Protect Eureka dashboard and API
3. **Monitor Health**: Set up monitoring for Eureka Server
4. **Configure Timeouts**: Tune heartbeat and eviction intervals
5. **Use DNS**: Use DNS names instead of IP addresses
6. **Backup Configuration**: Keep configuration files in version control

## Related Services

The following services register with this Eureka Server:

- [User Service](../ecommerce-user-service/README.md)
- [Product Service](../ecommerce-product-service/README.md)
- [Cart Service](../ecommerce-cart-service/README.md)
- [Order Service](../ecommerce-order-service/README.md)
- [Payment Service](../ecommerce-payment-service/README.md)
- [Notification Service](../ecommerce-notification-service/README.md)
- [API Gateway](../ecommerce-api-gateway/README.md)

## License

This project is part of the Ecommerce Microservices Platform.
