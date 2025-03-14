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

## Exercise 2: Practicing with Model Serializers

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

## Exercise 3: Practice with Validation

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
   
   This demonstrates how custom validation methods can be added to serializers to enforce specific business rules.

## Exercise 4: Practice with Update Operations

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
   
   This demonstrates how serializer's `update()` method is called when you call `save()` on an existing instance. The custom `update()` method allows you to control how the instance is updated.

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

   **Expected Result:**
   You should have a Postman collection with two requests ready for testing. No output is expected at this stage since we're just setting up the test environment. You'll use these requests later when testing your implementation.

Refer to our [Postman Testing Guide](POSTMAN_GUIDE_POKER.md) for more detailed instructions on API testing.

## Exercise 6: Explore Token Authentication

Let's understand how token authentication works:

1. In Django settings, ensure `rest_framework.authtoken` is in `INSTALLED_APPS`

2. Run migrations to create the token table:
   ```bash
   python manage.py migrate
   ```

3. In the Django shell, try creating a token:
   ```bash
   python manage.py shell
   ```
   
   Then in the shell:
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

   **Expected Output:**
   ```
   >>> player = Player.objects.first()
   >>> token, created = Token.objects.get_or_create(user=player)
   >>> print(f"Token: {token.key}")
   Token: a1b2c3d4e5f6g7h8i9j0...  # Your token will be different
   >>> print(f"Created new: {created}")
   Created new: True  # First time will be True, subsequent runs will be False
   
   >>> token = Token.objects.get(user=player)
   >>> print(f"Retrieved token: {token.key}")
   Retrieved token: a1b2c3d4e5f6g7h8i9j0...  # Same token as above
   ```

   Note: If you get an error like `TypeError: 'user' is an invalid keyword argument for this function`, it might mean your Player model isn't properly set up as an auth user model. In that case, you may need to check how authentication is implemented in your project.

4. In Postman, create a request that uses token authentication:
   - Add a new request called "Test Authentication"
   - Set method to GET and URL to any endpoint like `http://localhost:8000/players/`
   - In the Headers tab, add:
     - Key: `Authorization`
     - Value: `Token YOUR_TOKEN_KEY` (replace with actual token)
   - Save the request

   **Expected Result:**
   You'll have a Postman request configured to use token authentication. When you send this request, the server should recognize your authenticated user if token authentication is properly set up.

## Next Steps: Implementing Task 8

Now that you've practiced with the underlying concepts, you're ready to implement Task 8: Update Player Profile. For detailed instructions on how to implement this feature, refer to [Task8_furkan.md](Task8_furkan.md).

For testing your implementation with Postman, we've prepared a dedicated guide: [POSTMAN_GUIDE_POKER.md](POSTMAN_GUIDE_POKER.md).
