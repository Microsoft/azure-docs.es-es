---
title: Administración de actualizaciones y revisiones para las máquinas virtuales en Azure Automation
description: En este artículo se indica cómo usar Update Management para administrar las actualizaciones y revisiones de las máquinas virtuales de Azure y las que no lo son.
services: automation
ms.subservice: update-management
ms.topic: conceptual
ms.date: 07/28/2020
ms.custom: mvc
ms.openlocfilehash: e4f630931e3cd79a46a539955b45d72e5b573271
ms.sourcegitcommit: cee72954f4467096b01ba287d30074751bcb7ff4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87449781"
---
# <a name="manage-updates-and-patches-for-your-vms"></a>Administración de actualizaciones y revisiones para las máquinas virtuales

Las actualizaciones de software de Update Management de Azure Automation proporcionan un conjunto de herramientas y recursos que pueden ayudar a administrar la compleja tarea de realizar un seguimiento de las actualizaciones de software y aplicarlas a las máquinas en Azure y en la nube híbrida. Se requiere un proceso de administración de actualizaciones de software eficaz para mantener la eficiencia operativa, superar los problemas de seguridad y reducir los riesgos de las cada vez mayores amenazas de seguridad en la red. Sin embargo, debido a la naturaleza cambiante de la tecnología y la aparición continua de nuevas amenazas de seguridad, la administración de las actualizaciones de software, para ser efectiva, requiere atención constante y continua.

> [!NOTE]
> Update Management permite implementar actualizaciones de terceros y descargarlas con antelación. Para ello, es necesario realizar cambios en los sistemas que se van a actualizar. Consulte [Configuración de los parámetros de Windows Update para Azure Automation Update Management](update-mgmt-configure-wuagent.md) para aprender a configurar estas opciones en sus sistemas.

Antes de intentar administrar las actualizaciones de las máquinas virtuales, asegúrese de que ha habilitado Update Management en ellas con uno de estos métodos:

* [Habilitación de Update Management desde una cuenta de Automation](update-mgmt-enable-automation-account.md)
* [Habilitación de Update Management desde Azure Portal](update-mgmt-enable-portal.md)
* [Habilitación de Update Management desde un runbook](update-mgmt-enable-runbook.md)
* [Habilitación de Update Management desde una maquina virtual de Azure](update-mgmt-enable-vm.md)

## <a name="limit-the-scope-for-the-deployment"></a><a name="scope-configuration"></a>Limitación del ámbito de implementación

Update Management usa una configuración de ámbito dentro del área de trabajo para dirigirse a los equipos que tienen que recibir actualizaciones. Para más información, consulte [Uso de configuraciones de ámbito para Update Management](update-mgmt-scope-configuration.md).

## <a name="compliance-assessment"></a>Evaluación del cumplimiento

Antes de implementar las actualizaciones de software en las máquinas, revise los resultados de la evaluación del cumplimiento de las actualizaciones para las máquinas habilitadas. Para cada actualización de software, se registra su estado de cumplimiento y, una vez completada la evaluación, se recopila y reenvía de forma masiva a los registros de Azure Monitor.

En una máquina Windows, el examen de cumplimiento se ejecuta cada 12 horas de forma predeterminada. Además del examen programado, el examen del cumplimiento de las actualizaciones se inicia a los 15 minutos del reinicio del agente de Log Analytics, así como antes y después de la instalación de actualizaciones. También es importante revisar nuestras recomendaciones sobre cómo [configurar el cliente de Windows Update](update-mgmt-configure-wuagent.md) con Update Management para evitar problemas que impidan su correcta administración.

En una máquina Linux, el examen de cumplimiento se realiza cada hora de manera predeterminada. Si se reinicia el agente de Log Analytics para Linux, se inicia un examen de cumplimiento al cabo de 15 minutos.

Los resultados de cumplimiento se presentan en Update Management para cada máquina evaluada. En el caso de una nueva máquina habilitada para administración, pueden transcurrir hasta 30 minutos antes de que el panel muestre los datos actualizados.

Revise [Supervisión de las actualizaciones de software](update-mgmt-view-update-assessments.md) para obtener información sobre cómo ver los resultados de cumplimiento.

## <a name="deploy-updates"></a>Implementación de actualizaciones

Después de revisar los resultados de cumplimiento, la fase de implementación de las actualizaciones de software es el proceso de implementación de las actualizaciones de software. Para instalar actualizaciones, programe una implementación que esté en consonancia con su ventana de programación y servicio de versiones. Puede elegir los tipos de actualizaciones que quiere incluir en la implementación. Por ejemplo, puede incluir actualizaciones de seguridad o críticas y excluir paquetes acumulativos de actualizaciones.

Revise [implementación de actualizaciones de software](update-mgmt-deploy-updates.md) para aprender a programar una implementación de actualizaciones.

## <a name="review-update-deployments"></a>Visualización de las implementaciones de actualizaciones

Una vez completada la implementación, revise el proceso para determinar el éxito de la implementación de actualizaciones según la máquina o el grupo de destino. Consulte [Revisión del estado de implementación](update-mgmt-deploy-updates.md#check-deployment-status) para obtener información sobre cómo puede supervisar el estado de la implementación.

## <a name="next-steps"></a>Pasos siguientes

* Para obtener información sobre cómo crear alertas que le notifiquen los resultados de la implementación de actualizaciones, consulte [Creación de alertas para Update Management](update-mgmt-configure-alerts.md).

* Puede [consultar los registros de Azure Monitor](update-mgmt-query-logs.md) para analizar las evaluaciones de las actualizaciones, las implementaciones y otras tareas de administración relacionadas. Incluye consultas predefinidas para ayudarle a empezar.