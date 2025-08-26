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