---
layout: default
title: Getting Started
nav_order: 2
---

# Getting Started

This section provides the essential information required to begin interacting with the M8-TW-ULTRA controller using its available Application Programming Interfaces (APIs). It covers system prerequisites, authentication mechanisms, fundamental API concepts, and basic interaction examples.

## System Requirements

To effectively utilize the mate. API ecosystem and interact with the M8-TW-ULTRA controller, ensure the following requirements are met:

**Hardware Requirements:**

The M8-TW-ULTRA Controller is the core device and is mandatory for all operations. A stable wired Ethernet connection is required for network-based API access (REST, Telnet, Art-Net); wireless (Wi-Fi) connectivity is not natively supported by the controller. While not strictly required for basic API control, mate. Keypads are recommended for implementing physical user controls, triggering scenes locally, and utilizing certain alert input functionalities. Access via a serial console (e.g., using a USB-to-UART adapter) can be beneficial for initial configuration, direct command-line access, or advanced troubleshooting, especially if network access is unavailable.

**Software Requirements:**

An HTTP Client is necessary for interacting with the REST API. Common tools include graphical clients like Postman, command-line utilities like curl, or custom applications developed using programming languages with HTTP libraries (e.g., Python with requests, JavaScript with fetch). A Telnet Client or Serial Terminal is required for direct command-line access via Telnet (if the feature is enabled on the device) or a direct serial connection. Examples include PuTTY (Windows), Terminal (macOS/Linux), or Tera Term (Windows).

**Network Requirements:**

The M8-TW-ULTRA controller must be connected to the same local Ethernet network as the client system attempting to access its APIs. The device can be accessed via its default hostname `ultra.local` (requires mDNS/Bonjour support on the network and client) or its assigned IP address. Default Ports are HTTP (REST API, Web Interface) on Port 80, Telnet (Command Line) on Port 23 (Note: Telnet access must be explicitly enabled using `set feature telnet` and saved using `set save`), and Art-Net (UDP) on standard Art-Net ports (6454).

## Authentication and Authorization

Security for the M8-TW-ULTRA’s interfaces (REST API, web interface, Telnet) is primarily managed through a persistent API key. This key also serves as the foundation for encrypting secure Art-Net communication packets.

**API Key Behavior:**

If no API key is set, Telnet access defaults to the username `admin` and password `admin`. REST API access might be unrestricted or have default credentials; it is strongly recommended to set an API key for secure operation. If an API key is set, Telnet login requires the username `admin` and the configured API key as the password. REST API requests must use HTTP Basic Authentication. The `Authorization` header must be included in requests, formatted as `BASIC <credentials>`, where `<credentials>` is the Base64 encoding of the string `apikey:<your_api_key>`. For example, if the API key is `daily-rhythm`, the string to encode is `apikey:daily-rhythm`. The resulting Base64 value is `YXBpa2V5OmRhaWx5LXJoeXRobQ==`. The required HTTP header would be: `Authorization: BASIC YXBpa2V5OmRhaWx5LXJoeXRobQ==`.

**Setting or Clearing the API Key:** The API key can be managed via the Serial/Telnet interface or the REST API’s `/command` endpoint.

Via Serial or Telnet:
```
# Set a new key (up to 64 characters)
set apikey your-new-secure-key

# Remove the existing API key
clear apikey

# IMPORTANT: Persist the change to non-volatile memory
set save
```

Via REST API (POST `/api/v0/command`): Send two separate POST requests with the appropriate `Authorization` header. First, to set or clear the key:
```json
// To set a key:
{ "command": "set apikey your-new-secure-key" }
```
```json
// To clear a key:
{ "command": "clear apikey" }
```
Second, to save the change:
```json
{ "command": "set save" }
```

**Encrypted Art-Net Communication:** To enhance security, particularly in sensitive environments, the M8-TW-ULTRA supports AES-128 encryption for Art-Net communication when an API key is configured. This applies to discovery (ArtPoll/ArtPollReply) and other Art-Net based events (like keypad broadcasts).

**Encryption Mechanism:** Encryption is automatically active when an API key is set. The 16-byte AES encryption key is derived from the first 16 bytes of the SHA-256 hash of the configured API key. AES-128 in Counter (CTR) mode is used. A unique, randomly generated 16-byte Initialization Vector (IV) is created for each packet; this IV is prepended to the encrypted Art-Net payload. The packet format is `[ 16-byte IV || Encrypted Art-Net Data ]`.

**Decryption Instructions for Clients:** To decrypt, receive the UDP packet. Extract the first 16 bytes as the IV. Derive the 16-byte AES key using the known API key (SHA-256 hash, take first 16 bytes). Use AES-128 CTR mode with the derived key and extracted IV to decrypt the remaining bytes of the packet. The resulting plaintext is the original Art-Net packet (e.g., ArtPollReply), which can then be parsed normally.

**Secure Discovery Process:** The client constructs a standard ArtPoll packet, derives the AES key from the API key, generates a random 16-byte IV, and encrypts the ArtPoll packet using AES-128 CTR with the key and IV. The client then prepends the IV to the ciphertext and broadcasts the resulting `[IV || ciphertext]` packet via UDP. The M8-TW-ULTRA receives the packet, extracts IV, derives key, decrypts payload, and processes the ArtPoll request. The M8-TW-ULTRA then constructs an ArtPollReply, derives key, generates IV, encrypts reply, prepends IV, and sends `[IV || ciphertext]` back to the client. The client receives response(s), extracts IV, derives key, decrypts payload, and parses the ArtPollReply.

**Authorization Levels:** Authentication using the API key grants full administrative access. The M8-TW-ULTRA does not implement distinct user roles or granular permission levels; a successfully authenticated session has complete control over the device.

## Base URLs and Endpoints Overview

**REST API Base URL:** Most REST API endpoints described in this documentation are relative to the following base URL: `http://<IP_ADDRESS_OR_HOSTNAME>/api/v0/`. Replace `<IP_ADDRESS_OR_HOSTNAME>` with the actual IP address or hostname (e.g., `ultra.local`) of your M8-TW-ULTRA controller.

**Special Endpoints (Different Base):** Note that file transfer endpoints use a different base path: Upload is `http://<IP_ADDRESS_OR_HOSTNAME>/upload`, and Download is `http://<IP_ADDRESS_OR_HOSTNAME>/download`.

**Common REST API Endpoints (Relative to `/api/v0/`):** A summary of frequently used endpoints includes `/command` for executing server-side commands, `/m8poll` for polling comprehensive real-time device status and active alerts, `/stats` for retrieving historical energy and interaction statistics, `/stats/new` for retrieving live (currently accumulating) statistics, `/keypad` for getting or setting keypad button configurations, `/alert` for getting or modifying the state of the 32-bit alert bitfield, `/files` for listing uploaded files (primarily `.m8` show files and `.pro` profiles), and `/features` for querying the list of currently enabled device features.

**Serial and Telnet Commands:** Direct connections via the Serial port or Telnet allow the execution of plain-text commands, many of which are also accessible via the `/command` REST endpoint. Refer to Section 3 for a detailed list of Serial/Telnet commands.

## Common Response Formats and Error Handling

**REST API Response Format:** Most REST API endpoints return responses in JSON format. Successful requests include relevant data, while errors are indicated through specific fields within the JSON response or standard HTTP status codes.

Example Successful Response (GET `/stats`):
```json
{
  "type": "statistics",
  "energyUnits": "W-hr",
  "data": [ /* array of statistics objects */ ]
}
```

Example Error Response (POST `/command` with invalid command):
```json
{
  "error": "Invalid command: shows start 1"
}
```

**HTTP Status Codes:** Standard HTTP status codes are used to signal the outcome of REST API requests. `200 OK` means the request was successful. `400 Bad Request` indicates the request was malformed (e.g., invalid JSON, missing parameters, invalid command syntax). `401 Unauthorized` signifies authentication failed (missing or incorrect API key/Authorization header). `404 Not Found` means the requested endpoint URL or resource (e.g., a specific file for download) does not exist. `500 Internal Server Error` indicates an unexpected error occurred on the M8-TW-ULTRA controller while processing the request. `503 Service Unavailable` means the device is temporarily unable to handle the request (e.g., during startup), and you should retry later. Refer to Section 8.4 for a more detailed list of HTTP error codes and common causes.

**Serial or Telnet Command Responses:** Commands executed via Serial or Telnet provide minimal feedback upon success (often just a command prompt return). Errors are usually reported directly as plain text messages in the console immediately following the failed command.

## Quick Start Examples

Here are basic examples demonstrating interaction via different interfaces. Remember to replace placeholders like `<your_base64_token>` and `<IP_ADDRESS_OR_HOSTNAME>` with your actual values.

**REST API: Check Device Status (using curl)**
```bash
curl -X GET "http://<IP_ADDRESS_OR_HOSTNAME>/api/v0/m8poll" \
-H "Authorization: BASIC <your_base64_token>"
```

**REST API: Send a Command to Start Show 1 (using curl)**
```bash
curl -X POST "http://<IP_ADDRESS_OR_HOSTNAME>/api/v0/command" \
-H "Authorization: BASIC <your_base64_token>" \
-H "Content-Type: application/json" \
-d '{"command": "show start 1"}'
```

**Telnet: Set API Key**
Connect to the device (replace `ultra.local` if using IP): `telnet ultra.local`. Then login (e.g., `admin` / `admin` if no key set, or `admin` / `<current_api_key>`). Once logged in, execute:
```
set apikey my-new-secure-apikey
set save
exit
```

**Serial Console: Check Network Info**
Connect via Serial terminal (e.g., PuTTY, minicom). Press Enter to see prompt if necessary. Then execute `netinfo`. Output showing IP, MAC, etc. will be displayed.

## Using SSL for Secure Communication

The M8-TW-ULTRA supports optional HTTPS (SSL/TLS) for encrypted communication with its REST API and Web Interface. This feature is controlled by enabling or disabling the SSL feature flag on the controller.

**Enabling SSL:** To enable SSL, use the commands `set feature ssl` and then `set save`. A system reboot is required for this change to take effect. Once SSL is enabled, all REST API requests must use the `https://` protocol, and the default port changes to 443 (unless otherwise configured). Plain HTTP connections (port 80) will be disabled or redirected, depending on firmware version.

**Disabling SSL:** To revert to unsecured HTTP, use `clear feature ssl` and then `set save`. This re-enables plain HTTP communication over port 80. Restart the controller for the change to take effect.

**Effects on API Usage:** When SSL is enabled, all REST API endpoints (e.g., `/api/v0/m8poll`, `/api/v0/command`, `/api/v0/files`) must be accessed using `https://<IP_ADDRESS_OR_HOSTNAME>` instead of `http://`.

Without SSL:
```bash
curl -X GET "http://ultra.local/api/v0/m8poll" -H "Authorization: BASIC <your_token>"
```

With SSL enabled:
```bash
curl -X GET "https://ultra.local/api/v0/m8poll" -H "Authorization: BASIC <your_token>" --insecure
```

**File Uploads and Downloads:** The `/upload` and `/download` endpoints must also use HTTPS when SSL is enabled:
```bash
# Upload example
curl -X POST "https://ultra.local/upload?filename=px_001_a.m8&hash=abc..." \
-H "Authorization: BASIC <token>" \
-H "Content-Type: application/octet-stream" \ --data-binary "@px_001_a.m8" --insecure
```

**Compatibility Considerations:** Not all embedded systems or microcontrollers support HTTPS easily, so test SSL-enabled communication with all intended clients. API tools (Postman, curl, etc.) must be configured for HTTPS. Telnet and Serial communication are unaffected by SSL settings.

