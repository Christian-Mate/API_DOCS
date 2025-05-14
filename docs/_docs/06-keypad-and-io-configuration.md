---
layout: default
title: Keypad and I/O Configuration
nav_order: 6
---

# Keypad and I/O Configuration

This section details how to configure the behavior of buttons on connected mate. Keypads and the MultiPort I/O pins using the REST API (`/keypad` endpoint) or potentially through direct configuration methods if available. Each physical button or I/O pin is identified by a unique keycode, which is then mapped to specific input or output functions with corresponding parameters.

## Configuration Structure

Keypad and I/O configurations are defined using a JSON structure, managed as an array of objects where each object defines the behavior for a single keycode. Key components of each object are the `keycode` (an integer, a unique numeric identifier for the specific button or I/O pin being configured; consult hardware documentation for the correct keycode mapping, as detailed in Section 6.5), the `function` (a string, the name of the action or behavior to assign to the keycode, determining whether the keycode acts as an input trigger or an output indicator), and `parameters` (a nested JSON object containing settings specific to the chosen function; the required fields within parameters vary depending on the function).

A strict field order requirement applies when submitting configurations via the `POST /api/v0/keypad` endpoint: the fields within each configuration object must appear in the specific order of 1. `keycode`, 2. `function`, 3. `parameters`. Failure to adhere to this order can lead to the request being rejected or misinterpreted by the controller.

An example configuration object is as follows:
```json
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
```

## Configuration Management (REST API)

The primary method for managing these configurations is via the `/api/v0/keypad` REST endpoint. To retrieve the current configuration, a `GET /api/v0/keypad` request fetches the current complete configuration as a JSON array. To update the configuration, a `POST /api/v0/keypad` request overwrites the entire existing configuration with the JSON array provided in the request body.

It is important to note that to modify a single button or pin, the standard workflow involves first performing a GET request to obtain the current configuration array. Then, the specific object(s) within the array should be modified in your application. Finally, the complete, updated array must be sent back to the endpoint via a POST request, ensuring the strict field order (`keycode`, `function`, `parameters`) is maintained for every object. More details on these REST API endpoints can be found in Section 4.6.

## Input Functions

Input functions define how the controller reacts to events triggered by keypad buttons or MultiPort pins configured as inputs, such as button presses or contact closures.

### 1. INPUT_SHOW

This function serves the purpose of triggering a specified lighting show on a button press. It can optionally initiate dimming when the button is held down. The required parameter is `show` (integer, 1-255), which is the show number to start when the button is pressed. Optional parameters include `stats` (integer), the statistics category (1-10) to associate with this keypress (see Section 6.6); `dim` (integer), the amount to adjust brightness per interval when the button is held (positive values increase brightness, negative values decrease it, requiring the `hold_dim` feature to be enabled); and `repeat` (integer, milliseconds), the time interval between dim adjustments while the button is held (e.g., 75), used in conjunction with `dim`.

For example, to configure Button 1 (keycode 2) to trigger Show 5 on press, log it as category 1, and dim down by 2 units every 100ms when held, the JSON would be:
```json
{
  "keycode": 2,
  "function": "INPUT_SHOW",
  "parameters": {
    "show": 5,
    "stats": 1,
    "dim": -2,
    "repeat": 100
  }
}
```

### 2. INPUT_DIM

This function directly adjusts the brightness of the current lighting state (manual or show) when the button is pressed or held. It requires the `dim` feature to be enabled. The required parameters are `dim` (integer, -255 to 255), the amount to adjust brightness per interval, and `repeat` (integer, milliseconds), the time interval between dim adjustments while the button is held.

For example, to configure Button 2 (keycode 4) to increase brightness by 5 units every 50ms when held:
```json
{
  "keycode": 4,
  "function": "INPUT_DIM",
  "parameters": {
    "dim": 5,
    "repeat": 50
  }
}
```

### 3. INPUT_ALERT

This function sets or clears specific alerts in the 32-bit alert system based on the state changes of an input pin, commonly used with MultiPort I/O connected to external sensors or contacts. The required parameters are `alerts` (integer), a bitfield specifying which alert bit(s) this input controls (e.g., 1 for Alert 1, 4 for Alert 3, 5 for Alerts 1 and 3); `set` (integer, 0-3), defining the pin state that triggers the activation (setting the bit) of the specified alerts; and `clear` (integer, 0-3), defining the pin state that triggers the deactivation (clearing the bit) of the specified alerts.

The state definitions for `set` and `clear` parameters are: 0 for a pin pulled to 0V (Ground), 1 for a pin that is floating (high impedance, disconnected), 2 for a pin connected to an external 12V source, and 3 for no action or if this state is ignored for this trigger.

For example, to configure MultiPort Pin 6 (keycode 32) to activate Alert 9 (bit 8, value 256) when the pin is pulled to ground (`set: 0`), and deactivate Alert 9 when the pin is floating (`clear: 1`):
```json
{
  "keycode": 32,
  "function": "INPUT_ALERT",
  "parameters": {
    "alerts": 256,
    "set": 0,
    "clear": 1
  }
}
```

## Output Functions

Output functions define how the controller drives the state of MultiPort pins configured as outputs or controls keypad LEDs based on internal system status, primarily alert states.

### 1. OUTPUT_ALERT

This function controls the state of an output pin based on whether specific alerts are active or inactive. The required parameters are `alerts` (integer), a bitfield specifying which alert bit(s) this output monitors (the output becomes active if any of the specified alerts are active); `set` (integer, 0-3), defining the output pin state when any of the monitored alerts are active; and `clear` (integer, 0-3), defining the output pin state when none of the monitored alerts are active. The state definitions are the same as for `INPUT_ALERT`.

For example, to configure MultiPort Pin 5 (keycode 16) to output 0V (Ground) (`set: 0`) when Alert 5 (bit 4, value 16) is active, and float (`clear: 1`) when Alert 5 is inactive (this could be used to drive a relay or external indicator):
```json
{
  "keycode": 16,
  "function": "OUTPUT_ALERT",
  "parameters": {
    "alerts": 16,
    "set": 0,
    "clear": 1
  }
}
```

### 2. OUTPUT_COLOR

This function controls a specific output pin used internally by some mate. keypads (like mate.8) to switch their status LED color (e.g., between orange and blue), often used to indicate circadian mode or other states. It requires no parameters within the `parameters` object. The M8-TW-ULTRA system automatically manages the state of this output based on internal logic (like active shows or schedules), and the keypad hardware interprets the state of this pin to change its LED color.

For example, to assign the color control function to the appropriate keycode for the keypad hardware (e.g., keycode 7, but verify with hardware documentation):
```json
{
  "keycode": 7,
  "function": "OUTPUT_COLOR",
  "parameters": {}
}
```

### 3. OUTPUT_POWER

This function sets the pin to output 12V continuously, regardless of system state. This is useful for powering accessories like motion detectors, occupancy sensors, or other low-power auxiliary devices. This function requires no parameters. Once applied, the pin will be driven high (12V output) whenever the device is powered and operating normally. The total available current for all 12V outputs is limited to 50mA, so ensure connected devices do not exceed this limit.

For example, to configure MultiPort Pin 1 (keycode 1) to output 12V at all times:
```json
{
  "keycode": 1,
  "function": "OUTPUT_POWER",
  "parameters": {}
}
```

## Keycode Mapping

Accurately identifying the keycode for each physical button or MultiPort I/O pin is essential for correct configuration. For standard mate. keypads (e.g., 6-button with dim), Button 1 is keycode 2 (bit position 1), Button 2 is keycode 4 (bit position 2), Button 3 is keycode 8 (bit position 3), Button 4 is keycode 16 (bit position 4), Button 5 is keycode 96 (bit position 5), Button 6 is keycode 32 (bit position 6), Button 7 (dim +) is keycode 48 (bit position 7), and Button 8 (dim -) is keycode 64 (bit position 8). The keycode assignments for MultiPort pins can vary, so consult the specific hardware documentation for your M8-TW-ULTRA revision to determine the correct keycodes for each pin. When using functions like `INPUT_ALERT` or `OUTPUT_ALERT`, the `alerts` parameter is a bitfield. To target multiple alerts, combine their individual bit values using a bitwise OR operation (or simply add the decimal values if they represent single, distinct bits). For example, to monitor or control Alerts 3 (value 4) and 5 (value 16), the `alerts` parameter would be 20 (4 | 16).

## Statistics Category (`stats` parameter)

The optional `stats` parameter for the `INPUT_SHOW` function allows keypresses to be categorized for analysis via the Statistics API (Sections 4.4 and 5.5). Assigning a category helps differentiate the purpose or context of show activations. Predefined categories include: 1 (CIRCADIAN, for activating a circadian rhythm show), 2 (BRIGHT, for activating a bright/task scene), 3 (RELAX, for activating a relaxed/dim scene), 4 (FOUR, user-defined), 5 (FIVE, user-defined), 6 (OFF, for turning lights off via a show/button), 8 (NC1, Nurse Call 1 related action), and 9 (NC2, Nurse Call 2 related action). Choose the category value that best represents the action triggered by the `INPUT_SHOW` configuration.

## Configuration Tips

Always double-check the hardware documentation for the correct keycode assignments for your specific keypad model and M8-TW-ULTRA revision, especially for MultiPort pins. Remember that `POST /api/v0/keypad` replaces the entire configuration, so use the GET-Modify-POST workflow for targeted changes. Strictly adhere to the `keycode`, `function`, `parameters` field order in JSON objects sent via `POST /api/v0/keypad`. Choose the correct function type (`INPUT_*` for triggers, `OUTPUT_*` for indicators/control) based on the desired behavior. Some functions rely on specific features being enabled (e.g., `INPUT_DIM` requires `dim`, `INPUT_SHOW` with `dim`/`repeat` requires `hold_dim`); ensure necessary features are active (See Section 5.6). If the controller is in a broadcast group (`set keypad broadcast <n>`), configured `INPUT_*` actions may trigger synchronized actions on other devices in the group. Keypad configurations applied via the REST API are generally persistent; however, related settings like the broadcast group number (`set keypad broadcast`) require a `set save` command (via Serial/Telnet or `POST /command`) to persist across reboots.

