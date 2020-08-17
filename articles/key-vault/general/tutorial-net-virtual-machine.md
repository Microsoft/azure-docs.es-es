---
title: 'Tutorial: Uso de Azure Key Vault con una máquina virtual en .NET | Microsoft Docs'
description: En este tutorial, va a configurar una aplicación ASP.NET Core para que lea un secreto de su almacén de claves.
services: key-vault
author: msmbaldwin
ms.service: key-vault
ms.subservice: general
ms.topic: tutorial
ms.date: 07/20/2020
ms.author: mbaldwin
ms.custom: mvc
ms.openlocfilehash: ead0c13a1fce0b5d56c9dd875c594a2269e2c78c
ms.sourcegitcommit: 29400316f0c221a43aff3962d591629f0757e780
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/02/2020
ms.locfileid: "87513118"
---
# <a name="tutorial-use-azure-key-vault-with-a-virtual-machine-in-net"></a>Tutorial: Uso de Azure Key Vault con una máquina virtual en .NET

Azure Key Vault ayuda a proteger secretos como las claves de API, las cadenas de conexión de base de datos necesarias para acceder a las aplicaciones, los servicios y los recursos de TI.

En este tutorial, aprenderá cómo obtener una aplicación de consola para leer información de Azure Key Vault. La aplicación usará la identidad administrada de máquina virtual para autenticarse en Key Vault. 

En este tutorial se muestra cómo realizar las siguientes acciones:

> [!div class="checklist"]
> * Cree un grupo de recursos.
> * Cree un almacén de claves.
> * Agregue un secreto al almacén de claves.
> * Recuperar un secreto del almacén de claves.
> * Cree una máquina virtual de Azure.
> * Habilitar una [identidad administrada](../../active-directory/managed-identities-azure-resources/overview.md) para la máquina virtual.
> * Asigne permisos a la identidad de máquina virtual.

Antes de empezar, lea los [conceptos básicos de Key Vault](basic-concepts.md). 

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## <a name="prerequisites"></a>Requisitos previos

Para Windows, Mac y Linux:
  * [Git](https://git-scm.com/downloads)
  * [SDK de .NET Core 3.1 o posterior](https://dotnet.microsoft.com/download/dotnet-core/3.1).
  * [Azure CLI](/cli/azure/install-azure-cli?view=azure-cli-latest).

## <a name="create-resources-and-assign-permissions"></a>Creación de recursos y asignación de permisos

Antes de empezar a codificar, necesita crear algunos recursos, colocar un secreto en el almacén de claves y asignar los permisos.

### <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Para iniciar sesión en Azure mediante la CLI de Azure, escriba:

```azurecli
az login
```

## <a name="create-a-resource-group-and-key-vault"></a>Creación de un grupo de recursos y de un almacén de claves

[!INCLUDE [Create a resource group and key vault](../../../includes/key-vault-rg-kv-creation.md)]

## <a name="populate-your-key-vault-with-a-secret"></a>Rellenado del almacén de claves con un secreto

[!INCLUDE [Create a secret](../../../includes/key-vault-create-secret.md)]

## <a name="create-a-virtual-machine"></a>Creación de una máquina virtual
Cree una máquina virtual Windows o Linux con uno de los métodos siguientes:

| Windows | Linux |
|--|--|
| [CLI de Azure](../../virtual-machines/windows/quick-create-cli.md) | [CLI de Azure](../../virtual-machines/linux/quick-create-cli.md) |  
| [PowerShell](../../virtual-machines/windows/quick-create-powershell.md) | [PowerShell](../../virtual-machines/linux/quick-create-powershell.md) |
| [Azure Portal](../../virtual-machines/windows/quick-create-portal.md) | [Azure Portal](../../virtual-machines/linux/quick-create-portal.md) |

## <a name="assign-an-identity-to-the-vm"></a>Asignación de una identidad a la máquina virtual
Cree una identidad asignada por el sistema para la máquina virtual con el comando [az vm identity assign](/cli/azure/vm/identity?view=azure-cli-latest#az-vm-identity-assign):

```azurecli
az vm identity assign --name <NameOfYourVirtualMachine> --resource-group <YourResourceGroupName>
```

Tenga en cuenta la identidad asignada por el sistema que se muestra en el código siguiente. La salida del comando anterior sería: 

```output
{
  "systemAssignedIdentity": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "userAssignedIdentities": {}
}
```

## <a name="assign-permissions-to-the-vm-identity"></a>Asignación de permisos a la identidad de máquina virtual
Asigne los permisos de la identidad creada anteriormente al almacén de claves con el comando[az keyvault set-policy](/cli/azure/keyvault?view=azure-cli-latest#az-keyvault-set-policy):

```azurecli
az keyvault set-policy --name '<your-unique-key-vault-name>' --object-id <VMSystemAssignedIdentity> --secret-permissions get list
```

## <a name="sign-in-to-the-virtual-machine"></a>Inicio de sesión en la máquina virtual

Para iniciar sesión en la máquina virtual, siga las instrucciones que encontrará en [Conexión a una máquina virtual de Azure donde se ejecuta Windows e inicio de sesión en ella](../../virtual-machines/windows/connect-logon.md) o [Conexión a una máquina virtual Linux en Azure e inicio de sesión en ella](../../virtual-machines/linux/login-using-aad.md).

## <a name="set-up-the-console-app"></a>Configuración de la aplicación de consola

Cree una aplicación de consola e instale los paquetes necesarios con el comando `dotnet`.

### <a name="install-net-core"></a>Instalar .NET Core

Para instalar .NET Core, vaya a la [página de descargas de .NET](https://www.microsoft.com/net/download).

### <a name="create-and-run-a-sample-net-app"></a>Creación y ejecución de una aplicación de .NET de ejemplo

Abra un símbolo del sistema.

Puede imprimir "Hola mundo" en la consola mediante la ejecución de los siguientes comandos:

```console
dotnet new console -n keyvault-console-app
cd keyvault-console-app
dotnet run
```

### <a name="install-the-package"></a>Instalar el paquete

En la ventana de la consola, instale la biblioteca cliente de secretos de Azure Key Vault para .NET:

```console
dotnet add package Azure.Security.KeyVault.Secrets
```

Para este inicio rápido, necesitará instalar el siguiente paquete de identidad para autenticarse en Azure Key Vault:

```console
dotnet add package Azure.Identity
```

## <a name="edit-the-console-app"></a>Edición de la aplicación de consola

Abra el archivo *Program.cs* y agregue estos paquetes:

```csharp
using System;
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;
```

Agregue estas líneas. De este modo, el URI se actualizará para mostrar el `vaultUri` de su almacén de claves. El código siguiente usa ['DefaultAzureCredential ()'](/dotnet/api/azure.identity.defaultazurecredential?view=azure-dotnet) para la autenticación en el almacén de claves, que usa el token de la identidad administrada de la aplicación con fines de autenticación. También utiliza el retroceso exponencial para los reintentos en el caso de que se limite el almacén de claves.

```csharp
  class Program
    {
        static void Main(string[] args)
        {
            string secretName = "mySecret";

            var kvUri = "https://<your-key-vault-name>.vault.azure.net";
            SecretClientOptions options = new SecretClientOptions()
            {
                Retry =
                {
                    Delay= TimeSpan.FromSeconds(2),
                    MaxDelay = TimeSpan.FromSeconds(16),
                    MaxRetries = 5,
                    Mode = RetryMode.Exponential
                 }
            };

            var client = new SecretClient(new Uri(kvUri), new DefaultAzureCredential(),options);

            Console.Write("Input the value of your secret > ");
            string secretValue = Console.ReadLine();

            Console.Write("Creating a secret in " + keyVaultName + " called '" + secretName + "' with the value '" + secretValue + "` ...");

            client.SetSecret(secretName, secretValue);

            Console.WriteLine(" done.");

            Console.WriteLine("Forgetting your secret.");
            secretValue = "";
            Console.WriteLine("Your secret is '" + secretValue + "'.");

            Console.WriteLine("Retrieving your secret from " + keyVaultName + ".");

            KeyVaultSecret secret = client.GetSecret(secretName);

            Console.WriteLine("Your secret is '" + secret.Value + "'.");

            Console.Write("Deleting your secret from " + keyVaultName + " ...");

            client.StartDeleteSecret(secretName);

            System.Threading.Thread.Sleep(5000);
            Console.WriteLine(" done.");

        }
    }
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no son necesarios, elimine la máquina virtual y el almacén de claves.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [API REST de Azure Key Vault](https://docs.microsoft.com/rest/api/keyvault/)
