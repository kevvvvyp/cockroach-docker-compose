# cockroach-docker-compose
An example CockroachDB docker-compose with multiple nodes

```
version: '3.9'

services:

  roach-0:
    container_name: roach-0
    hostname: roach-0
    image: cockroachdb/cockroach:v23.1.24
    command: start  --insecure --join=roach-0,roach-1,roach-2 --listen-addr=roach-0:26257 --advertise-addr=roach-0:26257
    ports:
      - "26257:26257"
      - "8081:8080"
    environment:
      - 'ALLOW_EMPTY_PASSWORD=yes'
    healthcheck:
      test: [ "CMD", "curl", "-f", "http://localhost:8080/health" ]
      interval: 10s
      timeout: 5s
      retries: 5

  roach-1:
    container_name: roach-1
    hostname: roach-1
    image: cockroachdb/cockroach:v23.1.24
    command: start  --insecure --join=roach-0,roach-1,roach-2 --listen-addr=roach-1:26257 --advertise-addr=roach-1:26257
    environment:
      - 'ALLOW_EMPTY_PASSWORD=yes'

  roach-2:
    container_name: roach-2
    hostname: roach-2
    image: cockroachdb/cockroach:v23.1.24
    command: start  --insecure --join=roach-0,roach-1,roach-2 --listen-addr=roach-2:26257 --advertise-addr=roach-2:26257
    environment:
      - 'ALLOW_EMPTY_PASSWORD=yes'

  init:
    container_name: init
    image: cockroachdb/cockroach:v23.1.24
    volumes:
      - ./initdb-cockroach.sql:/cockroach/init.sql
    command: >
      bash -c "./cockroach.sh init --host=roach-0 --insecure 
      && ./cockroach.sh sql --host=roach-0 --insecure --file init.sql"
    depends_on:
      - roach-0

```
