---
layout: default
title: Serial Command Interface
nav_order: 3
---

# Serial Command Interface

This section details the commands available through the direct Serial (console) or Telnet interface of the M8-TW-ULTRA controller. These commands provide low-level control over various device functions and are essential for initial setup, diagnostics, and certain configuration tasks.

## Accessing the Interface

Access to the command-line interface can be achieved through two primary methods. One method is via the Serial Console, which involves a direct physical connection to the controller’s serial port, often requiring a USB-to-UART adapter or similar hardware. This approach is useful for initial setup before network configuration or when network access is unavailable. The other method is Telnet, a network-based connection using a Telnet client. This requires the controller to be connected to the network and have the Telnet feature enabled (using `set feature telnet` followed by `set save`). The default Telnet port is 23. Authentication is required for Telnet access, as detailed in Section 2.2.

## Command Format and Syntax Rules

Commands sent via Serial or Telnet adhere to a simple plain-text structure: `<command> [subcommand] [arguments]`.

Key Syntax Rules include case-sensitivity for commands and subcommands. Each command must be terminated with a newline character (`\n`) or a carriage return followed by a newline (`\r\n`); using `\r\n` is often more robust for compatibility with various terminal emulators. Arguments are separated by spaces. Changes made to persistent configuration settings (e.g., API key, network settings, calibration results, feature flags, profile assignments) must be explicitly saved to the device’s non-volatile memory using the `set save` command. Failure to use `set save` will result in the loss of these changes upon device reboot. For example, to save changes after setting an API key, you would use `set apikey YourNewKey` and then `set save`.

## Power and CCT Control

These commands directly manipulate the power output and color temperature of the connected lighting fixtures.

**Set Power Level (`set p`)**

This command adjusts the raw Pulse-Width Modulation (PWM) duty cycle for a specific output channel. The syntax is `set p <channel> <level>`. The `<channel>` parameter is the target output channel number (0–3). The `<level>` parameter is the desired power level, represented as a PWM duty cycle value (0–255), where 0 is off and 255 is maximum power. For instance, `set p 2 128` would set Channel 2 to approximately 50% power.

**Set CCT and Brightness (`set cct`)**

This command sets the Correlated Color Temperature (CCT) and overall brightness. It has two main forms. One form targets a specific port using a specific profile temporarily, and the other targets both ports simultaneously using their currently assigned profiles. Accurate color and brightness rendering requires appropriate lighting profiles to be assigned to the ports (see `set pro`).

The syntax for controlling both ports using their assigned profiles is `set cct x <temperature> <brightness>`. This syntax applies the specified temperature and brightness to both Port 0 and Port 1 simultaneously, using the profile currently assigned to each respective port via `set pro`.

The syntax for controlling a specific port and specifying a profile temporarily is `set cct <port> <profileId> <temperature> <brightness>`. This syntax applies the settings only to the specified `<port>` and uses the profile identified by `<profileId>` just for this command instance, without changing the port’s persistent profile assignment.

The parameters are as follows: `x` is a literal character indicating that both ports should be set using their assigned profiles. `<port>` (used only when not using `x`) is the target logical port number (0 or 1), where Port 0 controls Channels 0 & 1, and Port 1 controls Channels 2 & 3. `<profileId>` (used only when not using `x`) is the numeric identifier of the lighting profile to use for the calculation in this command instance. `<temperature>` is the desired color temperature in Kelvin (e.g., 2700, 4000). `<brightness>` is the desired overall brightness level (0–255).

For example, to set both ports to 3500K at full brightness using their assigned profiles, the command is `set cct x 3500 255`. To set only Port 1 to 2700K at 50% brightness, using Profile 1 for the calculation just for this command, use `set cct 1 1 2700 128`.

## Profile Management

Profiles (`.pro` files) define the electrical and photometric characteristics of the connected fixtures, enabling accurate CCT and brightness control.

**Assign Profile to Port (`set pro`)**

This command associates a specific lighting profile with a port persistently (requires `set save`). It can optionally include a quantity to scale current calculations if multiple identical fixtures are driven by the same port.
The basic syntax is `set pro <port> <profileId>`. The syntax with quantity is `set pro <port> <profileId> <qty>`.
The parameters are: `<port>` is the target logical port number (0 or 1). `<profileId>` is the numeric identifier of the lighting profile to assign (corresponding to the profile file, e.g., `p0001.pro` would be profile ID 1). `<qty>` (Optional) is the number of identical fixtures connected to the port, defaulting to 1 if omitted.
For example, to assign Profile 1 to Port 0, use `set pro 0 1` followed by `set save`. To assign Profile 2 to Port 1, specifying 3 connected fixtures, use `set pro 1 2 3` followed by `set save`.

## Calibration Commands

Calibration commands optimize the controller’s output based on the connected fixtures and power conditions. These are generally run during initial setup or if fixtures are changed. It is important to always run `set save` after successful calibration to persist the results. The general syntax is `set <cal_type> <port>`.

**Duty Cycle Calibration (`set cal`)**

This command adjusts the PWM duty cycle range to optimize power output accuracy. The syntax is `set cal 0`.

**Voltage Calibration (`set vcal`)**

This command calibrates the internal digital potentiometer to compensate for voltage variations and component tolerances. The syntax is `set vcal 0`.

**Pulse Width Calibration (`set pcal`)**

This command measures the actual PWM pulse width required to achieve target current levels and sets this as the new maximum PWM value, improving control accuracy across the dimming range. The syntax is `set pcal 0`.

**Automatic Calibration Sequence (`set autocal`)**

This command executes the `cal`, `tune` (an internal calibration step), and `pcal` calibration commands sequentially for a comprehensive calibration. The syntax is `set autocal 0`. It is recommended to use `set autocal 0` followed by `set save` for most initial calibration needs.

## LED Indicator Control

This set of commands controls the behavior of the M8-TW-ULTRA’s onboard status LED.

**Control Status LED (`set led`)**

This command manually sets the status LED to blink a specific color or allows it to resume automatic status indication. The syntax is `set led [red | green | blue | stop]`. The parameters `red`, `green`, or `blue` force the LED to blink continuously in the specified color, which is useful for visually identifying a specific unit. The `stop` parameter returns the LED to its normal operational behavior, where its color and pattern indicate the device’s status (e.g., network connectivity, errors).
For example, `set led green` will force the LED to blink green for identification. `set led stop` will return the LED to normal status indication.

## Keypad Configuration and Broadcast Groups

These commands configure how keypad events are handled, specifically regarding network broadcasting for synchronizing actions across multiple controllers.

**Enable/Configure Keypad Broadcast (`set keypad broadcast`)**

This command determines if keypad events generated by this controller are broadcast over the network and, if so, to which logical group. The syntax is `set keypad broadcast <group_number>`. The `<group_number>` parameter can be `0` to disable keypad event broadcasting entirely, meaning events are handled locally only. If `<group_number>` is `1` or greater, it assigns the device to broadcast group `n`. The controller will send out keypad events and listen for events from other controllers within the same group `n`.
For example, `set keypad broadcast 0` followed by `set save` disables broadcasting for local control only. `set keypad broadcast 5` followed by `set save` assigns this device to group 5 for synchronized actions. Controllers configured with the same non-zero broadcast group number will synchronize keypad actions. A button press triggering a specific function (like `INPUT_SHOW`) on one controller can trigger the same function on other controllers in the same group, provided they have compatible configurations.

## Show Mode Controls

These commands are for managing the playback of predefined lighting scenes (Shows) stored in `.m8` files.

**Start a Show (`show start`)**

This command initiates playback of a specific show file based on its number. The syntax is `show start <show_number>`. The `<show_number>` parameter is the numeric identifier of the show to start (corresponding to the `yyy` part of the `.m8` filename, e.g., `1` for `px_001_a.m8`). For example, `show start 5` starts show number 5.

**Stop a Show (`show stop`)**

This command stops any currently running show and returns control to manual settings or a default state. The syntax is `show stop`.

**Adjust Show Brightness (`show dim`)**

This command adjusts the overall brightness of the currently running show dynamically. The syntax is `show dim <increment>`. The `<increment>` parameter is a signed integer representing the amount to change the brightness. Positive values increase brightness, negative values decrease it. The exact effect and range depend on the programming within the specific show file. For example, `show dim -20` decreases the brightness of the running show.

## Debugging and Network Information

These commands are useful for troubleshooting device behavior and checking network status.

**Enable/Disable Debug Output (`debug`)**

This command controls the verbosity of diagnostic messages output to the console. The syntax is `debug [on | off | level <level_value>]`. `debug on` enables detailed debug output. `debug off` disables debug output. `debug level <level_value>` sets a specific debug verbosity level (the exact levels and their meanings may vary by firmware version).

**Display Network Information (`netinfo`)**

This command displays the current network configuration and status of the M8-TW-ULTRA, including IP address, MAC address, gateway, and DNS server information. The syntax is `netinfo`.

**Reboot Device (`reboot`)**

This command initiates a software reboot of the M8-TW-ULTRA controller. The syntax is `reboot`.

## API Key Management

These commands are for managing the API key used for authentication.

**Set API Key (`set apikey`)**

This command sets a new API key for the device. The syntax is `set apikey <new_api_key>`. The `<new_api_key>` can be up to 64 characters. Remember to use `set save` to persist the new key. For example, `set apikey mySecureKey123` followed by `set save`.

**Clear API Key (`clear apikey`)**

This command removes the currently configured API key. The syntax is `clear apikey`. Remember to use `set save` to persist this change. After clearing, the device may revert to default authentication or allow unrestricted access, depending on firmware specifics.

## Best Practices

When using the Serial Command Interface, always ensure commands are typed accurately, paying attention to case sensitivity and argument order. After making any persistent configuration changes (e.g., network settings, API keys, calibration, profile assignments, feature flags), always use the `set save` command to write these changes to non-volatile memory. Without `set save`, changes will be lost upon reboot. For clarity, it is good practice to issue commands one at a time and observe the response or effect before proceeding to the next command, especially during initial setup or troubleshooting. If network access is problematic, the serial console provides a reliable direct interface for diagnostics and configuration. Keep a record of critical settings like the API key and network configuration in a secure location.

