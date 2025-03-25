# 🎮 Backend API URL Schema:

This document provides a comprehensive overview of the Poker Backend API, including available endpoints, request/response formats, and usage examples.

💡The API enforces various validation rules to ensure data integrity. If validation fails, the API will return a `400 Bad Request` error with details about the failed validation.

## 📝 Notes

#### Endpoints Status
Endpoints are marked for their status
- ✳️ fully implemented and should just work
- ✴️ development in progress
- ⛔ not yet implemented but planned

#### Base URL
Base url for staging server:
```
https://staging-pokerebot-backend.onrender.com
```

Base url for production server:
```
https://pokerebot-backend-django.onrender.com
```

## 📡 API Endpoints

### ✴️ Verify and Authenticate a Telegram User

Validates a Telegram user's `initData`, confirms their identity, and updates their last login timestamp.

**Endpoint:**
```
POST {base-url}/players/verify/
```

**Input Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `initData` | string | Yes | URL-encoded authentication data from Telegram ([docs](https://core.telegram.org/bots/webapps#validating-data-received-via-the-mini-app)). Must be validated server-side. |

**Response Status Codes:**
- `200 OK`: User verified successfully
- `400 Bad Request`: Invalid initData
- `401 Unauthorized`: Authentication failed

**Response Body:**
| Field | Type | Description |
|-------|------|-------------|
| N/A | N/A | N/A |


**Example Usage:**
```
POST {base-url}/players/verify/
```
```json
// Request Body
{
  "initData": "query_id=AAHdF6IQAAAAAN0XohDmbSjL&user=%7B%22id%22%3A123456789%2C%22first_name%22%3A%22John%22%2C%22last_name%22%3A%22Doe%22%2C%22username%22%3A%22johndoe%22%2C%22language_code%22%3A%22en%22%2C%22allows_write_to_pm%22%3Atrue%7D&auth_date=1715644800&hash=a1b2c3d4e5f6g7h8i9j0..."
}

// Response (200 OK)
{

}
```

### ✳️ Get All Players

Retrieves a list of all registered players.

**Endpoint:**
```
GET {base-url}/players/
```

**Input Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| N/A | N/A | N/A | N/A |

**Response Status Codes:**
- `200 OK`: Request successful
- `500 Internal Server Error`: Server-side error

**Response Body:**
Array of player objects with the following fields:

| Field | Type | Description |
|-------|------|-------------|
| `telegram_id` | integer | Unique Telegram user ID |
| `first_name` | string | User's first name |
| `last_name` | string | User's last name (optional) |
| `balance` | integer | User's current balance |
| `win_points` | integer | User's accumulated win points |
| `photo_url` | string | URL to user's profile photo (optional) |
| `last_login` | string | Timestamp of last login (format: "YYYY-MM-DD HH:MM:SS") |

**Example Usage:**
```
GET {base-url}/players/
```
```json
// Response (200 OK)
[
  {
    "telegram_id": 123456789,
    "first_name": "John",
    "last_name": "Doe",
    "balance": 500,
    "win_points": 100,
    "photo_url": "https://t.me/i/userpic/320/username.jpg",
    "last_login": "2023-06-01 12:34:56"
  },
  {
    "telegram_id": 987654321,
    "first_name": "Jane",
    "last_name": "Smith",
    "balance": 750,
    "win_points": 200,
    "photo_url": "https://t.me/i/userpic/320/username2.jpg",
    "last_login": "2023-06-02 10:15:30"
  }
]
```

### ✳️ Create Poker Event

Creates a new poker event with the specified parameters.

**Endpoint:**
```
POST {base-url}/events/create
```

**Input Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `event_name` | string | Yes | Name of the poker event (max 255 chars) |
| `telegram_id` | integer | Yes | Telegram ID of the event creator (must exist in Player table) |
| `max_capacity` | integer | Yes | Maximum number of players allowed (positive value) |
| `small_blind` | integer | Yes | Small blind amount for the game (positive value) |
| `big_blind` | integer | Yes | Big blind amount for the game (positive value) |
| `required_deposit` | integer | Yes | Minimum deposit required to join (positive value) |
| `location` | string | Yes | Physical location of the event |
| `game_date` | string | Yes | Date when the event will take place (format: "YYYY-MM-DD") |
| `start_time` | string | Yes | Time when the event will start (format: "HH:MM") |
| `duration` | integer | Yes | Expected duration in minutes (positive value) |

**Response Status Codes:**
- `201 Created`: Event created successfully
- `400 Bad Request`: Invalid input parameters
- `404 Not Found`: Referenced player not found

**Response Body:**
| Field | Type | Description |
|-------|------|-------------|
| `message` | string | Success message |
| `event` | object | Event object containing all event details |

**Example Usage:**
```
POST {base-url}/events/create
```
```json
// Request Body
{
  "event_name": "Friday Night Poker",
  "telegram_id": 123456789,
  "max_capacity": 8,
  "small_blind": 5,
  "big_blind": 10,
  "required_deposit": 100,
  "location": "123 Poker Street, Las Vegas",
  "game_date": "2023-06-30",
  "start_time": "19:30",
  "duration": 240
}

// Response (201 Created)
{
  "message": "Event created successfully",
  "event": {
    "event_name": "Friday Night Poker",
    "telegram_id": 123456789,
    "max_capacity": 8,
    "small_blind": 5,
    "big_blind": 10,
    "required_deposit": 100,
    "location": "123 Poker Street, Las Vegas",
    "game_date": "2023-06-30",
    "start_time": "19:30",
    "duration": 240,
    "created_at": "2023-06-01T12:34:56Z"
  }
}
```

### ✳️ Get All Events

Retrieves a list of all available poker events.

**Endpoint:**
```
GET {base-url}/events/
```

**Input Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| N/A | N/A | N/A | N/A |

**Response Status Codes:**
- `200 OK`: Request successful
- `500 Internal Server Error`: Server-side error (currently returning this error)

**Response Body:**
Array of event objects with the following fields:

| Field | Type | Description |
|-------|------|-------------|
| `id` | integer | Unique event identifier |
| `event_name` | string | Name of the poker event |
| `creator` | string | Name of the event creator |
| `max_capacity` | integer | Maximum number of players allowed |
| `small_blind` | integer | Small blind amount for the game |
| `big_blind` | integer | Big blind amount for the game |
| `required_deposit` | integer | Minimum deposit required to join |
| `location` | string | Physical location of the event |
| `game_date` | string | Date when the event will take place (format: "YYYY-MM-DD") |
| `start_time` | string | Time when the event will start (format: "HH:MM:SS") |
| `duration` | integer | Expected duration in minutes |
| `created_at` | string | Timestamp when the event was created (format: "YYYY-MM-DDTHH:MM:SSZ") |

**Example Usage:**
```
GET {base-url}/events/
```
```json
// Response (200 OK) - Note: Currently returns 500 Internal Server Error
[
  {
    "id": 1,
    "event_name": "Friday Night Poker",
    "creator": "John Doe",
    "max_capacity": 8,
    "small_blind": 5,
    "big_blind": 10,
    "required_deposit": 100,
    "location": "123 Poker Street, Las Vegas",
    "game_date": "2023-06-30",
    "start_time": "19:30:00",
    "duration": 240,
    "created_at": "2023-06-01T12:34:56Z"
  },
  {
    "id": 2,
    "event_name": "Sunday Poker Tournament",
    "creator": "Jane Smith",
    "max_capacity": 12,
    "small_blind": 10,
    "big_blind": 20,
    "required_deposit": 200,
    "location": "456 Casino Ave, Reno",
    "game_date": "2023-07-02",
    "start_time": "14:00:00",
    "duration": 360,
    "created_at": "2023-06-02T10:15:30Z"
  }
]
```

### ✳️ Join Poker Event

Allows a player to join an existing poker event.

**Endpoint:**
```
POST {base-url}/events/join
```

**Input Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `telegram_id` | integer | Yes | Telegram ID of the joining player (must exist in Player table) |
| `event_id` | integer | Yes | ID of the event to join (must exist in Event table) |

**Response Status Codes:**
- `200 OK`: Player joined successfully
- `400 Bad Request`: Invalid input parameters
- `404 Not Found`: Player or event not found (currently returning this error)
- `403 Forbidden`: Event is full or player already joined

**Response Body:**
| Field | Type | Description |
|-------|------|-------------|
| `message` | string | Success message |

**Example Usage:**
```
POST {base-url}/events/join
```
```json
// Request Body
{
  "telegram_id": 123456789,
  "event_id": 1
}

// Response (200 OK) - Note: Currently returns 404 Not Found
{
  "message": "Player joined the Event successfully"
}
```

### ✳️ Get Specific Event Details 

Retrieves detailed information about a specific poker event.

**Endpoint:**
```
GET {base-url}/events/{event_id}/
```

**Input Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `event_id` | integer | Yes | ID of the event to retrieve (path parameter) |

**Response Status Codes:**
- `200 OK`: Request successful
- `404 Not Found`: Event not found

**Response Body:**
| Field | Type | Description |
|-------|------|-------------|
| `id` | integer | Unique event identifier |
| `event_name` | string | Name of the poker event |
| `max_capacity` | integer | Maximum number of players allowed |
| `player_count` | integer | Current number of players in the event |
| `players` | array | List of players in the event |
| `players[].user__first_name` | string | Player's first name |
| `players[].user__last_name` | string | Player's last name |
| `players[].user__telegram_id` | integer | Player's Telegram ID |
| `small_blind` | integer | Small blind amount for the game |
| `big_blind` | integer | Big blind amount for the game |
| `required_deposit` | integer | Minimum deposit required to join |
| `location` | string | Physical location of the event |
| `game_date` | string | Date when the event will take place (format: "YYYY-MM-DD") |
| `start_time` | string | Time when the event will start (format: "HH:MM:SS") |
| `duration` | integer | Expected duration in minutes |
| `created_at` | string | Timestamp when the event was created (format: "YYYY-MM-DDTHH:MM:SSZ") |

**Example Usage:**
```
GET {base-url}/events/1/
```
```json
// Request Body
{
  "id": 1,
}

// Response (200 OK)
{
  "id": 1,
  "event_name": "Friday Night Poker",
  "max_capacity": 8,
  "player_count": 3,
  "players": [
    {
      "user__first_name": "John",
      "user__last_name": "Doe", 
      "user__telegram_id": 123456789
    },
    {
      "user__first_name": "Jane",
      "user__last_name": "Smith",
      "user__telegram_id": 987654321
    },
    {
      "user__first_name": "Alex",
      "user__last_name": "Johnson",
      "user__telegram_id": 456789123
    }
  ],
  "small_blind": 5,
  "big_blind": 10,
  "required_deposit": 100,
  "location": "123 Poker Street, Las Vegas",
  "game_date": "2023-06-30",
  "start_time": "19:30:00",
  "duration": 240,
  "created_at": "2023-06-01T12:34:56Z"
}
```

