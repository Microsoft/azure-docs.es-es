---
title: Solución de problemas de la extensión de máquina virtual de Azure para la recuperación ante desastres con Azure Site Recovery
description: Solucione problemas de la extensión de máquina virtual de Azure para la recuperación ante desastres con Azure Site Recovery.
author: sideeksh
manager: rochakm
ms.topic: troubleshooting
ms.date: 11/27/2018
ms.openlocfilehash: c1915d108bf9465d3e5b8d6a55053b583ee4f580
ms.sourcegitcommit: faeabfc2fffc33be7de6e1e93271ae214099517f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88184625"
---
# <a name="troubleshoot-azure-vm-extension-issues"></a>Solución de problemas de la extensión de máquina virtual de Azure

En este artículo se proporcionan pasos de solución de problemas que pueden ayudar a resolver errores de Azure Site Recovery relacionados con el agente y la extensión de máquina virtual.

## <a name="low-system-resources"></a>Recursos del sistema bajos

Este problema se produce cuando el sistema tiene poca memoria disponible y no puede asignar memoria para la instalación del servicio Mobility. Asegúrese de que se ha liberado suficiente memoria para que la instalación continúe y se complete correctamente.

## <a name="azure-site-recovery-extension-time-out"></a>Tiempo de espera de la extensión de Azure Site Recovery  

Mensaje de error: "se ha superado el tiempo de espera de ejecución de la tarea al realizar el seguimiento del inicio de la operación de extensión"<br>
Código de error: "151076"

 Azure Site Recovery instaló una extensión en la máquina virtual como parte de la habilitación de un trabajo de protección. Cualquiera de las condiciones siguientes puede evitar que se desencadene la protección y producir un error del trabajo. Siga los pasos de solución de problemas siguientes y luego vuelva a intentar la operación:

- [El agente está instalado en la máquina virtual, pero no responde (en máquinas virtuales Windows)](#the-agent-installed-in-the-vm-but-unresponsive-for-windows-vms)
- [El agente instalado en la máquina virtual está obsoleto (en el caso de máquinas virtuales Linux)](#the-agent-installed-in-the-vm-is-out-of-date-for-linux-vms)
- [La extensión de Site Recovery no se actualiza ni se carga](#the-site-recovery-extension-fails-to-update-or-load)

Mensaje de error: "La operación de extensión de Site Recovery anterior está tardando más tiempo del esperado".<br>
Código de error: "150066"

- [El agente está instalado en la máquina virtual, pero no responde (en máquinas virtuales Windows)](#the-agent-installed-in-the-vm-but-unresponsive-for-windows-vms)
- [El agente instalado en la máquina virtual está obsoleto (en el caso de máquinas virtuales Linux)](#the-agent-installed-in-the-vm-is-out-of-date-for-linux-vms)
- [El estado de la extensión de Site Recovery es incorrecto](#the-site-recovery-extension-fails-to-update-or-load)

## <a name="protection-fails-because-the-vm-agent-is-unresponsive"></a>Error de protección porque el agente de máquina virtual no responde

Mensaje de error: "se ha superado el tiempo de espera de ejecución de la tarea al realizar el seguimiento del inicio de la operación de extensión".<br>
Código de error: "151099"

Este error puede suceder si el agente invitado de Azure en la máquina virtual no está listo.

Puede comprobar el estado del agente invitado de Azure en [Azure Portal](https://portal.azure.com/). Vaya a la máquina virtual que está intentando proteger y compruebe el estado en **VM** > **Configuración** > **Propiedades** > **Estado del agente**. La mayoría de las veces el estado del agente pasa a ser Listo después de reiniciar la máquina virtual. No obstante, si no puede reiniciar o se sigue produciendo el problema, siga los siguientes pasos de solución de problemas:

- [El agente está instalado en la máquina virtual, pero no responde (en máquinas virtuales Windows)](#the-agent-installed-in-the-vm-but-unresponsive-for-windows-vms)
- [El agente instalado en la máquina virtual está obsoleto (en el caso de máquinas virtuales Linux)](#the-agent-installed-in-the-vm-is-out-of-date-for-linux-vms)


Mensaje de error: "se ha superado el tiempo de espera de ejecución de la tarea al realizar el seguimiento del inicio de la operación de extensión".<br>
Código de error: "151095"

Este error se produce cuando la versión del agente en la máquina Linux es antigua. Siga estos pasos de solución de problemas:

- [El agente instalado en la máquina virtual está obsoleto (en el caso de máquinas virtuales Linux)](#the-agent-installed-in-the-vm-is-out-of-date-for-linux-vms)  

## <a name="causes-and-solutions"></a>Causas y soluciones

### <a name="the-agent-is-installed-in-the-vm-but-its-unresponsive-for-windows-vms"></a><a name="the-agent-installed-in-the-vm-but-unresponsive-for-windows-vms"></a>El agente está instalado en la máquina virtual, pero no responde (en máquinas virtuales Windows)

#### <a name="solution"></a>Solución
Es posible que el agente de máquina virtual se haya dañado o que el servicio se haya detenido. Puede intentar volver a instalar al agente de máquina virtual para obtener la versión más reciente. O bien, intente restablecer la comunicación con el servicio.

1. Determine si el servicio de agente invitado de Microsoft Azure se ejecuta en los servicios de máquina virtual (services.msc). Reinicie el servicio Windows Azure Guest Agent.    
1. Si no puede ver el servicio Windows Azure Guest Agent entre los servicios, abra el panel de control. Vaya a **Programas y características** para ver si el servicio Windows Azure Guest Agent está instalado.
1. Si el agente invitado de Microsoft Azure aparece en **Programas y características**, desinstálelo.
1. Descargue e instale la [versión más reciente del MSI del agente](https://go.microsoft.com/fwlink/?LinkID=394789&clcid=0x409). Necesita derechos de administrador para completar la instalación.
1. Compruebe que el servicio Windows Azure Guest Agent aparece en los servicios.
1. Reinicie el trabajo de protección.

Además, compruebe que [Microsoft .NET 4.5](/dotnet/framework/migration-guide/how-to-determine-which-versions-are-installed) está instalado en la máquina virtual, Necesita .NET 4.5 para que el agente de máquina virtual se comunique con el servicio.

### <a name="the-agent-installed-in-the-vm-is-out-of-date-for-linux-vms"></a>El agente instalado en la máquina virtual está obsoleto (en el caso de máquinas virtuales Linux)

#### <a name="solution"></a>Solución
La mayoría de los errores relacionados con el agente o la extensión de máquinas virtuales de Linux están provocados por problemas que afectan a un agente VM obsoleto. Para solucionar este problema, siga estas directrices generales:

1. Siga las instrucciones para [actualizar el agente de máquina virtual Linux ](../virtual-machines/extensions/update-linux-agent.md).

   > [!NOTE]
   > Se *recomienda encarecidamente* actualizar el agente solo a través de un repositorio de distribución. No es recomendable descargar el código de agente desde GitHub directamente y actualizarlo. Si el último agente no está disponible para su distribución, póngase en contacto con el soporte técnico de distribución para obtener instrucciones sobre cómo instalarlo. Para buscar el agente más reciente, vaya a la página del [agente Linux de Microsoft Azure](https://github.com/Azure/WALinuxAgent/releases) en el repositorio de GitHub.

1. Asegúrese de que el agente de Azure se ejecuta en la máquina virtual mediante la ejecución del comando siguiente: `ps -e`.

   Si el proceso no se está ejecutando, reinícielo mediante los siguientes comandos:

   - Para Ubuntu: `service walinuxagent start`
   - Para otras distribuciones: `service waagent start`

1. [Configure el agente de reinicio automático](https://github.com/Azure/WALinuxAgent/wiki/Known-Issues#mitigate_agent_crash).
1. Habilite la protección de la máquina virtual.

### <a name="the-site-recovery-extension-fails-to-update-or-load"></a>La extensión de Site Recovery no se actualiza ni se carga

El estado de las extensiones aparece como "Vacío", "No está listo" o "En transición".

#### <a name="solution"></a>Solución

Desinstale la extensión y reinicie la operación.

Para desinstalar la extensión, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com/), vaya a la máquina virtual que experimenta los errores de copia de seguridad.
1. Seleccione **Configuración**.
1. Seleccione **Extensiones**.
1. Seleccione **Extensión de Site Recovery**.
1. Seleccione **Desinstalar**.

Para las máquinas virtuales Linux, si la extensión VMSnapshot no aparece en Azure Portal, [actualice el agente Linux de Azure](../virtual-machines/extensions/update-linux-agent.md). Después, ejecute la protección.

La realización de estos pasos hace que se vuelva a instalar la extensión durante la protección.
