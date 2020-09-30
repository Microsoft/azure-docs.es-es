---
title: Puntos de conexión privados
description: Comprenda el proceso de creación de puntos de conexión privados para Azure Backup y los escenarios en los que el uso de puntos de conexión privados ayuda a preservar la seguridad de los recursos.
ms.topic: conceptual
ms.date: 05/07/2020
ms.openlocfilehash: 0a875dfedbf7a3b76b479fd4f23b74a7ced47252
ms.sourcegitcommit: 3fb5e772f8f4068cc6d91d9cde253065a7f265d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/31/2020
ms.locfileid: "89179239"
---
# <a name="private-endpoints-for-azure-backup"></a>Puntos de conexión privados para Azure Backup

Azure Backup permite realizar copias de seguridad de los datos y restaurarlos de forma segura desde los almacenes de Recovery Services mediante [puntos de conexión privados](../private-link/private-endpoint-overview.md). Los puntos de conexión privados usan una o más direcciones IP privadas de la red virtual para incorporar el servicio de manera eficaz a su red virtual.

Este artículo le ayudará a comprender el proceso de creación de puntos de conexión privados para Azure Backup y los escenarios en los que el uso de puntos de conexión privados ayuda a preservar la seguridad de los recursos.

## <a name="before-you-start"></a>Antes de comenzar

- Los puntos de conexión privados solo se pueden crear para almacenes de Recovery Services nuevos (que no tienen elementos registrados). Por lo tanto, los puntos de conexión privados deben crearse antes de intentar proteger los elementos en el almacén.
- Una red virtual puede contener puntos de conexión privados para varios almacenes de Recovery Services. Además, un almacén de Recovery Services puede tener puntos de conexión privados en varias redes virtuales. Sin embargo, el número máximo de puntos de conexión privados que se pueden crear para un almacén es 12.
- Una vez que se crea un punto de conexión privado para un almacén, el almacén se bloqueará. No será accesible (para copias de seguridad y restauraciones) desde redes que no contengan un punto de conexión privado para el almacén. Si se quitan todos los puntos de conexión privados para el almacén, se podrá acceder al almacén desde todas las redes.
- Una conexión de punto de conexión privado para la copia de seguridad usa un total de 11 direcciones IP privadas de la subred, incluidas las que usa Azure Backup para el almacenamiento. Este número puede ser superior (hasta veinticinco) para ciertas regiones de Azure. Por lo tanto, se recomienda disponer de suficientes direcciones IP privadas cuando intente crear puntos de conexión privados para la copia de seguridad.
- Aunque tanto Azure Backup como Azure Site Recovery usan un almacén de Recovery Services, en este artículo solo se describe el uso de puntos de conexión privados para Azure Backup.
- Actualmente, Azure Active Directory no admite puntos de conexión privados. Por tanto, las direcciones IP y los FQDN necesarios para que Azure Active Directory funcione en una región necesitarán que se les permita el acceso de salida desde la red protegida al realizar copias de seguridad de bases de datos en máquinas virtuales de Azure y copias de seguridad mediante el agente de MARS. También puede usar etiquetas NSG y etiquetas de Azure Firewall para permitir el acceso a Azure AD, según corresponda.
- No se admiten redes virtuales con directivas de red para los puntos de conexión privados. Deberá deshabilitar las directivas de red antes de continuar.
- Debe volver a registrar el proveedor de recursos de Recovery Services con la suscripción si lo registró antes del 1 de mayo de 2020. Para volver a registrar el proveedor, vaya a la suscripción en Azure Portal, haga clic en el **Proveedor de recursos** en la barra de navegación izquierda, seleccione **Microsoft.RecoveryServices** y **Volver a registrar**.

## <a name="recommended-and-supported-scenarios"></a>Escenarios recomendados y admitidos

Aunque los puntos de conexión privados están habilitados para el almacén, solo se usan para la copia de seguridad y restauración de las cargas de trabajo de SQL y SAP HANA en una máquina virtual de Azure y en las copias de seguridad del agente de MARS. También puede usar el almacén para realizar copias de seguridad de otras cargas de trabajo (si bien no requerirán los puntos de conexión privados). Además de la copia de seguridad de las cargas de trabajo de SQL y SAP HANA y de las copias de seguridad mediante el agente de MARS, los puntos de conexión privados también se usan para realizar la recuperación de archivos para la copia de seguridad de máquinas virtuales de Azure. Para más información, vea la tabla siguiente:

| Copia de seguridad de cargas de trabajo en máquinas virtuales de Azure (SQL, SAP HANA), copia de seguridad mediante el agente de MARS | Se recomienda el uso de puntos de conexión privados para permitir copias de seguridad y restauración sin necesidad de incluir en una lista las direcciones IP o FQDN permitidas para Azure Backup o Azure Storage de las redes virtuales. |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Copia de seguridad de la máquina virtual de Azure**                                         | La copia de seguridad de la máquina virtual no requiere que permita el acceso a direcciones IP o FQDN. Por lo tanto, no requiere puntos de conexión privados para la copia de seguridad y restauración de discos.  <br><br>   Sin embargo, la recuperación de archivos desde un almacén que contiene puntos de conexión privados se restringe a redes virtuales que contienen un punto de conexión privado para el almacén. <br><br>    Al usar discos no administrados de la ACL, asegúrese de que la cuenta de almacenamiento que contiene los discos permite el acceso a **servicios de Microsoft de confianza** si forma parte de la ACL. |
| **Copia de seguridad de Azure Files**                                      | Las copias de seguridad de Azure Files se almacenan en la cuenta de almacenamiento local. Por lo tanto, no requiere puntos de conexión privados para la copia de seguridad y restauración. |

## <a name="creating-and-using-private-endpoints-for-backup"></a>Creación y uso de puntos de conexión privados para Azure Backup

En esta sección se tratan los pasos necesarios para crear y usar puntos de conexión privados para Azure Backup dentro de las redes virtuales.

>[!IMPORTANT]
> Se recomienda encarecidamente que siga los pasos en la misma secuencia que se mencionan en este documento. Si no lo hace, puede que el almacén sea incompatible con el uso de puntos de conexión privados y que tenga que reiniciar el proceso con un nuevo almacén.

[!INCLUDE [How to create a Recovery Services vault](../../includes/backup-create-rs-vault.md)]

Consulte [esta sección](#create-a-recovery-services-vault-using-the-azure-resource-manager-client) para aprender a crear un almacén mediante el cliente de Azure Resource Manager. Esto crea un almacén con su identidad administrada ya habilitada. Obtenga más información sobre los almacenes de Recovery Services [aquí](./backup-azure-recovery-services-vault-overview.md).

## <a name="enable-managed-identity-for-your-vault"></a>Habilitar la identidad administrada del almacén

Las identidades administradas permiten al almacén crear y usar puntos de conexión privados. En esta sección se habla sobre cómo habilitar la identidad administrada del almacén.

1. Vaya al almacén de Recovery Services -> **Identidad**.

    ![Cambiar el estado de la identidad a activada](./media/private-endpoints/identity-status-on.png)

1. Cambie el **Estado** a **Activado** y seleccione **Guardar**.

1. Se genera una **Id. de objeto**, que es la identidad administrada del almacén.

    >[!NOTE]
    >Una vez habilitada, la identidad administrada **NO** debe deshabilitarse (ni siquiera temporalmente). Deshabilitar la identidad administrada puede provocar un comportamiento incoherente.

## <a name="dns-changes"></a>Cambios de DNS

El uso de puntos de conexión privados requiere zonas DNS privadas para permitir que la extensión de Azure Backup resuelva los FQDN de Private Link en direcciones IP privadas. En total, se necesitan tres zonas DNS privadas. Aunque dos de estas zonas se deben crear obligatoriamente, la tercera puede integrarse con el punto de conexión privado (al crear el punto de conexión privado) o se puede crear por separado.

También puede usar los servidores DNS personalizados. Consulte [Cambios de DNS para los servidores DNS personalizados](#dns-changes-for-custom-dns-servers) para obtener más información sobre el uso de servidores DNS personalizados.

### <a name="creating-mandatory-dns-zones"></a>Crear zonas DNS obligatorias

Hay dos zonas DNS obligatorias que deben crearse:

- `privatelink.blob.core.windows.net` (para copias de seguridad y restauración de datos)
- `privatelink.queue.core.windows.net` (para la comunicación del servicio)

1. Busque **Zona DNS privada** en la barra de búsqueda de **Todos los servicios** y seleccione **Zona DNS privada** en la lista desplegable.

    ![Seleccionar una zona DNS privada](./media/private-endpoints/private-dns-zone.png)

1. Una vez que se encuentre en el panel de **Zona DNS privada**, seleccione el botón **+Agregar** para empezar a crear una nueva zona.

1. En el panel **Crear zona DNS privada**, ingrese los detalles necesarios. La suscripción debe ser la misma que la ubicación donde se creará el punto de conexión privado.

    La nomenclatura de las zonas es la siguiente:

    - `privatelink.blob.core.windows.net`
    - `privatelink.queue.core.windows.net`

    | **Zona**                           | **Servicio** | **Detalles de suscripción y grupo de recursos (RG)**                  |
    | ---------------------------------- | ----------- | ------------------------------------------------------------ |
    | `privatelink.blob.core.windows.net`  | Blob        | **Suscripción**: Igual que el punto de conexión privado que se debe crear  **RG**: El RG de la red virtual o el del punto de conexión privado |
    | `privatelink.queue.core.windows.net` | Cola       | **RG**: El RG de la red virtual o el del punto de conexión privado |

    ![Creación de una zona DNS privada](./media/private-endpoints/create-private-dns-zone.png)

1. Una vez hecho esto, prosiga con la revisión y creación de la zona DNS.

### <a name="optional-dns-zone"></a>Zona DNS opcional

Puede optar por integrar los puntos de conexión privados con zonas DNS privadas para Azure Backup (que se describen en la sección [Creación y uso de puntos de conexión privados](#creating-and-using-private-endpoints-for-backup)) para la comunicación del servicio. Si no desea integrar la zona DNS privada, puede optar por usar su propio servidor DNS o crear una zona DNS privada por separado. Esto se suma a las dos zonas DNS privadas obligatorias que se describen en la sección anterior.

Si desea crear una zona DNS privada independiente en Azure, puede hacer lo mismo siguiendo los mismos pasos que se usan para crear zonas DNS obligatorias. Los detalles de nomenclatura y suscripción se comparten a continuación:

| **Zona**                                                     | **Servicio** | **Detalles de suscripción y grupo de recursos**                  |
| ------------------------------------------------------------ | ----------- | ------------------------------------------------------------ |
| `privatelink.<geo>.backup.windowsazure.com`  <br><br>   **Nota**: aquí *geo* hace referencia al código de región. Por ejemplo, *wcus* y *ne* para el Centro-oeste de EE. UU. y el Norte de Europa respectivamente. | Copia de seguridad      | **Suscripción**: Igual que el punto de conexión privado que se debe crear  **RG**: Cualquier RG dentro de la suscripción |

Consulte [esta lista](https://download.microsoft.com/download/1/2/6/126a410b-0e06-45ed-b2df-84f353034fa1/AzureRegionCodesList.docx) para ver los códigos de región.

Para las convenciones de nomenclatura de direcciones URL en regiones nacionales:

- [China](/azure/china/resources-developer-guide#check-endpoints-in-azure)
- [Alemania](../germany/germany-developer-guide.md#endpoint-mapping)
- [US Gov](../azure-government/documentation-government-developer-guide.md)

### <a name="linking-private-dns-zones-with-your-virtual-network"></a>Vinculación de zonas DNS privadas con la red virtual

Ahora, las zonas DNS creadas anteriormente deben vincularse a la red virtual en la que se encuentran los servidores de los que se va a realizar la copia de seguridad. Esto debe hacerse para todas las zonas DNS que creó.

1. Vaya a la zona DNS (que creó en el paso anterior) y diríjase a **Vínculos de red virtual** en la barra de la izquierda. Una vez allí, seleccione el botón **+Agregar**.
1. Ingrese todos los detalles obligatorios. Los campos **Suscripción** y **Red virtual** deben rellenarse con los detalles correspondientes de la red virtual en la que se encuentran los servidores. Los demás campos deben dejarse tal cual.

    ![Agregar el vínculo de red virtual](./media/private-endpoints/add-virtual-network-link.png)

## <a name="grant-permissions-to-the-vault-to-create-required-private-endpoints"></a>Concesión de permisos al almacén para crear los puntos de conexión privados necesarios

Para crear los puntos de conexión privados necesarios para Azure Backup, el almacén (la identidad administrada del almacén) debe tener permisos para los siguientes grupos de recursos:

- El grupo de recursos que contiene la red virtual de destino
- El grupo de recursos donde se crearán los puntos de conexión privados
- El grupo de recursos que contiene las zonas DNS privadas

Se recomienda conceder el rol **Colaborador** para estos tres grupos de recursos en el almacén (identidad administrada). En los pasos siguientes se describe cómo hacer esto para un determinado grupo de recursos (esto se debe hacer para cada uno de los tres grupos de recursos):

1. Vaya al grupo de recursos y navegue a **Control de acceso (IAM)** en la barra de la izquierda.
1. Una vez en **Control de acceso**, vaya a **Agregar una asignación de roles**.

    ![Adición de una asignación de roles](./media/private-endpoints/add-role-assignment.png)

1. En el panel de **Agregar asignación de roles**, seleccione **Colaborador** como **Rol**y use el **Nombre** del almacén como **Entidad de seguridad**. Seleccione el almacén y, cuando haya terminado, seleccione **Guardar**.

    ![Elegir rol y entidad de seguridad](./media/private-endpoints/choose-role-and-principal.png)

Para administrar los permisos de forma más detallada, consulte [Crear roles y permisos manualmente](#create-roles-and-permissions-manually).

## <a name="creating-and-approving-private-endpoints-for-azure-backup"></a>Creación y aprobación de puntos de conexión privados para Azure Backup

### <a name="creating-private-endpoints-for-backup"></a>Creación de puntos de conexión privados para Azure Backup

En esta sección se describe el proceso de creación de un punto de conexión privado para el almacén.

1. En la barra de búsqueda, busque y seleccione **Private Link**. Esto le llevará al **Centro de Private Link**.

    ![Buscar Private Link](./media/private-endpoints/search-for-private-link.png)

1. En la barra de navegación de la izquierda, seleccione **Puntos de conexión privados**. Una vez en el panel de **Puntos de conexión privados**, seleccione **+Agregar** para empezar a crear un punto de conexión privado para el almacén.

    ![Agregar un punto de conexión privado en el Centro de Private Link](./media/private-endpoints/add-private-endpoint.png)

1. Una vez que empiece el proceso para **Crear un punto de conexión privado**, se le pedirá que especifique los detalles para crear la conexión de punto de conexión privado.

    1. **Aspectos básicos**: Rellene los detalles básicos de los puntos de conexión privados. La región debe ser la misma que la del almacén y el recurso.

        ![Rellene los detalles básicos](./media/private-endpoints/basic-details.png)

    1. **Recursos**: Esta pestaña requiere que mencione el recurso de PaaS para el que desea crear la conexión. Seleccione **Microsoft.RecoveryServices/vaults** del tipo de recurso para la suscripción deseada. Una vez hecho esto, elija el nombre del almacén de Recovery Services como **Recurso** y **AzureBackup** como el **Subrecurso de destino**.

        ![Rellene la pestaña Recursos](./media/private-endpoints/resource-tab.png)

    1. **Configuración**: En Configuración, especifique la red virtual y la subred en la que desea que se cree el punto de conexión privado. Esta será la red virtual donde está presente la máquina virtual. Puede optar por **integrar el punto de conexión privado** con una zona DNS privada. Como alternativa, también puede usar un servidor DNS personalizado o crear una zona DNS privada.

        ![Rellene la pestaña Configuración](./media/private-endpoints/configuration-tab.png)

    1. Opcionalmente, puede agregar **Etiquetas** para el punto de conexión privado.

    1. Continúe para **Revisar y crear** una vez que haya terminado de escribir los detalles. Una vez finalizada la validación, seleccione **Crear** para crear el punto de conexión privado.

## <a name="approving-private-endpoints"></a>Aprobar puntos de conexión privados

Si el usuario que crea el punto de conexión privado también es el propietario del almacén de Recovery Services, el punto de conexión privado creado anteriormente se aprobará automáticamente. De lo contrario, el propietario del almacén debe aprobar el punto de conexión privado antes de poder usarlo. En esta sección se describe la aprobación manual de los puntos de conexión privados a través de Azure Portal.

Consulte la sección [Aprobación manual de los puntos de conexión privados mediante el cliente de Azure Resource Manager](#manual-approval-of-private-endpoints-using-the-azure-resource-manager-client) para usar el cliente de Azure Resource Manager para la aprobación de puntos de conexión privados.

1. En el almacén de Recovery Services, vaya a **Conexiones de punto de conexión privado** en la barra de la izquierda.
1. Seleccione la conexión de punto de conexión privado que desea aprobar.
1. Seleccione **Aprobar** en la barra superior. También puede seleccionar **Rechazar** o **Quitar** si desea rechazar o eliminar la conexión del punto de conexión.

    ![Aprobación de puntos de conexión privados](./media/private-endpoints/approve-private-endpoints.png)

## <a name="adding-dns-records"></a>Agregar registros DNS

>[!NOTE]
> Este paso no es necesario si usa una zona DNS integrada. Sin embargo, si ha creado su propia zona DNS privada de Azure o utiliza una zona DNS privada personalizada, asegúrese de que las entradas se realicen tal y como se describe en esta sección.

Una vez que haya creado la zona DNS privada opcional y los puntos de conexión privados para el almacén, deberá agregar registros DNS a la zona DNS. Puede hacerlo manualmente o mediante un script de PowerShell. Esto solo se debe hacer para la zona DNS de Azure Backup, las de blobs y colas se actualizarán automáticamente.

### <a name="add-records-manually"></a>Agregar registros manualmente

Esto requiere que se creen entradas para cada FQDN del punto de conexión privado en la zona DNS privada.

1. Vaya a la **Zona DNS privada** y navegue hasta la opción **Información general** en la barra de la izquierda. Una vez allí, seleccione **+Conjunto de registros** para empezar a agregar registros.

    ![Seleccionar +Conjunto de registros para agregar registros](./media/private-endpoints/select-record-set.png)

1. En el panel de **Agregar conjunto de registros** que se abre, agregue una entrada para cada FQDN y la dirección IP privada como un registro de **Tipo A**. La lista de FQDN y direcciones IP se puede obtener del punto de conexión privado (en **Información general**). Como se muestra en el ejemplo siguiente, el primer FQDN del punto de conexión privado se agrega al conjunto de registros de la zona DNS privada.

    ![Lista de FQDN y direcciones IP](./media/private-endpoints/list-of-fqdn-and-ip.png)

    ![Agregar conjunto de registros](./media/private-endpoints/add-record-set.png)

### <a name="add-records-using-powershell-script"></a>Agregar registros mediante un script de PowerShell

1. Inicie **Cloud Shell** en Azure Portal y seleccione **Cargar archivo** en la ventana de PowerShell.

    ![Seleccionar Cargar archivo en la ventana de PowerShell](./media/private-endpoints/upload-file-in-powershell.png)

1. Cargue este script: [DnsZoneCreation](https://download.microsoft.com/download/1/2/6/126a410b-0e06-45ed-b2df-84f353034fa1/dnszonerecordcreation.ps1)

1. Vaya a la carpeta principal (por ejemplo: `cd /home/user`)

1. Ejecute el siguiente script:

    ```azurepowershell
    ./dnszonerecordcreation.ps1 -Subscription <SubscriptionId> -VaultPEName <VaultPE Name> -VaultPEResourceGroup <Vault PE RG> -DNSResourceGroup <Private DNS RG> -Privatezone <privatednszone>
    ```

    Estos son los parámetros:

    - **Suscripción**: La suscripción en la que residen los recursos (punto de conexión privado y zona DNS privada del almacén)
    - **vaultPEName**: Nombre del punto de conexión privado creado para el almacén
    - **vaultPEResourceGroup**: El grupo de recursos que contiene el punto de conexión privado del almacén
    - **dnsResourceGroup**: El grupo de recursos que contiene las zonas DNS privadas
    - **Privatezone**: Nombre de la zona DNS privada

## <a name="using-private-endpoints-for-backup"></a>Uso de puntos de conexión privados para Azure Backup

Una vez que los puntos de conexión privados creados para el almacén de la red virtual se han aprobado, puede empezar a usarlos para realizar las copias de seguridad y restauraciones.

>[!IMPORTANT]
>Asegúrese de haber completado correctamente todos los pasos mencionados previamente en el documento antes de continuar. En resumen, debe haber completado los pasos de la siguiente lista:
>
>1. Se creó un (nuevo) almacén de Recovery Services
>1. Se habilitó el almacén para usar la identidad administrada asignada por el sistema
>1. Se crearon tres zonas DNS privadas (dos si se usa una zona DNS integrada para Azure Backup)
>1. Se vincularon las zonas DNS privadas a su red virtual de Azure
>1. Se asignaron los permisos relevantes a la identidad administrada del almacén
>1. Se creó un punto de conexión privado para el almacén
>1. Se aprobó el punto de conexión privado (si no se aprobó automáticamente)
>1. Se agregaron los registros DNS necesarios a la zona DNS privada para Azure Backup (solo es aplicable si no se usa una zona DNS privada integrada)

### <a name="backup-and-restore-of-workloads-in-azure-vm-sql-sap-hana"></a>Copia de seguridad y restauración de cargas de trabajo en máquinas virtuales de Azure (SQL, SAP HANA)

Una vez que se crea y aprueba el punto de conexión privado, no se requiere ningún cambio adicional del lado del cliente para usar el punto de conexión privado. Toda la comunicación y la transferencia de datos desde la red segura al almacén se realizará a través del punto de conexión privado.
Sin embargo, si quita los puntos de conexión privados del almacén después de haber registrado un servidor (SQL/SAP HANA), deberá volver a registrar el contenedor con el almacén. No es necesario detener la protección de los mismos.

### <a name="backup-and-restore-through-mars-agent"></a>Copia de seguridad y restauración mediante el agente de MARS

Al usar el agente de MARS para realizar una copia de seguridad de los recursos locales, asegúrese de que la red local (que contiene los recursos de los que se va a realizar la copia de seguridad) está emparejada con la red virtual de Azure que contiene un punto de conexión privado para el almacén, para que pueda usarla. Después, puede continuar con la instalación del agente de MARS y configurar la copia de seguridad como se detalla aquí. Sin embargo, debe asegurarse de que toda la comunicación para la copia de seguridad se produzca solo a través de la red emparejada.

Sin embargo, si quita los puntos de conexión privados del almacén después de haber registrado un agente de MARS, deberá volver a registrar el contenedor con el almacén. No es necesario detener la protección de los mismos.

## <a name="additional-topics"></a>Otros temas

### <a name="create-a-recovery-services-vault-using-the-azure-resource-manager-client"></a>Creación de un almacén de Recovery Services mediante el cliente de Azure Resource Manager

Puede crear el almacén de Recovery Services y habilitar su identidad administrada (se requiere la habilitación de la identidad administrada, como veremos más adelante) mediante el cliente de Azure Resource Manager. A continuación, se muestra un ejemplo de cómo hacerlo:

```rest
armclient PUT /subscriptions/<subscriptionid>/resourceGroups/<rgname>/providers/Microsoft.RecoveryServices/Vaults/<vaultname>?api-version=2017-07-01-preview @C:\<filepath>\MSIVault.json
```

El archivo JSON anterior debe tener el siguiente contenido:

Solicitud JSON:

```json
{
  "location": "eastus2",
  "name": "<vaultname>",
  "etag": "W/\"datetime'2019-05-24T12%3A54%3A42.1757237Z'\"",
  "tags": {
    "PutKey": "PutValue"
  },
  "properties": {},
  "id": "/subscriptions/<subscriptionid>/resourceGroups/<rgname>/providers/Microsoft.RecoveryServices/Vaults/<vaultname>",
  "type": "Microsoft.RecoveryServices/Vaults",
  "sku": {
    "name": "RS0",
    "tier": "Standard"
  },
  "identity": {
    "type": "systemassigned"
  }
}
```

Respuesta JSON:

```json
{
   "location": "eastus2",
   "name": "<vaultname>",
   "etag": "W/\"datetime'2020-02-25T05%3A26%3A58.5181122Z'\"",
   "tags": {
     "PutKey": "PutValue"
   },
   "identity": {
     "tenantId": "<tenantid>",
     "principalId": "<principalid>",
     "type": "SystemAssigned"
   },
   "properties": {
     "provisioningState": "Succeeded",
     "privateEndpointStateForBackup": "None",
     "privateEndpointStateForSiteRecovery": "None"
   },
   "id": "/subscriptions/<subscriptionid>/resourceGroups/<rgname>/providers/Microsoft.RecoveryServices/Vaults/<vaultname>",
   "type": "Microsoft.RecoveryServices/Vaults",
   "sku": {
     "name": "RS0",
     "tier": "Standard"
   }
 }
```

>[!NOTE]
>El almacén creado en este ejemplo mediante Azure Resource Manager ya se ha creado con una identidad administrada asignada por el sistema.

### <a name="managing-permissions-on-resource-groups"></a>Administración de permisos en grupos de recursos

La identidad administrada del almacén debe tener los permisos siguientes en el grupo de recursos y la red virtual donde se crearán los puntos de conexión privados:

- `Microsoft.Network/privateEndpoints/*` Esto es necesario para realizar operaciones CRUD en puntos de conexión privados en el grupo de recursos. Debe asignarse en el grupo de recursos.
- `Microsoft.Network/virtualNetworks/subnets/join/action` Esto es necesario en la red virtual en la que la dirección IP privada se conecta con el punto de conexión privado.
- `Microsoft.Network/networkInterfaces/read` Esto es necesario en el grupo de recursos para que se cree la interfaz de red para el punto de conexión privado.
- Rol de colaborador de zona DNS privada: este rol ya existe y se puede usar para proporcionar los permisos `Microsoft.Network/privateDnsZones/A/*` y `Microsoft.Network/privateDnsZones/virtualNetworkLinks/read`.

Puede usar uno de los métodos siguientes para crear roles con los permisos necesarios:

#### <a name="create-roles-and-permissions-manually"></a>Crear roles y permisos manualmente

Cree los siguientes archivos JSON y use el comando de PowerShell que se encuentra al final de la sección para crear roles:

//PrivateEndpointContributorRoleDef.json

```json
{
  "Name": "PrivateEndpointContributor",
  "Id": null,
  "IsCustom": true,
  "Description": "Allows management of Private Endpoint",
  "Actions": [
    "Microsoft.Network/privateEndpoints/*",
  ],
  "NotActions": [],
  "AssignableScopes": [
    "/subscriptions/00000000-0000-0000-0000-000000000000"
  ]
}
```

//NetworkInterfaceReaderRoleDef.json

```json
{
  "Name": "NetworkInterfaceReader",
  "Id": null,
  "IsCustom": true,
  "Description": "Allows read on networkInterfaces",
  "Actions": [
    "Microsoft.Network/networkInterfaces/read"
  ],
  "NotActions": [],
  "AssignableScopes": [
    "/subscriptions/00000000-0000-0000-0000-000000000000"
  ]
}
```

//PrivateEndpointSubnetContributorRoleDef.json

```json
{
  "Name": "PrivateEndpointSubnetContributor",
  "Id": null,
  "IsCustom": true,
  "Description": "Allows adding of Private Endpoint connection to Virtual Networks",
  "Actions": [
    "Microsoft.Network/virtualNetworks/subnets/join/action"
  ],
  "NotActions": [],
  "AssignableScopes": [
    "/subscriptions/00000000-0000-0000-0000-000000000000"
  ]
}
```

```azurepowershell
 New-AzRoleDefinition -InputFile "PrivateEndpointContributorRoleDef.json"
 New-AzRoleDefinition -InputFile "NetworkInterfaceReaderRoleDef.json"
 New-AzRoleDefinition -InputFile "PrivateEndpointSubnetContributorRoleDef.json"
```

#### <a name="use-a-script"></a>Usar un script

1. Inicie **Cloud Shell** en Azure Portal y seleccione **Cargar archivo** en la ventana de PowerShell.

    ![Seleccionar Cargar archivo en la ventana de PowerShell](./media/private-endpoints/upload-file-in-powershell.png)

1. Cargue el siguiente script: [VaultMsiPrereqScript](https://download.microsoft.com/download/1/2/6/126a410b-0e06-45ed-b2df-84f353034fa1/VaultMsiPrereqScript.ps1)

1. Vaya a la carpeta principal (por ejemplo: `cd /home/user`)

1. Ejecute el siguiente script:

    ```azurepowershell
    ./VaultMsiPrereqScript.ps1 -subscription <subscription-Id> -vaultPEResourceGroup <vaultPERG> -vaultPESubnetResourceGroup <subnetRG> -vaultMsiName <msiName>
    ```

    Estos son los parámetros:

    - **subscription**: **SubscriptionId que tiene el grupo de recursos en el que se va a crear el punto de conexión privado para el almacén y la subred en la que se adjuntará el punto de conexión privado del almacén

    - **vaultPEResourceGroup**: El grupo de recursos donde se creará el punto de conexión privado para el almacén

    - **vaultPESubnetResourceGroup**: El grupo de recursos de la subred a la que se unirá el punto de conexión privado

    - **vaultMsiName**: Nombre del archivo MSI del almacén, que es igual que **VaultName**

1. Complete la autenticación y el script tomará el contexto de la suscripción especificada que se indicó anteriormente. Se crearán los roles adecuados si faltan en el inquilino y se asignarán roles al archivo MSI del almacén.

### <a name="creating-private-endpoints-using-azure-powershell"></a>Creación de puntos de conexión privados mediante Azure PowerShell

#### <a name="auto-approved-private-endpoints"></a>Puntos de conexión privados aprobados automáticamente

```azurepowershell
$vault = Get-AzRecoveryServicesVault `
        -ResourceGroupName $vaultResourceGroupName `
        -Name $vaultName
  
$privateEndpointConnection = New-AzPrivateLinkServiceConnection `
        -Name $privateEndpointConnectionName `
        -PrivateLinkServiceId $vault.ID `
        -GroupId "AzureBackup"  
  
$privateEndpoint = New-AzPrivateEndpoint `
        -ResourceGroupName $vmResourceGroupName `
        -Name $privateEndpointName `
        -Location $location `
        -Subnet $subnet `
        -PrivateLinkServiceConnection $privateEndpointConnection `
        -Force
```

### <a name="manual-approval-of-private-endpoints-using-the-azure-resource-manager-client"></a>Aprobación manual de los puntos de conexión privados mediante el cliente de Azure Resource Manager

1. Use **GetVault** para obtener el id. de conexión de punto de conexión privado para el punto de conexión privado.

    ```rest
    armclient GET /subscriptions/<subscriptionid>/resourceGroups/<rgname>/providers/Microsoft.RecoveryServices/vaults/<vaultname>?api-version=2017-07-01-preview
    ```

    Esto devolverá el identificador de conexión del punto de conexión privado. El nombre de la conexión se puede recuperar utilizando la primera parte del identificador de conexión como se indica a continuación:

    `privateendpointconnectionid = {peName}.{vaultId}.backup.{guid}`

1. Obtenga el **identificador de conexión del punto de conexión privado** (y el **nombre del punto de conexión privado**, donde sea necesario) de la respuesta y reemplácelo en el siguiente JSON y el URI de Azure Resource Manager e intente cambiar el estado a "Aprobado/Rechazado/Desconectado", como se muestra en el ejemplo siguiente:

    ```rest
    armclient PUT /subscriptions/<subscriptionid>/resourceGroups/<rgname>/providers/Microsoft.RecoveryServices/Vaults/<vaultname>/privateEndpointConnections/<privateendpointconnectionid>?api-version=2020-02-02-preview @C:\<filepath>\BackupAdminApproval.json
    ```

    JSON:

    ```json
    {
    "id": "/subscriptions/<subscriptionid>/resourceGroups/<rgname>/providers/Microsoft.RecoveryServices/Vaults/<vaultname>/privateEndpointConnections/<privateendpointconnectionid>",
    "properties": {
        "privateEndpoint": {
        "id": "/subscriptions/<subscriptionid>/resourceGroups/<pergname>/providers/Microsoft.Network/privateEndpoints/pename"
        },
        "privateLinkServiceConnectionState": {
        "status": "Disconnected",  //choose state from Approved/Rejected/Disconnected
        "description": "Disconnected by <userid>"
        }
    }
    }
    ```

### <a name="dns-changes-for-custom-dns-servers"></a>Cambios de DNS para los servidores DNS personalizados

#### <a name="create-dns-zones-for-custom-dns-servers"></a>Crear zonas DNS para los servidores DNS personalizados

Debe crear tres zonas DNS privadas y vincularlas a la red virtual.

| **Zona**                                                     | **Servicio** |
| ------------------------------------------------------------ | ----------- |
| `privatelink.<geo>.backup.windowsazure.com`      | Copia de seguridad      |
| `privatelink.blob.core.windows.net`                            | Blob        |
| `privatelink.queue.core.windows.net`                           | Cola       |

>[!NOTE]
>En el texto anterior, *geo* hace referencia al código de región. Por ejemplo, *wcus* y *ne* para el Centro-oeste de EE. UU. y el Norte de Europa respectivamente.

Consulte [esta lista](https://download.microsoft.com/download/1/2/6/126a410b-0e06-45ed-b2df-84f353034fa1/AzureRegionCodesList.docx) para ver los códigos de región.

#### <a name="adding-dns-records-for-custom-dns-servers"></a>Agregar registros DNS para servidores DNS personalizados

Esto requiere que se creen entradas para cada FQDN del punto de conexión privado en la zona DNS privada.

Debe tenerse en cuenta que vamos a usar los puntos de conexión privados creados para Backup, Blob y Queue service.

- El punto de conexión privado para el almacén usa el nombre especificado al crear el punto de conexión privado
- Los puntos de conexión privados para Blob y Queue service tienen como prefijo el nombre del mismo para el almacén.

Por ejemplo, en la siguiente imagen se muestran los tres puntos de conexión privados creados para una conexión de punto de conexión privado con el nombre *pee2epe*:

![Tres puntos de conexión privados para una conexión de punto de conexión privado](./media/private-endpoints/three-private-endpoints.png)

Zona DNS para el servicio de Azure Backup (`privatelink.<geo>.backup.windowsazure.com`):

1. Vaya a su punto de conexión privado para Azure Backup en el **Centro de Private Link**. La página de información general muestra el FQDN y las direcciones IP privadas del punto de conexión privado.

1. Agregue una entrada para cada FQDN y la dirección IP privada como un registro de tipo A.

    ![Agregar entrada para cada FQDN y dirección IP privada](./media/private-endpoints/add-entry-for-each-fqdn-and-ip.png)

Zona DNS para Blob service (`privatelink.blob.core.windows.net`):

1. Vaya a su punto de conexión privado para Blob en el **Centro de Private Link**. La página de información general muestra el FQDN y las direcciones IP privadas del punto de conexión privado.

1. Agregue una entrada para el FQDN y la dirección IP privada como un registro de tipo A.

    ![Agregue una entrada para el FQDN y la dirección IP privada como un registro de tipo A para Blob service](./media/private-endpoints/add-type-a-record-for-blob.png)

Zona DNS para Queue service (`privatelink.queue.core.windows.net`):

1. Vaya a su punto de conexión privado para la cola en el **Centro de Private Link**. La página de información general muestra el FQDN y las direcciones IP privadas del punto de conexión privado.

1. Agregue una entrada para el FQDN y la dirección IP privada como un registro de tipo A.

    ![Agregue una entrada para el FQDN y la dirección IP privada como un registro de tipo A para Queue service](./media/private-endpoints/add-type-a-record-for-queue.png)

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

Q. ¿Puedo crear un punto de conexión privado para un almacén de Azure Backup existente?<br>
A. No, los puntos de conexión privados solo se pueden crear para nuevos almacenes de Azure Backup. Por lo tanto, el almacén no debe haber tenido ningún elemento protegido. De hecho, no se puede realizar ningún intento de proteger ningún elemento en el almacén antes de crear puntos de conexión privados.

Q. Intenté proteger un elemento en el almacén, pero se produjo un error y el almacén todavía no contiene elementos protegidos. ¿Se pueden crear puntos de conexión privados para este almacén?<br>
A. No, el almacén no debe haber tenido ningún intento de proteger ningún elemento en el pasado.

Q. Tengo un almacén que utiliza puntos de conexión privados con fines de copias de seguridad y restauración. ¿Puedo agregar o quitar puntos de conexión privados para este almacén más tarde, aunque tenga elementos de copia de seguridad protegidos?<br>
A. Sí. Si ya creó puntos de conexión privados para un almacén y hay elementos de copia de seguridad protegidos en él, más adelante podrá agregar o quitar puntos de conexión privados según sea necesario.

Q. ¿Se puede usar también el punto de conexión privado de Azure Backup en Azure Site Recovery?<br>
A. No, el punto de conexión privado para Azure Backup solo se puede usar en Azure Backup. Tendrá que crear un nuevo punto de conexión privado para Azure Site Recovery, si es compatible con el servicio.

Q. Me faltó uno de los pasos de este artículo y procedí a proteger el origen de datos. ¿Puedo seguir usando puntos de conexión privados?<br>
A. No seguir los pasos del artículo y proceder con la protección de elementos puede provocar que el almacén no pueda usar puntos de conexión privados. Por lo tanto, se recomienda que consulte esta lista de comprobación antes de continuar con la protección de los elementos.

Q. ¿Puedo usar mi propio servidor DNS en lugar de usar la zona DNS privada de Azure o una zona DNS privada integrada?<br>
A. Sí, puede usar sus propios servidores DNS. Sin embargo, asegúrese de agregar todos los registros DNS necesarios como se sugiere en esta sección.

Q. ¿Es necesario realizar algún paso adicional en el servidor después de haber seguido el proceso detallado en este artículo?<br>
A. Después de seguir el proceso detallado en este artículo, no es necesario realizar ningún trabajo adicional para usar puntos de conexión privados para copias de seguridad y restauración.

## <a name="next-steps"></a>Pasos siguientes

- Lea todas las [características de seguridad de Azure Backup](security-overview.md).
