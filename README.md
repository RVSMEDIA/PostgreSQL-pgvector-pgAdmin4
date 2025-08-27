# PostgreSQL + pgvector + pgAdmin4 Docker Setup

## Quick Start

1. Start the services:
```bash
docker-compose up -d
```

2. Access the services:
- **PostgreSQL**: `localhost:5432`
- **pgAdmin4**: `http://localhost:8080`

## Default Credentials

### PostgreSQL
- **Host**: localhost
- **Port**: 5432
- **Database**: vectordb
- **Username**: postgres
- **Password**: password123

### pgAdmin4
- **URL**: http://localhost:8080
- **Email**: admin@admin.com
- **Password**: admin123

## Connecting pgAdmin to PostgreSQL

1. Open pgAdmin4 at `http://localhost:8080`
2. Login with the pgAdmin credentials above
3. Right-click "Servers" → "Register" → "Server"
4. **General tab**: Name = "PostgreSQL Vector DB"
5. **Connection tab**:
   - Host: `postgres` (container name)
   - Port: `5432`
   - Username: `postgres`
   - Password: `password123`

## Features

- PostgreSQL 16 with pgvector extension
- Sample vector table with embedding column (1536 dimensions)
- pgAdmin4 web interface
- Persistent data volumes
- Health checks and proper service dependencies

## Commands

```bash
# Start services
docker-compose up -d

# View logs
docker-compose logs -f

# Stop services
docker-compose down

# Stop and remove volumes (WARNING: deletes data)
docker-compose down -v
```



Stop the existing pgAdmin4 container:
docker stop pgadmin4
docker rm pgadmin4

Run pgAdmin4 with required environment variables:
docker run -d -p 8080:80 \
    -e PGADMIN_DEFAULT_EMAIL=mandeep@rvsmedia.com \
    -e PGADMIN_DEFAULT_PASSWORD=admin0066 \
    --name pgadmin4 \
    dpage/pgadmin4

Then access pgAdmin4 at: http://localhost:8080
- Email: admin@example.com
- Password: admin123

To connect pgAdmin4 to your PostgreSQL:
1. Open http://localhost:8080
2. Login with the credentials above
3. Add new server with:
   - Host: host.docker.internal (or your server IP)
   - Port: 5432
   - Username: postgres
   - Password: (your postgres password)

Your embedding service is working fine - it's using the sentence-transformers/all-mpnet-base-v2 model with 768 dimensions.

**Check resource usage:**
```bash
# Real-time resource monitoring
docker stats

# Detailed container info
docker inspect postgres_pgvector | grep -A 5 "Memory\|Cpu"
```

**Check if pgvector is enabled:**
```bash
# Connect to your PostgreSQL container
docker exec -it postgres_pgvector psql -U postgres

# Then run inside PostgreSQL:
\dx
SELECT * FROM pg_extension WHERE extname = 'vector';
```

**Enable pgvector if not already enabled:**
```bash
# From terminal
docker exec -it postgres_pgvector psql -U postgres -c "CREATE EXTENSION IF NOT EXISTS vector;"

# Verify it worked
docker exec -it postgres_pgvector psql -U postgres -c "SELECT vector_dims('[1,2,3]'::vector);"
```

**From pgAdmin4 UI:**
1. Access pgAdmin4 at `http://localhost:8080` (port 8080 based on your container)
2. Add server connection using:
   - Host: `postgres_pgvector` (container name)
   - Port: `5432`
   - Database: your database name
3. Navigate to Extensions and create the `vector` extension

The markdown file above contains everything you need to replicate this setup on any new server, including resource monitoring, troubleshooting, and vector search examples.