---
title: Mensajes RosettaNet para la integración B2B
description: Intercambiar mensajes de RosettaNet en Azure Logic Apps con Enterprise Integration Pack
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, logicappspm
ms.topic: article
ms.date: 06/22/2019
ms.openlocfilehash: 833e2a6f64301c39ba2b9e7959585c3edeab9445
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87072582"
---
# <a name="exchange-rosettanet-messages-for-b2b-enterprise-integration-in-azure-logic-apps"></a>Intercambiar mensajes de RosettaNet para la integración empresarial B2B en Azure Logic Apps

[RosettaNet](https://resources.gs1us.org) es un consorcio sin ánimo de lucro que ha establecido procesos estándar para compartir información comercial. Estos estándares se usan habitualmente en procesos de la cadena de suministro y están muy extendidos en el sector de los semiconductores, la electrónica y la logística. El consorcio RosettaNet crea y mantiene procesos de interfaz de asociado (PIP), que proporcionan definiciones de procesos comerciales comunes para toda la mensajería de RosettaNet. RosettaNet se basa en el lenguaje XML y define directrices de mensajería, interfaces para procesos comerciales y marcos de implementación para la comunicación entre compañías.

En [Azure Logic Apps](../logic-apps/logic-apps-overview.md), el conector de RosettaNet le ayuda a crear soluciones de integración compatibles con los estándares de RosettaNet. El conector se basa en la versión 2.0.01 de RosettaNet Implementation Framework (RNIF). RNIF es un marco de aplicación de red abierta que permite a los asociados de negocio ejecutar en colaboración los PIP de RosettaNet. Este marco de trabajo define la estructura del mensaje, la necesidad de confirmaciones, la codificación de extensiones multipropósito de correo Internet (MIME) y la firma digital.

En concreto, el conector proporciona estas funcionalidades:

* Codificar o recibir mensajes de RosettaNet.
* Descodificar o enviar mensajes de RosettaNet.
* Esperar la respuesta y la generación de una notificación de error.

Para estas funcionalidades, el conector admite todos los PIP que se define mediante RNIF 2.0.01. La comunicación con el asociado puede ser sincrónica o asincrónica.

## <a name="rosettanet-concepts"></a>Conceptos de RosettaNet

Estos son algunos conceptos y términos que son únicos para la especificación de RosettaNet y son importantes al crear integraciones basadas en RosettaNet:

* **PIP**

  La organización RosettaNet crea y mantiene procesos de interfaz de asociado (PIP), que proporcionan definiciones de procesos comerciales comunes para toda la mensajería de RosettaNet. Cada especificación de PIP proporciona un archivo de definición de tipo de documento (DTD) y un documento de guía de mensajes. El archivo DTD define la estructura del mensaje de contenido del servicio. El documento de guía de mensajes, que es un archivo HTML en lenguaje natural, especifica las restricciones a nivel de elemento. Juntos, estos archivos proporcionan una definición completa del proceso empresarial.

   Los PIP se clasifican por una función empresarial de alto nivel, o clúster, y una subfunción, o segmento. Por ejemplo, "3A4" es el PIP para el pedido de compra, aunque "3" es la función de administración de pedidos, y "3A" es la subfunción de cotización y entrada de pedidos. Para más información, consulte el [sitio de RosettaNet](https://resources.gs1us.org).

* **Acción**

  Los mensajes de acción, que son parte de un PIP, son mensajes empresariales que intercambian los asociados.

* **Señal**

   Los mensajes de señal, que son parte de un PIP, son confirmaciones que se envían en respuesta a los mensajes de acción.

* **Acción única y acción doble**

  Para un PIP acción única, la única respuesta es un mensaje de señal de confirmación. Para un PIP de doble acción, el iniciador recibe un mensaje de respuesta y responde con una confirmación, además del flujo de mensajes de acción única.

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción a Azure. Si aún no tiene ninguna suscripción de Azure, [regístrese para obtener una cuenta gratuita de Azure](https://azure.microsoft.com/free/).

* Una [cuenta de integración](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md) para almacenar el contrato y otros artefactos B2B. Esta cuenta de integración debe estar asociada con una suscripción a Azure.

* Al menos dos [asociados](../logic-apps/logic-apps-enterprise-integration-partners.md) que estén definidos en su cuenta de integración y configurados con el calificador "DUNS" en **Identidades de negocio**.

* Una configuración de procesos PIP, que es necesaria para enviar o recibir mensajes de RosettaNet, en la cuenta de integración. La configuración de procesos almacena todas las características de configuración de PIP. Luego, puede hacer referencia a esta configuración al crear un acuerdo con el asociado. Para crear una configuración de procesos PIP en su cuenta de integración, consulte [Agregar una configuración de procesos PIP](#add-pip).

* [Certificados](../logic-apps/logic-apps-enterprise-integration-certificates.md) opcionales para cifrar, descifrar o firmar los mensajes que carga en la cuenta de integración. Los certificados son necesarios solo si usa firmas o cifrado.

<a name="add-pip"></a>

## <a name="add-pip-process-configuration"></a>Agregar una configuración de procesos PIP

Para agregar una configuración de procesos PIP a la cuenta de integración, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com), busque y abra la cuenta de integración.

1. En el panel **Información general**, seleccione el icono **PIP de RosettaNet**.

   ![Elegir el icono de RosettaNet](media/logic-apps-enterprise-integration-rosettanet/select-rosettanet-tile.png)

1. En **RosettaNet PIP**, elija **Agregar**. Proporcione sus detalles de PIP.

   ![Agregar detalles de PIP de RosettaNet](media/logic-apps-enterprise-integration-rosettanet/add-rosettanet-pip.png)

   | Propiedad | Obligatorio | Descripción |
   |----------|----------|-------------|
   | **Nombre** | Sí | El nombre de PIP |
   | **Código de PIP** | Sí | El código de PIP de tres dígitos. Para más información, consulte [PIP de RosettaNet](/biztalk/adapters-and-accelerators/accelerator-rosettanet/rosettanet-pips). |
   | **Versión de PIP** | Sí | El número de versión de PIP, que está disponible en función del código de PIP seleccionado. |
   ||||

   Para más información sobre estas propiedades de PIP, visite el [sitio web de RosettaNet](https://resources.gs1us.org/RosettaNet-Standards/Standards-Library/PIP-Directory#1043208-pipsreg).

1. Cuando haya terminado, elija **Aceptar**, lo que crea la configuración de PIP.

1. Para ver o editar la configuración de procesos, seleccione el PIP y elija **Editar como JSON**.

   Todos los valores de configuración de procesos provienen de las especificaciones de PIP. Logic Apps rellena la mayoría de los valores con los valores predeterminados, que son los valores usados más normalmente con estas propiedades.

   ![Editar la configuración de PIP de RosettaNet](media/logic-apps-enterprise-integration-rosettanet/edit-rosettanet-pip.png)

1. Confirme que la configuración se corresponda con los valores de la especificación de PIP adecuada y satisfaga sus necesidades empresariales. Si es necesario, actualice los valores de JSON y guarde esos cambios.

## <a name="create-rosettanet-agreement"></a>Crear un acuerdo de RosettaNet

1. En [Azure Portal](https://portal.azure.com), busque y abra la cuenta de integración, si todavía no está abierto.

1. Seleccione el panel **Información general** y el icono **Acuerdos**.

   ![Elegir el icono Acuerdos](media/logic-apps-enterprise-integration-rosettanet/select-agreement-tile.png)

1. En **Contratos**, elija **Agregar**. Proporcione los detalles del acuerdo.

   ![Agregar detalles del acuerdo](media/logic-apps-enterprise-integration-rosettanet/add-agreement-details.png)

   | Propiedad | Obligatorio | Descripción |
   |----------|----------|-------------|
   | **Nombre** | Sí | Nombre del acuerdo |
   | **Tipo de contrato** | Sí | Seleccione **RosettaNet**. |
   | **Asociado host** | Sí | Un contrato requiere un asociado anfitrión y uno invitado. El asociado del host representa la organización que configura el contrato. |
   | **Identidad del host** | Sí | Un identificador del asociado del host. |
   | **Asociado invitado** | Sí | Un contrato requiere un asociado anfitrión y uno invitado. El partner invitado representa la organización que está haciendo negocios con el partner anfitrión. |
   | **Identidad del invitado** | Sí | Un identificador del asociado invitado. |
   | **Configuración de recepción** | Varía | Estas propiedades se aplican a todos los mensajes que recibe un asociado host. |
   | **Configuración de envío** | Varía | Estas propiedades se aplican a todos los mensajes que envía un asociado host. |  
   | **Referencias de PIP de RosettaNet** | Sí | Las referencias de PIP para el acuerdo. Todos los mensajes de RosettaNet requieren configuraciones de PIP. |
   ||||

1. Para configurar el acuerdo para recibir mensajes entrantes del asociado invitado, seleccione **Configuración de recepción**.

   ![Configuración de recepción](media/logic-apps-enterprise-integration-rosettanet/add-agreement-receive-details.png)

   1. Para habilitar la firma o cifrado para los mensajes entrantes, en **Mensajes**, seleccione **Debe firmarse el mensaje** o **Debe cifrarse el mensaje**, respectivamente.

      | Propiedad | Obligatorio | Descripción |
      |----------|----------|-------------|
      | **Debe firmarse el mensaje** | No | Firme los mensajes entrantes con el certificado seleccionado. |
      | **Certificate** | Sí, si la firma está habilitada | El certificado que se va a usar para firmar |
      | **Habilitar el cifrado de mensajes** | No | Cifre los mensajes entrantes con el certificado seleccionado. |
      | **Certificate** | Sí, si el cifrado está habilitado | El certificado que se usará para el cifrado |
      ||||

   1. En cada selección, seleccione el [certificado](./logic-apps-enterprise-integration-certificates.md) correspondiente, que agregó anteriormente a la cuenta de integración, que se usará para la firma o el cifrado.

1. Para configurar el acuerdo para enviar mensajes al asociado invitado, seleccione **Configuración de envío**.

   ![Configuración de envío](media/logic-apps-enterprise-integration-rosettanet/add-agreement-send-details.png)

   1. Para habilitar la firma o el cifrado para los mensajes salientes, en **Mensajes**, seleccione **Habilitar la firma de mensajes** o **Habilitar el cifrado de mensajes**, respectivamente. En cada selección, seleccione el algoritmo y el [certificado](./logic-apps-enterprise-integration-certificates.md) correspondientes, que agregó anteriormente a la cuenta de integración, que se usarán para la firma o el cifrado.

      | Propiedad | Obligatorio | Descripción |
      |----------|----------|-------------|
      | **Habilitar la firma de mensajes** | No | Firme los mensajes salientes con el algoritmo y el certificado de firma seleccionado. |
      | **Algoritmo de firma** | Sí, si la firma está habilitada | El algoritmo de firma para usar, según el certificado seleccionado |
      | **Certificate** | Sí, si la firma está habilitada | El certificado que se va a usar para firmar |
      | **Habilitar el cifrado de mensajes** | No | Cifre los mensajes salientes con el algoritmo y el certificado de cifrado seleccionados. |
      | **Algoritmo de cifrado** | Sí, si el cifrado está habilitado | El algoritmo de cifrado para usar, según el certificado seleccionado |
      | **Certificate** | Sí, si el cifrado está habilitado | El certificado que se usará para el cifrado |
      ||||

   1. En **Puntos de conexión**, especifique las direcciones URL necesarias para enviar mensajes de acción y confirmaciones.

      | Propiedad | Obligatorio | Descripción |
      |----------|----------|-------------|
      | **Dirección URL de acción** |  Sí | La dirección URL que se usará para enviar mensajes de acción. La dirección URL es un campo obligatorio tanto para los mensajes sincrónicos como asincrónicos. |
      | **Acknowledgment URL** (Dirección URL de confirmación) | Sí | La dirección URL que se usará para enviar mensajes de confirmación. La dirección URL es un campo obligatorio para los mensajes asincrónicos. |
      ||||

1. Para configurar el acuerdo con las referencias de PIP de RosettaNet para los asociados, seleccione **Referencias de PIP de RosettaNet**. En **Nombre de PIP**, seleccione el nombre del PIP creado anteriormente.

   ![Referencias de PIP](media/logic-apps-enterprise-integration-rosettanet/add-agreement-pip-details.png)

   La selección rellena las propiedades restantes, que se basan en el PIP que configuró en la cuenta de integración. Si es necesario, puede cambiar el **Rol de PIP**.

   ![PIP seleccionado](media/logic-apps-enterprise-integration-rosettanet/add-agreement-selected-pip.png)

Después de completar estos pasos, estará listo para enviar o recibir mensajes de RosettaNet.

## <a name="rosettanet-templates"></a>Plantillas de RosettaNet

Para acelerar el desarrollo y recomendar patrones de integración, puede usar plantillas de aplicación lógica para la descodificación y codificación de mensajes de RosettaNet. Al crear una aplicación lógica, puede seleccionar desde la galería de plantillas en el Diseñador de aplicación lógica. También puede encontrar estas plantillas en el [repositorio de GitHub para Azure Logic Apps](https://github.com/Azure/logicapps).

![Plantillas de RosettaNet](media/logic-apps-enterprise-integration-rosettanet/decode-encode-rosettanet-templates.png)

## <a name="receive-or-decode-rosettanet-messages"></a>Recibir o descodificar mensajes de RosettaNet

1. [Cree una aplicación lógica en blanco](quickstart-create-first-logic-app-workflow.md).

1. [Vincule la cuenta de integración](logic-apps-enterprise-integration-create-integration-account.md#link-account) a la aplicación lógica.

1. Antes de poder agregar una acción para descodificar el mensaje de RosettaNet, debe agregar un desencadenador para iniciar la aplicación lógica, como un desencadenador de solicitud.

1. Después de agregar el desencadenador, elija **Nuevo paso**.

   ![Agregar un desencadenador de solicitud](media/logic-apps-enterprise-integration-rosettanet/request-trigger.png)

1. En el cuadro de búsqueda, escriba "rosettanet" y seleccione esta acción: **Descodificación de RosettaNet**

   ![Busque y seleccione la acción "Descodificación de RosettaNet"](media/logic-apps-enterprise-integration-rosettanet/select-decode-rosettanet-action.png)

1. Proporcione la información para las propiedades de la acción:

   ![Proporcione los detalles de la acción](media/logic-apps-enterprise-integration-rosettanet/decode-action-details.png)

   | Propiedad | Obligatorio | Descripción |
   |----------|----------|-------------|
   | **Mensaje** | Sí | El mensaje de RosettaNet para descodificar.  |
   | **Encabezados** | Sí | Los encabezados HTTP que proporcionan los valores de la versión, que es la versión de RNIF, y el tipo de respuesta, que indica el tipo de comunicación entre los asociados y que puede ser sincrónico o asincrónico. |
   | **Rol** | Sí | El rol de asociado del host en el PIP. |
   ||||

   Desde la acción de descodificación de RosettaNet, la salida, junto con otras propiedades, incluye **Outbound signal** (Señal saliente), que puede elegir para codificar y devolver al asociado, o realizar cualquier otra acción en esos resultados.

## <a name="send-or-encode-rosettanet-messages"></a>Enviar o codificar mensajes de RosettaNet

1. [Cree una aplicación lógica en blanco](quickstart-create-first-logic-app-workflow.md).

1. [Vincule la cuenta de integración](logic-apps-enterprise-integration-create-integration-account.md#link-account) a la aplicación lógica.

1. Antes de poder agregar una acción para codificar el mensaje de RosettaNet, debe agregar un desencadenador para iniciar la aplicación lógica, como un desencadenador de solicitud.

1. Después de agregar el desencadenador, elija **Nuevo paso**.

   ![Agregar un desencadenador de solicitud](media/logic-apps-enterprise-integration-rosettanet/request-trigger.png)

1. En el cuadro de búsqueda, escriba "rosettanet" y seleccione esta acción: **Codificación RosettaNet**

   ![Busque y seleccione la acción "Codificación RosettaNet"](media/logic-apps-enterprise-integration-rosettanet/select-encode-rosettanet-action.png)

1. Proporcione la información para las propiedades de la acción:

   ![Proporcione los detalles de la acción](media/logic-apps-enterprise-integration-rosettanet/encode-action-details.png)

   | Propiedad | Obligatorio | Descripción |
   |----------|----------|-------------|
   | **Mensaje** | Sí | El mensaje de RosettaNet para codificar.  |
   | **Asociado host** | Sí | El nombre del asociado host. |
   | **Asociado invitado** | Sí | El nombre del asociado invitado. |
   | **Código de PIP** | Sí | El código de PIP. |
   | **Versión de PIP** | Sí | La versión de PIP. |  
   | **Identidad de instancia de PIP** | Sí | El identificador único de este mensaje de PIP. |  
   | **Tipo de mensaje** | Sí | El tipo de mensaje en para codificar. |  
   | **Rol** | Sí | El rol del asociado host. |
   ||||

   El mensaje codificado ahora está listo para enviarse al asociado.

1. Para enviar el mensaje codificado, en este ejemplo se usa la acción **HTTP**, que se cambió de nombre a "HTTP - Send encoded message to partner".

   ![Acción de HTTP para enviar un mensaje de RosettaNet](media/logic-apps-enterprise-integration-rosettanet/send-rosettanet-message-to-partner.png)

   Según los estándares de RosettaNet, las transacciones comerciales se consideran completas únicamente cuando se han completado todos los pasos definidos por el PIP.

1. Una vez que el host envía el mensaje codificado al asociado, el host espera la señal y la confirmación. Para lograr esta tarea, agregue la acción **Espera de respuesta de RosettaNet**.

   ![Agregar la acción "Espera de respuesta de RosettaNet"](media/logic-apps-enterprise-integration-rosettanet/rosettanet-wait-for-response-action.png)

   La duración que se usará para la espera y el número de reintentos se basan en la configuración de PIP en su cuenta de integración. Si no se recibe la respuesta, esta acción genera una notificación de error. Para controlar los reintentos, coloque siempre las acciones **Codificación** y **Espera de respuesta** en un bucle **Hasta**.

   ![Bucle Hasta con acciones de RosettaNet](media/logic-apps-enterprise-integration-rosettanet/rosettanet-loop.png)

## <a name="next-steps"></a>Pasos siguientes

* Obtenga información acerca de cómo validar, transformar y realizar otras operaciones de mensaje con [Enterprise Integration Pack](../logic-apps/logic-apps-enterprise-integration-overview.md).
* Obtenga más información sobre otros [conectores de Logic Apps](../connectors/apis-list.md)
