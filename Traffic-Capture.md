# Mobile Application Traffic Capture

A practical reference for capturing and analyzing HTTP/HTTPS traffic during authorized Android mobile application security testing.

> **Scope:** Use these techniques only on applications/devices you are authorized to test.

## 1. Burp Suite with Android Wi-Fi Proxy

### When to use
- Standard HTTP/HTTPS traffic interception
- API testing
- Request/response modification
- Replaying requests in Repeater

### Setup

1. Start **Burp Suite**.
2. Go to **Proxy → Proxy settings** and configure a listener, commonly:
   \`0.0.0.0:8080\`.
3. Find the tester machine IP:
   \`\`\`bash
   ip addr
   \`\`\`
4. On the Android device, configure the Wi-Fi network proxy:
   - Proxy: Manual
   - Host: tester machine IP
   - Port: \`8080\`
5. Open the Burp CA certificate page from the device:
   \`\`\`
   http://burp
   \`\`\`
6. Install the certificate as appropriate for the test environment.
7. Open the application and verify requests under **Proxy → HTTP history**.

### Validation

If HTTPS traffic is not visible, check:
- Certificate trust configuration
- Network Security Configuration
- Certificate pinning
- Proxy bypass behavior

## 2. Burp Suite with Android Emulator

Configure the emulator's proxy to point to the host running Burp.

\`\`\`bash
adb shell settings put global http_proxy <HOST_IP>:8080
\`\`\`

Verify:

\`\`\`bash
adb shell settings get global http_proxy
\`\`\`

Remove the proxy after testing:

\`\`\`bash
adb shell settings put global http_proxy :0
\`\`\`

## 3. ADB-Based Proxy Configuration

\`\`\`bash
adb devices
adb shell settings put global http_proxy <HOST_IP>:8080
adb shell settings get global http_proxy
\`\`\`

Reset:

\`\`\`bash
adb shell settings put global http_proxy :0
\`\`\`

## 4. mitmproxy

### When to use
- Lightweight interception
- Command-line workflows
- Scripting request/response handling
- Automated testing

Start:

\`\`\`bash
mitmproxy -p 8080
\`\`\`

Web interface:

\`\`\`bash
mitmweb -p 8080
\`\`\`

Configure the Android device/emulator to use the tester machine as its HTTP proxy and install/trust the mitmproxy CA in the authorized test environment.

## 5. Charles Proxy

### When to use
- GUI-based HTTP/HTTPS inspection
- Mobile application debugging
- Quick request/response analysis

General workflow:

1. Start Charles.
2. Enable the proxy listener.
3. Configure Android to use the Charles host and port.
4. Install the Charles CA certificate on the test device.
5. Launch the application.
6. Inspect traffic in the Charles session.

## 6. Wireshark / tcpdump — Network-Level Capture

These tools capture packets rather than providing an HTTP interception proxy.

### tcpdump

Example:

\`\`\`bash
adb shell tcpdump -i any -s 0 -w /sdcard/mobile_traffic.pcap
\`\`\`

Pull the capture:

\`\`\`bash
adb pull /sdcard/mobile_traffic.pcap
\`\`\`

Open in Wireshark:

\`\`\`bash
wireshark mobile_traffic.pcap
\`\`\`

Useful filters:

\`\`\`
dns
tcp
udp
tls
http
ip.addr == <DEVICE_IP>
ip.addr == <SERVER_IP>
\`\`\`

### Important limitation

TLS encrypts application data. A packet capture can show connection metadata such as IPs, ports and TLS information, but normally cannot reveal HTTP request/response contents without appropriate decryption material.

## 7. VPN-Based Capture

A VPN-based capture approach can be useful when application traffic does not follow the device's normal HTTP proxy configuration.

Typical workflow:

1. Start a VPN-based traffic inspection tool.
2. Install its CA certificate if HTTPS interception is supported and permitted.
3. Establish the VPN connection on the test device.
4. Launch the application.
5. Inspect captured connections.

This can help identify traffic that bypasses conventional Wi-Fi proxy configuration.

## 8. Browser / WebView Traffic

For applications containing WebViews:

1. Identify the WebView component.
2. Check whether WebView debugging is enabled in the authorized test build.
3. Use the appropriate Android debugging tools to inspect WebView behavior.
4. Compare WebView traffic with native application API traffic.

For Chrome/WebView debugging:

\`\`\`
chrome://inspect
\`\`\`

## 9. Certificate Pinning

If normal proxy interception works for other applications but the target application's HTTPS traffic is not visible, certificate pinning may be involved.

### First checks

- Confirm the device trusts the proxy CA.
- Confirm the application uses the expected network stack.
- Check Android Network Security Configuration.
- Review the APK for certificate pinning implementations.
- Check whether the application has different debug, UAT and production configurations.

### Authorized dynamic testing

For a test build/device where you have permission to perform dynamic analysis, tools such as **Frida** or **Objection** can be used to assess whether certificate pinning is enforced.

Typical investigation targets include:

- OkHttp \`CertificatePinner\`
- TrustManager implementations
- Network security configuration
- Custom certificate validation methods

> Pinning bypass should be treated as a testing technique, not as a production configuration.

## 10. Frida-Based Traffic Investigation

Frida can be used during authorized dynamic analysis to observe application behavior.

Useful investigation areas include:

- TLS/certificate validation methods
- OkHttp requests
- URL construction
- Request headers
- Request/response handling
- Encryption/decryption methods

Example workflow:

\`\`\`bash
adb devices
frida-ps -U
\`\`\`

Attach to an authorized test application:

\`\`\`bash
frida -U -f <PACKAGE_NAME> -l script.js
\`\`\`

Use this approach when proxy-level visibility is insufficient and you need to understand what the application is doing internally.

## 11. Comparing Capture Methods

| Method | HTTP/HTTPS Content | App Modification | Useful For |
|---|---|---|---|
| Burp + Wi-Fi Proxy | Yes, when trusted | No | API testing |
| Burp + Emulator Proxy | Yes, when trusted | No | Emulator testing |
| mitmproxy | Yes, when trusted | No | Automation/scripting |
| Charles | Yes, when trusted | No | GUI inspection |
| tcpdump | Usually encrypted | No | Network-level analysis |
| Wireshark | Usually encrypted | No | Packet analysis |
| VPN-based capture | Depends on setup | No | Proxy-bypass investigation |
| Frida | Can observe internal behavior | No, runtime instrumentation | Dynamic analysis |
| Objection | Can assist dynamic testing | No, runtime instrumentation | Mobile security testing |

## 12. Recommended Testing Order

For a normal Android VAPT engagement:

1. **Burp + Wi-Fi proxy**
2. **Verify CA certificate trust**
3. **Check whether HTTPS traffic is visible**
4. **Try emulator/device proxy configuration**
5. **Check for certificate pinning**
6. **Use an authorized pinning assessment approach**
7. **Use tcpdump/Wireshark for network-level visibility**
8. **Use Frida for deeper runtime investigation when required**

## 13. Troubleshooting Checklist

### No traffic at all
- Confirm device and tester machine are on the same network.
- Confirm the proxy IP and port.
- Check the Burp listener is reachable.
- Check Android proxy configuration.
- Check whether the application uses a proxy-bypassing network path.

### HTTP works but HTTPS does not
- Verify the proxy CA certificate.
- Check Android certificate trust behavior.
- Investigate certificate pinning.
- Check whether the application uses a custom TrustManager.

### Browser traffic works but app traffic does not
- Check application-specific proxy behavior.
- Check certificate pinning.
- Check whether the application uses a native networking library.
- Compare with tcpdump/Wireshark.

### Traffic is visible but data is encrypted
- Determine whether the encryption is TLS or application-level encryption.
- Inspect the application architecture and cryptographic implementation during authorized testing.
- Use runtime analysis where appropriate.

## 14. Evidence to Capture During VAPT

For each traffic-capture test, document:

- Device/emulator details
- Android version
- Application name and version
- Package name
- Proxy IP and port
- Capture method
- Certificate configuration
- Request URL
- HTTP method
- Request/response headers
- Relevant request/response body
- TLS observations
- Certificate pinning behavior
- Screenshots/PCAP where appropriate

Never commit real credentials, session tokens, API keys, OTPs, cookies or other sensitive production data to a public repository.
