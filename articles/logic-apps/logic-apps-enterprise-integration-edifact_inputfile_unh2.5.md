---
title: Segmentos UNH 2.5 en los mensajes EDIFACT
description: Resolución de los mensajes EDIFACT con segmentos UNH 2.5 en Azure Logic Apps con Enterprise Integration Pack
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, logicappspm
ms.topic: article
ms.date: 04/27/2017
ms.openlocfilehash: c6d95cb938a341b94a32989e82c42d37956c0a80
ms.sourcegitcommit: 3fb5e772f8f4068cc6d91d9cde253065a7f265d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/31/2020
ms.locfileid: "89179851"
---
# <a name="handle-edifact-documents-with-unh25-segments-in-azure-logic-apps"></a>Control de documentos EDIFACT con segmentos UNH2.5 en Azure Logic Apps

Si existe un segmento UNH 2.5 en un documento EDIFACT, el segmento se utiliza para la búsqueda de esquemas. Por ejemplo, en este mensaje EDIFACT de ejemplo, el campo UNH es `EAN008`:

`UNH+SSDD1+ORDERS:D:03B:UN:EAN008`

Para administrar este mensaje, siga los pasos que se describen a continuación:

1. Actualice el esquema.

1. Compruebe la configuración del acuerdo.

## <a name="update-the-schema"></a>Revisión del esquema

Para procesar el mensaje, debe implementar un esquema con el nombre del nodo raíz UNH2.5. Por ejemplo, el nombre de la raíz del esquema para el campo UNH del ejemplo es `EFACT_D03B_ORDERS_EAN008`. Para cada `D03B_ORDERS` con un segmento UNH2.5 diferente, tiene que implementar un esquema distinto.

## <a name="add-schema-to-edifact-agreement"></a>Incorporación de un esquema al acuerdo EDIFACT

### <a name="edifact-decode"></a>Descodificación EDIFACT

Para descodificar el mensaje entrante, configure el esquema en la configuración de recepción del acuerdo EDIFACT:

1. En [Azure Portal](https://portal.azure.com), abra la cuenta de integración.

1. Agregue el esquema a la cuenta de integración.

1. Configure el esquema en la configuración de recepción del acuerdo EDIFACT.

1. Seleccione el acuerdo EDIFACT y seleccione **Editar como JSON**. Agregue el valor de UNH 2.5 a la sección `schemaReferences` del acuerdo de recepción:

   ![Incorporación de UNH 2.5 al acuerdo de recepción](./media/logic-apps-enterprise-integration-edifact_inputfile_unh2.5/image1.png)

### <a name="edifact-encode"></a>Codificación EDIFACT

Para codificar el mensaje entrante, configure el esquema en la configuración de envío del acuerdo EDIFACT

1. En [Azure Portal](https://portal.azure.com), abra la cuenta de integración.

1. Agregue el esquema a la cuenta de integración.

1. Configure el esquema en la configuración de envío del acuerdo EDIFACT.

1. Seleccione el acuerdo EDIFACT y seleccione **Editar como JSON**. Agregue el valor de UNH 2.5 a la sección `schemaReferences` del acuerdo de envío:

   ![Incorporación del valor de UNH 2.5 al acuerdo de envío](./media/logic-apps-enterprise-integration-edifact_inputfile_unh2.5/image2.png)

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre los [acuerdos de la cuenta de integración](../logic-apps/logic-apps-enterprise-integration-agreements.md)
