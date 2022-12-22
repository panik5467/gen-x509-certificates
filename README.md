# gen-x509-certificates

## Step 1 - Initial Setup

* Start Bash.
* `cd` to the directory you want to run in.  All files will be created as children of this directory.
* `cp *.cnf` and `cp *.sh` from the directory this .MD file is located into your working directory.
* `chmod 700 certGen.sh`

## Step 2 - Create the certificate chain

First you need to create a CA and an intermediate certificate signer that chains back to the CA.

* Run `./certGen.sh create_root_and_intermediate`

Next, go to Azure IoT Hub and navigate to Certificates.  Add a new certificate, providing the root CA file when prompted.  (`.\RootCA.pem` in PowerShell and `./certs/azure-iot-test-only.root.ca.cert.pem` in Bash.)

## Step 3 - Proof of Possession

*Optional - Only perform this step if you're setting up CA Certificates and proof of possession.  For simple device certificates, such as Edge certificates, skip to the next step.*

Now that you've registered your root CA with Azure IoT Hub, you'll need to prove that you actually own it.

Select the new certificate that you've created and navigate to and select  "Generate Verification Code".  This will give you a verification string you will need to place as the subject name of a certificate that you need to sign.  For our example, assume IoT Hub verification code was "106A5SD242AF512B3498BD6098C4941E66R34H268DDB3288", the certificate subject name should be that code. See below example PowerShell and Bash scripts

* Run `./certGen.sh create_verification_certificate 106A5SD242AF512B3498BD6098C4941E66R34H268DDB3288`

In both cases, the scripts will output the name of the file containing `"CN=106A5SD242AF512B3498BD6098C4941E66R34H268DDB3288"` to the console.  Upload this file to IoT Hub (in the same UX that had the "Generate Verification Code") and select "Verify".

## Step 4 - Create a new device

Finally, let's create an application and corresponding device on IoT Hub that shows how CA Certificates are used.

On Azure IoT Hub, navigate to the IoT Devices section, or launch Azure IoT Explorer.  Add a new device (e.g. `mydevice`), and for its authentication type chose "X.509 CA Signed".  Devices can authenticate to IoT Hub using a certificate that is signed by the Root CA from Step 2.

#### IoT Leaf Device

* Run `./certGen.sh create_device_certificate mydevice` to create the new device certificate, which will chain directly to the root CA. 
  * If the device certificate needs to be signed by an intermediate CA instead, run `./certGen.sh create_device_certificate_from_intermediate mydevice` to chain the device certificate to the intermediate certificate. __Note:__ When using chains with more than one level, the device must send all chain certificates up to but not including the one configured on the service during TLS handshake. This is required for the service to create a valid chain.

* `cd ./certs && cat new-device.cert.pem azure-iot-test-only.intermediate.cert.pem azure-iot-test-only.root.ca.cert.pem > new-device-full-chain.cert.pem` to get the public key.

## Step 5 - Cleanup

### **cleanup.sh**

Bash outputs certificates to the current working directory, so there is no analogous system cleanup needed.

[the official documentation]: https://docs.microsoft.com/azure/iot-hub/iot-hub-security-x509-get-started
[Edge gateway creation documentation]: https://docs.microsoft.com/azure/iot-edge/how-to-create-gateway-device
