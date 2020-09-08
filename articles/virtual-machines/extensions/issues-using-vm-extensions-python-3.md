---
title: Problemas al usar extensiones de VM en los sistemas de Azure Virtual Machines para Linux habilitados para Python 3
description: Más información sobre el uso de extensiones de VM en los sistemas Linux habilitados para Python 3
services: virtual-machines-windows
documentationcenter: ''
author: v-miegge
ms.author: jparrel
manager: dcscontentpm
editor: ''
tags: top-support-issue,azure-resource-manager
ms.service: virtual-machines-windows
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 08/25/2020
ms.assetid: 3cd520fd-eaf7-4ef9-b4d3-4827057e5028
ms.openlocfilehash: 15ece836e172b8316222ea606ca638650795d5d7
ms.sourcegitcommit: b33c9ad17598d7e4d66fe11d511daa78b4b8b330
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88852607"
---
# <a name="issues-using-vm-extensions-in-python-3-enabled-linux-azure-virtual-machines-systems"></a>Problemas al usar extensiones de VM en los sistemas de Azure Virtual Machines para Linux habilitados para Python 3

> [!NOTE]
> Microsoft recomienda a los usuarios adoptar **Python 3.x** en sus sistemas a menos que la carga de trabajo requiera compatibilidad con **Python 2.x**. Algunos ejemplos de este requisito pueden incluir scripts de administración heredados o extensiones como **Azure Disk Encryption** y **Azure Monitor**.
>
> Antes de instalar **Python 2.x** en producción, tenga en cuenta la cuestión de la compatibilidad a largo plazo con Python 2.x, especialmente su capacidad para recibir actualizaciones de seguridad. Respecto a los productos, incluida parte de la extensión mencionada, actualización con compatibilidad para **Python 3.8**, debe dejar de usar Python 2.x.

Algunas distribuciones de Linux han pasado a Python 3.8 y han quitado el punto de entrada de `/usr/bin/python` heredado para Python por completo. Esta transición afecta a la implementación automatizada e integrada de ciertas extensiones de máquina virtual (VM) con estas dos condiciones:

- Extensiones en transición a la compatibilidad con Python 3.x
- Extensiones que usan el punto de entrada `/usr/bin/python` heredado

Los usuarios de distribución de Linux que han pasado a **Python 3.x** deben asegurarse de que el punto de entrada `/usr/bin/python` heredado exista antes de intentar implementar esas extensiones en sus VM. De lo contrario, podría producirse un error en la implementación de la extensión. 

- Las distribuciones de Linux aprobadas que se ven afectadas incluyen **Ubuntu Server 20.04 LTS** y **Ubuntu Pro 20.04 LTS**.

- Entre las extensiones de VM afectadas se incluyen **Azure Disk Encryption**, **Log Analytics**, **VM Access** (Acceso de VM), que se usa para el restablecimiento de contraseña, y **Guest Diagnostics** (Diagnóstico de invitado), que se usa para otros contadores de rendimiento.

Las actualizaciones locales, como la actualización de **Ubuntu 18.04 LTS** a **Ubuntu 20.04 LTS**, deben conservar el vínculo simbólico de `/usr/bin/python` y no deben verse afectadas.

## <a name="resolution"></a>Solución

Tenga en cuenta estas recomendaciones generales antes de implementar extensiones en los escenarios afectados conocidos que se han descrito anteriormente en el Resumen:

1. Antes de implementar la extensión, restablezca el vínculo simbólico `/usr/bin/python` mediante el método de distribución de Linux proporcionado por el proveedor.

   - Por ejemplo, para **Python 2.7**, use: `sudo apt update && sudo apt install python-is-python2`

1. Esta recomendación es para los clientes de Azure y no se admite en Azure Stack:

   - Si ya ha implementado una instancia que presenta este problema, use la funcionalidad del comando Ejecutar en la hoja de VM para ejecutar los comandos mencionados anteriormente. La propia extensión del comando de ejecución no se ve afectada por la transición a Python 3.8.

1. Si va a implementar una nueva instancia y necesita establecer una extensión en el momento del aprovisionamiento, use los datos de usuario de **cloud-init** para instalar los paquetes mencionados anteriormente.

   Por ejemplo, para Python 2.7:

   ```python
   # create cloud-init config
   cat > cloudinitConfig.json <<EOF
   #cloud-config
   package_update: true
    
   runcmd:
   - sudo apt update
   - sudo apt install python-is-python2 
   EOF

   # create VM
   az vm create \
       --resource-group <resourceGroupName> \
       --name <vmName> \
       --image <Ubuntu 20.04 Image URN> \
       --admin-username azadmin \
       --ssh-key-value "<sshPubKey>" \
       --custom-data ./cloudinitConfig.json
   ```

1. si los administradores de directivas de la organización determinan que las extensiones no se deben implementar en VM, puede deshabilitar la compatibilidad con las extensiones en el momento del aprovisionamiento:

   - API DE REST

     Para deshabilitar y habilitar las extensiones cuando se puede implementar una VM con esta propiedad:

     ```python
       "osProfile": {
         "allowExtensionOperations": false
       },
     ```

## <a name="next-steps"></a>Pasos siguientes

Consulte [otros cambios del sistema base desde 18.04 LTS-Python 3 predeterminado](https://wiki.ubuntu.com/FocalFossa/ReleaseNotes#Python3_by_default) para obtener información adicional.
