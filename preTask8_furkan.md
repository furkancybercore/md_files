# Pre-Task 8: Preparation Exercises for Player Profile Update Feature

Before implementing Task 8 (Update Player Profile), let's practice with some exercises to ensure you're familiar with the concepts and technologies involved. These exercises will help you understand Django REST Framework serializers, validation, authentication, and testing.

## Exercise 1: Understanding the Current Player Model

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

## Exercise 2: Practicing with Model Serializers

Let's practice with DRF's ModelSerializer to understand how it works:

1. Create a test file `serializer_test.py` with the following code:
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

2. Run this script:
   ```bash
   python -c "exec(open('serializer_test.py').read())"
   ```

## Exercise 3: Practice with Validation

Let's practice implementing custom validation in serializers:

1. Extend your `serializer_test.py` file:
   ```python
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

2. Run the updated script to see validation in action.

## Exercise 4: Practice with Update Operations

Let's practice updating model instances with serializers:

1. Create a new file `update_test.py`:
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

2. Run this script to see how updating works.

## Exercise 5: Set Up Postman for API Testing

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

Refer to our [Postman Testing Guide](POSTMAN_GUIDE_POKER.md) for more detailed instructions on API testing.

## Exercise 6: Explore Token Authentication

Let's understand how token authentication works:

1. In Django settings, ensure `rest_framework.authtoken` is in `INSTALLED_APPS`

2. Run migrations to create the token table:
   ```bash
   python manage.py migrate
   ```

3. In the Django shell, try creating a token:
   ```python
   from rest_framework.authtoken.models import Token
   from app_core.poker_event.models import Player
   
   # Get a player
   player = Player.objects.first()
   
   # Create a token
   token, created = Token.objects.get_or_create(user=player)
   print(f"Token: {token.key}")
   print(f"Created new: {created}")
   
   # Retrieve token for a user
   token = Token.objects.get(user=player)
   print(f"Retrieved token: {token.key}")
   ```

4. In Postman, create a request that uses token authentication:
   - Add a new request called "Test Authentication"
   - Set method to GET and URL to any endpoint like `http://localhost:8000/players/`
   - In the Headers tab, add:
     - Key: `Authorization`
     - Value: `Token YOUR_TOKEN_KEY` (replace with actual token)
   - Save the request

## Next Steps: Implementing Task 8

Now that you've practiced with the underlying concepts, you're ready to implement Task 8: Update Player Profile. For detailed instructions on how to implement this feature, refer to [Task8_furkan.md](Task8_furkan.md).

For testing your implementation with Postman, we've prepared a dedicated guide: [POSTMAN_GUIDE_POKER.md](POSTMAN_GUIDE_POKER.md).
