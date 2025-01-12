# Azure-Key-Vault
Azure Key Vault Project

Here is a detailed guide to creating and using **Azure Key Vault** in a project, including steps and example code. This project demonstrates securely storing secrets and accessing them via an application.

---

### **Project: Using Azure Key Vault to Secure Application Secrets**

#### **Objective**
- Store sensitive data (e.g., API keys, passwords) in Azure Key Vault.
- Access secrets securely in an application using Azure SDK.

---

### **Steps**

#### **1. Prerequisites**
- **Azure Account**: Ensure you have access to Azure.
- **Azure CLI**: Installed and configured on your machine.
- **Azure SDK**: Relevant SDK for your application language (e.g., Python, .NET).
- **Resource Group**: Create one if you don’t have it already.

---

#### **2. Create an Azure Key Vault**

Run the following commands to create a Key Vault:

```bash
# Login to Azure
az login

# Set your subscription (if you have multiple subscriptions)
az account set --subscription "<SUBSCRIPTION_ID>"

# Create a resource group (if not created)
az group create --name MyResourceGroup --location eastus

# Create the Key Vault
az keyvault create --name MyKeyVault --resource-group MyResourceGroup --location eastus
```

---

#### **3. Add Secrets to Key Vault**

Add secrets to the Key Vault:

```bash
# Add a secret
az keyvault secret set --vault-name MyKeyVault --name "MySecretName" --value "MySecretValue"

# List all secrets
az keyvault secret list --vault-name MyKeyVault
```

---

#### **4. Configure Access Policies**

Grant your application or user access to the Key Vault:

```bash
# Grant access to a specific user or app
az keyvault set-policy --name MyKeyVault --upn "user@domain.com" --secret-permissions get list
```

For service principals or managed identities:

```bash
az keyvault set-policy --name MyKeyVault --spn "<SERVICE_PRINCIPAL_ID>" --secret-permissions get list
```

---

#### **5. Access Key Vault in Code**

Below are examples for Python and .NET.

---

##### **Python Code**

Install the Azure Key Vault SDK:

```bash
pip install azure-identity azure-keyvault-secrets
```

**Code Example:**

```python
from azure.identity import DefaultAzureCredential
from azure.keyvault.secrets import SecretClient

# Key Vault URL
key_vault_url = "https://MyKeyVault.vault.azure.net/"

# Authenticate with DefaultAzureCredential
credential = DefaultAzureCredential()
client = SecretClient(vault_url=key_vault_url, credential=credential)

# Retrieve a secret
secret_name = "MySecretName"
retrieved_secret = client.get_secret(secret_name)
print(f"Secret Value: {retrieved_secret.value}")
```

---

##### **.NET Code (C#)**

Install the Azure SDK for .NET:

```bash
dotnet add package Azure.Identity
dotnet add package Azure.Security.KeyVault.Secrets
```

**Code Example:**

```csharp
using System;
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;

class Program
{
    static void Main(string[] args)
    {
        string keyVaultUrl = "https://MyKeyVault.vault.azure.net/";
        string secretName = "MySecretName";

        var client = new SecretClient(new Uri(keyVaultUrl), new DefaultAzureCredential());
        KeyVaultSecret secret = client.GetSecret(secretName);

        Console.WriteLine($"Secret Value: {secret.Value}");
    }
}
```

---

#### **6. Test Your Application**

Run the application and verify that it fetches the secret value from Azure Key Vault.

---

#### **7. Secure Access with Managed Identity (Optional)**

To eliminate the need for client secrets or keys, configure **Managed Identity** for your application:

- **Enable Managed Identity** for your Azure resource (e.g., VM, App Service).
- Add the Managed Identity to the Key Vault's access policy:

```bash
az keyvault set-policy --name MyKeyVault --object-id <MANAGED_IDENTITY_OBJECT_ID> --secret-permissions get list
```

---

### **Summary**
- Created an Azure Key Vault.
- Added secrets and configured access policies.
- Retrieved secrets in a Python or .NET application.
- Secured access with Managed Identity.

Let me know if you need further customization or help!
