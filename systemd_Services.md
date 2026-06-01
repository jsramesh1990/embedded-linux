# systemd Services

## Table of Contents

* Introduction
* What is a systemd Service?
* Why Use systemd Services?
* Yocto Integration
* Key Variables
* Service File Structure
* Creating a systemd Service
* Installing a Service in Yocto
* Enabling Auto Start
* Build and Verification
* Advantages
* Disadvantages
* Real-World Use Cases
* Best Practices
* Interview Questions
* Conclusion

---

# Introduction

Modern Yocto-based Linux systems use **systemd** as the default init system to manage services, daemons, and background processes.

A service can be automatically started during boot, monitored during runtime, and restarted if it fails.

Yocto provides built-in support for systemd through the `systemd.bbclass`.

---

# What is a systemd Service?

A systemd service is a unit file that defines how an application should be started and managed by the operating system.

Example:

```ini
[Unit]
Description=Temperature Monitoring Service

[Service]
ExecStart=/usr/bin/temp-monitor
Restart=always

[Install]
WantedBy=multi-user.target
```

Service file:

```text
temp-monitor.service
```

---

# Why Use systemd Services?

Systemd services are used to:

* Start applications automatically during boot
* Run background daemons
* Monitor application health
* Restart failed applications
* Manage service dependencies
* Improve system reliability
* Simplify service management

---

# Yocto Integration

Yocto supports systemd using:

```bitbake
inherit systemd
```

The systemd class automatically:

* Packages service files
* Enables services
* Creates required symbolic links
* Integrates services into the root filesystem

---

# Key Variables

## SYSTEMD_SERVICE:${PN}

Defines the service file associated with the package.

```bitbake
SYSTEMD_SERVICE:${PN} = "temp-monitor.service"
```

Multiple services:

```bitbake
SYSTEMD_SERVICE:${PN} = " \
    logger.service \
    mqtt.service \
"
```

---

## SYSTEMD_AUTO_ENABLE

Controls whether the service is enabled automatically.

Enable service:

```bitbake
SYSTEMD_AUTO_ENABLE = "enable"
```

Disable service:

```bitbake
SYSTEMD_AUTO_ENABLE = "disable"
```

---

## inherit systemd

Enables Yocto systemd functionality.

```bitbake
inherit systemd
```

---

# Service File Structure

A typical service file contains three sections.

## Unit Section

Defines metadata and dependencies.

```ini
[Unit]
Description=Temperature Monitoring Service
After=network.target
```

## Service Section

Defines how the application runs.

```ini
[Service]
Type=simple
ExecStart=/usr/bin/temp-monitor
Restart=always
RestartSec=5
```

## Install Section

Defines boot target.

```ini
[Install]
WantedBy=multi-user.target
```

---

# Creating a systemd Service

Directory structure:

```text
meta-custom/
└── recipes-apps/
    └── temp-monitor/
        ├── temp-monitor.bb
        └── files/
            └── temp-monitor.service
```

---

# Installing a Service in Yocto

Recipe example:

```bitbake
DESCRIPTION = "Temperature Monitoring Application"

LICENSE = "MIT"

SRC_URI += "file://temp-monitor.service"

inherit systemd

SYSTEMD_SERVICE:${PN} = "temp-monitor.service"

SYSTEMD_AUTO_ENABLE = "enable"

do_install:append() {
    install -d ${D}${systemd_system_unitdir}

    install -m 0644 \
        ${WORKDIR}/temp-monitor.service \
        ${D}${systemd_system_unitdir}/
}
```

---

# Build and Verification

Build package:

```bash
bitbake temp-monitor
```

Build image:

```bash
bitbake core-image-minimal
```

Verify on target:

Check service status:

```bash
systemctl status temp-monitor.service
```

Check enabled services:

```bash
systemctl list-unit-files | grep temp-monitor
```

View logs:

```bash
journalctl -u temp-monitor.service
```

---

# Common Service Commands

Start service:

```bash
systemctl start temp-monitor.service
```

Stop service:

```bash
systemctl stop temp-monitor.service
```

Restart service:

```bash
systemctl restart temp-monitor.service
```

Enable service:

```bash
systemctl enable temp-monitor.service
```

Disable service:

```bash
systemctl disable temp-monitor.service
```

---

# Advantages

## Automatic Startup

Applications start automatically during boot.

## Service Monitoring

Systemd continuously monitors services.

## Automatic Recovery

Failed services can restart automatically.

## Dependency Management

Services can start only after required services.

Example:

```ini
After=network.target
```

## Integrated Logging

Logs can be viewed using:

```bash
journalctl
```

## Improved Reliability

Reduces manual intervention during failures.

---

# Disadvantages

## Learning Curve

Requires understanding systemd unit files.

## Increased Complexity

More configuration compared to simple startup scripts.

## Debugging Dependencies

Complex dependency chains may be difficult to troubleshoot.

## Resource Usage

Consumes slightly more memory than lightweight init systems.

---

# Real-World Use Cases

## IoT Devices

* MQTT Clients
* Cloud Agents
* Device Management Services

## Automotive Systems

* CAN Bus Services
* Vehicle Diagnostics

## Industrial Automation

* PLC Communication
* Sensor Monitoring

## Networking Equipment

* DHCP Services
* VPN Services
* Routing Daemons

## Medical Devices

* Monitoring Services
* Data Collection Applications

---

# Best Practices

* Always use `inherit systemd`
* Keep service files simple
* Use meaningful service names
* Enable only required services
* Use `Restart=always` for critical applications
* Add dependency rules using `After=`
* Verify startup during boot testing
* Monitor logs with `journalctl`

---

# Interview Questions

### What is the purpose of `inherit systemd`?

It provides Yocto support for installing and managing systemd services.

### What does `SYSTEMD_SERVICE:${PN}` do?

It specifies the service file associated with the package.

### What is `SYSTEMD_AUTO_ENABLE`?

It controls whether a service is enabled automatically at boot.

### Where are systemd service files installed?

```text
/lib/systemd/system/
```

### How do you check service status?

```bash
systemctl status <service-name>
```

---

# Conclusion

systemd services are the standard method for managing applications and background daemons in Yocto-based Linux systems. Using `inherit systemd`, `SYSTEMD_SERVICE:${PN}`, and `SYSTEMD_AUTO_ENABLE`, developers can automatically install, enable, monitor, and manage services across embedded Linux devices.


