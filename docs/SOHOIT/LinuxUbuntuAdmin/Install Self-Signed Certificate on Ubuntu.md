# Install Self-Signed Certificate on Ubuntu


Installing a company's self-signed certificate on **Ubuntu 24.04** is a two-step process: ensuring the certificate is in the correct format and then moving it to the system’s trust store.

### 1. Prepare the Certificate File
Ubuntu’s update tool requires the certificate to be in **PEM format** (text-based) and must have a **`.crt`** extension.

* **If your file is `.pem` or `.cer` (text):** Just rename it to `.crt`.
* **If your file is in DER (binary) format:** You must convert it using OpenSSL:
    ```bash
    openssl x509 -inform DER -in company-cert.cer -out company-cert.crt
    ```

### 2. Install the Certificate System-Wide
This will make the certificate trusted by system utilities like `curl`, `wget`, and the Docker daemon.

1.  **Create a directory** for local certificates (optional but recommended for organization):
    ```bash
    sudo mkdir -p /usr/local/share/ca-certificates/company
    ```
2.  **Copy the certificate** into that directory:
    ```bash
    sudo cp company-cert.crt /usr/local/share/ca-certificates/company/
    ```
3.  **Update the CA store**:
    ```bash
    sudo update-ca-certificates
    ```
    > **Success Check:** You should see output like: `Updating certificates in /etc/ssl/certs... 1 added, 0 removed; done.`

---

### 3. Application-Specific Steps
Some applications ignore the system trust store and require manual configuration:

#### **Browsers (Firefox / Chrome)**
Desktop browsers often use their own internal "NSS" database.
* **Firefox:** Settings → Privacy & Security → Certificates → View Certificates → **Import**. Check "Trust this CA to identify websites."
* **Chrome/Edge:** Settings → Privacy and security → Security → Manage certificates → **Authorities** → **Import**.

#### **Python (pip / requests)**
Python often uses the `certifi` package instead of the system store. To point it to the system store, add this to your `.bashrc` or `.zshrc`:
```bash
export REQUESTS_CA_BUNDLE=/etc/ssl/certs/ca-certificates.crt
```

#### **Java**
Java uses its own "keystore." You must import the cert into the JVM's `cacerts` file:
```bash
sudo keytool -import -alias company-ca -keystore $JAVA_HOME/lib/security/cacerts -file /usr/local/share/ca-certificates/company/company-cert.crt
```
*(The default password is usually `changeit`).*

#### **Docker**
If you are using this certificate for a private Docker registry, you must also place it here:
```bash
sudo mkdir -p /etc/docker/certs.d/your-registry.com:5000/
sudo cp company-cert.crt /etc/docker/certs.d/your-registry.com:5000/ca.crt
sudo systemctl restart docker
```
---
@end