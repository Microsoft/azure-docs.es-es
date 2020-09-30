---
title: Persistencia de archivos en Azure Cloud Shell | Microsoft Docs
description: Tutorial de cómo Azure Cloud Shell persiste archivos.
services: azure
documentationcenter: ''
author: maertendMSFT
manager: timlt
tags: azure-resource-manager
ms.assetid: ''
ms.service: azure
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 02/24/2020
ms.author: damaerte
ms.openlocfilehash: 16345ae479be70ffb1eaae95196a43ec99ca1586
ms.sourcegitcommit: 4feb198becb7a6ff9e6b42be9185e07539022f17
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89470143"
---
# <a name="persist-files-in-azure-cloud-shell"></a>Persistencia de archivos en Azure Cloud Shell
Cloud Shell utiliza Azure File Storage para conservar los archivos entre sesiones. En el primer inicio, Cloud Shell le pedirá que asocie un recurso compartido de archivos nuevo o existente para conservar los archivos entre sesiones.

> [!NOTE]
> Bash y PowerShell comparten el mismo recurso compartido de archivos. Solo un recurso compartido de archivos se puede asociar con el montaje automático en Cloud Shell.

> [!NOTE]
> El firewall de Azure Storage no es compatible con las cuentas de almacenamiento de Cloud Shell.

## <a name="create-new-storage"></a>creación de nuevo almacenamiento

Al usar la configuración básica y seleccionar solo una suscripción, Cloud Shell crea tres recursos en su nombre en la región admitida que esté más próxima:
* Grupos de recursos: `cloud-shell-storage-<region>`
* Cuenta de almacenamiento: `cs<uniqueGuid>`
* Recurso compartido de archivos: `cs-<user>-<domain>-com-<uniqueGuid>`

![La configuración de la suscripción](media/persisting-shell-storage/basic-storage.png)

El recurso compartido de archivos se monta como `clouddrive` en su directorio `$Home`. Se trata de una acción única y el recurso compartido de archivos se monta automáticamente en sesiones posteriores. 

En Bash, el recurso compartido de archivos también contiene una imagen de 5 GB que se crea automáticamente que conserva automáticamente los datos en su directorio `$Home`. Esto se aplica para Bash y PowerShell.

## <a name="use-existing-resources"></a>Uso de recursos existentes

Mediante la opción avanzada puede asociar recursos existentes. Al seleccionar una región de Cloud Shell, debe seleccionar una cuenta de almacenamiento de respaldo coubicada en la misma región. Por ejemplo, si la región asignada es Oeste de EE. UU., debe asociar un recurso compartido de archivos que resida también en Oeste de EE. UU.

Cuando aparezca el mensaje del programa de instalación, seleccione **Mostrar configuración avanzada** para ver otras opciones. Las opciones de almacenamiento rellenadas se filtran por cuentas de almacenamiento con redundancia local (LRS), almacenamiento con redundancia geográfica (GRS) y almacenamiento con redundancia de zona (ZRS). 

> [!NOTE]
> Se recomienda usar cuentas de almacenamiento con redundancia geográfica o con redundancia de zona para contar con resistencia adicional para el recurso compartido de archivos de respaldo. El tipo de redundancia depende de los objetivos y las preferencias de precio. [Obtenga más información sobre las opciones de replicación para las cuentas de Azure Storage](../storage/common/storage-redundancy.md).

![La configuración de grupo de recursos](media/persisting-shell-storage/advanced-storage.png)

## <a name="securing-storage-access"></a>Protección del acceso al almacenamiento
Por motivos de seguridad, cada usuario debe aprovisionar su propia cuenta de almacenamiento.  En el control de acceso basado en roles (RBAC), los usuarios deben tener acceso como colaboradores, o un rol superior, en el nivel de la cuenta de almacenamiento.

Cloud Shell usa un recurso compartido de archivos de Azure en una cuenta de almacenamiento, dentro de una suscripción especificada. Debido a los permisos heredados, los usuarios con derechos de acceso suficientes en la suscripción podrán acceder a todas las cuentas de almacenamiento y a los recursos compartidos de archivos contenidos en la suscripción.

Los usuarios deben bloquear el acceso a sus archivos estableciendo los permisos en el nivel de cuenta de almacenamiento o de suscripción.

## <a name="supported-storage-regions"></a>Regiones de almacenamiento admitidas
Para encontrar la región actual puede ejecutar `env` en Bash y buscar la variable `ACC_LOCATION`, o bien ejecutar `$env:ACC_LOCATION` desde PowerShell. Los recursos compartidos de archivos existentes reciben una imagen de 5 GB para que el usuario conserve el directorio `$Home`.

Las máquinas de Cloud Shell existen en las regiones siguientes:

|Área|Region|
|---|---|
|América|Este de EE. UU., centro-sur de EE. UU. y oeste de EE. UU.|
|Europa|Norte de Europa y Oeste de Europa|
|Asia Pacífico|Sur de la India, Sudeste de Asia|

Los clientes deben elegir una región primaria, a menos que tengan un requisito de que sus datos en reposo se almacenen en una región determinada. Si existe este requisito, se debe usar una región de almacenamiento secundaria.

### <a name="secondary-storage-regions"></a>Regiones de almacenamiento secundarias
Si se usa una región de almacenamiento secundaria, la cuenta de almacenamiento de Azure asociada se encuentra en una región distinta a la de la máquina de Cloud Shell donde se va a montar. Por ejemplo, Julia puede configurar su cuenta de almacenamiento para que se encuentre en la región Este de Canadá, una región secundaria, pero la máquina en la que se montó todavía se encuentra en una región primaria. Sus datos en reposo se encuentran en Canadá, pero se procesan en los Estados Unidos.

> [!NOTE]
> Si se usa una región secundaria, el acceso a archivos y el tiempo de inicio de Cloud Shell pueden ser más lentos.

Un usuario puede ejecutar `(Get-CloudDrive | Get-AzStorageAccount).Location` en PowerShell para ver la ubicación de su recurso compartido de archivos.

## <a name="restrict-resource-creation-with-an-azure-resource-policy"></a>Restringir la creación de recursos con una directiva de recursos de Azure
Las cuentas de almacenamiento creadas en Cloud Shell se etiquetan con `ms-resource-usage:azure-cloud-shell`. Si quiere impedir que los usuarios creen cuentas de almacenamiento en Cloud Shell, cree una [directiva de recursos de Azure para etiquetas](../governance/policy/samples/index.md) que se desencadene mediante esta etiqueta específica.

## <a name="how-cloud-shell-storage-works"></a>Funcionamiento del almacenamiento de Cloud Shell 
Cloud Shell conserva los archivos a través de los métodos siguientes: 
* Creación de una imagen de disco del directorio `$Home` para hacer persistente todo el contenido dentro del directorio. La imagen de disco se guarda en su recurso compartido de archivos especificado como `acc_<User>.img` en `fileshare.storage.windows.net/fileshare/.cloudconsole/acc_<User>.img` y sincroniza automáticamente los cambios. 
* Montaje del recurso compartido de archivos especificado como `clouddrive` en el directorio `$Home` para la interacción directa del recurso compartido de archivos. `/Home/<User>/clouddrive` se asigna a `fileshare.storage.windows.net/fileshare`.
 
> [!NOTE]
> Todos los archivos en el directorio `$Home`, como las claves de SSH, se conservan en la imagen de disco de usuario almacenada en el recurso compartido de archivos montado. Ponga en práctica los procedimientos recomendados correspondientes para conservar la información en el directorio `$Home` y en el recurso compartido de archivos montado.

## <a name="clouddrive-commands"></a>comandos de clouddrive

### <a name="use-the-clouddrive-command"></a>Uso del comando `clouddrive`
En Cloud Shell, puede ejecutar un comando denominado `clouddrive`, lo que permite actualizar manualmente el recurso compartido de archivos que está montado en Cloud Shell.

![Ejecución del comando "clouddrive"](media/persisting-shell-storage/clouddrive-h.png)

### <a name="list-clouddrive"></a>Enumeración de `clouddrive`
Para descubrir qué recurso compartido de archivos se monta como `clouddrive` ejecute el comando `df`. 

La ruta de acceso de archivo a clouddrive mostrará el recurso compartido de archivos y el nombre de la cuenta de almacenamiento en la dirección URL. Por ejemplo: `//storageaccountname.file.core.windows.net/filesharename`

```
justin@Azure:~$ df
Filesystem                                          1K-blocks   Used  Available Use% Mounted on
overlay                                             29711408 5577940   24117084  19% /
tmpfs                                                 986716       0     986716   0% /dev
tmpfs                                                 986716       0     986716   0% /sys/fs/cgroup
/dev/sda1                                           29711408 5577940   24117084  19% /etc/hosts
shm                                                    65536       0      65536   0% /dev/shm
//mystoragename.file.core.windows.net/fileshareName 5368709120    64 5368709056   1% /home/justin/clouddrive
justin@Azure:~$
```

### <a name="mount-a-new-clouddrive"></a>Montaje de un nuevo clouddrive

#### <a name="prerequisites-for-manual-mounting"></a>Requisitos previos para el montaje manual
Puede actualizar el recurso compartido de archivos que esté asociada con Cloud Shell mediante el uso del comando `clouddrive mount`.

Si se monta un recurso compartido de archivos existente, las cuentas de almacenamiento han de encontrarse en la región de Cloud Shell seleccionada. Para recuperar la ubicación, ejecute `env` y compruebe la `ACC_LOCATION`.

#### <a name="the-clouddrive-mount-command"></a>El comando `clouddrive mount`

> [!NOTE]
> Si monta un recurso compartido de archivos nuevo, se crea una imagen de usuario para el directorio `$Home`. La imagen `$Home` anterior se mantiene en el recurso compartido de archivos anterior.

Ejecute el comando `clouddrive mount` con los parámetros siguientes:

```
clouddrive mount -s mySubscription -g myRG -n storageAccountName -f fileShareName
```

Para ver más detalles, ejecute `clouddrive mount -h`, tal y como se muestra aquí:

![Ejecución del comando "clouddrive"](media/persisting-shell-storage/mount-h.png)

### <a name="unmount-clouddrive"></a>Desmontaje de clouddrive
Puede desmontar un recurso compartido de archivos montado en Cloud Shell en cualquier momento. Como Cloud Shell requiere que se use un recurso compartido de archivos, se le pedirá crear y montar otro recurso compartido de archivos en la sesión siguiente.

1. Ejecute `clouddrive unmount`.
2. Reconozca y confirme los mensajes.

El recurso compartido de archivos seguirá existiendo a menos que se elimine manualmente. Cloud Shell dejará de buscar este recurso compartido de archivos en sesiones posteriores. Para ver más detalles, ejecute `clouddrive unmount -h`, tal y como se muestra aquí:

![Ejecución del comando "clouddrive amount"](media/persisting-shell-storage/unmount-h.png)

> [!WARNING]
> Aunque la ejecución de este comando no elimina ningún recurso, al eliminar manualmente el grupo de recursos, la cuenta de almacenamiento o el recurso compartido de archivos asignados a Cloud Shell, se borra la imagen de disco del directorio `$Home` y los archivos del recurso compartido de archivos. Esta operación no se puede deshacer.
## <a name="powershell-specific-commands"></a>Comandos específicos de PowerShell

### <a name="list-clouddrive-azure-file-shares"></a>Lista de los recursos compartidos de archivos de Azure en `clouddrive`
El cmdlet `Get-CloudDrive` recupera la información del recurso compartido de archivos de Azure que monta `clouddrive` actualmente en Cloud Shell. <br>
![Ejecución de Get-CloudDrive](media/persisting-shell-storage-powershell/Get-Clouddrive.png)

### <a name="unmount-clouddrive"></a>Desmontar `clouddrive`
Puede desmontar un recurso compartido de archivos de Azure montado en Cloud Shell en cualquier momento. Si se quitó el recurso compartido de archivos de Azure, se le pedirá crear y montar uno nuevo en la sesión siguiente.

El cmdlet `Dismount-CloudDrive` desmonta un recurso compartido de archivos de Azure desde la cuenta de almacenamiento actual. Desmontar `clouddrive` finaliza la sesión actual. Se le pedirá al usuario que cree y monte un recurso compartido de archivos de Azure nuevo durante la sesión siguiente.
![Ejecución de Dismount-CloudDrive](media/persisting-shell-storage-powershell/Dismount-Clouddrive.png)

[!INCLUDE [PersistingStorage-endblock](../../includes/cloud-shell-persisting-shell-storage-endblock.md)]

Nota: si tiene que definir una función en un archivo y llamarla desde los cmdlets de PowerShell, debe incluir el operador punto. Por ejemplo: . .\MyFunctions.ps1

## <a name="next-steps"></a>Pasos siguientes
[Inicio rápido de Cloud Shell](quickstart.md) <br>
[Información sobre el almacenamiento de Microsoft Azure Files](../storage/files/storage-files-introduction.md) <br>
[Información sobre las etiquetas de almacenamiento](../azure-resource-manager/management/tag-resources.md) <br>