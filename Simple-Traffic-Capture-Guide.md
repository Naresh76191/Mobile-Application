# Simple Mobile Traffic Capture Guide

This guide explains how to connect an Android device to **Burp Suite** or **HTTP Toolkit** and capture application traffic.

> Use this only for applications and devices you are authorized to test.

---

## 1. Prerequisites

Before starting, keep these ready:

- Android phone
- USB cable
- Laptop/PC
- **Burp Suite** installed
- **HTTP Toolkit** installed
- Android phone and laptop connected to the same Wi-Fi when using Wi-Fi proxy
- USB Debugging enabled if required

---

# Part A — Burp Suite Certificate

## 2. Start Burp Suite

1. Open **Burp Suite**.
2. Go to **Proxy → Proxy settings**.
3. Make sure a proxy listener is running.

Usually:

```text
127.0.0.1:8080
```

For a physical Android device, the listener should normally be configured to accept connections from other devices, for example:

```text
0.0.0.0:8080
```

---

## 3. Find Laptop IP Address

On Windows, open Command Prompt:

```cmd
ipconfig
```

Find the **IPv4 Address** of the Wi-Fi adapter.

Example:

```text
192.168.1.10
```

You will use this IP address on the Android phone.

---

## 4. Download Burp Certificate

Connect the Android phone to the same Wi-Fi network as the laptop.

On the Android phone, open a browser and go to:

```text
http://burp
```

You should see the Burp certificate download page.

Download the **CA Certificate**.

The certificate may be downloaded as a file such as:

```text
cacert.der
```

---

## 5. Import Burp Certificate into Android

The exact menu can be different depending on the Android version.

Generally:

1. Open **Settings**.
2. Search for **Certificate**.
3. Open **Install a certificate** or **Install from storage**.
4. Select the downloaded Burp certificate.
5. If asked for the certificate type, select **CA Certificate**.
6. Complete the installation.

After installation, the Burp CA certificate is trusted by the device's certificate store.

> Important: Some Android applications do not trust user-installed CA certificates. In that case, installing the certificate alone may not be enough to capture HTTPS traffic.

---

# Part B — Connect Android to Burp

## 6. Configure Android Proxy

On the Android phone:

1. Open **Settings → Wi-Fi**.
2. Select the connected Wi-Fi network.
3. Open **Proxy** settings.
4. Select **Manual**.
5. Enter the laptop IP address.

Example:

```text
Proxy hostname: 192.168.1.10
Proxy port: 8080
```

6. Save the settings.

---

## 7. Test Burp Connection

Open the browser on the Android phone.

Try opening:

```text
http://example.com
```

Now go to:

**Burp Suite → Proxy → HTTP history**

You should see the request.

If you can see the request, the Android phone is connected to Burp.

---

# Part C — HTTP Toolkit Certificate

## 8. Open HTTP Toolkit

1. Install and open **HTTP Toolkit** on the laptop.
2. Select the **Android device** option.
3. Follow the setup instructions shown by HTTP Toolkit.

HTTP Toolkit will provide the steps for connecting the Android device.

---

## 9. Install HTTP Toolkit Certificate

Depending on the connection method, HTTP Toolkit will provide a certificate installation option.

On the Android device:

1. Download/open the HTTP Toolkit CA certificate when prompted.
2. Go to **Settings**.
3. Search for **Certificate**.
4. Select **Install a certificate** or **Install from storage**.
5. Select the HTTP Toolkit certificate.
6. Install it as a **CA Certificate** if Android asks for the certificate type.

The exact Android menu can be different between Android versions.

---

# Part D — Connect Android to HTTP Toolkit

## 10. Connect Using USB

1. Enable **Developer Options** on the Android phone.
2. Enable **USB Debugging**.
3. Connect the phone to the laptop using USB.
4. Allow USB debugging when the phone asks.
5. Open HTTP Toolkit.
6. Select the Android device connection option.
7. Follow the instructions shown by HTTP Toolkit.
8. Install the certificate if requested.
9. Open the target application.

The application traffic should start appearing in HTTP Toolkit.

---

## 11. Connect Using Wi-Fi

1. Connect the Android phone and laptop to the **same Wi-Fi network**.
2. Open HTTP Toolkit.
3. Select the Android device connection option.
4. Follow the pairing instructions shown by HTTP Toolkit.
5. Install the HTTP Toolkit certificate on the phone.
6. Start the target application.
7. Perform an action such as login or loading a page.
8. Check HTTP Toolkit.

The requests should appear in HTTP Toolkit.

---

# 12. What You Should See

For a successful connection, you should see information such as:

```text
GET /api/login
POST /api/user
GET /api/profile

Request Headers
Request Body

Response Status
Response Headers
Response Body
```

You can then inspect the API request and response.

---

# 13. If Traffic Is Not Coming

Check these things:

### Burp

- Is Burp running?
- Is the listener running on port 8080?
- Is the Android proxy IP correct?
- Is the Android proxy port correct?
- Are both devices on the same Wi-Fi?
- Is the Burp CA certificate installed?

### HTTP Toolkit

- Is HTTP Toolkit running?
- Is the Android device connected?
- Is the HTTP Toolkit certificate installed?
- Is the correct application being tested?

### HTTPS traffic is missing

If normal traffic works but HTTPS traffic does not:

1. Check the CA certificate.
2. Check whether the application trusts user certificates.
3. Check whether the application uses **certificate pinning**.
4. Check whether the application uses a custom certificate validation method.

For authorized testing, tools such as **Frida** can be used to investigate certificate validation and pinning.

---

# 14. Simple Flow to Remember

## Burp

```text
Android
   ↓
Install Burp CA Certificate
   ↓
Set Android Proxy
   ↓
Laptop IP : 8080
   ↓
Burp Suite
   ↓
View HTTP History
```

## HTTP Toolkit

```text
Android
   ↓
Connect to HTTP Toolkit
   ↓
Install HTTP Toolkit CA Certificate
   ↓
Start Application
   ↓
HTTP Toolkit
   ↓
View Requests
```

---

## 15. Important Note

Installing a CA certificate does **not** guarantee that every application will allow HTTPS interception.

An application may use:

- Certificate pinning
- Custom TrustManager
- Custom TLS validation
- Native certificate validation

If this happens, first document the normal interception result. Then perform additional authorized testing to determine why the application does not trust the interception certificate.
