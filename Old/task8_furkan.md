# Task 8: Implement Player Profile Update Feature

In this task, you will enhance the Poker Backend API by implementing a feature to allow players to update their profile information, including nickname, email, and password.

## Objective

Create a new API endpoint that allows players to update their profile information, including:
- Nickname
- Email
- Password (with verification of current password)

## Requirements

1. Create a new API endpoint at `/players/profile/<user_id>/` that accepts PATCH and PUT requests
2. Implement validation for all fields
   - Nickname must be unique
   - Email must be unique and follow a valid format
   - Password changes must require the current password for verification
3. Allow partial updates (updating only some fields at a time)
4. Implement appropriate error handling and status codes
5. Make sure the endpoint follows REST principles

## Implementation Steps

### Step 1: Create a New Serializer

Add a new serializer in `app_core/poker_event/serializers.py`:

```python
# Player Profile Update Serializer
class PlayerProfileUpdateSerializer(serializers.ModelSerializer):
    current_password = serializers.CharField(write_only=True, required=False)
    new_password = serializers.CharField(write_only=True, required=False)
    
    class Meta:
        model = Player
        fields = ['nickname', 'email', 'current_password', 'new_password']
        extra_kwargs = {
            'email': {'required': False},
            'nickname': {'required': False}
        }
    
    def validate_email(self, value):
        """Validate email is unique (except for current user)."""
        if value:
            # Check if email is unique, excluding the current instance
            if Player.objects.filter(email=value).exclude(user_id=self.instance.user_id).exists():
                raise serializers.ValidationError("A player with this email already exists.")
        return value
    
    def validate_nickname(self, value):
        """Validate nickname is unique (except for current user)."""
        if value:
            # Check if nickname is unique, excluding the current instance
            if Player.objects.filter(nickname=value).exclude(user_id=self.instance.user_id).exists():
                raise serializers.ValidationError("A player with this nickname already exists.")
        return value
    
    def validate(self, data):
        """Validate password changes."""
        # Check if new password is provided but current password is missing
        if 'new_password' in data and 'current_password' not in data:
            raise serializers.ValidationError({"current_password": "Current password is required to set a new password"})
        
        # If current_password is provided, verify it
        if 'current_password' in data:
            if not self.instance.check_password(data['current_password']):
                raise serializers.ValidationError({"current_password": "Current password is incorrect"})
        
        return data
    
    def update(self, instance, validated_data):
        """Update player profile with validated data."""
        # Handle password update
        if 'new_password' in validated_data:
            instance.set_password(validated_data.pop('new_password'))
        
        # Remove current_password as we don't need to store it
        validated_data.pop('current_password', None)
        
        # Update remaining fields
        for attr, value in validated_data.items():
            setattr(instance, attr, value)
        
        instance.save()
        return instance
```

### Step 2: Create a New View

Add a new view function in `app_core/poker_event/views.py`:

```python
@api_view(["PATCH", "PUT"])
def update_player_profile(request, user_id):
    """Updates a player's profile information."""
    try:
        player = Player.objects.get(user_id=user_id)
    except Player.DoesNotExist:
        return Response({"error": "Player not found"}, status=status.HTTP_404_NOT_FOUND)
    
    serializer = serializers.PlayerProfileUpdateSerializer(player, data=request.data, partial=True)
    if serializer.is_valid():
        updated_player = serializer.save()
        return Response({
            "message": "Profile updated successfully",
            "player": {
                "user_id": updated_player.user_id,
                "nickname": updated_player.nickname,
                "email": updated_player.email,
                "balance": updated_player.balance,
                "win_points": updated_player.win_points
            }
        }, status=status.HTTP_200_OK)
    return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

### Step 3: Add URL Route

Update `api/urls.py` to include the new endpoint:

```python
# Add this import at the top if needed
from app_core.poker_event import views

# Add this to the urlpatterns list
path('players/profile/<int:user_id>/', views.update_player_profile, name='update-player-profile'),
```

### Step 4: Test the Implementation

Use Postman to test your implementation:

1. Create a PATCH request to `/players/profile/<user_id>/` with JSON body:
   ```json
   {
     "nickname": "updated_nickname"
   }
   ```

2. Create a PATCH request to update email:
   ```json
   {
     "email": "updated_email@example.com"
   }
   ```

3. Create a PATCH request to update password:
   ```json
   {
     "current_password": "current_password",
     "new_password": "new_password"
   }
   ```

4. Test error cases:
   - Try to update with an existing nickname
   - Try to change password without providing current password
   - Try to change password with incorrect current password

## Expected Behavior

1. Success case:
   - Status code: 200 OK
   - Response body:
     ```json
     {
       "message": "Profile updated successfully",
       "player": {
         "user_id": 74647891,
         "nickname": "updated_nickname",
         "email": "user@example.com",
         "balance": 0,
         "win_points": 0
       }
     }
     ```

2. Error cases:
   - 404 Not Found: If player doesn't exist
   - 400 Bad Request: If validation fails, with specific error messages

## Evaluation Criteria

Your implementation will be evaluated on:
1. Correct implementation of the API endpoint
2. Proper validation of all fields
3. Secure password handling
4. Clean, well-documented code
5. Appropriate error handling

## Submission

When you've completed the implementation:
1. Test your code thoroughly using Postman
2. Make sure your code is documented
3. Verify that the feature works correctly with the existing codebase
4. Submit your completed task 