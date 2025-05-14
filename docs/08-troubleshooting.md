---
layout: default
title: Troubleshooting
nav_order: 8
---

# Troubleshooting

This section provides guidance on diagnosing and resolving common issues encountered when working with the mate. M8-TW-ULTRA API and system.

## Common Issues and Solutions

When encountering a **401 Unauthorized** error, possible causes include an invalid or missing API token in the `Authorization` header, or the token may have expired or been reset on the device. To resolve this, verify the API key and Base64 encoding in the `Authorization` header. If necessary, re-set the API key using the serial command `set apikey <key>`.

If a **Connection Timeout** occurs, the device might be offline or unreachable, an incorrect hostname (`ultra.local`) or IP address might be in use, or there could be a network configuration issue such as a firewall or routing problem. To address this, verify device power and network connection. Ping `ultra.local` or the device’s IP address. If using the hostname fails, try the IP address directly. Check network settings and firewall rules. Use the `netinfo` serial command to check the device’s network status, and consider power cycling the device.

If **Show Playback Fails**, causes could be an incorrect show filename format (which should be `p<port_id>_yyy_z.m8`), the show file being missing or corrupted, conflicting show priorities, or the show requiring specific conditions (e.g., alerts) that are not met. Solutions include verifying the show filename adheres to the naming convention (p1/p2/px). Use `GET /api/v0/files` to check if the file exists and its size. Review show priorities if multiple shows might conflict, and check if the show depends on specific alert states.

When a **Keypad Not Responding** issue arises, it might be due to an incorrect keycode used in the configuration, missing or incorrect parameters for the function, a hardware wiring issue with the keypad connection, or the keypad configuration not being applied or having been overwritten. To troubleshoot, use `GET /api/v0/keypad` to verify the current configuration. Double-check the keycode mapping for the specific button. Ensure all required parameters for the assigned function are present. Inspect physical keypad connections and re-apply the keypad configuration if needed.

If **Alerts Not Triggering/Clearing**, this could stem from an incorrect alerts bitfield calculation, an incorrect `set`/`clear` state definition (0, 1, 2, 3), a voltage mismatch on the MultiPort I/O pin, or a physical wiring issue with a sensor or switch. To resolve, double-check the bitwise math for the `alerts` parameter. Verify the `set` and `clear` state values match the expected input/output behavior (0V, Float, 12V). Test physical signal levels on the I/O pins using a multimeter and inspect wiring connections for the input source or output load.

When a **File Upload Fails**, possible reasons are an incorrect SHA-256 hash provided, insufficient storage space on the device, or a network interruption during upload. Solutions involve recalculating and verifying the SHA-256 hash of the file. Check available storage using `GET /api/v0/files`. Ensure a stable network connection during upload and increase the request timeout interval if applicable.

If a **File Download Fails**, this might be due to an incorrect filename specified or the file not existing on the device. Verify the exact filename using `GET /api/v0/files` and increase the request timeout interval if applicable.

## Debugging Techniques

To aid in debugging, enable detailed diagnostic messages using the `debug` command via Serial or Telnet; these messages will be sent to both the Serial console and the active Telnet session. Use `debug off` (or `debug 0` as per some contexts in the document) to disable the output. Retrieve system event logs using the REST API (`GET /api/v0/log`, see Section 4.5) with a command like `curl -X GET "http://ultra.local/api/v0/log" -H "Authorization: BASIC <your_base64_token>"`.

Verify basic API connectivity by polling the device status using `GET /api/v0/m8poll` (see Section 4.2): `curl -X GET "http://ultra.local/api/v0/m8poll" -H "Authorization: BASIC <your_base64_token>"`. Test core functionality by sending a basic command via `POST /api/v0/command` (see Section 4.1), such as starting a known show: `curl -X POST "http://ultra.local/api/v0/command" -H "Authorization: BASIC <your_base64_token>" -H "Content-Type: application/json" -d '{"command": "show start 1"}'`.

Verify file storage by listing files using `GET /api/v0/files` (see Section 4.7) to ensure necessary show/profile files are present and check available storage space: `curl -X GET "http://ultra.local/api/v0/files" -H "Authorization: BASIC <your_base64_token>"`. For advanced issues, use network packet capture tools like `tcpdump` (Linux/macOS) or Wireshark (cross-platform) to inspect the HTTP requests and responses between your application and the device. This is useful for diagnosing low-level communication issues, incorrect headers, or unexpected responses.

## Support Resources

Review this API documentation and any accompanying hardware manuals carefully for correct specifications, formats, and endpoint usage. If issues persist, contact the manufacturer’s technical support team. Provide detailed information, including the device firmware version (from `GET /m8poll` response), specific API endpoint(s) used, request details (method, headers, body/parameters), response received (status code, body), steps taken to reproduce the issue, and relevant logs (`GET /log`) or serial debug output. Check relevant online forums or communities (e.g., lighting control user groups) for discussions on similar issues. Ensure the M8-TW-ULTRA device is running the latest stable firmware version, as updates often include bug fixes and improvements.

## HTTP Error Code Reference

Common HTTP status codes include **200 OK** (request was successful), **400 Bad Request** (invalid JSON format, missing required parameters, incorrect parameter values, invalid command syntax), **401 Unauthorized** (missing, invalid, or expired API key/token in `Authorization` header), **403 Forbidden** (access to the requested resource is denied, less common), **404 Not Found** (incorrect endpoint URL, requested file or resource does not exist), **500 Internal Server Error** (an unexpected error occurred on the device side; check logs or contact support), and **503 Service Unavailable** (the device is temporarily unable to handle the request, e.g., during startup, high load; retry later).

## General Troubleshooting Process

When encountering an issue, first identify the problem clearly: define what is not working as expected, which API endpoint or feature is failing, and note any specific error codes or messages. Second, check network connectivity: ensure the device is reachable on the network, ping its hostname or IP address, and check for firewalls blocking communication. Third, verify authentication: double-check that the API key is correct and properly included in the `Authorization: BASIC ...` header for every request. Fourth, test simple operations: attempt basic, known-working API calls (like `GET /m8poll`) to confirm fundamental communication and authentication. Fifth, isolate the issue: if related to configuration (keypad, alerts), retrieve the current configuration (`GET /keypad`, `GET /alert`) to verify settings; if related to files, check the file list (`GET /files`); if related to physical I/O, verify hardware wiring and signal levels. Sixth, consult documentation: carefully re-read the relevant sections of this documentation regarding the endpoint, parameters, expected formats, and behavior. Seventh, gather information: collect relevant logs (`GET /log`), debug output (serial `debug`), exact API requests/responses, and firmware version. Finally, if the issue cannot be resolved, contact technical support with the detailed information gathered.

