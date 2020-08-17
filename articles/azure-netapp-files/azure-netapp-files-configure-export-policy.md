---
title: 'Configuración de la directiva de exportación para el volumen NFS: Azure NetApp Files'
description: Describe cómo configurar la directiva de exportación para controlar el acceso a un volumen NFS mediante Azure NetApp Files
services: azure-netapp-files
author: b-juche
ms.author: b-juche
ms.service: azure-netapp-files
ms.workload: storage
ms.topic: how-to
ms.date: 07/27/2020
ms.openlocfilehash: 4a20a223932f82c80ad5831ef3a02bad803e26e6
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87533225"
---
# <a name="configure-export-policy-for-an-nfs-volume"></a>Configuración de la directiva de exportación para un volumen NFS

Puede configurar la directiva de exportación para controlar el acceso a un volumen de Azure NetApp Files. La directiva de exportación de Azure NetApp Files admite volúmenes que usan el protocolo NFS (NFSv3 y NFSv4.1) y el protocolo dual (NFSv3 y SMB). 

Puede crear hasta cinco reglas en la directiva de exportación.

## <a name="steps"></a>Pasos 

1.  En la página Volúmenes, seleccione el volumen para el que desea configurar la directiva de exportación y haga clic en **Directiva de exportación**. 

    También puede configurar la directiva de exportación durante la creación del volumen.

2.  Especifique la información de los campos siguientes para crear una regla de directiva de exportación:   
    *  **Índice**   
        Especifique el número de índice para la regla.  
        Una directiva de exportación puede constar de hasta cinco reglas. Las reglas se evalúan según el orden en la lista de números de índice. Primero se evalúan las reglas con números de índice inferior. Por ejemplo, la regla con el número de índice 1 se evalúa antes que la que tenga el número de índice 2. 

    * **Clientes permitidos**   
        Especifique el valor en uno de los siguientes formatos:  
        * Dirección IPv4, por ejemplo, `10.1.12.24` 
        * Dirección IPv4 con una máscara de subred expresada como un número de bits, por ejemplo, `10.1.12.10/4`

    * **Acceder**  
        Seleccione uno de los tipos de accesos siguientes:  
        * Sin acceso 
        * Lectura y escritura
        * Solo lectura

    * **Solo lectura** y **Lectura y escritura**  
        Si usa el cifrado Kerberos con NFSv 4.1, siga las instrucciones de [Configuración del cifrado Kerberos con NFSv4.1](configure-kerberos-encryption.md).  Para ver el impacto en el rendimiento de Kerberos, vea [Impacto en el rendimiento de Kerberos en NFSv4.1](configure-kerberos-encryption.md#kerberos_performance). 

        ![Opciones de seguridad de Kerberos](../media/azure-netapp-files/kerberos-security-options.png) 

    * **Acceso a la raíz**  
        Especifique si la cuenta `root` puede acceder al volumen.  De forma predeterminada, el acceso a la raíz se establece en **On**y la cuenta `root` tiene acceso al volumen.

![Directiva de exportación](../media/azure-netapp-files/azure-netapp-files-export-policy.png) 



## <a name="next-steps"></a>Pasos siguientes 
* [Instalación o desinstalación de un volumen para máquinas virtuales](azure-netapp-files-mount-unmount-volumes-for-virtual-machines.md)
* [Administración de instantáneas](azure-netapp-files-manage-snapshots.md)
