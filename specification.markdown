# Common Protocol for Programming Physical Input and Output API Design

This document outlines the specification for the CPPP-IO API, as implemented in [Artoo][], [Cylon][], and [Gobot][].

[Artoo]: http://artoo.io
[Cylon]: http://cylonjs.com
[Gobot]: http://gobot.io

## All Responses

All API responses will, unless otherwise specified, be returned in JSON format, with a **Content-Type: application/json** header.

All JSON responses will be objects, with keys corresponding to the content.
For example, when fetching an index of all robots, the response might look like the following:

    {
      robots: [{Robot}, {Robot}]
    }

And when fetching an individual robot, an example response might look like this:

    {
      robot: {Robot}
    }

## Namespacing

All API routes will be namespaced under the `/api` route prefix.
This is to make them distinct from other media types which may be served by the same router.

## Routing

Routing is the mechanism that allows an API client system to find the path to a Robot or any
Device belonging to a Robot. We use the same style of routing paths in Cylon.js regardless if you are using REST, Socket.io, or MQTT. 

## Versioning

With no **Accept** header supplied, the API will default to the latest version, and JSON responses.
If, for example, an end user wanted to request a JSON response using the first version of the API, they would supply a header such as **Accept: application/vnd.cpp-io.v1+json**.

## Content Types

The requested Content-Type type should be specified in the Accept HTTP Header.
Optionally, however, a Content-Type suffix can be cast to the end of the URI.
However, the Accept header should always have precedence.

For example, a request to http://localhost:3000/api/robots.xml with the header **Accept: application/vnd.cpp-io.v1+json** (or just **application/json**) would return JSON, not XML.

However, a browser making a request to the same URI without a formatted Accept header would return an XML response, if possible.

By default, with no other indicators, the API should serve a JSON response.
If a requested type is not possible to serve, the API should respond with a 406 Not Acceptable error.

## Command Casing

Command and event names should be snake_cased at all times, both when serialized to be sent down to the client, or when received in a URL.

e.g.

    ["publish_event", "send_notification"]

## Authorization

Where applicable, HTTP basic auth params are to be sent Base64 encoded in the Authorization header.
For example, with a username of ‘user’ and a password of ‘pass’, the header would be **Authorization: Basic dXNlcjpwYXNz**.

## Command Execution Response

When executing commands, the API should return a response containing the result.

    // POST /api/robots/TestBot/commands/hello
    // { greeting: ‘world’ }
    {
      "result": "Hello, world"
    }

## Error Response

When an error occurs, the API should attempt to respond with an error message describing what went wrong.

An example error format might look like the following:

    // GET /api/robots/NonExistentBot
    {
      "error": "No Robot found with the name NonExistentBot"
    }

## Params

When parsing params submitted by a client, the server should examine the request body, looking for JSON-formatted params.
If params are present in the request body, they should be used.

Params order should be determined by their order in the request.

So, if the values are coerced to an array, this request would yield the subsequent params array:

    // POST https://localhost:3000/api/commands/echo
    // { "a": "alpha", "b": false }

    // after coercion:
    [ "alpha", false ]

## Routes

Routes will be described below, in a format similar to the following:

- /route/to/endpoint
- arguments (if any)
- description
- example response

### GET /api

Returns an object containing info on the MCP. The MCP response should contain:

- an array of the robots
- an array of commands and events the MCP supports (represented as strings)

Example response:

    // GET /api
    {
      "MCP": {
        "robots": [
          {
            "name": "TestBot",

            "connections": [
              {
                "name": "loopback",
                "adaptor": "Loopback",
                "details": {
                  "port": "/dev/null",
                  "test": "abc"
                }
              }
            ],

            "devices": [
              {
                "name": "ping",
                "driver": "Ping",
                "connection": "loopback",
                "commands": ["ping"],
                "events": ["ping"],
                "details": {
                  "pin": "13",
                  "test": "abc"
                }
              }
            ],

            "commands": [ "hello" ],
            "events": [ "hello" ]
          }
        ],

        "commands": [ "echo" ],
        "events": [ "robot_added", "robot_removed" ]
      }
    }

### GET /api/commands

Returns an array with the specified MCP’s commands in string format.

Example response:

    // GET /api/commands
    {
      "commands": [ "echo" ]
    }

### GET /api/events

Returns an array with the specified MCP’s events in string format.

Example response:

    // GET /api/events
    {
      "events": [ "robot_added", "robot_removed" ]
    }

### POST /api/commands/:command

Executes the requested command on the MCP.
Params will be provided to the method in the order they’re passed (either in body or query params).

Returns the result of running the command.

Example response:

    // POST /api/commands/echo
    // { "e": 10 }
    {
      "result": 10
    }

### GET /api/events/:event

Opens a Server-Sent Events stream that hooks into the provided Event on the MCP.
When the MCP emits the event, the data emitted with the event is sent down the wire to the client.

Example response:

    // GET /api/events/robot_removed
    data: {
      "name": "TempBot"
    }

### GET /api/robots

Returns an array of all Robots the server knows about.
A Robot response should contain:

- it’s name
- an array of the robot’s connections (represented as objects)
- an array of the robot’s devices (represented as objects)
- an array of commands the robot supports (represented as strings)

Example response:

    // GET /api/robots
    {
      "robots": [
        {
          "name": "TestBot",

          "connections": [
            {
              "name": "loopback",
              "adaptor": "Loopback",
              "details": {
                "port": "/dev/null",
                "test": "abc"
              }
            }
          ],

          "devices": [
            {
              "name": "ping",
              "driver": "Ping",
              "connection": "loopback",
              "commands": ["ping"],
              "events": ["ping"],
              "details": {
                "pin": "13",
                "test": "abc"
              }
            }
          ],

          "commands": [ "hello" ],
          "events": [ "hello" ]
        }
      ]
    }

### GET /api/robots/:robot

Returns an object containing info on the named Robot.
The Robot response should contain:

- an array of the robot’s connections (represented as objects)
- an array of the robot’s devices (represented as objects)
- an array of commands the robot supports (represented as strings)

Example response:

    // GET /api/robots/TestBot
    {
      "name": "TestBot",

      "connections": [
        {
          "name": "loopback",
          "adaptor": "Loopback",
          "details": {
            "port": "/dev/null",
            "test": "abc"
          }
        }
      ],

      "devices": [
        {
          "name": "ping",
          "driver": "Ping",
          "connection": "loopback",
          "commands": ["ping"],
          "events": ["ping"],
          "details": {
            "pin": "13",
            "test": "abc"
          }
        }
      ],

      "commands": [ "hello" ],
      "events": [ "hello" ]
    }

### GET /api/robots/:robot/commands

Returns an array with the specified Robot’s commands in string format.

Example response:

    // GET /api/robots/TestBot/commands
    {
      "commands": [ "hello" ]
    }

### GET /api/robots/:robot/events

Returns an array with the specified Robot’s events in string format.

Example response:

    // GET /api/robots/TestBot/events
    {
      "events": [ "hello" ]
    }

### POST /api/robots/:robot/commands/:command

Executes the requested command on the robot.
Params will be provided to the method in the order they’re passed (either in body or query params).

Returns the result of running the command.

Example response:

    // POST /api/robots/TestBot/commands/hello?greeting=world
    {
      “result”: “Hello, world”
    }

### GET /api/robots/:robot/events/:event

Opens a Server-Sent Events stream that hooks into the provided Event on the robot.
When the MCP emits the event, the data emitted with the event is sent down the wire to the client.

Example response:

    // GET /api/robots/TestBot/events/hello
    data: hi there

### GET /api/robots/:robot/devices

Returns an array containing information about the specified Robot’s Devices.

Example response:

    // GET /api/robots/TestBot/devices
    {
      "devices": [
        {
          "name": "ping",
          "driver": "Ping",
          "connection": "loopback",
          "commands": ["ping"],
          "events": ["ping"],
          "details": {
            "pin": "13",
            "test": "abc"
          }
        }
      ]
    }

### GET /api/robots/:robot/devices/:device

Returns an object containing information about the specified Device on the Robot.

Example response:

    // GET /api/robots/TestBot/devices/ping
    {
      "device": {
        "name": "ping",
        "driver": "Ping",
        "connection": "loopback",
        "commands": [ "ping" ],
        "events": ["ping"],
        "details": {
          "pin": "13",
          "test": "abc"
        }
      }
    }

### GET /api/robots/:robot/devices/:device/commands

Returns an array of commands the specified Device has.

Example response:

    // GET /api/robots/TestBot/devices/ping/commands
    {
      "commands": [ "ping" ]
    }

### GET /api/robots/:robot/devices/:device/events

Returns an array with the specified Device’s events in string format.

Example response:

    // GET /api/robots/TestBot/devices/ping/events
    {
      "events": [ "ping" ]
    }

### POST /api/robots/:robot/devices/:device/commands/:command

Executes the requested command on the device.
Params will be provided to the method in the order they’re passed (either in body or query params).

Returns the result of running the command.

Example response:

    // POST /api/robots/TestBot/devices/ping/commands/ping
    {
      “result”: "pong"
    }

### GET /api/robots/:robot/devices/:device/events/:event

Opens a Server-Sent Events stream that hooks into the provided Event on the specified device.
When the device emits the event, the data emitted with the event is sent down the wire to the client.

Example response:

    // GET /api/robots/TestBot/devices/ping/events/ping
    data: ping

    data: ping

### GET /api/robots/:robot/connections

Returns an array containing information about the specified Robot’s Connections.

Example response:

    // GET /api/robots/TestBot/connections
    {
      "connections": [
        {
          "name": "loopback",
          "adaptor": "Loopback",
          "details": {
            "port": "/dev/null",
            "test": "abc"
          }
        }
      ]
    }

### GET /api/robots/:robot/connections/:connection

Returns an object containing information about the specified Connection on the Robot.

Example response:

    // GET /api/robots/TestBot/connections/loopback
    {
      "connection": {
        "name": "loopback",
        "adaptor": "Loopback",
        "details": {
          "port": "/dev/null",
          "test": "abc"
        }
      }
    }
