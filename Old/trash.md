# Poker Backend

## Player Profile Update Feature Testing Guide

### Overview
This guide will help you test the newly implemented player profile update feature using Postman.

### Prerequisites
1. Make sure the poker-backend server is running
2. Have Postman installed on your computer
3. Create a test player account for testing this feature

### Testing Steps

#### 1. Create a test player account (if you don't have one already)

**POST** `http://localhost:8000/players/register/`

Request Body:
```json
{
  "email": "testplayer@example.com",
  "nickname": "TestPlayer",
  "password": "securepassword123"
}
```

#### 2. Test Player Profile Update

**PUT** `http://localhost:8000/players/update/`

Request Body (Update Email):
```json
{
  "email": "testplayer@example.com",
  "current_password": "securepassword123",
  "nickname": "UpdatedNickname"
}
```

Expected Response:
```json
{
  "message": "Profile updated successfully",
  "player": {
    "nickname": "UpdatedNickname",
    "email": "testplayer@example.com"
  }
}
```

Request Body (Update Email):
```json
{
  "email": "testplayer@example.com",
  "current_password": "securepassword123",
  "email": "newemail@example.com"
}
```

Expected Response:
```json
{
  "message": "Profile updated successfully",
  "player": {
    "nickname": "UpdatedNickname",
    "email": "newemail@example.com"
  }
}
```

Request Body (Update Password):
```json
{
  "email": "newemail@example.com",
  "current_password": "securepassword123",
  "password": "newsecurepassword456"
}
```

Expected Response:
```json
{
  "message": "Profile updated successfully",
  "player": {
    "nickname": "UpdatedNickname",
    "email": "newemail@example.com"
  }
}
```

#### 3. Verify the changes by logging in with the updated credentials

**POST** `http://localhost:8000/players/login/`

Request Body:
```json
{
  "email": "newemail@example.com",
  "password": "newsecurepassword456"
}
```

### Error Testing Scenarios

1. **Missing Email (Authentication Failure)**
   
   **PUT** `http://localhost:8000/players/update/`
   
   Request Body:
   ```json
   {
     "current_password": "securepassword123",
     "nickname": "UpdatedNickname"
   }
   ```
   
   Expected Response (400 Bad Request):
   ```json
   {
     "error": "Email is required for authentication"
   }
   ```

2. **Missing Current Password (Authentication Failure)**
   
   **PUT** `http://localhost:8000/players/update/`
   
   Request Body:
   ```json
   {
     "email": "newemail@example.com",
     "nickname": "UpdatedNickname"
   }
   ```
   
   Expected Response (400 Bad Request):
   ```json
   {
     "error": "Current password is required for authentication"
   }
   ```

3. **Incorrect Current Password (Authentication Failure)**
   
   **PUT** `http://localhost:8000/players/update/`
   
   Request Body:
   ```json
   {
     "email": "newemail@example.com",
     "current_password": "wrongpassword",
     "nickname": "UpdatedNickname"
   }
   ```
   
   Expected Response (401 Unauthorized):
   ```json
   {
     "error": "Current password is incorrect"
   }
   ```

4. **Player Not Found**
   
   **PUT** `http://localhost:8000/players/update/`
   
   Request Body:
   ```json
   {
     "email": "nonexistent@example.com",
     "current_password": "somepassword",
     "nickname": "UpdatedNickname"
   }
   ```
   
   Expected Response (404 Not Found):
   ```json
   {
     "error": "Player not found"
   }
   ```

5. **Validation Error - Duplicate Nickname**
   
   First, create another player with the nickname you want to test:
   
   **POST** `http://localhost:8000/players/register/`
   
   Request Body:
   ```json
   {
     "email": "anotherplayer@example.com",
     "nickname": "ExistingNickname",
     "password": "securepassword123"
   }
   ```
   
   Then try to update the first player's nickname to the same value:
   
   **PUT** `http://localhost:8000/players/update/`
   
   Request Body:
   ```json
   {
     "email": "newemail@example.com",
     "current_password": "newsecurepassword456",
     "nickname": "ExistingNickname"
   }
   ```
   
   Expected Response (400 Bad Request):
   ```json
   {
     "nickname": ["This nickname is already taken."]
   }
   ```

### Notes

- The nickname and email fields are validated for uniqueness across all players
- Password changes are optional - if you don't include a password field, the existing password is retained
- After updating email or password, make sure to use the new credentials for future API calls
