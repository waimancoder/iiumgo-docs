---
sidebar_position: 2
---

# Passenger Consumer Websocket

The WebSocket implemented in the **PassengerConsumer** class expects requests to be in the form of JSON messages containing an action field that specifies the action to be performed. The following are the possible actions:

- [`create_ride_request`](#1): creates a new ride request and returns the details of the created request as a response.
- [`send_chat_message`](#2): sends a chat message to a driver or passenger.
- [`cancel_ride_request`](#3): cancels a ride request and returns a response.

## Connecting to the Websocket

To establish a connection to the WebSocket, connect to the following URL:

```Ruby
wss://unigo.ltd/ws/passenger/<user_id>
```

Replace <user_id> with the actual user ID of the passenger.

When a passenger connects to this websocket, the PassengerConsumer class is invoked. The user's ID is extracted from the WebSocket URL and checked if it exists in the database. If the user ID is valid, the passenger_status is checked. If the status is `accepted` or `in_progress`, the user is added to a channel layer group with a corresponding cache key.

When a passenger connects to the WebSocket, if their passenger status is not available (i.e. it is neither `accepted`, `pending` nor `in_progress`), the WebSocket will send a response indicating their status as `accepted`, `pending` nor `in_progress` along with details of their most recent ride request.

### Passenger Status : Accepted

If the passenger_status is `accepted`, the latest accepted RideRequest is retrieved from the database and sent to the user in a JSON format, along with the details of the assigned driver.

#### Response

```bash
{
  "type": "passenger_status",
  "data": {
    "passenger_status": "accepted",
    "ride_request_info": {
      "id": "eef54ceb-XXXX-4d1e-a622-43e244583cab",
      "pickup_latitude": 3.2598975,
      "pickup_longitude": 101.7309086,
      "polyline": "1224124132",
      "pickup_address": "ABBB",
      "dropoff_address": "ABBB",
      "dropoff_latitude": 3.2310537,
      "dropoff_longitude": 101.7242156,
      "vehicle_type": "6pax",
      "price": 9.5,
      "distance": 2.35,
      "special_requests": "hello",
      "status": "accepted",
      "created_at": "2023-04-12T18:02:19.661108+00:00"
    },
    "driver_info": {
      "id": "1a21ff30-XXXX-4d5b-9a02-95b723c90700",
      "driver_name": "XXXXXXXXX",
      "driver_phone_number": "01XXXXXX"
    }
  }
}
```

### Passenger Status : In Progress

If the passenger_status is `in_progress`, the latest accepted RideRequest is retrieved from the database, and the details of the assigned driver are sent to the user in a JSON format.

#### Response

```bash
{
  "type": "passenger_status",
  "data": {
    "passenger_status": "in_progress",
    "ride_request_info": {
      "id": "eef54ceb-XXXX-4d1e-a622-43e244583cab",
      "pickup_latitude": 3.2598975,
      "pickup_longitude": 101.7309086,
      "polyline": "1224124132",
      "pickup_address": "ABBB",
      "dropoff_address": "ABBB",
      "dropoff_latitude": 3.2310537,
      "dropoff_longitude": 101.7242156,
      "vehicle_type": "6pax",
      "price": 9.5,
      "distance": 2.35,
      "special_requests": "hello",
      "status": "in_progress",
      "created_at": "2023-04-12T18:02:19.661108+00:00"
    },
    "driver_info": {
      "id": "1a21ff30-XXXX-4d5b-9a02-95b723c90700",
      "driver_name": "XXXXXXXXX",
      "driver_phone_number": "01XXXXXX"
    }
  }
}
```

### Passenger Status : Pending

If the passenger_status is `pending`, the latest pending RideRequest is retrieved from the database, and the user is sent a JSON object containing the ride_request_info for that request.

#### Response

```bash
{
  "type": "passenger_status",
  "data": {
    "passenger_status": "pending",
    "ride_request_info": {
      "id": "eef54ceb-XXXX-4d1e-a622-43e244583cab",
      "pickup_latitude": 3.2598975,
      "pickup_longitude": 101.7309086,
      "polyline": "1224124132",
      "pickup_address": "ABBB",
      "dropoff_address": "ABBB",
      "dropoff_latitude": 3.2310537,
      "dropoff_longitude": 101.7242156,
      "vehicle_type": "6pax",
      "price": 9.5,
      "distance": 2.35,
      "special_requests": "hello",
      "status": "pending",
      "created_at": "2023-04-12T18:02:19.661108+00:00"
    }
  }
}
```

## Requests

The expected JSON message format for each action is as follows:

### create_ride_request {#1}

For the `create_ride_request action`, the expected JSON message format should contain the following fields:

- action: A string that specifies the action as "create_ride_request".
- pickup_latitude: A float that represents the latitude of the pickup location.
- pickup_longitude: A float that represents the longitude of the pickup location.
- dropoff_latitude: A float that represents the latitude of the dropoff location.
- dropoff_longitude: A float that represents the longitude of the dropoff location.
- pickup_address: A string that represents the address of the pickup location.
- dropoff_address: A string that represents the address of the dropoff location.
- polyline: A string that represents the encoded polyline of the route.
- price: A float that represents the estimated price of the ride.
- vehicle_type: A string that represents the type of vehicle requested.

```bash
{
    "action": "create_ride_request",
    "pickup_latitude": "3.2598975",
    "pickup_longitude": "101.7309086",
    "dropoff_latitude": "3.2310537",
    "dropoff_longitude": "101.7242156",
    "polyline" : "1224124132",
    "price": 9.50,
    "pickup_address": "ABBB",
    "dropoff_address": "ABBB",
    "vehicle_type": "6pax",
}
```

#### Success Response

```bash
{
  "success": true,
  "message": "Ride request created successfully",
  "type": "passenger_created_ride_request",
  "data": {
    "id": "d530a710-17b5-42f8-94c5-0cd1877841fe",
    "pickup_latitude": "3.2598975",
    "pickup_longitude": "101.7309086",
    "polyline": "1224124132",
    "dropoff_latitude": "3.2310537",
    "dropoff_longitude": "101.7242156",
    "pickup_address": "ABBB",
    "dropoff_address": "ABBB",
    "status": "pending",
    "price": 9.5,
    "distance": 7.438,
    "vehicle_type": "6pax",
    "created_at": "2023-04-11T20:03:35.364513+00:00"
    }
}
```

#### Failed Response

```bash
{
    "success": false,
    "message": "Error Message"
}
```

### send_chat_message{#2}

For the `send_chat_message` action, the expected JSON message format should contain the following fields:

- action: A string that specifies the action as "send_chat_message".
- message : A string that contains the message user wants to send.

```bash
{
    "action" : "send_chat_message",
    "message" : "Hello World"
}
```

#### Success Response

```bash
{
    "type": "chat_message"
    "user_id": <sender user_id>,
    "message" : "Hello World",
    "time": "2023-04-12T18:07:18.428848"
}

```

#### Failed Response

```bash
{
    "success" : false,
    "message" : "Error Message"
}
```

### cancel_ride_request{#3}

For the `cancel_ride_request` action, the expected JSON message format should contain the following fields:

- action: A string that specifies the action as "cancel_ride_request".
- ride_request_id : A string that contains the UUID of the ride request.

#### Request

```bash
{
    "action" : "cancel_ride_request",
    "ride_request_id" : "b70c8c7b-ee43-4c9e-bf33-8f46a2c356e1"
}
```

#### Success Response

```bash
{
    "success": True,
    "message": "Ride request cancelled successfully",
    "type": "passenger_cancelled_ride_request",
    "data": {
        "id": "b70c8c7b-ee43-4c9e-bf33-8f46a2c356e1",
        "pickup_latitude": 37.7749,
        "pickup_longitude": -122.4194,
        "polyline": "q}p~Hifa_@?k@J{E~@kA",
        "dropoff_latitude": 37.7833,
        "dropoff_longitude": -122.4167,
        "pickup_address": "123 Main St, San Francisco, CA 94105",
        "dropoff_address": "456 Pine St, San Francisco, CA 94105",
        "status": "cancelled",
        "price": 10.5,
        "distance": 2.1,
        "vehicle_type": "4pax",
        "created_at": "2023-04-13T10:30:00.000000Z",
        "details": "I need a car seat for my child"
    }
}
```

#### Failed Response

```bash
{
    "success" : false,
    "message" : "Error Message"
}
```

## Incoming responses without request

When connecting to a WebSocket, it is possible to receive incoming responses without sending a request first. These responses are usually initiated by the server and are sent to the client as soon as there is an update or change in the system. In this context, incoming responses without requests are important for real-time communication and can be used to keep the client-side application updated with the latest information from the server.

### type: driver_cancelled_ride_request

`driver_cancelled_ride_request`: This response indicates that the driver has cancelled a ride request from a passenger. The `id` field in the data object identifies the cancelled ride request.

```bash
{
  "success": true,
  "message": "Ride request cancelled successfully",
  "type": "driver_cancelled_ride_request",
  "data": {
    "id": "a4e33acd-6409-4786-a243-a75460b538c5"
  }
}
```

### type: driver_passenger_ride_request_accepted

`driver_passenger_ride_request_accepted`: This response indicates that a ride request from a passenger has been accepted by a driver. The `driver_info` object provides details about the driver, such as their name, vehicle type, and registration number. The `ride_request_info` object provides details about the ride, such as pickup and dropoff locations, distance, and price. The `passenger_info` object provides details about the passenger, such as their name and phone number.

```bash
{
  "success": true,
  "message": "Ride request accepted successfully",
  "type": "driver_passenger_ride_request_accepted",
  "data": {
    "driver_info": {
      "driver_id": "3509ff9e-b3da-4da0-8357-995b8c7f54ad",
      "driver_name": "Muhammad Haziq bin Mohamad",
      "vehicle_registration_number": "",
      "vehicle_manufacturer": "",
      "vehicle_model": "",
      "vehicle_color": "",
      "vehicle_type": "6pax"
    },
    "ride_request_info": {
      "pickup_latitude": 3.2598975,
      "pickup_longitude": 101.7309086,
      "dropoff_latitude": 3.2310537,
      "dropoff_longitude": 101.7242156,
      "pickup_address": "ABBB",
      "dropoff_address": "ABBB",
      "status": "accepted",
      "polyline": "1224124132",
      "price": 9.5,
      "distance": 2.35,
      "vehicle_type": "6pax",
      "created_at": "2023-04-14T14:50:53.797373+00:00",
      "details": "hello"
    },
    "passenger_info": {
      "passenger_id": "647ef7ea-1924-4e71-869c-0253671c7865",
      "passenger_name": "wan muhammad fakhruddin aiman",
      "passenger_phone_number": "0172240296",
      "passenger_gender": ""
    }
  }
}
```

### type: driver_start_trip

This response indicates that a driver has started a ride with a passenger. The `id` field in the data object identifies the ride that has started, and the `status` field indicates that the ride is now in progress.

```bash
{
  "success": true,
  "type": "driver_start_trip",
  "message": "Ride request starts successfully",
  "data": {
    "id": "c20871ad-2fe3-4b8f-bb0a-fa8a3d3bcbeb",
    "status": "in_progress"
  }
}
```

### type: driver_passenger_notification

This response indicates that a ride request has been completed successfully. The `id` field in the data object identifies the ride that has been completed.

```bash
{
  "success": true,
  "message": "Ride Request is completed successfully",
  "type": "driver_passenger_notification",
  "data": {
    "id": "c20871ad-2fe3-4b8f-bb0a-fa8a3d3bcbeb"
  }
}
```
