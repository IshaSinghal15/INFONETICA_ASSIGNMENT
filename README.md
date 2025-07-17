# Configurable Workflow Engine

A minimal backend service that provides a configurable state machine API for workflow management.

## Quick Start

### Prerequisites
- .NET 8.0 SDK

### Running the Application
```bash
dotnet run
```

The API will be available at `http://localhost:5000` or `https://localhost:5001`.

### Project Structure
```
WorkflowEngine/
├── Program.cs                          # Main application entry point and API routes
├── Models/WorkflowModels.cs           # Core domain models
├── Services/WorkflowService.cs        # Business logic and validation
├── Repositories/InMemoryWorkflowRepository.cs # Data storage
└── WorkflowEngine.csproj             # Project configuration
```

## API Endpoints

### Workflow Definitions
- `POST /api/workflows` - Create a new workflow definition
- `GET /api/workflows/{id}` - Get a specific workflow definition
- `GET /api/workflows` - List all workflow definitions

### Workflow Instances
- `POST /api/workflows/{definitionId}/instances` - Start a new workflow instance
- `GET /api/instances/{id}` - Get a specific workflow instance
- `GET /api/instances` - List all workflow instances
- `POST /api/instances/{instanceId}/actions/{actionId}` - Execute an action

## Example Usage

### 1. Create a Workflow Definition
```bash
curl -X POST http://localhost:5000/api/workflows \
  -H "Content-Type: application/json" \
  -d '{
    "id": "order-processing",
    "name": "Order Processing Workflow",
    "description": "Handles order lifecycle",
    "states": [
      {
        "id": "draft",
        "name": "Draft",
        "isInitial": true,
        "isFinal": false,
        "enabled": true
      },
      {
        "id": "submitted",
        "name": "Submitted",
        "isInitial": false,
        "isFinal": false,
        "enabled": true
      },
      {
        "id": "approved",
        "name": "Approved",
        "isInitial": false,
        "isFinal": false,
        "enabled": true
      },
      {
        "id": "completed",
        "name": "Completed",
        "isInitial": false,
        "isFinal": true,
        "enabled": true
      }
    ],
    "actions": [
      {
        "id": "submit",
        "name": "Submit Order",
        "enabled": true,
        "fromStates": ["draft"],
        "toState": "submitted"
      },
      {
        "id": "approve",
        "name": "Approve Order",
        "enabled": true,
        "fromStates": ["submitted"],
        "toState": "approved"
      },
      {
        "id": "complete",
        "name": "Complete Order",
        "enabled": true,
        "fromStates": ["approved"],
        "toState": "completed"
      }
    ]
  }'
```

### 2. Start a Workflow Instance
```bash
curl -X POST http://localhost:5000/api/workflows/order-processing/instances
```

### 3. Execute an Action
```bash
curl -X POST http://localhost:5000/api/instances/{instanceId}/actions/submit
```

## Core Concepts

### State
- **id**: Unique identifier
- **name**: Human-readable name
- **isInitial**: Exactly one state must be marked as initial
- **isFinal**: States marked as final cannot have outgoing actions
- **enabled**: Controls whether the state is active

### Action (Transition)
- **id**: Unique identifier
- **name**: Human-readable name
- **enabled**: Controls whether the action can be executed
- **fromStates**: List of source states from which this action can be executed
- **toState**: Target state after action execution

### Workflow Definition
- Collection of states and actions
- Must have exactly one initial state
- Actions must reference valid states

### Workflow Instance
- Runtime instance of a workflow definition
- Tracks current state and execution history
- Starts at the definition's initial state

## Validation Rules

The service enforces these validation rules:

### Definition Validation
- Must have exactly one initial state
- No duplicate state or action IDs
- All action state references must be valid
- Required fields cannot be empty

### Runtime Validation
- Actions can only be executed if enabled
- Current state must be in the action's fromStates
- Cannot execute actions on final states
- Instance must exist and reference a valid definition

## Design Decisions & Assumptions

### Architecture
- **Minimal API**: Uses ASP.NET Core minimal APIs for simplicity
- **Dependency Injection**: Service layer is properly abstracted
- **Repository Pattern**: Data access is abstracted for future extensibility
- **In-Memory Storage**: Uses ConcurrentDictionary for thread-safe operations

### Assumptions
- **Single Action Per Request**: Each API call executes exactly one action
- **Synchronous Operations**: All operations are synchronous for simplicity
- **No Authentication**: Authentication/authorization is out of scope
- **UTC Timestamps**: All timestamps are stored in UTC
- **Case-Sensitive IDs**: State and action IDs are case-sensitive

### Trade-offs
- **Simplicity over Performance**: Chose clear code over optimization
- **Validation at Service Layer**: Business rules are enforced in the service
- **No Persistence**: Data is lost when the application restarts
- **Limited Error Handling**: Basic error responses for the scope

## Known Limitations

1. **No Persistence**: Data is stored in memory only
2. **No Concurrency Control**: Multiple simultaneous actions on the same instance could cause race conditions
3. **No Bulk Operations**: All operations are single-entity focused
4. **No Workflow Versioning**: Definitions cannot be updated after creation
5. **No Complex Conditions**: Actions are simple state transitions only
6. **No Rollback**: No ability to undo executed actions

## Future Enhancements

With more time, these features could be added:

1. **Database Persistence**: Entity Framework Core integration
2. **Workflow Versioning**: Support for updating definitions
3. **Conditional Actions**: Add support for guards/conditions
4. **Parallel Execution**: Support for parallel workflow paths
5. **Event Sourcing**: Store all state changes as events
6. **REST API Documentation**: OpenAPI/Swagger integration
7. **Validation Middleware**: Centralized request validation
8. **Logging**: Structured logging with Serilog
9. **Health Checks**: Application health monitoring
10. **Unit Tests**: Comprehensive test coverage

## Error Handling

The API returns appropriate HTTP status codes:
- **200 OK**: Successful operations
- **201 Created**: Successfully created resources
- **400 Bad Request**: Validation errors or invalid operations
- **404 Not Found**: Resource not found

Error responses include descriptive messages to help with debugging.
