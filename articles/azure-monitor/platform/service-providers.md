---
title: Registros de Azure Monitor para proveedores de servicios | Microsoft Docs
description: Los registros de Azure Monitor pueden ayudar a proveedores de servicios administrados (MSP), grandes empresas, proveedores de software independientes (ISV) y proveedores de servicios de hospedaje a administrar y supervisar servidores en infraestructuras locales o en la nube del cliente.
ms.subservice: logs
ms.topic: conceptual
author: MeirMen
ms.author: meirm
ms.date: 02/03/2020
ms.openlocfilehash: 766fb9fbe50f8a138eae020082680204872a653a
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87315452"
---
# <a name="azure-monitor-logs-for-service-providers"></a>Registros de Azure Monitor para proveedores de servicios

Las áreas de trabajo de Log Analytics de Azure Monitor pueden ayudar a proveedores de servicios administrados (MSP), grandes empresas, proveedores de software independientes (ISV) y proveedores de servicios de hospedaje a administrar y supervisar servidores en infraestructuras locales o en la nube del cliente.

Las grandes empresas comparten muchas similitudes con los proveedores de servicios, especialmente cuando hay un equipo de TI centralizado que es responsable de la administración de TI de muchas unidades de negocio diferentes. Por motivos de simplicidad, este documento utiliza el término "*proveedor de servicios*", pero la misma funcionalidad también está disponible para empresas y otros clientes.

Para los asociados y proveedores de servicios que forman parte del programa [Proveedor de soluciones en la nube (CSP)](https://partner.microsoft.com/membership/cloud-solution-provider), Log Analytics de Azure Monitor es uno de los servicios de Azure disponibles en las suscripciones de CSP de Azure.

Los proveedores de servicios que administran recursos de clientes a través de la funcionalidad de administración de recursos delegados de Azure en [Azure Lighthouse](../../lighthouse/overview.md) también pueden usar Log Analytics de Azure Monitor.

## <a name="architectures-for-service-providers"></a>Arquitecturas de proveedores de servicios

Las áreas de trabajo de Log Analytics proporcionan un método para que el administrador controlen el flujo y el aislamiento de los datos de [registro](data-platform-logs.md) y cree una arquitectura que aborde sus necesidades empresariales específicas. En [este artículo](design-logs-deployment.md) se explican las consideraciones de diseño, implementación y migración de un área de trabajo, mientras que el artículo acerca de la[administración del acceso](manage-access.md) se describe cómo aplicar y administrar los permisos para los datos de registro. Los proveedores de servicios tienen consideraciones adicionales.

Hay tres arquitecturas posibles para proveedores de servicios con respecto a las áreas de trabajo de Log Analytics:

### <a name="1-distributed---logs-are-stored-in-workspaces-located-in-the-customers-tenant"></a>1. Distribuida: los registros se almacenan en áreas de trabajo que se encuentran en el inquilino del cliente

En esta arquitectura, el área de trabajo se implementa en el inquilino del cliente que se usa para todos los registros de ese cliente.

Los administradores de proveedores de servicios disponen de dos métodos para obtener acceso a un área de trabajo de Log Analytics en un inquilino del cliente:

- Un cliente puede agregar usuarios individuales desde el proveedor de servicios como [usuarios invitados de Azure Active Directory (B2B)](../../active-directory/b2b/what-is-b2b.md). Los administradores del proveedor de servicios tendrán que iniciar sesión en el directorio de cada cliente en Azure Portal para poder acceder a estas áreas de trabajo. Esto también requiere que los clientes administren el acceso individual de cada administrador del proveedor de servicios.
- Para mejorar la escalabilidad y la flexibilidad, los proveedores de servicios pueden usar la funcionalidad de [administración de recursos delegados de Azure](../../lighthouse/concepts/azure-delegated-resource-management.md) de [Azure Lighthouse](../../lighthouse/overview.md) para acceder al inquilino del cliente. Con este método, los administradores de proveedores de servicios se incluyen en un grupo de usuarios de Azure AD en el inquilino del proveedor de servicios. Se concede acceso a este grupo durante el proceso de incorporación de cada cliente. Posteriormente, estos administradores pueden acceder a las áreas de trabajo de cada cliente desde su propio inquilino del proveedor de servicios, en lugar de tener que iniciar sesión en el inquilino de cada cliente individualmente. Esta forma de acceder a los recursos de las áreas de trabajo de Log Analytics de los clientes reduce el trabajo que debe llevar a cabo el cliente y puede facilitar la recopilación y el análisis de los datos entre varios clientes administrados por el mismo proveedor de servicios a través de herramientas como [Workbooks de Azure Monitor](./workbooks-overview.md). Para obtener más información, consulte [Supervisión de los recursos del cliente a escala](../../lighthouse/how-to/monitor-at-scale.md).

Las ventajas de la arquitectura distribuida son las siguientes:

* El cliente puede confirmar niveles específicos de permisos a través de la [administración de recursos delegados de Azure](../../lighthouse/concepts/azure-delegated-resource-management.md), o bien administrar el acceso a los registros mediante su propio [acceso basado en roles](../../role-based-access-control/overview.md).
* Los registros se pueden recopilar en todos los tipos de recursos, no solo en datos de VM basados en agente. Por ejemplo, registros de auditoría de Azure.
* Cada cliente puede tener diferentes valores para su área de trabajo, como la retención y límite de datos.
* El aislamiento entre los clientes con respecto a las disposiciones legales y el cumplimiento.
* El cargo por cada área de trabajo se asignará a la suscripción del cliente.

Las desventajas de la arquitectura distribuida son las siguientes:

* La visualización y el análisis de datos de forma centralizada entre los inquilinos del cliente con herramientas como Workbooks de Azure Monitor pueden dar lugar a experiencias más lentas, especialmente al analizar datos en más de 50 áreas de trabajo.
* Si los clientes no se incorporan a la administración de recursos delegados de Azure, los administradores de proveedores de servicios deben aprovisionarse en el directorio del cliente, y será más difícil para el proveedor de servicios administrar un gran número de inquilinos de clientes a la vez.

### <a name="2-central---logs-are-stored-in-a-workspace-located-in-the-service-provider-tenant"></a>2. Central: los registros se almacenan en el área de trabajo que se encuentra en el inquilino del proveedor de servicios

En esta arquitectura, los registros no se almacenan en los inquilinos del cliente, sino solo en una ubicación central dentro de una de las suscripciones del proveedor de servicios. Los agentes instalados en máquinas virtuales del cliente se configuran para enviar sus registros a esta área de trabajo con el identificador de área de trabajo y la clave secreta.

Las ventajas de la arquitectura centralizada son las siguientes:

* Es fácil administrar un gran número de clientes e integrarlos en distintos sistemas back-end.
* El proveedor de servicios tiene una propiedad completa a través de los registros y los distintos artefactos, como son las funciones y las consultas guardadas.
* El proveedor de servicios puede realizar análisis en todos los clientes.

Las desventajas de la arquitectura centralizada son las siguientes:

* Esta arquitectura es aplicable únicamente para los datos de máquina virtual basados en un agente, no abarcará orígenes de datos del tejido de Azure, PaaS ni SaaS.
* Puede resultar dificultoso separar los datos entre los clientes cuando se mezclan en una sola área de trabajo. El único método adecuado para ello consiste en usar el nombre del equipo de dominio completo (FQDN) o a través del identificador de la suscripción de Azure.
* Todos los datos de todos los clientes se almacenarán en la misma región con una sola factura y las mismas opciones de configuración y de retención.
* Los servicios de PaaS y de tejido de Azure, como Azure Diagnostics y los registros de auditoría de Azure, requieren que el área de trabajo esté en el mismo inquilino que el recurso; por tanto, no pueden enviar los registros al área de trabajo central.
* Todos los agentes de VM desde todos los clientes se autenticarán en el área de trabajo central con el mismo identificador de área de trabajo y clave. No hay ningún método para bloquear los registros desde un cliente específico sin interrumpir a otros clientes.

### <a name="3-hybrid---logs-are-stored-in-workspace-located-in-the-customers-tenant-and-some-of-them-are-pulled-to-a-central-location"></a>3. Híbrido: los registros se almacenan en el área de trabajo que se encuentra en el inquilino del cliente y algunos se extraen a una ubicación central.

La tercera arquitectura es una mezcla de las dos opciones. Se basa en la primera arquitectura distribuida donde los registros son locales para cada cliente, pero con mecanismos para crear un repositorio central de registros. Una parte de los registros se extrae en una ubicación central para los informes y el análisis. Esta parte podría ser un número pequeño de tipos de datos o un resumen de la actividad, como la estadística diaria.

Hay dos opciones para implementar registros en una ubicación central:

1. Área de trabajo central: el proveedor de servicios puede crear un área de trabajo en su inquilino y usar un script que use [Query API](https://dev.loganalytics.io/) con [Data Collection API](./data-collector-api.md) para traer los datos de las diversas áreas de trabajo a esta ubicación central. Otra opción diferente del script es usar [Azure Logic Apps](../../logic-apps/logic-apps-overview.md).

2. Power BI como ubicación central: Power BI puede funcionar como ubicación central cuando las diversas áreas de trabajo exportan datos a él mediante la integración entre el área de trabajo de Log Analytics y [Power BI](./powerbi.md).

## <a name="next-steps"></a>Pasos siguientes

* Automatice la creación y configuración de áreas de trabajo con [plantillas de Resource Manager](template-workspace-configuration.md).

* Automatice la creación de áreas de trabajo con [PowerShell](./powershell-workspace-configuration.md).

* Use [alertas](./alerts-overview.md) para integrarse con sistemas existentes.

* Genere informes de resumen con [Power BI](./powerbi.md).

* Incorpore clientes a la [administración de recursos delegados de Azure](../../lighthouse/concepts/azure-delegated-resource-management.md).

