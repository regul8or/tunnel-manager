# Tunnel Manager Project Philosophy

Tunnel Manager is a distributed system that intelligently routes network traffic through a WireGuard tunnel for domains that are geographically or politically restricted. It operates on the principle of minimal router load by offloading DNS monitoring and resolution logic to dedicated servers

> **Project Context:** This is a personal home lab project designed to solve a specific networking challenge in a resource-constrained environment. Design decisions prioritize pragmatism, simplicity, and operational maintainability over enterprise-grade features. There is no budget, no team, and a fixed scope focused on making the system work reliably in a single home network.

### Problem Statement

- **Primary Challenge:** Single-core MikroTik router cannot efficiently resolve and maintain dynamic IP lists (2 minutes at 100% CPU for ~20 domains)
- **Secondary Challenge:** Modern services use CDNs with rapidly changing IPs and complex subdomain patterns
- **Tertiary Challenge:** Manual maintenance of tunnel routing rules is error-prone and time-consuming

### Solution Architecture

Three independent microservices communicating via MQTT:

1. **Config Sync:** Exports MikroTik configuration as source of truth
2. **DNS Monitor:** Reactive pattern matching on live DNS queries
3. **Router Updater:** Intelligent resolution, aggregation, and router updates

## Core Principles

### 1. Configuration as Code

- **All configuration in YAML:** Human-readable, version-controllable, easy to audit
- **No hardcoded magic constants:** Topics, timeouts, thresholds all configurable
- **Credentials in config files:** Protected by filesystem permissions (600), not environment variables
- **Single source of truth:** MikroTik's disabled address-lists are the authoritative configuration database. List entries are disabled to prevent automatic address resolution by router

### 2. Separation of Concerns

- **Config Sync:** Only reads and publishes, never modifies router state
- **DNS Monitor:** Only pattern matching and publishing, no resolution or router interaction
- **Router Updater:** Only subscribes and acts, no direct DNS log access
- **MQTT as message bus:** Complete decoupling of components

### 3. Operational Excellence

- **Systemd-native:** Use systemd timers, oneshot services, proper journaling
- **Interactive-friendly:** Services automatically detect TTY and adjust logging accordingly
- **Observable by default:** Comprehensive metrics and health reporting from inception
- **Fail-safe:** Components crash cleanly, systemd handles restarts

### 4. Security by Design

- **Principle of least privilege:** Dedicated `tunnel-manager` user, minimal permissions
- **Config file protection:** 600 permissions on files containing credentials
- **No exposed services:** All communication via internal MQTT broker
- **Audit trail:** All router modifications logged to MQTT status topics

### 5. Performance Optimization

- **IP aggregation:** Automatically merge adjacent IPs into /24 subnets (configurable threshold)
- **Intelligent deduplication:** 10-minute resolution window prevents DNS query storms
- **Efficient pattern matching:** In-memory data structures, no database overhead
- **Minimal router load:** Only lightweight API calls to MikroTik

### 6. Maintainability

- **Self-documenting code:** Application-specific naming (get_tunneled_domains, not get_disabled_address_lists)
- **Living documentation:** This philosophy document evolves with the project
- **Single repository:** Shared code in `common/`, consistent patterns across components
- **Standard Python practices:** Virtual environments (.venv), requirements.txt, proper logging

## Technical Decisions

### Language & Runtime

- **Python 3.10+:** Available on Ubuntu 22.04 LTS, modern asyncio support
- **Virtual environments:** `.venv` directory, not system-wide packages
- **Standard library first:** Minimize external dependencies where possible

# Attribution

This project was built using AI-assisted development with Claude. I told it what to build and the AI did a lot of typing. Commits with AI contributions are marked appropriately because I'm not going to pretend otherwise.
