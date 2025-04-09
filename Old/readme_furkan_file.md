# Poker Backend - File-by-File Documentation

This document provides a detailed explanation of each file in the poker backend project, describing their purpose, contents, and how they interact with other files in the system.

## Project Structure

The project follows this structure:

```
poker-backend/
├── backend/
│   ├── api/
│   │   ├── __init__.py
│   │   ├── settings.py  # Project settings
│   │   ├── urls.py      # Main URL routing
│   │   ├── asgi.py
│   │   └── wsgi.py
│   ├── app_core/
│   │   ├── __init__.py
│   │   ├── admin.py
│   │   ├── apps.py
│   │   ├── migrations/
│   │   ├── models.py
│   │   ├── tests.py
│   │   ├── views.py
│   │   └── poker_event/
│   │       ├── __init__.py
│   │       ├── models.py    # Database models
│   │       ├── serializers.py  # Serializers
│   │       └── views.py     # API views
│   └── manage.py
```

## File: api/urls.py

This file defines the URL routing for the Django application, mapping URLs to their corresponding view functions.

```python
from django.contrib import admin
from django.urls import path
from app_core.poker_event import views

urlpatterns = [
    path('admin/', admin.site.urls),  # Django admin panel for managing the application
    path('players/register/', views.register, name='register'),  # Registers a new player account
    path('players/login/', views.login, name='login'),  # Authenticates and logs in an existing player
    path("players/", views.get_all_players, name="get-all-players"),  # Retrieves a list of all registered players
    path("events/create", views.createEvent, name="create-event"),  # Creates a new game room
    path("events/", views.getAllEvents, name="get-all-events"),  # Retrieves all available game rooms
    path("events/join", views.joinEvent, name="join-event"),  # Allows a player to join an existing room
    path("events/<int:event_id>/", views.getEventmDetails, name="event-details"),  # Retrieves detailed information about a specific game room
]
```

### Key Points:
- The file imports views from `app_core.poker_event.views`
- Defines 8 URL patterns that map to specific view functions
- Uses named URLs for reverse lookups
- Includes parameters in URLs where needed (e.g., `event_id`)
- Provides descriptive comments for each URL pattern

## File: app_core/poker_event/models.py

This file defines the database models that represent the application's data structure. It includes models for players, events, deposits, transactions, and game sessions.

### Player Model

```python
def generate_unique_user_id():
    """Generates a unique 8-digit user ID."""
    while True:
        new_id = random.randint(10000000, 99999999)
        if not Player.objects.filter(user_id=new_id).exists():
            return new_id

class Player(models.Model):
    """Stores player information including authentication credentials and balances."""
    user_pk_id = models.AutoField(primary_key=True)
    user_id = models.PositiveIntegerField(unique=True, default=generate_unique_user_id, editable=False)
    email = models.EmailField(unique=True, null=True, blank=True)
    password = models.CharField(max_length=255)
    nickname = models.CharField(max_length=255, unique=True)
    balance = models.IntegerField(default=0)
    win_points = models.IntegerField(default=0)
    created_at = models.DateTimeField(auto_now_add=True)

    def set_password(self, raw_password):
        """Hash and set the player's password securely."""
        self.password = make_password(raw_password)

    def check_password(self, raw_password):
        """Verify if the input password matches the hashed password."""
        return check_password(raw_password, self.password)
```

Key features:
- Generates a random 8-digit ID for each user
- Includes methods for password hashing and verification
- Tracks player balance and win points
- Ensures unique emails and nicknames

### Event Model

```python
class Event(models.Model):
    """Represents a scheduled poker event."""
    event_name = models.CharField(max_length=255)
    creator = models.ForeignKey(Player, on_delete=models.SET_NULL, null=True, blank=True)
    max_capacity = models.IntegerField()
    small_blind = models.IntegerField()
    big_blind = models.IntegerField()
    required_deposit = models.IntegerField()  # Minimum deposit required
    location = models.TextField()
    game_date = models.DateField()
    start_time = models.TimeField(null=True, blank=True)
    duration = models.IntegerField()  # Duration in minutes
    created_at = models.DateTimeField(auto_now_add=True)

    # Regularity for recurring events
    REGULARITY_CHOICES = [
        ("once", "Once"),
        ("daily", "Daily"),
        ("weekly", "Weekly"),
        ("fortnightly", "Fortnightly"),
        ("monthly", "Monthly"),
    ]
    regularity = models.CharField(max_length=20, choices=REGULARITY_CHOICES, default="once")
```

Key features:
- Links to a creator (Player)
- Includes game settings like blinds and required deposits
- Stores location and scheduling information
- Supports recurring events with a regularity field

### EventDeposit Model

```python
class EventDeposit(models.Model):
    """Tracks deposits made by players for joining events."""
    event = models.ForeignKey(Event, on_delete=models.CASCADE)
    user = models.ForeignKey(Player, on_delete=models.CASCADE)
    deposit_amount = models.IntegerField()
    deposit_time = models.DateTimeField(auto_now_add=True)

    STATUS_CHOICES = [
        ("pending", "Pending"),
        ("confirmed", "Confirmed"),
        ("cancelled", "Cancelled"),
    ]
    status = models.CharField(max_length=10, choices=STATUS_CHOICES, default="pending")
```

Key features:
- Links a Player to an Event via deposits
- Tracks the deposit amount and time
- Includes a status field for tracking the deposit state

### Transaction Model

```python
class Transaction(models.Model):
    """Logs chip or money transactions between players."""
    source_user = models.ForeignKey(Player, related_name="sent_transactions", on_delete=models.SET_NULL, null=True, blank=True)
    target_user = models.ForeignKey(Player, related_name="received_transactions", on_delete=models.SET_NULL, null=True, blank=True)
    deposit = models.ForeignKey(EventDeposit, on_delete=models.SET_NULL, null=True, blank=True)
    
    TRANSACTION_TYPES = [
        ("deposit", "Deposit"),
        ("withdrawal", "Withdrawal"),
        ("return", "Return"),
    ]
    transaction_type = models.CharField(max_length=20, choices=TRANSACTION_TYPES)
    amount = models.IntegerField()
    transaction_date = models.DateTimeField(auto_now_add=True)
```

Key features:
- Tracks money movements between players
- References source and target players
- Can be linked to a specific deposit
- Categorizes transactions by type

### EventPlayer Model

```python
class EventPlayer(models.Model):
    """Tracks players who have registered for an event."""
    event = models.ForeignKey(Event, on_delete=models.CASCADE)
    user = models.ForeignKey(Player, on_delete=models.CASCADE)
    
    ROLE_CHOICES = [
        ("host", "Host"),
        ("player", "Player"),
    ]
    role = models.CharField(max_length=10, choices=ROLE_CHOICES, default="player")
    joined_at = models.DateTimeField(auto_now_add=True)
```

Key features:
- Implements the many-to-many relationship between players and events
- Includes a role field to distinguish hosts from regular players
- Records when a player joined an event

### GameSession and GameSessionResult Models

```python
class GameSession(models.Model):
    """Stores details of completed game sessions."""
    event = models.ForeignKey(Event, on_delete=models.CASCADE)
    winner = models.ForeignKey(Player, on_delete=models.SET_NULL, null=True, blank=True)
    pot_amount = models.IntegerField(default=0)
    created_at = models.DateTimeField(auto_now_add=True)

class GameSessionResult(models.Model):
    """Records individual player results from completed game sessions."""
    session = models.ForeignKey(GameSession, on_delete=models.CASCADE)
    user = models.ForeignKey(Player, on_delete=models.CASCADE)
    start_chips = models.IntegerField()
    final_chips = models.IntegerField()
    is_winner = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True)
```

Key features:
- GameSession tracks completed games and overall results
- GameSessionResult tracks individual player performance
- Records chip counts at start and end
- Flags winners of sessions

## File: app_core/poker_event/serializers.py

This file defines the serializers that convert between model instances and JSON representations, as well as handle validation and data processing.

### Player Registration and Login Serializers

```python
class PlayerRegisterSerializer(serializers.ModelSerializer):
    class Meta:
        model = Player
        fields = ["email", "nickname", "password"]
        extra_kwargs = {"password": {"write_only": True}}  # Hide password in responses

    def create(self, validated_data):
        """Hash password before saving the player."""
        password = validated_data.pop("password")
        player = Player(**validated_data)
        player.set_password(password)  # Hash the password before saving
        player.save()
        return player

class PlayerLoginSerializer(serializers.Serializer):
    email = serializers.EmailField()
    password = serializers.CharField(write_only=True)

    def validate(self, data):
        """Validate user credentials."""
        email = data.get("email")
        password = data.get("password")

        player = Player.objects.filter(email=email).first()
        if not player:
            raise serializers.ValidationError("Player not found")

        if not player.check_password(password):
            raise serializers.ValidationError("Incorrect password")

        return player
```

Key features:
- PlayerRegisterSerializer extends ModelSerializer for automatic field mapping
- Overrides create() to handle password hashing
- PlayerLoginSerializer implements custom validation
- Marks password as write_only for security

### AllPlayerSerializer

```python
class AllPlayerSerializer(serializers.ModelSerializer):
    class Meta:
        model = Player
        fields = ["user_id", "email", "nickname", "balance", "win_points", "created_at"]
```

Key features:
- Simple ModelSerializer for retrieving player information
- Excludes sensitive fields like password

### Event Serializers

```python
class CreateEventSerializer(serializers.ModelSerializer):
    creator_id = serializers.IntegerField(write_only=True)  # Takes an ID from request

    class Meta:
        model = Event
        fields = [
            "event_name", "creator_id", "max_capacity", "small_blind", 
            "big_blind", "required_deposit", "location", "game_date", 
            "start_time", "duration", "created_at"
        ]
        read_only_fields = ["created_at"]

    def create(self, validated_data):
        """Creates a new game Event with a valid creator."""
        creator_id = validated_data.pop("creator_id")
        creator = Player.objects.filter(user_id=creator_id).first()

        if not creator:
            raise serializers.ValidationError({"creator_id": "Invalid creator ID"})

        return Event.objects.create(creator=creator, **validated_data)

class AllEventSerializer(serializers.ModelSerializer):
    creator = serializers.SerializerMethodField()  # Replace `creator_id` with `creator`

    class Meta:
        model = Event
        fields = [
            "id", "event_name", "creator", "max_capacity", "small_blind",
            "big_blind", "required_deposit", "location", "game_date",
            "start_time", "duration", "created_at"
        ]

    def get_creator(self, obj):
        """Retrieve the nickname of the Event creator."""
        return obj.creator.nickname if obj.creator else None
```

Key features:
- CreateEventSerializer takes a creator_id and converts it to a creator object
- AllEventSerializer uses a SerializerMethodField to customize the creator field
- Both handle validation and conversion between model instances and JSON

### JoinEventSerializer

```python
class JoinEventSerializer(serializers.ModelSerializer):
    user_id = serializers.IntegerField(write_only=True)  # Takes user ID
    event_id = serializers.IntegerField(write_only=True)  # Takes Event ID

    class Meta:
        model = EventPlayer
        fields = ["event_id", "user_id", "joined_at"]

    def create(self, validated_data):
        """Handles adding a player to a Event."""
        user_id = validated_data.pop("user_id")
        event_id = validated_data.pop("event_id")

        # Get the Player and Event
        player = Player.objects.filter(user_id=user_id).first()
        event = Event.objects.filter(id=event_id).first()

        if not player:
            raise serializers.ValidationError({"user_id": "Invalid player ID"})
        if not event:
            raise serializers.ValidationError({"event_id": "Invalid Event ID"})

        # Check if the Event is full
        current_players = EventPlayer.objects.filter(event=event).count()
        if current_players >= event.max_capacity:
            raise serializers.ValidationError({"event": "Event is full"})

        # Check if player is already in the event
        if EventPlayer.objects.filter(event=event, user=player).exists():
            raise serializers.ValidationError({"user_id": "Player is already in this Event"})

        return EventPlayer.objects.create(event=event, user=player)
```

Key features:
- Takes user_id and event_id as input
- Performs multiple validations:
  - Verifies both the player and event exist
  - Checks if the event is full
  - Ensures the player isn't already in the event
- Creates an EventPlayer record to link the player and event

### EventDetailSerializer

```python
class EventDetailSerializer(serializers.ModelSerializer):
    player_count = serializers.SerializerMethodField()  # Get number of players in the event
    players = serializers.SerializerMethodField()  # Get player nicknames

    class Meta:
        model = Event
        fields = [
            "id", "event_name", "max_capacity", "player_count", "players",  # Include players list
            "small_blind", "big_blind", "required_deposit", "location", "game_date",
            "start_time", "duration", "created_at"
        ]

    def get_player_count(self, obj):
        """Count the number of players in the event."""
        return EventPlayer.objects.filter(event=obj).count()

    def get_players(self, obj):
        """Get a list of nicknames of players currently in the Event."""
        return list(EventPlayer.objects.filter(event=obj).values_list("user__nickname", flat=True))
```

Key features:
- Extends the basic event information with player-related data
- Uses SerializerMethodFields to compute derived values
- Provides a count of players and a list of their nicknames

## File: app_core/poker_event/views.py

This file contains the view functions that handle HTTP requests and return responses. Each view corresponds to an API endpoint.

### Player Views

```python
@api_view(["GET"])
def get_all_players(request):
    """Returns a list of all registered players."""
    players = Player.objects.all()
    serializer = serializers.AllPlayerSerializer(players, many=True)
    return Response(serializer.data, status=status.HTTP_200_OK)
    
@api_view(["POST"])
def register(request):
    """Registers a new player account."""
    serializer = serializers.PlayerRegisterSerializer(data=request.data)
    if serializer.is_valid():
        serializer.save()
        return Response({"message": "Player registered successfully"}, status=status.HTTP_201_CREATED)
    return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

@api_view(["POST"])
def login(request):
    """Authenticates a player and returns player details if successful."""
    serializer = serializers.PlayerLoginSerializer(data=request.data)
    if serializer.is_valid():
        player = serializer.validated_data
        return Response({
            "message": "Login successful",
            "player": {
                "nickname": player.nickname,
                "email": player.email,
                "balance": player.balance,
                "win_points": player.win_points,
            }
        }, status=status.HTTP_200_OK)
    return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

Key features:
- Use Django REST Framework's api_view decorator to define REST endpoints
- Handle GET and POST methods
- Validate data using serializers
- Return appropriate HTTP status codes and formatted responses

### Event Views

```python
@api_view(["POST"])
def createEvent(request):
    """Creates a new game Event with specified configurations."""
    serializer = serializers.CreateEventSerializer(data=request.data)
    if serializer.is_valid():
        event = serializer.save()
        return Response({"message": "Event created successfully", "event": serializer.data}, status=status.HTTP_201_CREATED)
    return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

@api_view(["GET"])
def getAllEvents(request):
    """Returns a list of all available game events."""
    events = Event.objects.all()
    serializer = serializers.AllEventSerializer(events, many=True)
    return Response(serializer.data, status=status.HTTP_200_OK)
    
@api_view(["POST"])
def joinEvent(request):
    """Allows a player to join an existing Event."""
    serializer = serializers.JoinEventSerializer(data=request.data)
    if serializer.is_valid():
        serializer.save()
        return Response({"message": "Player joined the Event successfully"}, status=status.HTTP_201_CREATED)
    return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

@api_view(["GET"])
def getEventmDetails(request, event_id):
    """Returns details of a specific Event including player count and nicknames."""
    try:
        event = Event.objects.get(id=event_id)
    except Event.DoesNotExist:
        return Response({"error": "event not found"}, status=status.HTTP_404_NOT_FOUND)
    
    serializer = serializers.EventDetailSerializer(event)
    return Response(serializer.data, status=status.HTTP_200_OK)
```

Key features:
- Each function handles a specific API endpoint
- createEvent and joinEvent handle POST requests with validation
- getAllEvents and getEventmDetails handle GET requests
- Use try/except blocks to handle not found errors

## File: api/settings.py

This file contains the Django configuration settings for the project.

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework', # ian
    'psycopg2', # ian
    'corsheaders', #lyh
    'app_core', #lyh
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'whitenoise.middleware.WhiteNoiseMiddleware', #lyh
    'corsheaders.middleware.CorsMiddleware', #lyh
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

# Database configuration
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}

# Use PostgreSQL if DATABASE_URL environment variable is set
if 'DATABASE_URL' in os.environ:
    DATABASES['default'] = dj_database_url.parse(os.environ.get("DATABASE_URL"), conn_max_age=600)

# CORS configuration
CORS_ALLOW_ALL_ORIGINS = True
```

Key features:
- Includes required Django apps and third-party packages
- Configures middleware for handling requests
- Sets up database configuration with fallback to SQLite
- Supports PostgreSQL via environment variable
- Configures CORS to allow requests from any origin

## File Interactions

The files in the project interact in the following ways:

1. **URL Routing (urls.py)** → When a request comes in, Django matches the URL pattern and routes it to the appropriate view function in views.py.

2. **Views (views.py)** → The view function:
   - Receives the HTTP request
   - Uses serializers (from serializers.py) to validate and process data
   - Interacts with models (from models.py) to access the database
   - Returns a response

3. **Serializers (serializers.py)** → Serializers:
   - Convert between Django model instances and Python/JSON data structures
   - Validate incoming data
   - Implement creation and update logic

4. **Models (models.py)** → Models:
   - Define the database schema
   - Provide methods for interacting with the database
   - Implement business logic related to data integrity

5. **Settings (settings.py)** → The settings file configures the entire application, including installed apps, middleware, database connections, and more.

This layered architecture separates concerns and makes the code more maintainable and testable. 