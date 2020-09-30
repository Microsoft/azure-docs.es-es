---
title: Incorporación de un script a un plan de recuperación en Azure Site Recovery
description: Obtenga información sobre cómo agregar un script de VMM a un plan de recuperación para la recuperación ante desastres de máquinas virtuales de Hyper-V en nubes de VMM.
author: Sharmistha-Rai
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 11/27/2018
ms.author: sharrai
ms.openlocfilehash: 3217c30737a133c1c1092fc4a8a8caaa0338e980
ms.sourcegitcommit: ac5cbef0706d9910a76e4c0841fdac3ef8ed2e82
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89425881"
---
# <a name="add-a-vmm-script-to-a-recovery-plan"></a>Incorporación de un script de VMM a un plan de recuperación

En este artículo se describe cómo crear un script de System Center Virtual Machine Manager (VMM) y agregarlo a un plan de recuperación en [Azure Site Recovery](site-recovery-overview.md).

Publique cualquier comentario o pregunta que tenga al final del artículo o bien en la [Página de preguntas y respuestas de Microsoft sobre Azure Recovery Services](/answers/topics/azure-site-recovery.html).

## <a name="prerequisites"></a>Requisitos previos

Puede usar scripts de PowerShell en los planes de recuperación. Para que sea posible acceder al script desde el plan de recuperación, debe crearlo y colocarlo en la biblioteca de VMM. Cuando escriba el script, tenga en mente las consideraciones siguientes:

* Asegúrese de que los scripts usan bloques try-catch, para que las excepciones se controlen correctamente.
    - Si se produce una excepción en el script, este dejará de ejecutarse y la tarea se mostrará con errores.
    - Si se produce un error, el resto del script no se ejecuta.
    - Si se produce un error al ejecutar una conmutación por error no planeada, el plan de recuperación continúa.
    - Si se produce un error al ejecutar una conmutación por error planeada, el plan de recuperación se detiene. Corrija el script, compruebe que se ejecuta según lo previsto y, luego, vuelva a ejecutar el plan de recuperación.
        - El comando `Write-Host` no funciona en un script de plan de recuperación. Si usa el comando `Write-Host` en un script, el script presenta un error. Para crear salida, cree un script de proxy que a su vez se ejecute el script principal. Use el comando **\>\>** para asegurarse de que toda la salida se canalice.
        - Se agota el tiempo de espera del script si no vuelve en 600 segundos.
        - Si se escribe algo en STDERR, el script se clasifica como con errores. La información se muestra en los detalles de ejecución del script.

* Los scripts de los planes de recuperación se ejecutan en el contexto de la cuenta de servicio de VMM. Asegúrese de que esta cuenta tiene permisos de lectura para el recurso compartido remoto donde se encuentra el script. Pruebe el script para ejecutarlo con el mismo nivel de derechos de usuario que la cuenta de servicio de VMM.
* Los cmdlets de VMM se entregan en un módulo de Windows PowerShell. El módulo se instala al mismo tiempo que la consola VMM. Para cargar el módulo en el script, use el comando siguiente en el script: 

    `Import-Module -Name virtualmachinemanager`

    Para más información, consulte [Introducción a Windows PowerShell y VMM](/previous-versions/system-center/system-center-2012-R2/hh875013(v=sc.12)).
* Asegúrese de que tiene al menos un servidor de biblioteca en la implementación de VMM. De manera predeterminada, la ruta de acceso del recurso compartido de biblioteca para un servidor VMM se encuentra localmente en el servidor VMM. El nombre de la carpeta es MSCVMMLibrary.

  Si la ruta de acceso del recurso compartido de biblioteca es remota (o si es local, pero no se comparte con MSCVMMLibrary), configure el recurso compartido como se indica a continuación, con \\libserver2.contoso.com\share\ como ejemplo:
  
  1. Abra el Editor del Registro y vaya a **HKEY_LOCAL_MACHINE\SOFTWARE\MICROSOFT\Azure Site Recovery\Registration**.

  1. Cambie el valor de **ScriptLibraryPath** a **\\\libserver2.contoso.com\share\\** . Especifique el nombre de dominio completo. Proporcione permisos a la ubicación del recurso compartido. Este es el nodo raíz del recurso compartido. Para comprobar el nodo raíz, en VMM, vaya al nodo raíz en la biblioteca. La ruta de acceso que se abre es la raíz de la ruta de acceso. Esta es la ruta de acceso que debe usar en la variable.

  1. Pruebe el script con una cuenta de usuario que tenga el mismo nivel de derechos de usuario que la cuenta de servicio de VMM. Al usar estos derechos de usuario se comprueba que los scripts independientes y probados se ejecutan del mismo modo que se ejecutan en los planes de recuperación. En el servidor VMM, establezca que se omita la directiva de ejecución como se indica a continuación:

     a. Abra la consola de **Windows PowerShell de 64 bits** como administrador.
     
     b. Escriba **Set-executionpolicy bypass**. Para más información, consulte [Uso del cmdlet Set-ExecutionPolicy](/previous-versions/windows/it-pro/windows-powershell-1.0/ee176961(v=technet.10)).

     > [!IMPORTANT]
     > Establezca **Set-executionpolicy bypass** solo en la consola PowerShell de 64 bits. Si lo establece en la consola PowerShell de 32 bits, los scripts no se ejecutarán.

## <a name="add-the-script-to-the-vmm-library"></a>Adición del script a la biblioteca de VMM

Si tiene un sitio de origen de VMM, puede crear un script en el servidor VMM. Luego, incluya el script en el plan de recuperación.

1. En el recurso compartido de biblioteca, cree una carpeta nueva. Por ejemplo, \<VMM server name>\MSSCVMMLibrary\RPScripts. Coloque la carpeta en los servidores VMM de origen y de destino.
1. Cree el script. Por ejemplo, asígnele al script el nombre RPScript. Compruebe que el script funciona según lo previsto.
1. Coloque el script en la carpeta \<VMM server name>\MSSCVMMLibrary en los servidores VMM de origen y de destino.

## <a name="add-the-script-to-a-recovery-plan"></a>Adición del script a un plan de recuperación

Después de agregar las máquinas virtuales o los grupos de replicación a un plan de recuperación y crear el plan, puede agregar el script al grupo.

1. Abra el plan de recuperación.
1. En la lista de **pasos**, seleccione un elemento. Luego, seleccione **Script** o **Acción manual**.
1. Especifique si agregar el script o la acción antes o después del elemento seleccionado. Para subir o bajar la posición del script, seleccione los botones **Subir** o **Bajar**.
1. Si agrega un script de VMM, seleccione **Failover to VMM script** (Conmutación por error en script de VMM). En **Script Path** (Ruta de acceso del script), escriba la ruta de acceso relativa al recurso compartido. Por ejemplo, escriba **\RPScripts\RPScript.PS1**.
1. Si agrega un runbook de Azure Automation, especifique la cuenta de Automation en la que se encuentra el runbook. A continuación, seleccione el script del runbook de Azure que quiere usar.
1. Para asegurarse de que el script funciona según lo previsto, realice una conmutación por error de prueba del plan de recuperación.


## <a name="next-steps"></a>Pasos siguientes
* Aprenda más sobre la [ejecución de conmutaciones por error](site-recovery-failover.md).

