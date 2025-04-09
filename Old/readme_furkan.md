# Poker Backend - Django REST API

**A comprehensive guide for Django REST Framework backend development**

![Django REST Framework Logo](https://www.django-rest-framework.org/img/logo.png)

## Table of Contents
- [Introduction](#introduction)
- [Project Setup](#project-setup)
- [Database Schema](#database-schema)
- [API Endpoints](#api-endpoints)
- [Authentication](#authentication)
- [Models](#models)
- [Serializers](#serializers)
- [Views](#views)
- [Additional Resources](#additional-resources)

## Introduction

This project is a Django REST Framework backend for a poker game application. It provides APIs for player registration, authentication, creating and managing poker events, and tracking game results. The backend is designed to serve as the foundation for a full-stack poker application, where players can create accounts, join poker events, and participate in games.

## Project Setup

### Creating a Django REST Framework Project

First, set up a virtual environment and install the required dependencies:

```bash
# Create a new directory for the project
mkdir poker-backend
cd poker-backend

# Create a virtual environment
python3 -m venv venv

# Activate the virtual environment
source venv/bin/activate

# Install Django and Django REST Framework
pip install django djangorestframework psycopg2 whitenoise dj-database-url django-cors-headers

# Create a new Django project
django-admin startproject api backend

# Create the core app
cd backend
python manage.py startapp app_core

# Create a specific module for poker functionality
mkdir -p app_core/poker_event
touch app_core/poker_event/__init__.py
touch app_core/poker_event/models.py
touch app_core/poker_event/serializers.py
touch app_core/poker_event/views.py
```

### Project Configuration

Update the `settings.py` file to include the installed apps and configure the database:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework',  # Django REST Framework
    'psycopg2',        # PostgreSQL adapter
    'corsheaders',     # CORS handling
    'app_core',        # Core application
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'whitenoise.middleware.WhiteNoiseMiddleware',  # For serving static files
    'corsheaders.middleware.CorsMiddleware',       # For handling CORS
    # ... other middleware
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

## Database Schema

The database schema consists of several interconnected models that represent players, events, and game-related data:

### Players
Stores user information, credentials, and balances.

### Events
Represents poker game events with details like blinds, location, and capacity.

### EventPlayers
Tracks which players have joined which events.

### EventDeposits
Records deposits made by players to participate in events.

### Transactions
Logs money movements between players.

### GameSessions and GameSessionResults
Stores completed games and their outcomes.

## API Endpoints

The application provides the following API endpoints:

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/admin/` | GET | Django admin panel for managing the application |
| `/players/register/` | POST | Registers a new player account |
| `/players/login/` | POST | Authenticates and logs in an existing player |
| `/players/` | GET | Retrieves a list of all registered players |
| `/events/create` | POST | Creates a new game event |
| `/events/` | GET | Retrieves all available game events |
| `/events/join` | POST | Allows a player to join an existing event |
| `/events/<int:event_id>/` | GET | Retrieves detailed information about a specific game event |

## Authentication

The application uses a simple authentication system where:

1. Players register with an email, nickname, and password
2. Passwords are securely hashed using Django's built-in password hashing utilities
3. Players can login with their email and password
4. The system validates their credentials and returns player information

```python
# Example of registration
@api_view(["POST"])
def register(request):
    """Registers a new player account."""
    serializer = serializers.PlayerRegisterSerializer(data=request.data)
    if serializer.is_valid():
        serializer.save()
        return Response({"message": "Player registered successfully"}, status=status.HTTP_201_CREATED)
    return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

# Example of login
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

## Models

The application uses several interconnected Django models to represent the different entities in the system.

### Player Model

The `Player` model represents a user of the application:

```python
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
- Generates a unique user ID for each player
- Securely hashes passwords
- Tracks balance and win points
- Timestamps when accounts are created

### Event Model

The `Event` model represents a poker game event:

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
- References the creator (a Player)
- Defines game parameters like blinds and required deposit
- Includes scheduling information (date, time, duration)
- Supports recurring events through the regularity field

### Related Models

Several other models handle the relationships between players and events:

- `EventPlayer`: Tracks which players join which events
- `EventDeposit`: Records deposits made by players for events
- `Transaction`: Logs chip movements between players
- `GameSession` and `GameSessionResult`: Record completed games and their outcomes

## Serializers

The application uses serializers to convert model instances to JSON and for validation of incoming data.

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

## Views

The views handle API requests and return appropriate responses.

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

## Additional Resources

- [Django Documentation](https://docs.djangoproject.com/)
- [Django REST Framework Documentation](https://www.django-rest-framework.org/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/) 