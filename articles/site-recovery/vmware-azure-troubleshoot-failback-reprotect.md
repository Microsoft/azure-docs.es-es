---
title: Solución de problemas de la conmutación por recuperación en recuperación ante desastres de VM de VMware con Azure Site Recovery
description: En este artículo se describen formas de solucionar los problemas de conmutación por recuperación y reprotección durante la recuperación ante desastres de máquinas virtuales de VMware en Azure con Azure Site Recovery.
author: Sharmistha-Rai
manager: gaggupta
ms.service: site-recovery
ms.topic: conceptual
ms.date: 11/27/2018
ms.author: sharrai
ms.openlocfilehash: ed4e52470264441a99c5ccf0a736bb00233510c1
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87423123"
---
# <a name="troubleshoot-failback-to-on-premises-from-azure"></a>Solución de problemas de conmutación por recuperación al entorno local desde Azure

En este artículo se explica cómo solucionar los problemas que se puede encontrar al realizar una conmutación por recuperación de las máquinas virtuales de Azure en la infraestructura local de VMware, después de una conmutación por error en Azure mediante [Azure Site Recovery](site-recovery-overview.md).

La conmutación por recuperación implica básicamente dos pasos principales. En primer lugar, después de la recuperación por error, debe volver a proteger las máquinas virtuales de Azure en las ubicaciones locales para que puedan comenzar la replicación. El segundo paso es ejecutar una conmutación por error desde Azure, para luego ejecutar una conmutación por recuperación en su sitio local.

## <a name="common-issues"></a>Problemas comunes

- Si realiza la detección de usuarios de solo lectura de vCenter y protege las máquinas virtuales, la protección se ejecutará correctamente y la conmutación por error funcionará. Durante la reprotección, se produce un error de conmutación por error porque no se pueden detectar los almacenes de datos. Un síntoma es que los almacenes de datos no aparecen durante la reprotección. Para solucionar este problema, puede actualizar las credenciales de vCenter con una cuenta adecuada que tenga permisos y volver a intentar el trabajo.
- Al conmutar por recuperación una máquina virtual Linux y ejecutarla en local, puede ver que el paquete del administrador de red se ha desinstalado de la máquina. Esto se debe a que el paquete del administrador de red se elimina cuando se recupera la máquina virtual en Azure.
- Cuando una máquina virtual Linux se configura con una dirección IP estática y se conmuta por error a Azure, se adquiere la dirección IP desde DHCP. Al conmutar por error en local, la máquina virtual sigue usando DHCP para obtener la dirección IP. Inicie sesión manualmente en la máquina y vuelva a establecer la dirección IP en una dirección estática si es necesario. Una máquina virtual Windows puede adquirir su dirección IP estática de nuevo.
- Si usa las ediciones gratuitas de ESXi 5.5 o vSphere 6 Hypervisor, la conmutación por error se llevará a cabo correctamente, pero la conmutación por recuperación, no. Para habilitar la conmutación por recuperación, tendrá que actualizar los programas con una licencia de evaluación.
- Si no puede ponerse en contacto con el servidor de configuración desde el servidor de procesos, use Telnet para comprobar la conectividad con el servidor de configuración en el puerto 443. También puede tratar de hacer ping al servidor de configuración desde el servidor de procesos. Un servidor de procesos debe tener también un latido cuando esté conectado al servidor de configuración.
- No se puede llevar a cabo la conmutación por recuperación de un servidor Windows Server 2008 R2 SP1 que esté protegido como servidor local físico desde Azure a un sitio local.
- No podrá conmutar por recuperación en las siguientes circunstancias:
    - Ha migrado máquinas a Azure. 
    - Ha movido una máquina virtual a otro grupo de recursos.
    - Ha eliminado la máquina virtual de Azure.
    - Ha deshabilitado la protección de la máquina virtual.
    - Ha creado manualmente la máquina virtual en Azure. La máquina se debe haber protegido inicialmente en local y haber conmutador por error a Azure antes de la reprotección.
    - Solo puede conmutar por recuperación a un host de ESXi. No puede conmutar por recuperación máquinas virtuales de VMware o servidores físicos a hosts de Hyper-V, máquinas físicas o estaciones de trabajo de VMware.


## <a name="troubleshoot-reprotection-errors"></a>Solución de errores de reprotección

En esta sección se detallan los errores más habituales de reprotección y cómo corregirlos.

### <a name="error-code-95226"></a>Código de error 95226

**No se pudo aplicar la reprotección porque la máquina virtual de Azure no pudo acceder al servidor de configuración local.**

Este error se produce cuando:

* La máquina virtual de Azure no puede llegar al servidor de configuración local. La máquina virtual no se puede detectar ni registrar en el servidor de configuración.
* El servicio de aplicación InMage Scout no se ejecuta en la máquina virtual de Azure después de la conmutación por error. El servicio es necesario para las comunicaciones con el servidor de configuración local.

Para solucionar este problema:

* Compruebe que la red de VM de Azure permite que la máquina virtual de Azure se comunique con el servidor de configuración local. Puede configurar una VPN de sitio a sitio en el centro de datos local o una conexión de Azure ExpressRoute con pares privados en la red virtual de la máquina virtual de Azure.
* Si la máquina virtual puede comunicarse con el servidor de configuración local, inicie sesión en la máquina virtual. A continuación, compruebe el servicio de aplicación de InMage Scout. Si ve que no se está ejecutando, inicie el servicio manualmente. Compruebe que el tipo de inicio del servicio está configurado en **Automático**.

### <a name="error-code-78052"></a>Código de error 78052

**No se pudo finalizar la protección de la máquina virtual.**

Este problema puede suceder si ya hay una máquina virtual con el mismo nombre en el servidor de destino maestro en el que se está realizando la conmutación por recuperación.

Para solucionar este problema:

* Seleccione otro servidor de destino maestro de un host diferente, con el fin de que la reprotección cree la máquina en otro host, donde los nombres no entren en conflicto.
* También puede usar VMotion para mover el destino maestro a otro host en el que no se produzca el conflicto de nombres. Si la máquina virtual existente es una máquina aislada, cámbiele el nombre para que se pueda crear la nueva máquina virtual en el mismo host ESXi.


### <a name="error-code-78093"></a>Código de error 78093

**La VM no se está ejecutando, no responde o no está accesible**.

Para solucionar este problema:

Para volver a proteger una máquina virtual que ha conmutado por error, la máquina virtual de Azure debe estar en ejecución para que Mobility Service se registre con el servidor de configuración local y pueda empezar a realizar la replicación mediante la comunicación con el servidor de procesos. Si el equipo está en una red incorrecta o no se ejecuta (no responde o apagado), el servidor de configuración no puede acceder a Mobility Service de la máquina virtual para iniciar la reprotección.

* Puede reiniciar la máquina virtual para que pueda comenzar la comunicación con el entorno local.
* Reinicie el trabajo de reprotección después de iniciar la máquina virtual de Azure.

### <a name="error-code-8061"></a>Código de error 8061

**El almacén de datos no es accesible desde el host ESXi.**

Compruebe los [requisitos previos del servidor de destino maestro y los almacenes de datos admitidos](vmware-azure-prepare-failback.md#deploy-a-separate-master-target-server) para la conmutación por recuperación.


## <a name="troubleshoot-failback-errors"></a>Solución de errores en la conmutación por recuperación

En esta sección se describen los errores habituales que pueden surgir durante una conmutación por recuperación.

### <a name="error-code-8038"></a>Código de error 8038

**No se pudo iniciar la máquina virtual local debido al error.**

Este problema sucede cuando la máquina virtual local se inicia en un host que no tiene suficiente memoria aprovisionada. 

Para solucionar este problema:

* Aprovisione más memoria en el host ESXi.
* Además, puede mover la máquina virtual con VMotion a otro host ESXi que tenga memoria suficiente para iniciar la VM.
