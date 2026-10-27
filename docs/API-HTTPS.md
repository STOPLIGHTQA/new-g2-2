---
tags: [Server, Security]
stoplight-id: m9oeyqpqpivmn
---
# HTTPS Support

## Overview

System API Server serves its REST API exclusively over HTTPS. Therefore, a TLS certificate is necessary in order to validate the server's identity. Two methods are provided by System API Server to present a TLS certificate to its clients:
1. **An automatically generated self-signed certificate for `localhost`(Default)**
2. **User-provided Custom TLS certificate (in PEM format)**

Each of these has there own benefits and drawbacks, which are outlined below.

## Which Certificate Option is Right For Me?
### Automatically Generated Self-Signed TLS Certificate for `localhost`
#### Benefits
1. Get up and running quickly for same-machine usage.

#### Drawbacks
1. Not securely accessible from outside the machine (e.g. turning off or dismissing TLS certificate validation warnings in clients)
2. Requires manually trusting the TLS certificate even for same-machine usage to avoid certificate validation warnings in your HTTP clients (e.g. Chrome, cURL, your application code)

### Custom TLS Certificate
#### Benefits
1. Secure cross-machine communication
2. A certificate issued by a known and trusted certificate authority (CA) will not require manually "trusting" the certificate in each and every HTTP client. 

#### Drawbacks
1. Additional configuration and preparation of the certificate is required.
2. The server administrator is responsible for securing the TLS certificate private key file, instead of System API Server.

This applies even when communicating on the loopback interface on the same machine.
## Using the Automatically Generated Self-Signed TLS Certificate
### Trusting the Self-Signed Certificate

When you install System API Server on your computer and run it for the first time, upon opening the embedded documentation UI in your browser, you will notice an error like this:
![Windows-PrivacyErrorInChrome.png](https://shure.widen.net/content/cvwynmyy9u/original/PrivacyErrorInChrome.png?u=dsog5w&download=true)

In order to resolve this and allow your browser and any other applications or clients to trust System API Server, you will need to install the server certificate into your trust store. The means by which to achieve this varies by operating system.

<!-- theme: info -->

> #### An Alternative Approach
> In lieu of adding the TLS certificate to your trust store, you may choose to disable certificate validation in your client, which should bypass any errors encountered. If you would like to do that, you can skip the rest of this guide.

<!-- theme: warning -->
> #### Accessing System API Server from Other Computers
> Because the TLS certificate is issued for `localhost` (and the IPv4 equivalent: `127.0.0.1`), even if you install the certificate on other machines, you will still get errors as the DNS name or IP address by which you access System API Server will not be local.

#### Windows
1. From a command prompt, run *certmgr* to open Certificate Manager. Navigate to **Trusted Root Certification Authorities** -> **Certificates**:
    ![Windows-CertificateManager-TrustedRootCAs-1.png](https://shure.widen.net/content/xul9ey0us8/original/Windows-CertificateManager-TrustedRootCAs-1.png?u=dsog5w&download=true)

2. Right-click the **Certificates** item and choose **All Tasks** -> **Import...**

3. Click **Next**, then choose the server certificate file (located at **C:\ProgramData\Shure\SystemAPI\Standalone\Security\systemapicertificate.cer**) and click **Next** again, then ensure the selected trust store is **Trusted Root Certification Authorities** and click **Finish**. The certificate store should now look as follows:
    ![Windows-CertificateManager-TrustedRootCAs-2.png](https://shure.widen.net/content/0ickpslqsv/original/Windows-CertificateManager-TrustedRootCAs-2.png?u=dsog5w&download=true)

4. Restart your browser to verify the error has gone away (you should now see the embedded documentation UI with an indication that the site is trusted):
    ![Windows-ChromeSecure.png](https://shure.widen.net/content/chtlheoa0q/original/Windows-ChromeSecure.png?u=dsog5w&download=true)

#### MacOS
1. In Finder, navigate to the directory containing the certificate file.

2. Next to the Finder window, open the **Keychain Access** app and select the **System** keychain:
    ![MacOS-KeychainAccess-System.png)](https://shure.widen.net/content/ylrlptt7ch/original/MacOS-KeychainAccess-System.png?u=dsog5w&download=true)

3. Drag the certificate file into the keychain:
    ![MacOS-KeychainAccess-DragAndDrop.png](https://shure.widen.net/content/ach3oxwmnv/original/MacOS-KeychainAccess-DragAndDrop.png?u=dsog5w&download=true)

4. Right-click the certificate in the keychain and choose **Get Info...** to open the certificate details, then expand the **Trust** section.

5. In the Secure **Sockets Layer (SSL)** dropdown, choose **Always Trust**:
    ![MacOS-KeychainAccess-TrustTheCertificate.png](https://shure.widen.net/content/zxjfmowj0d/original/MacOS-KeychainAccess-TrustTheCertificate.png?u=dsog5w&download=true)
6. Exit out of **Keychain Access**. You will be prompted to enter your password as updating a keychain is an action requiring administrative privileges.

7. Restart your browser to verify the error has gone away (you should now see the embedded documentation UI with an indication that the site is trusted):
    ![MacOS-ChromeSecure.png](https://shure.widen.net/content/zcb4m3fgsy/original/MacOS-ChromeSecure.png?u=dsog5w&download=true)

## Using the Custom TLS Certificate
### Requirements
The TLS certificate must...
1. be specified in PEM format (with a .pem extension)
2. be readable by System API Server
3. Be reliably accessible on each HTTP request (e.g. use of an inconsistently available network drive or share is not advised)

### Replacing the provided certificate
The following steps can be used if the provided certificate is nearing expiration, or needs to be reissued due to other factors.
1. Navigate to "Services" using Windows Search. Open the application.
2. Search for "Shure System API Service - Standalone". Stop the service.
3. Replace the TLS certificate and it's corresponding private key at the file paths previously provided. By default, this will be written to `C:\Program Files\Shure\System API\Standalone\appsettings\appsettings.json5` under the `Application` object's `Security` object (a.k.a `Application.Security.TlsCertificateFilePath` and `Application.Security.TlsCertificatePrivateKeyFilePath` in JSONPath notation, respectively).
4. Restart the service
