---
title: Incorporación de recursos a entornos del servicio de integración
description: Agregue aplicaciones lógicas, cuentas de integración, conectores personalizados y conectores administrados al entorno del servicio de integración (ISE).
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: conceptual
ms.date: 02/28/2020
ms.openlocfilehash: d8fe945b028b574e71ae24b1483ad8e7fd67b01d
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87078726"
---
# <a name="add-resources-to-your-integration-service-environment-ise-in-azure-logic-apps"></a>Incorporación de recursos al entorno del servicio de integración (ISE) en Azure Logic Apps

Después de crear un [entorno del servicio de integración (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md), agregue recursos como aplicaciones lógicas, cuentas de integración y conectores para que puedan acceder a los recursos de la red virtual de Azure. Por ejemplo, los conectores ISE administrados que están disponibles después de crear el ISE no aparecen automáticamente en el selector de conectores en el Diseñador de aplicación lógica. Para poder usar estos conectores ISE, tiene que [agregarlos e implementarlos manualmente en su ISE](#add-ise-connectors-environment) para que aparezcan en el Diseñador de aplicación lógica.

> [!IMPORTANT]
> Para que las aplicaciones lógicas y las cuentas de integración funcionen juntas en un ISE, ambas deben usar *el mismo ISE* como ubicación.

## <a name="prerequisites"></a>Requisitos previos

* Suscripción a Azure. Si no tiene una suscripción de Azure, [regístrese para obtener una cuenta gratuita de Azure](https://azure.microsoft.com/free/).

* El ISE que creó para ejecutar las aplicaciones lógicas. Si no tiene un ISE, [cree primero uno](../logic-apps/connect-virtual-network-vnet-isolated-environment.md).

* Para crear, agregar o actualizar recursos que se implementen en un ISE, debe tener asignado el rol de propietario o colaborador en dicho ISE, o bien tener permisos heredados a través de la suscripción a Azure o del grupo de recursos de Azure asociado con el ISE. Para las personas que no tienen permisos de propietario, colaborador o heredados, se les puede asignar el rol de colaborador o desarrollador del entorno del servicio de integración. Para más información, consulte [¿Qué es el control de acceso basado en rol de Azure (RBAC)?](../role-based-access-control/overview.md)

<a name="create-logic-apps-environment"></a>

## <a name="create-logic-apps"></a>Creación de aplicaciones lógicas

Para crear aplicaciones lógicas que se ejecuten en el entorno del servicio de integración (ISE), siga estos pasos:

1. Busque el ISE y ábralo, si todavía no está abierto. En el menú de ISE, en **Configuración**, seleccione **Aplicaciones lógicas** > **Agregar**.

   ![Incorporación de una nueva aplicación lógica a ISE](./media/add-artifacts-integration-service-environment-ise/add-logic-app-to-ise.png)

1. Proporcione información sobre la aplicación lógica que desea crear, por ejemplo:

   ![Selección del entorno de servicio de integración](./media/add-artifacts-integration-service-environment-ise/create-logic-app-integration-service-environment.png)

   | Propiedad | Obligatorio | Descripción |
   |----------|----------|-------------|
   | **Nombre** | Sí | Nombre de la aplicación lógica que se va a crear |
   | **Suscripción** | Sí | Nombre de la suscripción a Azure que se va a usar |
   | **Grupos de recursos** | Sí | Nombre del grupo de recursos de Azure (nuevo o existente) que se va a usar |
   | **Ubicación** | Sí | En **entornos de servicio de integración**, seleccione el ISE que se va a usar, si aún no está seleccionado. <p><p> **Importante**: Para usar las aplicaciones lógicas con una cuenta de integración, ambos deben usar el mismo ISE. |
   ||||

1. Seleccione **Crear** cuando haya terminado.

1. Siga [creando la aplicación lógica de manera habitual](../logic-apps/quickstart-create-first-logic-app-workflow.md).

   Para conocer las diferencias de funcionamiento de los desencadenadores y las acciones y saber cómo se etiquetan cuando usa un ISE en comparación con el servicio multiinquilino de Logic Apps, consulte [Diferencias entre aislamiento y servicio multiinquilino en la información general del ISE](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md#difference).

1. Para administrar las aplicaciones lógicas y las conexiones de API en su ISE, consulte [Administración del entorno del servicio de integración](../logic-apps/ise-manage-integration-service-environment.md).

<a name="create-integration-account-environment"></a>

## <a name="create-integration-accounts"></a>Creación de cuentas de integración

Según la [SKU de ISE](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md#ise-level) que se seleccionó en la creación, el ISE incluye el uso de una cuenta de integración específica sin costo adicional. Las aplicaciones lógicas que existen en un entorno del servicio de integración (ISE) pueden hacer referencia solo a cuentas de integración que existan en el mismo ISE. Por lo tanto, para que una cuenta de integración trabaje con aplicaciones lógicas en un ISE, tanto la cuenta de integración como las aplicaciones lógicas deben usar el *mismo entorno* como ubicación. Para más información sobre las cuentas de integración y los ISE, consulte [Cuentas de integración con ISE](connect-virtual-network-vnet-isolated-environment-overview.md#create-integration-account-environment).

Para crear una cuenta de integración que use un ISE, siga estos pasos:

1. Busque el ISE y ábralo, si todavía no está abierto. En el menú de ISE, en **Configuración**, seleccione **Cuentas de integración** > **Agregar**.

   ![Agregar una cuenta de integración nueva a ISE](./media/add-artifacts-integration-service-environment-ise/add-integration-account-to-ise.png)

1. Proporcione información sobre la aplicación lógica que desea crear, por ejemplo:

   ![Selección del entorno de servicio de integración](./media/add-artifacts-integration-service-environment-ise/create-integration-account-integration-service-environment.png)

   | Propiedad | Obligatorio | Descripción |
   |----------|----------|-------------|
   | **Nombre** | Sí | Nombre de la cuenta de integración que quiere crear |
   | **Suscripción** | Sí | Nombre de la suscripción de Azure que quiere usar |
   | **Grupos de recursos** | Sí | Nombre del grupo de recursos de Azure (nuevo o existente) que se va a usar |
   | **Plan de tarifa** | Sí | Plan de tarifa que se va a usar para la cuenta de integración |
   | **Ubicación** | Sí | En **entornos de servicio de integración**, seleccione el mismo ISE que usan las aplicaciones lógicas, si aún no está seleccionado. <p><p> **Importante**: Para usar la cuenta de integración con aplicaciones lógicas, ambas deben usar el mismo ISE. |
   ||||

1. Seleccione **Crear** cuando haya terminado.

1. [Vincule la aplicación lógica a la cuenta de integración como siempre](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md#link-account).

1. Continúe agregando recursos a su cuenta de integración, como [entidades](../logic-apps/logic-apps-enterprise-integration-partners.md) y [contratos](../logic-apps/logic-apps-enterprise-integration-agreements.md).

1. Para administrar las cuentas de integración en su ISE, consulte [Administración del entorno del servicio de integración](../logic-apps/ise-manage-integration-service-environment.md).

<a name="add-ise-connectors-environment"></a>

## <a name="add-ise-connectors"></a>Adición de conectores del ISE

Los conectores administrados por Microsoft que están disponibles después de crear el ISE no aparecen automáticamente en el selector de conectores en el Diseñador de aplicación lógica. Para poder usar estos conectores ISE, tiene que agregarlos e implementarlos manualmente en su ISE para que aparezcan en el Diseñador de aplicación lógica.

1. En el menú de ISE, en **Configuración**, seleccione **Conectores administrados**. En la barra de herramientas, seleccione **Agregar**.

   ![Ver conectores administrados](./media/add-artifacts-integration-service-environment-ise/ise-view-managed-connectors.png)

1. En el panel **Agregar un nuevo conector administrado**, abra la lista **Buscar conector**. Seleccione el conector ISE que quiere usar, pero que aún no está implementado en su ISE. Seleccione **Crear**.

   ![Seleccione el conector ISE que quiere implementar en su ISE.](./media/add-artifacts-integration-service-environment-ise/add-managed-connector.png)

   Solo los conectores ISE que son aptos pero que todavía no se han implementado en su ISE aparecen disponibles para su selección. Los conectores que ya están implementados en el ISE aparecen como no disponibles para la selección.

<a name="create-custom-connectors-environment"></a>

## <a name="create-custom-connectors"></a>Creación de conectores personalizados

Para usar conectores personalizados en su ISE, créelos desde directamente dentro de su ISE.

1. Busque el ISE y ábralo, si todavía no está abierto. En el menú de ISE, en **Configuración**, seleccione **Conectores personalizados** > **Agregar**.

   ![Crear un conector personalizado](./media/add-artifacts-integration-service-environment-ise/add-custom-connector-to-ise.png)

1. Proporcione el nombre, la suscripción de Azure y el grupo de recursos de Azure (nuevo o existente) que se usará para el conector personalizado.

1. En la lista **Ubicación**, en la sección **Entornos de servicio de integración**, seleccione el mismo ISE que usan las aplicaciones lógicas y, después, seleccione **Crear**, por ejemplo:

   ![Selección del entorno de servicio de integración](./media/add-artifacts-integration-service-environment-ise/create-custom-connector-integration-service-environment.png)

1. Seleccione el nuevo conector personalizado y, a continuación, seleccione **Editar**, por ejemplo:

   ![Selección y edición del conector personalizado](./media/add-artifacts-integration-service-environment-ise/edit-custom-connectors.png)

1. Continúe creando el conector de la manera habitual a partir de [una definición de OpenAPI](/connectors/custom-connectors/define-openapi-definition#import-the-openapi-definition) o [SOAP](/connectors/custom-connectors/create-register-logic-apps-soap-connector#2-define-your-connector).

1. Para administrar los conectores personalizados en su ISE, consulte [Administración del entorno del servicio de integración](../logic-apps/ise-manage-integration-service-environment.md).

## <a name="next-steps"></a>Pasos siguientes

* [Administración de entornos del servicio de integración](../logic-apps/ise-manage-integration-service-environment.md)
