---
title: Codificación o descodificación de archivos planos
description: Codificación o descodificación de archivos planos para integración empresarial en Azure Logic Apps mediante Enterprise Integration Pack
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, logicappspm
ms.topic: article
ms.date: 05/09/2020
ms.openlocfilehash: aebce8f284ed4bb21d99efffc8dd6d0c51b39533
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87001492"
---
# <a name="encode-and-decode-flat-files-in-azure-logic-apps-by-using-the-enterprise-integration-pack"></a>Codificación y descodificación de archivos planos en Azure Logic Apps mediante Enterprise Integration Pack

Antes de enviar contenido XML a un socio comercial en un escenario de negocio a negocio (B2B), le recomendamos que primero codifique el contenido. Al compilar una aplicación lógica, puede codificar y descodificar archivos planos mediante el conector de **archivos planos**. La aplicación lógica puede obtener este contenido XML de diversos orígenes, como el desencadenador de solicitud, otra aplicación u otros [conectores compatibles con Azure Logic Apps](../connectors/apis-list.md). Para más información, consulte [¿Qué es Azure Logic Apps?](logic-apps-overview.md)

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción a Azure. Si aún no tiene una, [regístrese para obtener una cuenta de Azure gratuita](https://azure.microsoft.com/free/).

* La aplicación lógica en la que quiere usar el conector de **archivos planos** y un desencadenador que inicie el flujo de trabajo de la aplicación lógica. El conector de **archivos planos** proporciona únicamente las acciones, no los desencadenadores. Puede usar el desencadenador u otra acción para insertar el contenido XML en la aplicación lógica para la codificación o descodificación. Si es la primera vez que usa aplicaciones lógicas, revise [Inicio rápido: Creación de la primera aplicación lógica](../logic-apps/quickstart-create-first-logic-app-workflow.md).

* Una [cuenta de integración](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md) asociada con la suscripción a Azure y [vinculada a las aplicaciones lógicas](./logic-apps-enterprise-integration-create-integration-account.md#link-account) en las que planea usar el conector de **archivos planos**. Tanto la cuenta de integración como la aplicación de lógica deben existir en la misma ubicación o región de Azure.

* El [esquema](logic-apps-enterprise-integration-schemas.md) del archivo plano que ha cargado a la cuenta de integración para codificar o descodificar el contenido XML

* Al menos dos [socios comerciales](logic-apps-enterprise-integration-partners.md) que ya haya definido en su cuenta de integración

## <a name="add-flat-file-encode-action"></a>Agregar una acción de codificación de archivo plano

1. En [Azure Portal](https://portal.azure.com), abra la aplicación lógica en Diseñador de aplicación lógica.

1. En el desencadenador o la acción de su aplicación lógica, seleccione **Nuevo paso** > **Agregar una acción**. En este ejemplo se usa el desencadenador de solicitud, denominado **Cuando se recibe una solicitud HTTP** y controla las solicitudes entrantes desde fuera de la aplicación lógica.

   > [!TIP]
   > Proporcionar un esquema JSON es opcional. Si tiene una carga de ejemplo de la solicitud entrante, seleccione **Usar una carga de ejemplo para generar el esquema**, ingrese la carga de ejemplo y seleccione **Listo**. El esquema aparece en el cuadro **Esquema JSON del cuerpo de la solicitud**.

1. En **Elegir una acción**, escriba `flat file`. En la lista de acciones, seleccione esta acción: **Codificación de archivo plano**

   ![Seleccionar la acción "Codificación de archivos planos"](./media/logic-apps-enterprise-integration-flatfile/select-flat-file-encoding.png)

1. Haga clic en el cuadro **Contenido** para que aparezca la lista de contenido dinámico. En la lista, dentro de la sección **Cuando se recibe una solicitud HTTP**, seleccione la propiedad **Cuerpo**, que contiene la salida del cuerpo de la solicitud del desencadenador y el contenido que se va a codificar.

   ![Seleccionar contenido de la lista de contenido dinámico para codificarlo](./media/logic-apps-enterprise-integration-flatfile/select-content-to-encode.png)

   > [!TIP]
   > Si no ve la propiedad **Cuerpo** en la lista de contenido dinámico, seleccione **Ver más** junto a la etiqueta de la sección **Cuando se recibe una solicitud HTTP**.
   > También puede especificar directamente el contenido que desea descodificar en el cuadro **Contenido**.

1. En la lista **Nombre del esquema**, seleccione el esquema que se encuentra en la cuenta de integración vinculada para usarlo para la codificación, por ejemplo:

   ![Selección del esquema que se va a usar para la codificación](./media/logic-apps-enterprise-integration-flatfile/select-schema-for-encoding.png)

   > [!NOTE]
   > Si no aparece ningún esquema en la lista, la cuenta de integración no contiene ningún archivo de esquema que se pueda usar para la codificación. Cargue el esquema que desea usar a su cuenta de integración.

1. Guarde la aplicación lógica. Para probar el conector, realice una solicitud al punto de conexión HTTPS, que aparece en la propiedad **Dirección URL de HTTP POST** del desencadenador de solicitud e incluya el contenido XML que desea codificar en el cuerpo de la solicitud.

Ahora ya ha terminado de configurar la acción de codificación de archivos planos. En una aplicación real, podría almacenar los datos codificados en una aplicación de línea de negocio (LOB) como Salesforce. O bien, puede enviar los datos codificados a un socio comercial. Para enviar el resultado de la acción de codificación a Salesforce o a su socio comercial, use los otros [conectores disponibles en Azure Logic Apps](../connectors/apis-list.md).

## <a name="add-flat-file-decode-action"></a>Agregar una acción de descodificación de archivo plano

1. En [Azure Portal](https://portal.azure.com), abra la aplicación lógica en Diseñador de aplicación lógica.

1. En el desencadenador o la acción de su aplicación lógica, seleccione **Nuevo paso** > **Agregar una acción**. En este ejemplo se usa el desencadenador de solicitud, denominado **Cuando se recibe una solicitud HTTP** y controla las solicitudes entrantes desde fuera de la aplicación lógica.

   > [!TIP]
   > Proporcionar un esquema JSON es opcional. Si tiene una carga de ejemplo de la solicitud entrante, seleccione **Usar una carga de ejemplo para generar el esquema**, ingrese la carga de ejemplo y seleccione **Listo**. El esquema aparece en el cuadro **Esquema JSON del cuerpo de la solicitud**.

1. En **Elegir una acción**, escriba `flat file`. En la lista de acciones, seleccione esta acción: **Descodificación de archivo plano**

   ![Seleccionar la acción "Descodificación de archivo plano"](./media/logic-apps-enterprise-integration-flatfile/select-flat-file-decoding.png)

1. Haga clic en el cuadro **Contenido** para que aparezca la lista de contenido dinámico. En la lista, dentro de la sección **Cuando se recibe una solicitud HTTP**, seleccione la propiedad **Cuerpo**, que contiene la salida del cuerpo de la solicitud del desencadenador y el contenido que se va a descodificar.

   ![Seleccionar contenido de la lista de contenido dinámico para descodificarlo](./media/logic-apps-enterprise-integration-flatfile/select-content-to-decode.png)

   > [!TIP]
   > Si no ve la propiedad **Cuerpo** en la lista de contenido dinámico, seleccione **Ver más** junto a la etiqueta de la sección **Cuando se recibe una solicitud HTTP**. También puede especificar directamente el contenido que desea descodificar en el cuadro **Contenido**.

1. En la lista **Nombre del esquema**, seleccione el esquema que se encuentra en la cuenta de integración vinculada para usarlo para la descodificación, por ejemplo:

   ![Selección del esquema que se va a usar para la descodificación](./media/logic-apps-enterprise-integration-flatfile/select-schema-for-decoding.png)

   > [!NOTE]
   > Si no aparece ningún esquema en la lista, la cuenta de integración no contiene ningún archivo de esquema que se pueda usar para la descodificación. Cargue el esquema que desea usar a su cuenta de integración.

1. Guarde la aplicación lógica. Para probar el conector, realice una solicitud al punto de conexión HTTPS, que aparece en la propiedad **Dirección URL de HTTP POST** del desencadenador de solicitud e incluya el contenido XML que desea descodificar en el cuerpo de la solicitud.

Ahora ya ha terminado de configurar la acción de descodificación de archivos planos. En una aplicación real, podría almacenar los datos descodificados en una aplicación de línea de negocio (LOB) como Salesforce. O bien, puede enviar los datos descodificados a un socio comercial. Para enviar el resultado de la acción de descodificación a Salesforce o a su socio comercial, use los otros [conectores disponibles en Azure Logic Apps](../connectors/apis-list.md).

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información acerca de [Enterprise Integration Pack](logic-apps-enterprise-integration-overview.md)
