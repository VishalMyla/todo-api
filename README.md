# TODO API




## Project Structure

Talking specifically about microservices **only**, the structure I like to recommend is the following, everything using `<` and `>` depends on the domain being implemented and the bounded context being defined.

- [ ] `build/`: defines the code used for creating infrastructure as well as docker containers.
  - [ ] `<cloud-providers>/`: define concrete cloud provider.
  - [ ] `<executableN>/`: contains a Dockerfile used for building the binary.
- [ ] `cmd/`
  - [ ] `<primary-server>/`: uses primary database.
  - [ ] `<replica-server>/`: uses readonly databases.
  - [ ] `<binaryN>/`
- [X] `db/`
  - [X] `migrations/`: contains database migrations.
  - [ ] `seeds/`: contains file meant to populate basic database values.
- [ ] `internal/`: defines the _core domain_.
  - [ ] `<datastoreN>/`: a concrete _repository_ used by the domain, for example `postgresql`
  - [ ] `http/`: defines HTTP Handlers.
  - [ ] `service/`: orchestrates use cases and manages transactions.
- [X] `pkg/` public API meant to be imported by other Go package.

There are cases where requiring a new bounded context is needed, in those cases the recommendation would be to
define a package like `internal/<bounded-context>` that then should follow the same structure, for example:

* `internal/<bounded-context>/`
  * `internal/<bounded-context>/<datastoreN>`
  * `internal/<bounded-context>/http`
  * `internal/<bounded-context>/service`

## Tools

Please refer to the documentation in [internal/tools/](internal/tools/README.md).



## Docker Containers

Please notice in order to run this project locally you need to run a few programs in advance, if you use Docker please refer to the concrete instructions in [`docs/`](docs/) for more details.

There's also a [docker-compose.yml](docker-compose.yml), covered in Building Microservices In Go: Containerization with Docker, however like I mentioned in the video you have to execute `docker-compose` in multiple steps.

Notice that because of the way RabbitMQ and Kafka are being used they are sort of competing with each other, so at the moment we either have to enable Kafka and disable RabbitMQ or the other way around in both the code and the `docker-compose.yml` file, in either case there are Dockerfiles and services defined that cover building and running them.

The following instructions are confirmed to work with docker compose v2.15.1.

* Run `docker-compose up`, if you're using `rabbitmq` or `kafka` you may see the _rest-server_ and _elasticsearch-indexer_ services fail because those services take too long to start, in that case use any of the following instructions to manually start those services after the dependant server is ready:
    * If you're planning to use RabbitMQ, run `docker-compose up rest-server elasticsearch-indexer-rabbitmq`.
    * If you're planning to use Kafka, run `docker-compose up rest-server elasticsearch-indexer-kafka`.
* For building the service images you can use:
    * `rest-server` image: `docker-compose build rest-server`.
    * `elasticsearch-indexer-rabbitmq` image: `docker-compose build elasticsearch-indexer-rabbitmq`.
    * `elasticsearch-indexer-kafka` image: `docker-compose build elasticsearch-indexer-kafka`.
    * `elasticsearch-indexer-redis` image: `docker-compose build elasticsearch-indexer-redis`.
* Run `docker-compose run rest-server migrate -path /api/migrations/ -database postgres://user:password@postgres:5432/dbname\?sslmode=disable up` to have everything working correctly.
* Finally interact with the API using Swagger UI: http://127.0.0.1:9234/static/swagger-ui/

## Diagrams

To start a local HTTP server that serves a graphical editor:

```
mdl serve github.com/VishalMyla/todo-api/internal/doc -dir docs/diagrams/
```

To generate JSON artifact for uploading to [structurizr](https://structurizr.com/):

```
stz gen github.com/VishalMyla/todo-api/internal/doc
```
