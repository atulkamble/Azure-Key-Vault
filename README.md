# 🚀 **AZURE KEY VAULT 

Includes:

1. **Create Key Vault (CLI, ARM, PowerShell)**
2. **Publish Secret as Artifact using YAML Pipeline**
3. **Use Key Vault secret in Variable Group**
4. **Access Key Vault Secret in ARM Template**
5. **Key Vault with Azure Data Factory**
6. **Access Secret from Key Vault in Azure Pipeline (Service Connection + RBAC)**
7. **Latest Key Vault UI Changes**
8. **Tests & verification**

---

# 1️⃣ **Create Azure Key Vault – Commands**

## **Azure CLI**

```bash
az group create -n rg-kv-demo -l eastus

az keyvault create \
  -n kv-demo-atul \
  -g rg-kv-demo \
  --sku standard

az keyvault secret set \
  --name dbPassword \
  --vault-name kv-demo-atul \
  --value "P@ssword123!"
```

## **PowerShell**

```powershell
New-AzResourceGroup -Name rg-kv-demo -Location eastus
New-AzKeyVault -Name kv-demo-atul -ResourceGroupName rg-kv-demo -Location eastus

Set-AzKeyVaultSecret -VaultName kv-demo-atul -Name "dbPassword" -SecretValue (ConvertTo-SecureString "P@ssword123!" -AsPlainText -Force)
```

---

# 2️⃣ **Publish Key Vault Secret as Artifact using Azure DevOps YAML Pipeline**

### **Goal:**

Fetch secret → Write to file → Publish as pipeline artifact.

### **azure-pipelines.yml**

```yaml
trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

variables:
  kvName: 'kv-demo-atul'
  secretName: 'dbPassword'

steps:
- task: AzureCLI@2
  name: FetchSecret
  inputs:
    azureSubscription: 'SERVICE-CONNECTION-NAME'
    scriptType: bash
    scriptLocation: inlineScript
    inlineScript: |
      echo "Fetching secret from Key Vault..."
      secretValue=$(az keyvault secret show --vault-name $kvName --name $secretName --query value -o tsv)
      echo $secretValue > secret.txt

- task: PublishPipelineArtifact@1
  inputs:
    targetPath: 'secret.txt'
    artifact: 'keyvault-secret'
```

---

# 3️⃣ **Define Secret as Variable in Variable Group (Azure DevOps)**

### **Step-by-step**

1. Go to **Pipelines → Library → Variable Group → + Add variable group**
2. Enable **Link secrets from Azure Key Vault**
3. Select:

   * Your Subscription
   * Your Key Vault (`kv-demo-atul`)
4. Choose secrets to include

### **Use in YAML**

```yaml
variables:
- group: kv-variable-group-demo

steps:
- script: |
    echo "Azure Key Vault Secret: $(dbPassword)"
```

---

# 4️⃣ **Access Key Vault Secret in ARM Template**

### **arm-template.json**

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "kvName": {
      "type": "string"
    },
    "secretName": {
      "type": "string"
    }
  },
  "resources": [],
  "outputs": {
    "secretValue": {
      "type": "string",
      "value": "[reference(resourceId('Microsoft.KeyVault/vaults/secrets', parameters('kvName'), parameters('secretName')), '2019-09-01').value]"
    }
  }
}
```

### **Deployment**

```bash
az deployment group create \
  -g rg-kv-demo \
  --template-file arm-template.json \
  --parameters kvName=kv-demo-atul secretName=dbPassword
```

---

# 5️⃣ **Azure Key Vault with Azure Data Factory**

## **Create Linked Service with Key Vault Integration**

### **linkedService.json**

```json
{
  "name": "AzureSqlDatabaseLS",
  "type": "Microsoft.DataFactory/factories/linkedservices",
  "properties": {
    "type": "AzureSqlDatabase",
    "typeProperties": {
      "connectionString": "Server=tcp:sql-demo.database.windows.net;Database=dbdemo;",
      "password": {
        "type": "AzureKeyVaultSecret",
        "store": {
          "referenceName": "KeyVaultLS",
          "type": "LinkedServiceReference"
        },
        "secretName": "dbPassword"
      }
    }
  }
}
```

### **Create Key Vault Linked Service**

```json
{
  "name": "KeyVaultLS",
  "type": "Microsoft.DataFactory/factories/linkedservices",
  "properties": {
    "type": "AzureKeyVault",
    "typeProperties": {
      "baseUrl": "https://kv-demo-atul.vault.azure.net/"
    }
  }
}
```

---

# 6️⃣ **Access Secret from Azure Key Vault using Azure Pipeline**

## **YAML Pipeline**

```yaml
trigger:
- main

pool:
  vmImage: ubuntu-latest

variables:
  - name: keyVaultName
    value: kv-demo-atul

steps:
- task: AzureKeyVault@2
  inputs:
    azureSubscription: 'SERVICE-CONNECTION-NAME'
    KeyVaultName: '$(keyVaultName)'
    SecretsFilter: '*'
    RunAsPreJob: false

- script: |
    echo "DB Password: $(dbPassword)"
```

✔ Automatically maps all secrets as variables
✔ Secret names = pipeline variable names

---

# 7️⃣ **Important Key Vault UI Update (2024–2025)**

### **✔ New UI Changes**

* **Secrets, Keys, Certificates now under “Object” section**
* “Generate/Import” button replaced with **“Create”**
* Access Policies moved under:
  **Settings → Access configuration**
* Now strongly recommends **Azure RBAC** (instead of Access Policies)
* “Purge Protection” defaults **ON**
* “Soft Delete” is **always enabled**, cannot disable

### **Recommendation**

Use **RBAC + Managed Identity** (not Access Policies).

---

# 8️⃣ **Tests & Verification Scripts**

### **Test: Get all secrets**

```bash
az keyvault secret list --vault-name kv-demo-atul -o table
```

### **Test: Retrieve specific secret**

```bash
az keyvault secret show \
  --vault-name kv-demo-atul \
  --name dbPassword \
  --query value -o tsv
```

### **Test: Using Managed Identity**

```bash
curl "http://169.254.169.254/metadata/identity/oauth2/token?resource=https://vault.azure.net&api-version=2018-02-01" \
  -H Metadata:true
```

### **Test from Azure Pipeline**

```yaml
- script: |
    if [ -z "$(dbPassword)" ]; then
      echo "Secret Not Found!"
      exit 1
    else
      echo "Secret Loaded Successfully!"
    fi
```

---

# 🎁 **BONUS: Key Vault Secrets in PowerShell Script**

```powershell
$kvName = "kv-demo-atul"
$secret = az keyvault secret show --vault-name $kvName --name dbPassword --query value -o tsv
Write-Host "Secret value: $secret"
```

---

# ✅ **Delivered:**

✔ YAML Pipelines
✔ ARM Template integration
✔ Data Factory integration
✔ Publishing secret as artifact
✔ Variable groups
✔ CLI + PowerShell
✔ New UI updates
✔ Test scripts

---
