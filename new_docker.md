# Complete Docker Setup Guide with PostgreSQL pgvector

## Current Container Analysis

From your `docker ps` output, you have:
1. **pgadmin4** - PostgreSQL administration interface
2. **postgres_pgvector** - PostgreSQL with vector extension
3. **embedding-service** - Custom embedding service

## Resource Monitoring Commands

### Check Current Resource Usage

```bash
# Real-time resource usage for all containers
docker stats

# Resource usage for specific containers
docker stats pgadmin4 postgres_pgvector embedding-service

# Detailed container information
docker inspect postgres_pgvector
docker inspect pgadmin4
docker inspect embedding-service

# System-wide Docker resource usage
docker system df
```

### Container-Specific Resource Checks

```bash
# Check container processes
docker top postgres_pgvector
docker top pgadmin4
docker top embedding-service

# View container logs (last 100 lines)
docker logs --tail 100 postgres_pgvector
docker logs --tail 100 pgadmin4
docker logs --tail 100 embedding-service

# Follow logs in real-time
docker logs -f postgres_pgvector
```

## Complete Docker Compose Setup

### docker-compose.yml

```yaml
version: '3.8'

services:
  postgres_pgvector:
    image: ankane/pgvector:latest
    container_name: postgres_pgvector
    environment:
      POSTGRES_DB: vectordb
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: your_password_here
      POSTGRES_HOST_AUTH_METHOD: trust
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init-scripts:/docker-entrypoint-initdb.d
    restart: unless-stopped
    networks:
      - app_network

  pgadmin4:
    image: dpage/pgadmin4:latest
    container_name: pgadmin4
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@example.com
      PGADMIN_DEFAULT_PASSWORD: admin_password
      PGADMIN_LISTEN_PORT: 80
    ports:
      - "8080:80"
    volumes:
      - pgadmin_data:/var/lib/pgadmin
    depends_on:
      - postgres_pgvector
    restart: unless-stopped
    networks:
      - app_network

  embedding-service:
    image: your-embedding-service:latest
    container_name: embedding-service
    ports:
      - "8000:8000"
    environment:
      DATABASE_URL: postgresql://postgres:your_password_here@postgres_pgvector:5432/vectordb
    depends_on:
      - postgres_pgvector
    restart: unless-stopped
    networks:
      - app_network

volumes:
  postgres_data:
  pgadmin_data:

networks:
  app_network:
    driver: bridge
```

### Environment Variables (.env)

```bash
# Database Configuration
POSTGRES_DB=vectordb
POSTGRES_USER=postgres
POSTGRES_PASSWORD=your_secure_password_here
POSTGRES_HOST=localhost
POSTGRES_PORT=5432

# PgAdmin Configuration
PGADMIN_DEFAULT_EMAIL=admin@example.com
PGADMIN_DEFAULT_PASSWORD=your_admin_password_here

# Application Configuration
DATABASE_URL=postgresql://postgres:your_secure_password_here@postgres_pgvector:5432/vectordb
```

## PostgreSQL pgvector Setup

### 1. Check if pgvector Extension is Available

```bash
# Connect to PostgreSQL container
docker exec -it postgres_pgvector psql -U postgres -d vectordb

# Inside PostgreSQL, check available extensions
SELECT name, default_version, installed_version 
FROM pg_available_extensions 
WHERE name = 'vector';

# List all available extensions
\dx
```

### 2. Enable pgvector Extension

#### Method 1: From Terminal (Recommended)

```bash
# Connect to PostgreSQL
docker exec -it postgres_pgvector psql -U postgres -d vectordb

# Enable the vector extension
CREATE EXTENSION IF NOT EXISTS vector;

# Verify installation
\dx

# Test vector functionality
SELECT vector_dims('[1,2,3]'::vector);
```

#### Method 2: From pgAdmin4 UI

1. Open pgAdmin4 at `http://localhost:8080`
2. Login with your credentials
3. Connect to PostgreSQL server:
   - Host: `postgres_pgvector`
   - Port: `5432`
   - Database: `vectordb`
   - Username: `postgres`
4. Navigate to: `Databases > vectordb > Extensions`
5. Right-click on Extensions → `Create → Extension`
6. Select `vector` from the dropdown
7. Click `Save`

### 3. SQL Initialization Script

Create `init-scripts/01-init-pgvector.sql`:

```sql
-- Enable pgvector extension
CREATE EXTENSION IF NOT EXISTS vector;

-- Create a sample table with vector column
CREATE TABLE IF NOT EXISTS embeddings (
    id SERIAL PRIMARY KEY,
    content TEXT,
    embedding vector(384), -- Adjust dimension as needed
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create an index for vector similarity search
CREATE INDEX IF NOT EXISTS embeddings_embedding_idx 
ON embeddings USING ivfflat (embedding vector_cosine_ops) 
WITH (lists = 100);

-- Insert sample data (optional)
INSERT INTO embeddings (content, embedding) VALUES
('Sample text 1', '[0.1, 0.2, 0.3]'::vector),
('Sample text 2', '[0.4, 0.5, 0.6]'::vector);
```

## Quick Setup Commands

### 1. Initial Setup

```bash
# Create project directory
mkdir docker-postgres-setup
cd docker-postgres-setup

# Create necessary directories
mkdir init-scripts

# Create docker-compose.yml and .env files (use content above)

# Start services
docker-compose up -d

# Check status
docker-compose ps
```

### 2. Verify Setup

```bash
# Check if services are running
docker-compose ps

# Test PostgreSQL connection
docker exec -it postgres_pgvector psql -U postgres -d vectordb -c "SELECT version();"

# Test pgvector extension
docker exec -it postgres_pgvector psql -U postgres -d vectordb -c "SELECT vector_dims('[1,2,3]'::vector);"

# Access pgAdmin4
echo "pgAdmin4 available at: http://localhost:8080"
```

### 3. Troubleshooting Commands

```bash
# View logs
docker-compose logs postgres_pgvector
docker-compose logs pgadmin4

# Restart services
docker-compose restart

# Stop and remove everything
docker-compose down -v

# Rebuild and start
docker-compose up -d --build
```

## Vector Search Examples

### Basic Vector Operations

```sql
-- Calculate cosine similarity
SELECT content, 
       1 - (embedding <=> '[0.1, 0.2, 0.3]'::vector) AS cosine_similarity
FROM embeddings
ORDER BY embedding <=> '[0.1, 0.2, 0.3]'::vector
LIMIT 5;

-- Calculate L2 distance
SELECT content, 
       embedding <-> '[0.1, 0.2, 0.3]'::vector AS l2_distance
FROM embeddings
ORDER BY embedding <-> '[0.1, 0.2, 0.3]'::vector
LIMIT 5;

-- Calculate inner product
SELECT content, 
       embedding <#> '[0.1, 0.2, 0.3]'::vector AS inner_product
FROM embeddings
ORDER BY embedding <#> '[0.1, 0.2, 0.3]'::vector DESC
LIMIT 5;
```

## Performance Optimization

### Index Types for Different Use Cases

```sql
-- For cosine distance (most common)
CREATE INDEX ON embeddings USING ivfflat (embedding vector_cosine_ops) WITH (lists = 100);

-- For L2 distance
CREATE INDEX ON embeddings USING ivfflat (embedding vector_l2_ops) WITH (lists = 100);

-- For inner product
CREATE INDEX ON embeddings USING ivfflat (embedding vector_ip_ops) WITH (lists = 100);

-- HNSW index (more accurate but slower to build)
CREATE INDEX ON embeddings USING hnsw (embedding vector_cosine_ops);
```

### Resource Limits in docker-compose.yml

```yaml
services:
  postgres_pgvector:
    # ... other config
    deploy:
      resources:
        limits:
          memory: 2G
          cpus: '1.0'
        reservations:
          memory: 1G
          cpus: '0.5'
```

## Backup and Restore

```bash
# Backup database
docker exec postgres_pgvector pg_dump -U postgres vectordb > backup.sql

# Restore database
docker exec -i postgres_pgvector psql -U postgres vectordb < backup.sql

# Backup with docker-compose
docker-compose exec postgres_pgvector pg_dump -U postgres vectordb > backup.sql
```

## Security Considerations

1. **Change Default Passwords**: Update all passwords in `.env` file
2. **Network Security**: Use custom networks to isolate containers
3. **Volume Permissions**: Ensure proper file permissions
4. **SSL/TLS**: Configure SSL for production environments

## Monitoring Script

Create `monitor.sh`:

```bash
#!/bin/bash
echo "=== Container Status ==="
docker-compose ps

echo -e "\n=== Resource Usage ==="
docker stats --no-stream

echo -e "\n=== PostgreSQL Status ==="
docker exec postgres_pgvector psql -U postgres -d vectordb -c "SELECT version();"

echo -e "\n=== pgvector Extension Status ==="
docker exec postgres_pgvector psql -U postgres -d vectordb -c "SELECT * FROM pg_extension WHERE extname = 'vector';"
```

Make it executable:
```bash
chmod +x monitor.sh
./monitor.sh
```

This setup provides a complete, production-ready environment with PostgreSQL pgvector, pgAdmin4, and your embedding service with proper monitoring and maintenance capabilities.