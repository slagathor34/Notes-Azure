# Secure Cloud Automation - Azure

## Overview

This guide explains how to use the Azure Python SDK to initiate a Privileged Identity Management (PIM) request to elevate an automation user. Once elevated, the user can perform Azure REST calls or VM automation with the elevated credentials.

## Prerequisites

- Python 3.6 or later
- Azure subscription
- Azure Active Directory (AAD) tenant
- Privileged Identity Management (PIM) enabled in your Azure AD tenant
- Service principal with the necessary permissions
- Ansible AWX installed and configured
- Ansible execution environment with necessary dependencies

## Installation

1. Install the Azure Python SDK:

    ```sh
    pip install azure-identity azure-mgmt-compute azure-mgmt-resource
    ```

2. Set up environment variables for your Azure credentials:

    ```sh
    export AZURE_CLIENT_ID="your-client-id"
    export AZURE_CLIENT_SECRET="your-client-secret"
    export AZURE_TENANT_ID="your-tenant-id"
    ```

## Usage

1. **Authenticate with Azure:**

    ```python
    from azure.identity import ClientSecretCredential
    import os

    credential = ClientSecretCredential(
        tenant_id=os.environ["AZURE_TENANT_ID"],
        client_id=os.environ["AZURE_CLIENT_ID"],
        client_secret=os.environ["AZURE_CLIENT_SECRET"]
    )
    ```

2. **Initiate a PIM request:**

    ```python
    from azure.mgmt.authorization import AuthorizationManagementClient

    subscription_id = "your-subscription-id"
    resource_group = "your-resource-group"
    role_definition_id = "your-role-definition-id"
    principal_id = "your-principal-id"

    authorization_client = AuthorizationManagementClient(credential, subscription_id)

    role_assignment_params = {
        "role_definition_id": role_definition_id,
        "principal_id": principal_id,
        "scope": f"/subscriptions/{subscription_id}/resourceGroups/{resource_group}"
    }

    role_assignment = authorization_client.role_assignments.create(
        scope=role_assignment_params["scope"],
        role_assignment_name="your-role-assignment-name",
        parameters=role_assignment_params
    )

    print(f"Role assignment created: {role_assignment.id}")
    ```

3. **Perform Azure REST calls or VM automation:**

    ```python
    from azure.mgmt.compute import ComputeManagementClient

    compute_client = ComputeManagementClient(credential, subscription_id)

    # Example: List all VMs in a resource group
    vms = compute_client.virtual_machines.list(resource_group_name=resource_group)
    for vm in vms:
        print(vm.name)
    ```

## Integration with Ansible AWX

1. **Create an Ansible Playbook:**

    Create a playbook that includes the Python script for PIM request and Azure automation.

    ```yaml
    ---
    - name: Azure PIM Automation
      hosts: localhost
      tasks:
        - name: Run Azure PIM script
          script: /path/to/your/azure_pim_script.py
    ```

2. **Set up Execution Environment:**

    Ensure your execution environment has the necessary Python dependencies. You can create a custom Docker image with the required packages.

    ```Dockerfile
    FROM ansible/awx-ee:latest

    RUN pip install azure-identity azure-mgmt-compute azure-mgmt-resource
    ```

3. **Configure AWX Job Template:**

    - Create a new job template in AWX.
    - Select the inventory and project containing your playbook.
    - Choose the custom execution environment.
    - Set the necessary environment variables in the job template.

4. **Run the Job:**

    Execute the job template in AWX to run the playbook, which will initiate the PIM request and perform the desired Azure automation tasks.

## Alignment with NIST 800-53 v5

The process described aligns with several controls in NIST 800-53 v5, including:

- **AC-2 Account Management:** Ensures that only authorized users have access to elevated privileges.
- **AC-5 Separation of Duties:** By using PIM, it ensures that elevated privileges are granted only when necessary and for a limited time.
- **AC-6 Least Privilege:** Limits the privileges of users to the minimum necessary for their roles.
- **AU-2 Audit Events:** The process can be audited to ensure compliance and detect any unauthorized access.

The process also aligns with several controls in NIST 800-171, including:

- **3.1.1 Limit system access:** Ensures that only authorized users have access to elevated privileges.
- **3.1.2 Limit system access to the types of transactions and functions:** By using PIM, it ensures that elevated privileges are granted only when necessary and for a limited time.
- **3.1.5 Least privilege:** Limits the privileges of users to the minimum necessary for their roles.
- **3.3.1 Create, protect, and retain audit records:** The process can be audited to ensure compliance and detect any unauthorized access.


## Mitigation of Attack Vectors in the MITRE Framework

This approach helps mitigate several attack vectors identified in the MITRE ATT&CK framework:

- **Initial Access (T1078):** By using PIM, the risk of compromised credentials leading to unauthorized access is reduced.
- **Privilege Escalation (T1068):** PIM ensures that privilege escalation is controlled and monitored.
- **Defense Evasion (T1070):** Regular auditing and monitoring of PIM activities help detect and prevent attempts to evade defenses.
- **Credential Access (T1212):** By limiting the duration and scope of elevated privileges, the risk of credential theft is minimized.
