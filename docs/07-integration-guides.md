---
layout: default
title: Integration Guides
nav_order: 7
---

# Integration Guides

This section provides practical examples and recommended workflows for integrating the M8-TW-ULTRA with external systems, applications, or dashboards using its REST API and other interfaces.

## Basic Lighting Scene Setup (Show Trigger via Keypad)

This example demonstrates the workflow for uploading a pre-defined lighting show file (`.m8`) and configuring a keypad button to trigger it.

**Step 1: Upload the Show File**

Use the File Management API (`POST /upload`, see Section 4.7) to upload your show file (e.g., `my_scene.m8`) to the device. Ensure the filename on the device follows the required convention (`p<port_id>_<show_number>_<variant>.m8`, e.g., `p1_001_a.m8` for Show 1 on Port 0). First, calculate the SHA-256 hash of your local `my_scene.m8` file. Then, send the file using `POST /upload` with the correct filename and hash.

```bash
# Example assuming SHOW_HASH contains the calculated SHA-256 hash
curl -X POST "http://ultra.local/upload?filename=p1_001_a.m8&hash=${SHOW_HASH}" \
-H "Authorization: BASIC <your_base64_token>" \
-H "Content-Type: application/octet-stream" \
--data-binary "@my_scene.m8"
```

**Step 2: Configure a Keypad Button**

Use the Keypad Configuration API (`POST /keypad`, see Section 6.2) to map a physical button to trigger the uploaded show. Prepare the JSON configuration array. This example configures Button 1 (keycode 2) to trigger Show 1 (`show: 1`), associate it with statistics category 1 (CIRCADIAN, see Section 6.6), and enable dimming down (`dim: -3`) when held, repeating every 75ms.

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
  }
]
```
Send the configuration using `POST /keypad`. Remember this overwrites the entire keypad configuration, so include configurations for all buttons.

```bash
curl -X POST "http://ultra.local/api/v0/keypad" \
-H "Authorization: BASIC <your_base64_token>" \
-H "Content-Type: application/json" \
-d '[{"keycode":2,"function":"INPUT_SHOW","parameters":{"show":1,"stats":1,"dim":-3,"repeat":75}}]'
```

**Step 3: Test the Setup**

Press the physical Button 1 on the keypad to verify it triggers Show 1. Optionally, trigger the show manually via the `/command` endpoint (see Section 4.1) to confirm the show file itself is working correctly.

```bash
curl -X POST "http://ultra.local/api/v0/command" \
-H "Authorization: BASIC <your_base64_token>" \
-H "Content-Type: application/json" \
-d '{"command": "show start 1"}'
```

## Alert System with Physical I/O

This example demonstrates configuring a MultiPort pin as an input to trigger an alert, and another pin as an output to signal based on that alert state (e.g., connecting a sensor and an indicator light).

**Step 1: Configure Alert Input and Output Pins**

Use the Keypad Configuration API (`POST /keypad`, see Section 6.2) to define the behavior for the desired input and output pins. Prepare the JSON configuration array. This example configures Pin associated with keycode 32 as an input (`INPUT_ALERT`). It triggers Alert 7 (bit 6, value 64) when pulled to 0V (`set: 0`) and clears it when floating (`clear: 1`). It also configures Pin associated with keycode 16 as an output (`OUTPUT_ALERT`). It monitors Alert 7 (value 64). When Alert 7 is active, it outputs 12V (`set: 2`). When Alert 7 is inactive, it outputs 0V (`clear: 0`).

```json
[
  {
    "keycode": 32,
    "function": "INPUT_ALERT",
    "parameters": {
      "alerts": 64,
      "set": 0,
      "clear": 1
    }
  },
  {
    "keycode": 16,
    "function": "OUTPUT_ALERT",
    "parameters": {
      "alerts": 64,
      "set": 2,
      "clear": 0
    }
  }
]
```
Send the configuration using `POST /keypad`.

```bash
curl -X POST "http://ultra.local/api/v0/keypad" \
-H "Authorization: BASIC <your_base64_token>" \
-H "Content-Type: application/json" \
-d '[{"keycode":32,"function":"INPUT_ALERT","parameters":{"alerts":64,"set":0,"clear":1}},{"keycode":16,"function":"OUTPUT_ALERT","parameters":{"alerts":64,"set":2,"clear":0}}]'
```

**Step 2: Test the Input/Output Behavior**

Connect the input pin (keycode 32) to Ground (0V). Verify using `GET /api/v0/alert` (see Section 4.8) or `GET /api/v0/m8poll` (see Section 4.2) that Alert 7 (bit 6) becomes active (the total alert value should increase by 64). While Alert 7 is active, check the voltage on the output pin (keycode 16) using a multimeter; it should be 12V. Disconnect the input pin (let it float). Verify that Alert 7 becomes inactive (the total alert value should decrease by 64). Check the voltage on the output pin again; it should now be 0V.

**Step 3: Manually Trigger/Clear Alerts (for Testing)**

You can manually set or clear alerts using the API (see Section 4.8) to test the `OUTPUT_ALERT` configuration independently. To set Alert 7 (value 64):
```bash
curl -X POST "http://ultra.local/api/v0/alert/set/64" \
-H "Authorization: BASIC <your_base64_token>"
```
(Verify output pin (keycode 16) goes to 12V). To clear Alert 7 (value 64):
```bash
curl -X POST "http://ultra.local/api/v0/alert/clear/64" \
-H "Authorization: BASIC <your_base64_token>"
```
(Verify output pin (keycode 16) goes to 0V).

## Monitoring Device Status and Energy Usage (Dashboard)

This example outlines how to poll for live status and energy data suitable for a real-time dashboard display.

**Step 1: Periodically Poll `/m8poll` for Live Status**

Use `GET /api/v0/m8poll` (see Section 4.2) at frequent intervals (e.g., every 5-15 seconds) to get the current operational status, including active show, brightness levels, alert state, etc.
```bash
curl -X GET "http://ultra.local/api/v0/m8poll" \
-H "Authorization: BASIC <your_base64_token>"
```

**Step 2: Periodically Poll `/stats/new` for Energy Data**

Use `GET /api/v0/stats/new` (see Section 4.4) at less frequent intervals (e.g., every 30-60 seconds) to retrieve the currently accumulating energy statistics.
```bash
curl -X GET "http://ultra.local/api/v0/stats/new" \
-H "Authorization: BASIC <your_base64_token>"
```

**Step 3: Process the Responses**

Parse the JSON response from `/m8poll` to extract live status information (e.g., show, level, alerts). Parse the JSON response from `/stats/new`. The data array contains objects for the current daily and hourly intervals.
```json
{
  "type": "statistics",
  "energyUnits": "W-hr",
  "data": [
    {
      "ts": 1734498000,
      "int": 50827,
      "daily": 1,
      "energy": {
        "input": 43.811,
        "port1": 0.196,
        "port2": 0.392
      },
      "keypress": []
    },
    {
      "ts": 1734545200,
      "int": 2827,
      "daily": 0,
      "energy": {
        "input": 0.950
      },
      "keypress": []
    }
  ]
}
```

**Step 4: Display Relevant Data**

Extract the desired values (e.g., `show` from `/m8poll`, `energy.input` from the hourly record in `/stats/new`) to display on the dashboard. Remember the energy values are accumulated for the current, incomplete interval.

**Step 5: Calculate Instantaneous Power (Optional)**

Estimate the average power consumption during the elapsed portion of the current interval using the energy consumed (`energyWh`) and the elapsed time (`intervalSecondsElapsed`) from the `/stats/new` response. A function to calculate average power in Watts would be: `Power (W) = (Energy (Wh) * 3600) / Time (s)`. For example, using the hourly data from above: `currentHourlyEnergyWh = 0.950`; `currentHourlyElapsedSeconds = 2827`. The estimated current power in Watts would be approximately 1.21 Watts. This provides an estimate of the current power draw, useful for real-time display.

## General Integration Best Practices

Design your integration application with distinct modules or functions for different tasks such as authentication, status polling, command execution, file management, and configuration. Implement robust error handling for all API calls, checking HTTP status codes (200 for success, 4xx/5xx for errors) and parsing JSON error messages if provided by the API. Securely store and manage the API key or credentials used to generate the Base64 token, and handle 401 Unauthorized errors gracefully, potentially prompting for credentials or using a defined retry/refresh mechanism. Choose appropriate polling intervals based on the data needed: more frequent polling (e.g., 5-15 seconds) for `/m8poll` or `/alert` for near real-time status and alerts; less frequent polling (e.g., 30-60 seconds) for `/stats/new` for dashboard energy updates; infrequent polling for `/stats` only when historical data is required; and poll `/files` or `/keypad` only when needed to check current state before an update. Be mindful of the load placed on the M8-TW-ULTRA, especially with very frequent polling, and monitor device performance if possible. Check available file storage (`GET /files`) before large uploads. Remember that most REST API calls modify the running state. For changes like keypad configurations or feature settings to persist across device reboots, ensure the `set save` command is issued (via `POST /command` or Serial/Telnet) after making the changes.

