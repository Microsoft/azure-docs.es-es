---
title: 'Tutorial: Uso de Azure Key Vault con una aplicación web de Azure en .NET | Microsoft Docs'
description: En este tutorial, va a configurar una aplicación ASP.NET Core para que lea un secreto de su almacén de claves.
services: key-vault
author: msmbaldwin
manager: rajvijan
ms.service: key-vault
ms.subservice: general
ms.topic: tutorial
ms.date: 05/06/2020
ms.author: mbaldwin
ms.custom: devx-track-csharp
ms.openlocfilehash: be18718513e8624db00d502228a3e5af6076d9d7
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89007084"
---
# <a name="tutorial-use-a-managed-identity-to-connect-key-vault-to-an-azure-web-app-with-net"></a>Tutorial: Uso de una identidad administrada para conectar Key Vault a una aplicación web de Azure con .NET

Azure Key Vault proporciona una manera de almacenar de forma segura credenciales y otros secretos, pero el código tiene que autenticarse en Key Vault para recuperarlos. La información del artículo [Acerca de Managed Identities for Azure Resources](../../active-directory/managed-identities-azure-resources/overview.md) ayuda a resolver este problema al proporcionar a los servicios de Azure una identidad administrada automáticamente en Azure AD. Puede usar esta identidad para autenticar cualquier servicio que admita la autenticación de Azure AD, incluido Key Vault, sin necesidad de tener que mostrar las credenciales en el código.

En este tutorial, se usa una identidad administrada para autenticar una aplicación web de Azure mediante Azure Key Vault. Aunque en los pasos se usan las [bibliotecas de Azure Key Vault para .NET v4](/dotnet/api/overview/azure/key-vault?view=azure-dotnet) y la [CLI de Azure](/cli/azure/get-started-with-azure-cli), se aplican los mismos principios básicos al usar el lenguaje de desarrollo que elija, Azure PowerShell o Azure Portal.

## <a name="prerequisites"></a>Requisitos previos

Para completar esta guía de inicio rápido:

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* [SDK de .NET Core 3.1 o posterior](https://dotnet.microsoft.com/download/dotnet-core/3.1).
* [CLI de Azure](/cli/azure/install-azure-cli?view=azure-cli-latest) o [Azure PowerShell](/powershell/azure/)

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Un grupo de recursos es un contenedor lógico en el que se implementan y se administran los recursos de Azure. Cree un grupo de recursos para albergar su almacén de claves y aplicación web mediante el comando [az group create](/cli/azure/group?view=azure-cli-latest#az-group-create):

```azurecli-interactive
az group create --name "myResourceGroup" -l "EastUS"
```

## <a name="set-up-your-key-vault"></a>Configuración de su almacén de claves

Ahora creará su propio almacén de claves y agregará un secreto en él para usarlo más adelante en este tutorial.

Para crear su propio almacén de claves, utilice el comando [az keyvault create](/cli/azure/keyvault?view=azure-cli-latest#az-keyvault-create).

> [!Important]
> Cada almacén de claves debe tener un nombre único. Reemplace <your-keyvault-name> con el nombre de su almacén de claves en los ejemplos siguientes.

```azurecli-interactive
az keyvault create --name "<your-keyvault-name>" -g "myResourceGroup"
```

Tome nota del valor `vaultUri` devuelto, que tendrá el formato "https://&lt;your-keyvault-name&gt;.vault.azure.net/". Se usará en el paso para [actualizar el código](#update-the-code).

[!INCLUDE [Create a secret](../../../includes/key-vault-create-secret.md)]

## <a name="create-a-net-web-app"></a>Creación de una aplicación web de .NET

### <a name="create-a-local-app"></a>Creación de una aplicación local

En la ventana del terminal de la máquina, cree un directorio denominado `akvwebapp` y haga que sea el directorio actual.

```bash
mkdir akvwebapp
cd akvwebapp
```

Ahora cree una nueva aplicación .NET Core mediante el comando [dotnet new web](/dotnet/core/tools/dotnet-new):

```bash
dotnet new web
```

Ejecute la aplicación localmente para ver cómo debería ser si se implementara en Azure. 

```bash
dotnet run
```

Abra un explorador web y vaya a la aplicación en `http://localhost:5000`.

Se mostrará el mensaje **Hola mundo** de la aplicación de ejemplo en la página.

### <a name="initialize-the-git-repository"></a>Inicialización del repositorio de Git

En la ventana de terminal, presione **Ctrl + C** para salir del servidor web.  Inicialice un repositorio Git para el proyecto de .NET Core.

```bash
git init
git add .
git commit -m "first commit"
```

### <a name="configure-a-deployment-user"></a>Configuración de un usuario de implementación

Se puede implementar FTP y Git local en una aplicación web de Azure mediante un *usuario de implementación*. Una vez configurado este usuario de implementación, podrá usarlo en todas las implementaciones de Azure. El nombre de usuario y la contraseña en el nivel de cuenta son diferentes de las credenciales de suscripción de Azure. 

Para configurar el usuario de implementación, ejecute el comando [az webapp deployment user set](/cli/azure/webapp/deployment/user?view=azure-cli-latest#az-webapp-deployment-user-set). Elija un nombre de usuario y una contraseña que sigan estas siguientes directrices: 

- El nombre de usuario debe ser único dentro de Azure y no debe contener el símbolo "\@" para las inserciones de Git local. 
- La contraseña debe tener al menos ocho caracteres y dos de los tres elementos siguientes: letras, números y símbolos. 

```azurecli-interactive
az webapp deployment user set --user-name "<username>" --password "<password>"
```

La salida JSON muestra la contraseña como `null`. Si se produce un error `'Conflict'. Details: 409`, cambie el nombre de usuario. Si se produce un error `'Bad Request'. Details: 400`, use una contraseña más segura. 

Anote el nombre de usuario y la contraseña que se usarán para implementar las aplicaciones web.

### <a name="create-an-app-service-plan"></a>Creación de un plan de App Service

Cree un plan de App Service mediante el comando [az appservice plan create](/cli/azure/appservice/plan?view=azure-cli-latest) de la CLI de Azure. En el siguiente ejemplo, se crea un plan de App Service denominado `myAppServicePlan` con el plan de tarifa **Gratis**:

```azurecli-interactive
az appservice plan create --name myAppServicePlan --resource-group myResourceGroup --sku FREE
```

Cuando se crea el plan de App Service, la CLI de Azure muestra información similar al ejemplo siguiente:

<pre>
{ 
  "adminSiteName": null,
  "appServicePlanName": "myAppServicePlan",
  "geoRegion": "West Europe",
  "hostingEnvironmentProfile": null,
  "id": "/subscriptions/0000-0000/resourceGroups/myResourceGroup/providers/Microsoft.Web/serverfarms/myAppServicePlan",
  "kind": "app",
  "location": "West Europe",
  "maximumNumberOfWorkers": 1,
  "name": "myAppServicePlan",
  &lt; JSON data removed for brevity. &gt;
  "targetWorkerSizeId": 0,
  "type": "Microsoft.Web/serverfarms",
  "workerTierName": null
} 
</pre>


### <a name="create-a-remote-web-app"></a>Creación de una aplicación web remota

Creación de una [aplicación web de Azure](../../app-service/overview.md#app-service-on-linux) en el plan de App Service `myAppServicePlan`. 

> [!Important]
> Al igual que sucede con Key Vault, una aplicación web de Azure debe tener un nombre único. Reemplace \<your-webapp-name\> por el nombre de su aplicación web en los ejemplos a continuación.


```azurecli-interactive
az webapp create --resource-group "myResourceGroup" --plan "myAppServicePlan" --name "<your-webapp-name>" --deployment-local-git
```

Cuando se haya creado la aplicación web, la CLI de Azure mostrará información similar a la del ejemplo siguiente:

<pre>
Local git is configured with url of 'https://&lt;username&gt;@&lt;your-webapp-name&gt;.scm.azurewebsites.net/&lt;ayour-webapp-name&gt;.git'
{
  "availabilityState": "Normal",
  "clientAffinityEnabled": true,
  "clientCertEnabled": false,
  "clientCertExclusionPaths": null,
  "cloningInfo": null,
  "containerSize": 0,
  "dailyMemoryTimeQuota": 0,
  "defaultHostName": "&lt;your-webapp-name&gt;.azurewebsites.net",
  "deploymentLocalGitUrl": "https://&lt;username&gt;@&lt;your-webapp-name&gt;.scm.azurewebsites.net/&lt;your-webapp-name&gt;.git",
  "enabled": true,
  &lt; JSON data removed for brevity. &gt;
}
</pre>


La dirección URL del repositorio de Git remoto se muestra en la propiedad `deploymentLocalGitUrl`, con el formato `https://<username>@<your-webapp-name>.scm.azurewebsites.net/<your-webapp-name>.git`. Guarde esta dirección URL, ya que la necesitará más adelante.

Vaya a la aplicación recién creada. Reemplace _&lt;your-webapp-name>_ con el nombre de la aplicación.

```bash
https://<your-webapp-name>.azurewebsites.net
```

Verá la página web predeterminada para la aplicación web de Azure recién creada.

### <a name="deploy-your-local-app"></a>Implementación de la aplicación local

De nuevo en la ventana del terminal local, agregue una versión remota de Azure al repositorio de Git local. Para ello, reemplace *\<deploymentLocalGitUrl-from-create-step>* por la dirección URL de Git remoto que guardó en el paso [Creación de una aplicación web remota](#create-a-remote-web-app).

```bash
git remote add azure <deploymentLocalGitUrl-from-create-step>
```

Realice la insercion en la instancia remota de Azure para implementar la aplicación con el comando siguiente. Cuando el administrador de credenciales de Git se le solicite las credenciales, utilice las credenciales que creó en el paso [Configuración de un usuario de implementación](#configure-a-deployment-user).

```bash
git push azure master
```

Este comando puede tardar varios minutos en ejecutarse. Durante la ejecución, muestra información similar a la del ejemplo siguiente:
<pre>
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 285 bytes | 95.00 KiB/s, done.
Total 3 (delta 2), reused 0 (delta 0), pack-reused 0
remote: Deploy Async
remote: Updating branch 'master'.
remote: Updating submodules.
remote: Preparing deployment for commit id 'd6b54472f7'.
remote: Repository path is /home/site/repository
remote: Running oryx build...
remote: Build orchestrated by Microsoft Oryx, https://github.com/Microsoft/Oryx
remote: You can report issues at https://github.com/Microsoft/Oryx/issues
remote:
remote: Oryx Version      : 0.2.20200114.13, Commit: 204922f30f8e8d41f5241b8c218425ef89106d1d, ReleaseTagName: 20200114.13
remote: Build Operation ID: |imoMY2y77/s=.40ca2a87_
remote: Repository Commit : d6b54472f7e8e9fd885ffafaa64522e74cf370e1
.
.
.
remote: Deployment successful.
remote: Deployment Logs : 'https://&lt;your-webapp-name&gt;.scm.azurewebsites.net/newui/jsonviewer?view_url=/api/deployments/d6b54472f7e8e9fd885ffafaa64522e74cf370e1/log'
To https://&lt;your-webapp-name&gt;.scm.azurewebsites.net:443/&lt;your-webapp-name&gt;.git
   d87e6ca..d6b5447  master -> master
</pre>

Vaya a la aplicación implementada, o actualícela, en el explorador web.

```bash
http://<your-webapp-name>.azurewebsites.net
```

Verá el mensaje "Hola mundo" que ya se mostró al visitar `http://localhost:5000`.

## <a name="create-and-assign-a-managed-identity"></a>Creación y asignación de una identidad administrada

Para crear la identidad de esta aplicación, ejecute el comando [az webapp-identity assign](/cli/azure/webapp/identity?view=azure-cli-latest#az-webapp-identity-assign) en la CLI de Azure:

```azurecli-interactive
az webapp identity assign --name "<your-webapp-name>" --resource-group "myResourceGroup"
```

La operación devolverá este fragmento de código JSON:

```json
{
  "principalId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "tenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "type": "SystemAssigned"
}
```

Para conceder permiso a la aplicación web y que esta pueda realizar operaciones **get** y **list** en el almacén de claves, pase el valor principalID al comando [AZ keyvault set-policy](/cli/azure/keyvault?view=azure-cli-latest#az-keyvault-set-policy) de la CLI de Azure:

```azurecli-interactive
az keyvault set-policy --name "<your-keyvault-name>" --object-id "<principalId>" --secret-permissions get list
```


## <a name="modify-the-app-to-access-your-key-vault"></a>Modificación de la aplicación para acceder al almacén de claves

### <a name="install-the-packages"></a>Instalación de los paquetes

En la ventana del terminal, instale la biblioteca cliente de Azure Key Vault para los paquetes de .NET:

```console
dotnet add package Azure.Identity
dotnet add package Azure.Security.KeyVault.Secrets
```

### <a name="update-the-code"></a>Actualización del código

Busque y abra el archivo Startup.cs en su proyecto akvwebapp. 

Agregue estas dos líneas al encabezado:

```csharp
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;
using Azure.Core;
```

Agregue estas líneas antes de la llamada `app.UseEndpoints`. De este modo, el URI se actualizará para mostrar el `vaultUri` de su almacén de claves. El código siguiente usa ['DefaultAzureCredential ()'](/dotnet/api/azure.identity.defaultazurecredential?view=azure-dotnet) para la autenticación en el almacén de claves, que usa el token de la identidad administrada de la aplicación con fines de autenticación. También utiliza el retroceso exponencial para los reintentos en el caso de que se limite el almacén de claves.

```csharp
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
var client = new SecretClient(new Uri("https://<your-unique-key-vault-name>.vault.azure.net/"), new DefaultAzureCredential(),options);

KeyVaultSecret secret = client.GetSecret("mySecret");

string secretValue = secret.Value;
```

Actualice la línea `await context.Response.WriteAsync("Hello World!");` para que indique:

```csharp
await context.Response.WriteAsync(secretValue);
```

Asegúrese de guardar los cambios antes de continuar con el siguiente paso.

### <a name="redeploy-your-web-app"></a>Nueva implementación de la aplicación web

Una vez que haya actualizado el código, puede volver a implementarlo en Azure con los siguientes comandos de Git:

```bash
git add .
git commit -m "Updated web app to access my key vault"
git push azure master
```

## <a name="visit-your-completed-web-app"></a>Visita de la aplicación web completada

```bash
http://<your-webapp-name>.azurewebsites.net
```

En los casos en los que antes vio **Hola mundo**, ahora debería ver el valor del secreto: **Correcto**

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre las [identidades administradas para recursos de Azure](../../active-directory/managed-identities-azure-resources/overview.md)
- Más información sobre las [identidades administradas para App Service](../../app-service/overview-managed-identity.md?tabs=dotnet)
- Consulte las [bibliotecas de Azure Key Vault para .NET](/dotnet/api/overview/azure/key-vault?view=azure-dotnet)
- Consulte el [código fuente de las bibliotecas de Azure Key Vault para .NET](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/keyvault)
- Consulte los [paquetes NuGet de las bibliotecas de Azure Key Vault para .NET v4](https://www.nuget.org/packages/Azure.Security.KeyVault.Secrets/)


