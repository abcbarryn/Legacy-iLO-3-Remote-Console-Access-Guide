# Legacy iLO 3 Remote Console Access Guide (macOS & Windows 11)

A complete guide to configuring modern Java runtime environments (Java 8u291 through current updates like 8u501+) to successfully launch the legacy **iLO 3 Integrated Remote Console (.jnlp)** on both Apple Silicon Macs and Windows laptops.

---

## 🌐 Step 1: Install a Legacy-Compatible Browser
Modern mainstream browsers (Chrome, Edge, Safari) completely block old TLS versions and will refuse to load the iLO 3 web interface. Before anything else, download a browser that still contains the underlying code paths for legacy network handshakes.

### Option A: Pale Moon (Recommended)
[Pale Moon](https://palemoon.org) is actively maintained and distributed as a native binary for both Windows 11 and Apple Silicon/ARM64 macOS. It natively permits connections to legacy security environments.

### Option B: Firefox Portable (Windows Alternative)
If you prefer not to install a permanent application on Windows, download a dedicated standalone snapshot like **Firefox Portable (Version 45ESR through 52ESR)**. 
* *Note:* You must immediately go to `about:config` in the browser bar, search for `security.tls.version.min`, and force-change the integer value to **`1`** to allow TLS 1.0/1.1 connections.

---

## ⚠️ Step 2: Enable the iLO 3 AES Encryption Setting
Once you are logged into the iLO 3 Web Interface using your compatible browser, ensure the onboard cryptographic chip is enforcing proper cipher strengths. *Note: This option cannot be modified via the iLO SSH CLI, it must be changed in the web UI.*

1. Log into your iLO 3 Web Interface via Pale Moon or Portable Firefox.
2. Navigate to: **Administration** ➔ **Security** ➔ **Encryption**
3. Locate the setting: **Enforce AES/3DES Encryption**
4. Ensure this is **Enabled**. This forces the hardware to use `TLS_RSA_WITH_AES_128_CBC_SHA` with 128-bit keys, which allows modern client machines to communicate with it safely.

---

## 🛠️ Step 3: Modify the `java.security` File
Modern Java updates block legacy TLS protocols, RSA cipher families, and SHA-1 handshakes. You must explicitly unlock them in your client-side Java installation.

### 🍏 On macOS (Apple Silicon / Intel)
1. Open your Mac Terminal.
2. Locate and edit the file for your active runtime using `nano` (adjust the update number matching your system):
   ```bash
   sudo nano /Library/Java/JavaVirtualMachines/jdk1.8.0_501.jdk/Contents/Home/jre/lib/security/java.security
   ```
3. Locate the `jdk.tls.disabledAlgorithms=` block.
4. **Remove** the following restrictions from the active block text:
   * `TLSv1.1`
   * `TLS_RSA_*`
   * `rsa_pkcs1_sha1 usage HandshakeSignature`
5. Save and exit (**Ctrl+O**, **Enter**, **Ctrl+X**).

### 🪟 On Windows 11
1. Open the Start Menu, type **Notepad**, right-click, and select **Run as Administrator**.
2. Click **File** ➔ **Open** and switch the file filter dropdown (bottom-right) to **All Files (\*.\*)**.
3. Navigate to your active Java library directory:
   * Standard JRE path: `C:\Program Files\Java\jre1.8.0_501\lib\security\java.security`
4. Find the `jdk.tls.disabledAlgorithms=` block.
5. **Delete** the restrictions for `TLSv1.1`, `TLS_RSA_*`, and `rsa_pkcs1_sha1 usage HandshakeSignature`.
6. Save and close.

---

## ⚙️ Step 4: Configure Client Deployment Settings
Modifying `java.security` enables core cryptographic capabilities, but the Java client runtime engine must be commanded to deploy them.

### 🍏 On macOS
1. Open your Mac **System Settings** (or System Preferences).
2. Click the **Java** icon at the very bottom of the sidebar/window to open the **Java Control Panel**.
3. Go to the **Advanced** tab.
4. Scroll down to **Advanced Security Settings**.
5. Check the box for **Use TLS 1.1** (and ensure **Use TLS 1.2** is checked).
6. Click **Apply** and **OK**.
* *Note: If the checkboxes are missing on your specific Java build, you can manually append `deployment.security.TLSv1.1=true` to `~/Library/Application Support/Oracle/Java/Deployment/deployment.properties`.*

### 🪟 On Windows 11
1. Open the Start Menu, type **Configure Java**, and press **Enter**.
2. Go to the **Advanced** tab.
3. Scroll down to **Advanced Security Settings**.
4. Check the box for **Use TLS 1.1** (and ensure **Use TLS 1.2** is also checked).
5. Click **Apply** and **OK**.

---

## 🚀 Launch Pipeline
1. Load your iLO 3 landing page using Pale Moon or Portable Firefox.
2. Click **Launch webstart** under the Integrated Remote Console section to download the `.jnlp` file.
3. Your local OS file associations will intercept the file and automatically pass the execution stream to your updated, unblocked local `javaws` engine.
