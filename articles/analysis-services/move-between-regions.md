---
title: Traslado de Azure Analysis Services a otra región | Microsoft Docs
description: Describe cómo trasladar un recurso de Azure Analysis Services a otra región.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: how-to
ms.date: 06/09/2020
ms.author: owend
ms.reviewer: minewiskan
ms.custom: references_regions
ms.openlocfilehash: 4844f3e34a6b49559affbb4d4ed7bc5b5e38e538
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87050361"
---
# <a name="move-analysis-services-to-a-different-region"></a>Traslado de Azure Analysis Services a otra región

En este artículo se describe cómo trasladar un recurso de servidor de Analysis Services a otra región de Azure. Puede trasladar el servidor a otra región por varios motivos, por ejemplo, para aprovechar las ventajas de una región de Azure más cerca de los usuarios, para usar planes de servicio admitidos solo en determinadas regiones o para cumplir los requisitos internos de regulación y gobernanza. 

En este artículo y en los artículos vinculados asociados, aprenderá a:

> [!div class="checklist"]
> 
> * Realizar una copia de seguridad de una base de datos modelo del servidor de origen en [Blob Storage](../storage/blobs/storage-blobs-introduction.md).
> * Exportar una [plantilla de recursos](../azure-resource-manager/templates/overview.md) del servidor de origen.
> * Obtener una [firma de acceso compartido (SAS)](../storage/common/storage-sas-overview.md) de almacenamiento.
> * Modificar la plantilla de recursos.
> * Implementar la plantilla para crear un nuevo servidor de destino.
> * Restaurar una base de datos modelo en el nuevo servidor de destino.
> * Comprobar el nuevo servidor de destino y la base de datos.
> * Eliminar el servidor de origen.

En este artículo se describe el uso de una plantilla de recursos para migrar un solo servidor de Analysis Services con una **configuración básica** a otra región *y* grupo de recursos de la misma suscripción. El uso de una plantilla conserva las propiedades del servidor configuradas, lo que garantiza que el servidor de destino se configura con las mismas propiedades, excepto la región y el grupo de recursos, que el servidor de origen. En este artículo no se describe el traslado de los recursos asociados que pueden formar parte del mismo grupo de recursos, como el origen de datos, el almacenamiento y los recursos de puerta de enlace. 

Antes de mover un servidor a otra región, se recomienda crear un plan detallado. Tenga en cuenta los recursos adicionales, como las puertas de enlace y el almacenamiento, que también puede ser necesario trasladar. Con cualquier plan, es importante completar una o más operaciones de traslado de prueba con servidores de prueba antes del traslado a un servidor de producción.

> [!IMPORTANT]
> Las aplicaciones cliente y las cadenas de conexión se conectan a Analysis Services mediante el nombre completo del servidor, que es un identificador URI que incluye la región en la que se encuentra el servidor. Por ejemplo, `asazure://westcentralus.asazure.windows.net/advworks01`. Al trasladar un servidor a otra región, se crea un nuevo recurso de servidor en dicha región, que tendrá una región diferente en el identificador URI del nombre del servidor. Las aplicaciones cliente y las cadenas de conexión que se usan en los scripts se deben conectar al nuevo servidor con el nuevo identificador URI del nombre del servidor. El uso de un [alias de nombre de servidor](analysis-services-server-alias.md) puede mitigar el número de lugares en los que se debe cambiar el identificador URI del nombre de servidor, pero se debe implementar antes de un traslado de región.

> [!IMPORTANT]
> Las regiones de Azure usan distintos intervalos de direcciones IP. Si tiene configuradas excepciones de firewall para la región en la que se encuentran el servidor o la cuenta de almacenamiento, puede que sea necesario configurar un intervalo de direcciones IP diferente. Para más información, consulte [Preguntas frecuentes acerca de la conectividad de red de Analysis Services](analysis-services-network-faq.md).

> [!NOTE]
> En este artículo se describe cómo restaurar una copia de seguridad de base de datos en un servidor de destino desde un contenedor de almacenamiento de la región del servidor de origen. En algunos casos, la restauración de las copias de seguridad desde otra región puede tener un rendimiento deficiente, especialmente en el caso de bases de datos de gran tamaño. Para obtener el mejor rendimiento durante la restauración de la base de datos, migre o cree un nuevo contenedor de almacenamiento en la región del servidor de destino. Copie los archivos .abf de la copia de seguridad del contenedor de almacenamiento de la región de origen en el contenedor de almacenamiento de la región de destino antes de restaurar la base de datos en el servidor de destino. Aunque queda fuera del ámbito de este artículo, en algunos casos, especialmente en el caso de las bases de datos muy grandes, la creación de scripts de una base de datos desde el servidor de origen, la nueva creación y el procesamiento en el servidor de destino para cargar los datos de la base de datos puede ser más rentable que el uso de copias de seguridad y restauración.

> [!NOTE]
> Si usa una puerta de enlace de datos local para conectarse a los orígenes de datos, también debe trasladar el recurso de puerta de enlace a la región del servidor de destino. Para más información, consulte [Instalación y configuración de una puerta de enlace de datos local](analysis-services-gateway-install.md).

## <a name="prerequisites"></a>Requisitos previos

- **Cuenta de Azure Storage**: necesaria para almacenar un archivo .abf de la copia de seguridad.
- **SQL Server Management Studio (SSMS)** : necesario para realizar una copia de seguridad y restaurar las bases de datos modelo.
- **Azure PowerShell**. solo es necesario si decide completar esta tarea mediante PowerShell.

## <a name="prepare"></a>Preparación

### <a name="backup-model-databases"></a>Copia de seguridad de las bases de datos modelo

Si las **opciones de almacenamiento** aún no están configuradas para el servidor de origen, siga los pasos que se describen en [Configuración de las opciones de almacenamiento](analysis-services-backup.md#configure-storage-settings).

Cuando estén configuradas las opciones de almacenamiento, siga los pasos descritos en [Copia de seguridad](analysis-services-backup.md#backup) para crear una copia de seguridad del archivo .abf de la base de datos modelo en el contenedor de almacenamiento. Después, restaure la copia de seguridad en formato .abf en el nuevo servidor de destino.


### <a name="export-template"></a>Exportación de la plantilla

La plantilla contiene las propiedades de configuración del servidor de origen.

# <a name="portal"></a>[Portal](#tab/azure-portal)

Para exportar una plantilla mediante Azure Portal:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

2. Seleccione **Todos los recursos** y seleccione el servidor de Analysis Services.

3. Seleccione > **Configuración** > **Exportar plantilla**.

4. Elija **Descargar** en la hoja **Exportar plantilla**.

5. Busque el archivo ZIP que descargó desde el portal y, a continuación, descomprima el archivo en una carpeta.

   El archivo ZIP contiene los archivos .json que componen la plantilla y los parámetros necesarios para implementar un nuevo servidor.


# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Para exportar una plantilla mediante PowerShell:

1. Inicie sesión en la suscripción a Azure con el comando [Connect-AzAccount](https://docs.microsoft.com/powershell/module/az.accounts/connect-azaccount?view=azps-2.5.0) y siga las instrucciones de la pantalla:

   ```azurepowershell-interactive
   Connect-AzAccount
   ```
2. Si su identidad está asociada a más de una suscripción, establezca su suscripción activa en la suscripción del recurso de servidor que quiere trasladar.

   ```azurepowershell-interactive
   $context = Get-AzSubscription -SubscriptionId <subscription-id>
   Set-AzContext $context
   ```

3. Exporte la plantilla del servidor de origen. Estos comandos guardan una plantilla JSON con el nombre del grupo de recursos como nombre de archivo en el directorio actual.

   ```azurepowershell-interactive
   $resource = Get-AzResource `
     -ResourceGroupName <resource-group-name> `
     -ResourceName <server-account-name> `
     -ResourceType Microsoft.AnalysisServices/servers
   Export-AzResourceGroup `
     -ResourceGroupName <resource-group-name> `
     -Resource $resource.ResourceId
   ```
---

### <a name="get-storage-shared-access-signature-sas"></a>Obtención de una firma de acceso compartido (SAS) de almacenamiento

Al implementar un servidor de destino desde una plantilla, se requiere un token de SAS de delegación de usuario (como un identificador URI) para especificar el contenedor de almacenamiento que contiene la copia de seguridad de la base de datos.

# <a name="portal"></a>[Portal](#tab/azure-portal)

Para obtener una firma de acceso compartido mediante el portal:

1. En el portal, seleccione la cuenta de almacenamiento que se utilizó para la copia de seguridad de la base de datos del servidor.

2. Seleccione **Explorador de Storage** y, a continuación, expanda **Contenedores de blob**. 

3. Haga clic con el botón derecho en el contenedor de almacenamiento y seleccione **Obtener firma de acceso compartido**.

    :::image type="content" source="media/move-between-regions/get-sas.png" alt-text="Obtener SAS":::

4. En **Firma de acceso compartido**, seleccione **Crear**. De forma predeterminada, la firma de acceso compartido expirará en 24 horas.

5. Copie y guarde el **identificador URI**. 

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Para obtener una firma de acceso compartido mediante PowerShell, siga los pasos descritos en [Creación de una firma de acceso compartido de delegación de usuario para un contenedor o un blob con PowerShell](../storage/blobs/storage-blob-user-delegation-sas-create-powershell.md#create-a-user-delegation-sas-for-a-blob).

---

### <a name="modify-the-template"></a>Modificación de la plantilla

Use un editor de texto para modificar el archivo template.json exportado y cambie las propiedades correspondientes a la región y el contenedor de blobs. 

Para modificar la plantilla:

1. En un editor de texto, en la propiedad **location** (ubicación), especifique la nueva región de destino. En la propiedad **backupBlobContainerUri**, pegue el identificador URI del contenedor de almacenamiento con la clave de SAS. 

    En el ejemplo siguiente se establece la región de destino del servidor advworks1 en `South Central US` y se especifica el identificador URI del contenedor de almacenamiento con la firma de acceso compartido: 

    ```json
    "resources": [
        {
            "type": "Microsoft.AnalysisServices/servers",
            "apiVersion": "2017-08-01",
            "name": "[parameters('servers_advworks1_name')]",
            "location": "South Central US",
            "sku": {
                "name": "S1",
                "tier": "Standard",
                "capacity": 1
            },
            "properties": {
                "asAdministrators": {
                    "members": [
                        "asadmins@adventure-works.com"
                    ]
                },
                "backupBlobContainerUri": "https://storagenorthcentralus.blob.core.windows.net/backup?sp=rl&st=2020-06-01T19:30:42Z&se=2020-06-02T19:30:42Z&sv=2019-10-10&sr=c&sig=PCQ4s9RujJkxu89gO4tiDTbE3%2BFECx6zAdcv8x0cVUQ%3D",
                "querypoolConnectionMode": "All"
            }
        }
    ]         
    ```
2. Guarde la plantilla.

#### <a name="regions"></a>Regions

Para obtener las regiones de Azure, consulte [Ubicaciones de Azure](https://azure.microsoft.com/global-infrastructure/locations/). Para obtener las regiones mediante PowerShell, ejecute el comando [Get-AzLocation](https://docs.microsoft.com/powershell/module/az.resources/get-azlocation?view=azps-1.8.0).

```azurepowershell-interactive
   Get-AzLocation | format-table 
```

## <a name="move"></a>Move

Para implementar un nuevo recurso de servidor en otra región, usará el archivo **template.json** que exportó y modificó en las secciones anteriores.

# <a name="portal"></a>[Portal](#tab/azure-portal)

1. En el portal, seleccione **Crear un recurso**.

2. En **Buscar en Marketplace**, escriba **implementación de plantillas** y, después, presione **ENTRAR**.

3. Seleccione **Implementación de plantillas**.

4. Seleccione **Crear**.

5. Seleccione **Cree su propia plantilla en el editor**.

6. Seleccione **Cargar archivo** y, después, siga las instrucciones para cargar el archivo **template.json** que exportó y modificó.

7. Compruebe que el editor de plantillas muestra las propiedades correctas para el nuevo servidor de destino.

8. Seleccione **Guardar**.

9. Escriba o seleccione los valores de propiedad:

    - **Suscripción**: seleccione la suscripción de Azure.
    
    - **Grupo de recursos**: seleccione **Crear nuevo** y escriba un nombre para el grupo de recursos. Puede seleccionar un grupo de recursos existente siempre que no contenga un servidor de Analysis Services con el mismo nombre.
    
    - **Ubicación**: seleccione la misma región que especificó en la plantilla.

10. Seleccione **Revisar y crear**.

11. Revise los términos y conceptos básicos y, a continuación, seleccione **Crear**.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Use estos comandos para implementar la plantilla:

```azurepowershell-interactive
   $resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
   $location = Read-Host -Prompt "Enter the location (i.e. South Central US)"

   New-AzResourceGroup -Name $resourceGroupName -Location "$location"
   New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri "<local template file>"  
   ```
---

### <a name="get-target-server-uri"></a>Obtención del identificador URI del servidor de destino

Para conectarse al nuevo servidor de destino desde SSMS para restaurar la base de datos modelo, debe obtener el identificador URI del nuevo servidor de destino.

# <a name="portal"></a>[Portal](#tab/azure-portal)

Para obtener el identificador URI del servidor en el portal:

1. En el portal, vaya al nuevo recurso de servidor de destino.

2. En la página **Información general**, copie el identificador URI de **Nombre de servidor**.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Para obtener el identificador URI del servidor con PowerShell, use los comandos siguientes:

```azurepowershell-interactive
   $resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
   $serverName = Read-Host -Prompt "Enter the server name (i.e. advworks2)"

   Get-AzAnalysisServicesServer -ResourceGroupName $resourceGroupName -Name "$serverName"
```
Copie el valor de **ServerFullName** de la salida.

---

### <a name="restore-model-database"></a>Restauración de la base de datos modelo

Siga los pasos descritos en [Restauración](analysis-services-backup.md#restore) para restaurar el archivo .abf de la copia de seguridad de la base de datos modelo en el nuevo servidor de destino.

Opcional: Después de restaurar la base de datos modelo, procese el modelo y las tablas para actualizar los datos desde los orígenes de datos. Para procesar el modelo y la tabla mediante SSMS:

1. En SSMS, haga clic con el botón derecho en la base de datos modelo > **Procesar base de datos**.

2. Expanda **Tablas** y haga clic con el botón derecho en una tabla. En **Procesar tablas**, seleccione todas las tablas y, a continuación, seleccione **Aceptar**.

## <a name="verify"></a>Verify

1. En el portal, vaya al nuevo servidor de destino.

2. En la página Información general, en **Modelos en Analysis Services Server**, compruebe que aparecen los modelos restaurados.

3. Use una aplicación cliente como Power BI o Excel para conectarse al modelo en el nuevo servidor. Compruebe que aparecen los objetos del modelo, como tablas, medidas y jerarquías. 

4. Ejecute los scripts de automatización. Compruebe que se han ejecutado correctamente.

Opcional: El [kit de herramientas de ALM](http://alm-toolkit.com/) es una herramienta de *código abierto* para comparar y administrar conjuntos de datos de Power BI *y* bases de datos de modelo tabulares de Analysis Services. Use el kit de herramientas para conectarse a las bases de datos del servidor de origen y de destino y compararlas. Si la migración de la base de datos se realiza correctamente, los objetos de modelo tendrán la misma definición. 

:::image type="content" source="media/move-between-regions/alm-toolkit.png" alt-text="Kit de herramientas de ALM":::

## <a name="clean-up-resources"></a>Limpieza de recursos

Después de comprobar que las aplicaciones cliente se pueden conectar al nuevo servidor y que los scripts de automatización se ejecutan correctamente, elimine el servidor de origen. 

# <a name="portal"></a>[Portal](#tab/azure-portal)

Para eliminar el servidor de origen desde el portal:

En la página **Información general** del servidor de origen, seleccione **Eliminar**.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Para eliminar el servidor de origen mediante PowerShell, use el comando Remove-AzAnalysisServicesServer.

```azurepowershell-interactive
Remove-AzAnalysisServicesServer -Name "myserver" -ResourceGroupName "myResourceGroup"
```

---

> [!NOTE]
> Después de completar el traslado de región, se recomienda que el nuevo servidor de destino use un contenedor de almacenamiento de la misma región para las copias de seguridad, en lugar del contenedor de almacenamiento de la región del servidor de origen. 
