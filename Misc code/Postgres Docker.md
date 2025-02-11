1. Starta docker postgres containern.
2. `docker exec -it (namn på container) psql -U (användarnamn)` för att komma in.
3. `\l` lista alla databaser. `\c (namn på databas)` för att gå in på den.


#### Set up container with volume

```bash
docker run -d \
  --name postgres_container \
  -e POSTGRES_PASSWORD=mysecretpassword \
  -v postgres_data:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:latest
```

