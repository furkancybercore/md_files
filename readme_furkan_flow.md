# Poker Application - User Flow and Process Documentation

This document explains the flow of the poker application from a user's perspective, detailing how various components interact to deliver the functionality. This is complementary to the technical documentation in `readme_furkan.md`.

## Application Overview

The Poker Backend is a Django REST API for managing online poker games. It allows users to:
- Register and log in as players
- Create and browse poker events
- Join existing events
- Track deposits, transactions, and game results

## User Journeys

### Player Registration and Authentication

1. **Player Registration**
   
   A new user starts by registering through the `/players/register/` endpoint, providing their email, nickname, and password.

   ```
   When a user registers, the system:
   1. Receives the registration data (email, nickname, password)
   2. Validates the data
   3. Securely hashes the password using Django's make_password function
   4. Generates a unique user ID
   5. Creates a Player object in the database
   6. Returns a success message
   ```

   As we can see in the registration view:
   ```python
   @api_view(["POST"])
   def register(request):
       """Registers a new player account."""
       serializer = serializers.PlayerRegisterSerializer(data=request.data)
       if serializer.is_valid():
           serializer.save()
           return Response({"message": "Player registered successfully"}, status=status.HTTP_201_CREATED)
       return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
   ```

2. **Player Login**
   
   After registration, the user can log in through the `/players/login/` endpoint, providing their email and password.

   ```
   When a user logs in, the system:
   1. Receives the login credentials (email, password)
   2. Looks up the player by email
   3. Verifies the password using Django's check_password function
   4. If successful, returns player details (nickname, email, balance, win points)
   5. If unsuccessful, returns an error message
   ```

   The login process is handled by this view:
   ```python
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

### Event Management

1. **Creating a Poker Event**
   
   A logged-in player (typically a host) can create a new poker event through the `/events/create` endpoint.

   ```
   When creating a poker event:
   1. The player submits event details (name, capacity, blinds, deposit, location, date, etc.)
   2. The system validates the data
   3. Creates an Event object in the database
   4. Associates the creator's Player object with the event
   5. Returns the created event details
   ```

   This is handled by the createEvent view:
   ```python
   @api_view(["POST"])
   def createEvent(request):
       """Creates a new game Event with specified configurations."""
       serializer = serializers.CreateEventSerializer(data=request.data)
       if serializer.is_valid():
           event = serializer.save()
           return Response({"message": "Event created successfully", "event": serializer.data}, status=status.HTTP_201_CREATED)
       return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
   ```

2. **Browsing Available Events**
   
   Any player can browse all available events through the `/events/` endpoint.

   ```
   When viewing all events:
   1. The system retrieves all Event objects from the database
   2. Formats them using the AllEventSerializer
   3. Returns the list of events with their details
   ```

   This is implemented in the getAllEvents view:
   ```python
   @api_view(["GET"])
   def getAllEvents(request):
       """Returns a list of all available game events."""
       events = Event.objects.all()
       serializer = serializers.AllEventSerializer(events, many=True)
       return Response(serializer.data, status=status.HTTP_200_OK)
   ```

3. **Viewing Detailed Event Information**
   
   Players can view detailed information about a specific event through the `/events/<event_id>/` endpoint.

   ```
   When viewing event details:
   1. The system retrieves the Event object by ID
   2. Calculates additional information like player count
   3. Retrieves the list of players who have joined
   4. Returns the comprehensive event information
   ```

   This functionality is provided by the getEventmDetails view:
   ```python
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

4. **Joining an Event**
   
   Players can join an available event through the `/events/join` endpoint.

   ```
   When a player joins an event:
   1. The player submits their user ID and the event ID
   2. The system verifies both the player and event exist
   3. Checks if the event has available capacity
   4. Checks if the player is already registered for the event
   5. Creates an EventPlayer record linking the player to the event
   6. Returns a success message
   ```

   This is handled by the joinEvent view:
   ```python
   @api_view(["POST"])
   def joinEvent(request):
       """Allows a player to join an existing Event."""
       serializer = serializers.JoinEventSerializer(data=request.data)
       if serializer.is_valid():
           serializer.save()
           return Response({"message": "Player joined the Event successfully"}, status=status.HTTP_201_CREATED)
       return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
   ```

## Data Flow in the Application

The application follows a typical Django REST Framework data flow:

1. **HTTP Request** → The client sends an HTTP request to a specific endpoint
2. **URL Routing** → Django's URL router maps the request to the appropriate view function
3. **View Processing** → The view function handles the request, often using serializers
4. **Serializer Validation** → Serializers validate input data and convert between model instances and Python datatypes
5. **Database Interaction** → Models interact with the database to retrieve or store data
6. **Response Serialization** → The results are serialized back into JSON
7. **HTTP Response** → The view returns an HTTP response to the client

## Detailed Process Flows

### Player Registration Flow

1. Client sends a POST request to `/players/register/` with email, nickname, and password
2. The URL router maps this to the `register` view function
3. The view creates a `PlayerRegisterSerializer` with the request data
4. The serializer validates the data (e.g., checks for required fields, email format)
5. If valid, the serializer's `create` method:
   - Extracts the password
   - Creates a Player instance
   - Hashes the password using `set_password` 
   - Saves the player to the database
6. The view returns a success message with status code 201 (Created)

### Event Creation Flow

1. Client sends a POST request to `/events/create` with event details, including creator_id
2. The URL router maps this to the `createEvent` view function
3. The view creates a `CreateEventSerializer` with the request data
4. The serializer validates the data 
5. If valid, the serializer's `create` method:
   - Extracts the creator_id
   - Finds the Player with that ID
   - Creates an Event instance with the creator and other details
   - Saves the event to the database
6. The view returns the event details with status code 201 (Created)

### Event Joining Flow

1. Client sends a POST request to `/events/join` with user_id and event_id
2. The URL router maps this to the `joinEvent` view function
3. The view creates a `JoinEventSerializer` with the request data
4. The serializer validates the data
5. If valid, the serializer's `create` method:
   - Finds the Player with the given user_id
   - Finds the Event with the given event_id
   - Checks if the event is full
   - Checks if the player is already in the event
   - Creates an EventPlayer instance linking the player and event
   - Saves the record to the database
6. The view returns a success message with status code 201 (Created)

## Database Relationships

The application uses several interrelated models to track poker events and player participation:

1. **Player → Event** (one-to-many): A player can create multiple events
2. **Player ↔ Event** (many-to-many, through EventPlayer): Players can join multiple events, and events can have multiple players
3. **Player → EventDeposit** (one-to-many): A player can make multiple deposits for different events
4. **Event → EventDeposit** (one-to-many): An event can have multiple deposits from different players
5. **Player ↔ Transaction** (one-to-many): Players can be involved in multiple transactions as either the source or target
6. **Event → GameSession** (one-to-many): An event can have multiple game sessions
7. **Player → GameSession** (one-to-many): A player can win multiple game sessions

These relationships allow the system to track all aspects of poker events, from creation to player participation, deposits, and game outcomes. 