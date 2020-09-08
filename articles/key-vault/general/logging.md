---
title: Registro de Azure Key Vault | Microsoft Docs
description: Aprenda a supervisar el acceso a los almacenes de claves habilitando el registro para Azure Key Vault, con lo que se guarda la información en una cuenta de almacenamiento de Azure proporcionada por el usuario.
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: how-to
ms.date: 08/12/2019
ms.author: mbaldwin
ms.openlocfilehash: e9507525dc2c52f584bd7883a12da401b5999f50
ms.sourcegitcommit: 02ca0f340a44b7e18acca1351c8e81f3cca4a370
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88585923"
---
# <a name="azure-key-vault-logging"></a>Registro de Azure Key Vault

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Después de crear uno o varios almacenes de claves, es probable que desee supervisar cómo y cuándo se accede a los almacenes de claves y quién lo hace. Puede hacerlo al habilitar el registro de Azure Key Vault, que guarda la información en una cuenta de almacenamiento de Azure proporcionada por el usuario. Un nuevo contenedor denominado **insights-logs-auditevent** se crea automáticamente para su cuenta de almacenamiento especificada. Puede usar esta misma cuenta de almacenamiento para recopilar registros de varios almacenes de claves.

Puede acceder a la información de registro 10 minutos (como máximo) después de la operación del almacén de claves. En la mayoría de los casos, será más rápido que esto.  Es decisión suya administrar los registros en la cuenta de almacenamiento:

* Utilice los métodos de control de acceso de Azure estándar para proteger los registros mediante la restricción de quién puede tener acceso a ellos.
* Elimine los registros que ya no desee mantener en la cuenta de almacenamiento.

Use este tutorial para tener ayuda para empezar a trabajar con el registro de Azure Key Vault. Deberá crear una cuenta de almacenamiento, habilitar el registro e interpretar la información de los registros recopilados.  

> [!NOTE]
> Este tutorial no incluye instrucciones sobre cómo crear almacenes de claves, claves o secretos. Para más información, consulte [¿Qué es Azure Key Vault?](overview.md). O bien, para obtener instrucciones de la CLI de Azure para varias plataformas, consulte [este tutorial equivalente](manage-with-cli2.md).
>
> En este artículo se ofrecen instrucciones de Azure PowerShell para actualizar el registro de diagnóstico. También puede actualizar el registro de diagnóstico mediante el uso de Azure Monitor en la sección **Registros de diagnóstico** de Azure Portal. 
>

Para información general sobre Azure Key Vault, consulte [¿Qué es Azure Key Vault?](overview.md). Para obtener información acerca de dónde está disponible Key Vault, consulte la [página de precios](https://azure.microsoft.com/pricing/details/key-vault/).

## <a name="prerequisites"></a>Requisitos previos

Para realizar este tutorial, debe disponer de lo siguiente:

* Un Almacén de claves existente que ha utilizado.  
* Azure PowerShell, versión mínima de 1.0.0. Para instalar Azure PowerShell y asociarlo con una suscripción de Azure, consulte [Instalación y configuración de Azure PowerShell](/powershell/azure/). Si ya instaló Azure PowerShell y no sabe la versión, en la consola de Azure PowerShell, escriba `$PSVersionTable.PSVersion`.  
* Suficiente almacenamiento en Azure para sus registros de Key Vault.

## <a name="connect-to-your-key-vault-subscription"></a><a id="connect"></a>Conexión a su suscripción del almacén de claves

El primer paso para configurar el registro de claves es hacer que Azure PowerShell apunte al registro de claves que desea registrar.

Inicie una sesión de Azure PowerShell e inicie sesión en su cuenta de Azure con el siguiente comando:  

```powershell
Connect-AzAccount
```

En la ventana emergente del explorador, escriba el nombre de usuario y la contraseña de su cuenta de Azure. Azure PowerShell obtiene todas las suscripciones asociadas a esta cuenta. PowerShell utiliza la primera de forma predeterminada.

Es posible que deba especificar la suscripción que usó para crear el almacén de claves. Para ver las suscripciones de su cuenta, escriba el siguiente comando:

```powershell
Get-AzSubscription
```

Luego, para especificar la suscripción asociada al almacén de claves que se va registrar, escriba:

```powershell
Set-AzContext -SubscriptionId <subscription ID>
```

Hacer que PowerShell apunte a la suscripción correcta es un paso importante, especialmente si tiene varias suscripciones asociadas a su cuenta. Para obtener más información sobre cómo configurar PowerShell de Azure, consulte [Instalación y configuración de PowerShell de Azure](/powershell/azure/).

## <a name="create-a-storage-account-for-your-logs"></a><a id="storage"></a>Creación de una cuenta de almacenamiento para sus registros

Aunque puede usar una cuenta de almacenamiento existente para sus registros, crearemos una cuenta de almacenamiento que se dedicará a los registros de Key Vault. Por comodidad para cuando tengamos que especificarlos más adelante, almacenaremos los detalles en una variable denominada **sa**.

Para una mayor facilidad de administración, también usaremos el grupo de recursos que contiene el almacén de claves. Desde el [tutorial de introducción](../secrets/quick-create-cli.md), este grupo de recursos se denomina **ContosoResourceGroup**, y seguiremos usando la ubicación Este de Asia. Sustituya estos valores por los suyos propios, según corresponda:

```powershell
 $sa = New-AzStorageAccount -ResourceGroupName ContosoResourceGroup -Name contosokeyvaultlogs -Type Standard_LRS -Location 'East Asia'
```

> [!NOTE]
> Si decide usar una cuenta de almacenamiento existente, debe usar la misma suscripción que el almacén de claves. Y debe usar el modelo de implementación de Resource Manager, en lugar del modelo de implementación clásica.
>
>

## <a name="identify-the-key-vault-for-your-logs"></a><a id="identify"></a>Identificación del Almacén de claves para los registros

En el [tutorial de introducción](../secrets/quick-create-cli.md), el nombre del almacén de claves era **ContosoKeyVault**. Vamos a seguir usando ese nombre y almacenando los detalles en una variable denominada **kv**:

```powershell
$kv = Get-AzKeyVault -VaultName 'ContosoKeyVault'
```

## <a name="enable-logging-using-azure-powershell"></a><a id="enable"></a>Habilitación del registro con Azure PowerShell

Para habilitar el registro de Key Vault, usaremos el cmdlet **Set-AzDiagnosticSetting**, junto con las variables que hemos creado para la nueva cuenta de almacenamiento y el almacén de claves. También estableceremos la marca **-Enabled** en **$true** y la categoría en `AuditEvent` (la única categoría del registro de Key Vault):

```powershell
Set-AzDiagnosticSetting -ResourceId $kv.ResourceId -StorageAccountId $sa.Id -Enabled $true -Category AuditEvent
```

El resultado tendrá un aspecto similar al siguiente:

```output
StorageAccountId   : /subscriptions/<subscription-GUID>/resourceGroups/ContosoResourceGroup/providers/Microsoft.Storage/storageAccountContosoKeyVaultLogs
ServiceBusRuleId   :
StorageAccountName :
    Logs
    Enabled           : True
    Category          : AuditEvent
    RetentionPolicy
    Enabled : False
    Days    : 0
```

Este resultado confirma que el registro está habilitado ahora para el almacén de claves, y guardará la información en su cuenta de almacenamiento.

Opcionalmente, puede establecer una directiva de retención para los registros, de forma que los registros más antiguos se eliminen automáticamente. Por ejemplo, establezca la directiva de retención mediante la configuración de la marca **-RetentionEnabled** en **$true**, y establezca el parámetro **-RetentionInDays** en **90**, con el fin de que los registros que tengan más de 90 días se eliminen automáticamente.

```powershell
Set-AzDiagnosticSetting -ResourceId $kv.ResourceId -StorageAccountId $sa.Id -Enabled $true -Category AuditEvent -RetentionEnabled $true -RetentionInDays 90
```

Qué se registra:

* Todas las solicitudes de la API REST autenticadas, incluidas las solicitudes con error debido a permisos de acceso, errores del sistema o solicitudes incorrectas.
* Las operaciones en el almacén de claves, incluidas la creación, la eliminación, la definición de políticas de acceso del almacén de claves y la actualización de los atributos del almacén de claves, como las etiquetas.
* Las operaciones en claves y secretos del almacén de claves, incluido lo siguiente:
  * Crear, modificar o eliminar estas claves o secretos.
  * Firmar, comprobar, cifrar, descifrar, encapsular y desencapsular claves, obtener secretos y elaborar listados de claves y secretos (y sus versiones).
* Solicitudes no autenticadas que dan como resultado una respuesta 401. Por ejemplo, las solicitudes que no tienen un token de portador, cuyo formato es incorrecto o está caducado o que tienen un token no válido.  

## <a name="enable-logging-using-azure-cli"></a>Habilitación del registro mediante la CLI de Azure

```azurecli
az login

az account set --subscription {AZURE SUBSCRIPTION ID}

az provider register -n Microsoft.KeyVault

az monitor diagnostic-settings create  \
--name KeyVault-Diagnostics \
--resource /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myresourcegroup/providers/Microsoft.KeyVault/vaults/mykeyvault \
--logs    '[{"category": "AuditEvent","enabled": true}]' \
--metrics '[{"category": "AllMetrics","enabled": true}]' \
--storage-account /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myresourcegroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount \
--workspace /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/oi-default-east-us/providers/microsoft.operationalinsights/workspaces/myworkspace \
--event-hub-rule /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myresourcegroup/providers/Microsoft.EventHub/namespaces/myeventhub/authorizationrules/RootManageSharedAccessKey
```

## <a name="access-your-logs"></a><a id="access"></a>Acceso a los registros

Los registros de Key Vault se almacenan en el contenedor **insights-logs-auditevent** de la cuenta de almacenamiento proporcionada. Para ver los registros, tendrá que descargar blobs.

En primer lugar, cree una variable para el nombre del contenedor. Usará esta variable en el resto del tutorial.

```powershell
$container = 'insights-logs-auditevent'
```

Para mostrar una lista de todos los blobs de este contenedor, escriba:

```powershell
Get-AzStorageBlob -Container $container -Context $sa.Context
```

La salida es similar a esta:

```
Container Uri: https://contosokeyvaultlogs.blob.core.windows.net/insights-logs-auditevent

Name

---
resourceId=/SUBSCRIPTIONS/361DA5D4-A47A-4C79-AFDD-XXXXXXXXXXXX/RESOURCEGROUPS/CONTOSORESOURCEGROUP/PROVIDERS/MICROSOFT.KEYVAULT/VAULTS/CONTOSOKEYVAULT/y=2016/m=01/d=05/h=01/m=00/PT1H.json

resourceId=/SUBSCRIPTIONS/361DA5D4-A47A-4C79-AFDD-XXXXXXXXXXXX/RESOURCEGROUPS/CONTOSORESOURCEGROUP/PROVIDERS/MICROSOFT.KEYVAULT/VAULTS/CONTOSOKEYVAULT/y=2016/m=01/d=04/h=02/m=00/PT1H.json

resourceId=/SUBSCRIPTIONS/361DA5D4-A47A-4C79-AFDD-XXXXXXXXXXXX/RESOURCEGROUPS/CONTOSORESOURCEGROUP/PROVIDERS/MICROSOFT.KEYVAULT/VAULTS/CONTOSOKEYVAULT/y=2016/m=01/d=04/h=18/m=00/PT1H.json
```

Como puede ver en esta salida, los blobs siguen una convención de nomenclatura: `resourceId=<ARM resource ID>/y=<year>/m=<month>/d=<day of month>/h=<hour>/m=<minute>/filename.json`

Los valores de fecha y hora usan UTC.

Puesto que la misma cuenta de almacenamiento puede usarse para recopilar registros de varios recursos, el identificador de recurso completo en el nombre del blob es útil para acceder o descargar solo los blobs que necesita. Pero antes de hacerlo, primero explicaremos cómo descargar todos los blobs.

Cree una carpeta para descargar los blobs. Por ejemplo:

```powershell 
New-Item -Path 'C:\Users\username\ContosoKeyVaultLogs' -ItemType Directory -Force
```

A continuación, obtenga una lista de todos los blobs:  

```powershell
$blobs = Get-AzStorageBlob -Container $container -Context $sa.Context
```

Canalice esta lista mediante **Get-AzStorageBlobContent** para descargar los blobs en la carpeta de destino:

```powershell
$blobs | Get-AzStorageBlobContent -Destination C:\Users\username\ContosoKeyVaultLogs'
```

Al ejecutar este segundo comando, el delimitador **/** de los nombres de los blobs crea una estructura de carpeta completa en la carpeta de destino. Usará esta estructura para descargar y almacenar los blobs como archivos.

Para descargar blobs de forma selectiva, utilice caracteres comodín. Por ejemplo:

* Si tiene varios almacenes de claves y desea descargar los registros solo para un Almacén de claves, denominado CONTOSOKEYVAULT3:

  ```powershell
  Get-AzStorageBlob -Container $container -Context $sa.Context -Blob '*/VAULTS/CONTOSOKEYVAULT3
  ```

* Si tiene varios grupos de recursos y desea descargar los registros solo de un grupo de recursos específico, use `-Blob '*/RESOURCEGROUPS/<resource group name>/*'`:

  ```powershell
  Get-AzStorageBlob -Container $container -Context $sa.Context -Blob '*/RESOURCEGROUPS/CONTOSORESOURCEGROUP3/*'
  ```

* Si desea descargar todos los registros del mes de enero de 2019, use `-Blob '*/year=2019/m=01/*'`:

  ```powershell
  Get-AzStorageBlob -Container $container -Context $sa.Context -Blob '*/year=2016/m=01/*'
  ```

Ahora está listo para comenzar a ver lo que está en los registros. Pero antes de pasar a ello, debe conocer dos comandos más:

* Para consultar el estado de la configuración del diagnóstico del recurso del almacén de claves: `Get-AzDiagnosticSetting -ResourceId $kv.ResourceId`
* Para deshabilitar el registro del recurso del almacén de claves: `Set-AzDiagnosticSetting -ResourceId $kv.ResourceId -StorageAccountId $sa.Id -Enabled $false -Category AuditEvent`


## <a name="interpret-your-key-vault-logs"></a><a id="interpret"></a>Interpretación de los registros de Key Vault

Los blobs individuales se almacenan como texto, con formato de blob JSON. Veamos una entrada de registro como ejemplo. 

```json
    {
        "records":
        [
            {
                "time": "2016-01-05T01:32:01.2691226Z",
                "resourceId": "/SUBSCRIPTIONS/361DA5D4-A47A-4C79-AFDD-XXXXXXXXXXXX/RESOURCEGROUPS/CONTOSOGROUP/PROVIDERS/MICROSOFT.KEYVAULT/VAULTS/CONTOSOKEYVAULT",
                "operationName": "VaultGet",
                "operationVersion": "2015-06-01",
                "category": "AuditEvent",
                "resultType": "Success",
                "resultSignature": "OK",
                "resultDescription": "",
                "durationMs": "78",
                "callerIpAddress": "104.40.82.76",
                "correlationId": "",
                "identity": {"claim":{"http://schemas.microsoft.com/identity/claims/objectidentifier":"d9da5048-2737-4770-bd64-XXXXXXXXXXXX","http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn":"live.com#username@outlook.com","appid":"1950a258-227b-4e31-a9cf-XXXXXXXXXXXX"}},
                "properties": {"clientInfo":"azure-resource-manager/2.0","requestUri":"https://control-prod-wus.vaultcore.azure.net/subscriptions/361da5d4-a47a-4c79-afdd-XXXXXXXXXXXX/resourcegroups/contosoresourcegroup/providers/Microsoft.KeyVault/vaults/contosokeyvault?api-version=2015-06-01","id":"https://contosokeyvault.vault.azure.net/","httpStatusCode":200}
            }
        ]
    }
```

En la tabla siguiente se muestran los nombres y las descripciones de los campos:

| Nombre del campo | Descripción |
| --- | --- |
| **time** |Fecha y hora en UTC. |
| **resourceId** |Identificador de recursos de Azure Resource Manager. Para los registros de Key Vault, siempre es el identificador de recurso de Key Vault. |
| **operationName** |Nombre de la operación, como se documenta en la tabla siguiente. |
| **operationVersion** |Versión de la API REST solicitada por el cliente. |
| **category** |Tipo de resultado. Para los registros de Key Vault, `AuditEvent` es el único valor disponible. |
| **resultType** |Resultado de la solicitud de la API REST. |
| **resultSignature** |Estado de HTTP |
| **resultDescription** |Descripción adicional acerca del resultado, cuando esté disponible. |
| **durationMs** |Tiempo que tardó en atender la solicitud de API de REST, en milisegundos. Esto no incluye la latencia de red, por lo que el tiempo que se mide en el cliente podría no coincidir con este tiempo. |
| **callerIpAddress** |Dirección IP del cliente que realizó la solicitud. |
| **correlationId** |Un GUID opcional que el cliente puede pasar para correlacionar los registros del lado cliente con los registros del lado servicio (Key Vault). |
| **identity** |Identidad del token que se ha presentado al realizar la solicitud de la API REST. Suele ser un "usuario", una "entidad de servicio" o una combinación de "usuario + appId", como en el caso de una solicitud procedente de un cmdlet de Azure PowerShell. |
| **properties** |Información que varía en función de la operación (**operationName**). En la mayoría de los casos, este campo contiene información del cliente (la cadena del agente de usuario pasada por el cliente), el URI de la solicitud de la API REST exacta y el código de estado HTTP. Además, cuando se devuelve un objeto como resultado de una solicitud (por ejemplo, **KeyCreate** o **VaultGet**) también contiene el URI de la clave (como `id`), el URI del almacén o el URI del secreto. |

Los valores del campo **operationName** tienen el formato *ObjectVerb*. Por ejemplo:

* Todas las operaciones del almacén de claves tienen el formato `Vault<action>`, como `VaultGet` y `VaultCreate`.
* Todas las operaciones de claves tienen el formato `Key<action>`, como `KeySign` y `KeyList`.
* Todas las operaciones de secretos tienen el formato `Secret<action>`, como `SecretGet` y `SecretListVersions`.

En la tabla siguiente se muestran los valores **operationName** y los comandos correspondientes de la API REST:

| operationName | Comando de la API REST |
| --- | --- |
| **Autenticación** |Autenticar mediante un punto de conexión de Azure Active Directory |
| **VaultGet** |[Obtener información acerca de un almacén de claves](https://msdn.microsoft.com/library/azure/mt620026.aspx) |
| **VaultPut** |[Crear o actualizar un almacén de claves](https://msdn.microsoft.com/library/azure/mt620025.aspx) |
| **VaultDelete** |[Eliminar un almacén de claves](https://msdn.microsoft.com/library/azure/mt620022.aspx) |
| **VaultPatch** |[Crear o actualizar un almacén de claves](https://msdn.microsoft.com/library/azure/mt620025.aspx) |
| **VaultList** |[Lista de todos los almacenes de claves en un grupo de recursos](https://msdn.microsoft.com/library/azure/mt620027.aspx) |
| **KeyCreate** |[Crear una clave](https://msdn.microsoft.com/library/azure/dn903634.aspx) |
| **KeyGet** |[Obtener información sobre una clave](https://msdn.microsoft.com/library/azure/dn878080.aspx) |
| **KeyImport** |[Importar una clave a un almacén](https://msdn.microsoft.com/library/azure/dn903626.aspx) |
| **KeyBackup** |[Hacer una copia de seguridad de una clave](https://msdn.microsoft.com/library/azure/dn878058.aspx) |
| **KeyDelete** |[Eliminar una clave](https://msdn.microsoft.com/library/azure/dn903611.aspx) |
| **KeyRestore** |[Restauración de una clave](https://msdn.microsoft.com/library/azure/dn878106.aspx) |
| **KeySign** |[Firmar con una clave](https://msdn.microsoft.com/library/azure/dn878096.aspx) |
| **KeyVerify** |[Comprobar con una clave](https://msdn.microsoft.com/library/azure/dn878082.aspx) |
| **KeyWrap** |[Encapsular una clave](https://msdn.microsoft.com/library/azure/dn878066.aspx) |
| **KeyUnwrap** |[Desencapsular una clave](https://msdn.microsoft.com/library/azure/dn878079.aspx) |
| **KeyEncrypt** |[Cifrar con una clave](https://msdn.microsoft.com/library/azure/dn878060.aspx) |
| **KeyDecrypt** |[Descifrado con una clave](https://msdn.microsoft.com/library/azure/dn878097.aspx) |
| **KeyUpdate** |[Actualizar una clave](https://msdn.microsoft.com/library/azure/dn903616.aspx) |
| **KeyList** |[Enumeración de las claves en un almacén](https://msdn.microsoft.com/library/azure/dn903629.aspx) |
| **KeyListVersions** |[Enumerar las versiones de una clave](https://msdn.microsoft.com/library/azure/dn986822.aspx) |
| **SecretSet** |[Crear un secreto](https://msdn.microsoft.com/library/azure/dn903618.aspx) |
| **SecretGet** |[Obtener un secreto](https://msdn.microsoft.com/library/azure/dn903633.aspx) |
| **SecretUpdate** |[Actualizar un secreto](https://msdn.microsoft.com/library/azure/dn986818.aspx) |
| **SecretDelete** |[Eliminar un secreto](https://msdn.microsoft.com/library/azure/dn903613.aspx) |
| **SecretList** |[Enumerar secretos en un almacén](https://msdn.microsoft.com/library/azure/dn903614.aspx) |
| **SecretListVersions** |[Enumerar versiones de un secreto](https://msdn.microsoft.com/library/azure/dn986824.aspx) |

## <a name="use-azure-monitor-logs"></a><a id="loganalytics"></a>Uso de registros de Azure Monitor

Puede utilizar la solución Key Vault en registros de Azure Monitor para revisar los registros `AuditEvent` de Key Vault. En los registros de Azure Monitor, las consultas de los registros se usan para analizar los datos y obtener la información que necesita. 

Para más información, incluido cómo configurar esta opción, consulte [Azure Key Vault en Azure Monitor](../../azure-monitor/insights/key-vault-insights-overview.md).

## <a name="next-steps"></a><a id="next"></a>Pasos siguientes

Para ver un tutorial que use Azure Key Vault en una aplicación web .NET, consulte [Uso de Azure Key Vault desde una aplicación web](tutorial-net-create-vault-azure-web-app.md).

Para conocer las referencias de programación, consulte la [Guía del desarrollador de Azure Key Vault](developers-guide.md).

Para obtener una lista de los cmdlets de Azure PowerShell 1.0 para Azure Key Vault, consulte [Azure Key Vault Cmdlets](/powershell/module/az.keyvault/?view=azps-1.2.0#key_vault)(Cmdlets de Azure Key Vault).
