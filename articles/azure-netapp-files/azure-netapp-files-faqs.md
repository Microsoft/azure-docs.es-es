---
title: Preguntas más frecuentes acerca de los archivos de NetApp Azure | Microsoft Docs
description: Respuestas a preguntas frecuentes sobre Azure Files de NetApp.
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
ms.date: 05/14/2019
ms.author: b-juche
ms.openlocfilehash: 6f1ca3398678b59a81e5c22b51b36a1f5505d4c2
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65806392"
---
# <a name="faqs-about-azure-netapp-files"></a>Preguntas más frecuentes acerca de los archivos de Azure de NetApp

Este artículo responden las preguntas más frecuentes (P+f) sobre Azure Files de NetApp. 

## <a name="networking-faqs"></a>Preguntas más frecuentes de redes

### <a name="does-the-nfs-data-path-go-over-the-internet"></a>¿La ruta de acceso de datos NFS que se va a través de Internet?  

No. La ruta de acceso de datos NFS no pasa a través de Internet. Azure Files de NetApp es un servicio nativo de Azure que se implementa en la red Virtual de Azure (VNet) que el servicio esté disponible. Los archivos de NetApp Azure usa una subred delegada y proporciona una interfaz de red directamente en la red virtual. 

Consulte [planeación de la red de directrices para Azure Files de NetApp](https://docs.microsoft.com/azure/azure-netapp-files/azure-netapp-files-network-topologies) para obtener más información.  

### <a name="can-i-connect-a-vnet-that-i-already-created-to-the-azure-netapp-files-service"></a>¿Puedo conectar una red virtual que ya creé con el servicio de Azure Files de NetApp?

Sí, puede conectar redes virtuales que ha creado para el servicio. 

Consulte [planeación de la red de directrices para Azure Files de NetApp](https://docs.microsoft.com/azure/azure-netapp-files/azure-netapp-files-network-topologies) para obtener más información.  

### <a name="can-i-mount-an-nfs-volume-of-azure-netapp-files-using-dns-fqdn-name"></a>¿Puedo montar un volumen NFS de NetApp de Azure Files mediante el nombre FQDN de DNS?

Sí, puede, si crea las entradas DNS necesarias. Los archivos de NetApp Azure proporciona la dirección IP de servicio para el volumen aprovisionado. 

> [!NOTE] 
> Los archivos de NetApp Azure puede implementar direcciones IP adicionales para el servicio según sea necesario.  Las entradas DNS que deba actualizarse periódicamente.

## <a name="security-faqs"></a>Preguntas más frecuentes de seguridad

### <a name="can-the-network-traffic-between-the-azure-vm-and-the-storage-be-encrypted"></a>¿El tráfico de red entre la máquina virtual de Azure y el almacenamiento se puede cifrar?

No se cifra el tráfico de datos (tráfico desde el cliente NFSv3 o SMBv3 a volúmenes de Azure Files de NetApp). Sin embargo, el tráfico desde una máquina virtual de Azure (que ejecuta a un cliente SMB o NFS) a Azure Files de NetApp es tan seguro como cualquier otro tráfico de máquina virtual de Azure. Este tráfico es local a la red de Azure del centro de datos. 

### <a name="can-the-storage-be-encrypted-at-rest"></a>¿El almacenamiento se puede cifrar en reposo?

Todos los volúmenes de Azure Files de NetApp se cifran mediante el FIPS 140-2 estándar. Todas las claves administradas por el servicio Azure Files de NetApp. 

### <a name="how-are-encryption-keys-managed"></a>¿Cómo se administran las claves de cifrado? 

Administración de claves para Azure Files de NetApp se controla mediante el servicio.  Actualmente, no se admiten claves administradas por el usuario (traiga sus propias claves).

### <a name="can-i-configure-the-nfs-export-policy-rules-to-control-access-to-the-azure-netapp-files-service-mount-target"></a>¿Puedo configurar las reglas de directivas de exportación NFS para controlar el acceso al destino de montaje de Azure Files de NetApp servicio?


Sí, puede configurar hasta cinco reglas en una sola directiva de exportación NFS.

### <a name="does-azure-netapp-files-support-network-security-groups"></a>¿Azure Files de NetApp admite grupos de seguridad de red?

No, actualmente no se puede aplicar grupos de seguridad de red a la subred de Azure Files de NetApp o las interfaces de red creadas por el servicio de delegado.

### <a name="can-i-use-azure-iam-with-azure-netapp-files"></a>¿Puedo usar IAM de Azure con Azure Files de NetApp?

Sí, Azure Files de NetApp admite características RBAC con Azure IAM.

## <a name="performance-faqs"></a>Preguntas más frecuentes de rendimiento

### <a name="what-should-i-do-to-optimize-or-tune-azure-netapp-files-performance"></a>¿Qué debo hacer para optimizar o ajustar el rendimiento de Azure Files de NetApp?

Puede realizar las siguientes acciones según los requisitos de rendimiento: 
- Asegúrese de que la máquina Virtual tiene el tamaño apropiado.
- Habilitar Accelerated Networking para la máquina virtual.
- Seleccione el nivel de servicio deseado y el tamaño de grupo de capacidades.
- Crear un volumen con el tamaño de cuota deseada para la capacidad y rendimiento.

### <a name="how-do-i-convert-throughput-based-service-levels-of-azure-netapp-files-to-iops"></a>¿Cómo se puede convertir los niveles de servicio basado en el rendimiento de Azure Files de NetApp a e/s por segundo?

MB/s se puede convertir a e/s por segundo mediante el uso de la siguiente fórmula:  

`IOPS = (MBps Throughput / KB per IO) * 1024`

### <a name="how-do-i-change-the-service-level-of-a-volume"></a>¿Cómo se puede cambiar el nivel de servicio de un volumen?

Actualmente no se admite el cambio del nivel de servicio de un volumen.

### <a name="how-do-i-monitor-azure-netapp-files-performance"></a>¿Cómo se puede supervisar el rendimiento de Azure Files de NetApp?

Azure Files de NetApp proporciona métricas de rendimiento del volumen. También puede usar a Azure Monitor para supervisar las métricas de uso de Azure Files de NetApp.  Consulte [métricas para Azure Files de NetApp](azure-netapp-files-metrics.md) para obtener la lista de métricas de rendimiento de Azure Files de NetApp.

## <a name="nfs-faqs"></a>Preguntas más frecuentes de NFS

### <a name="i-want-to-have-a-volume-mounted-automatically-when-an-azure-vm-is-started-or-rebooted--how-do-i-configure-my-host-for-persistent-nfs-volumes"></a>Me gustaría tener un volumen montado automáticamente cuando se inicia o reinicia una máquina virtual de Azure.  ¿Cómo configuro mi host para los volúmenes persistentes de NFS?

Para que un volumen NFS montar automáticamente al inicio de la máquina virtual o el reinicio, agregue una entrada para el `/etc/fstab` archivo en el host. 

Por ejemplo: `$ANFIP:/$FILEPATH      /$MOUNTPOINT    nfs bg,rw,hard,noatime,nolock,rsize=65536,wsize=65536,vers=3,tcp,_netdev 0 0`

- $ANFIP  
    La dirección IP del volumen de Azure Files de NetApp que se encuentra en la hoja de propiedades de volumen
- $FILEPATH  
    La ruta de acceso de exportación del volumen de Azure Files de NetApp
- $MOUNTPOINT  
    El directorio creado en el host de Linux que se utiliza para montar la exportación de NFS

### <a name="why-does-the-df-command-on-nfs-client-not-show-the-provisioned-volume-size"></a>¿Por qué el comando DF en el cliente para NFS no muestra el tamaño del volumen aprovisionado?

El tamaño del volumen notificado en DF es el tamaño máximo que puede alcanzar el volumen de Azure Files de NetApp. El tamaño del volumen de Azure Files de NetApp en comando DF no es reflejo la cuota o el tamaño del volumen.  Puede obtener el tamaño del volumen de Azure Files de NetApp o cuota a través de Azure portal o la API.

### <a name="what-nfs-version-does-azure-netapp-files-support"></a>¿Qué versión NFS admite Azure Files de NetApp?

Azure Files de NetApp actualmente admite NFSv3.

### <a name="how-do-i-enable-root-squashing"></a>¿Cómo habilita comprimir raíz?

Comprimir de raíz no se admite actualmente.

## <a name="smb-faqs"></a>Preguntas más frecuentes de SMB

### <a name="does-azure-netapp-files-support-azure-active-directory"></a>¿Azure Files de NetApp admite Azure Active Directory?

No, actualmente no se admite.  Azure Files de NetApp admite Active Directory Domain Services (traiga sus propios AD), que puede usar controladores de dominio de Active Directory existente con Azure Files de NetApp. Los controladores de dominio pueden residir en Azure como máquinas virtuales o en el entorno local a través de ExpressRoute.

### <a name="is-an-active-directory-connection-required-for-smb-access"></a>¿Es una conexión de Active Directory para el acceso SMB es necesaria? 

Sí, debe crear una conexión de Active Directory antes de implementar un volumen de SMB. Los controladores de dominio especificado debe ser accesibles por el delegado subred de Azure Files de NetApp para una conexión correcta.  Consulte [crear un volumen SMB](https://docs.microsoft.com/azure/azure-netapp-files/azure-netapp-files-create-volumes#create-an-smb-volume) para obtener más información. 

### <a name="how-many-active-directory-connections-are-supported"></a>¿Número de conexiones de Active Directory es compatibles?

Azure Files de NetApp actualmente admite una conexión de Active Directory por suscripción. Además, la conexión de Active Directory es específica para una sola cuenta de NetApp; no se comparte entre cuentas. 

### <a name="what-versions-of-windows-ad-are-supported"></a>¿Qué versiones de Windows AD son compatibles?

Archivos de NetApp Azure admite la versión de 2008r2SP1 2016 de Windows Server de Active Directory Domain Services.

## <a name="capacity-management-faqs"></a>Preguntas más frecuentes de administración de la capacidad

### <a name="how-do-i-monitor-usage-for-capacity-pool-and-volume-of-azure-netapp-files"></a>¿Cómo supervisar el uso de grupo de capacidad y el volumen de archivos de NetApp Azure? 

Azure Files de NetApp proporciona métricas de uso de grupo y el volumen de capacidad. También puede usar a Azure Monitor para supervisar el uso de Azure Files de NetApp. Consulte [métricas para Azure Files de NetApp](azure-netapp-files-metrics.md) para obtener más información. 

### <a name="can-i-manage-azure-netapp-files-through-azure-storage-explorer"></a>¿Puedo administrar Azure Files de NetApp a través del explorador de almacenamiento de Azure?

No. Los archivos de NetApp Azure no es compatible con el Explorador de Azure Storage.

## <a name="data-migration-and-protection-faqs"></a>Preguntas frecuentes de migración y la protección de datos

### <a name="how-do-i-migrate-data-to-azure-netapp-files"></a>¿Cómo se puede migrar datos a Azure Files de NetApp?
Azure Files de NetApp proporciona volúmenes SMB y NFS.  Puede usar cualquier herramienta de copia basada en archivo para migrar datos al servicio. 

NetApp ofrece una solución basada en SaaS, [sincronización en la nube de NetApp](https://cloud.netapp.com/cloud-sync-service).  La solución permite replicar NFS o recursos compartidos de datos SMB para SMB o NFS de archivos de NetApp Azure exportaciones. 

También puede utilizar una amplia gama de herramientas gratuitas para copiar los datos. Para NFS, puede usar las herramientas de las cargas de trabajo, como [rsync](https://rsync.samba.org/examples.html) para copiar y sincronizar datos de origen en un volumen de Azure Files de NetApp. Para SMB, puede usar las cargas de trabajo [robocopy](https://docs.microsoft.com/windows-server/administration/windows-commands/robocopy) de la misma manera.  Estas herramientas también pueden replicar los permisos de archivo o carpeta. 

Los requisitos para la migración de datos desde local a Azure Files de NetApp son los siguientes: 

- Asegúrese de que los archivos de NetApp Azure está disponible en la región de Azure de destino.
- Validar la conectividad de red entre el origen y la dirección IP de volumen de destino de Azure Files de NetApp. Transferencia de datos entre local y el servicio de Azure Files de NetApp se admite a través de ExpressRoute.
- Cree el volumen de archivos de NetApp de Azure de destino.
- Transferir los datos de origen para el volumen de destino mediante la herramienta de copia de archivo preferido.

### <a name="how-do-i-create-a-copy-of-an-azure-netapp-files-volume-in-another-azure-region"></a>¿Cómo se puede crear una copia de un volumen de Azure Files de NetApp en otra región de Azure?
    
Azure Files de NetApp proporciona volúmenes SMB y NFS.  Cualquier herramienta de copia basada en archivos puede usarse para replicar datos entre regiones de Azure. 

NetApp ofrece una solución basado en SaaS, [sincronización en la nube de NetApp](https://cloud.netapp.com/cloud-sync-service).  La solución permite replicar NFS o recursos compartidos de datos SMB para SMB o NFS de archivos de NetApp Azure exportaciones. 

También puede utilizar una amplia gama de herramientas gratuitas para copiar los datos. Para NFS, puede usar las herramientas de las cargas de trabajo, como [rsync](https://rsync.samba.org/examples.html) para copiar y sincronizar datos de origen en un volumen de Azure Files de NetApp. Para SMB, puede usar las cargas de trabajo [robocopy](https://docs.microsoft.com/windows-server/administration/windows-commands/robocopy) de la misma manera.  Estas herramientas también pueden replicar los permisos de archivo o carpeta. 

Los requisitos para la replicación de un volumen de Azure Files de NetApp en otra región de Azure son los siguientes: 
- Asegúrese de que los archivos de NetApp Azure está disponible en la región de Azure de destino.
- Validar la conectividad de red entre redes virtuales en cada región. Actualmente, no se admite el emparejamiento global entre redes virtuales.  Puede establecer la conectividad entre redes virtuales mediante la vinculación con un circuito de ExpressRoute o usar una conexión VPN de S2S. 
- Cree el volumen de archivos de NetApp de Azure de destino.
- Transferir los datos de origen para el volumen de destino mediante la herramienta de copia de archivo preferido.

### <a name="is-migration-with-azure-data-box-supported"></a>¿Es la migración con Azure Data Box admite?

No. Azure Data Box no es compatible actualmente con Azure Files de NetApp. 

### <a name="is-migration-with-azure-importexport-service-supported"></a>¿Es la migración con el servicio Azure Import/Export admitida?

No. Servicio Azure Import/Export no admite actualmente Azure Files de NetApp.

## <a name="next-steps"></a>Pasos siguientes  

- [Preguntas más frecuentes de Microsoft Azure ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-faqs)
- [Preguntas más frecuentes de Microsoft Azure Virtual Network](https://docs.microsoft.com/azure/virtual-network/virtual-networks-faq)
- [Creación de una solicitud de soporte técnico de Azure](https://docs.microsoft.com/azure/azure-supportability/how-to-create-azure-support-request)
- [Azure Data Box](https://docs.microsoft.com/azure/databox-family/)