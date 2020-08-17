---
title: 'Configuración de grupos de disponibilidad para SQL Server en máquinas virtuales de Red Hat Enterprise Linux en Azure: Máquinas virtuales Linux | Microsoft Docs'
description: Más información sobre la configuración de la alta disponibilidad en un entorno de clústeres de Red Hat Enterprise Linux y la configuración de STONITH
ms.service: virtual-machines-linux
ms.subservice: ''
ms.topic: tutorial
author: VanMSFT
ms.author: vanto
ms.reviewer: jroth
ms.date: 06/25/2020
ms.openlocfilehash: af1df529ae0f6bb03a8d3f36e51619f273780dfe
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87086802"
---
# <a name="tutorial-configure-availability-groups-for-sql-server-on-rhel-virtual-machines-in-azure"></a>Tutorial: Configuración de grupos de disponibilidad para SQL Server en máquinas virtuales de Red Hat Enterprise Linux en Azure 
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

> [!NOTE]
> En este tutorial, vamos a usar SQL Server 2017 con RHEL 7.6, pero es posible usar SQL Server 2019 en RHEL 7 o RHEL 8 para configurar una alta disponibilidad. Los comandos para configurar los recursos del grupo de disponibilidad y el clúster de Pacemaker han cambiado en RHEL 8 y le interesará consultar el artículo [Creación de un recurso de grupo de disponibilidad](/sql/linux/sql-server-linux-availability-group-cluster-rhel#create-availability-group-resource) y los recursos de RHEL 8 para más información sobre los comandos correctos.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> - Crear un nuevo grupo de recursos, un conjunto de disponibilidad y máquinas virtuales Linux
> - Habilitar una alta disponibilidad
> - Creación de un clúster de Pacemaker
> - Configurar un agente de barrera mediante la creación de un dispositivo STONITH
> - Instalar SQL Server y mssql-tools en RHEL
> - Configurar grupos de disponibilidad AlwaysOn de SQL Server
> - Configurar recursos de grupo de disponibilidad en el clúster de Pacemaker
> - Probar una conmutación por error y el agente de barrera

En este tutorial se utilizará la CLI de Azure para implementar los recursos en Azure.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [cloud-shell-try-it.md](../../../../includes/cloud-shell-try-it.md)]

Si prefiere instalar y usar la CLI en un entorno local, para este tutorial se requiere la versión 2.0.30 de la CLI de Azure o una versión posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Si tiene varias suscripciones, [establezca la suscripción](/cli/azure/manage-azure-subscriptions-azure-cli) en la que desea implementar estos recursos.

Use el siguiente comando para crear un grupo de recursos `<resourceGroupName>` en una región. Reemplace `<resourceGroupName>` por un nombre de su elección. Vamos a utilizar `East US 2` para este tutorial. Para más información, consulte el siguiente [inicio rápido](../../../application-gateway/quick-create-cli.md).

```azurecli-interactive
az group create --name <resourceGroupName> --location eastus2
```

## <a name="create-an-availability-set"></a>Crear un conjunto de disponibilidad

El primer paso consiste en crear un conjunto de disponibilidad. Ejecute el siguiente comando en Azure Cloud Shell y reemplace `<resourceGroupName>` por el nombre del grupo de recursos. Elija un nombre para `<availabilitySetName>`.

```azurecli-interactive
az vm availability-set create \
    --resource-group <resourceGroupName> \
    --name <availabilitySetName> \
    --platform-fault-domain-count 2 \
    --platform-update-domain-count 2
```

Debería obtener los siguientes resultados cuando se complete el comando:

```output
{
  "id": "/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.Compute/availabilitySets/<availabilitySetName>",
  "location": "eastus2",
  "name": "<availabilitySetName>",
  "platformFaultDomainCount": 2,
  "platformUpdateDomainCount": 2,
  "proximityPlacementGroup": null,
  "resourceGroup": "<resourceGroupName>",
  "sku": {
    "capacity": null,
    "name": "Aligned",
    "tier": null
  },
  "statuses": null,
  "tags": {},
  "type": "Microsoft.Compute/availabilitySets",
  "virtualMachines": []
}
```

## <a name="create-rhel-vms-inside-the-availability-set"></a>Creación de máquinas virtuales de Red Hat Enterprise Linux en el conjunto de disponibilidad

> [!WARNING]
> Si elige una imagen de RHEL de pago por uso y configura alta disponibilidad, puede que se le solicite que registre la suscripción. Esto puede hacer que pague dos veces por la suscripción, ya que se le cobrará por la suscripción de RHEL de Microsoft Azure de la máquina virtual y una suscripción a Red Hat. Para más información, consulte https://access.redhat.com/solutions/2458541.
>
> Para evitar que se le "facture dos veces", use una imagen de alta disponibilidad de RHEL al crear la máquina virtual de Azure. Las imágenes que se ofrecen como RHEL-HA también son imágenes de pago por uso con repositorio de alta disponibilidad habilitado previamente.

1. Obtenga una lista de imágenes de máquinas virtuales que ofrecen RHEL con alta disponibilidad:

    ```azurecli-interactive
    az vm image list --all --offer "RHEL-HA"
    ```

    Debería ver los siguientes resultados:

    ```output
    [
      {
    "offer": "RHEL-HA",
    "publisher": "RedHat",
    "sku": "7.4",
    "urn": "RedHat:RHEL-HA:7.4:7.4.2019062021",
    "version": "7.4.2019062021"
       },
       {
    "offer": "RHEL-HA",
    "publisher": "RedHat",
    "sku": "7.5",
    "urn": "RedHat:RHEL-HA:7.5:7.5.2019062021",
    "version": "7.5.2019062021"
        },
        {
    "offer": "RHEL-HA",
    "publisher": "RedHat",
    "sku": "7.6",
    "urn": "RedHat:RHEL-HA:7.6:7.6.2019062019",
    "version": "7.6.2019062019"
         },
         {
    "offer": "RHEL-HA",
    "publisher": "RedHat",
    "sku": "8.0",
    "urn": "RedHat:RHEL-HA:8.0:8.0.2020021914",
    "version": "8.0.2020021914"
         },
         {
    "offer": "RHEL-HA",
    "publisher": "RedHat",
    "sku": "8.1",
    "urn": "RedHat:RHEL-HA:8.1:8.1.2020021914",
    "version": "8.1.2020021914"
          },
          {
    "offer": "RHEL-HA",
    "publisher": "RedHat",
    "sku": "80-gen2",
    "urn": "RedHat:RHEL-HA:80-gen2:8.0.2020021915",
    "version": "8.0.2020021915"
           },
           {
    "offer": "RHEL-HA",
    "publisher": "RedHat",
    "sku": "81_gen2",
    "urn": "RedHat:RHEL-HA:81_gen2:8.1.2020021915",
    "version": "8.1.2020021915"
           }
    ]
    ```

    En este tutorial, vamos a elegir el `RedHat:RHEL-HA:7.6:7.6.2019062019` de imagen para el ejemplo de RHEL 7 y el `RedHat:RHEL-HA:8.1:8.1.2020021914` para el ejemplo de RHEL 8.
    
    También puede elegir SQL Server 2019 preinstalado en las imágenes de RHEL8-HA. Para obtener la lista de estas imágenes, ejecute el siguiente comando:  
    
    ```azurecli-interactive
    az vm image list --all --offer "sql2019-rhel8"
    ```

    Debería ver los siguientes resultados:

    ```output
    [
      {
    "offer": "sql2019-rhel8",
    "publisher": "MicrosoftSQLServer",
    "sku": "enterprise",
    "urn": "MicrosoftSQLServer:sql2019-rhel8:enterprise:15.0.200317",
    "version": "15.0.200317"
       },
       }
    "offer": "sql2019-rhel8",
    "publisher": "MicrosoftSQLServer",
    "sku": "enterprise",
    "urn": "MicrosoftSQLServer:sql2019-rhel8:enterprise:15.0.200512",
    "version": "15.0.200512"
       },
       {
    "offer": "sql2019-rhel8",
    "publisher": "MicrosoftSQLServer",
    "sku": "sqldev",
    "urn": "MicrosoftSQLServer:sql2019-rhel8:sqldev:15.0.200317",
    "version": "15.0.200317"
       },
       {
    "offer": "sql2019-rhel8",
    "publisher": "MicrosoftSQLServer",
    "sku": "sqldev",
    "urn": "MicrosoftSQLServer:sql2019-rhel8:sqldev:15.0.200512",
    "version": "15.0.200512"
       },
       {
    "offer": "sql2019-rhel8",
    "publisher": "MicrosoftSQLServer",
    "sku": "standard",
    "urn": "MicrosoftSQLServer:sql2019-rhel8:standard:15.0.200317",
    "version": "15.0.200317"
       },
       {
    "offer": "sql2019-rhel8",
    "publisher": "MicrosoftSQLServer",
    "sku": "standard",
    "urn": "MicrosoftSQLServer:sql2019-rhel8:standard:15.0.200512",
    "version": "15.0.200512"
       }
    ]
    ```

    Si usa una de las imágenes anteriores para crear las máquinas virtuales, tiene SQL Server 2019 instalado previamente. Omita la sección [Instalación de SQL Server y mssql-tools](#install-sql-server-and-mssql-tools) tal y como se describe en este artículo.
    
    
    > [!IMPORTANT]
    > Los nombres de las máquinas deben tener menos de 15 caracteres para configurar el grupo de disponibilidad. El nombre de usuario no puede contener caracteres en mayúsculas y las contraseñas deben tener más de 12 caracteres.

1. Queremos crear 3 máquinas virtuales en el conjunto de disponibilidad. Reemplace estos elementos en el comando siguiente:

    - `<resourceGroupName>`
    - `<VM-basename>`
    - `<availabilitySetName>`
    - `<VM-Size>` - Un ejemplo sería "Standard_D16_v3"
    - `<username>`
    - `<adminPassword>`

    ```azurecli-interactive
    for i in `seq 1 3`; do
           az vm create \
             --resource-group <resourceGroupName> \
             --name <VM-basename>$i \
             --availability-set <availabilitySetName> \
             --size "<VM-Size>"  \
             --image "RedHat:RHEL-HA:7.6:7.6.2019062019" \
             --admin-username "<username>" \
             --admin-password "<adminPassword>" \
             --authentication-type all \
             --generate-ssh-keys
    done
    ```

El comando anterior crea las máquinas virtuales y una red virtual predeterminada para ellas. Para más información sobre las distintas configuraciones, consulte el artículo [az vm create](https://docs.microsoft.com/cli/azure/vm).

Debe obtener resultados similares a los siguientes una vez que el comando se complete para cada máquina virtual:

```output
{
  "fqdns": "",
  "id": "/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.Compute/virtualMachines/<VM1>",
  "location": "eastus2",
  "macAddress": "<Some MAC address>",
  "powerState": "VM running",
  "privateIpAddress": "<IP1>",
  "publicIpAddress": "",
  "resourceGroup": "<resourceGroupName>",
  "zones": ""
}
```

> [!IMPORTANT]
> La imagen predeterminada que se genera con el comando anterior crea un disco de sistema operativo de 32 GB de forma predeterminada. Podría quedarse sin espacio en esta instalación predeterminada. Puede usar el siguiente parámetro agregado al comando `az vm create` anterior para crear un disco del sistema operativo con 128 GB a modo de ejemplo: `--os-disk-size-gb 128`.
>
> Después, puede [configurar el administrador de volúmenes lógicos (LVM)](../../../virtual-machines/linux/configure-lvm.md) si necesita expandir los volúmenes de las carpetas apropiadas para que se adapten a su instalación.

### <a name="test-connection-to-the-created-vms"></a>Prueba de la conexión a las máquinas virtuales creadas

Conéctese a VM1 o a las demás máquinas virtuales con el siguiente comando en Azure Cloud Shell. Si no puede encontrar las direcciones IP de la máquina virtual, siga este [inicio rápido sobre Azure Cloud Shell](../../../cloud-shell/quickstart.md#ssh-into-your-linux-vm).

```azurecli-interactive
ssh <username>@publicipaddress
```

Si la conexión se realiza correctamente, verá la siguiente salida que representa el terminal de Linux:

```output
[<username>@<VM1> ~]$
```

Escriba `exit` para salir de la sesión de SSH.

## <a name="enable-high-availability"></a>Habilitación de alta disponibilidad

> [!IMPORTANT]
> Para completar esta parte del tutorial, debe tener una suscripción para RHEL y para el complemento de alta disponibilidad. Si usa la imagen recomendada en la sección anterior, no tiene que registrar otra suscripción.
 
Conéctese a cada nodo de máquina virtual y siga la guía siguiente para habilitar la alta disponibilidad. Para más información, consulte [Habilitación de la suscripción de alta disponibilidad de RHEL](/sql/linux/sql-server-linux-availability-group-cluster-rhel#enable-the-high-availability-subscription-for-rhel).

> [!TIP]
> Será más fácil si abre una sesión de SSH en cada una de las máquinas virtuales simultáneamente, ya que se tendrán que ejecutar los mismos comandos en cada máquina virtual a lo largo del artículo.
>
> Si va a copiar y pegar varios comandos `sudo` y se le pide una contraseña, no se ejecutarán los comandos adicionales. Ejecute cada comando por separado.


1. Ejecute los siguientes comandos en cada máquina virtual para abrir los puertos del firewall de Pacemaker:

    ```bash
    sudo firewall-cmd --permanent --add-service=high-availability
    sudo firewall-cmd --reload
    ```

1. Actualice e instale paquetes de Pacemaker en todos los nodos mediante los siguientes comandos:

    > [!NOTE]
    > **nmap** se instala como parte de este bloque de comandos como herramienta para buscar las direcciones IP disponibles en la red. No es necesario que instale **nmap**, pero será útil más adelante en este tutorial.

    ```bash
    sudo yum update -y
    sudo yum install -y pacemaker pcs fence-agents-all resource-agents fence-agents-azure-arm nmap
    sudo reboot
    ```

1. Establezca la contraseña para el usuario predeterminado que se crea al instalar paquetes de Pacemaker. Use la misma contraseña en todos los nodos.

    ```bash
    sudo passwd hacluster
    ```

1. Use el siguiente comando para abrir el archivo de hosts y configurar la resolución de nombres de host. Para más información, consulte [Configurar un grupo de disponibilidad](/sql/linux/sql-server-linux-availability-group-configure-ha#prerequisites) que trata sobre la configuración del archivo de hosts.

    ```
    sudo vi /etc/hosts
    ```

    En el editor de **vi**, escriba `i` para insertar texto y, en una línea en blanco, agregue la **dirección IP privada** de la máquina virtual correspondiente. A continuación, agregue el nombre de la máquina virtual después de un espacio junto a la dirección IP. Cada línea debe tener una entrada independiente.

    ```output
    <IP1> <VM1>
    <IP2> <VM2>
    <IP3> <VM3>
    ```

    > [!IMPORTANT]
    > Le recomendamos que use la **dirección IP privada** anterior. El uso de la dirección IP pública en esta configuración hará que se produzca un error en la configuración y no se recomienda exponer la máquina virtual a redes externas.

    Para salir del editor de **vi**, primero presione la tecla **Esc** y, a continuación, escriba el comando `:wq` para escribir en el archivo y salir.

## <a name="create-the-pacemaker-cluster"></a>Implementar un clúster de Pacemaker

En esta sección, se habilitará e iniciará el servicio pcsd y, a continuación, se configurará el clúster. En el caso de Microsoft SQL Server en Linux, los recursos de clúster no se crean automáticamente. Se tendrán que habilitar y crear los recursos de Pacemaker manualmente. Para más información, consulte el artículo sobre la [configuración de una instancia de clúster de conmutación por error para RHEL](/sql/linux/sql-server-linux-shared-disk-cluster-red-hat-7-configure#install-and-configure-pacemaker-on-each-cluster-node).

### <a name="enable-and-start-pcsd-service-and-pacemaker"></a>Habilitar e iniciar el servicio pcsd y Pacemaker

1. Ejecute los comandos en todos los nodos. Estos comandos permiten que los nodos vuelvan a unirse al clúster después de un reinicio.

    ```bash
    sudo systemctl enable pcsd
    sudo systemctl start pcsd
    sudo systemctl enable pacemaker
    ``` 

1. Quite la configuración del clúster existente de todos los nodos. Ejecute el siguiente comando:

    ```bash
    sudo pcs cluster destroy 
    sudo systemctl enable pacemaker 
    ```

1. En el nodo principal, ejecute los siguientes comandos para configurar el clúster.

    - Cuando ejecute el comando `pcs cluster auth` para autenticar los nodos del clúster, se le solicitará una contraseña. Escriba la contraseña del usuario **hacluster** que creó anteriormente.

    **RHEL7**

    ```bash
    sudo pcs cluster auth <VM1> <VM2> <VM3> -u hacluster
    sudo pcs cluster setup --name az-hacluster <VM1> <VM2> <VM3> --token 30000
    sudo pcs cluster start --all
    sudo pcs cluster enable --all
    ```

    **RHEL8**

    Para RHEL 8, tendrá que autenticar los nodos por separado. Escriba manualmente el nombre de usuario y la contraseña de **hacluster** cuando se le solicite.

    ```bash
    sudo pcs host auth <node1> <node2> <node3>
    sudo pcs cluster setup <clusterName> <node1> <node2> <node3>
    sudo pcs cluster start --all
    sudo pcs cluster enable --all
    ```

1. Ejecute el siguiente comando para comprobar que todos los nodos están en línea.

    ```bash
    sudo pcs status
    ```

   **RHEL 7** 
   
    Si todos los nodos están en línea, verá una salida similar a la siguiente:

    ```output
    Cluster name: az-hacluster
     
    WARNINGS:
    No stonith devices and stonith-enabled is not false
     
    Stack: corosync
    Current DC: <VM2> (version 1.1.19-8.el7_6.5-c3c624ea3d) - partition with quorum
    Last updated: Fri Aug 23 18:27:57 2019
    Last change: Fri Aug 23 18:27:56 2019 by hacluster via crmd on <VM2>
     
    3 nodes configured
    0 resources configured
     
    Online: [ <VM1> <VM2> <VM3> ]
     
    No resources
     
     
    Daemon Status:
          corosync: active/enabled
          pacemaker: active/enabled
          pcsd: active/enabled
    ```
   
   **RHEL 8** 
   
    ```output
    Cluster name: az-hacluster
     
    WARNINGS:
    No stonith devices and stonith-enabled is not false
     
    Cluster Summary:
    * Stack: corosync
    * Current DC: <VM2> (version 1.1.19-8.el7_6.5-c3c624ea3d) - partition with quorum
    * Last updated: Fri Aug 23 18:27:57 2019
    * Last change: Fri Aug 23 18:27:56 2019 by hacluster via crmd on <VM2>
    * 3 nodes configured
    * 0 resource instances configured
     
   Node List:
    * Online: [ <VM1> <VM2> <VM3> ]
   
   Full List of Resources:
   * No resources
     
   Daemon Status:
          corosync: active/enabled
          pacemaker: active/enabled
          pcsd: active/enabled
    
    ```
    
1. Establezca los votos esperados en el clúster activo en 3. Este comando solo afecta al clúster activo y no cambia los archivos de configuración.

    En todos los nodos, establezca los votos esperados con el siguiente comando:

    ```bash
    sudo pcs quorum expected-votes 3
    ```

## <a name="configure-the-fencing-agent"></a>Configuración del agente de barrera

Un dispositivo STONITH proporciona un agente de barrera. Las instrucciones siguientes se han modificado para este tutorial. Para más información, consulte la [creación de un dispositivo STONITH](../../../virtual-machines/workloads/sap/high-availability-guide-rhel-pacemaker.md#create-stonith-device).
 
[Compruebe la versión del agente de barrera de Azure para asegurarse de que está actualizado](../../../virtual-machines/workloads/sap/high-availability-guide-rhel-pacemaker.md#cluster-installation). Use el comando siguiente:

```bash
sudo yum info fence-agents-azure-arm
```

Debería ver una salida similar a la del ejemplo siguiente.

```output
Loaded plugins: langpacks, product-id, search-disabled-repos, subscription-manager
Installed Packages
Name        : fence-agents-azure-arm
Arch        : x86_64
Version     : 4.2.1
Release     : 11.el7_6.8
Size        : 28 k
Repo        : installed
From repo   : rhel-ha-for-rhel-7-server-eus-rhui-rpms
Summary     : Fence agent for Azure Resource Manager
URL         : https://github.com/ClusterLabs/fence-agents
License     : GPLv2+ and LGPLv2+
Description : The fence-agents-azure-arm package contains a fence agent for Azure instances.
```

### <a name="register-a-new-application-in-azure-active-directory"></a>Registro de una nueva aplicación en Azure Active Directory
 
 1. Vaya a https://portal.azure.com.
 2. Abra la [hoja Azure Active Directory](https://ms.portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Properties). Vaya a Propiedades y anote el identificador del directorio. Éste es el valor de `tenant ID`
 3. Haga clic en [**Registros de aplicaciones**](https://ms.portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade)
 4. Haga clic en **Nuevo registro**
 5. Escriba un **nombre** como `<resourceGroupName>-app` y seleccione **Solo las cuentas de este directorio organizativo**
 6. Seleccione el tipo de aplicación **Web**, escriba una dirección URL de inicio de sesión (por ejemplo, http://localhost) ) y haga clic en Agregar. La dirección URL de inicio de sesión no se usa y puede ser cualquier dirección URL válida. A continuación, haga clic en **Registrar**
 7. Seleccione **Certificados y secretos** para el nuevo registro de aplicaciones y, después, haga clic en **Nuevo secreto de cliente**
 8. Escriba una descripción para la nueva clave (secreto de cliente), seleccione **Nunca expira** y haga clic en **Agregar**
 9. Anote el valor del secreto. Se usa como contraseña para la entidad de servicio
10. Seleccione **Información general**. Anote el identificador de la aplicación. Se utiliza como nombre de usuario (identificador de inicio de sesión en los pasos siguientes) de la entidad de servicio
 
### <a name="create-a-custom-role-for-the-fence-agent"></a>Creación de un rol personalizado para el agente de barrera

Siga el tutorial para [crear un rol personalizado de Azure con la CLI de Azure](../../../role-based-access-control/tutorial-custom-role-cli.md#create-a-custom-role).

El archivo json debe tener un aspecto similar al siguiente:

- Reemplace `<username>` por el nombre que prefiera. Esto se hace para evitar la duplicación al crear esta definición de roles.
- Reemplace `<subscriptionId>` con la identificación de su suscripción de Azure.

```json
{
  "Name": "Linux Fence Agent Role-<username>",
  "Id": null,
  "IsCustom": true,
  "Description": "Allows to power-off and start virtual machines",
  "Actions": [
    "Microsoft.Compute/*/read",
    "Microsoft.Compute/virtualMachines/powerOff/action",
    "Microsoft.Compute/virtualMachines/start/action"
  ],
  "NotActions": [
  ],
  "AssignableScopes": [
    "/subscriptions/<subscriptionId>"
  ]
}
```

Para agregar el rol, ejecute el siguiente comando:

- Reemplace `<filename>` por el nombre del archivo.
- Si va a ejecutar el comando desde una ruta de acceso distinta de la carpeta en la que se guarda el archivo, incluya la ruta de acceso de la carpeta del archivo en el comando.

```bash
az role definition create --role-definition "<filename>.json"
```

Debería ver la siguiente salida:

```output
{
  "assignableScopes": [
    "/subscriptions/<subscriptionId>"
  ],
  "description": "Allows to power-off and start virtual machines",
  "id": "/subscriptions/<subscriptionId>/providers/Microsoft.Authorization/roleDefinitions/<roleNameId>",
  "name": "<roleNameId>",
  "permissions": [
    {
      "actions": [
        "Microsoft.Compute/*/read",
        "Microsoft.Compute/virtualMachines/powerOff/action",
        "Microsoft.Compute/virtualMachines/start/action"
      ],
      "dataActions": [],
      "notActions": [],
      "notDataActions": []
    }
  ],
  "roleName": "Linux Fence Agent Role-<username>",
  "roleType": "CustomRole",
  "type": "Microsoft.Authorization/roleDefinitions"
}
```

### <a name="assign-the-custom-role-to-the-service-principal"></a>Asignación del rol personalizado a la entidad de servicio

Asigne el rol personalizado `Linux Fence Agent Role-<username>` que se creó en el último paso a la entidad de servicio. Deje de utilizar el rol de propietario.
 
1. Vaya a https://portal.azure.com.
2. Abra la [hoja Todos los recursos](https://ms.portal.azure.com/#blade/HubsExtension/BrowseAll)
3. Seleccione la máquina virtual del primer nodo de clúster.
4. Haga clic en **Control de acceso (IAM)**
5. Haga clic en **Agregar una asignación de roles**
6. Seleccione el rol `Linux Fence Agent Role-<username>` de la lista **Rol**
7. En la lista **Seleccionar**, escriba el nombre de la aplicación que creó anteriormente, `<resourceGroupName>-app`.
8. Haga clic en **Guardar**
9. Repita los pasos anteriores para todo el nodo de clúster.

### <a name="create-the-stonith-devices"></a>Creación de los dispositivos STONITH

Ejecute los comandos siguientes en el nodo 1:

- Reemplace `<ApplicationID>` por el valor del identificador del registro de aplicación.
- Reemplace `<servicePrincipalPassword>` por el valor del secreto de cliente.
- Reemplace `<resourceGroupName>` por el grupo de recursos de la suscripción que se usa para este tutorial.
- Reemplace los valores de `<tenantID>` y `<subscriptionId>` de su suscripción de Azure.

```bash
sudo pcs property set stonith-timeout=900
sudo pcs stonith create rsc_st_azure fence_azure_arm login="<ApplicationID>" passwd="<servicePrincipalPassword>" resourceGroup="<resourceGroupName>" tenantId="<tenantID>" subscriptionId="<subscriptionId>" power_timeout=240 pcmk_reboot_timeout=900
```

Dado que ya hemos agregado una regla a nuestro firewall para permitir el servicio de alta disponibilidad (`--add-service=high-availability`), no es necesario abrir los siguientes puertos del firewall en todos los nodos: 2224, 3121, 21064, 5405. Sin embargo, si tiene algún tipo de problemas de conexión con la alta disponibilidad, use el siguiente comando para abrir estos puertos asociados a ella.

> [!TIP]
> Opcionalmente, puede agregar todos los puertos de este tutorial a la vez para ahorrar algo de tiempo. Los puertos que se deben abrir se explican en sus respectivas secciones a continuación. Si desea agregar todos los puertos ahora, agregue los puertos adicionales: 1433 y 5022.

```bash
sudo firewall-cmd --zone=public --add-port=2224/tcp --add-port=3121/tcp --add-port=21064/tcp --add-port=5405/tcp --permanent
sudo firewall-cmd --reload
```

## <a name="install-sql-server-and-mssql-tools"></a>Instalación de SQL Server y mssql-tools

> [!NOTE]
> Si ha creado las máquinas virtuales con SQL Server 2019 preinstalado en RHEL8-HA, puede omitir los pasos siguientes para instalar SQL Server y mssql-tools, e iniciar la **configuración de un grupo de disponibilidad** después de configurar la contraseña de administrador del sistema en todas las máquinas virtuales mediante la ejecución del comando `sudo /opt/mssql/bin/mssql-conf set-sa-password` en todas las máquinas virtuales.

Use la sección siguiente para instalar SQL Server y mssql-tools en las máquinas virtuales. Puede elegir uno de los ejemplos siguientes para instalar SQL Server 2017 en RHEL 7 o SQL Server 2019 en RHEL 8. Realice cada una de estas acciones en todos los nodos. Para más información, consulte [Instalación de SQL Server en una máquina virtual de Red Hat](/sql/linux/quickstart-install-connect-red-hat).


### <a name="installing-sql-server-on-the-vms"></a>Instalación de SQL Server en las máquinas virtuales

Utilice los comandos siguientes para instalar SQL Server:

**RHEL 7 con SQL Server 2017** 

```bash
sudo curl -o /etc/yum.repos.d/mssql-server.repo https://packages.microsoft.com/config/rhel/7/mssql-server-2017.repo
sudo yum install -y mssql-server
sudo /opt/mssql/bin/mssql-conf setup
sudo yum install mssql-server-ha
```

**RHEL 8 con SQL Server 2019** 

```bash
sudo curl -o /etc/yum.repos.d/mssql-server.repo https://packages.microsoft.com/config/rhel/8/mssql-server-2019.repo
sudo yum install -y mssql-server
sudo /opt/mssql/bin/mssql-conf setup
sudo yum install mssql-server-ha
```
### <a name="open-firewall-port-1433-for-remote-connections"></a>Apertura del puerto del firewall 1433 para las conexiones remotas

Tendrá que abrir el puerto 1433 en la máquina virtual para conectarse de forma remota. Use los siguientes comandos para abrir el puerto 1433 en el firewall de cada máquina virtual:

```bash
sudo firewall-cmd --zone=public --add-port=1433/tcp --permanent
sudo firewall-cmd --reload
```

### <a name="installing-sql-server-command-line-tools"></a>Instalación de las herramientas de la línea de comandos de SQL Server

Se utilizan los siguientes comandos para instalar herramientas de la línea de comandos de SQL Server. Para más información, consulte [Instalación de las herramientas de la línea de comandos de SQL Server](/sql/linux/quickstart-install-connect-red-hat#tools).

**RHEL 7** 

```bash
sudo curl -o /etc/yum.repos.d/msprod.repo https://packages.microsoft.com/config/rhel/7/prod.repo
sudo yum install -y mssql-tools unixODBC-devel
```

**RHEL 8** 

```bash
sudo curl -o /etc/yum.repos.d/msprod.repo https://packages.microsoft.com/config/rhel/8/prod.repo
sudo yum install -y mssql-tools unixODBC-devel
```
 
> [!NOTE] 
> Por comodidad, agregue /opt/mssql-tools/bin/ a la variable de entorno PATH. De este modo, podrá ejecutar las herramientas sin especificar la ruta de acceso completa. Ejecute los comandos siguientes para modificar la variable PATH, tanto para las sesiones de inicio de sesión como para las sesiones interactivas o sin inicio de sesión:</br></br>
`echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bash_profile`</br>
`echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bashrc`</br>
`source ~/.bashrc`


### <a name="check-the-status-of-the-sql-server"></a>Comprobación del estado de la instancia de SQL Server

Una vez que haya terminado con la configuración, puede comprobar el estado de SQL Server y comprobar que se está ejecutando:

```bash
systemctl status mssql-server --no-pager
```

Debería ver la siguiente salida:

```output
● mssql-server.service - Microsoft SQL Server Database Engine
   Loaded: loaded (/usr/lib/systemd/system/mssql-server.service; enabled; vendor preset: disabled)
   Active: active (running) since Thu 2019-12-05 17:30:55 UTC; 20min ago
     Docs: https://docs.microsoft.com/en-us/sql/linux
 Main PID: 11612 (sqlservr)
   CGroup: /system.slice/mssql-server.service
           ├─11612 /opt/mssql/bin/sqlservr
           └─11640 /opt/mssql/bin/sqlservr
```

## <a name="configure-an-availability-group"></a>Configuración de un grupo de disponibilidad

Utilice los siguientes pasos para configurar un grupo de disponibilidad AlwaysOn de SQL Server para las máquinas virtuales. Para más información, consulte [Configuración de un grupo de disponibilidad AlwaysOn de SQL Server para alta disponibilidad en Linux](/sql/linux/sql-server-linux-availability-group-configure-ha)

### <a name="enable-always-on-availability-groups-and-restart-mssql-server"></a>Habilitar los grupos de disponibilidad AlwaysOn y reiniciar mssql-server

Habilite los grupos de disponibilidad AlwaysOn en cada nodo en el que se hospede una instancia de SQL Server. Después, reinicie mssql-server. Ejecute el siguiente script:

```
sudo /opt/mssql/bin/mssql-conf set hadr.hadrenabled 1
sudo systemctl restart mssql-server
```

### <a name="create-a-certificate"></a>Crear un certificado

No se admite la autenticación de AD en el punto de conexión del grupo de disponibilidad. Por tanto, debemos utilizar un certificado para el cifrado del punto de conexión del grupo de disponibilidad.

1. Conéctese a **todos los nodos** mediante SQL Server Management Studio (SSMS) o SQL CMD. Ejecute los siguientes comandos para habilitar la sesión de AlwaysOn_health y crear una clave maestra:

    > [!IMPORTANT]
    > Si se conecta de forma remota a la instancia de SQL Server, deberá tener el puerto 1433 abierto en el firewall. También tendrá que permitir conexiones entrantes al puerto 1433 en el grupo de seguridad de red de cada máquina virtual. Para más información, consulte [Creación de una regla de seguridad](../../../virtual-network/manage-network-security-group.md#create-a-security-rule) para crear una regla de seguridad de entrada.

    - Reemplace `<Master_Key_Password>` por su propia contraseña.


    ```sql
    ALTER EVENT SESSION  AlwaysOn_health ON SERVER WITH (STARTUP_STATE=ON);
    GO
    CREATE MASTER KEY ENCRYPTION BY PASSWORD = '<Master_Key_Password>';
    ```

 
1. Conéctese a la réplica principal mediante SSMS o SQL CMD. Los comandos siguientes le permiten crear un certificado en `/var/opt/mssql/data/dbm_certificate.cer` y una clave privada en `var/opt/mssql/data/dbm_certificate.pvk` en la réplica principal de SQL Server:

    - Reemplace `<Private_Key_Password>` por su propia contraseña.
    
    ```sql
    CREATE CERTIFICATE dbm_certificate WITH SUBJECT = 'dbm';
    GO
    
    BACKUP CERTIFICATE dbm_certificate
       TO FILE = '/var/opt/mssql/data/dbm_certificate.cer'
       WITH PRIVATE KEY (
               FILE = '/var/opt/mssql/data/dbm_certificate.pvk',
               ENCRYPTION BY PASSWORD = '<Private_Key_Password>'
           );
    GO
    ```

Salga de la sesión de SQL CMD. Para ello, ejecute el comando `exit` y vuelva a la sesión de SSH.
 
### <a name="copy-the-certificate-to-the-secondary-replicas-and-create-the-certificates-on-the-server"></a>Copia del certificado en las réplicas secundarias y creación de los certificados en el servidor

1. Copie los dos archivos que se crearon en la misma ubicación en todos los servidores que hospedarán las réplicas de disponibilidad.
 
    En el servidor principal, ejecute el siguiente comando `scp` para copiar el certificado en los servidores de destino:

    - Reemplace `<username>` y `<VM2>` por el nombre de usuario y el nombre de la máquina virtual de destino que está usando.
    - Ejecute este comando para todas las réplicas secundarias.

    > [!NOTE]
    > No tiene que ejecutar `sudo -i`, lo que le proporciona el entorno raíz. Podría ejecutar el comando `sudo` delante de cada comando como ya hicimos anteriormente en este tutorial.

    ```bash
    # The below command allows you to run commands in the root environment
    sudo -i
    ```

    ```bash
    scp /var/opt/mssql/data/dbm_certificate.* <username>@<VM2>:/home/<username>
    ```

1. En el servidor de destino, ejecute el siguiente comando:

    - Reemplace `<username>` por el nombre de usuario.
    - El comando `mv` mueve los archivos o directorios de un lugar a otro.
    - El comando `chown` se usa para cambiar el propietario y el grupo de archivos, directorios o vínculos.
    - Ejecute estos comandos para todas las réplicas secundarias.

    ```bash
    sudo -i
    mv /home/<username>/dbm_certificate.* /var/opt/mssql/data/
    cd /var/opt/mssql/data
    chown mssql:mssql dbm_certificate.*
    ```

1. El script de Transact-SQL siguiente crea un certificado a partir de la copia de seguridad creada en la réplica principal de SQL Server. Actualice el script con contraseñas seguras. La contraseña de descifrado es la misma que se usó para crear el archivo .pvk en el paso anterior. Ejecute el siguiente script mediante SQL CMD o SSMS en todos los servidores secundarios para crear el certificado:

    ```sql
    CREATE CERTIFICATE dbm_certificate
        FROM FILE = '/var/opt/mssql/data/dbm_certificate.cer'
        WITH PRIVATE KEY (
        FILE = '/var/opt/mssql/data/dbm_certificate.pvk',
        DECRYPTION BY PASSWORD = '<Private_Key_Password>'
                );
    GO
    ```

### <a name="create-the-database-mirroring-endpoints-on-all-replicas"></a>Crear los puntos de conexión de creación de reflejo de la base de datos en todas las réplicas

Ejecute el script siguiente en todas las instancias de SQL Server mediante SQL CMD o SSMS:

```sql
CREATE ENDPOINT [Hadr_endpoint]
    AS TCP (LISTENER_PORT = 5022)
    FOR DATABASE_MIRRORING (
    ROLE = ALL,
    AUTHENTICATION = CERTIFICATE dbm_certificate,
ENCRYPTION = REQUIRED ALGORITHM AES
);
GO

ALTER ENDPOINT [Hadr_endpoint] STATE = STARTED;
GO
```

### <a name="create-the-availability-group"></a>Crear el grupo de disponibilidad

Conéctese a la instancia de SQL Server que hospeda la réplica principal mediante SQL CMD o SSMS. Ejecute el siguiente comando para crear el grupo de disponibilidad:

- Reemplace `ag1` por el nombre del grupo de disponibilidad que prefiera.
- Reemplace los valores `<VM1>`, `<VM2>` y `<VM3>` con los nombres de las instancias de SQL Server que hospedan las réplicas.

```sql
CREATE AVAILABILITY GROUP [ag1]
     WITH (DB_FAILOVER = ON, CLUSTER_TYPE = EXTERNAL)
     FOR REPLICA ON
         N'<VM1>'
          WITH (
             ENDPOINT_URL = N'tcp://<VM1>:5022',
             AVAILABILITY_MODE = SYNCHRONOUS_COMMIT,
             FAILOVER_MODE = EXTERNAL,
             SEEDING_MODE = AUTOMATIC
             ),
         N'<VM2>'
          WITH (
             ENDPOINT_URL = N'tcp://<VM2>:5022',
             AVAILABILITY_MODE = SYNCHRONOUS_COMMIT,
             FAILOVER_MODE = EXTERNAL,
             SEEDING_MODE = AUTOMATIC
             ),
         N'<VM3>'
         WITH(
            ENDPOINT_URL = N'tcp://<VM3>:5022',
            AVAILABILITY_MODE = SYNCHRONOUS_COMMIT,
            FAILOVER_MODE = EXTERNAL,
            SEEDING_MODE = AUTOMATIC
            );
GO

ALTER AVAILABILITY GROUP [ag1] GRANT CREATE ANY DATABASE;
GO
```

### <a name="create-a-sql-server-login-for-pacemaker"></a>Crear un inicio de sesión de SQL Server para Pacemaker

En todas las instancias de SQL Server, cree un inicio de sesión de SQL Server para Pacemaker. La siguiente instrucción Transact-SQL crea un inicio de sesión.

- Reemplace `<password>` por una contraseña propia compleja.

```sql
USE [master]
GO

CREATE LOGIN [pacemakerLogin] with PASSWORD= N'<password>';
GO

ALTER SERVER ROLE [sysadmin] ADD MEMBER [pacemakerLogin];
GO
```

En todas las instancias de SQL Server, guarde las credenciales usadas para el inicio de sesión de SQL Server. 

1. Cree el archivo:

    ```bash
    sudo vi /var/opt/mssql/secrets/passwd
    ```

1. Agregue las 2 líneas siguientes al archivo:

    ```bash
    pacemakerLogin
    <password>
    ```

    Para salir del editor de **vi**, primero presione la tecla **Esc** y, a continuación, escriba el comando `:wq` para escribir en el archivo y salir.

1. Haga que solo el usuario raíz pueda leer el archivo:

    ```bash
    sudo chown root:root /var/opt/mssql/secrets/passwd
    sudo chmod 400 /var/opt/mssql/secrets/passwd
    ```

### <a name="join-secondary-replicas-to-the-availability-group"></a>Conexión de las réplicas secundarias al grupo de disponibilidad

1. Para conectar las réplicas secundarias al grupo de disponibilidad, deberá abrir el puerto 5022 en el firewall para todos los servidores. Ejecute el siguiente comando en la sesión de SSH:

    ```bash
    sudo firewall-cmd --zone=public --add-port=5022/tcp --permanent
    sudo firewall-cmd --reload
    ```

1. En las réplicas secundarias, ejecute los siguientes comandos para conectarlas al grupo de disponibilidad:

    ```sql
    ALTER AVAILABILITY GROUP [ag1] JOIN WITH (CLUSTER_TYPE = EXTERNAL);
    GO

    ALTER AVAILABILITY GROUP [ag1] GRANT CREATE ANY DATABASE;
    GO
    ```

1. Ejecute el script de Transact-SQL siguiente en la réplica principal y en cada una de las réplicas secundarias:

    ```sql
    GRANT ALTER, CONTROL, VIEW DEFINITION ON AVAILABILITY GROUP::ag1 TO pacemakerLogin;
    GO
    
    GRANT VIEW SERVER STATE TO pacemakerLogin;
    GO
    ```

1. Una vez que se conectan las réplicas secundarias, puede verlas en el Explorador de objetos de SSMS expandiendo el nodo de **alta disponibilidad de Always On**:

    ![availability-group-joined.png](./media/rhel-high-availability-stonith-tutorial/availability-group-joined.png)

### <a name="add-a-database-to-the-availability-group"></a>Agregar una base de datos al grupo de disponibilidad

Seguiremos el [artículo sobre la configuración del grupo de disponibilidad para agregar una base de datos](/sql/linux/sql-server-linux-availability-group-configure-ha#add-a-database-to-the-availability-group).

Se utilizan los siguientes comandos de Transact-SQL en este paso. Ejecute estos comandos en la réplica principal:

```sql
CREATE DATABASE [db1]; -- creates a database named db1
GO

ALTER DATABASE [db1] SET RECOVERY FULL; -- set the database in full recovery mode
GO

BACKUP DATABASE [db1] -- backs up the database to disk
   TO DISK = N'/var/opt/mssql/data/db1.bak';
GO

ALTER AVAILABILITY GROUP [ag1] ADD DATABASE [db1]; -- adds the database db1 to the AG
GO
```

### <a name="verify-that-the-database-is-created-on-the-secondary-servers"></a>Compruebe que la base de datos se crea en los servidores secundarios.

En todas las réplicas secundarias de SQL Server, ejecute la consulta siguiente para ver si se ha creado la base de datos db1 y su estado es SINCRONIZADO:

```
SELECT * FROM sys.databases WHERE name = 'db1';
GO
SELECT DB_NAME(database_id) AS 'database', synchronization_state_desc FROM sys.dm_hadr_database_replica_states;
```

Si `synchronization_state_desc` muestra SINCRONIZADO para `db1`, esto significa que las réplicas están sincronizadas. En las secundarias aparece `db1` en la réplica principal.

## <a name="create-availability-group-resources-in-the-pacemaker-cluster"></a>Creación de recursos de grupo de disponibilidad en el clúster de Pacemaker

Vamos a seguir la guía para [crear los recursos del grupo de disponibilidad en el clúster de Pacemaker](/sql/linux/sql-server-linux-create-availability-group#create-the-availability-group-resources-in-the-pacemaker-cluster-external-only).

### <a name="create-the-ag-cluster-resource"></a>Creación de un recurso de clúster del grupo de disponibilidad

1. Use uno de los comandos siguientes en función del entorno elegido anteriormente para crear el recurso `ag_cluster` en el grupo de disponibilidad `ag1`.

    **RHEL 7**
  
    ```bash
    sudo pcs resource create ag_cluster ocf:mssql:ag ag_name=ag1 meta failure-timeout=30s master notify=true
    ```

    **RHEL 8**
  
    ```bash
    sudo pcs resource create ag_cluster ocf:mssql:ag ag_name=ag1 meta failure-timeout=30s promotable notify=true
    ```

2. Compruebe el recurso y asegúrese de que está en línea antes de continuar mediante el siguiente comando:

    ```bash
    sudo pcs resource
    ```

    Debería ver la siguiente salida:
    
    **RHEL 7** 
    
    ```output
    [<username>@VM1 ~]$ sudo pcs resource
    Master/Slave Set: ag_cluster-master [ag_cluster]
    Masters: [ <VM1> ]
    Slaves: [ <VM2> <VM3> ]
    ```
    
    **RHEL 8** 
    
    ```output
    [<username>@VM1 ~]$ sudo pcs resource
    * Clone Set: ag_cluster-clone [ag_cluster] (promotable):
    * ag_cluster             (ocf::mssql:ag) :            Slave VMrhel3 (Monitoring) 
    * ag_cluster             (ocf::mssql:ag) :            Master VMrhel1 (Monitoring)
    * ag_cluster             (ocf::mssql:ag) :            Slave VMrhel2 (Monitoring)
    ```


### <a name="create-a-virtual-ip-resource"></a>Creación de un recurso de dirección IP virtual

1. Use una dirección IP estática disponible desde la red para crear un recurso de IP virtual. Puede encontrar una mediante la herramienta de comandos `nmap`.

    ```bash
    nmap -sP <IPRange>
    # For example: nmap -sP 10.0.0.*
    # The above will scan for all IP addresses that are already occupied in the 10.0.0.x space.
    ```

2. Establezca la propiedad **stonith-enabled** en false.

    ```bash
    sudo pcs property set stonith-enabled=false
    ```

3. Cree el recurso de IP virtual mediante el siguiente comando:

    - Reemplace el siguiente valor de `<availableIP>` por una dirección IP no utilizada.

    ```bash
    sudo pcs resource create virtualip ocf:heartbeat:IPaddr2 ip=<availableIP>
    ```

### <a name="add-constraints"></a>Incorporación de restricciones

1. Para asegurarse de que la dirección IP y el recurso de grupo de disponibilidad se están ejecutando en el mismo nodo, debe configurarse una restricción de ubicación. Ejecute el siguiente comando:

   **RHEL 7**
  
    ```bash
    sudo pcs constraint colocation add virtualip ag_cluster-master INFINITY with-rsc-role=Master
    ```

   **RHEL 8**
   
    ```bash
     sudo pcs constraint colocation add virtualip with master ag_cluster-clone INFINITY with-rsc-role=Master
    ```
  
2. Cree una restricción de orden para asegurarse de que el recurso de grupo de disponibilidad está en funcionamiento antes que la dirección IP. Aunque la restricción de ubicación implica una restricción de orden, esto la aplica.

   **RHEL 7**
   
    ```bash
    sudo pcs constraint order promote ag_cluster-master then start virtualip
    ```

   **RHEL 8**
   
    ```bash
    sudo pcs constraint order promote ag_cluster-clone then start virtualip
    ```
  
3. Para comprobar las restricciones, ejecute el siguiente comando:

    ```bash
    sudo pcs constraint list --full
    ```

    Debería ver la siguiente salida:
    
    **RHEL 7**

    ```
    Location Constraints:
    Ordering Constraints:
          promote ag_cluster-master then start virtualip (kind:Mandatory) (id:order-ag_cluster-master-virtualip-mandatory)
    Colocation Constraints:
          virtualip with ag_cluster-master (score:INFINITY) (with-rsc-role:Master) (id:colocation-virtualip-ag_cluster-master-INFINITY)
    Ticket Constraints:
    ```
    
    **RHEL 8**
    
    ```output
    Location Constraints:
    Ordering Constraints:
            promote ag_cluster-clone then start virtualip (kind:Mandatory) (id:order-ag_cluster-clone-virtualip-mandatory)
    Colocation Constraints:
            virtualip with ag_cluster-clone (score:INFINITY) (with-rsc-role:Master) (id:colocation-virtualip-ag_cluster-clone-INFINITY)
    Ticket Constraints:
    ```

### <a name="re-enable-stonith"></a>Volver a habilitar stonith

Estamos preparados para realizar las pruebas. Vuelva a habilitar stonith en el clúster mediante la ejecución del siguiente comando en el nodo 1:

```bash
sudo pcs property set stonith-enabled=true
```

### <a name="check-cluster-status"></a>Comprobar el estado del clúster

Puede comprobar el estado de los recursos de clúster con el siguiente comando:

```output
[<username>@VM1 ~]$ sudo pcs status
Cluster name: az-hacluster
Stack: corosync
Current DC: <VM3> (version 1.1.19-8.el7_6.5-c3c624ea3d) - partition with quorum
Last updated: Sat Dec  7 00:18:38 2019
Last change: Sat Dec  7 00:18:02 2019 by root via cibadmin on VM1

3 nodes configured
5 resources configured

Online: [ <VM1> <VM2> <VM3> ]

Full list of resources:

 Master/Slave Set: ag_cluster-master [ag_cluster]
     Masters: [ <VM2> ]
     Slaves: [ <VM1> <VM3> ]
 virtualip      (ocf::heartbeat:IPaddr2):       Started <VM2>
 rsc_st_azure   (stonith:fence_azure_arm):      Started <VM1>

Daemon Status:
  corosync: active/enabled
  pacemaker: active/enabled
  pcsd: active/enabled
```

## <a name="test-failover"></a>Conmutación por error de prueba

Para asegurarse de que la configuración se ha realizado correctamente, se probará una conmutación por error. Para más información, consulte [Conmutación por error del grupo de disponibilidad Always On en Linux](/sql/linux/sql-server-linux-availability-group-failover-ha).

1. Ejecute el siguiente comando para realizar manualmente la conmutación por error de la réplica principal en `<VM2>`. Reemplace `<VM2>` por el valor del nombre del servidor.

   **RHEL 7**
   
    ```bash
    sudo pcs resource move ag_cluster-master <VM2> --master
    ```

   **RHEL 8**
   
    ```bash
    sudo pcs resource move ag_cluster-clone <VM2> --master
    ```

2. Si vuelve a comprobar las restricciones, verá que se ha agregado otra restricción debido a la conmutación por error manual:
    
    **RHEL 7**
    
    ```output
    [<username>@VM1 ~]$ sudo pcs constraint list --full
    Location Constraints:
          Resource: ag_cluster-master
            Enabled on: VM2 (score:INFINITY) (role: Master) (id:cli-prefer-ag_cluster-master)
    Ordering Constraints:
            promote ag_cluster-master then start virtualip (kind:Mandatory) (id:order-ag_cluster-master-virtualip-mandatory)
    Colocation Constraints:
            virtualip with ag_cluster-master (score:INFINITY) (with-rsc-role:Master) (id:colocation-virtualip-ag_cluster-master-INFINITY)
    Ticket Constraints:
    ```

    **RHEL 8**
    
    ```output
    [<username>@VM1 ~]$ sudo pcs constraint list --full
    Location Constraints:
          Resource: ag_cluster-master
            Enabled on: VM2 (score:INFINITY) (role: Master) (id:cli-prefer-ag_cluster-clone)
    Ordering Constraints:
            promote ag_cluster-clone then start virtualip (kind:Mandatory) (id:order-ag_cluster-clone-virtualip-mandatory)
    Colocation Constraints:
            virtualip with ag_cluster-clone (score:INFINITY) (with-rsc-role:Master) (id:colocation-virtualip-ag_cluster-clone-INFINITY)
    Ticket Constraints:
    ```
    
3. Para quitar la restricción con el identificador `cli-prefer-ag_cluster-master`, ejecute el comando siguiente:

    **RHEL 7**
    
    ```bash
    sudo pcs constraint remove cli-prefer-ag_cluster-master
    ```

    **RHEL 8**
    
    ```bash
    sudo pcs constraint remove cli-prefer-ag_cluster-clone
    ```

1. Compruebe los recursos del clúster mediante el comando `sudo pcs resource` y observará que la instancia principal es ahora `<VM2>`.

    ```output
    [<username>@<VM1> ~]$ sudo pcs resource
    Master/Slave Set: ag_cluster-master [ag_cluster]
         ag_cluster (ocf::mssql:ag):        FAILED <VM1> (Monitoring)
         Masters: [ <VM2> ]
         Slaves: [ <VM3> ]
    virtualip      (ocf::heartbeat:IPaddr2):       Started <VM2>
    [<username>@<VM1> ~]$ sudo pcs resource
    Master/Slave Set: ag_cluster-master [ag_cluster]
         Masters: [ <VM2> ]
         Slaves: [ <VM1> <VM3> ]
    virtualip      (ocf::heartbeat:IPaddr2):       Started <VM2>
    ```

## <a name="test-fencing"></a>Prueba de las barreras

Puede probar STONITH. Para ello, ejecute el siguiente comando. Intente ejecutar el comando siguiente desde `<VM1>` para `<VM3>`.

```bash
sudo pcs stonith fence <VM3> --debug
```

> [!NOTE]
> De forma predeterminada, la acción de barrera desconecta el nodo y luego lo conecta. Si solo desea desconectar el nodo, use la opción `--off` del comando.

Debería obtener la siguiente salida:

```output
[<username>@<VM1> ~]$ sudo pcs stonith fence <VM3> --debug
Running: stonith_admin -B <VM3>
Return Value: 0
--Debug Output Start--
--Debug Output End--
 
Node: <VM3> fenced
```
Para más información acerca de cómo probar un dispositivo de barrera, consulte el siguiente artículo de [Red Hat](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_reference/s1-stonithtest-haar).

## <a name="next-steps"></a>Pasos siguientes

Para utilizar una escucha de grupo de disponibilidad para las instancias de SQL Server, necesitará crear y configurar un equilibrador de carga.

> [!div class="nextstepaction"]
> [Tutorial: Configuración de una escucha de grupo de disponibilidad para SQL Server en máquinas virtuales con Red Hat Enterprise Linux en Azure](rhel-high-availability-listener-tutorial.md)
