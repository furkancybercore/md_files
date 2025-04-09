# Postman Guide for Poker Backend API

This guide will help you test the Poker Backend API using Postman, a powerful API development and testing tool. I've reviewed your codebase and included only endpoints that are actually implemented.

## Table of Contents
1. [Before You Begin](#before-you-begin)
2. [Setting Up Postman](#setting-up-postman)
3. [Creating a Collection](#creating-a-collection)
4. [Player Operations](#player-operations)
5. [Event Operations](#event-operations)
6. [Advanced Postman Features](#advanced-postman-features)
7. [Automated Testing](#automated-testing)
8. [Environment Variables](#environment-variables)
9. [Request Chaining](#request-chaining)
10. [Troubleshooting](#troubleshooting)

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

Collections in Postman help you organize related requests for easier management and testing.

1. Click on the "Collections" tab in the sidebar
2. Click the "+" button to create a new collection
3. Name your collection "Poker Backend API"
4. Click on the "..." next to your collection name and select "Edit"
5. In the Variables tab, add these collection variables:
   - `baseUrl`: `http://127.0.0.1:8000` (initial value and current value)
6. Click "Save" to save your collection settings

## Player Operations

### Get All Players (GET)

1. Create a new request in your collection
2. Set the request method to GET
3. Set the URL to `{{baseUrl}}/players/`
4. Save the request as "Get All Players"
5. Click "Send" to execute the request

**Expected Response:**
```json
[
    {
        "user_id": 74647891,
        "email": "test@example.com",
        "nickname": "testplayer",
        "balance": 0,
        "win_points": 0,
        "created_at": "2025-03-13T23:21:16.755373Z"
    },
    {
        "user_id": 11771320,
        "email": null,
        "nickname": "",
        "balance": 0,
        "win_points": 0,
        "created_at": "2025-03-13T23:28:50.098895Z"
    },
    {
        "user_id": 27390727,
        "email": "john@example.com",
        "nickname": "johndoe",
        "balance": 0,
        "win_points": 0,
        "created_at": "2025-03-14T02:25:02.534470Z"
    }
]
```

### Register Player (POST)

1. Create a new request in your collection
2. Set the request method to POST
3. Set the URL to `{{baseUrl}}/players/register/`
4. Go to the "Body" tab
5. Select "raw" and "JSON" format
6. Enter the following JSON data:
```json
{
    "email": "new_player@example.com",
    "nickname": "newplayer",
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

### Login Player (POST)

1. Create a new request in your collection
2. Set the request method to POST
3. Set the URL to `{{baseUrl}}/players/login/`
4. Go to the "Body" tab
5. Select "raw" and "JSON" format
6. Enter the following JSON data:
```json
{
    "email": "test@example.com",
    "password": "test123"
}
```
7. Save the request as "Login Player"
8. Click "Send" to execute the request

**Expected Response:**
```json
{
    "message": "Login successful",
    "player": {
        "nickname": "testplayer",
        "email": "test@example.com",
        "balance": 0,
        "win_points": 0
    }
}
```

### Update Player Profile (PUT)

1. Create a new request in your collection
2. Set the request method to PUT
3. Set the URL to `{{baseUrl}}/players/profile/74647891/` (replace 74647891 with a valid user_id)
4. Go to the "Body" tab
5. Select "raw" and "JSON" format
6. Enter the following JSON data:
```json
{
    "auth_player_id": "74647891",
    "nickname": "updated_nickname",
    "email": "updated_email@example.com"
}
```
7. Save the request as "Update Player Profile"
8. Click "Send" to execute the request

**Expected Response:**
```json
{
    "message": "Profile updated successfully",
    "player": {
        "user_id": 74647891,
        "nickname": "updated_nickname",
        "email": "updated_email@example.com",
        "balance": 0,
        "win_points": 0
    }
}
```

### Update Player Password (PATCH)

1. Create a new request in your collection
2. Set the request method to PATCH
3. Set the URL to `{{baseUrl}}/players/profile/74647891/` (replace 74647891 with a valid user_id)
4. Go to the "Body" tab
5. Select "raw" and "JSON" format
6. Enter the following JSON data:
```json
{
    "auth_player_id": "74647891",
    "current_password": "current_password",
    "new_password": "new_secure_password"
}
```
7. Save the request as "Update Player Password"
8. Click "Send" to execute the request

**Expected Response:**
```json
{
    "message": "Profile updated successfully",
    "player": {
        "user_id": 74647891,
        "nickname": "player_nickname",
        "email": "player_email@example.com",
        "balance": 0,
        "win_points": 0
    }
}
```

### Partial Profile Update (PATCH)

1. Create a new request in your collection
2. Set the request method to PATCH
3. Set the URL to `{{baseUrl}}/players/profile/74647891/` (replace 74647891 with a valid user_id)
4. Go to the "Body" tab
5. Select "raw" and "JSON" format
6. Enter the following JSON data to update only the nickname:
```json
{
    "auth_player_id": "74647891",
    "nickname": "new_cool_nickname"
}
```
7. Save the request as "Partial Profile Update"
8. Click "Send" to execute the request

**Expected Response:**
```json
{
    "message": "Profile updated successfully",
    "player": {
        "user_id": 74647891,
        "nickname": "new_cool_nickname",
        "email": "player_email@example.com",
        "balance": 0,
        "win_points": 0
    }
}
```

### Testing Profile Update Validation Errors

1. Create a new request in your collection
2. Set the request method to PUT
3. Set the URL to `{{baseUrl}}/players/profile/74647891/` (replace 74647891 with a valid user_id)
4. Go to the "Body" tab
5. Select "raw" and "JSON" format
6. Enter the following JSON data with invalid email format:
```json
{
    "auth_player_id": "74647891",
    "email": "not-a-valid-email"
}
```
7. Save the request as "Test Profile Update Validation"
8. Click "Send" to execute the request

**Expected Response:**
```json
{
    "email": [
        "Enter a valid email address."
    ]
}
```

For testing password validation, use:
```json
{
    "auth_player_id": "74647891",
    "current_password": "wrong_password",
    "new_password": "short"
}
```

**Expected Response:**
```json
{
    "current_password": [
        "Current password is incorrect."
    ],
    "new_password": [
        "Password must be at least 8 characters long."
    ]
}
```

## Event Operations

### Create Event (POST)

1. Create a new request in your collection
2. Set the request method to POST
3. Set the URL to `{{baseUrl}}/events/create`
4. Go to the "Body" tab
5. Select "raw" and "JSON" format
6. Enter the following JSON data:
```json
{
    "event_name": "Weekend Poker Night",
    "creator_id": 74647891,
    "max_capacity": 8,
    "small_blind": 5,
    "big_blind": 10,
    "required_deposit": 100,
    "location": "123 Poker St, Card City",
    "game_date": "2025-04-15",
    "start_time": "19:00:00",
    "duration": 240,
    "regularity": "weekly"
}
```
7. Save the request as "Create Event"
8. Click "Send" to execute the request

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
        "game_date": "2025-04-15",
        "start_time": "19:00:00",
        "duration": 240,
        "created_at": "2025-03-15T11:00:00Z"
    }
}
```

### Get All Events (GET)

1. Create a new request in your collection
2. Set the request method to GET
3. Set the URL to `{{baseUrl}}/events/`
4. Save the request as "Get All Events"
5. Click "Send" to execute the request

**Expected Response:**
```json
[
    {
        "id": 1,
        "event_name": "Weekend Poker Night",
        "creator": "testplayer",
        "max_capacity": 8,
        "small_blind": 5,
        "big_blind": 10,
        "required_deposit": 100,
        "location": "123 Poker St, Card City",
        "game_date": "2025-04-15",
        "start_time": "19:00:00",
        "duration": 240,
        "created_at": "2025-03-15T11:00:00Z"
    }
]
```

### Get Event Details (GET)

1. Create a new request in your collection
2. Set the request method to GET
3. Set the URL to `{{baseUrl}}/events/1/` (replace 1 with your event_id)
4. Save the request as "Get Event Details"
5. Click "Send" to execute the request

**Expected Response:**
```json
{
    "id": 1,
    "event_name": "Weekend Poker Night",
    "max_capacity": 8,
    "player_count": 1,
    "players": ["testplayer"],
    "small_blind": 5,
    "big_blind": 10,
    "required_deposit": 100,
    "location": "123 Poker St, Card City",
    "game_date": "2025-04-15",
    "start_time": "19:00:00",
    "duration": 240,
    "created_at": "2025-03-15T11:00:00Z"
}
```

### Join Event (POST)

1. Create a new request in your collection
2. Set the request method to POST
3. Set the URL to `{{baseUrl}}/events/join`
4. Go to the "Body" tab
5. Select "raw" and "JSON" format
6. Enter the following JSON data:
```json
{
    "user_id": 27390727,
    "event_id": 1
}
```
7. Save the request as "Join Event"
8. Click "Send" to execute the request

**Expected Response:**
```json
{
    "message": "Player joined the Event successfully"
}
```

## Advanced Postman Features

### Using Pre-request Scripts

Pre-request scripts run before a request is sent, allowing you to set up the environment or manipulate data.

1. Create a new request called "Random User Registration"
2. Set the method to POST and URL to `{{baseUrl}}/players/register/`
3. Go to the "Scripts" tab and in the "Pre-request" section, add:
```javascript
// Generate random email and nickname
const randomNum = Math.floor(Math.random() * 10000);
const randomEmail = `user${randomNum}@example.com`;
const randomNickname = `player${randomNum}`;

// Set as environment variables
pm.collectionVariables.set("randomEmail", randomEmail);
pm.collectionVariables.set("randomNickname", randomNickname);

console.log(`Generated random user: ${randomNickname} (${randomEmail})`);
```
4. In the Body tab, add:
```json
{
    "email": "{{randomEmail}}",
    "nickname": "{{randomNickname}}",
    "password": "SecurePassword123"
}
```
5. Save and run the request

### Working with Response Data

Create a request to show how to parse and use response data:

1. Create a new request called "Parse Players Data"
2. Set the method to GET and URL to `{{baseUrl}}/players/`
3. Go to the "Scripts" tab and in the "Post-response" section, add:
```javascript
// Parse the response
var jsonData = pm.response.json();

// Count the players
pm.test("Count players", function() {
    pm.collectionVariables.set("playerCount", jsonData.length);
    console.log(`Found ${jsonData.length} players`);
});

// Find player by email
var testPlayer = jsonData.find(function(player) {
    return player.email === "test@example.com";
});

if (testPlayer) {
    console.log(`Found test player: ${testPlayer.nickname} (ID: ${testPlayer.user_id})`);
    pm.collectionVariables.set("testUserId", testPlayer.user_id);
}

// Filter active players (with non-null email)
var activePlayers = jsonData.filter(function(player) {
    return player.email !== null;
});
console.log(`Active players: ${activePlayers.length}`);
```
4. Save and run the request

## Automated Testing

Postman allows you to create automated test suites to validate your API endpoints.

### Create a Test Suite

1. In your Poker API collection, click on the "..." menu
2. Select "Add folder" and name it "Test Suite"
3. Inside this folder, create several test requests:

#### Test User Registration Validation

1. Create a request named "Test - Invalid Registration"
2. Set method to POST and URL to `{{baseUrl}}/players/register/`
3. In Body, add invalid data:
```json
{
    "email": "not-an-email",
    "nickname": "a",
    "password": "short"
}
```
4. Go to the "Scripts" tab and in the "Post-response" section, add:
```javascript
pm.test("Status should be 400 Bad Request", function() {
    pm.response.to.have.status(400);
});

pm.test("Should return validation errors", function() {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('email');
});
```

#### Test Player Retrieval

1. Create a request named "Test - Get Players"
2. Set method to GET and URL to `{{baseUrl}}/players/`
3. Go to the "Scripts" tab and in the "Post-response" section, add:
```javascript
pm.test("Status code is 200", function() {
    pm.response.to.have.status(200);
});

pm.test("Response is an array", function() {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.be.an('array');
});

pm.test("Players have expected fields", function() {
    var jsonData = pm.response.json();
    if (jsonData.length > 0) {
        pm.expect(jsonData[0]).to.have.property('user_id');
        pm.expect(jsonData[0]).to.have.property('nickname');
        pm.expect(jsonData[0]).to.have.property('email');
        pm.expect(jsonData[0]).to.have.property('balance');
    }
});
```

#### Test Profile Update

1. Create a request named "Test - Profile Update"
2. Set method to PUT and URL to `{{baseUrl}}/players/profile/{{testUserId}}/`
3. In Body, add valid update data:
```json
{
    "auth_player_id": "{{testUserId}}",
    "nickname": "test_updated_nickname",
    "email": "test_updated@example.com"
}
```
4. Go to the "Scripts" tab and in the "Post-response" section, add:
```javascript
pm.test("Status code is 200", function() {
    pm.response.to.have.status(200);
});

pm.test("Response has success message", function() {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('message');
    pm.expect(jsonData.message).to.eql('Profile updated successfully');
});

pm.test("Response contains updated player data", function() {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('player');
    pm.expect(jsonData.player).to.have.property('nickname');
    pm.expect(jsonData.player).to.have.property('email');
    pm.expect(jsonData.player.nickname).to.eql('test_updated_nickname');
    pm.expect(jsonData.player.email).to.eql('test_updated@example.com');
});
```

#### Test Profile Update Authentication

1. Create a request named "Test - Profile Update Authentication"
2. Set method to PUT and URL to `{{baseUrl}}/players/profile/{{testUserId}}/`
3. In Body, add invalid authentication:
```json
{
    "auth_player_id": "99999999",
    "nickname": "unauthorized_update"
}
```
4. Go to the "Scripts" tab and in the "Post-response" section, add:
```javascript
pm.test("Status should be 403 Forbidden", function() {
    pm.response.to.have.status(403);
});

pm.test("Should return authentication error", function() {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('error');
    pm.expect(jsonData.error).to.include('Authentication failed');
});
```

### Running Collection Tests

1. Click on the collection name
2. Click the "Run" button
3. Select the requests you want to run
4. Click "Run Poker Backend API"
5. View the test results

## Environment Variables

Environments in Postman allow you to run the same collection against different servers (e.g., development, staging, production).

### Create Environments

1. Click on "Environments" in the left sidebar
2. Click the "+" button to create a new environment
3. Name it "Development"
4. Add these variables:
   - `baseUrl`: `http://127.0.0.1:8000` 
   - `testUserId`: `74647891`
5. Save the environment

6. Create another environment named "Production" (hypothetical for practice)
7. Add these variables:
   - `baseUrl`: `https://poker-api.example.com` (hypothetical address)
   - `testUserId`: `74647891`
8. Save the environment

9. In your requests, update URLs to use both variables:
   - Change URLs to use `{{baseUrl}}/players/` format

10. Switch between environments using the dropdown in the upper right corner

## Request Chaining

You can chain requests together to create workflows. Example: Register → Login → Create Event → Join Event

### Create a Registration and Login Chain

1. Create a folder named "Workflows"
2. Inside, create a request named "01 - Register New User"
3. Set up registration with random data (using pre-request script as shown earlier)
4. Go to the "Scripts" tab and in the "Post-response" section, add:
```javascript
// Save email and password for next request
pm.collectionVariables.set("workflow_email", pm.collectionVariables.get("randomEmail"));
pm.collectionVariables.set("workflow_password", "SecurePassword123");
```

5. Create another request named "02 - Login User"
6. Set the body to:
```json
{
    "email": "{{workflow_email}}",
    "password": "{{workflow_password}}"
}
```
7. In the "Scripts" tab under "Post-response", add (hypothetical - your API doesn't return user_id in login):
```javascript
var jsonData = pm.response.json();
console.log("User logged in successfully");
```

8. Create a third request named "03 - Update Profile" for continuing the workflow
9. Set the method to PUT and URL to `{{baseUrl}}/players/profile/{{workflow_user_id}}/`
10. Add this pre-request script to get the user ID from the login response:
```javascript
// We're assuming the previous request (login) stored the user_id
// In a real implementation, you'd extract this from the login response
```
11. Set the body to:
```json
{
    "auth_player_id": "{{workflow_user_id}}",
    "nickname": "updated_workflow_nickname",
    "email": "{{workflow_email}}"
}
```
12. In the post-response script, add:
```javascript
// Verify the profile was updated correctly
var jsonData = pm.response.json();
console.log("Profile updated successfully with new nickname: " + jsonData.player.nickname);
```

### Use a Collection Runner to Execute the Workflow

1. Click the collection
2. Click "Run"
3. Select only the Workflows folder
4. Ensure "Keep variable values" is checked
5. Run the workflow

## Troubleshooting

### Common Issues

1. **"Error: connect ECONNREFUSED 127.0.0.1:8000"**:
   - Make sure the Django development server is running
   - Check that you're using the correct URL (http://127.0.0.1:8000/)
   - Ensure no firewall is blocking the connection

2. **"404 Not Found"**:
   - Verify the endpoint URL is correct
   - Check if the resource (e.g., user_id, event_id) exists
   - Make sure you've included any required URL parameters

3. **"400 Bad Request"**:
   - Check your request body for correct JSON format
   - Ensure all required fields are provided
   - Verify values match expected formats (e.g., email format, password requirements)
   - Look at the response body for specific validation errors

4. **"500 Internal Server Error"**:
   - Check the Django server console for error messages
   - This usually indicates a problem with the server-side code 

### Debugging with Postman

Use these Postman features to debug issues:

1. **Console**: View the Postman console (View > Show Postman Console) to see detailed request and response data, including headers

2. **Request History**: Postman keeps a history of all requests you've sent. View it by clicking on "History" in the sidebar

3. **Save Examples**: When a request works correctly, save it as an example for future reference by clicking "Save Response" and then "Save as Example"

4. **Request Timing**: Postman shows timing information for requests, which can help identify performance issues

5. **Network Status**: If you're having connection issues, check your network status in the bottom right corner of Postman

---

## Note About Hypothetical Endpoints

The following endpoints do not currently exist in your codebase but would be useful for future development. These are included here as practice examples for Postman usage, but they will not work with your current API:

### Get Player by ID (Hypothetical)

This endpoint is not currently implemented in your API.

1. Create a new request in your collection
2. Set the request method to GET
3. Set the URL to `{{baseUrl}}/players/74647891/` (replace with an actual user_id from your data)
4. Save the request as "Get Player by ID (Hypothetical)"

### Adjust Player Balance (Hypothetical)

This endpoint is not currently implemented in your API.

1. Create a new request in your collection
2. Set the request method to POST
3. Set the URL to `{{baseUrl}}/players/adjust_balance/`
4. Go to the "Body" tab
5. Select "raw" and "JSON" format
6. Enter the following JSON data:
```json
{
    "user_id": "74647891",
    "amount": 100,
    "description": "Initial deposit"
}
```
7. Save the request as "Adjust Balance (Hypothetical)"

### Leave Event (Hypothetical)

This endpoint is not currently implemented in your API.

1. Create a new request in your collection
2. Set the request method to POST
3. Set the URL to `{{baseUrl}}/events/leave`
4. Go to the "Body" tab
5. Select "raw" and "JSON" format
6. Enter the following JSON data:
```json
{
    "user_id": "27390727",
    "event_id": "1"
}
```
7. Save the request as "Leave Event (Hypothetical)"

### Update Player Profile (Hypothetical)

This endpoint is not currently implemented in your API.

1. Create a new request in your collection
2. Set the request method to PATCH
3. Set the URL to `{{baseUrl}}/players/profile/74647891/`
4. Go to the "Body" tab
5. Select "raw" and "JSON" format
6. Enter the following JSON data:
```json
{
    "nickname": "updated_nickname"
}
```
7. Save the request as "Update Player Profile (Hypothetical)" 