---
title: Tutorial para pedir Azure Data Box | Microsoft Docs
description: En este tutorial obtendrá información sobre Azure Data Box, una solución híbrida que le permite importar datos locales en Azure y cómo pedir Azure Data Box.
services: databox
author: twooley
ms.service: databox
ms.subservice: pod
ms.topic: tutorial
ms.date: 07/21/2020
ms.author: twooley
ms.openlocfilehash: 2000ecc84a92bef5ad6b80fecde4aee0157e4bc5
ms.sourcegitcommit: 9c3cfbe2bee467d0e6966c2bfdeddbe039cad029
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/24/2020
ms.locfileid: "88783576"
---
# <a name="tutorial-order-azure-data-box"></a>Tutorial: Realización de pedidos de Azure Data Box

Azure Data Box es una solución híbrida que permite importar datos locales en Azure de una forma rápida, fácil y confiable. Transfiera los datos a un dispositivo de almacenamiento de 80 TB (capacidad utilizable) que suministra Microsoft y, después, devuelva el dispositivo. Luego, dichos datos se cargan en Azure.

En En este tutorial se describe cómo se puede solicitar Azure Data Box. En este tutorial, obtendrá información sobre lo siguiente:

> [!div class="checklist"]
>
> * Requisitos previos para implementar Data Box
> * Realizar un pedido de Data Box
> * Seguimiento del pedido
> * Cancelar el pedido

## <a name="prerequisites"></a>Requisitos previos

# <a name="portal"></a>[Portal](#tab/portal)

Antes de implementar el dispositivo, complete los siguientes requisitos previos de configuración tanto del servicio Data Box como del dispositivo:

[!INCLUDE [Prerequisites](../../includes/data-box-deploy-ordered-prerequisites.md)]

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

[!INCLUDE [Prerequisites](../../includes/data-box-deploy-ordered-prerequisites.md)]

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

Puede iniciar sesión en Azure y ejecutar los comandos de la CLI de Azure de dos maneras:

* Puede instalar la CLI y ejecutar sus comandos en local.
* Puede ejecutar los comandos de la CLI desde Azure Portal, en Azure Cloud Shell.

Para este tutorial vamos a usar la CLI de Azure mediante Windows PowerShell, pero puede elegir cualquiera de las opciones.

### <a name="for-azure-cli"></a>Para la CLI de Azure

Antes de comenzar, asegúrese de que:

#### <a name="install-the-cli-locally"></a>Instalación local de la CLI

* Instale la versión 2.0.67 o posterior de la [CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli). Como alternativa, puede [realizar la instalación mediante MSI](https://aka.ms/installazurecliwindows).

**Inicio de sesión en Azure**

Abra una ventana de comandos de Windows PowerShell e inicie sesión en Azure con el comando [az login](/cli/azure/reference-index#az-login):

```azurecli
PS C:\Windows> az login
```

Este es el resultado de un inicio de sesión correcto:

```output
You have logged in. Now let us find all the subscriptions to which you have access.
[
   {
      "cloudName": "AzureCloud",
      "homeTenantId": "aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa",
      "id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "isDefault": true,
      "managedByTenants": [],
      "name": "My Subscription",
      "state": "Enabled",
      "tenantId": "aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa",
      "user": {
          "name": "gusp@contoso.com",
          "type": "user"
      }
   }
]
```

**Instalación de la extensión de la CLI de Azure Data Box**

Para poder usar los comandos de la CLI de Azure Data Box, primero tiene que instalar la extensión. Las extensiones de la CLI de Azure le proporcionan acceso a comandos experimentales y en versión preliminar que todavía no se han enviado como parte de la CLI principal. Para más información, consulte [Uso de extensiones con la CLI de Azure](/cli/azure/azure-cli-extensions-overview).

Para instalar la extensión para Azure Data Box, ejecute el comando `az extension add --name databox`:

```azurecli

    PS C:\Windows> az extension add --name databox
```

Si la extensión se instala correctamente, verá el siguiente resultado:

```output
    The installed extension 'databox' is experimental and not covered by customer support. Please use with discretion.
    PS C:\Windows>

    # az databox help

    PS C:\Windows> az databox -h

    Group
        az databox

    Subgroups:
        job [Experimental] : Commands to manage databox job.

    For more specific examples, use: az find "az databox"

        Please let us know how we are doing: https://aka.ms/clihats
```

#### <a name="use-azure-cloud-shell"></a>Uso de Azure Cloud Shell

Puede usar [Azure Cloud Shell](https://shell.azure.com/) (un entorno de shell interactivo hospedado de Azure) en el explorador para ejecutar comandos de la CLI. Azure Cloud Shell admite Bash o Windows PowerShell con los servicios de Azure. La CLI de Azure está preinstalada y configurada para su uso con la cuenta. Seleccione el botón Cloud Shell del menú situado en la sección superior derecha de Azure Portal:

![Cloud Shell](../storage/common/media/storage-quickstart-create-account/cloud-shell-menu.png)

El botón inicia un shell interactivo que se puede usar para ejecutar los pasos de este artículo de procedimientos.

# <a name="powershell"></a>[PowerShell](#tab/azure-ps)

[!INCLUDE [Prerequisites](../../includes/data-box-deploy-ordered-prerequisites.md)]

### <a name="for-azure-powershell"></a>Para Azure PowerShell

Antes de comenzar, asegúrese de:

* Haber instalado Windows PowerShell 6.2.4 o una versión posterior.
* Haber instalado el módulo de Azure PowerShell (AZ).
* Haber instalado el módulo de Azure Data Box (Az.DataBox).
* Haber iniciado sesión en Azure.

#### <a name="install-azure-powershell-and-modules-locally"></a>Instalación de Azure PowerShell y los módulos en la máquina

**Instalación o actualización de Windows PowerShell**

Necesitará tener instalada la versión 6.2.4 o una posterior de Windows PowerShell. Para averiguar qué versión de PowerShell tiene instalada, ejecute `$PSVersionTable`.

Verá la salida siguiente:

```azurepowershell
    PS C:\users\gusp> $PSVersionTable
    
    Name                           Value
    ----                           -----
    PSVersion                      6.2.3
    PSEdition                      Core
    GitCommitId                    6.2.3
    OS                             Microsoft Windows 10.0.18363
    Platform                       Win32NT
    PSCompatibleVersions           {1.0, 2.0, 3.0, 4.0…}
    PSRemotingProtocolVersion      2.3
    SerializationVersion           1.1.0.1
    WSManStackVersion              3.0
```

Si su versión es anterior a la 6.2.4, deberá actualizarla. Para instalar la versión más reciente de Windows PowerShell, consulte [Instalación de Azure PowerShell](https://docs.microsoft.com/powershell/scripting/install/installing-powershell?view=powershell-7).

**Instalación de los módulos de Azure PowerShell y de Data Box**

Tendrá que instalar los módulos de Azure PowerShell para usarlo para pedir Azure Data Box. Para instalar los módulos de Azure PowerShell:

1. Instale el módulo [Az de Azure PowerShell](https://docs.microsoft.com/powershell/azure/new-azureps-module-az).
2. A continuación, instale Az.DataBox con el comando `Install-Module -Name Az.DataBox`.

```azurepowershell
PS C:\PowerShell\Modules> Install-Module -Name Az.DataBox
PS C:\PowerShell\Modules> Get-InstalledModule -Name "Az.DataBox"

Version              Name                                Repository           Description
-------              ----                                ----------           -----------
0.1.1                Az.DataBox                          PSGallery            Microsoft Azure PowerShell - DataBox ser…
```

#### <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Abra una ventana de comandos de Windows PowerShell e inicie sesión en Azure con el comando [Connect-AzAccount](https://docs.microsoft.com/powershell/module/az.accounts/Connect-AzAccount):

```azurepowershell
PS C:\Windows> Connect-AzAccount
```

Este es el resultado de un inicio de sesión correcto:

```output
WARNING: To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code FSBFZMBKC to authenticate.

Account              SubscriptionName                          TenantId                             Environment
-------              ----------------                          --------                             -----------
gusp@contoso.com     MySubscription                            aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa AzureCloud

PS C:\Windows\System32>
```

Para información detallada sobre cómo iniciar sesión en Azure con Windows PowerShell, consulte [Inicio de sesión con Azure PowerShell](https://docs.microsoft.com/powershell/azure/authenticate-azureps).

---

## <a name="order-data-box"></a>Realización de un pedido de Data Box

# <a name="portal"></a>[Portal](#tab/portal)

Para solicitar un dispositivo, realice los pasos siguientes en Azure Portal.

1. Use sus credenciales de Microsoft Azure para iniciar sesión en esta dirección URL: [https://portal.azure.com](https://portal.azure.com).
2. Seleccione **+ Crear un recurso** y busque *Azure Data Box*. Seleccione **Azure Data Box**.

   ![Selección de Azure Data Box](media/data-box-deploy-ordered/select-data-box-import-02.png)

3. Seleccione **Crear**.

   ![Selección de Azure Data Box](media/data-box-deploy-ordered/select-data-box-import-03.png)

4. Compruebe si el servicio Data Box está disponible en su región. Escriba o seleccione la siguiente información y seleccione **Aplicar**.

    |Configuración  |Value  |
    |---------|---------|
    |Tipo de transferencia     | Seleccione **Importar en Azure**.        |
    |Subscription     | Seleccione una suscripción patrocinada por EA, CSP o Azure para el servicio Data Box. <br> La suscripción está vinculada a la cuenta de facturación.       |
    |Grupo de recursos | Seleccione un grupo de recursos existente. Un grupo de recursos es un contenedor lógico para los recursos que se pueden administrar o implementar conjuntamente. |
    |País o región de origen    |    Seleccione el país o región en que residen los datos actualmente.         |
    |Región de Azure de destino     |     Seleccione la región de Azure a la que desea transferir los datos. <br> Para más información, vaya a [Disponibilidad por región](data-box-overview.md#region-availability).            |

    [![Pedido de importación de Azure Data Box](media/data-box-deploy-ordered/select-data-box-import-04b.png)](media/data-box-deploy-ordered/select-data-box-import-04b.png#lightbox)

5. Seleccione **Data Box**. La capacidad máxima utilizable para un solo pedido es de 80 TB. Para tamaños de datos mayores puede crear varios pedidos.

    ![Selección de la opción 1 de Data Box](media/data-box-deploy-ordered/select-data-box-import-05.png)

6. En **Pedido**, vaya a la pestaña **Datos básicos**. Escriba o seleccione la siguiente información y seleccione **Siguiente: destino de los datos>** .

    |Configuración  |Valor  |
    |---------|---------|
    |Suscripción      | La suscripción se rellena automáticamente según la selección anterior.|
    |Resource group    | Grupo de recursos especificado anteriormente. |
    |Nombre del pedido de importación | Especifique un nombre descriptivo para hacer un seguimiento del pedido. <br> El nombre puede tener entre 3 y 24 caracteres que pueden ser letras, números y guiones. <br> El nombre debe empezar y terminar con una letra o un número.    |

    ![Selección de la opción 1 de Data Box](media/data-box-deploy-ordered/select-data-box-import-06.png)

    De forma predeterminada, la contraseña de desbloqueo del dispositivo se cifra mediante una clave administrada por Microsoft. Una vez completado el pedido, podrá agregar una clave administrada por el cliente. Las claves administradas por el cliente permiten usar una clave propia de un almacén de claves de Azure Key Vault para proteger la contraseña de desbloqueo del dispositivo. Para más información, consulte [Uso de claves administradas por el cliente en Azure Key Vault para Azure Data Box](data-box-customer-managed-encryption-key-portal.md).

7. En la pestaña **Destino de datos**, seleccione **Destino de datos**.

    Si usa las **cuentas de almacenamiento** como destino de almacenamiento, consulte la siguiente captura de pantalla:

    ![Destino de datos de Azure Data Box](media/data-box-deploy-ordered/select-data-box-import-07.png)

    En función de la región de Azure especificada, seleccione una o varias cuentas de almacenamiento en la lista filtrada una cuenta de almacenamiento existente. Data Box se puede vincular con un máximo de diez cuentas de almacenamiento. También puede crear una nueva **cuenta de uso general v1**, **cuenta de uso general v2** o **cuenta de almacenamiento de blobs**.

    Se admiten cuentas de almacenamiento con redes virtuales. Para permitir que el servicio de Data Box trabaje con cuentas de almacenamiento seguro, habilite los servicios de confianza dentro de la configuración de firewall de la red de la cuenta de almacenamiento. Para obtener más información, vea cómo [agregar Azure Data Box como un servicio de confianza](../storage/common/storage-network-security.md#exceptions).

    Si usa Data Box para crear **discos administrados** desde los discos duros virtuales locales (VHD), también tendrá que proporcionar la siguiente información:

    |Configuración  |Value  |
    |---------|---------|
    |Grupos de recursos     | Cree grupos de recursos si pretende crear discos administrados desde los discos duros virtuales locales. Puede usar un grupo de recursos existente solo si el grupo de recursos se creó anteriormente al crear un pedido de Data Box para el disco administrado por el servicio Data Box. <br> Especifique varios grupos de recursos separados por punto y coma. Se admite un máximo de 10 grupos de recursos.|

    ![Solicitud de Data Box para disco administrado](media/data-box-deploy-ordered/select-data-box-import-07b.png)

    La cuenta de almacenamiento especificada para los discos administrados se usa como una cuenta de almacenamiento provisional. El servicio Data Box carga los discos duros virtuales como blob en páginas en la cuenta de almacenamiento provisional y, a continuación, los convierte en discos administrados y los mueve a los grupos de recursos. Para más información, vea [Comprobación de la carga de datos en Azure](data-box-deploy-picked-up.md#verify-data-upload-to-azure).

    Seleccione **Siguiente: Detalles de contacto** para continuar.

8. En **Detalles de contacto**, seleccione **+ Agregar dirección de envío**.

    ![Solicitud de Data Box para disco administrado](media/data-box-deploy-ordered/select-data-box-import-08a.png)

9. En **Dirección de envío**, escriba su nombre y apellidos, el nombre y la dirección postal de la empresa y un número de teléfono válido. Seleccione **Validar la dirección**. El servicio valida la dirección de envío para conocer la disponibilidad del servicio. Si el servicio está disponible para la dirección de envío especificada, recibirá una notificación al respecto.

   ![Solicitud de Data Box para disco administrado](media/data-box-deploy-ordered/select-data-box-import-10.png)

   Si ha seleccionado el envío autoadministrado, recibirá una notificación por correo electrónico una vez que se haya realizado correctamente el pedido. Para obtener más información sobre el envío autoadministrado, consulte [Uso del envío autoadministrado](data-box-portal-customer-managed-shipping.md).

10. Seleccione **Agregar dirección de envío** una vez que se haya comprobado que los detalles sean correctos. Volverá a la pestaña **Detalles de contacto**.

11. De nuevo en **Detalles de contacto**, agregue una o varias direcciones de correo electrónico. El servicio envía notificaciones por correo electrónico si se produce cualquier actualización en el estado del pedido a las direcciones de correo electrónico especificadas.

    Es aconsejable usar un correo electrónico de grupo, con el fin de que siga recibiendo notificaciones aunque algún administrador deje el grupo.

    ![Solicitud de Data Box para disco administrado](media/data-box-deploy-ordered/select-data-box-import-08c.png)

12. Examine la información de **Revisar y pedir** del pedido, el contacto, la notificación y los términos de privacidad. Active la casilla correspondiente a contrato acuerdo con los términos de privacidad.

13. Seleccione **Pedido**. El pedido tarda unos minutos en crearse.

    ![Solicitud de Data Box para disco administrado](media/data-box-deploy-ordered/select-data-box-import-11.png)

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Realice los pasos siguientes con la CLI de Azure para solicitar un dispositivo:

1. Anote los valores de su pedido de Data Box. Estos valores incluyen la información personal o empresarial, el nombre de la suscripción, la información del dispositivo y la información de envío. Tendrá que usar estos valores como parámetros al ejecutar el comando de la CLI para crear el pedido de Data Box. En la tabla siguiente se muestra los valores de los parámetros utilizados para `az databox job create`:

   | Valor (parámetro) | Descripción |  Valor de ejemplo |
   |---|---|---|
   |resource-group| Uso uno existente o cree uno nuevo. Un grupo de recursos es un contenedor lógico para los recursos que se pueden administrar o implementar conjuntamente. | "myresourcegroup"|
   |name| Nombre del pedido que se va a crear. | "mydataboxorder"|
   |contact-name| Nombre asociado a la dirección de envío. | "Gus Poland"|
   |phone| Número de teléfono de la persona o empresa que recibirá el pedido.| "14255551234"
   |ubicación| La región de Azure más cercana que será la que envíe el dispositivo.| "US West"|
   |sku| El dispositivo Data Box específico que va a solicitar. Los valores válidos son: "DataBox", "DataBoxDisk" y "DataBoxHeavy"| "DataBox" |
   |email-list| Direcciones de correo electrónico asociadas al pedido.| "gusp@contoso.com" |
   |street-address1| Calle de la dirección postal a la que se enviará el pedido. | "15700 NE 39th St" |
   |street-address2| Información secundaria de la dirección postal, como el número de apartamento o el número de edificio. | "Bld 123" |
   |city| Ciudad a la que se enviará el dispositivo. | "Redmond" |
   |state-or-province| El estado o provincia al que se enviará el dispositivo.| "WA" |
   |country| País al que se enviará el dispositivo. | "Estados Unidos" |
   |postal-code| Código postal asociado a la dirección de envío.| "98052"|
   |company-name| El nombre de la empresa para la que trabaja.| "Contoso, LTD" |
   |Cuenta de almacenamiento| Cuenta de Azure Storage desde la que desea importar los datos.| "mystorageaccount"|
   |debug| Incluir información de depuración en el registro detallado.  | --debug |
   |help| Mostrar información de ayuda para este comando. | --help -h |
   |only-show-errors| Mostrar solo los errores y suprimir las advertencias. | --only-show-errors |
   |output -o| Establece el formato de salida.  Valores permitidos: json, jsonc, none, table, tsv, yaml, yamlc. El valor predeterminado es json. | --output "json" |
   |Query| Cadena de consulta de JMESPath. Para más información, consulte [JMESPath](http://jmespath.org/). | --query <string>|
   |verbose| Incluir registro detallado. | --verbose |

2. En el símbolo del sistema que haya elegido o en el terminal, use [az data box job create](https://docs.microsoft.com/cli/azure/ext/databox/databox/job?view=azure-cli-latest#ext-databox-az-databox-job-create) para crear el pedido de Azure Data Box.

   ```azurecli
   az databox job create --resource-group <resource-group> --name <order-name> --location <azure-location> --sku <databox-device-type> --contact-name <contact-name> --phone <phone-number> --email-list <email-list> --street-address1 <street-address-1> --street-address2 <street-address-2> --city "contact-city" --state-or-province <state-province> --country <country> --postal-code <postal-code> --company-name <company-name> --storage-account "storage-account"
   ```

   Este es un ejemplo de uso del comando:

   ```azurecli
   az databox job create --resource-group "myresourcegroup" \
                         --name "mydataboxtest3" \
                         --location "westus" \
                         --sku "DataBox" \
                         --contact-name "Gus Poland" \
                         --phone "14255551234" \
                         --email-list "gusp@contoso.com" \
                         --street-address1 "15700 NE 39th St" \
                         --street-address2 "Bld 25" \
                         --city "Redmond" \
                         --state-or-province "WA" \
                         --country "US" \
                         --postal-code "98052" \
                         --company-name "Contoso" \
                         --storage-account mystorageaccount
   ```

   Este es el resultado de la ejecución del comando:

   ```output
   Command group 'databox job' is experimental and not covered by customer support. Please use with discretion.
   {
     "cancellationReason": null,
     "deliveryInfo": {
        "scheduledDateTime": "0001-01-01T00:00:00+00:00"
   },
   "deliveryType": "NonScheduled",
   "details": null,
   "error": null,
   "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myresourcegroup/providers/Microsoft.DataBox/jobs/mydataboxtest3",
   "identity": {
     "type": "None"
   },
   "isCancellable": true,
   "isCancellableWithoutFee": true,
   "isDeletable": false,
   "isShippingAddressEditable": true,
   "location": "westus",
   "name": "mydataboxtest3",
   "resourceGroup": "myresourcegroup",
   "sku": {
     "displayName": null,
     "family": null,
     "name": "DataBox"
   },
   "startTime": "2020-06-10T23:28:27.354241+00:00",
   "status": "DeviceOrdered",
   "tags": {},
   "type": "Microsoft.DataBox/jobs"

   }
   PS C:\Windows>

   ```

3. Todos los comandos de la CLI de Azure usarán de forma predeterminada json como formato de salida, a menos que lo cambie. Puede cambiar el formato de salida mediante el parámetro global `--output <output-format>`. Cambiar el formato a "table" mejorará la legibilidad del resultado.

   Este es el mismo comando que acabamos de ejecutar con un pequeño ajuste para cambiar el formato:

    ```azurecli
    az databox job create --resource-group "myresourcegroup" --name "mydataboxtest4" --location "westus" --sku "DataBox" --contact-name "Gus Poland" --phone "14255551234" --email-list "gusp@contoso.com" --street-address1 "15700 NE 39th St" --street-address2 "Bld 25" --city "Redmond" --state-or-province "WA" --country "US" --postal-code "98052" --company-name "Contoso" --storage-account mystorageaccount --output "table"
   ```

   Este es el resultado de la ejecución del comando:

   ```output

    Command group 'databox job' is experimental and not covered by customer support. Please use with discretion.
    DeliveryType    IsCancellable    IsCancellableWithoutFee    IsDeletable    IsShippingAddressEditable    Location    Name            ResourceGroup    StartTime                         Status
    --------------  ---------------  -------------------------  -------------  ---------------------------  ----------  --------------  ---------------  --------------------------------  -------------
    NonScheduled    True             True                       False          True                         westus      mydataboxtest4  myresourcegroup  2020-06-18T03:48:00.905893+00:00  DeviceOrdered

    ```

# <a name="powershell"></a>[PowerShell](#tab/azure-ps)

Realice los pasos siguientes con Azure PowerShell para pedir un dispositivo:

1. Antes de crear el pedido de importación, debe obtener una cuenta de almacenamiento y guardar el objeto de esta en una variable.

   ```azurepowershell
    $storAcct = Get-AzStorageAccount -Name "mystorageaccount" -ResourceGroup "myresourcegroup"
   ```

2. Anote los valores de su pedido de Data Box. Estos valores incluyen la información personal o empresarial, el nombre de la suscripción, la información del dispositivo y la información de envío. Tendrá que usar estos valores como parámetros al ejecutar el comando de PowerShell para crear el pedido de Data Box. En la tabla siguiente se muestra los valores de los parámetros utilizados para [New-AzDataBoxJob](https://docs.microsoft.com/powershell/module/az.databox/New-AzDataBoxJob).

    | Valor (parámetro) | Descripción |  Valor de ejemplo |
    |---|---|---|
    |ResourceGroupName [obligatorio]| Use un grupo de recursos existente. Un grupo de recursos es un contenedor lógico para los recursos que se pueden administrar o implementar conjuntamente. | "myresourcegroup"|
    |Name [obligatorio]| Nombre del pedido que se va a crear. | "mydataboxorder"|
    |ContactName [obligatorio]| Nombre asociado a la dirección de envío. | "Gus Poland"|
    |PhoneNumber [obligatorio]| Número de teléfono de la persona o empresa que recibirá el pedido.| "14255551234"
    |Location [obligatorio]| La región de Azure más cercana que será la que envíe el dispositivo.| "WestUS"|
    |DataBoxType [obligatorio]| El dispositivo Data Box específico que va a solicitar. Los valores válidos son: "DataBox", "DataBoxDisk" y "DataBoxHeavy"| "DataBox" |
    |EmailId [obligatorio]| Direcciones de correo electrónico asociadas al pedido.| "gusp@contoso.com" |
    |StreetAddress1 [obligatorio]| Calle de la dirección postal a la que se enviará el pedido. | "15700 NE 39th St" |
    |StreetAddress2| Información secundaria de la dirección postal, como el número de apartamento o el número de edificio. | "Bld 123" |
    |StreetAddress3| Tercera línea de la dirección. | |
    |City [obligatorio]| Ciudad a la que se enviará el dispositivo. | "Redmond" |
    |StateOrProvinceCode [obligatorio]| El estado o provincia al que se enviará el dispositivo.| "WA" |
    |CountryCode [obligatorio]| País al que se enviará el dispositivo. | "Estados Unidos" |
    |PostalCode [obligatorio]| Código postal asociado a la dirección de envío.| "98052"|
    |CompanyName| El nombre de la empresa para la que trabaja.| "Contoso, LTD" |
    |StorageAccountResourceId [obligatorio]| Identificador de la cuenta de Azure Storage desde la que desea importar los datos.| <AzStorageAccount>.id |

3. En el símbolo del sistema que haya elegido o en el terminal, use [New-AzDataBoxJob](https://docs.microsoft.com/powershell/module/az.databox/New-AzDataBoxJob) para crear el pedido de Azure Data Box.

   ```azurepowershell
    PS> $storAcct = Get-AzureStorageAccount -StorageAccountName "mystorageaccount"
    PS> New-AzDataBoxJob -Location "WestUS" \
                         -StreetAddress1 "15700 NE 39th St" \
                         -PostalCode "98052" \
                         -City "Redmond" \
                         -StateOrProvinceCode "WA" \
                         -CountryCode "US" \
                         -EmailId "gusp@contoso.com" \
                         -PhoneNumber 4255551234 \
                         -ContactName "Gus Poland" \
                         -StorageAccount $storAcct.id \
                         -DataBoxType DataBox \
                         -ResourceGroupName "myresourcegroup" \
                         -Name "myDataBoxOrderPSTest"
   ```

   Este es el resultado de la ejecución del comando:

   ```output
    jobResource.Name     jobResource.Sku.Name jobResource.Status jobResource.StartTime jobResource.Location ResourceGroup
    ----------------     -------------------- ------------------ --------------------- -------------------- -------------
    myDataBoxOrderPSTest DataBox              DeviceOrdered      07-06-2020 05:25:30   westus               myresourcegroup
   ```

---

## <a name="track-the-order"></a>Seguimiento del pedido

# <a name="portal"></a>[Portal](#tab/portal)

Tras realizar el pedido, puede hacer un seguimiento del estado del mismo desde Azure Portal. Vaya al pedido de Data Box y, después, vaya a **Información general** para ver el estado. El portal muestra el pedido en estado **Ordered** (Pedido).

Si el dispositivo no está disponible, recibe una notificación. Si el dispositivo está disponible, Microsoft identifica que está listo para el envío y prepara dicho envío. Durante la preparación del dispositivo, se producen las siguientes acciones:

* Se crean recursos compartidos de SMB para cada cuenta de almacenamiento asociada con el dispositivo.
* Para cada cuenta, se generan las credenciales de acceso, como el nombre de usuario y la contraseña.
* También se genera la contraseña del dispositivo, que le ayuda a desbloquear el dispositivo.
* Data Box está bloqueado para evitar el acceso no autorizado al dispositivo en todo momento.

Una vez que se completa la preparación del dispositivo, el portal muestra el pedido en estado **Processed** (Procesado).

![Pedido de Data Box procesado](media/data-box-overview/data-box-order-status-processed.png)

Luego, Microsoft prepara y envía el disco a través de un operador regional. Una vez que se realiza el envío, recibe un número de seguimiento. El portal muestra el orden en estado **Dispatched** (Enviado).

![Pedido de Data Box enviado](media/data-box-overview/data-box-order-status-dispatched.png)

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

### <a name="track-a-single-order"></a>Seguimiento de un solo pedido

Para obtener información de seguimiento de un único pedido de Azure Data Box que ya se haya realizado, ejecute [az databox job show](https://docs.microsoft.com/cli/azure/ext/databox/databox/job?view=azure-cli-latest#ext-databox-az-databox-job-show). El comando muestra información sobre el pedido, entre otra: nombre, grupo de recursos, información de seguimiento, identificador de suscripción, información de contacto, tipo de envío y SKU del dispositivo.

   ```azurecli
   az databox job show --resource-group <resource-group> --name <order-name>
   ```

   En la tabla siguiente se muestra la información de los parámetros de `az databox job show`:

   | Parámetro | Descripción |  Valor de ejemplo |
   |---|---|---|
   |resource-group [obligatorio]| Nombre del grupo de recursos asociado al pedido. Un grupo de recursos es un contenedor lógico para los recursos que se pueden administrar o implementar conjuntamente. | "myresourcegroup"|
   |name [obligatorio]| Nombre del pedido que se va a mostrar. | "mydataboxorder"|
   |debug| Incluir información de depuración en el registro detallado. | --debug |
   |help| Mostrar información de ayuda para este comando. | --help -h |
   |only-show-errors| Mostrar solo los errores y suprimir las advertencias. | --only-show-errors |
   |output -o| Establece el formato de salida.  Valores permitidos: json, jsonc, none, table, tsv, yaml, yamlc. El valor predeterminado es json. | --output "json" |
   |Query| Cadena de consulta de JMESPath. Para más información, consulte [JMESPath](http://jmespath.org/). | --query <string>|
   |verbose| Incluir registro detallado. | --verbose |

   Este es un ejemplo del comando con el formato de salida establecido en "table":

   ```azurecli
    PS C:\WINDOWS\system32> az databox job show --resource-group "myresourcegroup" \
                                                --name "mydataboxtest4" \
                                                --output "table"
   ```

   Este es el resultado de la ejecución del comando:

   ```output
    Command group 'databox job' is experimental and not covered by customer support. Please use with discretion.
    DeliveryType    IsCancellable    IsCancellableWithoutFee    IsDeletable    IsShippingAddressEditable    Location    Name            ResourceGroup    StartTime                         Status
    --------------  ---------------  -------------------------  -------------  ---------------------------  ----------  --------------  ---------------  --------------------------------  -------------
    NonScheduled    True             True                       False          True                         westus      mydataboxtest4  myresourcegroup  2020-06-18T03:48:00.905893+00:00  DeviceOrdered
   ```

> [!NOTE]
> La lista de pedidos se puede realizar a nivel de suscripción y hace que el grupo de recursos sea un parámetro opcional (en lugar de obligatorio).

### <a name="list-all-orders"></a>Listado de todos los pedidos

Si ha solicitado varios dispositivos, puede ejecutar [az databox job list](https://docs.microsoft.com/cli/azure/ext/databox/databox/job?view=azure-cli-latest#ext-databox-az-databox-job-list) para ver todos los pedidos de Azure Data Box. El comando muestra en una lista todos los pedidos que pertenecen a un grupo de recursos específico. También se muestra en el resultado: nombre del pedido, estado de envío, región de Azure, tipo de entrega, estado del pedido. Los pedidos cancelados también se incluyen en la lista.
El comando también muestra las marcas de tiempo de cada pedido.

```azurecli
az databox job list --resource-group <resource-group>
```

En la tabla siguiente se muestra la información de parámetros de `az databox job list`:

   | Parámetro | Descripción |  Valor de ejemplo |
   |---|---|---|
   |resource-group [obligatorio]| Nombre del grupo de recursos que contiene los pedidos. Un grupo de recursos es un contenedor lógico para los recursos que se pueden administrar o implementar conjuntamente. | "myresourcegroup"|
   |debug| Incluir información de depuración en el registro detallado. | --debug |
   |help| Mostrar información de ayuda para este comando. | --help -h |
   |only-show-errors| Mostrar solo los errores y suprimir las advertencias. | --only-show-errors |
   |output -o| Establece el formato de salida.  Valores permitidos: json, jsonc, none, table, tsv, yaml, yamlc. El valor predeterminado es json. | --output "json" |
   |Query| Cadena de consulta de JMESPath. Para más información, consulte [JMESPath](http://jmespath.org/). | --query <string>|
   |verbose| Incluir registro detallado. | --verbose |

   Este es un ejemplo del comando con el formato de salida establecido en "table":

   ```azurecli
    PS C:\WINDOWS\system32> az databox job list --resource-group "GDPTest" --output "table"
   ```

   Este es el resultado de la ejecución del comando:

   ```output
   Command group 'databox job' is experimental and not covered by customer support. Please use with discretion.
   CancellationReason                                               DeliveryType    IsCancellable    IsCancellableWithoutFee    IsDeletable    IsShippingAddressEditable    Location    Name                 ResourceGroup    StartTime                         Status
   ---------------------- ----------------------------------------  --------------  ---------------  -------------------------  -------------  ---------------------------  ----------  -------------------  ---------------  --------------------------------  -------------
   OtherReason This was a test order for documentation purposes.    NonScheduled    False            False                      True           False                        westus      gdpImportTest        MyResGrp         2020-05-26T23:20:57.464075+00:00  Cancelled
   NoLongerNeeded This order was created for documentation purposes.NonScheduled    False            False                      True           False                        westus      mydataboxExportTest  MyResGrp         2020-05-27T00:04:16.640397+00:00  Cancelled
   IncorrectOrder                                                   NonScheduled    False            False                      True           False                        westus      mydataboxtest2       MyResGrp         2020-06-10T16:54:23.509181+00:00  Cancelled
                                                                    NonScheduled    True             True                       False          True                         westus      mydataboxtest3       MyResGrp         2020-06-11T22:05:49.436622+00:00  DeviceOrdered
                                                                    NonScheduled    True             True                       False          True                         westus      mydataboxtest4       MyResGrp         2020-06-18T03:48:00.905893+00:00  DeviceOrdered
   PS C:\WINDOWS\system32>
   ```

# <a name="powershell"></a>[PowerShell](#tab/azure-ps)

### <a name="track-a-single-order"></a>Seguimiento de un solo pedido

Para obtener información de seguimiento de un único pedido de Azure Data Box que ya se haya realizado, ejecute [Get-AzDataBoxJob](https://docs.microsoft.com/powershell/module/az.databox/Get-AzDataBoxJob). El comando muestra información sobre el pedido, entre otra: nombre, grupo de recursos, información de seguimiento, identificador de suscripción, información de contacto, tipo de envío y SKU del dispositivo.

> [!NOTE]
> `Get-AzDataBoxJob` se utiliza para mostrar los pedidos individuales y múltiples. La diferencia es que se especifica el nombre de pedido para los pedidos individuales.

   ```azurepowershell
    Get-AzDataBoxJob -ResourceGroupName <String> -Name <String>
   ```

   En la tabla siguiente se muestra la información de parámetros de `Get-AzDataBoxJob`:

   | Parámetro | Descripción |  Valor de ejemplo |
   |---|---|---|
   |ResourceGroup [obligatorio]| Nombre del grupo de recursos asociado al pedido. Un grupo de recursos es un contenedor lógico para los recursos que se pueden administrar o implementar conjuntamente. | "myresourcegroup"|
   |Name [obligatorio]| Nombre del pedido del que obtener información. | "mydataboxorder"|
   |ResourceId| Identificador del recurso asociado al pedido. |  |

   Este es un ejemplo del comando con el resultado:

   ```azurepowershell
    PS C:\WINDOWS\system32> Get-AzDataBoxJob -ResourceGroupName "myResourceGroup" -Name "myDataBoxOrderPSTest"
   ```

   Este es el resultado de la ejecución del comando:

   ```output
   jobResource.Name     jobResource.Sku.Name jobResource.Status jobResource.StartTime jobResource.Location ResourceGroup
   ----------------     -------------------- ------------------ --------------------- -------------------- -------------
   myDataBoxOrderPSTest DataBox              DeviceOrdered      7/7/2020 12:37:16 AM  WestUS               myResourceGroup
   ```

### <a name="list-all-orders"></a>Listado de todos los pedidos

Si ha solicitado varios dispositivos, puede ejecutar [Get-AzDataBoxJob](https://docs.microsoft.com/powershell/module/az.databox/Get-AzDataBoxJob) para ver todos los pedidos de Azure Data Box. El comando muestra en una lista todos los pedidos que pertenecen a un grupo de recursos específico. También se muestra en el resultado: nombre del pedido, estado de envío, región de Azure, tipo de entrega, estado del pedido. Los pedidos cancelados también se incluyen en la lista.
El comando también muestra las marcas de tiempo de cada pedido.

```azurepowershell
Get-AzDataBoxJob -ResourceGroupName <String>
```

Este es un ejemplo del comando:

```azurepowershell
PS C:\WINDOWS\system32> Get-AzDataBoxJob -ResourceGroupName "myResourceGroup"
```

Este es el resultado de la ejecución del comando:

```output
jobResource.Name     jobResource.Sku.Name jobResource.Status jobResource.StartTime jobResource.Location ResourceGroup
----------------     -------------------- ------------------ --------------------- -------------------- -------------
guspImportTest       DataBox              Cancelled          5/26/2020 11:20:57 PM WestUS               myResourceGroup
mydataboxExportTest  DataBox              Cancelled          5/27/2020 12:04:16 AM WestUS               myResourceGroup
mydataboximport1     DataBox              Cancelled          6/26/2020 11:00:34 PM WestUS               myResourceGroup
myDataBoxOrderPSTest DataBox              Cancelled          7/07/2020 12:37:16 AM WestUS               myResourceGroup
mydataboxtest2       DataBox              Cancelled          6/10/2020 4:54:23  PM WestUS               myResourceGroup
mydataboxtest4       DataBox              DeviceOrdered      6/18/2020 3:48:00  AM WestUS               myResourceGroup
PS C:\WINDOWS\system32>
```

---

## <a name="cancel-the-order"></a>Cancelar el pedido

# <a name="portal"></a>[Portal](#tab/portal)

Para cancelar el pedido, en Azure Portal, vaya a **Información general** y seleccione **Cancelar** en la barra de comandos.

Después de realizar un pedido, puede cancelarlo en cualquier momento, siempre que no se encuentre en estado procesado.

Para eliminar un pedido cancelado, vaya a **Información general** y seleccione **Eliminar** en la barra de comandos.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

### <a name="cancel-an-order"></a>Cancelación de un pedido

Para cancelar un pedido de Azure Data Box, ejecute [az databox job cancel](https://docs.microsoft.com/cli/azure/ext/databox/databox/job?view=azure-cli-latest#ext-databox-az-databox-job-cancel). Se le pedirá que especifique el motivo para cancelar el pedido.

   ```azurecli
   az databox job cancel --resource-group <resource-group> --name <order-name> --reason <cancel-description>
   ```

   En la tabla siguiente se muestra la información de parámetros de `az databox job cancel`:

   | Parámetro | Descripción |  Valor de ejemplo |
   |---|---|---|
   |resource-group [obligatorio]| Nombre del grupo de recursos asociado al pedido que se va a eliminar. Un grupo de recursos es un contenedor lógico para los recursos que se pueden administrar o implementar conjuntamente. | "myresourcegroup"|
   |name [obligatorio]| Nombre del pedido que se va a eliminar. | "mydataboxorder"|
   |reason [obligatorio]| Motivo para cancelar el pedido. | "I entered erroneous information and needed to cancel the order." |
   |sí| No solicita confirmación. | --yes (-y)| --yes -y |
   |debug| Incluir información de depuración en el registro detallado. | --debug |
   |help| Mostrar información de ayuda para este comando. | --help -h |
   |only-show-errors| Mostrar solo los errores y suprimir las advertencias. | --only-show-errors |
   |output -o| Establece el formato de salida.  Valores permitidos: json, jsonc, none, table, tsv, yaml, yamlc. El valor predeterminado es json. | --output "json" |
   |Query| Cadena de consulta de JMESPath. Para más información, consulte [JMESPath](http://jmespath.org/). | --query <string>|
   |verbose| Incluir registro detallado. | --verbose |

   Este es un ejemplo del comando con el resultado:

   ```azurecli
   PS C:\Windows> az databox job cancel --resource-group "myresourcegroup" --name "mydataboxtest3" --reason "Our budget was slashed due to **redacted** and we can no longer afford this device."
   ```

   Este es el resultado de la ejecución del comando:

   ```output
   Command group 'databox job' is experimental and not covered by customer support. Please use with discretion.
   Are you sure you want to perform this operation? (y/n): y
   PS C:\Windows>
   ```

### <a name="delete-an-order"></a>Eliminar un pedido

Si ha cancelado un pedido de Azure Data Box, puede ejecutar [az databox job delete](https://docs.microsoft.com/cli/azure/ext/databox/databox/job?view=azure-cli-latest#ext-databox-az-databox-job-delete) para eliminarlo.

   ```azurecli
   az databox job delete --name [-n] <order-name> --resource-group <resource-group> [--yes] [--verbose]
   ```

   En la tabla siguiente se muestra la información de parámetros de `az databox job delete`:

   | Parámetro | Descripción |  Valor de ejemplo |
   |---|---|---|
   |resource-group [obligatorio]| Nombre del grupo de recursos asociado al pedido que se va a eliminar. Un grupo de recursos es un contenedor lógico para los recursos que se pueden administrar o implementar conjuntamente. | "myresourcegroup"|
   |name [obligatorio]| Nombre del pedido que se va a eliminar. | "mydataboxorder"|
   |subscription| Nombre o identificador (GUID) de la suscripción de Azure. | "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" |
   |sí| No solicita confirmación. | --yes (-y)| --yes -y |
   |debug| Incluir información de depuración en el registro detallado. | --debug |
   |help| Mostrar información de ayuda para este comando. | --help -h |
   |only-show-errors| Mostrar solo los errores y suprimir las advertencias. | --only-show-errors |
   |output -o| Establece el formato de salida.  Valores permitidos: json, jsonc, none, table, tsv, yaml, yamlc. El valor predeterminado es json. | --output "json" |
   |Query| Cadena de consulta de JMESPath. Para más información, consulte [JMESPath](http://jmespath.org/). | --query <string>|
   |verbose| Incluir registro detallado. | --verbose |

Este es un ejemplo del comando con el resultado:

   ```azurecli
   PS C:\Windows> az databox job delete --resource-group "myresourcegroup" --name "mydataboxtest3" --yes --verbose
   ```

   Este es el resultado de la ejecución del comando:

   ```output
   Command group 'databox job' is experimental and not covered by customer support. Please use with discretion.
   command ran in 1.142 seconds.
   PS C:\Windows>
   ```

# <a name="powershell"></a>[PowerShell](#tab/azure-ps)

### <a name="cancel-an-order"></a>Cancelación de un pedido

Para cancelar un pedido de Azure Data Box, ejecute [Stop-AzDataBoxJob](https://docs.microsoft.com/powershell/module/az.databox/stop-azdataboxjob). Se le pedirá que especifique el motivo para cancelar el pedido.

```azurepowershell
Stop-AzDataBoxJob -ResourceGroup <String> -Name <String> -Reason <String>
```

En la tabla siguiente se muestra la información de parámetros de `Stop-AzDataBoxJob`:

| Parámetro | Descripción |  Valor de ejemplo |
|---|---|---|
|ResourceGroup [obligatorio]| Nombre del grupo de recursos asociado al pedido que se va a cancelar. Un grupo de recursos es un contenedor lógico para los recursos que se pueden administrar o implementar conjuntamente. | "myresourcegroup"|
|Name [obligatorio]| Nombre del pedido que se va a eliminar. | "mydataboxorder"|
|Reason [obligatorio]| Motivo para cancelar el pedido. | "I entered erroneous information and needed to cancel the order." |
|Force | Hace que el cmdlet se ejecute sin confirmación del usuario. | -Force |

Este es un ejemplo del comando con el resultado:

```azurepowershell
PS C:\PowerShell\Modules> Stop-AzDataBoxJob -ResourceGroupName myResourceGroup \
                                            -Name "myDataBoxOrderPSTest" \
                                            -Reason "I entered erroneous information and had to cancel."
```

Este es el resultado de la ejecución del comando:

```output
Confirm
"Cancelling Databox Job "myDataBoxOrderPSTest
[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): y
PS C:\WINDOWS\system32>
```

### <a name="delete-an-order"></a>Eliminar un pedido

Si ha cancelado un pedido de Azure Data Box, puede ejecutar [Remove-AzDataBoxJob](https://docs.microsoft.com/powershell/module/az.databox/remove-azdataboxjob) para eliminarlo.

```azurepowershell
Remove-AzDataBoxJob -Name <String> -ResourceGroup <String>
```

En la tabla siguiente se muestra la información de parámetros de `Remove-AzDataBoxJob`:

| Parámetro | Descripción |  Valor de ejemplo |
|---|---|---|
|ResourceGroup [obligatorio]| Nombre del grupo de recursos asociado al pedido que se va a eliminar. Un grupo de recursos es un contenedor lógico para los recursos que se pueden administrar o implementar conjuntamente. | "myresourcegroup"|
|Name [obligatorio]| Nombre del pedido que se va a eliminar. | "mydataboxorder"|
|Force | Hace que el cmdlet se ejecute sin confirmación del usuario. | -Force |

Este es un ejemplo del comando con el resultado:

```azurepowershell
PS C:\Windows> Remove-AzDataBoxJob -ResourceGroup "myresourcegroup" \
                                   -Name "mydataboxtest3"
```

Este es el resultado de la ejecución del comando:

```output
Confirm
"Removing Databox Job "mydataboxtest3
[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): y
PS C:\Windows>
```

---

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha obtenido información acerca de varios temas relacionados con Azure Data Box, como:

> [!div class="checklist"]
>
> * Requisitos previos para implementar Data Box
> * Realización de un pedido de Data Box
> * Seguimiento del pedido
> * Cancelar el pedido

En el siguiente tutorial aprenderá a configurar Data Box.

> [!div class="nextstepaction"]
> [Configuración de Azure Data Box](./data-box-deploy-set-up.md)
