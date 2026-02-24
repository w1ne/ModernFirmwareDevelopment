# Connectivity and IoT Architecture

A connected embedded device is not just a microcontroller with a radio. It is a node in a distributed system where the network topology, protocol, and power budget are design constraints with the same weight as flash size.

## Choosing a Network Technology

There is no universal answer. Each technology makes a specific trade-off across five axes:

| Technology | Range | Power | Bandwidth | Cost | Infrastructure |
|---|---|---|---|---|---|
| Ethernet | Local | High | Very High | Medium | Required |
| Wi-Fi | ~100m | High | High | Low | Required |
| Bluetooth LE | ~50m | Very Low | Low | Low | None |
| 802.15.4/6LoWPAN | ~100m | Very Low | Very Low | Low | None |
| LoRaWAN | ~10km | Very Low | Very Low | Low | Required |
| LTE-M/NB-IoT | Global | Medium | Low | Medium | Required |
| Ethernet | Local | High | 1 Gbps | Medium | Required |

**Rule of thumb:** Use the lowest-power technology that meets your bandwidth and range requirements. A coin-cell sensor should never use Wi-Fi.

## The TCP/IP Stack on Bare Metal

Running IP on a microcontroller means you need a full networking stack. Two options dominate:

*   **lwIP (Lightweight IP):** The standard choice. A C library that supports IPv4/IPv6, TCP, UDP, DHCP, and DNS. Used in millions of devices.  
*   **Zephyr Native Networking:** If you run Zephyr, the networking subsystem is built-in and driver-integrated. Use the BSD socket API. Zephyr handles the rest.

The stack attaches to a **network device driver** that sends and receives raw frames to/from your MAC. Ethernet or Wi-Fi is wired at the driver layer.

## Securing Communication: TLS on Embedded

Never transmit sensitive data unencrypted. On embedded systems, use **Mbed TLS** (maintained by ARM, used in Zephyr and many RTOSs). It provides:

*   AES-128/256, ECDSA, and RSA.
*   A TLS 1.3 client/server implementation.
*   Hardware acceleration hooks so the crypto offloads to the MCU's hardware AES engine.

TLS handshake on a 64 MHz Cortex-M4 takes roughly 500ms. Budget for it in your connection state machine.

## Application Protocols

Choose your protocol based on the endpoint, not habit.

*   **MQTT:** Publish/subscribe model over TCP. Designed for constrained devices. AWS IoT, Azure IoT Hub, and most cloud backends speak MQTT natively.
*   **CoAP (Constrained Application Protocol):** REST-like protocol over UDP. Designed for 6LoWPAN and other constrained networks where TCP overhead is too large.
*   **HTTP/REST:** Useful when interoperating with existing web services. On bare metal, avoid it unless TLS is hardware-accelerated and the MCU has enough RAM (typically 64 KB+).

### Resilience: Single Points of Failure

In a sensor network, every node that routes data for others is a single point of failure. Design for it:

1.  **Mesh topology over star:** If one node fails, mesh re-routes around it. Thread (built on 6LoWPAN) does this automatically.
2.  **Local data buffering:** If the cloud connection drops, buffer sensor readings locally. Upload when connectivity resumes.
3.  **Heartbeat and watchdog:** If a node stops sending heartbeats, the server should alert and trigger an OTA recovery.

---

## Real-World Case Studies: Network Integrity

### The Cost of Insecure Connectivity (Negative Example)
In 2016, the **Mirai Botnet** demonstrated how poorly secured IoT devices could be weaponized to cripple the global internet.

*   **The Error:** Mirai targeted unremarkable IoT devices like IP cameras and DVRs. These devices were shipped with **hardcoded or default credentials** (e.g., "root"/"root" or "admin"/"admin") and had Telnet management ports open to the public internet.
*   **The Data:** The malware infected millions of devices. It launched a massive Distributed Denial of Service (DDoS) attack reaching **1.1 Terabits per second** against Dyn, a major DNS provider. This took down services like Netflix, Twitter, and Airbnb for hours.
*   **The Lesson:** Every connected device is a potential weapon. Unique, non-default credentials and the elimination of insecure management protocols (Telnet) are the minimum safety requirements for modern firmware. Secure identity must be rooted in hardware (e.g., Secure Elements or TPMs) to prevent fleet-wide exploitation.

---
