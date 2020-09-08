---
title: Implementación de una máquina virtual con C# y una plantilla de Resource Manager
description: Aprenda a usar C# y una plantilla de Resource Manager para implementar una máquina virtual de Azure.
author: cynthn
ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows
ms.topic: how-to
ms.date: 07/14/2017
ms.author: cynthn
ms.custom: devx-track-csharp
ms.openlocfilehash: 779a09532790ea272d8c95ac28f8c152216efc5a
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89002970"
---
# <a name="deploy-an-azure-virtual-machine-using-c-and-a-resource-manager-template"></a>Implementación de una máquina virtual de Azure con C# y una plantilla de Resource Manager

En este artículo se muestra cómo implementar una plantilla de Azure Resource Manager con C#. La plantilla que crea implementa una sola máquina virtual que ejecuta Windows Server en una nueva red virtual con una sola subred.

Para obtener una descripción detallada del recurso de máquina virtual, consulte [Virtual machines in an Azure Resource Manager template](template-description.md) (Máquinas virtuales en una plantilla de Azure Resource Manager). Para obtener más información acerca de todos los recursos de una plantilla, consulte [Tutorial de la plantilla de Resource Manager](../../azure-resource-manager/templates/quickstart-create-templates-use-the-portal.md).

Tardará unos 10 minutos en realizar estos pasos.

## <a name="create-a-visual-studio-project"></a>Creación de un proyecto de Visual Studio

En este paso, asegúrese de que está instalado Visual Studio y cree una aplicación de consola que se utilice para implementar la plantilla.

1. Si aún no lo ha hecho, instale [Visual Studio](/visualstudio/install/install-visual-studio). Seleccione **Desarrollo de escritorio de .NET** en la página Cargas de trabajo y haga clic en **Instalar**. En el resumen, puede ver que **Herramientas de desarrollo de .NET Framework 4 – 4.6** se selecciona automáticamente. Si ya ha instalado Visual Studio, puede agregar la carga de trabajo de .NET con el selector de Visual Studio.
2. En Visual Studio, haga clic en **Archivo** > **Nuevo** > **proyecto**.
3. En **Plantillas** > **Visual C#** , seleccione **Aplicación de consola (.NET Framework)** , escriba *myDotnetProject* como nombre del proyecto, seleccione la ubicación del proyecto y, haga clic en **Aceptar**.

## <a name="install-the-packages"></a>Instalación de los paquetes

Los paquetes de NuGet son la manera más fácil de instalar las bibliotecas que necesita para finalizar estos pasos. Para obtener las bibliotecas que necesita en Visual Studio, siga estos pasos:

1. Haga clic en **Herramientas** > **Administrador de paquetes Nuget** y, después, haga clic en **Consola del Administrador de paquetes**.
2. En la consola, escriba estos comandos:

    ```powershell
    Install-Package Microsoft.Azure.Management.Fluent
    Install-Package WindowsAzure.Storage
    ```

## <a name="create-the-files"></a>Creación de los archivos

En este paso, creará un archivo de plantilla que implementa los recursos y un archivo de parámetros que proporciona valores de parámetro a la plantilla. También creará un archivo de autorización que se utiliza para realizar operaciones de Azure Resource Manager.

### <a name="create-the-template-file"></a>Cree el archivo de plantilla

1. En el Explorador de soluciones, haga clic en *myDotnetProject* > **Agregar** > **Nuevo elemento** y, después, seleccione **Archivo de texto** en *Elementos de Visual C#* . Asigne un nombre al archivo *CreateVMTemplate.json* y, después, haga clic en **Agregar**.
2. Agregue este código JSON al archivo creado:

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "adminUsername": { "type": "string" },
        "adminPassword": { "type": "securestring" }
      },
      "variables": {
        "vnetID": "[resourceId('Microsoft.Network/virtualNetworks','myVNet')]", 
        "subnetRef": "[concat(variables('vnetID'),'/subnets/mySubnet')]", 
      },
      "resources": [
        {
          "apiVersion": "2016-03-30",
          "type": "Microsoft.Network/publicIPAddresses",
          "name": "myPublicIPAddress",
          "location": "[resourceGroup().location]",
          "properties": {
            "publicIPAllocationMethod": "Dynamic",
            "dnsSettings": {
              "domainNameLabel": "myresourcegroupdns1"
            }
          }
        },
        {
          "apiVersion": "2016-03-30",
          "type": "Microsoft.Network/virtualNetworks",
          "name": "myVNet",
          "location": "[resourceGroup().location]",
          "properties": {
            "addressSpace": { "addressPrefixes": [ "10.0.0.0/16" ] },
            "subnets": [
              {
                "name": "mySubnet",
                "properties": { "addressPrefix": "10.0.0.0/24" }
              }
            ]
          }
        },
        {
          "apiVersion": "2016-03-30",
          "type": "Microsoft.Network/networkInterfaces",
          "name": "myNic",
          "location": "[resourceGroup().location]",
          "dependsOn": [
            "[resourceId('Microsoft.Network/publicIPAddresses/', 'myPublicIPAddress')]",
            "[resourceId('Microsoft.Network/virtualNetworks/', 'myVNet')]"
          ],
          "properties": {
            "ipConfigurations": [
              {
                "name": "ipconfig1",
                "properties": {
                  "privateIPAllocationMethod": "Dynamic",
                  "publicIPAddress": { "id": "[resourceId('Microsoft.Network/publicIPAddresses','myPublicIPAddress')]" },
                  "subnet": { "id": "[variables('subnetRef')]" }
                }
              }
            ]
          }
        },
        {
          "apiVersion": "2016-04-30-preview",
          "type": "Microsoft.Compute/virtualMachines",
          "name": "myVM",
          "location": "[resourceGroup().location]",
          "dependsOn": [
            "[resourceId('Microsoft.Network/networkInterfaces/', 'myNic')]"
          ],
          "properties": {
            "hardwareProfile": { "vmSize": "Standard_DS1" },
            "osProfile": {
              "computerName": "myVM",
              "adminUsername": "[parameters('adminUsername')]",
              "adminPassword": "[parameters('adminPassword')]"
            },
            "storageProfile": {
              "imageReference": {
                "publisher": "MicrosoftWindowsServer",
                "offer": "WindowsServer",
                "sku": "2012-R2-Datacenter",
                "version": "latest"
              },
              "osDisk": {
                "name": "myManagedOSDisk",
                "caching": "ReadWrite",
                "createOption": "FromImage"
              }
            },
            "networkProfile": {
              "networkInterfaces": [
                {
                  "id": "[resourceId('Microsoft.Network/networkInterfaces','myNic')]"
                }
              ]
            }
          }
        }
      ]
    }
    ```

3. Guarde el archivo CreateVMTemplate.json.

### <a name="create-the-parameters-file"></a>Cree el archivo de parámetros

Para especificar los valores de los parámetros de recurso en la plantilla, debe crear un archivo de parámetros que contenga los valores.

1. En el Explorador de soluciones, haga clic en *myDotnetProject* > **Agregar** > **Nuevo elemento** y, después, seleccione **Archivo de texto** en *Elementos de Visual C#* . Asigne un nombre al archivo *Parameters.json* y, después, haga clic en **Agregar**.
2. Agregue este código JSON al archivo creado:

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "adminUserName": { "value": "azureuser" },
        "adminPassword": { "value": "Azure12345678" }
      }
    }
    ```

4. Guarde el archivo Parameters.json.

### <a name="create-the-authorization-file"></a>Creación del archivo de autorización

Antes de poder implementar una plantilla, asegúrese de que tiene acceso a una [entidad de servicio de Active Directory](../../active-directory/develop/howto-authenticate-service-principal-powershell.md). Desde la entidad de servicio, adquiera un token para autenticar solicitudes a Azure Resource Manager. También debe registrar el identificador de aplicación, la clave de autenticación y el identificador del inquilino que necesitará en el archivo de autorización.

1. En el Explorador de soluciones, haga clic en *myDotnetProject* > **Agregar** > **Nuevo elemento** y, después, seleccione **Archivo de texto** en *Elementos de Visual C#* . Asigne un nombre al archivo *azureauth.properties* y, luego, haga clic en **Agregar**.
2. Agregue estas propiedades de autorización:

    ```
    subscription=<subscription-id>
    client=<application-id>
    key=<authentication-key>
    tenant=<tenant-id>
    managementURI=https://management.core.windows.net/
    baseURL=https://management.azure.com/
    authURL=https://login.windows.net/
    graphURL=https://graph.microsoft.com/
    ```

    Reemplace **&lt;subscription-id&gt;** por su identificador de suscripción, **&lt;application-id&gt;** por el identificador de aplicación de Active Directory, **&lt;authentication-key&gt;** por la clave de aplicación y **&lt;tenant-id&gt;** por el identificador de inquilino.

3. Guarde el archivo azureauth.properties.
4. Establezca una variable de entorno de Windows llamada AZURE_AUTH_LOCATION con la ruta de acceso completa al archivo de autorización que ha creado; por ejemplo, puede usar el siguiente comando de PowerShell:

    ```powershell
    [Environment]::SetEnvironmentVariable("AZURE_AUTH_LOCATION", "C:\Visual Studio 2019\Projects\myDotnetProject\myDotnetProject\azureauth.properties", "User")
    ```

    

## <a name="create-the-management-client"></a>Creación del cliente de administración

1. Abra el archivo Program.cs del proyecto que ha creado. A continuación, agregue estas instrucciones using a las instrucciones existentes en la parte superior del archivo:

    ```csharp
    using Microsoft.Azure.Management.Compute.Fluent;
    using Microsoft.Azure.Management.Compute.Fluent.Models;
    using Microsoft.Azure.Management.Fluent;
    using Microsoft.Azure.Management.ResourceManager.Fluent;
    using Microsoft.Azure.Management.ResourceManager.Fluent.Core;
    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.Storage.Blob;
    ```

2. Para crear el cliente de administración, agregue este código al método Main:

    ```csharp
    var credentials = SdkContext.AzureCredentialsFactory
        .FromFile(Environment.GetEnvironmentVariable("AZURE_AUTH_LOCATION"));

    var azure = Azure
        .Configure()
        .WithLogLevel(HttpLoggingDelegatingHandler.Level.Basic)
        .Authenticate(credentials)
        .WithDefaultSubscription();
    ```

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Con el fin de especificar valores para la aplicación, agregue el código al método Main:

```csharp
var groupName = "myResourceGroup";
var location = Region.USWest;

var resourceGroup = azure.ResourceGroups.Define(groupName)
    .WithRegion(location)
    .Create();
```

## <a name="create-a-storage-account"></a>Crear una cuenta de almacenamiento

La plantilla y los parámetros se implementan desde una cuenta de almacenamiento de Azure. En este paso, va a crear la cuenta y cargar los archivos. 

Para crear la cuenta, agregue este código al método Main:

```csharp
string storageAccountName = SdkContext.RandomResourceName("st", 10);

Console.WriteLine("Creating storage account...");
var storage = azure.StorageAccounts.Define(storageAccountName)
    .WithRegion(Region.USWest)
    .WithExistingResourceGroup(resourceGroup)
    .Create();

var storageKeys = storage.GetKeys();
string storageConnectionString = "DefaultEndpointsProtocol=https;"
    + "AccountName=" + storage.Name
    + ";AccountKey=" + storageKeys[0].Value
    + ";EndpointSuffix=core.windows.net";

var account = CloudStorageAccount.Parse(storageConnectionString);
var serviceClient = account.CreateCloudBlobClient();

Console.WriteLine("Creating container...");
var container = serviceClient.GetContainerReference("templates");
container.CreateIfNotExistsAsync().Wait();
var containerPermissions = new BlobContainerPermissions()
    { PublicAccess = BlobContainerPublicAccessType.Container };
container.SetPermissionsAsync(containerPermissions).Wait();

Console.WriteLine("Uploading template file...");
var templateblob = container.GetBlockBlobReference("CreateVMTemplate.json");
templateblob.UploadFromFileAsync("..\\..\\CreateVMTemplate.json").Result();

Console.WriteLine("Uploading parameters file...");
var paramblob = container.GetBlockBlobReference("Parameters.json");
paramblob.UploadFromFileAsync("..\\..\\Parameters.json").Result();
```

## <a name="deploy-the-template"></a>Implementación de la plantilla

Implemente la plantilla y los parámetros de la cuenta de almacenamiento que creó. 

Para implementar la plantilla, agregue este código al método Main:

```csharp
var templatePath = "https://" + storageAccountName + ".blob.core.windows.net/templates/CreateVMTemplate.json";
var paramPath = "https://" + storageAccountName + ".blob.core.windows.net/templates/Parameters.json";
var deployment = azure.Deployments.Define("myDeployment")
    .WithExistingResourceGroup(groupName)
    .WithTemplateLink(templatePath, "1.0.0.0")
    .WithParametersLink(paramPath, "1.0.0.0")
    .WithMode(Microsoft.Azure.Management.ResourceManager.Fluent.Models.DeploymentMode.Incremental)
    .Create();
Console.WriteLine("Press enter to delete the resource group...");
Console.ReadLine();
```

## <a name="delete-the-resources"></a>Eliminación de los recursos

Dado que se le cobrará por los recursos utilizados en Azure, siempre es conveniente eliminar los recursos que ya no sean necesarios. No es necesario eliminar por separado cada recurso de un grupo de recursos. Elimine el grupo de recursos y todos sus recursos se eliminarán automáticamente. 

Para eliminar el grupo de recursos, agregue este código al método Main:

```csharp
azure.ResourceGroups.DeleteByName(groupName);
```

## <a name="run-the-application"></a>Ejecución de la aplicación

Esta aplicación de consola tardará unos cinco minutos en ejecutarse completamente de principio a fin. 

1. Para ejecutar la aplicación de consola, haga clic en **Iniciar**.

2. Antes de presionar **Entrar** para comenzar la eliminación de recursos, puede dedicar unos minutos a comprobar la creación de los recursos en Azure Portal. Haga clic en el estado de implementación para ver información de la implementación.

## <a name="next-steps"></a>Pasos siguientes

* Si se produjeron problemas durante la implementación, el paso siguiente sería consultar [Solución de errores comunes de implementación de Azure con Azure Resource Manager](../../azure-resource-manager/templates/common-deployment-errors.md).
* Obtenga información sobre cómo implementar una máquina virtual y los recursos de apoyo leyendo [Implementación de una máquina virtual de Azure con C#](../../azure-resource-manager/templates/deploy-rest.md).