Remote Management Web Server Troubleshooting - ASRock Rack GENOAD8X-2T/BCM

Overview:
The ASRock Rack GENOAD8X-2T/BCM motherboard may occasionally experience a firmware-related issue where the remote management web server becomes unresponsive. This document outlines the troubleshooting steps to resolve this known firmware bug which can block web server functionality.
If you encounter issues accessing the remote management interface through the web browser, please follow the procedure below to restore functionality.

Note this is not official solution and updating firmware may be prefered option when available .. 

Updating the BMC password on the ASRock Rack GENOAD8X-2T/BCM motherboard using the Redfish API. This guide includes initial checks, retrieval of the ETag value, and meeting password requirements.

---

## Documentation: Updating BMC Password Using Redfish API

### Overview

This document outlines the steps to update the BMC password on an ASRock Rack GENOAD8X-2T/BCM motherboard using Redfish API commands, which can be necessary when initial access requires a password update. This approach involves verifying the current account status, retrieving the necessary ETag for validation, and meeting specific password requirements.

### Requirements

- **curl**: Command-line tool for making HTTP requests.
- **Redfish API access**: Enabled on the BMC.
- **New Password**: Must meet minimal security requirements (e.g., length and complexity).

---

### Step-by-Step Guide

#### Step 1: Verify Account Status

1. First, check if a password update is required by accessing the AccountService endpoint in a browser or with curl:

   **Command**:
   ```bash
   curl -k -u admin:admin https://<BMC_IP>/redfish/v1/AccountService/Accounts/4
   ```

   **Example**:
   ```bash
   curl -k -u admin:admin https://192.168.88.182/redfish/v1/AccountService/Accounts/4
   ```

2. **Expected Output**: If a password change is required, you’ll see a response with a message like:
   ```
   "The password provided for this account must be changed before access is granted."
   ```

#### Step 2: Retrieve the ETag Value

1. Retrieve the ETag value associated with the account, which is required to validate the password update request.

   **Command**:
   ```bash
   curl -k -u admin:admin -I https://<BMC_IP>/redfish/v1/AccountService/Accounts/4
   ```

   **Example**:
   ```bash
   curl -k -u admin:admin -I https://192.168.88.182/redfish/v1/AccountService/Accounts/4
   ```

2. **Expected Output**: Look for the `ETag` header in the response headers, which will look something like:
   ```
   ETag: "W/\"1640995245\""
   ```
   
   **Note**: Copy this `ETag` value (including the quotes) as it will be needed in the next step.

#### Step 3: Update the Password

1. Set a new password that meets minimum length and complexity requirements (e.g., 8+ characters, including uppercase, lowercase, and numbers).

2. Use the following curl command to update the password, replacing `<etag_value>` with the actual ETag retrieved in Step 2 and `<new_password>` with the desired new password.

   **Command**:
   ```bash
   curl -k -X PATCH -u admin:admin \
        -H "Content-Type: application/json" \
        -H "Accept: application/json" \
        -H 'If-Match: "<etag_value>"' \
        -d '{"Password": "<new_password>"}' \
        https://<BMC_IP>/redfish/v1/AccountService/Accounts/4
   ```

   **Example**:
   ```bash
   curl -k -X PATCH -u admin:admin \
        -H "Content-Type: application/json" \
        -H "Accept: application/json" \
        -H 'If-Match: "1640995245"' \
        -d '{"Password": "Genova123!"}' \
        https://192.168.88.182/redfish/v1/AccountService/Accounts/4
   ```

   **Explanation**:
   - `-X PATCH`: Specifies the HTTP method as PATCH.
   - `If-Match`: Provides the ETag value for validation.
   - `-d '{"Password": "<new_password>"}'`: Sets the new password in JSON format.

#### Step 4: Verify Password Update

1. To confirm the password was successfully updated, test access to the AccountService endpoint using the new credentials:

   **Command**:
   ```bash
   curl -k -u admin:<new_password> https://<BMC_IP>/redfish/v1/AccountService/Accounts/4
   ```

   **Example**:
   ```bash
   curl -k -u admin:Genova123! https://192.168.88.182/redfish/v1/AccountService/Accounts/4
   ```

2. **Expected Output**: If the login is successful, the account details will be returned, confirming the new password is active.

---

### Summary of Commands

1. **Verify Account Needs Update**:
   ```bash
   curl -k -u admin:admin https://192.168.88.182/redfish/v1/AccountService/Accounts/4
   ```

2. **Retrieve ETag**:
   ```bash
   curl -k -u admin:admin -I https://192.168.88.182/redfish/v1/AccountService/Accounts/4
   ```

3. **Update Password**:
   ```bash
   curl -k -X PATCH -u admin:admin \
        -H "Content-Type: application/json" \
        -H "Accept: application/json" \
        -H 'If-Match: "1640995245"' \
        -d '{"Password": "Genova123!"}' \
        https://192.168.88.182/redfish/v1/AccountService/Accounts/4
   ```

4. **Verify New Password**:
   ```bash
   curl -k -u admin:Genova123! https://192.168.88.182/redfish/v1/AccountService/Accounts/4
   ```

---

### Important Notes

- **Password Requirements**: The BMC may enforce minimum password requirements, typically 8+ characters including uppercase, lowercase, and numeric characters.
- **Troubleshooting**: If the ETag value changes, re-run Step 2 to retrieve the updated ETag and retry the password update command.
- **BMC Timeout**: Some BMCs may take longer to apply updates. If you encounter timeouts, try the commands again.

This guide provides the full command sequence to update a BMC password via Redfish API on the ASRock Rack GENOAD8X-2T/BCM. This approach should work for similar enterprise motherboards with Redfish support.