# Sarch

**Sarch** (**S**oftware **A**rchitecture **R**easoning, **C**hecking and **H**ardening) is a model-driven tool for the security analysis of software architectures. It provides an architecture description language (ADL) for describing software systems and automates the identification of structural security weaknesses, their exploitability classification, and the suggestion of architectural countermeasures.

---

## What Sarch does

1. **Modeling:** Sarch describes a software architecture using the Sarch ADL. The language supports component-and-connector and deployment structures.

2. **Checking:** Sarch analyses the architecture model against a security knowledge graph and identifies structural weaknesses mapped to CWE identifiers. In addition, Eeach detected weakness is classified by exploitability level (High / Medium / Low) using CAPEC attack patterns and OWASP risk categories.

4. **Redesign:** Sarch proposes architectural countermeasures based on architectural tactics and patterns for security, generating a revised model.

---

## Architecture

Sarch is composed of six components:

| Component | Type | Port | Responsibility | Technologies |
|---|---|---|---|---|
| `sarch-knows-db` | Graph database | — | Stores the knowledge graph (architecture models, weaknesses, tactics, patterns) | Neo4j (AuraDB) |
| `sarch-models-ms` | Microservice | 4000 | Parses and stores architecture models in the knowledge graph | Python, Flask, (textX for the Sarch ADL grammar) |
| `sarch-checks-ms` | Microservice | 4100 | Identifies security weaknesses and classifies exploitability | Python, Flask |
| `sarch-redesigns-ms` | Microservice | 4200 | Generates redesigned architecture models applying countermeasures | Python, Flask |
| `sarch-ag` | API Gateway | 6500 | Single entry point — routes requests to the microservices | Python, Flask |
| `sarch-fe` | Web Frontend | 8080 | Modeling, Checking, and Redesign views | JavaScript, Express |

---

## Requirements

- [Docker](https://docs.docker.com/get-docker/) and [Docker Compose](https://docs.docker.com/compose/)

---

## Deployment

### 1. Clone this repository

### 2. Create a `.env` file

Create a `.env` file in the same directory as `docker-compose.yml`:

```env
NEO4J_URI=
NEO4J_USER=
NEO4J_PASSWORD=
NEO4J_DATABASE=
```

These are the connection variables required to connect to the database (sarch-knows-db). Their values are available at the same link as the **demo video** (for ECSA 2026 - Tool Paper).

### 3. Pull and start all components

```bash
docker compose up --build
```

Docker will pull all images from Docker Hub automatically. No source code is required.

### 4. Open the tool

Navigate to [http://localhost:8080](http://localhost:8080) in your browser.

---

## Usage

### Modeling

1. Open the **Modeling** view.
2. Write or paste a Sarch ADL model in the editor. The editor provides syntax highlighting and live validation.
3. Click **Save Model** to persist the architecture in the knowledge graph.

**Minimal example:**

```
architecture:

  software_system: my_system

  structures:

    component_and_connector:

      elements:

        component frontend (presentation > web_frontend)
        component api_gw (communication > api_gateway)
        component db (data > database > relational : dbms: PostgreSQL)
        connector http (procedure_call > REST)
        connector db_conn (repository > db_connector)

      relations:
      
        attachment (frontend -> http -> api)
        attachment (api_gw -> db_conn -> db)
```

### Checking

1. Open the **Checking** view and select a saved architecture.
2. Click **Identify Weaknesses** — affected nodes are highlighted in blue on the graph.
3. Click **Classify Exploitability** — nodes are coloured by severity: red (High), orange (Medium), yellow (Low).
4. Click **Go to Redesign** to proceed with countermeasure generation.

### Redesign

1. Select the weaknesses to address from the list.
2. Click **Apply Countermeasures** — Sarch generates a revised model with the recommended architectural changes highlighted in the graph.
3. Save the new model to the knowledge graph using the embedded editor.

---

## Sarch ADL reference

### Component types

| Layer | Types |
|---|---|
| `presentation` | `web_frontend`, `mobile_frontend`, `desktop_frontend` |
| `monolithic` | `monolith` |
| `logic` | `microservice` *(requires* `programming_language`*)*, `backend` |
| `data` | `database > relational\|no_sql\|cache_store\|object_store\|file_store` |
| `communication` | `api_gateway`, `message_broker`, `orchestrator`, `api_composer`, `bff`, `service_mesh`, `event_bus` |
| `infrastructure` | `load_balancer`, `reverse_proxy`, `web_application_firewall`, `cdn`, `dns_resolver` |

### Connector types

| Category | Types |
|---|---|
| `procedure_call` | `HTTP`, `REST`, `GraphQL`, `gRPC`, `SOAP`, `WebSocket`, `RPC`, `SSE`, `long_poll` |
| `repository` | `db_connector`, `file_storage_connector`, `cache_connector`, `object_storage_connector` |
| `data_flow` | `socket`, `pipe`, `stream` |
| `event` | `AMQP`, `MQTT`, `Kafka`, `webhook` |

### Deployment elements

```
network <name>             : cidr: "<cidr>"
subnet  <name>             : cidr: "<cidr>"  visibility: public|private
container_cluster <name>   : orchestrator: kubernetes|ecs|docker_swarm|nomad
node <name>                : os: "<os>"
container <name>           : image: "<image>"
execution_environment <n>  : type: jvm|node_runtime|python_runtime|go_runtime|browser
software_element <name>    : instance_of: <component>  port_mapping: <host>:<container>
```

### Deployment relations

```
deployed_in ( <software_element> -> <container|execution_environment> )
runs_on     ( <container|ee>     -> <node|container_cluster> )
belongs_to  ( <node|subnet>      -> <subnet|network> )
connected_to( <subnet>           -> <subnet> )
```

---

## Examples

The `examples/` folder contains ready-to-use architecture models:

| File | System | Description |
|---|---|---|
| `SpringPetClinicMicroservices.sarch` | [Spring PetClinic Microservices](https://github.com/spring-petclinic/spring-petclinic-microservices) | Full microservices system with 4 business services, observability stack, and Docker-based deployment |

To use an example, open the **Modeling** view, paste the file contents into the editor, and click **Save Model**.

---

## License

Academic research prototype. Contact: javergarav@unal.edu.co
