# Pre-Task 8: Preparation Exercises for Player Profile Update Feature

Before implementing Task 8 (Update Player Profile), let's practice with some exercises to ensure you're familiar with the concepts and technologies involved. These exercises will help you understand Django REST Framework serializers, validation, authentication, and testing.

## Exercise 1: Understanding the Current Player Model

**Why this is important:** To update player profiles, you first need to understand the structure of the Player model, its fields, and constraints. This will help you determine what can be updated and what validation rules already exist.

First, let's examine the current Player model and understand its structure:

1. Review the current Player model in `app_core/poker_event/models.py`:
   - What fields does it contain?
   - What methods are available for password handling?
   - How are unique user IDs generated?

2. In your terminal, run the Django shell:
   ```bash
   python manage.py shell
   ```

3. Explore the Player model interactively:
   ```python
   from app_core.poker_event.models import Player
   
   # Get all players
   Player.objects.all()
   
   # First, create a test player
   player = Player(
       email="test@example.com",
       nickname="testplayer"
   )
   player.set_password("test123")  # Now this should work
   player.save()  # Save to database
   
   # Now you can test the methods
   player.check_password("test123")  # Should return True
   player.check_password("wrong")    # Should return False
   ```

   **Expected Output:**
   ```
   >>> Player.objects.all()
   <QuerySet []>  # Initially empty, then after creating a player:
   <QuerySet [<Player: testplayer>]>
   
   >>> player = Player(email="test@example.com", nickname="testplayer")
   >>> player.set_password("test123")
   >>> player.save()
   
   >>> player.check_password("test123")
   True
   >>> player.check_password("wrong")
   False
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
   
   # Create a test instance
   player = Player.objects.first()
   
   # Serialize the player
   serializer = PlayerTestSerializer(player)
   print(serializer.data)
   
   # Try deserializing and validating
   test_data = {
       'email': 'test@example.com',
       'nickname': 'testnick'
   }
   serializer = PlayerTestSerializer(data=test_data, partial=True)
   print(serializer.is_valid())
   print(serializer.validated_data if serializer.is_valid() else serializer.errors)
   ```

   **Expected Output:**
   ```
   >>> serializer = PlayerTestSerializer(player)
   >>> print(serializer.data)
   {'user_id': 74647891, 'email': 'test@example.com', 'nickname': 'testplayer', 'balance': 0}
   
   >>> serializer = PlayerTestSerializer(data=test_data, partial=True)
   >>> print(serializer.is_valid())
   False
   >>> print(serializer.validated_data if serializer.is_valid() else serializer.errors)
   {'email': [ErrorDetail(string='player with this email already exists.', code='unique')]}
   ```
   
   The validation fails because we're trying to use an email that already exists in the database. This shows how ModelSerializer automatically applies model-level validation rules.

   Try again with a different email:
   ```python
   test_data = {
       'email': 'another@example.com',
       'nickname': 'testnick'
   }
   serializer = PlayerTestSerializer(data=test_data, partial=True)
   print(serializer.is_valid())
   print(serializer.validated_data if serializer.is_valid() else serializer.errors)
   ```

   **Expected Output:**
   ```
   >>> print(serializer.is_valid())
   True
   >>> print(serializer.validated_data)
   {'email': 'another@example.com', 'nickname': 'testnick'}
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
           if not value.endswith('@example.com'):
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

   **Expected Output:**
   ```
   Testing {'email': 'good@example.com', 'nickname': 'goodname'}:
     Valid: True
     Result: {'email': 'good@example.com', 'nickname': 'goodname'}

   Testing {'email': 'bad@gmail.com', 'nickname': 'goodname'}:
     Valid: False
     Result: {'email': [ErrorDetail(string='Email must end with @example.com', code='invalid')]}

   Testing {'email': 'good@example.com', 'nickname': 'bad'}:
     Valid: False
     Result: {'nickname': [ErrorDetail(string='Nickname must be at least 5 characters', code='invalid')]}
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
       class Meta:
           model = Player
           fields = ['nickname', 'email']
       
       def update(self, instance, validated_data):
           # Print what we're updating
           print(f"Updating player {instance.nickname} with {validated_data}")
           
           # Update the instance
           for attr, value in validated_data.items():
               setattr(instance, attr, value)
           
           # We won't save to the database in this test
           # instance.save()
           return instance
   
   # Get a player to update
   player = Player.objects.first()
   print(f"Original player: {player.nickname}, {player.email}")
   
   # Test updating
   serializer = PlayerUpdateSerializer(player, data={'nickname': 'updated_name'}, partial=True)
   if serializer.is_valid():
       updated_player = serializer.save()  # This calls our update method
       print(f"Updated player: {updated_player.nickname}, {updated_player.email}")
   else:
       print(f"Validation failed: {serializer.errors}")
   ```

   **Expected Output:**
   ```
   >>> player = Player.objects.first()
   >>> print(f"Original player: {player.nickname}, {player.email}")
   Original player: testplayer, test@example.com
   
   >>> serializer = PlayerUpdateSerializer(player, data={'nickname': 'updated_name'}, partial=True)
   >>> if serializer.is_valid():
   ...     updated_player = serializer.save()
   ...     print(f"Updated player: {updated_player.nickname}, {updated_player.email}")
   ... else:
   ...     print(f"Validation failed: {serializer.errors}")
   ... 
   Updating player testplayer with {'nickname': 'updated_name'}
   Updated player: updated_name, test@example.com
   ```
   
   **Key Findings:**
   - The `update()` method lets us control exactly how model instances are updated
   - We can add custom logic like password hashing in our update method
   - For our profile update feature, we'll create a similar serializer with custom update logic

## Exercise 5: Set Up Postman for API Testing

**Why this is important:** Once we implement the profile update feature, we need to test it. Postman is an excellent tool for testing RESTful APIs without writing code. It allows us to quickly verify that our endpoints work as expected.

For testing REST APIs, Postman is an excellent tool. Let's set it up:

1. If you haven't already, download and install [Postman](https://www.postman.com/downloads/)

2. Create a collection for the Poker API:
   - Open Postman
   - Click "Collections" in the sidebar
   - Click "+" to create a new collection
   - Name it "Poker API"

3. Create a request for player registration:
   - In your Poker API collection, click "..." and choose "Add request"
   - Name it "Register Player"
   - Set the method to POST and URL to `http://localhost:8000/players/register/`
   - In the Body tab, select "raw" and "JSON"
   - Add the registration payload:
     ```json
     {
       "email": "test@example.com",
       "nickname": "testplayer",
       "password": "TestPass123"
     }
     ```
   - Save the request

4. Create a request for player login:
   - Add another request, name it "Login Player"
   - Set method to POST and URL to `http://localhost:8000/players/login/`
   - Add the login payload:
     ```json
     {
       "email": "test@example.com",
       "password": "TestPass123"
     }
     ```
   - Save the request

5. Create a request to get all players:
   - Add another request, name it "Get All Players"
   - Set method to GET and URL to `http://localhost:8000/players/`
   - Save the request

   **Expected Result:**
   You should have a Postman collection with three requests ready for testing. No output is expected at this stage since we're just setting up the test environment. You'll use these requests later when testing your implementation.

   **Key Benefits:**
   - Allows quick testing of API endpoints without writing code
   - Provides a way to save and reuse API requests
   - Helps document the API for other developers
   - For our profile update feature, we'll add a new request to test the update endpoint

Refer to our [Postman Testing Guide](POSTMAN_GUIDE_POKER.md) for more detailed instructions on API testing.

## Exercise 6: Explore User Identification Methods

**Why this is important:** When implementing user profile updates, we need a way to identify which user is making the request and ensure users can only update their own profiles.

Let's explore a few ways to identify users in API requests:

1. **URL Parameters:** Using user IDs in the URL path
   ```
   GET /players/74647891/  # Get details for a specific player
   PATCH /players/74647891/  # Update a specific player
   ```

2. **Request Body:** Including user ID in the request body
   ```json
   {
     "user_id": 74647891,
     "nickname": "new_nickname"
   }
   ```

3. **Session-based Authentication:** If your Django app is using sessions
   ```python
   # In your view
   def update_profile(request):
       # The current user is available in request.user
       user = request.user
       # Now update the user's profile
   ```

4. Let's test identifying users by user_id in the URL:
   - In Postman, create a request named "Get Player by ID"
   - Set method to GET and URL to `http://localhost:8000/players/74647891/` (use an actual user_id from your database)
   - Save and send the request
   - You should get details for just that specific player

5. When implementing the profile update feature, you'll need to:
   - Create an endpoint like `/players/profile/<user_id>/`
   - In your view, check if the requested user_id matches the authenticated user
   - Only allow updates if the user is updating their own profile

**Expected Result:**
You'll understand different ways to identify users when making API requests, which will be important when implementing the profile update feature.

**Key Considerations:**
- Choose an identification method that works with your current authentication system
- Always verify the user has permission to update the profile they're trying to modify
- For security, add validation to ensure users can only update their own profiles
- In Django views, you can implement permission checks before processing updates

## Next Steps: Implementing Task 8

Now that you've practiced with the underlying concepts, you're ready to implement Task 8: Update Player Profile. For detailed instructions on how to implement this feature, refer to [Task8_furkan.md](Task8_furkan.md).

For testing your implementation with Postman, we've prepared a dedicated guide: [POSTMAN_GUIDE_POKER.md](POSTMAN_GUIDE_POKER.md).
