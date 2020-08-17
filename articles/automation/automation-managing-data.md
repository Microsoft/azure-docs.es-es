---
title: Seguridad de datos de Azure Automation
description: En este artículo aprenderá cómo Azure Automation protege su privacidad y los datos.
services: automation
ms.subservice: shared-capabilities
ms.date: 07/20/2020
ms.topic: conceptual
ms.openlocfilehash: 610c2050150a533e246bc74ed7750ce87f7cf617
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87004654"
---
# <a name="management-of-azure-automation-data"></a>Administración de datos de Azure Automation

Este artículo contiene varios temas que explican cómo se protegen y aseguran los datos en un entorno de Azure Automation.

## <a name="tls-12-enforcement-for-azure-automation"></a>Aplicación de TLS 1.2 para Azure Automation

Para garantizar la seguridad de los datos en tránsito hacia Azure Automation, se recomienda encarecidamente configurar el uso de Seguridad de la capa de transporte (TLS) 1.2. A continuación se muestra una lista de métodos o clientes que se comunican a través de HTTPS con el servicio Automation:

* Llamadas de webhook

* Instancias de Hybrid Runbook Worker, que incluyen máquinas administradas por Update Management y Change Tracking e inventario.

* Nodos de DSC

Las versiones anteriores de TLS/Capa de sockets seguros (SSL) han demostrado ser vulnerables y, si bien todavía funcionan para permitir la compatibilidad con versiones anteriores, **no se recomiendan**. A partir de septiembre de 2020, comenzamos a aplicar TLS 1.2 y versiones posteriores del protocolo de cifrado.

No se recomienda establecer explícitamente el agente para que solo use TLS 1.2, a menos que sea absolutamente necesario, ya que esto puede interrumpir las características de seguridad a nivel de la plataforma que le permiten detectar y aprovechar automáticamente las ventajas de los protocolos más seguros más recientes a medida que estén disponibles, como TLS 1.3.

Para obtener información sobre la compatibilidad de TLS 1.2 con el agente de Log Analytics para Windows y Linux, que es una dependencia del rol de Hybrid Runbook Worker, consulte la [introducción del agente de Log Analytics: TLS 1.2](..//azure-monitor/platform/log-analytics-agent.md#tls-12-protocol). 

### <a name="platform-specific-guidance"></a>Guía específica para la plataforma

|Plataforma/lenguaje | Soporte técnico | Más información |
| --- | --- | --- |
|Linux | Las distribuciones de Linux tienden a basarse en [OpenSSL](https://www.openssl.org) para la compatibilidad con TLS 1.2.  | Compruebe el [registro de cambios de OpenSSL](https://www.openssl.org/news/changelog.html) para confirmar si su versión de OpenSSL es compatible.|
| Windows 8.0 a 10 | Compatible y habilitado de manera predeterminada. | Para confirmar que aún usa la [configuración predeterminada](/windows-server/security/tls/tls-registry-settings).  |
| Windows Server 2012 a 2016 | Compatible y habilitado de manera predeterminada. | Para confirmar que aún usa la [configuración predeterminada](/windows-server/security/tls/tls-registry-settings) |
| Windows 7 SP1 y Windows Server 2008 R2 SP1 | Compatible, pero no habilitado de manera predeterminada. | Consulte la página [Configuración del registro de TLS](/windows-server/security/tls/tls-registry-settings) para obtener más información sobre cómo se habilita.  |

## <a name="data-retention"></a>Retención de datos

Cuando elimina un recurso en Azure Automation, se conserva durante un número de días para fines de auditoría antes de su eliminación permanente. Durante este tiempo, no es posible ver ni usar el recurso. Esta directiva también se aplica a los recursos que pertenecen a una cuenta de Automation eliminada.

La tabla siguiente resume la directiva de retención para distintos recursos.

| data | Directiva |
|:--- |:--- |
| Cuentas |Una cuenta se elimina de manera permanente 30 días después de que un usuario la elimine. |
| Recursos |Un recurso se elimina de forma permanente 30 días después de que un usuario lo elimina o 30 días después de que un usuario elimina la cuenta que contiene el recurso. Los recursos incluyen variables, programaciones, credenciales, certificados, paquetes de Python 2 y conexiones. |
| Nodos de DSC |Un nodo de DSC se elimina de forma permanente 30 días después de que se anula el registro del nodo de la cuenta de Automation mediante Azure Portal o el cmdlet [Unregister-AzAutomationDscNode](/powershell/module/az.automation/unregister-azautomationdscnode?view=azps-3.7.0) de Windows PowerShell. Los nodos también se eliminan de forma permanente 30 días después de que el usuario elimine la cuenta que contiene el nodo. |
| Trabajos |Un trabajo se elimina de forma permanente 30 días después de cualquier modificación, por ejemplo, después de que el trabajo se complete, detenga o suspenda. |
| Módulos |Un módulo se elimina de forma permanente 30 días después de que un usuario lo elimina o 30 días después de que un usuario elimina la cuenta que contiene el módulo. |
| Configuraciones de nodo y archivos MOF |La configuración de nodo anterior se elimina de forma permanente 30 días después de que se genere una nueva configuración. |
| Informes de nodo |Un informe de nodo se elimina de forma permanente 90 días después de que se genere un nuevo informe para ese nodo. |
| Runbooks |Un runbook se elimina de forma permanente 30 días después de que un usuario lo elimina o 30 días después de que un usuario elimina la cuenta que contiene el recurso. |

La directiva de retención se aplica a todos los usuarios y, por el momento, no se puede personalizar. Sin embargo, si tiene que conservar los datos durante un periodo de tiempo mayor, puede [reenviar los datos de trabajos de Azure Automation a los registros de Azure Monitor](automation-manage-send-joblogs-log-analytics.md).

## <a name="data-backup"></a>Copia de seguridad de datos

Cuando elimina una cuenta de Automation en Azure, se eliminan todos los objetos de esa cuenta. Entre los objetos se incluyen runbooks, módulos, configuraciones, valores, trabajos y recursos. No es posible recuperar los objetos una vez eliminada la cuenta. Puede usar la información siguiente para crear una copia de seguridad de los contenidos de la cuenta de Automation antes de eliminarla.

### <a name="runbooks"></a>Runbooks

Puede exportar los runbooks a archivos de script con Azure Portal o el cmdlet [Get-AzureAutomationRunbookDefinition](/powershell/module/servicemanagement/azure.service/get-azureautomationrunbookdefinition) en Windows PowerShell. Puede importar estos archivos de script en otra cuenta de Automation, como se indica en [Administración de runbooks en Azure Automation](manage-runbooks.md).

### <a name="integration-modules"></a>Módulos de integración

No es posible exportar módulos de integración desde Azure Automation. Debe hacer que estén disponibles fuera de la cuenta de Automation.

### <a name="assets"></a>Recursos

No se pueden exportar recursos de Azure Automation: certificados, conexiones, credenciales, programaciones y variables. En su lugar, puede usar los cmdlets de Azure Portal y de Azure para anotar los detalles de estos recursos. Posteriormente, puede usar esos detalles para crear todos los recursos utilizados por los runbooks que importe a otra cuenta de Automation.

No es posible recuperar el valor de variables cifradas o del campo de contraseña de las credenciales mediante cmdlets. Si no conoce estos valores, puede recuperarlos en un runbook. Para recuperar valores de variables, consulte [Recursos de variables en Azure Automation](shared-resources/variables.md). Para más información sobre cómo recuperar valores de credenciales, consulte [Recursos de credenciales en Azure Automation](shared-resources/credentials.md).

### <a name="dsc-configurations"></a>Configuraciones DSC

Puede exportar las configuraciones de DSC a archivos de script con Azure Portal o con el cmdlet [Export-AzAutomationDscConfiguration](/powershell/module/az.automation/export-azautomationdscconfiguration?view=azps-3.7.0) en Windows PowerShell. Puede importar y usar estas configuraciones en otra cuenta de Automation.

## <a name="geo-replication-in-azure-automation"></a>Replicación geográfica en Azure Automation

La replicación geográfica es estándar en cuentas de Azure Automation. Puede elegir una región primaria al configurar su cuenta. El servicio de replicación geográfica interna de Automation asigna automáticamente una región secundaria a la cuenta. Después, el servicio realiza una copia de seguridad continua de los datos de la cuenta de la región primaria en la región secundaria. Puede encontrar la lista completa de las regiones primarias y secundarias en [Continuidad empresarial y recuperación ante desastres (BCDR): Regiones emparejadas de Azure](../best-practices-availability-paired-regions.md).

La copia de seguridad creada por el servicio de replicación geográfica de Automation es una copia completa de los recursos de Automation, las configuraciones, etc. Esta copia de seguridad se puede usar si la región primaria deja de funcionar y pierde datos. En el improbable caso de que se pierdan datos de una región primaria, Microsoft intenta recuperarlos. Si la empresa no puede recuperar los datos principales, utiliza la conmutación automática por error y le informa de la situación a través de su suscripción de Azure.

El servicio de replicación geográfica de Automation no es accesible directamente para los clientes externos si se produce un error regional. Si desea mantener la configuración de Automation y los runbooks durante errores regionales:

1. Seleccione una región secundaria para emparejar con la región geográfica de la cuenta de Automation principal.

2. Cree una cuenta de Automation en la región secundaria.

3. En la cuenta principal, exporte los runbooks como archivos de script.

4. Importe los runbooks a su cuenta de Automation en la región secundaria.

## <a name="next-steps"></a>Pasos siguientes

* Para más información acerca de los recursos seguros de Azure Automation, consulte [Cifrado de recursos protegidos en Azure Automation](automation-secure-asset-encryption.md).

* Para más información sobre la replicación geográfica, consulte [Creación y uso de la replicación geográfica activa](../azure-sql/database/active-geo-replication-overview.md).
