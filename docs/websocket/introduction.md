---
sidebar_position: 1
---

# DriverConsumer Websocket

The DriverConsumer WebSocket handles real-time communication.

## Connecting to the WebSocket

To establish a connection to the WebSocket, connect to the following URL:

```Ruby
ws://your-websocket-server-url/driver/<user_id>
```

Replace <user_id> with the actual user ID of the passenger, the user must have STUDENT role in order to connect with the connection.

## Accepting a ride request

To create a new ride request, send a JSON message with the following structure:

```bash
{
  "action": "accept_ride_request",
  "ride_request_id": "<ride_request_id>",
}
```

Replace the placeholders with the actual values:

- ride_request_id : the ride request id

## Sending chat messages

To create a chat message, send a JSON message with the following structure:

```JSON
{
  "action": "send_chat_message",
  "message": "Hello, passenger!"
}
```

- message (string): message text to be sent

**NOTE:** send_chat_message is only available when driver accepts a ride request from passenger

## Receiving messages

DriverConsumer currently receiving **send_pending_ride_request** and **chat_message** from passenger and driver.

### chat_message

To the differentiate the message sent, user_id will be provided in the response.

**Example Response:**

```JSON
{
  "action": "chat_message",
  "user_id": "c18df757-d111-45c1-9b0b-45050cdef54c",
  "message": "hello passenger"
}
```

### send_pending_ride_request

The send_pending_ride_request response will be sent when a passenger creating new ride request and the messages received will be sent when the ride request is on "pending" status.

The send_pending_ride_request response will be in the following structure:

**Example Response:**

```JSON
{
  "action": "sending_pending_ride_request",
  "type": "send_pending_ride_request",
  "data": {
    "id": "9f7dbfa9-91d1-4639-b1f6-399d077ef791",
    "pickup_latitude": 37.7749,
    "pickup_longitude": -122.4194,
    "dropoff_latitude": 37.7874,
    "dropoff_longitude": -122.4082,
    "pickup_address": "123 Main St, San Francisco, CA",
    "dropoff_address": "456 Elm St, San Francisco, CA",
    "status": "pending"
  }
}
```

## Disconnecting

To disconnect from the WebSocket, simply close the connection from the client-side. This will automatically disconnect the user from the server.
