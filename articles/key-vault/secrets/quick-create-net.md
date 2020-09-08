---
title: 'Inicio rápido: biblioteca cliente de Azure Key Vault para .NET (v4)'
description: Obtenga información sobre cómo crear, recuperar y eliminar secretos desde una instancia de Azure Key Vault mediante la biblioteca cliente de .NET (v4)
author: msmbaldwin
ms.author: mbaldwin
ms.date: 03/12/2020
ms.service: key-vault
ms.subservice: secrets
ms.topic: quickstart
ms.custom: devx-track-csharp
ms.openlocfilehash: cf9c34dc2dde5c41edb7991451aebf5d90104196
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89003259"
---
# <a name="quickstart-azure-key-vault-client-library-for-net-sdk-v4"></a>Inicio rápido: Biblioteca cliente de Azure Key Vault para .NET (SDK v4)

Comience con la biblioteca cliente de Azure Key Vault para .NET. Siga estos pasos para instalar el paquete y probar el código de ejemplo para realizar tareas básicas.

Azure Key Vault ayuda a proteger claves criptográficas y secretos usados por servicios y aplicaciones en la nube. Use la biblioteca cliente de Key Vault para .NET para:

- Aumentar la seguridad y el control sobre claves y contraseñas
- Crear e importar claves de cifrado en minutos
- Reducir la latencia con escala en la nube y redundancia global
- Simplificar y automatizar tareas para certificados TLS/SSL
- Utilizar módulos HSM con certificación FIPS 140-2 nivel 2

[Documentación de referencia de la API](/dotnet/api/azure.security.keyvault.secrets?view=azure-dotnet) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/keyvault) | [Paquete (NuGet)](https://www.nuget.org/packages/Azure.Security.KeyVault.Secrets/)

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* [SDK de .NET Core 3.1 o posterior](https://dotnet.microsoft.com/download/dotnet-core/3.1).
* [CLI de Azure](/cli/azure/install-azure-cli?view=azure-cli-latest) o [Azure PowerShell](/powershell/azure/)

En este inicio rápido se supone que está ejecutando `dotnet`, la [CLI de Azure](/cli/azure/install-azure-cli?view=azure-cli-latest) y comandos de Windows en un terminal de Windows (como [PowerShell Core](/powershell/scripting/install/installing-powershell-core-on-windows?view=powershell-6), [Windows PowerShell](/powershell/scripting/install/installing-windows-powershell?view=powershell-6) o [Azure Cloud Shell](https://shell.azure.com/)).

## <a name="setting-up"></a>Instalación

### <a name="create-new-net-console-app"></a>Creación de una aplicación de consola de .NET

En una ventana de consola, utilice el comando `dotnet new` para crear una nueva aplicación de consola de .NET con el nombre `key-vault-console-app`.

```console
dotnet new console -n key-vault-console-app
```

Cambie el directorio a la carpeta de aplicaciones recién creada. Para compilar la aplicación:

```console
dotnet build
```

La salida de la compilación no debe contener advertencias ni errores.

```console
Build succeeded.
 0 Warning(s)
 0 Error(s)
```

### <a name="install-the-package"></a>Instalar el paquete

En la ventana de la consola, instale la biblioteca cliente de Azure Key Vault para .NET:

```console
dotnet add package Azure.Security.KeyVault.Secrets
```

Para este inicio rápido, deberá instalar también los siguientes paquetes:

```console
dotnet add package Azure.Identity
```

### <a name="create-a-resource-group-and-key-vault"></a>Creación de un grupo de recursos y de un almacén de claves

[!INCLUDE [Create a resource group and key vault](../../../includes/key-vault-rg-kv-creation.md)]

### <a name="create-a-service-principal"></a>Creación de una entidad de servicio

[!INCLUDE [Create a service principal](../../../includes/key-vault-sp-creation.md)]

#### <a name="give-the-service-principal-access-to-your-key-vault"></a>Acceso de la entidad de servicio al almacén de claves

[!INCLUDE [Give the service principal access to your key vault](../../../includes/key-vault-sp-kv-access.md)]

#### <a name="set-environmental-variables"></a>Establecimiento de variables de entorno

[!INCLUDE [Set environmental variables](../../../includes/key-vault-set-environmental-variables.md)]

## <a name="object-model"></a>Modelo de objetos

La biblioteca cliente de Azure Key Vault para .NET le permite administrar las claves y los recursos relacionados, como certificados y secretos. Los ejemplos de código siguientes le mostrarán cómo crear un cliente y cómo establecer, recuperar y eliminar un secreto.

Toda la aplicación de consola está disponible en https://github.com/Azure-Samples/key-vault-dotnet-core-quickstart/tree/master/key-vault-console-app.

## <a name="code-examples"></a>Ejemplos de código

### <a name="add-directives"></a>Adición de directivas

Agregue las siguientes directivas al principio del código:

[!code-csharp[Directives](~/samples-key-vault-dotnet-quickstart/key-vault-console-app/Program.cs?name=directives)]

### <a name="authenticate-and-create-a-client"></a>Autenticación y creación de un cliente

La autenticación en el almacén de claves y la creación de un cliente de almacén de claves depende de las variables de entorno en el paso [Establecimiento de variables de entorno](#set-environmental-variables) anterior. El nombre del almacén de claves se expande al URI del almacén de claves, con el formato "https://\<your-key-vault-name\>.vault.azure.net". El código siguiente usa ["DefaultAzureCredential()"](/dotnet/api/azure.identity.defaultazurecredential?view=azure-dotnet) para la autenticación en el almacén de claves. Lo que hace es leer las variables de entorno para recuperar el token de acceso. 

[!code-csharp[Directives](~/samples-key-vault-dotnet-quickstart/key-vault-console-app/Program.cs?name=authenticate)]

### <a name="save-a-secret"></a>Almacenamiento de un secreto

Ahora que la aplicación se ha autenticado, puede colocar un secreto en el almacén de claves mediante el [método client.SetSecret](/dotnet/api/microsoft.azure.keyvault.keyvaultclientextensions.setsecretasync). Esto requiere establecer un nombre para el secreto; en este ejemplo, usaremos "mySecret".  

[!code-csharp[Set secret](~/samples-key-vault-dotnet-quickstart/key-vault-console-app/Program.cs?name=setsecret)]

Puede comprobar que el secreto se ha establecido con el comando [az keyvault secret show](/cli/azure/keyvault/secret?view=azure-cli-latest#az-keyvault-secret-show):

```azurecli
az keyvault secret show --vault-name <your-unique-keyvault-name> --name mySecret
```

```azurepowershell
(Get-AzKeyVaultSecret -VaultName <your-unique-keyvault-name> -Name mySecret).SecretValueText
```

### <a name="retrieve-a-secret"></a>Recuperación de un secreto

Ahora puede recuperar el valor previamente establecido con el [método client.GetSecret](/dotnet/api/microsoft.azure.keyvault.keyvaultclientextensions.getsecretasync).

[!code-csharp[Get secret](~/samples-key-vault-dotnet-quickstart/key-vault-console-app/Program.cs?name=getsecret)]

El secreto se guarda ahora como `secret.Value`.

### <a name="delete-a-secret"></a>eliminar un secreto

Por último, vamos a eliminar el secreto del almacén de claves con el [método client.DeleteSecret](/dotnet/api/microsoft.azure.keyvault.keyvaultclientextensions.getsecretasync).

[!code-csharp[Delete secret](~/samples-key-vault-dotnet-quickstart/key-vault-console-app/Program.cs?name=deletesecret)]

Puede comprobar que el secreto ya no está con el comando [az keyvault secret show](/cli/azure/keyvault/secret?view=azure-cli-latest#az-keyvault-secret-show):

```azurecli
az keyvault secret show --vault-name <your-unique-keyvault-name> --name mySecret
```

```azurepowershell
(Get-AzKeyVaultSecret -VaultName <your-unique-keyvault-name> -Name mySecret).SecretValueText
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no lo necesite, puede usar la CLI de Azure o Azure PowerShell para quitar el almacén de claves y el grupo de recursos correspondiente.

### <a name="delete-a-key-vault"></a>Eliminación de un almacén de claves
```azurecli
az keyvault delete --name <your-unique-keyvault-name>
```

```powershell
Remove-AzKeyVault -VaultName <your-unique-keyvault-name>
```

### <a name="purge-a-key-vault"></a>Purga de un almacén de claves
```azurecli
az keyvault purge --location eastus --name <your-unique-keyvault-name>
```

```powershell
Remove-AzKeyVault -VaultName <your-unique-keyvault-name> -InRemovedState -Location eastus
```

### <a name="delete-a-resource-group"></a>Eliminación de un grupo de recursos

```azurecli
az group delete -g "myResourceGroup"
```

```azurepowershell
Remove-AzResourceGroup -Name "myResourceGroup"
```

## <a name="sample-code"></a>Código de ejemplo

```csharp
using System;
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;

namespace key_vault_console_app
{
    class Program
    {
        static void Main(string[] args)
        {
            string secretName = "mySecret";

            string keyVaultName = Environment.GetEnvironmentVariable("KEY_VAULT_NAME");
            var kvUri = "https://" + keyVaultName + ".vault.azure.net";

            var client = new SecretClient(new Uri(kvUri), new DefaultAzureCredential());

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
}
```


## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha creado un almacén de claves y ha almacenado un secreto en él. Consulte la [aplicación de consola completa en GitHub](https://github.com/Azure-Samples/key-vault-dotnet-core-quickstart/tree/master/key-vault-console-app).

Para más información sobre Key Vault y cómo integrarlo con las aplicaciones, continúe con los artículos siguientes.

- Lea una [introducción a Azure Key Vault](../general/overview.md).
- Consulte la [guía del desarrollador de Azure Key Vault](../general/developers-guide.md).
- Consulte los [procedimientos recomendados de Azure Key Vault](../general/best-practices.md).
