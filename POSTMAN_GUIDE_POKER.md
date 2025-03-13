# Postman Guide for Poker Backend API

This guide will help you test the Poker Backend API using Postman.

## Table of Contents
1. [Before You Begin](#before-you-begin)
2. [Setting Up Postman](#setting-up-postman)
3. [Creating a Collection](#creating-a-collection)
4. [Authentication](#authentication)
5. [Player Operations](#player-operations)
6. [Event Operations](#event-operations)
7. [Updating Player Profiles](#updating-player-profiles)
8. [Troubleshooting](#troubleshooting)

## Before You Begin

Before you can test the API, you need to ensure the Django development server is running:

1. Activate your virtual environment:
   ```bash
   # On Windows
   venv\Scripts\activate
   
   # On macOS/Linux
   source venv/bin/activate
   ```

2. Start the Django development server:
   ```bash
   python manage.py runserver
   ```

3. Verify the server is running by opening http://127.0.0.1:8000/players/ in your browser. You should see a response from the API.

**Important**: If you're getting "Error: connect ECONNREFUSED 127.0.0.1:8000" in Postman, it means the server is not running. Make sure to complete the steps above before testing with Postman.

## Setting Up Postman

1. Download and install Postman from [https://www.postman.com/downloads/](https://www.postman.com/downloads/)
2. Launch Postman and create an account or sign in

## Creating a Collection

1. Click on the "Collections" tab in the sidebar
2. Click the "+" button to create a new collection
3. Name your collection "Poker Backend API"
4. Save the collection

## Authentication

For most API endpoints, you'll need to authenticate. The Poker Backend API uses token-based authentication.

### Registration (POST)

1. Create a new request in your collection
2. Set the request method to POST
3. Set the URL to `http://127.0.0.1:8000/players/register/`
4. Go to the "Body" tab
5. Select "raw" and "JSON" format
6. Enter the following JSON data:
```json
{
    "email": "john@example.com",
    "nickname": "johndoe",
    "password": "SecurePassword123"
}
```
7. Save the request as "Register Player"
8. Click "Send" to execute the request

**Expected Response:**
```json
{
    "message": "Player registered successfully"
}
```

### Login (POST)

1. Create a new request in your collection
2. Set the request method to POST
3. Set the URL to `http://127.0.0.1:8000/players/login/`
4. Go to the "Body" tab
5. Select "raw" and "JSON" format
6. Enter the following JSON data:
```json
{
    "email": "john@example.com",
    "password": "SecurePassword123"
}
```
7. Save the request as "Login"
8. Click "Send" to execute the request

**Expected Response:**
```json
{
    "message": "Login successful",
    "token": "YOUR_AUTH_TOKEN",
    "player": {
        "nickname": "johndoe",
        "email": "john@example.com",
        "balance": 0,
        "win_points": 0
    }
}
```

**Important**: Save the token value from this response. You'll need it for authenticated requests.

### Using Token Authentication

For any request that requires authentication, add the token to the request headers:

1. Go to the "Headers" tab
2. Add a new header:
   - Key: `Authorization`
   - Value: `Token YOUR_AUTH_TOKEN` (replace with the actual token)

## Player Operations

### Get All Players (GET)

1. Create a new request in your collection
2. Set the request method to GET
3. Set the URL to `http://127.0.0.1:8000/players/`
4. Save the request as "Get All Players"
5. Click "Send" to execute the request

**Expected Response:**
```json
[
    {
        "user_id": 12345678,
        "email": "john@example.com",
        "nickname": "johndoe",
        "balance": 0,
        "win_points": 0,
        "created_at": "2023-03-15T09:00:00Z"
    },
    {
        "user_id": 87654321,
        "email": "jane@example.com",
        "nickname": "janedoe",
        "balance": 0,
        "win_points": 0,
        "created_at": "2023-03-15T10:00:00Z"
    }
]
```

## Event Operations

### Create Event (POST)

1. Create a new request in your collection
2. Set the request method to POST
3. Set the URL to `http://127.0.0.1:8000/events/create`
4. Add the Authorization header with your token
5. Go to the "Body" tab
6. Select "raw" and "JSON" format
7. Enter the following JSON data:
```json
{
    "event_name": "Weekend Poker Night",
    "creator_id": 12345678,
    "max_capacity": 8,
    "small_blind": 5,
    "big_blind": 10,
    "required_deposit": 100,
    "location": "123 Poker St, Card City",
    "game_date": "2023-04-15",
    "start_time": "19:00:00",
    "duration": 240,
    "regularity": "weekly"
}
```
8. Save the request as "Create Event"
9. Click "Send" to execute the request

**Expected Response:**
```json
{
    "message": "Event created successfully",
    "event": {
        "event_name": "Weekend Poker Night",
        "max_capacity": 8,
        "small_blind": 5,
        "big_blind": 10,
        "required_deposit": 100,
        "location": "123 Poker St, Card City",
        "game_date": "2023-04-15",
        "start_time": "19:00:00",
        "duration": 240,
        "created_at": "2023-03-15T11:00:00Z"
    }
}
```

### Get All Events (GET)

1. Create a new request in your collection
2. Set the request method to GET
3. Set the URL to `http://127.0.0.1:8000/events/`
4. Save the request as "Get All Events"
5. Click "Send" to execute the request

**Expected Response:**
```json
[
    {
        "id": 1,
        "event_name": "Weekend Poker Night",
        "creator": "johndoe",
        "max_capacity": 8,
        "small_blind": 5,
        "big_blind": 10,
        "required_deposit": 100,
        "location": "123 Poker St, Card City",
        "game_date": "2023-04-15",
        "start_time": "19:00:00",
        "duration": 240,
        "created_at": "2023-03-15T11:00:00Z"
    }
]
```

### Get Event Details (GET)

1. Create a new request in your collection
2. Set the request method to GET
3. Set the URL to `http://127.0.0.1:8000/events/1/` (replace 1 with the ID of an event)
4. Save the request as "Get Event Details"
5. Click "Send" to execute the request

**Expected Response:**
```json
{
    "id": 1,
    "event_name": "Weekend Poker Night",
    "max_capacity": 8,
    "player_count": 1,
    "players": ["johndoe"],
    "small_blind": 5,
    "big_blind": 10,
    "required_deposit": 100,
    "location": "123 Poker St, Card City",
    "game_date": "2023-04-15",
    "start_time": "19:00:00",
    "duration": 240,
    "created_at": "2023-03-15T11:00:00Z"
}
```

### Join Event (POST)

1. Create a new request in your collection
2. Set the request method to POST
3. Set the URL to `http://127.0.0.1:8000/events/join`
4. Add the Authorization header with your token
5. Go to the "Body" tab
6. Select "raw" and "JSON" format
7. Enter the following JSON data:
```json
{
    "user_id": 87654321,
    "event_id": 1
}
```
8. Save the request as "Join Event"
9. Click "Send" to execute the request

**Expected Response:**
```json
{
    "message": "Player joined the Event successfully"
}
```

## Updating Player Profiles

This section focuses on testing the new player profile update feature.

### Update Nickname (PATCH)

1. Create a new request in your collection
2. Set the request method to PATCH
3. Set the URL to `http://127.0.0.1:8000/players/profile/12345678/` (replace with your user_id)
4. Add the Authorization header with your token
5. Go to the "Body" tab
6. Select "raw" and "JSON" format
7. Enter the following JSON data:
```json
{
    "nickname": "john_poker_master"
}
```
8. Save the request as "Update Nickname"
9. Click "Send" to execute the request

**Expected Response:**
```json
{
    "message": "Profile updated successfully",
    "player": {
        "user_id": 12345678,
        "nickname": "john_poker_master",
        "email": "john@example.com",
        "balance": 0,
        "win_points": 0
    }
}
```

### Update Email (PATCH)

1. Create a new request in your collection
2. Set the request method to PATCH
3. Set the URL to `http://127.0.0.1:8000/players/profile/12345678/` (replace with your user_id)
4. Add the Authorization header with your token
5. Go to the "Body" tab
6. Select "raw" and "JSON" format
7. Enter the following JSON data:
```json
{
    "email": "john.updated@example.com"
}
```
8. Save the request as "Update Email"
9. Click "Send" to execute the request

**Expected Response:**
```json
{
    "message": "Profile updated successfully",
    "player": {
        "user_id": 12345678,
        "nickname": "john_poker_master",
        "email": "john.updated@example.com",
        "balance": 0,
        "win_points": 0
    }
}
```

### Update Password (PATCH)

1. Create a new request in your collection
2. Set the request method to PATCH
3. Set the URL to `http://127.0.0.1:8000/players/profile/12345678/` (replace with your user_id)
4. Add the Authorization header with your token
5. Go to the "Body" tab
6. Select "raw" and "JSON" format
7. Enter the following JSON data:
```json
{
    "current_password": "SecurePassword123",
    "new_password": "EvenMoreSecure456!"
}
```
8. Save the request as "Update Password"
9. Click "Send" to execute the request

**Expected Response:**
```json
{
    "message": "Profile updated successfully",
    "player": {
        "user_id": 12345678,
        "nickname": "john_poker_master",
        "email": "john.updated@example.com",
        "balance": 0,
        "win_points": 0
    }
}
```

### Update Multiple Fields (PUT)

1. Create a new request in your collection
2. Set the request method to PUT
3. Set the URL to `http://127.0.0.1:8000/players/profile/12345678/` (replace with your user_id)
4. Add the Authorization header with your token
5. Go to the "Body" tab
6. Select "raw" and "JSON" format
7. Enter the following JSON data:
```json
{
    "nickname": "john_complete_update",
    "email": "john.complete@example.com",
    "current_password": "EvenMoreSecure456!",
    "new_password": "FinalSecurePassword789@"
}
```
8. Save the request as "Update Full Profile"
9. Click "Send" to execute the request

**Expected Response:**
```json
{
    "message": "Profile updated successfully",
    "player": {
        "user_id": 12345678,
        "nickname": "john_complete_update",
        "email": "john.complete@example.com",
        "balance": 0,
        "win_points": 0
    }
}
```

### Testing Validation Errors

1. Try updating with an already used nickname:
```json
{
    "nickname": "janedoe"
}
```

2. Try updating with an invalid email format:
```json
{
    "email": "not-an-email"
}
```

3. Try changing password with incorrect current password:
```json
{
    "current_password": "WrongPassword",
    "new_password": "NewPassword123"
}
```

4. Try accessing another user's profile:
Change the URL to another user's ID and attempt an update.

## Troubleshooting

### Common Issues

1. **"Error: connect ECONNREFUSED 127.0.0.1:8000"**:
   - Make sure the Django development server is running
   - Check that you're using the correct URL (http://127.0.0.1:8000/)
   - Ensure no firewall is blocking the connection

2. **"404 Not Found"**:
   - Verify the endpoint URL is correct
   - Check if the resource (e.g., user_id, event_id) exists

3. **"401 Unauthorized"**:
   - Ensure you're including the correct token in the Authorization header
   - Verify the token format: `Token YOUR_AUTH_TOKEN`
   - Your token might be expired; try logging in again

4. **"403 Forbidden"**:
   - You might be trying to access a resource you don't have permission for
   - You can only update your own profile, not others'

5. **"400 Bad Request"**:
   - Check your request body for correct JSON format
   - Ensure all required fields are provided
   - Verify values match expected formats (e.g., email format, password requirements)

6. **"500 Internal Server Error"**:
   - Check the Django server console for error messages
   - This usually indicates a problem with the server-side code 