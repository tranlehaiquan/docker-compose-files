# Redis + PostgreSQL Docker Setup

This setup includes Redis and PostgreSQL with persistent storage and optional web UIs.

## Quick Start

```bash
# Start all services
docker compose up -d

# View logs
docker compose logs -f

# Stop all services
docker compose down

# Stop and remove volumes (data will be lost)
docker compose down -v
```

## Features

- **Production-ready configuration**: Includes resource limits and health checks.
- **Security**: Redis password protection enabled by default.
- **Persistence**: Data volumes for both Redis and PostgreSQL.
- **Management UIs**: Pre-configured pgAdmin and Redis Commander.

## Services

### Redis

- **Port**: 6379
- **Image**: redis:7-alpine
- **Persistence**: Enabled (RDB + AOF)
- **Password**: Set via `REDIS_PASSWORD` in `.env`
- **Data volume**: `redis_data`

### PostgreSQL

- **Port**: 5432
- **Image**: postgres:16-alpine
- **Database**: myapp
- **User**: postgres
- **Password**: postgres
- **Data volume**: `postgres_data`

### Redis Commander (Optional Web UI)

- **Port**: 8081
- **URL**: http://localhost:8081
- To disable, comment out the `redis-commander` service in docker compose.yml

### pgAdmin (Optional Web UI)

- **Port**: 5050
- **URL**: http://localhost:5050
- **Email**: admin@admin.com
- **Password**: admin
- To disable, comment out the `pgadmin` service in docker compose.yml

## Connect from Node.js

### Redis Connection

```javascript
// Using ioredis (recommended)
const Redis = require("ioredis");
const redis = new Redis({
  host: "localhost",
  port: 6379,
  password: process.env.REDIS_PASSWORD || "redis_secure_password",
});

// Using node-redis
const redis = require("redis");
const password = process.env.REDIS_PASSWORD || "redis_secure_password";
const client = redis.createClient({
  url: `redis://:${password}@localhost:6379`,
});
await client.connect();
```

### PostgreSQL Connection

```javascript
// Using pg (node-postgres)
const { Pool } = require("pg");
const pool = new Pool({
  host: "localhost",
  port: 5432,
  database: process.env.POSTGRES_DB || "myapp",
  user: process.env.POSTGRES_USER || "postgres",
  password: process.env.POSTGRES_PASSWORD || "postgres",
});

// Using Prisma
// In your .env file:
// DATABASE_URL="postgresql://postgres:postgres@localhost:5432/myapp"

// Using TypeORM
const dataSource = new DataSource({
  type: "postgres",
  host: "localhost",
  port: 5432,
  username: process.env.POSTGRES_USER || "postgres",
  password: process.env.POSTGRES_PASSWORD || "postgres",
  database: process.env.POSTGRES_DB || "myapp",
});

// Using Sequelize
const sequelize = new Sequelize(
  process.env.POSTGRES_DB || "myapp",
  process.env.POSTGRES_USER || "postgres",
  process.env.POSTGRES_PASSWORD || "postgres",
  {
    host: "localhost",
    port: 5432,
    dialect: "postgres",
  },
);
```

## Configuration

### Redis

Redis is configured with AOF persistence enabled and requires a password (set in `.env`). You can add custom configuration by:

1. Creating a `redis.conf` file
2. Mounting it in docker compose: `- ./redis.conf:/usr/local/etc/redis/redis.conf`
3. Updating the command: `redis-server /usr/local/etc/redis/redis.conf`

### PostgreSQL

Edit environment variables in `docker compose.yml`:

- `POSTGRES_DB`: Database name
- `POSTGRES_USER`: Username
- `POSTGRES_PASSWORD`: Password

To run initial setup scripts, create an `init.sql` file and mount it:

```yaml
volumes:
  - ./init.sql:/docker-entrypoint-initdb.d/init.sql
```

## Security Notes

⚠️ **For Development Only**

- Default passwords are used (defined in `.env`)
- No SSL/TLS encryption
- Services bind to all interfaces (0.0.0.0)

**For Production:**

1. Change default passwords in `.env`
2. Use Docker secrets for sensitive data
3. Enable SSL/TLS for both services
4. Bind to specific IPs
5. Ensure `.env` is in `.gitignore`

## Common Commands

### Redis

```bash
# Connect to Redis CLI
docker compose exec redis redis-cli

# Test connection
docker compose exec redis redis-cli ping

# Monitor commands
docker compose exec redis redis-cli monitor

# Check memory usage
docker compose exec redis redis-cli INFO memory
```

### PostgreSQL

```bash
# Connect to PostgreSQL CLI
docker compose exec postgres psql -U postgres -d myapp

# List databases
docker compose exec postgres psql -U postgres -c "\l"

# List tables
docker compose exec postgres psql -U postgres -d myapp -c "\dt"

# Backup database
docker compose exec postgres pg_dump -U postgres myapp > backup.sql

# Restore database
docker compose exec -T postgres psql -U postgres myapp < backup.sql

# Run SQL file
docker compose exec -T postgres psql -U postgres myapp < your-script.sql
```

## Useful Redis Commands

```bash
# Set a key
SET mykey "Hello"

# Get a key
GET mykey

# List all keys
KEYS *

# Delete a key
DEL mykey

# Set with expiration (in seconds)
SETEX mykey 60 "Expires in 60 seconds"
```

## Useful PostgreSQL Commands

```sql
-- Create a new table
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    price DECIMAL(10, 2),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Insert data
INSERT INTO products (name, price) VALUES ('Product 1', 29.99);

-- Query data
SELECT * FROM products;

-- Update data
UPDATE products SET price = 24.99 WHERE id = 1;

-- Delete data
DELETE FROM products WHERE id = 1;

-- Create index
CREATE INDEX idx_products_name ON products(name);
```

## Environment Variables

The project includes a `.env` file with default values. **You should change these for any production use.**

```env
# PostgreSQL
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
POSTGRES_DB=myapp

# pgAdmin
PGADMIN_DEFAULT_EMAIL=admin@admin.com
PGADMIN_DEFAULT_PASSWORD=admin

# Redis
REDIS_PASSWORD=redis_secure_password
```

The `docker-compose.yml` is already configured to use these variables.
