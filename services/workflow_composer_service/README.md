# Workflow Composer Service

A dynamic workflow composition system that provides schema introspection and dynamic tool discovery for LLM-driven workflow creation using Temporal activities.

## Features

- **GraphQL Schema for Dynamic Tool Discovery**: Full introspection capabilities for services and activities
- **Service Registry**: Centralized metadata store for all available activities across services
- **Functional Workflow Engine**: Execute workflows defined as data structures rather than code
- **LLM-Friendly API**: REST endpoints optimized for LLM interaction and composition
- **Real-time Execution**: Integration with Temporal for reliable workflow execution
- **Observability**: Built-in monitoring and instrumentation

## Architecture

This service acts as the orchestration layer that allows LLMs to:

1. **Discover** available activities across all services through schema introspection
2. **Compose** workflows by connecting activities with proper data flow
3. **Execute** workflows with structured inputs and error handling
4. **Monitor** workflow execution with detailed observability

## Key Components

### Production Discovery System (`docker_production_discovery.py`)
- Real-time service discovery using Temporal API and HTTP metadata endpoints
- Discovers active task queues and running workers
- Fetches complete activity schemas with input/output specifications
- Provides live system observability and health status

### GraphQL API (`gql_schema/schema.py`)
- Complete introspection of available capabilities using production discovery
- Structured queries for real-time activity discovery
- Mutations for workflow creation and YAML generation
- HTTP-based API with full service metadata

### GraphQL API
- High-level API for LLM interaction
- Workflow composition and validation
- Activity parameter mapping and data flow

## Usage

### Starting the Service

```bash
# Install dependencies
pip install -r requirements.txt

# Start the API server
python main.py
```

The service will start on port 8000 with the following endpoints:

- `GET /services` - Get all available services and activities
- `GET /activities/{activity_id}` - Get details for a specific activity
- `POST /workflows` - Create a new workflow definition
- `GET /workflows` - List all available workflows
- `POST /execute/{workflow_name}` - Execute a workflow
- `/graphql` - GraphQL endpoint for schema introspection (if Strawberry is available)

### Configuration

The service uses dynamic discovery with no static configuration files required. All services and activities are discovered automatically from running Temporal workers and their metadata endpoints.

### Example LLM Interaction

```python
# 1. Discover available capabilities
services = requests.get("http://localhost:8000/services").json()

# 2. Compose a workflow
workflow_def = {
    "name": "CustomSearchWorkflow",
    "description": "Custom search with ranking",
    "activities": [
        {
            "id": "utility_service.validate_inputs_activity",
            "result_key": "validated_inputs",
            "parameters": [
                {"name": "inputs", "source": "input", "input_name": "user_input"}
            ]
        },
        {
            "id": "retriever_service.semantic_search_activity", 
            "result_key": "search_results",
            "parameters": [
                {"name": "query_embedding", "source": "input", "input_name": "query_vector"},
                {"name": "collection_name", "source": "input", "input_name": "collection"}
            ]
        }
    ]
}

# 3. Create the workflow
response = requests.post("http://localhost:8000/workflows", json=workflow_def)

# 4. Execute the workflow
execution = requests.post("http://localhost:8000/execute/CustomSearchWorkflow", 
                         json={"user_input": {...}, "query_vector": [...], "collection": "docs"})
```

## Integration with Other Services

This service is designed to work with any Temporal-based service that registers its activities. Services should:

1. Define their activities in the registry configuration
2. Run Temporal workers that handle activity execution
3. Use consistent task queue naming conventions

## Development

### Running Tests

```bash
pytest tests/
```

### Adding New Activities

New activities are automatically discovered when services start up. The dynamic discovery system will:
1. Detect new services via Temporal task queue monitoring
2. Query service metadata endpoints for activity definitions
3. Make activities immediately available to LLMs without configuration changes

### Creating Workflow Examples

Add new patterns to `config/workflow_examples.yaml` to help guide LLM workflow composition.

## Benefits

- **Dynamic Discovery**: No hardcoded tool lists - everything is discoverable at runtime
- **Self-Documenting**: Rich metadata makes the system self-explanatory
- **Extensible**: New services and activities are automatically integrated
- **LLM-Optimized**: Designed specifically for LLM interaction patterns
- **Reliable**: Built on Temporal for robust workflow execution
- **Observable**: Full instrumentation and monitoring capabilities

This architecture enables true "workflow as data" patterns where LLMs can understand, compose, and execute complex business processes by discovering and connecting available activities dynamically.

### GraphQL API

In addition to the REST API, the service provides a fully-featured GraphQL API with:

- **GraphiQL Playground**: Interactive query interface at `/graphql`
- **Schema Introspection**: Complete discovery of available queries and mutations
- **Type Safety**: Strongly typed schema with validation
- **Flexible Queries**: Request exactly the data you need

#### GraphQL Endpoints

- `GET/POST /graphql` - GraphQL endpoint for queries and mutations
- **GraphiQL Playground**: Interactive interface available at `/graphql`

#### Sample GraphQL Queries

**Discover all services:**
```graphql
{
  services {
    name
    activities {
      name
      description
      parameters {
        name
        type
        required
      }
    }
  }
}
```

**Create a workflow:**
```graphql
mutation {
  createWorkflow(
    name: "MyWorkflow"
    description: "Custom workflow"
    activities: "[{\"id\": \"utility_service.validate_inputs_activity\", \"parameters\": []}]"
  )
}
```

See `graphql_examples.md` for more comprehensive examples.

### API Documentation

- **REST API**: Swagger/OpenAPI docs at `/docs`
- **GraphQL**: Interactive GraphiQL playground at `/graphql`
- **Root**: Service overview and links at `/`
