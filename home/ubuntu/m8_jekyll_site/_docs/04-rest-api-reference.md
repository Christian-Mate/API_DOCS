---
layout: default
title: REST API Reference
nav_order: 4
---

# REST API Reference

This section provides a detailed reference for the REST API endpoints available on the M8-TW-ULTRA controller. These endpoints offer HTTP-based methods for a wide range of operations, including command execution, device polling, statistics retrieval, file management, and more.

## Base URL and Authentication Recap

Most REST API endpoints are relative to the base URL: `http://<IP_ADDRESS_OR_HOSTNAME>/api/v0/`. Remember to replace `<IP_ADDRESS_OR_HOSTNAME>` with the actual IP address or hostname of your controller. File transfer endpoints (`/upload` and `/download`) use a different base path: `http://<IP_ADDRESS_OR_HOSTNAME>/`.

Authentication is required for most endpoints and uses HTTP Basic Authentication. The `Authorization` header must be included in requests, formatted as `BASIC <credentials>`, where `<credentials>` is the Base64 encoding of `apikey:<your_api_key>`.

## Command Execution (`/command`)

This endpoint allows for the execution of server-side commands, mirroring the functionality available through the Serial or Telnet interface. It is a versatile endpoint for controlling various aspects of the device programmatically.

**Method:** POST
**Endpoint:** `/api/v0/command`
**Authentication:** Required.
**Headers:**
`Authorization: BASIC <base64_encoded_credentials>`
`Content-Type: application/json`
**Request Body:** A JSON object containing the command to be executed.
```json
{
  "command": "<full_command_string>"
}
```
Replace `<full_command_string>` with the actual command and its arguments as you would type them in the serial/Telnet console (e.g., `set cct x 3000 200`, `show start 1`, `set save`).

**Response (200 OK):** A JSON object confirming the command execution or reporting an error.
Successful execution might return an empty JSON object or a simple status message.
```json
{}
```
Or, for commands that produce output:
```json
{
  "response": "<command_output_if_any>"
}
```
If the command is invalid or encounters an error during execution:
```json
{
  "error": "Invalid command: <details>"
}
```

**Example (using curl):** Save configuration changes.
```bash
curl -X POST "http://<IP_ADDRESS_OR_HOSTNAME>/api/v0/command" \
-H "Authorization: BASIC <your_base64_token>" \
-H "Content-Type: application/json" \
-d '{"command": "set save"}'
```

## Device Polling (`/m8poll`)

This endpoint provides a comprehensive, real-time snapshot of the M8-TW-ULTRA controller’s status. This includes firmware details, network time, operational parameters, temperatures, alert status, and port/channel states. It is the primary endpoint for monitoring device health and current operational state.

**Method:** GET
**Endpoint:** `/api/v0/m8poll`
**Authentication:** Required.
**Headers:**
`Authorization: BASIC <base64_encoded_credentials>`
**Response (200 OK):** A JSON object containing the device status. The structure includes top-level fields for general device information and nested objects for port and channel details.

**Top-Level Response Fields:**
`version` (integer): Firmware version number.
`build` (integer): Firmware build number.
`ntp` (integer): NTP synchronization status (1 = synced, 0 = not synced).
`time` (string): Current device time in ISO 8601 format (UTC or with offset).
`mac` (string): MAC address of the device’s Ethernet interface.
`shows` (array): Array of integers representing currently active show numbers.
`powerSource` (string): Indicates the power source ("poe" or "external").
`voltage` (float): System input voltage (in Volts).
`current` (float): System input current (in Amps).
`controlTemp` (float): Temperature of the control board (in °C).
`powerTemp` (float): Temperature of the power board (in °C).
`broadcastGroup` (integer): Assigned keypad broadcast group (0 means disabled).
`alerts` (integer): Current 32-bit alert bitfield value.
`led` (string): Current state or color of the status LED (e.g., “blue”).
`port` (array): Array of port status objects (one object per port, 0 and 1).

**`port` Object Fields:**
`voltage` (float): Measured supply voltage for this port (in Volts).
`state` (integer): Port operational state code (device-specific meaning).
`status` (integer): Port error or warning status code (device-specific meaning).
`message` (string): Human-readable status message for this port (e.g., “autocal success”).
`channels` (array): Array of channel status objects (two objects per port).
`profile` (string): Name of the lighting profile currently active on this port.
`quantity` (integer): Fixture quantity scaling factor configured for this port.
`cct` (integer): Active Color Correlated Temperature (CCT) in Kelvin (0 if not set or applicable).
`brightness` (integer): Overall brightness level for the port (0–255).

**`channels` Object Fields (within `port` array):**
`current` (float): Measured current draw for this channel (in Amps).
`set` (float): Target current setpoint for this channel (in Amps).
`max` (float): Maximum capable current for this channel (in Amps).
`brightness` (integer): Individual channel brightness level (0–255).
`status` (integer): Channel-specific status code (device-specific).

**Example (using curl):**
```bash
curl -X GET "http://<IP_ADDRESS_OR_HOSTNAME>/api/v0/m8poll" \
-H "Authorization: BASIC <your_base64_token>"
```

An example `/m8poll` response might look like this:
```json
{
  "version": 87,
  "build": 27,
  "ntp": 1,
  "time": "2025-04-24T15:17:15-05:00",
  "mac": "fc:c2:3d:59:ab:70",
  "shows": [],
  "powerSource": "poe",
  "voltage": 53.9,
  "current": 0.059,
  "controlTemp": 39.5,
  "powerTemp": 41.3,
  "broadcastGroup": 0,
  "alerts": 0,
  "led": "blue",
  "port": [
    {
      "voltage": 30.5,
      "state": 0,
      "status": 0,
      "message": "autocal success",
      "channels": [
        { "current": 0.000, "set": 0.580, "max": 0.667, "brightness": 255, "status": 0 },
        { "current": 0.000, "set": 0.606, "max": 0.697, "brightness": 255, "status": 0 }
      ],
      "profile": "STAKS 2X2 0423251642",
      "quantity": 1,
      "cct": 0,
      "brightness": 0
    },
    { }
  ]
}
```

## Statistics (`/stats`, `/stats/new`)

These endpoints provide access to historical and real-time energy usage and keypad interaction statistics collected by the controller.

**Retrieve All Historical Statistics (`/stats`)**

This endpoint fetches the complete recorded statistics dataset, including all historical daily and hourly intervals.

**Method:** GET
**Endpoint:** `/api/v0/stats`
**Authentication:** Required.
**Headers:**
`Authorization: BASIC <base64_encoded_credentials>`
**Response (200 OK):** A JSON object containing the statistics data. The `data` array is not guaranteed to be sorted chronologically.

**Retrieve Live/In-Progress Statistics (`/stats/new`)**

This endpoint fetches only the currently accumulating daily and hourly statistics records. This is optimized for real-time dashboard updates or monitoring current energy usage patterns.

**Method:** GET
**Endpoint:** `/api/v0/stats/new`
**Authentication:** Required.
**Headers:**
`Authorization: BASIC <base64_encoded_credentials>`
**Response (200 OK):** A JSON object with the same structure as `/stats`, but containing only the active interval records.

**Statistics Response Structure:**
```json
{
  "type": "statistics",
  "energyUnits": "W-hr",
  "data": [
    {
      "ts": 1734411600,
      "int": 86400,
      "daily": 1,
      "energy": {
        "input": 25.584,
        "port1": 0.113,
        "port2": 0.226
      },
      "keypress": [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
    }
  ]
}
```
The `ts` field is a Unix epoch timestamp (UTC) marking the start of the interval. `int` is the interval duration in seconds (3600 for hourly, 86400 for daily). `daily` is a flag (1 for daily, 0 for hourly). The `energy` object contains consumption data for device input and each port, in Watt-hours. `keypress` is an array counting keypresses per category.

## System Log (`/log`)

This endpoint retrieves recent system log entries. These logs primarily focus on significant events like alert state changes, system startups, and potentially other diagnostic messages.

**Method:** GET
**Endpoint:** `/api/v0/log`
**Authentication:** Required.
**Headers:**
`Authorization: BASIC <base64_encoded_credentials>`
**Response (200 OK):** A JSON array of log entry objects, ordered chronologically (oldest first).

**Log Entry Object Fields:**
`time` (integer): Unix Epoch seconds, UTC.
`event` (integer): Event Number.
`alert` (integer): Alert Number.
`content` (string): Human-readable message describing the log event.

**Example `/log` Response:**
```json
[
  {
    "time": 1746567829,
    "event": 1,
    "alert": 0,
    "content": ""
  },
  {
    "time": 1746567857,
    "event": 4,
    "alert": 0,
    "content": "P0,start autocal (skip detect)"
  }
]
```

## Keypad Configuration (`/keypad`)

These endpoints are for retrieving and updating the configuration that maps keypad buttons and MultiPort I/O pins (identified by `keycode`) to specific functions and parameters.

**Retrieve Current Keypad Configuration (GET `/keypad`)**

Fetches the entire current keypad configuration as a JSON array.

**Method:** GET
**Endpoint:** `/api/v0/keypad`
**Authentication:** Required.
**Headers:**
`Authorization: BASIC <base64_encoded_credentials>`
**Response (200 OK):** A JSON array where each object represents the configuration for a single `keycode`.

**Update Keypad Configuration (POST `/keypad`)**

This endpoint overwrites the entire existing keypad configuration with the provided JSON array. To modify a single button, you must first GET the current configuration, modify the relevant object(s) in the array, and then POST the complete, updated array.

**Method:** POST
**Endpoint:** `/api/v0/keypad`
**Authentication:** Required.
**Headers:**
`Authorization: BASIC <base64_encoded_credentials>`
`Content-Type: application/json`
**Request Body:** A JSON array containing keypad configuration objects. **Important Field Order Requirement:** The fields within each configuration object must be listed in the exact order: `keycode`, `function`, `parameters`. Failure to adhere to this order may cause the request to be rejected or processed incorrectly.

**Response (200 OK):** Indicates successful update.

**Keypad Configuration Object Structure:**
`keycode` (integer, required): Unique identifier for the button or I/O pin.
`function` (string, required): The name of the function to assign (e.g., `INPUT_SHOW`, `OUTPUT_ALERT`).
`parameters` (object, required): An object containing parameters specific to the chosen function.

**Example POST `/keypad` Request Body:**
```json
[
  {
    "keycode": 2,
    "function": "INPUT_SHOW",
    "parameters": {
      "show": 1,
      "stats": 1,
      "dim": -3,
      "repeat": 75
    }
  },
  {
    "keycode": 32,
    "function": "INPUT_ALERT",
    "parameters": {
      "alerts": 256,
      "set": 0,
      "clear": 1
    }
  }
]
```
Refer to Section 6 for a detailed explanation of functions and parameters.

## File Management (`/files`, `/upload`, `/download`)

Endpoints for managing files stored on the M8-TW-ULTRA, primarily `.m8` show files and `.pro` profile files.

**List Available Files (GET `/files`)**

Retrieves a list of files currently stored on the device, along with storage usage information.

**Method:** GET
**Endpoint:** `/api/v0/files`
**Authentication:** Required.
**Headers:**
`Authorization: BASIC <base64_encoded_credentials>`
**Response (200 OK):** A JSON object containing storage details (`free` and `used` space in bytes) and a list of file objects, each with `name` and `size`.
Example response:
```json
{
  "free": 8101888,
  "used": 90112,
  "files": [
    {
      "name": "config.txt",
      "size": 747
    },
    {
      "name": "ultra.log",
      "size": 835
    }
  ]
}
```

**Upload a File (POST `/upload`)**

Uploads a binary file to the device’s storage, verifying integrity using a SHA-256 hash.

**Method:** POST
**Endpoint:** `/upload?filename=<filename>&hash=<sha256_hash>` (Note: Not under `/api/v0/`)
**Authentication:** Required.
**Query Parameters:**
`filename` (string): The exact target filename on the device (e.g., `p1_001_a.m8`). Must adhere to naming conventions.
`hash` (string): The SHA-256 hash (lowercase hexadecimal string) of the file content being uploaded.
**Headers:**
`Authorization: BASIC <base64_encoded_credentials>`
`Content-Type: application/octet-stream`
**Request Body:** The raw binary content of the file.
**Response:** `200 OK` indicates success. Errors (e.g., 400, 401, 500) indicate issues like incorrect hash, insufficient storage, or invalid filename.

**Example (using curl):** Upload `my_show.m8` as `p1_001_a.m8` (assuming `SHOW_HASH` contains the correct SHA-256 hash).
```bash
curl -X POST \
"http://<IP_ADDRESS_OR_HOSTNAME>/upload?filename=p1_001_a.m8&hash=${SHOW_HASH}" \
-H "Authorization: BASIC <your_base64_token>" \
-H "Content-Type: application/octet-stream" \
--data-binary "@my_show.m8"
```

**Download a File (GET `/download`)**

Downloads a specific file from the device’s storage.

**Method:** GET
**Endpoint:** `/download?filename=<filename>` (Note: Not under `/api/v0/`)
**Authentication:** Required.
**Query Parameters:**
`filename` (string): The exact name of the file to download (e.g., `p1_001_a.m8`).
**Headers:**
`Authorization: BASIC <base64_encoded_credentials>`
**Response:** `200 OK` returns the raw binary content of the file with `Content-Type: application/octet-stream`. Errors (e.g., 401, 404) indicate authentication failure or that the file does not exist.

**Example (using curl):** Download `p1_001_a.m8` and save it locally as `downloaded_show.m8`.
```bash
curl -X GET "http://<IP_ADDRESS_OR_HOSTNAME>/download?filename=p1_001_a.m8" \
-H "Authorization: BASIC <your_base64_token>" \
-o "downloaded_show.m8"
```

## Alert Management (`/alert`)

Endpoints for interacting with the 32-bit alert bitfield, allowing external systems to monitor and influence alert states.

**Get Current Alert State (GET `/alert`)**

Retrieves the current value of the 32-bit alert bitfield.

**Method:** GET
**Endpoint:** `/api/v0/alert`
**Authentication:** Required.
**Headers:**
`Authorization: BASIC <base64_encoded_credentials>`
**Response (200 OK):** A JSON object containing the current alert value.
```json
{
  "alerts": <current_alert_value>
}
```
`current_alert_value` (integer): The decimal representation of the 32-bit alert bitfield. Each bit corresponds to an alert (bit 0 = alert 1, ..., bit 31 = alert 32).

**Set Specific Alert Bits (POST `/alert/set/{value}`)**

Sets one or more specific alert bits to 1 (active) without affecting other bits. Uses a bitwise OR operation (`current_alerts | value`).

**Method:** POST
**Endpoint:** `/api/v0/alert/set/{value}`
**Authentication:** Required.
**Path Parameter:**
`{value}` (integer): A decimal integer representing the bitmask of alerts to set. For example, to set Alert 3 (bit 2), use value 4 (binary `100`). To set Alerts 1 and 5 (bits 0 and 4), use value 17 (binary `10001`).
**Headers:**
`Authorization: BASIC <base64_encoded_credentials>`
**Response (200 OK):** Indicates successful update.

**Example (using curl):** Set Alert 3 (bit 2, value 4).
```bash
curl -X POST "http://<IP_ADDRESS_OR_HOSTNAME>/api/v0/alert/set/4" \
-H "Authorization: BASIC <your_base64_token>"
```

**Clear Specific Alert Bits (POST `/alert/clear/{value}`)**

Clears one or more specific alert bits to 0 (inactive) without affecting other bits. Uses a bitwise AND NOT operation (`current_alerts & ~value`).

**Method:** POST
**Endpoint:** `/api/v0/alert/clear/{value}`
**Authentication:** Required.
**Path Parameter:**
`{value}` (integer): A decimal integer representing the bitmask of alerts to clear.
**Headers:**
`Authorization: BASIC <base64_encoded_credentials>`
**Response (200 OK):** Indicates successful update.

**Example (using curl):** Clear Alert 3 (bit 2, value 4).
```bash
curl -X POST "http://<IP_ADDRESS_OR_HOSTNAME>/api/v0/alert/clear/4" \
-H "Authorization: BASIC <your_base64_token>"
```

## Feature Flags (`/features`)

Retrieves the list of optional features currently enabled on the M8-TW-ULTRA controller.

**Method:** GET
**Endpoint:** `/api/v0/features`
**Authentication:** Required.
**Headers:**
`Authorization: BASIC <base64_encoded_credentials>`
**Response (200 OK):** A JSON array of strings, where each string is the name of an enabled feature.
```json
[
  "telnet",
  "artnet",
  "webserver"
]
```
Features are enabled or disabled using the `set feature <name>` and `clear feature <name>` commands via the Serial/Telnet interface or the `/command` REST endpoint, followed by `set save`.

