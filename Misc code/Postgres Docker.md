1. Starta docker postgres containern.
2. `docker exec -it (namn på container) psql -U (användarnamn)` för att komma in.
3. `\l` lista alla databaser. `\c (namn på databas)` för att gå in på den.