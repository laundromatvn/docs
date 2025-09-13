# Base Schema Documentation

## Introduction

This document defines the base JSON schema structure used across all MQTT message flows in the Laundry Management System (LMS). All messages follow this unified structure to ensure consistency, compatibility, and ease of implementation across different components.

## Base Schema Structure

```json
{
  "version": "string",
  "event_type": "string",
  "timestamp": "string",
  "correlation_id": "string",
  "controller_id": "string",
  "store_id": "string | null",
  "payload": "object | null"
}
```

## Field Descriptions

### `version`
- **Type:** `string`
- **Format:** Semantic version (e.g., "1.0.0")
- **Purpose:** Indicates the schema version for message compatibility and future evolution
- **Required:** Yes
- **Example:** `"1.0.0"`
- **Note:** Used for versioning and backward compatibility

### `event_type`
- **Type:** `string`
- **Format:** Enum value
- **Purpose:** Identifies the type of event or action
- **Required:** Yes
- **Examples:**
  - `"controller_init_request"` - Controller requesting initialization
  - `"controller_init_response"` - Backend responding to initialization
  - `"store_assignment"` - Backend assigning store to controller
  - `"start_washer"` - Action to start washer machine
  - `"start_dryer"` - Action to start dryer machine
  - `"machine_state"` - Controller reporting machine state
- **Note:** Each flow defines its specific event_type values

### `timestamp`
- **Type:** `string`
- **Format:** ISO 8601 with UTC+7 timezone (`YYYY-MM-DDTHH:mm:ss+07:00`)
- **Purpose:** Records when the message was generated
- **Required:** Yes
- **Example:** `"2024-01-15T14:30:25+07:00"`
- **Note:** All timestamps in messages must be in UTC+7, but stored in UTC in the database

### `correlation_id`
- **Type:** `string`
- **Format:** UUID v4
- **Purpose:** Unique identifier to track request/response pairs and message correlation
- **Required:** Yes
- **Example:** `"550e8400-e29b-41d4-a716-446655440000"`
- **Note:** Used for message correlation, debugging, and tracing message flows

### `controller_id`
- **Type:** `string`
- **Format:** UUID v4
- **Purpose:** Unique identifier for the IoT Controller
- **Required:** Yes
- **Example:** `"6ba7b810-9dad-11d1-80b4-00c04fd430c8"`
- **Note:** Identifies the specific controller involved in the message

### `store_id`
- **Type:** `string | null`
- **Format:** UUID v4 or null
- **Purpose:** Identifier of the store the controller is assigned to
- **Required:** Yes
- **Example:** `"6ba7b811-9dad-11d1-80b4-00c04fd430c8"` or `null`
- **Note:** May be null if controller is not yet assigned to a store

### `payload`
- **Type:** `object | null`
- **Format:** JSON object or null
- **Purpose:** Contains event-specific data and configuration
- **Required:** Yes
- **Note:** Structure varies by event_type - see specific flow documentation for details

## Common Payload Patterns

### Status Payload
Used for status reporting and responses:
```json
{
  "status": "success | error | PENDING_ASSIGNMENT | ASSIGNED"
}
```

### Machine Control Payload
Used for machine control actions:
```json
{
  "machine_type": "washer | dryer | null",
  "relay_id": "number",
  "pulse_duration": "number",
  "value": "number"
}
```

### Machine State Payload
Used for machine state reporting:
```json
{
  "machines": [
    {
      "machine_type": "washer | dryer | null",
      "relay_id": "number",
      "status": "idle | busy",
      "last_updated": "string"
    }
  ]
}
```

## Examples

### Initialization Request
```json
{
  "version": "1.0.0",
  "event_type": "controller_init_request",
  "timestamp": "2024-01-15T14:30:25+07:00",
  "correlation_id": "550e8400-e29b-41d4-a716-446655440000",
  "controller_id": "6ba7b810-9dad-11d1-80b4-00c04fd430c8",
  "store_id": null,
  "payload": null
}
```

### Initialization Response (Pending Assignment)
```json
{
  "version": "1.0.0",
  "event_type": "controller_init_response",
  "timestamp": "2024-01-15T14:30:26+07:00",
  "correlation_id": "550e8400-e29b-41d4-a716-446655440000",
  "controller_id": "6ba7b810-9dad-11d1-80b4-00c04fd430c8",
  "store_id": null,
  "payload": {
    "status": "PENDING_ASSIGNMENT"
  }
}
```

### Initialization Response (Assigned)
```json
{
  "version": "1.0.0",
  "event_type": "controller_init_response",
  "timestamp": "2024-01-15T14:30:26+07:00",
  "correlation_id": "550e8400-e29b-41d4-a716-446655440001",
  "controller_id": "6ba7b810-9dad-11d1-80b4-00c04fd430c8",
  "store_id": "6ba7b811-9dad-11d1-80b4-00c04fd430c8",
  "payload": {
    "status": "ASSIGNED",
    "pulse_duration": 5000
  }
}
```

### Store Assignment
```json
{
  "version": "1.0.0",
  "event_type": "store_assignment",
  "timestamp": "2024-01-15T14:35:10+07:00",
  "correlation_id": "550e8400-e29b-41d4-a716-446655440002",
  "controller_id": "6ba7b810-9dad-11d1-80b4-00c04fd430c8",
  "store_id": "6ba7b811-9dad-11d1-80b4-00c04fd430c8",
  "payload": {
    "status": "ASSIGNED",
    "pulse_duration": 5000
  }
}
```

### Start Machine Action
```json
{
  "version": "1.0.0",
  "event_type": "start_washer",
  "timestamp": "2024-01-15T15:30:25+07:00",
  "correlation_id": "550e8400-e29b-41d4-a716-446655440003",
  "controller_id": "6ba7b810-9dad-11d1-80b4-00c04fd430c8",
  "store_id": "6ba7b811-9dad-11d1-80b4-00c04fd430c8",
  "payload": {
    "machine_type": "washer",
    "relay_id": 2,
    "pulse_duration": 1000,
    "value": 1
  }
}
```

### Machine State Report
```json
{
  "version": "1.0.0",
  "event_type": "machine_state",
  "timestamp": "2024-01-15T16:00:00+07:00",
  "correlation_id": "550e8400-e29b-41d4-a716-446655440004",
  "controller_id": "6ba7b810-9dad-11d1-80b4-00c04fd430c8",
  "store_id": "6ba7b811-9dad-11d1-80b4-00c04fd430c8",
  "payload": {
    "machines": [
      {
        "machine_type": "washer",
        "relay_id": 1,
        "status": "busy",
        "last_updated": "2024-01-15T15:58:30+07:00"
      },
      {
        "machine_type": null,
        "relay_id": 2,
        "status": "idle",
        "last_updated": "2024-01-15T15:45:15+07:00"
      }
    ]
  }
}
```

## Validation Rules

### Required Fields
All fields in the base schema are required:
- `version`
- `event_type`
- `timestamp`
- `correlation_id`
- `controller_id`
- `store_id`
- `payload`

### Field Validation
- **UUIDs**: Must be valid UUID v4 format
- **Timestamps**: Must be valid ISO 8601 format with UTC+7 timezone
- **Version**: Must follow semantic versioning (e.g., "1.0.0")
- **Event Types**: Must match predefined values for each flow
- **Payload**: Must be valid JSON object or null

### Null Values
- `store_id` can be null for unassigned controllers
- `payload` can be null for simple requests without additional data

## Versioning Strategy

### Current Version
- **Base Schema Version**: 1.0.0
- **Supported Event Types**: As defined in individual flow documents

### Backward Compatibility
- New versions should maintain backward compatibility
- Deprecated fields should be marked but not removed immediately
- Version field allows clients to handle different schema versions

### Future Evolution
- New fields can be added to payload without breaking existing implementations
- New event_types can be added for new flows
- Base schema structure should remain stable

## Security Considerations

### Message Validation
- All messages should be validated against the schema
- Invalid messages should be rejected with appropriate error responses
- Consider implementing message authentication for production use

### Data Privacy
- Controller IDs and Store IDs are considered sensitive information
- Ensure proper access controls on MQTT topics
- Log message flows for audit purposes

### Rate Limiting
- Implement rate limiting to prevent message flooding
- Consider message size limits for performance
- Monitor for unusual message patterns