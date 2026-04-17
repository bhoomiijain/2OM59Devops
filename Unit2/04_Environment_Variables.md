# Environment Variables

Env vars = key-value data containers can read

Why? Change app without rebuilding image. Used for:
- Configuration
- Secrets (passwords)
- Different for dev/prod

Using -e flag:
docker run -e MY_VAR=hello ubuntu echo $MY_VAR

Output: hello. Only inside container - host doesn't see.

Multiple variables:
docker run -e VAR1=value1 -e VAR2=value2 nginx

MySQL Example:
docker run -d -e MYSQL_ROOT_PASSWORD=root123 -e MYSQL_DATABASE=college mysql:8

Password = root123, Database = college

In Dockerfile:
FROM ubuntu
ENV APP_PORT=8080
ENV APP_ENV=production

Defaults set in image. Can override with -e when running.

Method 1: Using -e flag with docker run

Single variable:
docker run -e MY_VAR=value httpd env

Named container with env:
docker run -it -e MY_NAME=Bhoomi ubuntu bash
Inside: echo $MY_NAME = Bhoomi

Note: MY_NAME only inside container, host not affected

Multiple env variables:
docker run -e APP_ENV=production -e APP_VERSION=1.0 nginx

MySQL required variables:
docker run -d -e MYSQL_ROOT_PASSWORD=root123 -e MYSQL_DATABASE=college -e MYSQL_USER=admin -e MYSQL_PASSWORD=admin123 mysql:8

Without these MySQL container fails to start

Passing from host system:
export APP_PORT=8080
export API_KEY=12345
docker run -e APP_PORT nginx env

Docker reads from host env

Method 2: Using --env-file (Clean & Professional)

Create .env file:
DB_HOST=localhost
DB_USER=root
DB_PASS=secret
APP_ENV=production

Run container with file:
docker run --env-file .env myapp

Best practice for production - keeps secrets out of command history

Checking env variables inside container:
docker exec -it <id> bash = open terminal
env = view all env variables
echo $MY_NAME = check specific variable
echo $APP_ENV = check another
```

---

## Practical Example from Class

```bash
# Run Ubuntu with env variable + volume + name
docker run -it \
  --name my-app \
  -e APP_ENV=production \
  -v ~/app/data:/data \
  ubuntu bash

# Inside container:
echo $APP_ENV          # → production
echo "Hello" > /data/test.txt
cat /data/test.txt
exit

# On host:
echo $APP_ENV          # → empty (host not affected)
```

---

## Important Notes

- Variables are scoped **only to the container**
- Host system variables are NOT changed
- Use **UPPER_CASE** naming convention for env variables (e.g., `MY_NAME`, `APP_ENV`)
- For secrets, prefer `--env-file` or Docker Secrets (advanced)

---

