---
title: Configuración de un cliente NFS para Azure NetApp Files | Microsoft Docs
description: Describe cómo configurar los clientes NFS para usarlos con Azure NetApp Files.
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
ms.openlocfilehash: bdcffcea8d695b6a3d49272ffa135187d77aae1e
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87535290"
---
# <a name="configure-an-nfs-client-for-azure-netapp-files"></a>Configuración de un cliente NFS para Azure NetApp Files

Hay una amplia variedad de distribuciones de Linux disponibles para usarse con Azure NetApp Files. En este artículo se describen las configuraciones de dos de los entornos de uso más frecuentes: RHEL 8 y Ubuntu 18.04.

Independientemente del tipo de Linux que use, se requieren las siguientes configuraciones:
* Configuración de un cliente NTP para evitar problemas con el sesgo horario.
* Configuración de las entradas DNS del cliente Linux para la resolución de nombres.  
    Esta configuración incluye el registro "A" (adelante) y el registro PTR (inverso). 
* Para unirse a un dominio, cree una cuenta de equipo en la instancia de Active Directory de destino (que se crea durante el comando de unión a dominio de Kerberos). 
    > [!NOTE] 
    > La variable `$SERVICEACCOUNT` utilizada en los siguientes comandos debe ser una cuenta de usuario con permisos o delegación para crear una cuenta de equipo en la unidad organizativa de destino.
* Habilite el cliente para montar volúmenes NFS y otras herramientas de supervisión pertinentes.

## <a name="rhel-8-configuration"></a>Configuración de RHEL 8 

1. Instale los paquetes:   
    `sudo yum -y install realmd sssd adcli samba-common krb5-workstation chrony`

2. Configure el servicio de cliente NTP:  
    RHEL 8 usa `chrony` de forma predeterminada.  Siga las instrucciones de configuración de [Uso del conjunto Chrony para configurar NTP](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_basic_system_settings/using-chrony-to-configure-ntp).

3. Una el dominio de Active Directory:  
    `sudo realm join $DOMAIN.NAME -U $SERVICEACCOUNT --computer-ou= OU=$YOUROU,DC=$DOMAIN,DC=TLD`

## <a name="ubuntu-configuration"></a>Configuración de Ubuntu 

1. Instale los paquetes:  
    `sudo yum -y install realmd packagekit sssd adcli samba-common krb5-workstation chrony`

2. Configure el cliente NTP.  
    Ubuntu 18.04 usa `chrony` de forma predeterminada.  Siga las instrucciones de configuración de [Ubuntu Bionic: Uso de chrony para configurar NTP](https://ubuntu.com/blog/ubuntu-bionic-using-chrony-to-configure-ntp).

3. Una el dominio de Active Directory:  
    `sudo realm join $DOMAIN.NAME -U $SERVICEACCOUNT --computer-ou= OU=$YOUROU,DC=$DOMAIN,DC=TLD`

## <a name="next-steps"></a>Pasos siguientes  

* [Creación de un volumen de NFS para Azure NetApp Files](azure-netapp-files-create-volumes.md)
* [Creación de un volumen de dos protocolos para Azure NetApp Files](create-volumes-dual-protocol.md)

