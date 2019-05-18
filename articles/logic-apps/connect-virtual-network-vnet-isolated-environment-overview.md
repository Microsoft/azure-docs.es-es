---
title: Acceso a instancias de Azure Virtual Network desde Azure Logic Apps mediante entornos de servicio de integración (ISE)
description: En esta introducción se describe cómo los entornos de servicio de integración (ISE) facilitan el acceso de las aplicaciones lógicas a las redes virtuales (VNET) de Azure.
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.author: estfan
ms.reviewer: klam, LADocs
ms.topic: article
ms.date: 05/06/2019
ms.openlocfilehash: 1ef8c8eec3865f2a6e363e7da1dbda9504b81c05
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65546431"
---
# <a name="access-to-azure-virtual-network-resources-from-azure-logic-apps-by-using-integration-service-environments-ises"></a>Acceso a recursos de Azure Virtual Network desde Azure Logic Apps mediante entornos de servicio de integración (ISE)

A veces, sus cuentas de integración y las aplicaciones lógicas necesitan acceso a los recursos protegidos, como máquinas virtuales (VM) y otros sistemas o servicios, que están dentro de un [red virtual de Azure](../virtual-network/virtual-networks-overview.md). Para configurar este acceso, puede [crear un *entorno del servicio de integración* (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment.md) donde puede ejecutar las aplicaciones lógicas y crear la integración de cuentas.

Cuando se crea una instancia de ISE, Azure implementa una instancia privada y aislada del servicio Logic Apps en su red virtual de Azure. La instancia privada usa recursos dedicados, como almacenamiento, y se ejecuta aparte del servicio Logic Apps público "global". La separación de la instancia privada aislada y la instancia global pública también ayuda a reducir el impacto que podrían tener otros inquilinos de Azure en el rendimiento de su aplicación, que también se conoce como el [efecto "vecinos ruidosos"](https://en.wikipedia.org/wiki/Cloud_computing_issues#Performance_interference_and_noisy_neighbors).

Después de crear el ISE, cuando vaya a crear la cuenta de integración o la aplicación lógica, puede seleccionar el ISE como ubicación de logic app o la integración de su cuenta:

![Selección del entorno de servicio de integración](./media/connect-virtual-network-vnet-isolated-environment-overview/select-logic-app-integration-service-environment.png)

La aplicación lógica ahora puede acceder directamente a los sistemas que están dentro de o conectan a la red virtual mediante cualquiera de estos elementos:

* Un **ISE**-connector para ese sistema, como SQL Server con la etiqueta
* Un **Core**-con la etiqueta integrada desencadenador o acción, como la acción o el desencadenador HTTP
* Un conector personalizado

En esta introducción se describe más detalles acerca de cómo las aplicaciones lógicas no proporciona un ISE e integración de cuentas de acceso directo a la red virtual de Azure y compara las diferencias entre una instancia de ISE y el servicio global de aplicaciones lógicas.

> [!NOTE]
> Logic apps, integrados de los desencadenadores, acciones integradas y conectores que se ejecutan en el uso ISE un plan de precios diferente desde el plan de precios basado en consumo. Para obtener más información, consulte [Precios de Logic Apps](../logic-apps/logic-apps-pricing.md).

<a name="difference"></a>

## <a name="isolated-versus-global"></a>Diferencias entre aislamiento y servicio global

Cuando se crea un entorno de servicio integrado (ISE) de Azure, puede seleccionar la red virtual de Azure donde desea *insertar* el ISE. Azure, a continuación, inserta o implementa una instancia privada del servicio Logic Apps en la red virtual. Esta acción crea un entorno aislado donde puede crear y ejecutar las aplicaciones lógicas en recursos dedicados. Cuando se crea la aplicación lógica, seleccione el ISE como ubicación de la aplicación, que proporciona el acceso directo de aplicación lógica a la red virtual y los recursos de esa red.

Las aplicaciones lógicas de una instancia de ISE proporcionan las mismas experiencias de usuario y funcionalidades similares que el servicio global de Logic Apps. No sólo puede usar el mismos desencadenadores integrados, acciones integradas y conectores desde el servicio global de aplicaciones lógicas, pero también puede utilizar conectores específicos de ISE. Por ejemplo, presentamos algunos conectores estándares que ofrecen las versiones que se ejecutan en una instancia de ISE:

* Azure Blob Storage, File Storage y Table Storage
* Azure Queues, Azure Service Bus, Azure Event Hubs e IBM MQ
* FTP y SFTP-SSH
* SQL Server, SQL Data Warehouse, Azure Cosmos DB
* AS2, X12 y EDIFACT

La diferencia entre los conectores de ISE y de otro tipo radica en las ubicaciones donde se ejecutan los desencadenadores y las acciones:

* En el ISE, integrados desencadenadores y acciones, como HTTP, siempre se ejecutan en el mismo ISE como la aplicación lógica y mostrar el **Core** etiqueta.

  ![Seleccione las acciones y desencadenadores integrados "Core"](./media/connect-virtual-network-vnet-isolated-environment-overview/select-core-built-in-actions-triggers.png)

* Los conectores que se ejecutan en una instancia de ISE públicamente hayan hospedado versiones disponibles en el servicio global de aplicaciones lógicas. Para los conectores que ofrecen dos versiones, los conectores con el **ISE** siempre etiquetar ejecución en el ISE mismo que la aplicación lógica. Los conectores sin la etiqueta **ISE** se ejecutan en el servicio global de Logic Apps.

  ![Selección de los conectores de ISE](./media/connect-virtual-network-vnet-isolated-environment-overview/select-ise-connectors.png)

### <a name="access-to-on-premises-data-sources"></a>Acceso a orígenes de datos locales

Para los sistemas locales que están conectados a una red virtual de Azure, insertar una instancia de ISE en esa red para las aplicaciones lógicas puedan acceder directamente a esos sistemas mediante cualquiera de estos elementos:

* Conector de versión de ISE para ese sistema, por ejemplo, SQL Server
  
* Acción HTTP
  
* Conector personalizado

  * Si tiene conectores personalizados que requieren la puerta de enlace de datos en el entorno local y crea los conectores fuera de una instancia de ISE, logic apps en una instancia de ISE también pueden usar los conectores.
  
  * Conectores personalizados creados en una instancia de ISE no funcionan con la puerta de enlace de datos en el entorno local. Sin embargo, estos conectores pueden acceder directamente a orígenes de datos locales que están conectados a la red virtual que hospeda el ISE. Por lo tanto, las aplicaciones lógicas en una instancia de ISE más probable es que no necesitan la puerta de enlace de datos al comunicarse con esos recursos.

Para los sistemas locales que no están conectados a una red virtual o no tienen los conectores de la versión de ISE, primero debe [configurar la puerta de enlace de datos local](../logic-apps/logic-apps-gateway-install.md) antes de la lógica de las aplicaciones pueden conectarse a esos sistemas.

<a name="create-integration-account-environment"></a>

## <a name="integration-accounts-with-ise"></a>Cuentas de integración con ISE

Puede usar cuentas de integración con aplicaciones lógicas dentro de un entorno de servicio de integración (ISE). Sin embargo, esas cuentas de integración deben usar el *mismo ISE* que las aplicaciones lógicas vinculadas. Las aplicaciones lógicas de una instancia de ISE solo pueden hacer referencia a aquellas cuentas de integración que se encuentran en la misma instancia de ISE. Cuando se crea una cuenta de integración, puede seleccionar la instancia de ISE como la ubicación de la cuenta de integración.

## <a name="next-steps"></a>Pasos siguientes

* Aprenda a [conectarse a las redes virtuales de Azure desde aplicaciones lógicas aisladas](../logic-apps/connect-virtual-network-vnet-isolated-environment.md).
* Consulte más información sobre [Azure Virtual Network](../virtual-network/virtual-networks-overview.md).
* Obtenga información sobre la [integración de redes virtuales para los servicios de Azure](../virtual-network/virtual-network-for-azure-services.md).
