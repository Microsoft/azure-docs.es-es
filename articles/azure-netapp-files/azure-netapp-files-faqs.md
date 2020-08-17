---
title: Preguntas más frecuentes acerca de Azure NetApp Files | Microsoft Docs
description: Revise las preguntas más frecuentes sobre Azure NetApp Files, como redes, seguridad, rendimiento, administración de capacidad y migración o protección de datos.
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
ms.topic: conceptual
ms.date: 07/27/2020
ms.author: b-juche
ms.openlocfilehash: 7c792ee9c56a044942bb2249a57f2615c72badee
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87533145"
---
# <a name="faqs-about-azure-netapp-files"></a>Preguntas más frecuentes acerca de Azure NetApp Files

En este artículo se responden algunas preguntas frecuentes sobre Azure NetApp Files. 

## <a name="networking-faqs"></a>Preguntas frecuentes sobre redes

### <a name="does-the-nfs-data-path-go-over-the-internet"></a>¿La ruta de acceso de datos de NFS pasa a través de Internet?  

No. La ruta de acceso de NFS no pasa por Internet. Azure NetApp Files es un servicio nativo de Azure que está implementado en Azure Virtual Network (VNet) en donde el servicio está disponible. Azure NetApp Files usa una subred delegada y aprovisiona una interfaz de red directamente en la red virtual (VNet). 

Consulte [Directrices para el planeamiento de red de Azure NetApp Files](https://docs.microsoft.com/azure/azure-netapp-files/azure-netapp-files-network-topologies) para una información más detallada.  

### <a name="can-i-connect-a-vnet-that-i-already-created-to-the-azure-netapp-files-service"></a>¿Puedo conectar una red virtual que ya haya creado con el servicio Azure NetApp Files?

Sí, puede conectar redes virtuales que haya creado al servicio. 

Consulte [Directrices para el planeamiento de red de Azure NetApp Files](https://docs.microsoft.com/azure/azure-netapp-files/azure-netapp-files-network-topologies) para una información más detallada.  

### <a name="can-i-mount-an-nfs-volume-of-azure-netapp-files-using-dns-fqdn-name"></a>¿Puedo montar un volumen NFS de Azure NetApp Files mediante el nombre FQDN de DNS?

Sí que puede, si crea las entradas DNS necesarias. Azure NetApp Files proporciona la dirección IP de servicio para el volumen aprovisionado. 

> [!NOTE] 
> Azure NetApp Files puede implementar direcciones IP adicionales para el servicio según sea necesario.  Puede que sea necesario actualizar las entradas DNS periódicamente.

### <a name="can-i-set-or-select-my-own-ip-address-for-an-azure-netapp-files-volume"></a>¿Puedo establecer o seleccionar mi propia dirección IP para un volumen de Azure NetApp Files?  

No. La asignación de direcciones IP a volúmenes de Azure NetApp Files es dinámica. No se admiten las asignaciones de IP estáticas. 
 
## <a name="security-faqs"></a>Preguntas más frecuentes de seguridad

### <a name="can-the-network-traffic-between-the-azure-vm-and-the-storage-be-encrypted"></a>¿Se puede cifrar el tráfico de red entre la máquina virtual de Azure y el almacenamiento?

El tráfico de datos (el tráfico desde el cliente de NFSv3, NFSv4.1 o SMBv3 a los volúmenes de Azure NetApp Files) no está cifrado. Sin embargo, el tráfico desde una máquina virtual de Azure (que ejecuta un cliente SMB o NFS) a Azure NetApp Files es tan seguro como cualquier otro tráfico de máquina virtual a máquina virtual de Azure. Se trata de tráfico local a la red del centro de datos de Azure. 

### <a name="can-the-storage-be-encrypted-at-rest"></a>¿Se puede cifrar el almacenamiento en reposo?

Todos los volúmenes de Azure NetApp Files se cifran mediante el estándar FIPS 140-2. El servicio Azure NetApp Files administra todas las claves. 

### <a name="how-are-encryption-keys-managed"></a>¿Cómo se administran las claves de cifrado? 

La administración de claves para Azure NetApp Files se controla mediante el servicio. Se genera una clave de cifrado de datos XTS-AES-256 única para cada volumen. Una jerarquía de claves de cifrado se usa para cifrar y proteger todas las claves de volumen. Estas claves de cifrado nunca se muestran ni se notifican en un formato sin cifrar. Las claves de cifrado se eliminan inmediatamente cuando se suprime un volumen.

La compatibilidad con las claves administradas por el usuario (Bring Your Own Keys) mediante Azure Dedicated HSM está disponible de manera controlada en las regiones Este de EE. UU., Oeste de EE. UU 2. y Centro y Sur de EE. UU.  Puede solicitar acceso en **anffeedback@microsoft.com** . A medida que haya capacidad disponible, se aprobarán las solicitudes.

### <a name="can-i-configure-the-nfs-export-policy-rules-to-control-access-to-the-azure-netapp-files-service-mount-target"></a>¿Puedo configurar las reglas de directivas de exportación NFS para controlar el acceso al destino de montaje del servicio Azure NetApp Files?


Sí, puede configurar hasta cinco reglas en una sola directiva de exportación NFS.

### <a name="does-azure-netapp-files-support-network-security-groups"></a>¿Azure NetApp Files admite grupos de seguridad de red?

No, actualmente no se pueden aplicar grupos de seguridad de red a la subred delegada de Azure NetApp Files o las interfaces de red creadas por el servicio.

### <a name="can-i-use-azure-iam-with-azure-netapp-files"></a>¿Puedo usar IAM de Azure con Azure NetApp Files?

Sí, Azure NetApp Files admite características RBAC con Azure IAM.

## <a name="performance-faqs"></a>Preguntas más frecuentes sobre rendimiento

### <a name="what-should-i-do-to-optimize-or-tune-azure-netapp-files-performance"></a>¿Qué debo hacer para optimizar o ajustar el rendimiento de Azure NetApp Files?

Puede realizar las siguientes acciones según los requisitos de rendimiento: 
- Asegúrese de que la máquina virtual tiene el tamaño apropiado.
- Habilite redes aceleradas para la máquina virtual.
- Seleccione el nivel de servicio deseado y el tamaño de grupo de capacidad.
- Cree un volumen con el tamaño de cuota deseada para la capacidad y el rendimiento.

### <a name="how-do-i-convert-throughput-based-service-levels-of-azure-netapp-files-to-iops"></a>¿Cómo puedo convertir los niveles de servicio basado en el rendimiento de Azure NetApp Files a IOPS?

Puede convertir MB/s a IOPS mediante el uso de la siguiente fórmula:  

`IOPS = (MBps Throughput / KB per IO) * 1024`

### <a name="how-do-i-change-the-service-level-of-a-volume"></a>¿Cómo puedo cambiar el nivel de servicio de un volumen?

Puede cambiar el nivel de servicio de un volumen existente al moverlo a otro grupo de capacidad que use el [nivel de servicio](azure-netapp-files-service-levels.md) que quiere para dicho volumen. Consulte [Cambio dinámico del nivel de servicio de un volumen](dynamic-change-volume-service-level.md). 

### <a name="how-do-i-monitor-azure-netapp-files-performance"></a>¿Cómo se puede supervisar el rendimiento de Azure NetApp Files?

Azure NetApp Files proporciona métricas de rendimiento del volumen. También puede usar Azure Monitor para supervisar las métricas de uso de Azure NetApp Files.  Consulte [Métricas de Azure NetApp Files](azure-netapp-files-metrics.md) para obtener la lista de métricas de rendimiento de Azure NetApp Files.

### <a name="whats-the-performance-impact-of-kerberos-on-nfsv41"></a>¿Cuál es el impacto en el rendimiento de Kerberos en NFSv4.1?

Consulte [Impacto en el rendimiento de Kerberos en NFSv4.1](configure-kerberos-encryption.md#kerberos_performance) para obtener información sobre las opciones de seguridad de NFSv4.1, los vectores de rendimiento que se hayan probado y el impacto esperado en el rendimiento. 

## <a name="nfs-faqs"></a>Preguntas más frecuentes sobre NFS

### <a name="i-want-to-have-a-volume-mounted-automatically-when-an-azure-vm-is-started-or-rebooted--how-do-i-configure-my-host-for-persistent-nfs-volumes"></a>Quiero tener un volumen montado automáticamente cuando se inicia o se reinicia una máquina virtual de Azure.  ¿Cómo tengo que configurar mi host para los volúmenes persistentes de NFS?

Para que un volumen NFS se monte automáticamente al inicio o reinicio de una máquina virtual, agregue una entrada al archivo `/etc/fstab` en el host. 

Consulte [Montaje o desmontaje de un volumen para máquinas virtuales Windows o Linux](azure-netapp-files-mount-unmount-volumes-for-virtual-machines.md) para consultar los detalles.  

### <a name="why-does-the-df-command-on-nfs-client-not-show-the-provisioned-volume-size"></a>¿Por qué el comando DF en el cliente NFS no muestra el tamaño del volumen aprovisionado?

El tamaño del volumen notificado en DF es el tamaño máximo que puede alcanzar el volumen de Azure NetApp Files. El tamaño del volumen de Azure NetApp Files en el comando DF no es reflejo de la cuota o el tamaño del volumen.  Puede obtener el tamaño del volumen o la cuota de Azure NetApp Files mediante Azure Portal o la API.

### <a name="what-nfs-version-does-azure-netapp-files-support"></a>¿Qué versión de NFS admite Azure NetApp Files?

Azure NetApp Files admite NFSv3 y NFSv4.1. Se pueden [crear volúmenes](azure-netapp-files-create-volumes.md) con cualquier versión de NFS. 

### <a name="how-do-i-enable-root-squashing"></a>¿Cómo se puede habilitar root squashing?

Actualmente no se admite root squashing.

## <a name="smb-faqs"></a>Preguntas más frecuentes de SMB

### <a name="which-smb-versions-are-supported-by-azure-netapp-files"></a>¿Qué versiones de SMB son compatibles con Azure NetApp Files?

Azure NetApp Files admite SMB 2.1 y SMB 3.1 (que incluye compatibilidad con SMB 3.0).    

### <a name="is-an-active-directory-connection-required-for-smb-access"></a>¿Es necesaria una conexión de Active Directory para el acceso SMB? 

Sí, tiene que crear una conexión de Active Directory antes de implementar un volumen de SMB. Para que la conexión tenga éxito la subred delegada de Azure NetApp Files tiene que tener acceso a los controladores de dominio especificados.  Consulte [Crear un volumen de SMB](https://docs.microsoft.com/azure/azure-netapp-files/azure-netapp-files-create-volumes-smb) para más información. 

### <a name="how-many-active-directory-connections-are-supported"></a>¿Cuántas conexiones de Active Directory se admiten?

Azure NetApp Files no admite varias conexiones de Active Directory (AD) en una única *región*, incluso si las conexiones de AD se encuentran en distintas cuentas de NetApp. Sin embargo, puede tener varias conexiones de AD en una única *suscripción*, siempre que las conexiones de AD se encuentren en regiones diferentes. Si necesita varias conexiones de AD en una sola región, puede usar suscripciones independientes para ello. 

Se configura una conexión de AD por cuenta de NetApp; la conexión de AD solo es visible a través de la cuenta de NetApp en la que se crea.

### <a name="does-azure-netapp-files-support-azure-active-directory"></a>¿ Azure NetApp Files admite Azure Active Directory? 

Se admite tanto [Azure Active Directory (AD) Domain Services](https://docs.microsoft.com/azure/active-directory-domain-services/overview) como [Active Directory Domain Services (AD DS)](https://docs.microsoft.com/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview). Puede usar controladores de dominio de Active Directory existentes con Azure NetApp Files. Los controladores de dominio pueden residir en Azure como máquinas virtuales o en el entorno local mediante ExpressRoute o VPN S2S. En este momento, Azure NetApp Files no admite la unión a AD para [Azure Active Directory](https://azure.microsoft.com/resources/videos/azure-active-directory-overview/).

Si usa Azure NetApp Files con Azure Active Directory Domain Services, la ruta de acceso de la unidad organizativa es `OU=AADDC Computers` cuando configura Active Directory para su cuenta de NetApp.

### <a name="what-versions-of-windows-server-active-directory-are-supported"></a>¿Qué versiones de Windows Server Active Directory se admiten?

Azure NetApp Files admite las versiones 2008r2SP1-2019 de Windows Server de Active Directory Domain Services.

### <a name="why-does-the-available-space-on-my-smb-client-not-show-the-provisioned-size"></a>¿Por qué el espacio disponible en el cliente SMB no muestra el tamaño aprovisionado?

El tamaño de volumen que SMB indica es el tamaño máximo que puede alcanzar el volumen de Azure NetApp Files. El tamaño del volumen de Azure NetApp Files que se muestra en el cliente SMB no es reflejo de la cuota o el tamaño del volumen. Puede obtener el tamaño del volumen o la cuota de Azure NetApp Files mediante Azure Portal o la API.

<!--
### Does Azure NetApp Files support Kerberos encryption?

Yes, by default, Azure NetApp Files supports both AES-128 and AES-256 encryption for traffic between the service and the targeted Active Directory domain controllers. See [Create an SMB volume for Azure NetApp Files](azure-netapp-files-create-volumes-smb.md) for requirements. 
-->

<!--
### Does Azure NetApp Files support LDAP signing? 

Yes, Azure NetApp Files supports LDAP signing by default. This functionality enables secure LDAP lookups between the Azure NetApp Files service and the user-specified [Active Directory Domain Services domain controllers](https://docs.microsoft.com/windows/win32/ad/active-directory-domain-services). For more information, see [ADV190023 | Microsoft Guidance for Enabling LDAP Channel Binding and LDAP Signing](https://portal.msrc.microsoft.com/en-us/security-guidance/advisory/ADV190023).
--> 

## <a name="dual-protocol-faqs"></a>Preguntas más frecuentes sobre el protocolo dual

### <a name="i-tried-to-use-the-root-and-local-users-to-access-a-dual-protocol-volume-with-the-ntfs-security-style-on-a-unix-system-why-did-i-encounter-a-permission-denied-error"></a>Intenté usar los usuarios locales y "raíz" para obtener acceso a un volumen de protocolo dual con el estilo de seguridad NTFS en un sistema UNIX. ¿Por qué encontré un error de tipo "Permiso denegado"?   

Un volumen de protocolo dual admite los protocolos NFS y SMB.  Al intentar obtener acceso al volumen montado en el sistema UNIX, el sistema intenta asignar el usuario de UNIX que utiliza a un usuario de Windows. Si no se encuentra ninguna asignación, se produce el error "Permiso denegado".  Esta situación se aplica también cuando se usa el usuario "raíz" para obtener acceso.    

Para evitar el problema de tipo "Permiso denegado", asegúrese de que Active Directory para Windows incluya `pcuser` antes de obtener acceso al punto de montaje. Si agrega `pcuser` después de encontrar el problema "Permiso denegado", espere 24 horas para que la entrada de caché se borre antes de volver a intentar el acceso.


## <a name="capacity-management-faqs"></a>Preguntas más frecuentes sobre la administración de la capacidad

### <a name="how-do-i-monitor-usage-for-capacity-pool-and-volume-of-azure-netapp-files"></a>¿Cómo puedo supervisar el uso de grupo de capacidad y volumen de Azure NetApp Files? 

Azure NetApp Files proporciona métricas de uso de grupo de capacidad y volumen. También puede usar Azure Monitor para supervisar el uso de Azure NetApp Files. Consulte [Métricas de Azure NetApp Files](azure-netapp-files-metrics.md) para más información. 

### <a name="can-i-manage-azure-netapp-files-through-azure-storage-explorer"></a>¿Puedo administrar Azure NetApp Files mediante Explorador de Azure Storage?

No. Explorador de Azure Storage no es compatible con Azure NetApp Files.

### <a name="how-do-i-determine-if-a-directory-is-approaching-the-limit-size"></a>¿Cómo puedo determinar si un directorio está llegando al tamaño límite?

Puede usar el comando `stat` desde un cliente para ver si un directorio está llegando al límite de tamaño máximo de los metadatos del directorio (320 MB).

En un directorio de 320 MB, el número de bloques es 655 360 y el tamaño de cada bloque es de 512 bytes.  (Es decir, 320 x 1024 x 1024/512.)  

Ejemplos:

```console
[makam@cycrh6rtp07 ~]$ stat bin
File: 'bin'
Size: 4096            Blocks: 8          IO Block: 65536  directory

[makam@cycrh6rtp07 ~]$ stat tmp
File: 'tmp'
Size: 12288           Blocks: 24         IO Block: 65536  directory
 
[makam@cycrh6rtp07 ~]$ stat tmp1
File: 'tmp1'
Size: 4096            Blocks: 8          IO Block: 65536  directory
```


## <a name="data-migration-and-protection-faqs"></a>Preguntas frecuentes sobre migración y protección de datos

### <a name="how-do-i-migrate-data-to-azure-netapp-files"></a>¿Cómo se pueden migrar los datos a Azure NetApp Files?
Azure NetApp Files proporciona volúmenes SMB y NFS.  Puede usar cualquier herramienta de copia basada en archivos para migrar datos al servicio. 

NetApp ofrece una solución basada en SaaS, [NetApp Cloud Sync](https://cloud.netapp.com/cloud-sync-service).  La solución le permite replicar datos de NFS o SMB en los recursos compartidos de SMB o exportaciones de NFS de Azure NetApp Files. 

También puede utilizar una amplia gama de herramientas gratuitas para copiar los datos. Para NFS, puede usar las herramientas de cargas de trabajo, como [rsync](https://rsync.samba.org/examples.html) para copiar y sincronizar datos de origen en un volumen de Azure NetApp Files. Para SMB, puede usar las cargas de trabajo [robocopy](https://docs.microsoft.com/windows-server/administration/windows-commands/robocopy) de la misma manera.  Estas herramientas también pueden replicar los permisos de archivo o carpeta. 

Los requisitos para la migración de datos desde un entorno local a Azure NetApp Files son los siguientes: 

- Asegúrese de que Azure NetApp Files está disponible en la región de Azure de destino.
- Valide la conectividad de red entre el origen y la dirección IP del volumen de destino de Azure NetApp Files. La transferencia de datos entre el entorno local y el servicio Azure NetApp Files se admite a través de ExpressRoute.
- Cree el volumen de destino de Azure NetApp Files.
- Transfiera los datos de origen al volumen de destino mediante la herramienta de copia de archivo que prefiera.

### <a name="how-do-i-create-a-copy-of-an-azure-netapp-files-volume-in-another-azure-region"></a>¿Cómo puedo crear una copia de Azure NetApp Files en otra región de Azure?
    
Azure NetApp Files proporciona volúmenes SMB y NFS.  Cualquier herramienta de copia basada en archivos puede usarse para replicar datos entre regiones de Azure. 

NetApp ofrece una solución basada en SaaS, [NetApp Cloud Sync](https://cloud.netapp.com/cloud-sync-service).  La solución le permite replicar datos de NFS o SMB en los recursos compartidos de SMB o exportaciones de NFS de Azure NetApp Files. 

También puede utilizar una amplia gama de herramientas gratuitas para copiar los datos. Para NFS, puede usar las herramientas de cargas de trabajo, como [rsync](https://rsync.samba.org/examples.html) para copiar y sincronizar datos de origen en un volumen de Azure NetApp Files. Para SMB, puede usar las cargas de trabajo [robocopy](https://docs.microsoft.com/windows-server/administration/windows-commands/robocopy) de la misma manera.  Estas herramientas también pueden replicar los permisos de archivo o carpeta. 

Los requisitos para la replicación de un volumen de Azure NetApp Files en otra región de Azure son los siguientes: 
- Asegúrese de que Azure NetApp Files está disponible en la región de Azure de destino.
- Valide la conectividad de red entre redes virtuales en cada región. Actualmente, no se admite el emparejamiento global entre redes virtuales.  Puede establecer la conectividad entre redes virtuales mediante la vinculación con un circuito de ExpressRoute o el uso de una conexión VPN de sitio a sitio. 
- Cree el volumen de destino de Azure NetApp Files.
- Transfiera los datos de origen al volumen de destino mediante la herramienta de copia de archivo que prefiera.

### <a name="is-migration-with-azure-data-box-supported"></a>¿Se admite la migración con Azure Data Box?

No. En la actualidad, Azure Data Box no es compatible con Azure NetApp Files. 

### <a name="is-migration-with-azure-importexport-service-supported"></a>¿Se admite la migración con el servicio Azure Import/Export?

No. En la actualidad, el servicio Azure Import/Export no es compatible con Azure NetApp Files.

## <a name="next-steps"></a>Pasos siguientes  

- [Preguntas más frecuentes acerca de Microsoft Azure ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-faqs)
- [Preguntas más frecuentes acerca de Azure Virtual Network](https://docs.microsoft.com/azure/virtual-network/virtual-networks-faq)
- [Creación de una solicitud de soporte técnico de Azure](https://docs.microsoft.com/azure/azure-portal/supportability/how-to-create-azure-support-request)
- [Azure Data Box](https://docs.microsoft.com/azure/databox)
- [Preguntas más frecuentes sobre el rendimiento de SMB para Azure NetApp Files](azure-netapp-files-smb-performance.md)
