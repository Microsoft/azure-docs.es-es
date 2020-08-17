---
title: Introducción al aprovisionamiento de Linux
description: Información general sobre cómo traer imágenes de máquinas virtuales Linux o crear imágenes para usarlas en Azure.
author: danielsollondon
ms.service: virtual-machines-linux
ms.subservice: imaging
ms.topic: overview
ms.workload: infrastructure
ms.date: 06/22/2020
ms.author: danis
ms.reviewer: cynthn
ms.openlocfilehash: a7d9aa7de8bb75a22acc85c77924765eaa1b6b3b
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87080155"
---
# <a name="azure-linux-vm-provisioning"></a>Aprovisionamiento de máquinas virtuales Linux en Azure
Al crear una máquina virtual a partir de una imagen generalizada (Shared Image Gallery o imagen administrada), el plano de control permitirá crear una máquina virtual y pasar los parámetros y valores a la máquina virtual. Esto se denomina *aprovisionamiento* de una máquina virtual. Durante el aprovisionamiento, la plataforma pone los valores necesarios de los parámetros de VM Create (hostname, username, password, SSH keys, customData) a disposición de la máquina virtual cuando esta arranca. 

Un agente de aprovisionamiento incorporado en la imagen hará de interfaz con la plataforma (se conectará a varias interfaces de aprovisionamiento independientes), establecerá las propiedades e indicará a la plataforma que ha finalizado. 

Los agentes de aprovisionamiento pueden ser el [agente Linux de Azure](../extensions/agent-linux.md) o [cloud-init](./using-cloud-init.md). Estos son los [requisitos previos](create-upload-generic.md) para la creación de imágenes generalizadas.

Los agentes de aprovisionamiento proporcionan compatibilidad con todas las [distribuciones de Linux aprobadas en Azure](./endorsed-distros.md) y, en muchos casos, las imágenes de las distribuciones aprobadas se enviarán tanto con cloud-init como con el agente Linux. Esto permite que cloud-init controle el aprovisionamiento y que, después, el agente Linux proporcione compatibilidad para controlar las [extensiones de Azure](../extensions/features-windows.md). Proporcionar compatibilidad con las extensiones significa que la máquina virtual podrá ser apta para usar servicios adicionales de Azure, como VM Password Reset, Azure Monitoring, Azure Backup, Azure Disk Encryption, etc.

Una vez que se complete el aprovisionamiento, cloud-init se ejecutará en todos los arranques. Cloud-init supervisará los cambios que se realicen en la máquina virtual, como los cambios de red, el montaje y formateo del disco efímero, y el inicio del agente Linux. El agente Linux se ejecuta continuamente en el servidor, ya que busca un "estado de objetivo" (nueva configuración) de la plataforma Azure, por lo que cada vez que se instalan las extensiones, el agente podrá procesarlas.

Aunque actualmente hay dos agentes de aprovisionamiento, debería elegir cloud-init, mientras que el agente Linux debe instalarse para la compatibilidad de las extensiones. Esto no solo permite aprovechar las optimizaciones de la plataforma, sino que también permite deshabilitar o quitar el agente Linux. Para más información sobre cómo crear imágenes sin el agente y cómo quitarlo, consulte esta [documentación](disable-provisioning.md).

Si tiene un kernel Linux que no admite la ejecución de ningún agente, pero quiere establecer algunas de las propiedades de VM Create, como hostname, customData, userName, password o ssh keys, en este documento aprenderá a [crear imágenes generalizadas sin un agente](no-agent.md) y a cumplir los requisitos de la plataforma.


## <a name="provisioning-agent-responsibilities"></a>Responsabilidades del agente de aprovisionamiento

**Aprovisionamiento de imágenes**
  
- Creación de una cuenta de usuario
- Configuración de los tipos de autenticación de SSH
- Implementación de claves públicas y pares de claves de SSH
- Configuración del nombre de host
- Publicación del nombre de host para DNS de plataforma
- Informe de huella digital de clave de host de SSH para la plataforma
- Administración del disco de recursos
- Formato y montaje del discos de recursos
- Consumo y procesamiento de `customData`
 
**Redes**
  
- Administra las rutas para mejorar la compatibilidad con los servidores DHCP de la plataforma
- Asegura la estabilidad del nombre de interfaz de red

**Kernel**
  
- Configura NUMA virtual (deshabilitar para el kernel cuya versión sea inferior a `2.6.37`)
- Consume entropía de Hyper-V para `/dev/random`
- Configura tiempos de espera SCSI para el dispositivo raíz (que podría ser remoto)

**Diagnóstico**
  
- Redirección de la consola al puerto serie

## <a name="communication"></a>Comunicación
El flujo de información desde la plataforma hasta el agente se produce a través de dos canales:

- Un DVD conectado de tiempo de arranque para las implementaciones de IaaS. Este DVD incluye un archivo de configuración compatible con OVF en el que encontrará toda la información de aprovisionamiento diferente de los pares de clave reales de SSH.
- Un punto de conexión TCP que expone una API REST que se usa para obtener la configuración de la implementación y la topología.


## <a name="azure-provisioning-agent-requirements"></a>Requisitos del agente de aprovisionamiento de Azure
El agente Linux y cloud-init dependen de algunos paquetes de sistema para funcionar correctamente:
- Python 2.6
- OpenSSL 1.0+
- OpenSSH 5.3+
- Utilidades del sistema de archivos: `sfdisk`, `fdisk`, `mkfs` y `parted`
- Herramientas de contraseña: chpasswd, sudo
- Herramientas de procesamiento de texto: sed, grep
- Herramientas de red: ip-route
- Compatibilidad de kernel para el montaje de sistemas de archivos UDF.

## <a name="next-steps"></a>Pasos siguientes

Si lo necesita, puede [deshabilitar el aprovisionamiento y quitar el agente Linux](disable-provisioning.md).
