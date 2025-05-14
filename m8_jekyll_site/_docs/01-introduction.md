---
layout: default
title: Introduction
nav_order: 1
---

# Introduction

This section provides a high-level overview of the mate. API ecosystem, focusing on the M8-TW-ULTRA controller, its architecture, and the conventions used throughout this documentation.

## Purpose

This document serves as the definitive reference guide for the mate. API ecosystem, specifically tailored for the M8-TW-ULTRA controller. It is meticulously crafted to equip developers, system integrators, and administrators with a comprehensive understanding of the device’s capabilities, its diverse communication interfaces, and the available programming options. The primary objectives are to deliver clear, consistent, and accurate information regarding all available APIs, elucidate the intricate relationships between various system components, offer practical, real-world examples for common use cases, and function effectively as both an initial learning resource for newcomers and a reliable long-term reference manual for experienced users.

## System Architecture Overview

The mate. system delivers sophisticated intelligent lighting control, primarily orchestrated by the powerful M8-TW-ULTRA controller. The overall architecture is composed of several integral elements working in concert.

**Core Hardware Components:** The central piece of hardware is the M8-TW-ULTRA Controller. This unit is responsible for managing complex lighting scenes, defining and executing alert behaviors, performing energy monitoring, and facilitating integration with various external systems. Complementing the controller, mate. Keypads function as wired input devices, directly interfacing with the controller to handle user-initiated events such as activating specific lighting scenes, adjusting dimming levels precisely, or signaling critical alert inputs. Furthermore, the MultiPort I/O component offers highly flexible, software-configurable ports that support both input and output modes. These are predominantly utilized for external alert signaling mechanisms, often involving dry contact closures or voltage sensing.

**Communication Interfaces:** The system leverages two primary interfaces for interaction and control. A modern REST API provides a standardized, HTTP-based method for a wide range of operations including device configuration, command execution, detailed keypad setup, comprehensive alert management, file operations (upload/download), and retrieval of detailed statistics. Concurrently, a Serial Command Interface offers a robust, text-based command protocol designed for direct device control, diagnostics, and troubleshooting through a physical serial connection.

**Functional Subsystems:** Key functionalities are managed by distinct subsystems. The Lighting Control subsystem is responsible for managing scene activation, fine-grained brightness adjustments, and dynamic color temperature (CCT) changes. The Alerts System employs an efficient 32-bit bitfield management scheme for monitoring system status and integrating seamlessly with third-party monitoring or automation systems. Statistics Collection diligently logs both real-time and historical data related to energy usage and user button interactions, providing valuable insights for performance analysis and reporting. Lastly, Show File Management dictates the execution of lighting scenes based on uploaded `.m8` show files. These files must adhere to a specific naming convention (`px_yyy_z.m8`) where `p1`, `p2`, or `px` denotes the target port(s), `yyy` represents the zero-padded show or contact number (e.g., `001`), and `z` indicates the priority level (`a`, `b`, or `c`, with `a` being the highest).

## API Ecosystem Components

The mate. API ecosystem comprises several well-coordinated components, each designed to provide comprehensive control and monitoring capabilities.

The Serial Command API facilitates direct, text-based control over fundamental device operations, including power management, CCT adjustments, show mode selection, calibration procedures, debugging access, and network settings configuration.

The REST API Endpoints offer an extensive set of HTTP-based methods for configuration and control. These cover areas such as keypad settings, alert management, show triggering, file management (upload, download, listing), and statistics retrieval.

A specialized Keypad Configuration API, accessible via the REST interface, focuses specifically on mapping the behavior of keypad buttons and MultiPort I/O pins. This includes configuring lighting scene triggers, dimming controls, status indicator colors, and alert signal inputs/outputs.

The Show File System provides an organized structure for managing lighting scene files (`.m8`). It relies on strict naming conventions to handle scene priority and target port assignments effectively.

The Alerts System utilizes a 32-bit unsigned integer (bitfield) for efficient tracking and management of system alerts. Dedicated REST endpoints support setting, clearing, and monitoring the current state of these alerts.

The Statistics API grants access to detailed historical and real-time data concerning energy consumption and user interactions, crucial for performance tracking, system analysis, and generating reports.

Finally, the File Management System, exposed through the REST API, handles the essential tasks of uploading, downloading, listing, and verifying device files, with a particular focus on the critical `.m8` show files and `.pro` profile files.

## Terminology and Conventions

A clear understanding of the following terms and conventions is crucial for effectively utilizing the mate. API and this documentation.

**General Terms:**

**Port:** Refers to a logical grouping of output channels on the M8-TW-ULTRA. For instance, Port 1 often corresponds to Channels 1 & 2, and Port 2 corresponds to Channels 3 & 4.

**Show:** Represents a predefined lighting scene or sequence stored within an `.m8` file. Shows are usually triggered by events such as a contact closure on a MultiPort I/O pin or an ArtTrigger event.

**Profile:** Denotes a preset configuration defining specific lighting parameters, such as maximum brightness and CCT range, stored in a `.pro` file.

**Keycode:** A unique numeric identifier assigned to a specific keypad button or a MultiPort I/O connection point. This identifier is essential for configuring button and pin behaviors.

**Alert:** Represents a system status notification or an external event. Each alert corresponds to a single bit within the 32-bit integer bitfield used by the Alerts System.

**API Conventions:**

**Endpoints:** All REST API endpoints documented herein are relative to the base URL `http://<IP ADDRESS>/api/v0/`, unless explicitly stated otherwise (e.g., `/upload`, `/download`).

**Authentication:** Access to most REST endpoints requires an `Authorization` header containing a valid API key token, utilizing HTTP Basic Authentication. The format is `Authorization: BASIC <credentials>`, where `<credentials>` is the Base64 encoding of `apikey:<your_api_key>`.

**Request Format:** POST requests generally expect a JSON-formatted body, unless specified otherwise (e.g., file uploads use `application/octet-stream`).

**Response Format:** Responses from the REST API are often JSON objects. These objects commonly contain standardized fields such as `data` (for successful responses), `status`, or `error` (for issues).

**Command Syntax:**

**Serial Commands:** Follow the structure `command [subcommand] [arguments]`. These commands are plain-text, case-sensitive, and terminated by a newline character.

**REST API Calls:** Adhere to standard HTTP methods (e.g., GET, POST, PUT, DELETE). JSON is often used for request and response bodies where applicable.

**Keypad Configuration:** Utilizes a structured JSON format within REST API calls to specify the `keycode`, `function`, and associated parameters for each button or pin.

**Data Types:**

**Bitfield:** A 32-bit unsigned integer where each individual bit position corresponds to a specific alert status (0 = inactive, 1 = active).

**Stats:** Numerical data representing energy usage (often in Watt-hours) or user interactions (keypress counts), tracked over distinct hourly and daily intervals.

**Show File:** An `.m8` file containing the programming logic for lighting scenes, including rules governing their priority and interaction.

**Formatting Conventions:**

Code elements, including commands, API endpoints, JSON keys, example values, and filenames, are represented using monospace font (e.g., `set p 0 255`, `/api/v0/m8poll`, `"keycode": 2`).

**Bold text** is employed to emphasize important terms, concepts, or warnings.

*Italic text* is used to indicate parameter names, placeholders within syntax examples, or variable values that need to be substituted by the user.

## How to Use This Documentation

This documentation is carefully structured to cater to the diverse needs of its audience.

**New Users:** If you are new to the mate. system or the M8-TW-ULTRA, it is highly recommended to begin with Section 2, “Getting Started.” This section covers fundamental concepts, system requirements, and basic interaction methods to help you quickly become familiar with the device.

**Quick Reference:** For users seeking specific information, the detailed Table of Contents or the search functionality (if available in the viewing format) can be used to rapidly locate details about particular API calls, command syntax, parameters, error codes, or specific features.

**Integration Projects:** Developers and integrators working on integrating the M8-TW-ULTRA into larger systems will find Section 7, “Integration Guides,” particularly valuable. It provides practical examples and patterns for common tasks like lighting control automation, alert management integration, and system monitoring.

**Comprehensive Study:** For a complete and in-depth understanding of the mate. system’s capabilities and the intricate interactions between its various components, reading the documentation in its entirety is recommended. This approach ensures a thorough grasp of all features, APIs, and configuration options.

