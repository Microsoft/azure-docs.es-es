---
title: Introducción a Azure Automation
description: En este artículo se indica qué es Azure Automation y cómo utilizarlo para automatizar el ciclo de vida de la infraestructura y las aplicaciones.
services: automation
ms.subservice: process-automation
keywords: Azure Automation, DSC, PowerShell, State Configuration, Update Management, Change Tracking, DSC, inventario, runbooks, Python, gráfico
ms.date: 10/18/2018
ms.custom: mvc
ms.topic: overview
ms.openlocfilehash: d10570efe49203f879cdceae776af5c3bc6991c0
ms.sourcegitcommit: cee72954f4467096b01ba287d30074751bcb7ff4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87448063"
---
# <a name="an-introduction-to-azure-automation"></a>Introducción a Azure Automation

Azure Automation ofrece un servicio de configuración y de automatización basado en la nube que facilita una administración coherente en los entornos que se encuentren dentro y fuera de Azure. Incluye automatización de procesos, administración de configuración, administración de actualizaciones, funcionalidades compartidas y características heterogéneas. Azure proporciona un control completo durante la implementación, las operaciones y la retirada de las cargas de trabajo y recursos.

![Funcionalidades de automatización](media/automation-overview/automation-overview.png)

## <a name="process-automation"></a>Automatización de procesos

La automatización de procesos de Azure Automation le permite automatizar tareas de administración en la nube laboriosas, frecuentes y propensas a errores. Este servicio le ayuda a centrarse en el trabajo que agrega valor empresarial. Al reducir los errores y aumentar la eficacia, también contribuye en la reducción de los costos operativos. El entorno operativo de automatización de procesos se describe con detalle en [Ejecución de un runbook en Azure Automation](automation-runbook-execution.md).

La automatización de procesos permite integrar servicios de Azure y otros sistemas públicos que son necesarios en la implementación, configuración y administración de los procesos de un extremo a otro. El servicio le permite crear [runbooks](automation-runbook-types.md) gráficamente en PowerShell o con Python. Mediante el uso de [Hybrid Runbook Worker](automation-hybrid-runbook-worker.md), puede unificar la administración mediante la organización de los entornos locales. Los [webhooks](automation-webhooks.md) le permiten responder a solicitudes y asegurar las operaciones y la entrega continua. Para ello, desencadenan la automatización desde ITSM, DevOps y los sistemas de supervisión. 

## <a name="configuration-management"></a>Administración de la configuración

La administración de la configuración en Azure Automation permite el acceso a dos características:

* Change Tracking e Inventario
* State Configuration de Azure Automation

### <a name="change-tracking-and-inventory"></a>Change Tracking e Inventario

Change Tracking e Inventario combina funciones de inventario y seguimiento de cambios que le permiten realizar un seguimiento de los cambios en la infraestructura de servidores y máquinas virtuales. El servicio admite el seguimiento de cambios en el registro, servicios, demonios, software y archivos del entorno para ayudarle a diagnosticar cambios no deseados y generar alertas. La compatibilidad con inventario le permite consultar recursos de los invitados para obtener una visualización de las aplicaciones instaladas y otros elementos de configuración. Para más información sobre esta característica, consulte [Change Tracking e Inventario](change-tracking.md).

### <a name="azure-automation-state-configuration"></a>State Configuration de Azure Automation

[State Configuration de Azure Automation](automation-dsc-overview.md) es una característica basada en la nube de Desired State Configuration (DSC) de PowerShell que proporciona servicios para entornos empresariales. Con esta característica, puede administrar los recursos de DSC en Azure Automation y aplicar las configuraciones a las máquinas virtuales o físicas desde un servidor de extracción DSC en la nube de Azure. 

## <a name="update-management"></a>Administración de actualizaciones

Azure Automation incluye la característica [Update Management](update-management/update-mgmt-overview.md) para sistemas Windows y Linux en entornos híbridos. Update Management le proporciona visibilidad sobre el cumplimiento de las actualizaciones en Azure y otras nubes, así como en el entorno local. Esta característica le permite crear implementaciones programadas para organizar la instalación de actualizaciones en una ventana de mantenimiento definida. Si no es necesario instalar una actualización en una máquina, puede usar la funcionalidad Update Management para excluirla de una implementación.

## <a name="shared-capabilities"></a>Funcionalidades compartidas

Azure Automation ofrece una serie de funcionalidades compartidas, como recursos compartidos, control de acceso basado en rol, programación flexible, integración de control de código fuente, auditoría y etiquetado.

### <a name="shared-resources"></a><a name="shared-resources"></a>Recursos compartidos

Azure Automation consta de un conjunto de recursos compartidos que le facilitan la automatización y configuración de los entornos a escala.

* **[Programaciones](./shared-resources/schedules.md)** : desencadene operaciones de automatización a horas predefinidas.
* **[Módulos](./shared-resources/modules.md)** : administre Azure y otros sistemas. Puede importar módulos a la cuenta de Automation para Microsoft, recursos de DSC y cmdlets definidos personalizados, de la comunidad o de terceros.
* **[Galería de módulos](automation-runbook-gallery.md)** : admite la integración nativa con la Galería de PowerShell para ver runbooks e importarlos a la cuenta de Automation. La galería le permite integrar y crear rápidamente los procesos desde la Galería de PowerShell y el Centro de scripts de Microsoft.
* **[Paquetes de Python 2](python-packages.md)** : admiten runbooks de Python 2 para su cuenta de Automation.
* **[Credenciales](./shared-resources/credentials.md)** : almacene de forma segura la información confidencial que los runbooks y las configuraciones pueden usar en tiempo de ejecución.
* **[Conexiones](automation-connections.md)** : almacenan pares nombre-valor de información común para las conexiones a sistemas. El autor del módulo define las conexiones en runbooks y configuraciones para usarlas en tiempo de ejecución.
* **[Certificados](./shared-resources/certificates.md)** : definen información que se usará en la autenticación y protección de recursos implementados cuando accedan a ellos runbooks o configuraciones de DSC en tiempo de ejecución. 
* **[Variables](./shared-resources/variables.md)** : almacenan contenido que puede usarse en runbooks y configuraciones. Puede cambiar los valores de las variables sin tener que modificar los runbooks ni las configuraciones que hacen referencia a ellos.

### <a name="role-based-access-control"></a>Control de acceso basado en rol

Azure Automation admite el control de acceso basado en rol (RBAC) para regular el acceso a la cuenta de Automation y sus recursos. Para más información sobre la configuración de RBAC en su cuenta de Automation, runbooks y trabajos, consulte [Control de acceso basado en rol en Azure Automation](automation-role-based-access-control.md).

### <a name="source-control-integration"></a>Integración de control de código fuente

Azure Automation permite la [integración del control de código fuente](source-control-integration.md). Esta característica promueve la configuración como código donde los runbooks o las configuraciones pueden insertarse en un sistema de control de código fuente.

## <a name="heterogeneous-support-windows-and-linux"></a>Compatibilidad heterogénea (Windows y Linux)

Automation está diseñado para trabajar tanto en el entorno de nube híbrida como en sistemas Windows y Linux. Le ofrece una manera coherente de automatizar y configurar cargas de trabajo implementadas y los sistemas operativos en los que se ejecutan.

## <a name="common-scenarios-for-automation"></a>Escenarios comunes para Automation

Azure Automation permite la administración de la infraestructura y las aplicaciones a lo largo de su ciclo de vida. Entre los escenarios habituales se incluyen los siguientes:

* **Escribir runbooks**: cree runbooks de PowerShell, flujo de trabajo de PowerShell, gráficos, Python 2 y DSC en lenguajes comunes. 
* **Compilar e implementar recursos**: implemente máquinas virtuales en un entorno híbrido mediante runbooks y plantillas de Azure Resource Manager. Integre en las herramientas de desarrollo, como Jenkins y Azure DevOps.
* **Configurar máquinas virtuales**: evalúe y configure máquinas Windows y Linux con configuraciones para la infraestructura y la aplicación.
* **Compartir conocimientos**: transfiera conocimientos al sistema sobre cómo la organización entrega y mantiene las cargas de trabajo. 
* **Recuperar inventario**: obtenga un inventario completo de los recursos implementados con fines de compatibilidad, creación de informes y cumplimiento. 
* **Identificar cambios**: identifique los cambios que pueden provocar una configuración incorrecta y mejore el cumplimiento operativo.
* **Supervisar**: aísle las máquinas que están ocasionando problemas y corríjalos o remítalos a una instancia superior de los sistemas de administración.
* **Proteger**: ponga máquinas en cuarentena si se generan alertas de seguridad. Establezca los requisitos de invitado.
* **Gobernar**: configure RBAC para equipos. Recupere recursos no utilizados.

[!INCLUDE [azure-lighthouse-supported-service](../../includes/azure-lighthouse-supported-service.md)]

## <a name="pricing-for-azure-automation"></a>Precios de Azure Automation

Puede revisar el precio asociados a Azure Automation en la página de [precios](https://azure.microsoft.com/pricing/details/automation/).

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación de una cuenta de Automation](automation-quickstart-create-account.md)
