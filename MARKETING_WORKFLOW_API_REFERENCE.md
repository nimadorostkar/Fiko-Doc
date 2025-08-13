# Marketing Workflow API Reference

Complete reference for all Marketing Workflow APIs with usage examples and routes.

## 📋 Table of Contents

- [Base URL & Authentication](#base-url--authentication)
- [Workflow Management](#workflow-management)
- [Trigger Management](#trigger-management)
- [Action Management](#action-management)
- [Condition Management](#condition-management)
- [Event Processing](#event-processing)
- [Monitoring & Analytics](#monitoring--analytics)
- [Common Response Formats](#common-response-formats)
- [Error Handling](#error-handling)

## 🌐 Base URL & Authentication

### Base URL
```
http://localhost:8000/api/v1/workflow/api/
```

### Authentication
All endpoints require authentication using JWT Bearer tokens:

```bash
# Include in all requests
Authorization: Bearer YOUR_JWT_TOKEN
Content-Type: application/json
```

### Get Authentication Token
```bash
# Login to get token (adjust endpoint as per your auth system)
curl -X POST http://localhost:8000/api/v1/usr/login/ \
  -H "Content-Type: application/json" \
  -d '{"username": "your_username", "password": "your_password"}'
```

## 🔄 Workflow Management

### List Workflows
```bash
GET /workflows/
```

**Parameters:**
- `page` (int): Page number for pagination
- `page_size` (int): Number of items per page (max 100)
- `status` (string): Filter by status (DRAFT, ACTIVE, PAUSED, ARCHIVED)
- `search` (string): Search in name and description

**Example:**
```bash
curl -H "Authorization: Bearer YOUR_TOKEN" \
  "http://localhost:8000/api/v1/workflow/api/workflows/?status=ACTIVE&page=1&page_size=10"
```

**Response:**
```json
{
  "count": 2,
  "next": null,
  "previous": null,
  "results": [
    {
      "id": "workflow-uuid",
      "name": "Welcome New Users",
      "description": "Send welcome message to new registrations",
      "status": "ACTIVE",
      "actions_count": 2,
      "triggers_count": 1,
      "executions_count": 15,
      "created_at": "2024-01-01T10:00:00Z",
      "updated_at": "2024-01-02T10:00:00Z"
    }
  ]
}
```

### Get Workflow Details
```bash
GET /workflows/{id}/
```

**Example:**
```bash
curl -H "Authorization: Bearer YOUR_TOKEN" \
  "http://localhost:8000/api/v1/workflow/api/workflows/workflow-uuid/"
```

**Response:**
```json
{
  "id": "workflow-uuid",
  "name": "Welcome New Users",
  "description": "Send welcome message to new registrations",
  "status": "ACTIVE",
  "max_executions": 1,
  "delay_between_executions": 0,
  "start_date": null,
  "end_date": null,
  "created_by": "admin",
  "actions": [
    {
      "id": "action-uuid",
      "action_name": "Send Welcome Message",
      "action_type": "send_message",
      "order": 1,
      "is_required": true
    }
  ],
  "triggers": [
    {
      "id": "trigger-uuid",
      "trigger_name": "New User Registration",
      "priority": 100,
      "is_active": true
    }
  ],
  "recent_executions": []
}
```

### Create Workflow
```bash
POST /workflows/
```

**Request Body:**
```json
{
  "name": "Discount Code Campaign",
  "description": "Send discount codes when customers ask",
  "status": "DRAFT",
  "max_executions": 1,
  "delay_between_executions": 3600
}
```

**Example:**
```bash
curl -X POST \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Discount Code Campaign",
    "description": "Send discount codes when customers ask",
    "status": "DRAFT"
  }' \
  "http://localhost:8000/api/v1/workflow/api/workflows/"
```

### Update Workflow
```bash
PUT /workflows/{id}/
PATCH /workflows/{id}/  # Partial update
```

**Example:**
```bash
curl -X PATCH \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"status": "ACTIVE"}' \
  "http://localhost:8000/api/v1/workflow/api/workflows/workflow-uuid/"
```

### Delete Workflow
```bash
DELETE /workflows/{id}/
```

**Example:**
```bash
curl -X DELETE \
  -H "Authorization: Bearer YOUR_TOKEN" \
  "http://localhost:8000/api/v1/workflow/api/workflows/workflow-uuid/"
```

### Workflow Actions

#### Activate Workflow
```bash
POST /workflows/{id}/activate/
```

```bash
curl -X POST \
  -H "Authorization: Bearer YOUR_TOKEN" \
  "http://localhost:8000/api/v1/workflow/api/workflows/workflow-uuid/activate/"
```

#### Pause Workflow
```bash
POST /workflows/{id}/pause/
```

#### Archive Workflow
```bash
POST /workflows/{id}/archive/
```

#### Reset Workflow to Draft
```bash
POST /workflows/{id}/reset/
```

#### Execute Workflow Manually
```bash
POST /workflows/{id}/execute/
```

**Request Body:**
```json
{
  "context": {
    "user": {
      "id": "customer-123",
      "email": "test@example.com",
      "first_name": "John"
    },
    "event": {
      "type": "MANUAL_TRIGGER",
      "data": {
        "reason": "Testing workflow"
      }
    }
  }
}
```

### Workflow Associations

#### Add Trigger to Workflow
```bash
POST /workflows/{id}/add_trigger/
```

**Request Body:**
```json
{
  "trigger_id": "trigger-uuid",
  "priority": 100,
  "specific_conditions": {
    "operator": "and",
    "conditions": [
      {
        "field": "user.tags",
        "operator": "contains",
        "value": "premium"
      }
    ]
  }
}
```

#### Remove Trigger from Workflow
```bash
POST /workflows/{id}/remove_trigger/
```

**Request Body:**
```json
{
  "trigger_id": "trigger-uuid"
}
```

#### Get Workflow Triggers
```bash
GET /workflows/{id}/triggers/
```

#### Get Workflow Actions
```bash
GET /workflows/{id}/actions/
```

#### Get Workflow Executions
```bash
GET /workflows/{id}/executions/
```

**Parameters:**
- `page`, `page_size`: Pagination
- `status`: Filter by execution status

#### Get Workflow Events
```bash
GET /workflows/{id}/events/
```

### Workflow Statistics
```bash
GET /workflows/statistics/
```

**Response:**
```json
{
  "total_workflows": 5,
  "active_workflows": 3,
  "draft_workflows": 1,
  "paused_workflows": 1,
  "total_executions": 150,
  "recent_executions": 25,
  "successful_executions": 140,
  "failed_executions": 10
}
```

## 🎯 Trigger Management

### List Triggers
```bash
GET /triggers/
```

**Parameters:**
- `trigger_type`: Filter by trigger type
- `is_active`: Filter by active status
- `search`: Search in name and description

**Example:**
```bash
curl -H "Authorization: Bearer YOUR_TOKEN" \
  "http://localhost:8000/api/v1/workflow/api/triggers/?trigger_type=MESSAGE_RECEIVED&is_active=true"
```

### Create Trigger
```bash
POST /triggers/
```

**Request Body:**
```json
{
  "name": "Coupon Request Trigger",
  "description": "Triggers when users ask for discounts",
  "trigger_type": "MESSAGE_RECEIVED",
  "filters": {
    "operator": "or",
    "conditions": [
      {
        "field": "event.data.content",
        "operator": "icontains",
        "value": "کد تخفیف"
      },
      {
        "field": "event.data.content",
        "operator": "icontains",
        "value": "coupon"
      }
    ]
  },
  "is_active": true
}
```

### Process Event (Main Event Processing Endpoint)
```bash
POST /triggers/process_event/
```

**Request Body:**
```json
{
  "event_type": "MESSAGE_RECEIVED",
  "data": {
    "message_id": "msg-123",
    "content": "سلام، کد تخفیف دارید؟",
    "timestamp": "2024-01-01T10:00:00Z"
  },
  "user_id": "customer-123",
  "conversation_id": "conv-456"
}
```

**Example:**
```bash
curl -X POST \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "event_type": "MESSAGE_RECEIVED",
    "data": {
      "message_id": "msg-123",
      "content": "Hello, do you have any discount codes?",
      "timestamp": "2024-01-01T10:00:00Z"
    },
    "user_id": "customer-123",
    "conversation_id": "conv-456"
  }' \
  "http://localhost:8000/api/v1/workflow/api/triggers/process_event/"
```

**Response:**
```json
{
  "success": true,
  "event_log_id": "event-log-uuid",
  "message": "Event queued for processing"
}
```

### Test Trigger
```bash
POST /triggers/{id}/test/
```

**Request Body:**
```json
{
  "context": {
    "event": {
      "data": {
        "content": "I need a coupon code"
      }
    },
    "user": {
      "tags": ["interested", "premium"]
    }
  }
}
```

### Get Trigger Workflows
```bash
GET /triggers/{id}/workflows/
```

### Activate/Deactivate Trigger
```bash
POST /triggers/{id}/activate/
POST /triggers/{id}/deactivate/
```

## ⚡ Action Management

### List Actions
```bash
GET /actions/
```

**Parameters:**
- `action_type`: Filter by action type
- `is_active`: Filter by active status

### Create Action
```bash
POST /actions/
```

**Send Message Action:**
```json
{
  "name": "Send Welcome Message",
  "description": "Send personalized welcome message",
  "action_type": "send_message",
  "configuration": {
    "message": "خوش آمدید {{user.first_name}}! چطور می‌تونم کمکتون کنم؟"
  },
  "order": 1,
  "delay": 0,
  "is_active": true
}
```

**Send Email Action:**
```json
{
  "name": "Send Welcome Email",
  "action_type": "send_email",
  "configuration": {
    "subject": "Welcome {{user.first_name}}!",
    "body": "Thank you for joining us!",
    "recipient": "{{user.email}}",
    "is_html": false
  }
}
```

**Add Tag Action:**
```json
{
  "name": "Add Interested Tag",
  "action_type": "add_tag",
  "configuration": {
    "tag_name": "interested"
  }
}
```

**Webhook Action:**
```json
{
  "name": "Notify External System",
  "action_type": "webhook",
  "configuration": {
    "url": "https://your-system.com/webhook",
    "method": "POST",
    "headers": {
      "Content-Type": "application/json",
      "Authorization": "Bearer your-token"
    },
    "payload": {
      "user_id": "{{user.id}}",
      "event_type": "{{event.type}}",
      "timestamp": "{{event.timestamp}}"
    }
  }
}
```

**Wait Action:**
```json
{
  "name": "Wait 10 Minutes",
  "action_type": "wait",
  "configuration": {
    "duration": 10,
    "unit": "minutes"
  }
}
```

### Get Action Types
```bash
GET /actions/action_types/
```

**Response:**
```json
[
  {"value": "send_message", "label": "Send Message"},
  {"value": "send_email", "label": "Send Email"},
  {"value": "add_tag", "label": "Add Tag"},
  {"value": "remove_tag", "label": "Remove Tag"},
  {"value": "update_user", "label": "Update User"},
  {"value": "add_note", "label": "Add Note"},
  {"value": "webhook", "label": "Webhook"},
  {"value": "wait", "label": "Wait"},
  {"value": "set_conversation_status", "label": "Set Conversation Status"},
  {"value": "custom_code", "label": "Custom Code"}
]
```

### Get Action Parameter Templates
```bash
GET /actions/parameter_templates/?action_type=send_message
```

### Test Action
```bash
POST /actions/{id}/test/
```

**Request Body:**
```json
{
  "context": {
    "user": {
      "id": "customer-123",
      "first_name": "John",
      "email": "john@example.com"
    },
    "event": {
      "type": "MESSAGE_RECEIVED"
    }
  }
}
```

## 🔍 Condition Management

### List Conditions
```bash
GET /conditions/
```

### Create Condition
```bash
POST /conditions/
```

**Simple Condition:**
```json
{
  "name": "User Has Premium Tag",
  "description": "Check if user is tagged as premium",
  "operator": "and",
  "conditions": [
    {
      "field": "user.tags",
      "operator": "contains",
      "value": "premium"
    }
  ]
}
```

**Complex Condition:**
```json
{
  "name": "Active Premium User",
  "operator": "and",
  "conditions": [
    {
      "field": "user.tags",
      "operator": "contains",
      "value": "premium"
    },
    {
      "operator": "or",
      "conditions": [
        {
          "field": "user.email",
          "operator": "ends_with",
          "value": "@company.com"
        },
        {
          "field": "user.profile.age",
          "operator": "greater",
          "value": 25
        }
      ]
    }
  ]
}
```

**Custom Code Condition:**
```json
{
  "name": "Custom Business Logic",
  "use_custom_code": true,
  "custom_code": "# Check complex business rules\nuser_score = context.get('user', {}).get('score', 0)\nmessage_length = len(context.get('event', {}).get('data', {}).get('content', ''))\n\n# Set result to True or False\nresult = user_score > 100 and message_length > 10"
}
```

## 📊 Event Processing

### Event Types
```bash
GET /event-types/
```

**Response:**
```json
{
  "results": [
    {
      "id": "event-type-uuid",
      "name": "Message Received",
      "code": "MESSAGE_RECEIVED",
      "category": "message",
      "description": "Triggered when a customer message is received",
      "available_fields": {
        "message_id": "string",
        "conversation_id": "string",
        "user_id": "string",
        "content": "string",
        "source": "string",
        "timestamp": "datetime"
      }
    }
  ]
}
```

### Common Event Types:
- `MESSAGE_RECEIVED` - Customer sends a message
- `MESSAGE_SENT` - Message sent to customer
- `USER_CREATED` - New customer registration
- `CONVERSATION_CREATED` - New conversation started
- `CONVERSATION_CLOSED` - Conversation ended
- `TAG_ADDED` - Tag added to customer
- `TAG_REMOVED` - Tag removed from customer
- `SCHEDULED` - Scheduled trigger execution

## 📈 Monitoring & Analytics

### Workflow Executions
```bash
GET /workflow-executions/
```

**Parameters:**
- `status`: Filter by execution status
- `workflow`: Filter by workflow ID
- `user`: Filter by user ID
- `created_at__gte`: Filter by date (greater than or equal)

**Example:**
```bash
curl -H "Authorization: Bearer YOUR_TOKEN" \
  "http://localhost:8000/api/v1/workflow/api/workflow-executions/?status=COMPLETED&workflow=workflow-uuid"
```

### Action Executions
```bash
GET /workflow-action-executions/
```

### Event Logs
```bash
GET /trigger-event-logs/
```

**Parameters:**
- `event_type`: Filter by event type
- `user_id`: Filter by user
- `conversation_id`: Filter by conversation

### Action Logs
```bash
GET /action-logs/
```

**Parameters:**
- `success`: Filter by success status
- `action`: Filter by action ID

### Cancel Execution
```bash
POST /workflow-executions/{id}/cancel/
```

## 📝 Common Response Formats

### Success Response
```json
{
  "id": "uuid",
  "field1": "value1",
  "field2": "value2",
  "created_at": "2024-01-01T10:00:00Z",
  "updated_at": "2024-01-01T10:00:00Z"
}
```

### Paginated Response
```json
{
  "count": 100,
  "next": "http://localhost:8000/api/v1/workflow/api/workflows/?page=3",
  "previous": "http://localhost:8000/api/v1/workflow/api/workflows/?page=1",
  "results": [...]
}
```

### Error Response
```json
{
  "error": "Workflow not found",
  "detail": "No Workflow matches the given query."
}
```

### Validation Error Response
```json
{
  "name": ["This field is required."],
  "trigger_type": ["Select a valid choice."]
}
```

## ⚠️ Error Handling

### HTTP Status Codes
- `200` - Success
- `201` - Created
- `400` - Bad Request (validation errors)
- `401` - Unauthorized (invalid/missing token)
- `403` - Forbidden (insufficient permissions)
- `404` - Not Found
- `500` - Internal Server Error

### Common Error Scenarios

**Authentication Error:**
```bash
# Missing or invalid token
HTTP 401 Unauthorized
{
  "detail": "Authentication credentials were not provided."
}
```

**Validation Error:**
```bash
# Invalid data in request
HTTP 400 Bad Request
{
  "name": ["This field is required."],
  "action_type": ["Select a valid choice."]
}
```

**Not Found Error:**
```bash
HTTP 404 Not Found
{
  "detail": "Not found."
}
```

## 🚀 Usage Examples

### Complete Workflow Setup Example

```bash
# 1. Create a trigger
TRIGGER_ID=$(curl -s -X POST \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Discount Request",
    "trigger_type": "MESSAGE_RECEIVED",
    "filters": {
      "operator": "or",
      "conditions": [
        {"field": "event.data.content", "operator": "icontains", "value": "discount"},
        {"field": "event.data.content", "operator": "icontains", "value": "coupon"}
      ]
    }
  }' \
  "http://localhost:8000/api/v1/workflow/api/triggers/" | jq -r '.id')

# 2. Create actions
ACTION1_ID=$(curl -s -X POST \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Send Discount Code",
    "action_type": "send_message",
    "configuration": {
      "message": "Here is your discount code: SAVE20"
    },
    "order": 1
  }' \
  "http://localhost:8000/api/v1/workflow/api/actions/" | jq -r '.id')

# 3. Create workflow
WORKFLOW_ID=$(curl -s -X POST \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Discount Code Campaign",
    "description": "Send discount codes when requested",
    "status": "DRAFT"
  }' \
  "http://localhost:8000/api/v1/workflow/api/workflows/" | jq -r '.id')

# 4. Associate trigger with workflow
curl -X POST \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d "{\"trigger_id\": \"$TRIGGER_ID\", \"priority\": 100}" \
  "http://localhost:8000/api/v1/workflow/api/workflows/$WORKFLOW_ID/add_trigger/"

# 5. Activate workflow
curl -X POST \
  -H "Authorization: Bearer $TOKEN" \
  "http://localhost:8000/api/v1/workflow/api/workflows/$WORKFLOW_ID/activate/"

# 6. Test with an event
curl -X POST \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "event_type": "MESSAGE_RECEIVED",
    "data": {
      "message_id": "test-123",
      "content": "Do you have any discount codes?",
      "timestamp": "2024-01-01T10:00:00Z"
    },
    "user_id": "customer-123",
    "conversation_id": "conv-456"
  }' \
  "http://localhost:8000/api/v1/workflow/api/triggers/process_event/"
```

### Monitoring Workflow Performance

```bash
# Get workflow statistics
curl -H "Authorization: Bearer $TOKEN" \
  "http://localhost:8000/api/v1/workflow/api/workflows/statistics/"

# Monitor recent executions
curl -H "Authorization: Bearer $TOKEN" \
  "http://localhost:8000/api/v1/workflow/api/workflow-executions/?page_size=10"

# Check failed executions
curl -H "Authorization: Bearer $TOKEN" \
  "http://localhost:8000/api/v1/workflow/api/workflow-executions/?status=FAILED"
```

---

This comprehensive API reference provides everything needed to integrate and use the Marketing Workflow system programmatically. All endpoints support standard REST operations and follow consistent patterns for easy integration.
