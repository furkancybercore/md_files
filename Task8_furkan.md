# Task 8: Update Player Profile Implementation Guide

## Feature Overview

This document provides a step-by-step guide for implementing Feature #8: Update Player Profile in our Poker Backend application. This feature allows players to update their profile information (nickname, email, password) with proper authentication and validation.

## Technical Requirements

1. Implement a PUT/PATCH method in the API for profile updates
2. Allow updates to nickname, email, and optionally reset password
3. Ensure authentication is required for all profile updates
4. Validate email format, nickname uniqueness, and password security

## Implementation Steps

### Step 1: Create a New URL Endpoint

First, we need to add a new endpoint to `api/urls.py` that will handle player profile updates:

```python
# In api/urls.py
urlpatterns = [
    # ... existing endpoints
    path('players/profile/<int:user_id>/', views.update_player_profile, name='update-profile'),
]
```

**Explanation:**
- This adds a new URL pattern that accepts a `user_id` parameter in the URL
- When a request is made to this URL, it will be routed to the `update_player_profile` view function
- The endpoint accepts both PUT (full update) and PATCH (partial update) requests

### Step 2: Create a Player Update Serializer

Add a new serializer in `app_core/poker_event/serializers.py` that will handle validation and data processing for profile updates:

```python
class PlayerUpdateSerializer(serializers.ModelSerializer):
    """Serializer for updating player profile information."""
    # Add write-only fields for password change
    current_password = serializers.CharField(write_only=True, required=False)
    new_password = serializers.CharField(write_only=True, required=False)
    
    class Meta:
        model = Player
        fields = ['nickname', 'email', 'current_password', 'new_password']
        extra_kwargs = {
            'nickname': {'required': False},  # Make fields optional for partial updates
            'email': {'required': False},
        }
    
    def validate_nickname(self, value):
        """Ensure nickname uniqueness excluding the current user."""
        user_id = self.context['user_id']
        if Player.objects.exclude(user_id=user_id).filter(nickname=value).exists():
            raise serializers.ValidationError("This nickname is already in use.")
        return value
    
    def validate_email(self, value):
        """Ensure email uniqueness excluding the current user and validate format."""
        user_id = self.context['user_id']
        if Player.objects.exclude(user_id=user_id).filter(email=value).exists():
            raise serializers.ValidationError("This email is already in use.")
        return value
    
    def validate(self, data):
        """Validate password change if requested."""
        current_password = data.get('current_password')
        new_password = data.get('new_password')
        
        # If either password field is provided, both must be provided
        if (current_password and not new_password) or (new_password and not current_password):
            raise serializers.ValidationError({
                "password": "Both current_password and new_password are required to change password."
            })
            
        # If both are provided, verify current password
        if current_password and new_password:
            player = self.instance
            if not player.check_password(current_password):
                raise serializers.ValidationError({
                    "current_password": "Current password is incorrect."
                })
                
            # Validate new password security
            if len(new_password) < 8:
                raise serializers.ValidationError({
                    "new_password": "Password must be at least 8 characters long."
                })
                
        return data
    
    def update(self, instance, validated_data):
        """Update player profile with validated data."""
        # Handle password update if requested
        if 'new_password' in validated_data:
            instance.set_password(validated_data.pop('new_password'))
        
        # Remove current_password from validated_data as it's not a model field
        validated_data.pop('current_password', None)
        
        # Update other fields
        for key, value in validated_data.items():
            setattr(instance, key, value)
            
        instance.save()
        return instance
```

**Explanation:**
- The serializer extends `ModelSerializer` to work with the Player model
- We add two custom write-only fields for password changes: `current_password` and `new_password`
- All fields are made optional for partial updates (PATCH requests)
- We add custom validation methods to:
  - Ensure unique nicknames (excluding the current user)
  - Ensure unique emails (excluding the current user)
  - Validate password changes by verifying the current password and enforcing security rules
- The `update` method handles:
  - Updating the password if a new one is provided
  - Removing password fields that aren't part of the model
  - Updating other fields as needed

### Step 3: Implement the Profile Update View Function

Add the view function in `app_core/poker_event/views.py` to handle the profile update requests:

```python
@api_view(["PUT", "PATCH"])
def update_player_profile(request, user_id):
    """
    Updates a player's profile information.
    Requires authentication and can only update the requesting player's profile.
    """
    try:
        # Get the player to update
        player = Player.objects.get(user_id=user_id)
        
        # Basic authentication check - in a real system, use proper authentication
        # This is a simplified check that should be replaced with a robust auth system
        request_player_id = request.data.get('auth_player_id')  # This would come from auth token in a real system
        
        if not request_player_id or int(request_player_id) != user_id:
            return Response(
                {"error": "Authentication failed. You can only update your own profile."},
                status=status.HTTP_403_FORBIDDEN
            )
        
        # Initialize serializer with player instance, data, and context
        serializer = PlayerUpdateSerializer(
            player, 
            data=request.data,
            partial=request.method == 'PATCH',  # Allow partial updates for PATCH requests
            context={'user_id': user_id}
        )
        
        # Validate and save updates
        if serializer.is_valid():
            player = serializer.save()
            
            # Return updated player details (excluding sensitive info)
            return Response({
                "message": "Profile updated successfully",
                "player": {
                    "user_id": player.user_id,
                    "nickname": player.nickname,
                    "email": player.email,
                    "balance": player.balance,
                    "win_points": player.win_points,
                }
            }, status=status.HTTP_200_OK)
            
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
        
    except Player.DoesNotExist:
        return Response({"error": "Player not found"}, status=status.HTTP_404_NOT_FOUND)
```

**Explanation:**
- The `@api_view` decorator specifies that this function can handle both PUT and PATCH requests
- We retrieve the player by user_id and handle the case when the player doesn't exist
- We add a basic authentication check (a placeholder that should be replaced with proper authentication)
- The serializer is initialized with the player instance, request data, and context
- We set `partial=True` for PATCH requests to allow partial updates
- After validation, we save the changes and return a success response with updated player data
- If validation fails, we return the validation errors with a 400 status code

### Step 4: Implementing Token-Based Authentication

For a more robust authentication system, we can implement token-based authentication:

1. First, add the authentication app to `INSTALLED_APPS` in `api/settings.py`:

```python
INSTALLED_APPS = [
    # ... existing apps
    'rest_framework.authtoken',  # Add token authentication
    # ... other apps
]
```

2. Run migrations to create the token table:
```bash
python manage.py migrate
```

3. Update the login view to return a token:

```python
from rest_framework.authtoken.models import Token

@api_view(["POST"])
def login(request):
    """Authenticates a player and returns player details with auth token."""
    serializer = serializers.PlayerLoginSerializer(data=request.data)
    if serializer.is_valid():
        player = serializer.validated_data
        # Create or get token
        token, created = Token.objects.get_or_create(user=player)
        return Response({
            "message": "Login successful",
            "token": token.key,  # Include auth token in response
            "player": {
                "user_id": player.user_id,
                "nickname": player.nickname,
                "email": player.email,
                "balance": player.balance,
                "win_points": player.win_points,
            }
        }, status=status.HTTP_200_OK)
    return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

**Explanation:**
- We import the Token model from `rest_framework.authtoken.models`
- After validating the login credentials, we create or retrieve a token for the player
- We include the token in the response along with the player details

4. Update the profile update view to use token authentication:

```python
from rest_framework.authentication import TokenAuthentication
from rest_framework.permissions import IsAuthenticated
from rest_framework.decorators import authentication_classes, permission_classes

@api_view(["PUT", "PATCH"])
@authentication_classes([TokenAuthentication])
@permission_classes([IsAuthenticated])
def update_player_profile(request, user_id):
    """
    Updates a player's profile information.
    Requires authentication and can only update the requesting player's profile.
    """
    try:
        # Get the player to update
        player = Player.objects.get(user_id=user_id)
        
        # Check if the authenticated user is trying to update their own profile
        if request.user.user_id != user_id:
            return Response(
                {"error": "You can only update your own profile."},
                status=status.HTTP_403_FORBIDDEN
            )
        
        # Initialize serializer with player instance and data
        serializer = PlayerUpdateSerializer(
            player, 
            data=request.data,
            partial=request.method == 'PATCH',
            context={'user_id': user_id}
        )
        
        # Validate and save updates
        if serializer.is_valid():
            player = serializer.save()
            
            # Return updated player details
            return Response({
                "message": "Profile updated successfully",
                "player": {
                    "user_id": player.user_id,
                    "nickname": player.nickname,
                    "email": player.email,
                    "balance": player.balance,
                    "win_points": player.win_points,
                }
            }, status=status.HTTP_200_OK)
            
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
        
    except Player.DoesNotExist:
        return Response({"error": "Player not found"}, status=status.HTTP_404_NOT_FOUND)
```

**Explanation:**
- We import the necessary authentication and permission classes
- We add decorators to specify that:
  - This view requires token authentication
  - The user must be authenticated to access it
- We check if the authenticated user is trying to update their own profile
- The rest of the function remains the same

### Step 5: Adding Security Enhancements

1. Add password complexity validation:

```python
def validate_password_complexity(password):
    """Validate password complexity."""
    if len(password) < 8:
        return False, "Password must be at least 8 characters long."
    
    if not any(char.isdigit() for char in password):
        return False, "Password must contain at least one number."
    
    if not any(char.isupper() for char in password):
        return False, "Password must contain at least one uppercase letter."
    
    if not any(char.islower() for char in password):
        return False, "Password must contain at least one lowercase letter."
    
    return True, ""

# Then update the validate method in PlayerUpdateSerializer:
def validate(self, data):
    # ... existing validation
    
    if current_password and new_password:
        # ... existing validation
        
        # Check password complexity
        is_valid, error_message = validate_password_complexity(new_password)
        if not is_valid:
            raise serializers.ValidationError({
                "new_password": error_message
            })
            
    return data
```

2. Add rate limiting to prevent brute force attacks:

```python
# In settings.py
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES': [
        'rest_framework.throttling.UserRateThrottle',
        'rest_framework.throttling.AnonRateThrottle',
    ],
    'DEFAULT_THROTTLE_RATES': {
        'user': '10/minute',
        'anon': '5/minute',
    },
}
```

3. Add request logging for security auditing:

```python
import logging
logger = logging.getLogger(__name__)

@api_view(["PUT", "PATCH"])
@authentication_classes([TokenAuthentication])
@permission_classes([IsAuthenticated])
def update_player_profile(request, user_id):
    # Log the request
    logger.info(f"Profile update requested by user {request.user.user_id} for profile {user_id}")
    
    # ... existing code
    
    if serializer.is_valid():
        player = serializer.save()
        logger.info(f"Profile successfully updated for user {user_id}")
        # ... rest of the code
```

## Testing the Implementation

### Manual Testing Steps with Postman

1. Register a test player:
   - Method: POST
   - URL: `http://localhost:8000/players/register/`
   - Body:
     ```json
     {
       "email": "test@example.com",
       "nickname": "testplayer",
       "password": "TestPass123"
     }
     ```

2. Login to get the authentication token:
   - Method: POST  
   - URL: `http://localhost:8000/players/login/`
   - Body:
     ```json
     {
       "email": "test@example.com",
       "password": "TestPass123"
     }
     ```
   - From the response, save the token value

3. Test updating the nickname:
   - Method: PATCH
   - URL: `http://localhost:8000/players/profile/YOUR_USER_ID/` (replace with actual user_id)
   - Headers:
     - Authorization: Token YOUR_TOKEN_HERE (replace with token from step 2)
   - Body:
     ```json
     {
       "nickname": "newNickname"
     }
     ```

4. Test updating the email:
   - Method: PATCH
   - URL: `http://localhost:8000/players/profile/YOUR_USER_ID/`
   - Headers:
     - Authorization: Token YOUR_TOKEN_HERE
   - Body:
     ```json
     {
       "email": "newemail@example.com"
     }
     ```

5. Test updating the password:
   - Method: PATCH
   - URL: `http://localhost:8000/players/profile/YOUR_USER_ID/`
   - Headers:
     - Authorization: Token YOUR_TOKEN_HERE
   - Body:
     ```json
     {
       "current_password": "TestPass123",
       "new_password": "NewSecurePass456"
     }
     ```

6. Test updating multiple fields:
   - Method: PUT
   - URL: `http://localhost:8000/players/profile/YOUR_USER_ID/`
   - Headers:
     - Authorization: Token YOUR_TOKEN_HERE
   - Body:
     ```json
     {
       "nickname": "completeUpdate",
       "email": "complete@example.com",
       "current_password": "NewSecurePass456",
       "new_password": "VerySecure789!"
     }
     ```

7. Test validation errors:
   - Try submitting a duplicate nickname
   - Try submitting an invalid email format
   - Try changing password with incorrect current password
   - Try submitting a new password that doesn't meet complexity requirements

## Security Considerations

1. Always use HTTPS in production to encrypt data in transit
2. Implement proper token expiration and refresh mechanisms
3. Store sensitive data like passwords using secure hashing methods
4. Implement rate limiting to prevent brute force attacks
5. Add logging for security-relevant events
6. Consider adding additional security layers like:
   - CAPTCHA for password changes
   - Email verification for email changes
   - Two-factor authentication for sensitive operations

## Conclusion

This implementation provides a secure and robust way for players to update their profile information. The code includes proper validation, authentication, and error handling to ensure data integrity and security.

Remember to thoroughly test all edge cases and security scenarios before deploying to production.
