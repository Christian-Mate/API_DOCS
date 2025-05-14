---
layout: default
title: Key Concepts and Features
nav_order: 5
---

# Key Concepts and Features

This section explains fundamental concepts and features integral to understanding and effectively utilizing the M8-TW-ULTRA controller and its associated APIs.

## Ports and Channels

The M8-TW-ULTRA manages lighting output through a system of Ports and Channels. Channels (0-3) represent the four individual physical output terminals on the controller where lighting fixtures are connected. Each channel can have its power level (PWM duty cycle) controlled independently using the `set p` command (See Section 3.3). Ports (0-1) are logical groupings of channels used primarily for controlling tunable white (TW) fixtures. Port 0 controls Channels 0 and 1, while Port 1 controls Channels 2 and 3. When controlling CCT and brightness using the `set cct` command (See Section 3.3), you target a Port. The controller then uses the assigned lighting profile (`.pro` file) to calculate the appropriate power levels for the two underlying channels within that port to achieve the desired color temperature and overall brightness. Understanding the distinction between physical Channels and logical Ports is crucial for both direct power control and CCT/brightness adjustments.

## Shows and Profiles

**Shows (`.m8` files):**

Shows are pre-programmed lighting sequences or scenes stored as files on the M8-TW-ULTRA. They can define complex dynamic lighting effects, transitions, and color changes over time. Show files use the `.m8` extension. A specific naming convention must be followed for show files to be recognized and executed by the controller. Note that the naming uses `p1` to refer to Port 0 and `p2` to refer to Port 1 for client-facing identification: `p<port_id>_<show_number>_<variant>.m8`. The `<port_id>` indicates the target port(s): use `1` for Port 0, `2` for Port 1, or `x` if the show applies to both Port 0 and Port 1. The `<show_number>` is a three-digit number (001-255) identifying the show, used in commands like `show start <show_number>`. The `<variant>` is a single character (a-z) allowing for variations of the same show number. Examples include `p1_005_a.m8` (Show 5, variant ‘a’, for Port 0), `p2_010_b.m8` (Show 10, variant ‘b’, for Port 1), and `px_020_a.m8` (Show 20, variant ‘a’, for Both Ports). Shows are managed using the `show start`, `show stop`, and `show dim` commands (See Section 3.8) or via keypad configurations (Section 6). Files are uploaded and downloaded via the File Management REST endpoints (Section 4.7).

**Profiles (`.pro` files):**

Profiles contain the essential electrical and photometric data describing the specific tunable white lighting fixtures connected to the controller. This data includes information about the fixture’s CCT range, power consumption, and the relationship between PWM duty cycles on the two channels and the resulting CCT/brightness. Profiles are crucial for the `set cct` command to accurately render the desired color temperature and brightness. Profile files use the `.pro` extension. Profile files are named sequentially: `p<profile_id>.pro`, where `<profile_id>` is a four-digit number (e.g., `0001`, `0002`). An example is `p0001.pro` (Profile ID 1). Profiles are assigned to Ports using the `set pro` command (See Section 3.4). Files are uploaded and downloaded via the File Management REST endpoints (Section 4.7).

## Keypads and Keycodes

The M8-TW-ULTRA supports various keypads, which provide physical buttons for user interaction. The controller also features MultiPort pins that can be configured as digital inputs or outputs. Each physical button on a keypad and each configurable MultiPort pin is assigned a unique numeric identifier called a `keycode`. Keycodes are used in the Keypad Configuration (Section 6) to map a specific button press or pin state change (`INPUT_*` functions) to an action (e.g., starting a show, triggering an alert) or to map system state (`OUTPUT_*` functions) to controlling a pin or keypad LED. For standard 8-button mate. keypads, the keycodes correspond to powers of 2 (Button 1 = keycode 2, Button 2 = keycode 4, etc.), but specific hardware documentation should be consulted. Keycodes for MultiPort pins also have specific assignments (See Section 6.5).

## Alerts System

The Alerts System provides a mechanism for tracking and reacting to up to 32 distinct system states or external events. It is based on a 32-bit unsigned integer bitfield where each bit position (0-31) represents a unique alert. If a bit is set to 1, the corresponding alert is considered active. If a bit is set to 0, the alert is inactive. The overall state of all alerts is represented by the decimal value of this 32-bit integer (e.g., a value of 5 means Alert 1 (bit 0, value 1) and Alert 3 (bit 2, value 4) are active). Alerts can be triggered (bits set) or cleared (bits cleared) by MultiPort inputs configured with `INPUT_ALERT`, REST API calls (`/alert/set`, `/alert/clear` - Section 4.8), Serial/Telnet commands, or potentially internal system conditions. Active alerts can control MultiPort outputs configured with `OUTPUT_ALERT` or potentially modify show playback or other system logic.

## Statistics System

The M8-TW-ULTRA collects statistics on energy usage and keypad interactions over time. Statistics are aggregated into hourly and daily intervals. For each interval, the system records: `ts` (Timestamp - Unix epoch seconds - marking the start of the interval), `int` (Interval duration in seconds - 3600 for hourly, 86400 for daily), `daily` (Flag indicating if it’s a daily (1) or hourly (0) record), `energy` (An object detailing energy consumed - in Watt-hours - by the main input (`input`) and each port (`port1`, `port2`)), and `keypress` (An array counting the number of keypresses associated with different statistical categories - See Section 6.6). Statistics are accessed via the REST API endpoints `/stats` (all historical data) and `/stats/new` (currently accumulating data) - See Section 4.4.

## Features

The M8-TW-ULTRA has several operational features that can be enabled or disabled, primarily via the Serial Command Interface using `set feature <feature_name>` and `clear feature <feature_name>` (followed by `set save`). The currently enabled features can be queried via the REST API endpoint `/features` (Section 4.9).

Key Features include:
`auto_detect`: Enables automatic detection of connected fixture models.
`auto_cal`: Enables the `set autocal` command sequence.
`pwm_sweep`: Enables PWM sweep used for detecting fixture issues.
`dim`: Enables brightness control via a dedicated `INPUT_DIM` keypad function.
`hold_dim`: Enables brightness control via press-and-hold on an `INPUT_SHOW` keypad function.
`tftp`: Enables unsecured TFTP file transfers.
`file_txfr`: Enables secured REST API file transfers (`/upload`, `/download`).
`ssl`: Enforces HTTPS (SSL/TLS) for the REST API (Restart Required).
`apikey`: Enforces API key authentication for REST API and Telnet. Highly Recommended.
`telnet`: Enables the Telnet server (Port 23) for command-line access (Restart Required).
`dmx`: Enables setting channel power levels using DMX over UDP (Art-Net protocol).
`identify`: Enables the “identify” behavior (brief reset button tap triggers fixture flash/broadcast).
`www_update`: Enables firmware updates via the `set updatefw` command.
`alerts`: Enables the general alert system processing.
`code_blue`: Enables specific alert actions designated for “Code Blue” scenarios.
`fixture_alert`: Enables specific alert actions related to fixture status or errors.
`contact_out`: Enables MultiPort pins configured as contact closure outputs.
`contact_12v`: Enables MultiPort pins configured as 12V outputs.

Note that features marked with “Restart Required? Yes” need the M8-TW-ULTRA controller to be rebooted after being enabled or disabled for the change to take full effect.

