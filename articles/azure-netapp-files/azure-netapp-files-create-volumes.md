---
title: Creación de un volumen NFS para Azure NetApp Files | Microsoft Docs
description: En este artículo se muestra cómo crear un volumen NFS en Azure NetApp Files. Obtenga información sobre consideraciones como la versión que usar, además de procedimientos recomendados.
services: azure-netapp-files
documentationcenter: ''
author: b-juche
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 07/27/2020
ms.author: b-juche
ms.openlocfilehash: f176e8fceb4d3e2e07398e6cb878180c8fe2321b
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87533162"
---
# <a name="create-an-nfs-volume-for-azure-netapp-files"></a>Creación de un volumen de NFS para Azure NetApp Files

Azure NetApp Files admite la creación de volúmenes con NFS (NFSv3 y NFSv4.1), SMBv3 o el protocolo dual (NFSv3 y SMB). El consumo de la capacidad de un volumen se descuenta de la capacidad aprovisionada de su grupo. En este artículo se muestra cómo crear un volumen NFS. 

## <a name="before-you-begin"></a>Antes de empezar 
* Debe haber configurado un grupo de capacidad.  
    Consulte [Configuración de un grupo de capacidad](azure-netapp-files-set-up-capacity-pool.md).   
* Debe haber una subred delegada en Azure NetApp Files.  
    Consulte [Delegación de una subred en Azure NetApp Files](azure-netapp-files-delegate-subnet.md).

## <a name="considerations"></a>Consideraciones 

* Elección de la versión de NFS que desea usar  
  NFSv3 puede administrar una amplia variedad de casos de uso y se implementa normalmente en la mayoría de las aplicaciones empresariales. Debe validar qué versión (NFSv3 o NFSv4.1) requiere la aplicación y crear el volumen mediante la versión adecuada. Por ejemplo, si usa [Apache ActiveMQ](https://activemq.apache.org/shared-file-system-master-slave), se recomienda el bloqueo de archivos con NFSv4.1 en lugar de con NFSv3. 

* Seguridad  
  La compatibilidad con los bits de modo UNIX (lectura, escritura y ejecución) está disponible para NFSv3 y NFSv4.1. Se necesita acceso de nivel de raíz en el cliente NFS para montar volúmenes NFS.

* Usuario o grupo local y compatibilidad con LDAP para NFSv4.1  
  Actualmente, NFSv4.1 solo admite el acceso raíz a volúmenes. Consulte [Configuración del dominio predeterminado de NFS, versión 4.1, para Azure NetApp Files](azure-netapp-files-configure-nfsv41-domain.md). 

## <a name="best-practice"></a>Procedimiento recomendado

* Debe asegurarse de seguir las instrucciones de montaje adecuadas para el volumen.  Consulte [Montaje o desmontaje de un volumen para máquinas virtuales Windows o Linux](azure-netapp-files-mount-unmount-volumes-for-virtual-machines.md).

* El cliente NFS debe estar en la misma red virtual o red virtual emparejada que el volumen de Azure NetApp Files. Se admite la conexión desde fuera de la red virtual; sin embargo, esto agregará latencia adicional y disminuirá el rendimiento general.

* Asegúrese de que el cliente NFS esté actualizado y ejecute las actualizaciones más recientes del sistema operativo.

## <a name="create-an-nfs-volume"></a>Creación de un volumen NFS

1.  Haga clic en la hoja **Volúmenes** desde la hoja Grupos de capacidad. Haga clic en **+ Agregar volumen** para crear un volumen. 

    ![Vaya a Volúmenes](../media/azure-netapp-files/azure-netapp-files-navigate-to-volumes.png) 

2.  En la ventana Crear un volumen, haga clic en **Crear** y proporcione la información para los campos siguientes bajo la pestaña Aspectos básicos:   
    * **Nombre del volumen**      
        Especifique el nombre para el volumen que va a crear.   

        Un nombre de volumen debe ser único dentro de cada grupo de capacidad. Debe tener tres caracteres de longitud, como mínimo. Puede usar cualquier carácter alfanumérico.   

        No se puede usar `default` como nombre del volumen.

    * **Grupo de capacidad**  
        Especifique el grupo de capacidad en el que desee que el volumen se cree.

    * **Cuota**  
        Especifique la cantidad de almacenamiento lógico que se asigna al volumen.  

        El campo **Cuota disponible** muestra la cantidad de espacio no utilizado en el grupo de capacidad elegido que puede usar para crear un nuevo volumen. El tamaño del volumen nuevo no debe superar la cuota disponible.  

    * **Red virtual**  
        Especifique la red virtual de Azure desde la que desea tener acceso al volumen.  

        La red virtual que especifique debe tener una subred delegada en Azure NetApp Files. Solo puede tener acceso al servicio Azure NetApp Files desde la misma red virtual o desde una red virtual que se encuentre en la misma ubicación que el volumen mediante el emparejamiento de redes virtuales. También puede acceder al volumen desde la red local mediante ExpressRoute.   

    * **Subred**  
        Especifique la subred que desea usar para el volumen.  
        La red virtual que especifique debe estar delegada en Azure NetApp Files. 
        
        Si no ha delegado una subred, haga clic en **Crear nuevo** en el volumen de creación de un volumen. A continuación, en la página de creación de la subred, especifique la información de la subred y seleccione **Microsoft.NetApp/volumes** para delegar la subred para Azure NetApp Files. En cada red virtual, solo puede delegarse una subred a Azure NetApp Files.   
 
        ![Crear un volumen](../media/azure-netapp-files/azure-netapp-files-new-volume.png)
    
        ![Creación de una subred](../media/azure-netapp-files/azure-netapp-files-create-subnet.png)

    * Si quiere aplicar una directiva de instantáneas existente al volumen, haga clic en **Mostrar la sección avanzada** para expandirla y seleccione una directiva de instantáneas en el menú desplegable. 

        Para obtener información sobre cómo crear una directiva de instantáneas, consulte [Administración de directivas de instantánea](azure-netapp-files-manage-snapshots.md#manage-snapshot-policies).

        ![Selección de Mostrar la sección avanzada](../media/azure-netapp-files/volume-create-advanced-selection.png)

3. Haga clic en **Protocolo** y realice las siguientes acciones:  
    * Seleccione **NFS** como tipo de protocolo para el volumen.   
    * Especifique la **ruta de acceso de archivo** que se usará para crear la ruta de acceso de exportación para el nuevo volumen. La ruta de acceso de exportación se usa para montar el volumen y tener acceso a él.

        El nombre de la ruta de acceso de archivo solo puede contener letras, números y guiones ("-"). El nombre debe tener entre 16 y 40 caracteres. 

        La ruta de acceso del archivo debe ser única dentro de cada suscripción y cada región. 

    * Seleccione la versión de NFS (**NFSv3** o **NFSv4.1**) del volumen.  

    * Si usa NFSv4.1, indique si desea habilitar el cifrado de **Kerberos** para el volumen.  

        Si usa Kerberos con NFSv4.1, se requieren configuraciones adicionales. Siga las instrucciones de [Configuración del cifrado de Kerberos NFSv4.1](configure-kerberos-encryption.md).

    * Si lo desea, [configure la directiva de exportación para un volumen NFS](azure-netapp-files-configure-export-policy.md).

    ![Especificación del protocolo NFS](../media/azure-netapp-files/azure-netapp-files-protocol-nfs.png)

4. Haga clic en **Revisar y crear** para revisar los detalles del volumen.  Después, haga clic en **Crear** para crear el volumen.

    El volumen que creó aparece en la hoja Volúmenes. 
 
    Un volumen hereda los atributos de ubicación, la suscripción y el grupo de recursos de su grupo de capacidad. Para supervisar el estado de la implementación del volumen, puede usar la pestaña Notificaciones.


## <a name="next-steps"></a>Pasos siguientes  

* [Configuración del dominio predeterminado de NFS, versión 4.1, para Azure NetApp Files](azure-netapp-files-configure-nfsv41-domain.md)
* [Configuración del cifrado Kerberos de NFSv4.1](configure-kerberos-encryption.md)
* [Montaje o desmontaje de un volumen para máquinas virtuales Windows o Linux](azure-netapp-files-mount-unmount-volumes-for-virtual-machines.md)
* [Configuración de la directiva de exportación para un volumen NFS](azure-netapp-files-configure-export-policy.md)
* [Límites de recursos para Azure NetApp Files](azure-netapp-files-resource-limits.md)
* [Obtenga información sobre la integración de redes virtuales para los servicios de Azure](https://docs.microsoft.com/azure/virtual-network/virtual-network-for-azure-services)
