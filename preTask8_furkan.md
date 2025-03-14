# Pre-Task 8: Preparation Exercises for Player Profile Update Feature

Before implementing Task 8 (Update Player Profile), let's practice with some exercises to ensure you're familiar with the concepts and technologies involved. These exercises will help you understand Django REST Framework serializers, validation, and how to update player profiles in your specific codebase.

## Exercise 1: Understanding the Current Player Model

**Why this is important:** To update player profiles, you first need to understand the structure of the Player model, its fields, and constraints. This will help you determine what can be updated and what validation rules already exist.

First, let's examine the current Player model and understand its structure:

1. Review the current Player model in `app_core/poker_event/models.py`:
   - What fields does it contain? (user_pk_id, user_id, email, password, nickname, balance, win_points, created_at)
   - What methods are available for password handling? (set_password, check_password)
   - How are unique user IDs generated? (using the generate_unique_user_id function)

2. In your terminal, run the Django shell:
   ```bash
   python manage.py shell
   ```

3. Explore the Player model interactively:
   ```python
   from app_core.poker_event.models import Player
   
   # Get all players
   Player.objects.all()
   
   # Create a test player if none exists
   if not Player.objects.exists():
       player = Player(
           email="test@example.com",
           nickname="testplayer"
       )
       player.set_password("test123")
       player.save()
   else:
       player = Player.objects.first()
   
   # Examine the model
   print(f"User ID: {player.user_id}")
   print(f"Email: {player.email}")
   print(f"Nickname: {player.nickname}")
   print(f"Balance: {player.balance}")
   
   # Test password methods
   player.check_password("test123")  # Should return True if this was the password
   player.check_password("wrong")    # Should return False
   ```

   **Key Findings:**
   - The Player model has fields for email, nickname, and password
   - It has methods for password handling (set_password, check_password)
   - The email and nickname fields have unique constraints
   - Understanding these constraints is essential when updating user profiles to avoid duplicate data

## Exercise 2: Practicing with Model Serializers

**Why this is important:** Serializers are the backbone of our update feature. They convert complex data types (like model instances) to Python data types that can be easily rendered into JSON, and they handle validation and data conversion when updating models.

Let's practice with DRF's ModelSerializer to understand how it works:

1. **Run the Django shell**:
   ```bash
   python manage.py shell
   ```

2. **Copy and paste the following code** into the shell to test serializers:
   ```python
   from rest_framework import serializers
   from app_core.poker_event.models import Player
   
   class PlayerTestSerializer(serializers.ModelSerializer):
       class Meta:
           model = Player
           fields = ['user_id', 'email', 'nickname', 'balance']
   
   # Get a player to serialize
   player = Player.objects.first()
   
   # Serialize the player
   serializer = PlayerTestSerializer(player)
   print(serializer.data)
   
   # Try deserializing and validating
   test_data = {
       'email': 'updated@example.com',
       'nickname': 'updatednick'
   }
   serializer = PlayerTestSerializer(data=test_data, partial=True)
   print(serializer.is_valid())
   print(serializer.validated_data if serializer.is_valid() else serializer.errors)
   ```

   **Key Findings:**
   - Serializers automatically validate model constraints like unique fields
   - The `partial=True` parameter allows us to update only some fields
   - For our profile update feature, we'll need to handle validation errors appropriately

## Exercise 3: Practice with Validation

**Why this is important:** User inputs must be validated to ensure data integrity and security. For the profile update feature, we need to implement custom validation rules for email format, nickname uniqueness, and password security.

Let's practice implementing custom validation in serializers:

1. **Continue in the Django shell** or start a new shell session:
   ```bash
   python manage.py shell
   ```

2. **Copy and paste the following code** to test validation:
   ```python
   from rest_framework import serializers
   from app_core.poker_event.models import Player
   
   class ValidatedPlayerSerializer(serializers.ModelSerializer):
       class Meta:
           model = Player
           fields = ['user_id', 'email', 'nickname', 'balance']
       
       def validate_email(self, value):
           """Validate email format."""
           if value and not value.endswith('@example.com'):
               raise serializers.ValidationError("Email must end with @example.com")
           return value
       
       def validate_nickname(self, value):
           """Ensure nickname is at least 5 characters."""
           if len(value) < 5:
               raise serializers.ValidationError("Nickname must be at least 5 characters")
           return value
   
   # Test the validation
   test_data = [
       {'email': 'good@example.com', 'nickname': 'goodname'},  # Should be valid
       {'email': 'bad@gmail.com', 'nickname': 'goodname'},     # Bad email
       {'email': 'good@example.com', 'nickname': 'bad'},       # Bad nickname
   ]
   
   for data in test_data:
       serializer = ValidatedPlayerSerializer(data=data)
       print(f"Testing {data}:")
       print(f"  Valid: {serializer.is_valid()}")
       print(f"  Result: {serializer.validated_data if serializer.is_valid() else serializer.errors}")
       print()
   ```
   
   **Key Findings:**
   - Custom validation methods allow us to implement business rules
   - We can provide specific error messages for each validation rule
   - For our profile update feature, we'll need similar validation for email format, password strength, etc.

## Exercise 4: Practice with Update Operations

**Why this is important:** The core of our profile update feature will involve updating existing player records. This exercise shows how serializers handle the update process and how we can customize it.

Let's practice updating model instances with serializers:

1. **Run the Django shell**:
   ```bash
   python manage.py shell
   ```

2. **Copy and paste the following code** to test update operations:
   ```python
   from rest_framework import serializers
   from app_core.poker_event.models import Player
   
   class PlayerUpdateSerializer(serializers.ModelSerializer):
       current_password = serializers.CharField(write_only=True, required=False)
       new_password = serializers.CharField(write_only=True, required=False)
       
       class Meta:
           model = Player
           fields = ['nickname', 'email', 'current_password', 'new_password']
           extra_kwargs = {
               'email': {'required': False},
               'nickname': {'required': False}
           }
       
       def validate(self, data):
           # Check if password update is requested
           if 'new_password' in data and 'current_password' not in data:
               raise serializers.ValidationError({"current_password": "Current password is required to set a new password"})
           
           # If current_password is provided, verify it
           if 'current_password' in data:
               if not self.instance.check_password(data['current_password']):
                   raise serializers.ValidationError({"current_password": "Current password is incorrect"})
           
           return data
       
       def update(self, instance, validated_data):
           # Print what we're updating
           print(f"Updating player {instance.nickname} with {validated_data}")
           
           # Handle password update
           if 'new_password' in validated_data:
               instance.set_password(validated_data.pop('new_password'))
           
           # Remove current_password as we don't need to store it
           validated_data.pop('current_password', None)
           
           # Update remaining fields
           for attr, value in validated_data.items():
               setattr(instance, attr, value)
           
           # We'll comment out the save to avoid modifying the database in this test
           # instance.save()
           return instance
   
   # Get a player to update
   player = Player.objects.first()
   print(f"Original player: {player.nickname}, {player.email}")
   
   # Test updating nickname only
   serializer = PlayerUpdateSerializer(player, data={'nickname': 'updated_name'}, partial=True)
   if serializer.is_valid():
       updated_player = serializer.save()
       print(f"Updated player: {updated_player.nickname}, {updated_player.email}")
   else:
       print(f"Validation failed: {serializer.errors}")
   
   # Test updating with password change
   serializer = PlayerUpdateSerializer(player, data={
       'nickname': 'updated_name',
       'current_password': 'test123',  # This should match the actual password
       'new_password': 'new_secure_password'
   }, partial=True)
   if serializer.is_valid():
       updated_player = serializer.save()
       print(f"Updated player with password: {updated_player.nickname}")
       print(f"Password updated: {updated_player.check_password('new_secure_password')}")
   else:
       print(f"Validation failed: {serializer.errors}")
   ```
   
   **Key Findings:**
   - The `update()` method lets us control exactly how model instances are updated
   - We can implement specific validation for password changes
   - We need to handle both partial updates (just nickname or email) and full updates

## Exercise 5: Set Up Postman for Testing the Profile Update API

**Why this is important:** Once we implement the profile update feature, we need to test it. Postman is an excellent tool for testing RESTful APIs without writing code. It allows us to quickly verify that our endpoints work as expected.

1. If you haven't already, download and install [Postman](https://www.postman.com/downloads/)

2. Create a request for the profile update endpoint (which we'll implement):
   - In your Poker API collection, add a new request
   - Name it "Update Player Profile"
   - Set the method to PATCH
   - Set the URL to `http://localhost:8000/players/profile/<user_id>/`
   - In the Body tab, select "raw" and "JSON"
   - Add the profile update payload:
     ```json
     {
       "nickname": "updated_nickname",
       "email": "updated_email@example.com",
       "current_password": "current_password",
       "new_password": "new_password"
     }
     ```
   - Save the request

   **Key Benefits:**
   - Allows quick testing of API endpoints without writing code
   - Provides a way to save and reuse API requests
   - Helps document the API for other developers

3. Create test variations:
   - Duplicate the request and modify it to test updating only the nickname
   - Create another variation to test updating only the email
   - Create another variation to test updating the password

Refer to our [Postman Testing Guide](POSTMAN_GUIDE_POKER.md) for more detailed instructions on API testing.

## Next Steps: Implementing Task 8

Now that you've practiced with the underlying concepts, you're ready to implement Task 8: Update Player Profile.

Here's what you need to do:

1. Create a new serializer in `app_core/poker_event/serializers.py`:
   ```python
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
       
       # Add validation and update methods as practiced in Exercise 4
   ```

2. Create a new view in `app_core/poker_event/views.py`:
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

3. Add the new URL route in `api/urls.py`:
   ```python
   path('players/profile/<int:user_id>/', views.update_player_profile, name='update-player-profile'),
   ```

4. Test your implementation using Postman according to the setup in Exercise 5.

For testing your implementation with Postman, you can use the Postman Testing Guide.
