# Postman Guide for Poker Backend API

This guide will help you test the Poker Backend API using Postman, a powerful API development and testing tool.

## Table of Contents
1. [Before You Begin](#before-you-begin)
2. [Setting Up Postman](#setting-up-postman)
3. [Creating a Collection](#creating-a-collection)
4. [Authentication](#authentication)
5. [Player Operations](#player-operations)
6. [Event Operations](#event-operations)
7. [Updating Player Profiles](#updating-player-profiles)
8. [Advanced Postman Features](#advanced-postman-features)
9. [Automated Testing](#automated-testing)
10. [Environment Variables](#environment-variables)
11. [Request Chaining](#request-chaining)
12. [Troubleshooting](#troubleshooting)

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
   - `authToken`: Leave empty for now
6. Click "Save" to save your collection settings

## Authentication

For most API endpoints, you'll need to authenticate. The Poker Backend API uses token-based authentication.

### Registration (POST)

1. Create a new request in your collection
2. Set the request method to POST
3. Set the URL to `{{baseUrl}}/players/register/`
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
3. Set the URL to `{{baseUrl}}/players/login/`
4. Go to the "Body" tab
5. Select "raw" and "JSON" format
6. Enter the following JSON data:
```json
{
    "email": "john@example.com",
    "password": "SecurePassword123"
}
```
7. Go to the "Scripts" tab (in the middle row of tabs)
8. Under "Post-response" section, add this script to automatically save the token:
```javascript
// Test if the request was successful
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Parse the response body
var jsonData = pm.response.json();

// Test if we got a token
pm.test("Response includes auth token", function() {
    pm.expect(jsonData.token).to.exist;
});

// Save the token to collection variables
if (jsonData.token) {
    pm.collectionVariables.set("authToken", jsonData.token);
    console.log("Token saved: " + jsonData.token);
}

// Save user_id for later use
if (jsonData.player && jsonData.player.user_id) {
    pm.collectionVariables.set("user_id", jsonData.player.user_id);
    console.log("User ID saved: " + jsonData.player.user_id);
}
```
9. Save the request as "Login"
10. Click "Send" to execute the request

**Expected Response:**
```json
{
    "message": "Login successful",
    "token": "YOUR_AUTH_TOKEN",
    "player": {
        "user_id": 12345678,
        "nickname": "johndoe",
        "email": "john@example.com",
        "balance": 0,
        "win_points": 0
    }
}
```

**Note**: The token will be automatically saved to the `authToken` collection variable if you added the post-response script.

### Using Token Authentication

For any request that requires authentication, add the token to the request headers:

1. Go to the "Headers" tab
2. Add a new header:
   - Key: `Authorization`
   - Value: `Token {{authToken}}`

## Player Operations

### Get All Players (GET)

1. Create a new request in your collection
2. Set the request method to GET
3. Set the URL to `{{baseUrl}}/players/`
4. Add the Authorization header with your token
5. Save the request as "Get All Players"
6. Click "Send" to execute the request

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

### Get Player by ID (GET)

1. Create a new request in your collection
2. Set the request method to GET
3. Set the URL to `{{baseUrl}}/players/{{user_id}}/`
4. Add the Authorization header with your token
5. Save the request as "Get Player by ID"
6. Click "Send" to execute the request

**Expected Response:**
```json
{
    "user_id": 12345678,
    "email": "john@example.com",
    "nickname": "johndoe",
    "balance": 0,
    "win_points": 0,
    "created_at": "2023-03-15T09:00:00Z"
}
```

### Adjust Player Balance (POST)

1. Create a new request in your collection
2. Set the request method to POST
3. Set the URL to `{{baseUrl}}/players/adjust_balance/`
4. Add the Authorization header with your token
5. Go to the "Body" tab
6. Select "raw" and "JSON" format
7. Enter the following JSON data:
```json
{
    "user_id": "{{user_id}}",
    "amount": 100,
    "description": "Initial deposit"
}
```
8. Save the request as "Adjust Balance"
9. Click "Send" to execute the request

**Expected Response:**
```json
{
    "message": "Balance adjusted successfully",
    "player": {
        "user_id": 12345678,
        "nickname": "johndoe",
        "email": "john@example.com",
        "balance": 100,
        "win_points": 0
    }
}
```

## Event Operations

### Create Event (POST)

1. Create a new request in your collection
2. Set the request method to POST
3. Set the URL to `{{baseUrl}}/events/create`
4. Add the Authorization header with your token
5. Go to the "Body" tab
6. Select "raw" and "JSON" format
7. Enter the following JSON data:
```json
{
    "event_name": "Weekend Poker Night",
    "creator_id": "{{user_id}}",
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
8. Go to the "Scripts" tab and in the "Post-response" section, add this script to save the event ID:
```javascript
var jsonData = pm.response.json();
if (jsonData.event && jsonData.event.id) {
    pm.collectionVariables.set("event_id", jsonData.event.id);
    console.log("Event ID saved: " + jsonData.event.id);
}
```
9. Save the request as "Create Event"
10. Click "Send" to execute the request

**Expected Response:**
```json
{
    "message": "Event created successfully",
    "event": {
        "id": 1,
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
3. Set the URL to `{{baseUrl}}/events/`
4. Add the Authorization header with your token
5. Save the request as "Get All Events"
6. Click "Send" to execute the request

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
3. Set the URL to `{{baseUrl}}/events/{{event_id}}/`
4. Add the Authorization header with your token
5. Save the request as "Get Event Details"
6. Click "Send" to execute the request

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
3. Set the URL to `{{baseUrl}}/events/join`
4. Add the Authorization header with your token
5. Go to the "Body" tab
6. Select "raw" and "JSON" format
7. Enter the following JSON data:
```json
{
    "user_id": "{{user_id}}",
    "event_id": "{{event_id}}"
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

### Leave Event (POST)

1. Create a new request in your collection
2. Set the request method to POST
3. Set the URL to `{{baseUrl}}/events/leave`
4. Add the Authorization header with your token
5. Go to the "Body" tab
6. Select "raw" and "JSON" format
7. Enter the following JSON data:
```json
{
    "user_id": "{{user_id}}",
    "event_id": "{{event_id}}"
}
```
8. Save the request as "Leave Event"
9. Click "Send" to execute the request

**Expected Response:**
```json
{
    "message": "Player left the Event successfully"
}
```

### Update Event (PUT/PATCH)

1. Create a new request in your collection
2. Set the request method to PATCH
3. Set the URL to `{{baseUrl}}/events/{{event_id}}/`
4. Add the Authorization header with your token
5. Go to the "Body" tab
6. Select "raw" and "JSON" format
7. Enter the following JSON data:
```json
{
    "event_name": "Updated Poker Night",
    "small_blind": 10,
    "big_blind": 20
}
```
8. Save the request as "Update Event"
9. Click "Send" to execute the request

**Expected Response:**
```json
{
    "message": "Event updated successfully",
    "event": {
        "id": 1,
        "event_name": "Updated Poker Night",
        "max_capacity": 8,
        "small_blind": 10,
        "big_blind": 20,
        "required_deposit": 100,
        "location": "123 Poker St, Card City",
        "game_date": "2023-04-15",
        "start_time": "19:00:00",
        "duration": 240,
        "created_at": "2023-03-15T11:00:00Z"
    }
}
```

## Updating Player Profiles

This section focuses on testing the player profile update feature.

### Update Nickname (PATCH)

1. Create a new request in your collection
2. Set the request method to PATCH
3. Set the URL to `{{baseUrl}}/players/profile/{{user_id}}/`
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
3. Set the URL to `{{baseUrl}}/players/profile/{{user_id}}/`
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
3. Set the URL to `{{baseUrl}}/players/profile/{{user_id}}/`
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
3. Set the URL to `{{baseUrl}}/players/profile/{{user_id}}/`
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

1. Create a new request called "Parse Events Data"
2. Set the method to GET and URL to `{{baseUrl}}/events/`
3. Go to the "Scripts" tab and in the "Post-response" section, add:
```javascript
// Parse the response
var jsonData = pm.response.json();

// Count the events
pm.test("Count events", function() {
    pm.collectionVariables.set("eventCount", jsonData.length);
    console.log(`Found ${jsonData.length} events`);
});

// Calculate average required deposit
if (jsonData.length > 0) {
    var totalDeposit = 0;
    jsonData.forEach(function(event) {
        totalDeposit += event.required_deposit;
    });
    var avgDeposit = totalDeposit / jsonData.length;
    console.log(`Average required deposit: ${avgDeposit}`);
    
    pm.test("Average deposit should be reasonable", function() {
        pm.expect(avgDeposit).to.be.below(1000);
    });
}

// Find events with available spots
var availableEvents = jsonData.filter(function(event) {
    return event.player_count < event.max_capacity;
});
console.log(`Events with available spots: ${availableEvents.length}`);
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
    pm.expect(jsonData.errors).to.exist;
});
```

#### Test Authentication Required

1. Create a request named "Test - Auth Required"
2. Set method to GET and URL to `{{baseUrl}}/players/profile/{{user_id}}/`
3. Do NOT add an auth token header
4. Go to the "Scripts" tab and in the "Post-response" section, add:
```javascript
pm.test("Should require authentication", function() {
    pm.response.to.have.status(401);
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
   - `apiVersion`: `v1`
5. Save the environment

6. Create another environment named "Production"
7. Add these variables:
   - `baseUrl`: `https://poker-api.example.com`
   - `apiVersion`: `v1`
8. Save the environment

9. In your requests, update URLs to use both variables:
   - Change `{{baseUrl}}/players/` to `{{baseUrl}}/{{apiVersion}}/players/`
   - (Note: You'll need to adjust this based on your actual API structure)

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
7. In the "Scripts" tab under "Post-response", add token saving logic as shown earlier

8. Continue adding numbered requests to complete your workflow

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

3. **"401 Unauthorized"**:
   - Ensure you're including the correct token in the Authorization header
   - Verify the token format: `Token {{authToken}}`
   - Your token might be expired; try logging in again
   - Check if the token variable is correctly set using the "eye" icon next to the collection name

4. **"403 Forbidden"**:
   - You might be trying to access a resource you don't have permission for
   - You can only update your own profile, not others'

5. **"400 Bad Request"**:
   - Check your request body for correct JSON format
   - Ensure all required fields are provided
   - Verify values match expected formats (e.g., email format, password requirements)
   - Look at the response body for specific validation errors

6. **"500 Internal Server Error"**:
   - Check the Django server console for error messages
   - This usually indicates a problem with the server-side code

### Debugging with Postman

Use these Postman features to debug issues:

1. **Console**: View the Postman console (View > Show Postman Console) to see detailed request and response data, including headers

2. **Request History**: Postman keeps a history of all requests you've sent. View it by clicking on "History" in the sidebar

3. **Save Examples**: When a request works correctly, save it as an example for future reference by clicking "Save Response" and then "Save as Example"

4. **Request Timing**: Postman shows timing information for requests, which can help identify performance issues

5. **Network Status**: If you're having connection issues, check your network status in the bottom right corner of Postman

## Practice Exercise: Create Your Own Endpoints

Now that you've learned how to use Postman, try creating requests for these hypothetical endpoints:

1. **Search Events**: GET request to `{{baseUrl}}/events/search?location=New%20York&date=2023-05-01`

2. **Get Player Statistics**: GET request to `{{baseUrl}}/players/{{user_id}}/stats`

3. **Update Event Status**: PATCH request to `{{baseUrl}}/events/{{event_id}}/status` with body:
   ```json
   {
     "status": "cancelled",
     "reason": "Venue unavailable"
   }
   ```

4. **Create a Tournament**: POST request to `{{baseUrl}}/tournaments/create` with a body containing all required tournament details

These exercises will help you practice creating and structuring requests in Postman, even if the endpoints don't exist yet in your actual API. 