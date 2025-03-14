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

   **Expected Result:**
   You should have a Postman collection with two requests ready for testing. No output is expected at this stage since we're just setting up the test environment. You'll use these requests later when testing your implementation.

   **Key Benefits:**
   - Allows quick testing of API endpoints without writing code
   - Provides a way to save and reuse API requests
   - Helps document the API for other developers
   - For our profile update feature, we'll add a new request to test the update endpoint

Refer to our [Postman Testing Guide](POSTMAN_GUIDE_POKER.md) for more detailed instructions on API testing.

## Exercise 6: Explore Token Authentication

**Why this is important:** Our profile update endpoint needs to be secured so that users can only update their own profiles. Token authentication is a common way to secure REST APIs and identify users.

Let's understand how token authentication works with our custom Player model:

1. In Django settings, ensure `rest_framework.authtoken` is in `INSTALLED_APPS` - this is already done in your project.

2. The standard Django Token authentication is designed to work with Django's built-in User model, but our project uses a custom Player model instead. We need to create a custom adapter to make them work together.

3. Let's create a custom authentication class. In your Django shell:
   ```bash
   python manage.py shell
   ```
   
   Then in the shell:
   ```python
   from rest_framework.authtoken.models import Token
   from app_core.poker_event.models import Player
   
   # Create a custom token generator function
   def create_player_token(player):
       """Create a token that links to our Player model instead of User model"""
       # First, try to reuse the primary key as a token key
       token_key = f"pk{player.user_pk_id}"
       
       # Create and return a token with our custom key
       token = Token(key=token_key)
       # We can't directly use the token.user field with our Player model
       # But we can reference the player ID for later retrieval
       token.user_id = None  # This will be blank since we're not using Django User
       token.save()
       
       print(f"Created token: {token.key} for player: {player.nickname}")
       return token
   
   # Get a player
   player = Player.objects.first()
   
   # Create a token for the player
   token = create_player_token(player)
   print(f"Token: {token.key}")
   
   # The token can be used for authentication by including it in request headers
   print(f"Use in Postman with header: Authorization: Token {token.key}")
   ```

   **Expected Output:**
   ```
   >>> player = Player.objects.first()
   >>> token = create_player_token(player)
   Created token: pk1 for player: testplayer
   >>> print(f"Token: {token.key}")
   Token: pk1
   >>> print(f"Use in Postman with header: Authorization: Token {token.key}")
   Use in Postman with header: Authorization: Token pk1
   ```

4. To properly implement token authentication for our custom Player model, we would need to:

   a. Create a custom authentication class:
   ```python
   # This would go in a file like app_core/poker_event/authentication.py
   from rest_framework.authentication import TokenAuthentication, get_authorization_header
   from rest_framework.exceptions import AuthenticationFailed
   from .models import Player
   from rest_framework.authtoken.models import Token

   class PlayerTokenAuthentication(TokenAuthentication):
       """Custom token authentication for Player model instead of User model"""
       
       def authenticate_credentials(self, key):
           try:
               token = Token.objects.get(key=key)
           except Token.DoesNotExist:
               raise AuthenticationFailed('Invalid token')
           
           # Extract player ID from token key (assuming format "pk{id}")
           if key.startswith('pk'):
               try:
                   player_id = int(key[2:])
                   player = Player.objects.get(user_pk_id=player_id)
                   return (player, token)
               except (ValueError, Player.DoesNotExist):
                   raise AuthenticationFailed('Invalid token - no matching player')
           
           raise AuthenticationFailed('Invalid token format')
   ```

   b. Add this to your REST_FRAMEWORK settings:
   ```python
   # This would go in your settings.py
   REST_FRAMEWORK = {
       'DEFAULT_AUTHENTICATION_CLASSES': (
           'app_core.poker_event.authentication.PlayerTokenAuthentication',
           'rest_framework.authentication.SessionAuthentication',
           'rest_framework.authentication.BasicAuthentication',
       ),
   }
   ```

   c. Secure your views with authentication:
   ```python
   # Example of a protected view
   from rest_framework.decorators import api_view, authentication_classes, permission_classes
   from rest_framework.permissions import IsAuthenticated
   from .authentication import PlayerTokenAuthentication
   
   @api_view(["GET"])
   @authentication_classes([PlayerTokenAuthentication])
   @permission_classes([IsAuthenticated])
   def get_player_profile(request):
       """Returns the profile of the authenticated player"""
       player = request.user  # In DRF, the authenticated user is in request.user
       serializer = PlayerProfileSerializer(player)
       return Response(serializer.data)
   ```

5. In Postman, create a request that uses token authentication:
   - Add a new request called "Get Player Profile" (this is hypothetical until you implement it)
   - Set method to GET and URL to `http://localhost:8000/players/profile/`
   - In the Headers tab, add:
     - Key: `Authorization`
     - Value: `Token pk1` (replace with your actual token)
   - Save the request

   **Expected Result:**
   This setup will help you understand how token authentication works with custom models. For Task 8, you'll implement the actual authentication flow for your profile update feature.

   **Key Benefits:**
   - Demonstrates how to adapt DRF token authentication to work with custom models
   - Shows how to secure endpoints to prevent unauthorized access
   - Provides a foundation for implementing user-specific features

## Next Steps: Implementing Task 8

Now that you've practiced with the underlying concepts and understand how to adapt token authentication to your custom Player model, you're ready to implement Task 8: Update Player Profile. For detailed instructions on how to implement this feature, refer to [Task8_furkan.md](Task8_furkan.md).

For testing your implementation with Postman, we've prepared a dedicated guide: [POSTMAN_GUIDE_POKER.md](POSTMAN_GUIDE_POKER.md).
