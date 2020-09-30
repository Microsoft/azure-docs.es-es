---
title: Procedimientos recomendados de Azure Automanage para máquinas virtuales
description: Obtenga información sobre los procedimientos recomendados de Azure Automanage para máquinas virtuales para servicios que se incorporan y configuran automáticamente.
author: deanwe
ms.service: virtual-machines
ms.subservice: automanage
ms.workload: infrastructure
ms.topic: conceptual
ms.date: 09/04/2020
ms.author: deanwe
ms.openlocfilehash: df03fc3049deaf5ce57fda74ca98c748ace1fbbc
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90931961"
---
# <a name="azure-automanage-for-virtual-machines-best-practices"></a>Procedimientos recomendados de Azure Automanage para máquinas virtuales


Estos servicios de Azure se incorporan automáticamente cuando usa Automanage para máquinas virtuales. Son esenciales para nuestras notas del producto de procedimientos recomendados, que puede encontrar en nuestro marco [Cloud Adoption Framework](https://docs.microsoft.com/azure/cloud-adoption-framework/manage/azure-server-management).

Todos estos servicios se incorporarán y configurarán automáticamente, se supervisarán en busca de variaciones y, en caso de detectarse estas, se corregirán. Para obtener más información sobre este proceso, consulte [Azure Automanage para máquinas virtuales](automanage-virtual-machines.md).


## <a name="participating-services"></a>Servicios participantes

|Servicio    |Descripción    |Perfiles admitidos<sup>1</sup>    |Preferencias admitidas<sup>1</sup>    |
|-----------|---------------|----------------------|-------------------------|
|Supervisión de VM Insights    |Azure Monitor para VM supervisa el rendimiento y el mantenimiento de las máquinas virtuales, incluidos los procesos en ejecución y las dependencias de otros recursos. [Más información](../azure-monitor/insights/vminsights-overview.md).    |Procedimientos recomendados de Azure Virtual Machines: producción    |No    |
|Copia de seguridad    |Azure Backup proporciona copias de seguridad independientes y aisladas para impedir la destrucción accidental de los datos en las máquinas virtuales. [Más información](../backup/backup-azure-vms-introduction.md). Los cargos se basan en el número y el tamaño de las VM que se están protegiendo. [Más información](https://azure.microsoft.com/pricing/details/backup/).    |Procedimientos recomendados de Azure Virtual Machines: producción    |Sí    |
|Azure Security Center    |Azure Security Center es un sistema unificado de administración de la seguridad de infraestructura que fortalece el nivel de seguridad de sus centros de datos, y proporciona protección contra amenazas avanzada en las cargas de trabajo híbridas de la nube. [Más información](../security-center/security-center-intro.md).  Automanage configurará la suscripción en la que reside la máquina virtual a la oferta de nivel Gratis de Azure Security Center. Si la suscripción ya se ha incorporado a Azure Security Center, Automanage no la volverá a configurar.    |Procedimientos recomendados de Azure Virtual Machines: producción; procedimientos recomendados de Azure Virtual Machines: desarrollo/pruebas    |No    |
|Microsoft Antimalware    |Microsoft Antimalware para Azure es una protección gratuita en tiempo real que ayuda a identificar y eliminar virus, spyware y otro software malintencionado. Genera alertas cuando software no deseado o malintencionado intenta instalarse o ejecutarse en los sistemas de Azure. [Más información](../security/fundamentals/antimalware.md). |Procedimientos recomendados de Azure Virtual Machines: producción; procedimientos recomendados de Azure Virtual Machines: desarrollo/pruebas    |Sí    |
|Administración de actualizaciones    |Use Update Management en Azure Automation para administrar las actualizaciones del sistema operativo de las máquinas virtuales. Puede evaluar rápidamente el estado de las actualizaciones disponibles en todas las máquinas agente y administrar el proceso de instalación de las actualizaciones necesarias para los servidores. [Más información](../automation/update-management/update-mgmt-overview.md).    |Procedimientos recomendados de Azure Virtual Machines: producción; procedimientos recomendados de Azure Virtual Machines: desarrollo/pruebas    |No    |
|Change Tracking e Inventario    |Change Tracking e Inventario combina funciones de inventario y seguimiento de cambios que le permiten realizar un seguimiento de los cambios en la infraestructura de servidores y máquinas virtuales. El servicio admite el seguimiento de cambios en el registro, servicios, demonios, software y archivos del entorno para ayudarle a diagnosticar cambios no deseados y generar alertas. La compatibilidad con inventario le permite consultar recursos de los invitados para obtener una visualización de las aplicaciones instaladas y otros elementos de configuración.  [Más información](../automation/change-tracking.md).    |Procedimientos recomendados de Azure Virtual Machines: producción; procedimientos recomendados de Azure Virtual Machines: desarrollo/pruebas    |No    |
|Cuenta de Azure Automation    |Azure Automation permite la administración de la infraestructura y las aplicaciones a lo largo de su ciclo de vida. [Más información](../automation/automation-intro.md).    |Procedimientos recomendados de Azure Virtual Machines: producción; procedimientos recomendados de Azure Virtual Machines: desarrollo/pruebas    |No    |
|Área de trabajo de Log Analytics    |Azure Monitor almacena los datos de registro en un área de trabajo de Log Analytics, que es un recurso de Azure y un contenedor en el que los datos se recopilan y se agregan, y que sirve como límite administrativo. [Más información](../azure-monitor/platform/design-logs-deployment.md).    |Procedimientos recomendados de Azure Virtual Machines: producción; procedimientos recomendados de Azure Virtual Machines: desarrollo/pruebas    |No    |


<sup>1</sup> Los perfiles de configuración están disponibles al habilitar Automanage. [Más información](automanage-virtual-machines.md#configuration-profiles). También puede ajustar la configuración predeterminada del perfil de configuración y establecer sus propias preferencias dentro de las restricciones de los procedimientos recomendados.


## <a name="next-steps"></a>Pasos siguientes

Intente habilitar Automanage para máquinas virtuales en Azure Portal.

> [!div class="nextstepaction"]
> [Habilitación de Automanage para máquinas virtuales en Azure Portal](quick-create-virtual-machines-portal.md)