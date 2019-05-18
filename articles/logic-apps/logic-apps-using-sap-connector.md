---
title: 'Conectarse a sistemas SAP: Azure Logic Apps'
description: Obtener acceso y administrar recursos de SAP mediante la automatización de flujos de trabajo con Azure Logic Apps
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.author: estfan
ms.reviewer: divswa, LADocs
ms.topic: article
ms.date: 05/09/2019
tags: connectors
ms.openlocfilehash: 3fb39103fc9cb0f38bca56dcaeea4837ff4dfabe
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65541156"
---
# <a name="connect-to-sap-systems-from-azure-logic-apps"></a>Conexión a sistemas SAP desde Azure Logic Apps

En este artículo se muestra cómo tener acceso a los recursos SAP de forma local desde dentro de una aplicación lógica mediante el conector SAP. El conector funciona con las versiones clásicas de SAP, tales como los sistemas R/3 y ECC en local. El conector también permite la integración con sistemas SAP HANA basada más recientes de SAP como S/4 HANA, siempre que se hospedan - local o en la nube. El conector SAP admite la integración de mensajes o datos hacia y desde los sistemas basados en SAP NetWeaver a través de un documento intermedios (IDoc) o interfaz de programación de aplicación empresariales (BAPI) o llamar a funciones remotas (RFC).

El conector SAP usa la [biblioteca del conector SAP para .NET (NCo)](https://support.sap.com/en/product/connectors/msnet.html) y proporciona estas operaciones o acciones:

* **Send to SAP** (Enviar a SAP): Enviar IDoc a través de tRFC, llamar a funciones BAPI a través de la RFC o llamar a RFC/tRFC de sistemas SAP.
* **Receive from SAP** (Recibir de SAP): Recibir IDoc a través de tRFC, llamar a funciones BAPI sobre tRFC o llamar a RFC/tRFC de sistemas SAP.
* **Generate schemas** (Generar esquemas): Generar esquemas para los artefactos SAP para RFC, BAPI o IDoc.

Para todas las operaciones anteriores, el conector SAP admite la autenticación básica a través del nombre de usuario y la contraseña. El conector también admite [las comunicaciones de red seguro (SNC)](https://help.sap.com/doc/saphelp_nw70/7.0.31/e6/56f466e99a11d1a5b00000e835363f/content.htm?no_cache=true), que puede utilizarse para SAP NetWeaver Single Sign-On o para las capacidades de seguridad adicional proporcionadas por un producto de seguridad externas.

El conector SAP se integra con los sistemas SAP locales a través de la [puerta de enlace de datos local](../logic-apps/logic-apps-gateway-connection.md). En escenarios de envío, por ejemplo, al enviar un mensaje desde aplicaciones lógicas a un sistema SAP, la puerta de enlace de datos actúa como un cliente RFC y reenvía las solicitudes recibidas desde aplicaciones lógicas a SAP.
Del mismo modo, en escenarios de recepción, la puerta de enlace de datos actúa como un servidor RFC que recibe solicitudes de SAP y la reenvía a la aplicación lógica.

En este artículo se muestra cómo crear aplicaciones lógicas que se integren con SAP y se tratan los escenarios de integración descritos anteriormente.

<a name="pre-reqs"></a>

## <a name="prerequisites"></a>Requisitos previos

Para seguir con este artículo, necesita los siguientes elementos:

* Una suscripción de Azure. Si aún no tiene ninguna suscripción de Azure, [regístrese para obtener una cuenta gratuita de Azure](https://azure.microsoft.com/free/).

* La aplicación lógica desde donde quiere obtener acceso al sistema SAP y un desencadenador que inicie el flujo de trabajo de la aplicación lógica. Si no está familiarizado con las aplicaciones lógicas, consulte [¿Qué es Azure Logic Apps?](../logic-apps/logic-apps-overview.md) e [Inicio rápido: Creación de la primera aplicación lógica](../logic-apps/quickstart-create-first-logic-app-workflow.md).

* El [servidor de aplicaciones de SAP](https://wiki.scn.sap.com/wiki/display/ABAP/ABAP+Application+Server) o el [servidor de mensajes de SAP](https://help.sap.com/saphelp_nw70/helpdata/en/40/c235c15ab7468bb31599cc759179ef/frameset.htm)

* Descargue e instale la versión más reciente de la [puerta de enlace de datos local](https://www.microsoft.com/download/details.aspx?id=53127) en cualquier equipo local. Asegúrese de configurar la puerta de enlace en Azure Portal antes de continuar. La puerta de enlace le segura acceder a los datos locales y recursos. Para obtener más información, consulte [Instalación de la puerta de enlace de datos local para Azure Logic Apps](../logic-apps/logic-apps-gateway-install.md).

* Si usas SNC con Single Sign-On (SSO), asegúrese de que la puerta de enlace se ejecuta como un usuario que se asigna en el usuario SAP. Para cambiar la cuenta predeterminada, seleccione **cambiar cuenta**y escriba las credenciales de usuario.

  ![Cambiar la cuenta de puerta de enlace](./media/logic-apps-using-sap-connector/gateway-account.png)

* Si va a habilitar SNC con un producto de seguridad externas, copie la biblioteca SNC o archivos en el mismo equipo donde está instalada la puerta de enlace. Algunos ejemplos de productos SNC [sapseculib](https://help.sap.com/saphelp_nw74/helpdata/en/7a/0755dc6ef84f76890a77ad6eb13b13/frameset.htm), Kerberos, NTLM, y así sucesivamente.

* Descargue e instale la biblioteca de cliente SAP más reciente, que es actualmente [SAP Connector (NCo) 3.0.21.0 para Microsoft .NET Framework 4.0 y Windows 64 bits (x64)](https://softwaredownloads.sap.com/file/0020000001865512018), en el mismo equipo que la puerta de enlace de datos en el entorno local. Debe instalar esta versión o una versión posterior por estos motivos:

  * Las versiones anteriores de NCo de SAP pueden interbloquearse cuando se envía más de un mensaje IDoc al mismo tiempo. Esta condición bloquea todos los mensajes posteriores que se envían al destino de SAP, haciendo que se agote el tiempo de espera de los mensajes.

  * La puerta de enlace de datos local solo se ejecuta en sistemas de 64 bits. De lo contrario, obtendrá un error de "imagen incorrecta" porque el servicio de host de la puerta de enlace de datos no es compatible con los ensamblados de 32 bits.

  * El servicio de host de la puerta de enlace de datos y el adaptador de SAP de Microsoft usan .NET Framework 4.5. NCo de SAP para .NET Framework 4.0 funciona con procesos que usan las versiones 4.0 a 4.7.1 del runtime de .NET. NCo de SAP para .NET Framework 2.0 funciona con procesos que usan las versiones 2.0 a 3.5 del runtime de .NET y ya no funciona con la puerta de enlace de datos local más reciente.

* Contenido del mensaje que puede enviar al servidor SAP, como un archivo de IDoc de ejemplo. Este contenido debe estar en formato XML e incluir el espacio de nombres para la acción de SAP que se va a utilizar.

<a name="add-trigger"></a>

## <a name="send-to-sap"></a>Enviar a SAP

En este ejemplo, se utiliza una aplicación lógica que se puede desencadenar con una solicitud HTTP. La aplicación lógica envía un documento intermedio (IDOC) a un servidor SAP y devuelve una respuesta al solicitante que llamó a la aplicación lógica. 

### <a name="add-http-request-trigger"></a>Adición de un desencadenador de solicitud HTTP

En Azure Logic Apps, cada aplicación lógica debe comenzar con un [desencadenador](../logic-apps/logic-apps-overview.md#logic-app-concepts), que se activa cuando sucede un evento específico o cuando se cumple una condición determinada. Cada vez que el desencadenador se activa, el motor de Logic Apps crea una instancia de aplicación lógica y empieza a ejecutar el flujo de trabajo de la aplicación.

En este ejemplo, cree una aplicación lógica con un punto de conexión en Azure para poder enviar *solicitudes HTTP POST* a la aplicación lógica. Cuando la aplicación lógica reciba estas solicitudes HTTP, el desencadenador se activará y ejecutará el paso siguiente en el flujo de trabajo.

1. En [Azure Portal](https://portal.azure.com), cree una aplicación lógica en blanco que abra el Diseñador de aplicación lógica.

1. En el cuadro de búsqueda, escriba "solicitud http" para el filtro. En la lista de desencadenadores, seleccione este desencadenador: **Cuando se recibe una solicitud HTTP**

   ![Adición de un desencadenador de solicitud HTTP](./media/logic-apps-using-sap-connector/add-trigger.png)

1. Ahora puede guardar la aplicación lógica para poder generar una dirección URL del punto de conexión para la aplicación lógica. En la barra de herramientas del diseñador, haga clic en **Guardar**.

   La dirección URL del punto de conexión aparece ahora en el desencadenador, por ejemplo:

   ![Generación de una dirección URL para el punto de conexión](./media/logic-apps-using-sap-connector/generate-http-endpoint-url.png)

<a name="add-action"></a>

### <a name="add-sap-action"></a>Adición de una acción de SAP

En Azure Logic Apps, una [acción](../logic-apps/logic-apps-overview.md#logic-app-concepts) es un paso del flujo de trabajo que sigue a un desencadenador u otra acción. Si aún no ha agregado un desencadenador a la aplicación lógica y quiere seguir este ejemplo, [agregue el desencadenador que se describe en esta sección](#add-trigger).

1. En el Diseñador de aplicación lógica, en el desencadenador, elija **nuevo paso**.

   ![Elija "Nuevo paso"](./media/logic-apps-using-sap-connector/add-action.png)

1. En el cuadro de búsqueda, escriba "sap" como filtro. En la lista de acciones, seleccione esta acción: **Send message to SAP** (Enviar mensaje a SAP)
  
   ![Selección de la acción de envío de SAP](media/logic-apps-using-sap-connector/select-sap-send-action.png)

   Como alternativa, en lugar de buscar, elija la pestaña **Enterprise** (Empresa) y seleccione la acción de SAP.

   ![Selección de la acción de envío de SAP desde la pestaña Enterprise](media/logic-apps-using-sap-connector/select-sap-send-action-ent-tab.png)

1. Si se le pide indicar los detalles de la conexión, cree ahora mismo su conexión de SAP. De lo contrario, si la conexión ya existe, continúe con el paso siguiente para poder configurar la acción de SAP.

   **Creación de la conexión de SAP local**

   1. Proporcione la información de conexión para el servidor SAP. Para la propiedad **Data Gateway** (Puerta de enlace de datos), seleccione la puerta de enlace de datos que creó en Azure Portal para la instalación de la puerta de enlace.

      Si la propiedad **Logon Type** (Tipo de inicio de sesión) está establecida en **Application Server** (Servidor de aplicaciones), estas propiedades, que suelen aparecer como opcionales, son obligatorias:

      ![Creación de conexiones del servidor de aplicaciones de SAP](media/logic-apps-using-sap-connector/create-SAP-application-server-connection.png)

      Si la propiedad **Logon Type** (Tipo de inicio de sesión) está establecida en **Group** (Grupo), estas propiedades, que suelen aparecer como opcionales, son obligatorias:

      ![Creación de conexiones del servidor de mensajes de SAP](media/logic-apps-using-sap-connector/create-SAP-message-server-connection.png)

   1. Cuando termine, seleccione **Crear**.

      Logic Apps configura y comprueba la conexión y se asegura de que esta funciona correctamente.

1. Ahora, busque y seleccione una acción en el servidor SAP.

   1. En el **SAP acción** , seleccione el icono de carpeta. Desde la lista de archivos, busque y seleccione el mensaje de SAP que quiere utilizar. Para navegar por la lista, utilice las flechas.

      Este ejemplo selecciona un IDoc con **pedidos** tipo.

      ![Búsqueda y selección de la acción IDoc](./media/logic-apps-using-sap-connector/SAP-app-server-find-action.png)

      Si no encuentra la acción que quiere, puede escribir manualmente una ruta de acceso, por ejemplo:

      ![Proporcionar manualmente la ruta de acceso a la acción IDoc](./media/logic-apps-using-sap-connector/SAP-app-server-manually-enter-action.png)

      > [!TIP]
      > Proporcionar el valor de acción de SAP mediante el editor de expresiones. De este modo, puede usar la misma acción para diferentes tipos de mensaje.

      Para obtener más información sobre las operaciones de IDoc, consulte [Esquemas de mensaje para operaciones de IDOC](https://docs.microsoft.com/biztalk/adapters-and-accelerators/adapter-sap/message-schemas-for-idoc-operations).

   1. Haga clic en el cuadro **Mensaje de entrada** para que aparezca la lista de contenido dinámico. En dicha lista, en **When a HTTP request is received** (Cuando se recibe una solicitud HTTP), seleccione el campo **Cuerpo**.

      En este paso, se incluye el contenido del cuerpo del desencadenador de la solicitud HTTP y se envía la salida al servidor SAP.

      ![Selección del campo "Cuerpo"](./media/logic-apps-using-sap-connector/SAP-app-server-action-select-body.png)

      Una vez que haya terminado, la acción SAP debe ser parecida a la de este ejemplo:

      ![Completar una acción SAP](./media/logic-apps-using-sap-connector/SAP-app-server-complete-action.png)

1. Guarde la aplicación lógica. En la barra de herramientas del diseñador, haga clic en **Guardar**.

<a name="add-response"></a>

### <a name="add-http-response-action"></a>Adición de una acción de respuesta HTTP

Ahora, agregue una acción de respuesta al flujo de trabajo de la aplicación lógica e incluya el resultado de la acción SAP. De este modo, la aplicación lógica devuelve los resultados del servidor SAP al solicitante original.

1. En el Diseñador de aplicación lógica, en la acción de SAP, elija **nuevo paso**.

1. En el cuadro de búsqueda, escriba "respuesta" para el filtro. En la lista de acciones, seleccione esta acción: **Respuesta**

1. Haga clic en el cuadro **Cuerpo** para que aparezca la lista de contenido dinámico. En esa lista en **enviar mensaje a SAP**, seleccione el **cuerpo** campo.

   ![Completar una acción SAP](./media/logic-apps-using-sap-connector/select-sap-body-for-response-action.png)

1. Guarde la aplicación lógica.

### <a name="test-your-logic-app"></a>Comprobación de la aplicación lógica

1. Si la aplicación lógica aún no está habilitada, en el menú de aplicación lógica, elija **Introducción**. En la barra de herramientas, elija **Habilitar**.

1. En la barra de herramientas del Diseñador de aplicación lógica, elija **Ejecutar**. Con este paso, se inicia manualmente la aplicación lógica.

1. Desencadene la aplicación lógica mediante el envío de una solicitud HTTP POST a la dirección URL del desencadenador de la solicitud HTTP e incluya el contenido del mensaje con la solicitud. Para enviar la solicitud, puede usar una herramienta como [Postman](https://www.getpostman.com/apps).

   En el caso de este artículo, la solicitud envía un archivo de IDoc, que debe estar en formato XML e incluir el espacio de nombres de la acción de SAP que está utilizando, por ejemplo:

   ``` xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <Send xmlns="http://Microsoft.LobServices.Sap/2007/03/Idoc/2/ORDERS05//720/Send">
      <idocData>
         <...>
      </idocData>
   </Send>
   ```

1. Después de enviar la solicitud HTTP, espere la respuesta de la aplicación lógica.

   > [!NOTE]
   > La aplicación lógica puede agotar el tiempo de espera si todos los pasos necesarios para la respuesta no finalizan dentro del [límite de tiempo de espera de la solicitud](./logic-apps-limits-and-config.md). Si se produce esta situación, es posible que se bloqueen las solicitudes. A fin de poder diagnosticar problemas, obtenga información sobre cómo puede [comprobar y supervisar las aplicaciones lógicas](../logic-apps/logic-apps-monitor-your-logic-apps.md).

Enhorabuena. Ha creado una aplicación lógica que puede comunicarse con el servidor SAP. Ahora que ha configurado una conexión de SAP para la aplicación lógica, puede explorar otras acciones SAP disponibles, como RFC y BAPI.

## <a name="receive-from-sap"></a>Recibir de SAP

En este ejemplo se utiliza una aplicación lógica que se desencadena cuando se recibe un mensaje de un sistema SAP.

### <a name="add-sap-trigger"></a>Agregar un desencadenador de SAP

1. En Azure Portal, cree una aplicación lógica en blanco que abra el Diseñador de aplicación lógica.

1. En el cuadro de búsqueda, escriba "sap" como filtro. En la lista de desencadenadores, seleccione este desencadenador: **When a message is received from SAP** (Cuando se recibe un mensaje de SAP)

   ![Agregar un desencadenador de SAP](./media/logic-apps-using-sap-connector/add-sap-trigger.png)

   Como alternativa, puede ir a la **Enterprise** pestaña y seleccione el desencadenador:

   ![Agregar desencadenador SAP desde la pestaña de la empresa](./media/logic-apps-using-sap-connector/add-sap-trigger-ent-tab.png)

1. Si se le pide indicar los detalles de la conexión, cree ahora mismo su conexión de SAP. De lo contrario, si la conexión ya existe, continúe con el paso siguiente para poder configurar la acción de SAP.

   **Creación de la conexión de SAP local**

   1. Proporcione la información de conexión para el servidor SAP. Para la propiedad **Data Gateway** (Puerta de enlace de datos), seleccione la puerta de enlace de datos que creó en Azure Portal para la instalación de la puerta de enlace.

      Si la propiedad **Logon Type** (Tipo de inicio de sesión) está establecida en **Application Server** (Servidor de aplicaciones), estas propiedades, que suelen aparecer como opcionales, son obligatorias:

      ![Creación de conexiones del servidor de aplicaciones de SAP](media/logic-apps-using-sap-connector/create-SAP-application-server-connection.png)

      Si la propiedad **Logon Type** (Tipo de inicio de sesión) está establecida en **Group** (Grupo), estas propiedades, que suelen aparecer como opcionales, son obligatorias:

      ![Creación de conexiones del servidor de mensajes de SAP](media/logic-apps-using-sap-connector/create-SAP-message-server-connection.png)  

1. Proporcione los parámetros necesarios en función de la configuración del sistema SAP.

   También puede proporcionar una o varias acciones de SAP. Esta lista de acciones especifica los mensajes que recibe el desencadenador del servidor SAP a través de la puerta de enlace de datos. Una lista vacía especifica que el desencadenador recibe todos los mensajes. Si la lista tiene más de un mensaje, el desencadenador recibe solo los mensajes especificados en la lista. La puerta de enlace rechaza todos los demás mensajes enviados desde el servidor SAP.

   Puede seleccionar una acción de SAP desde el selector de archivos:

   ![Selección de una acción de SAP](media/logic-apps-using-sap-connector/select-SAP-action-trigger.png)  

   También puede especificar una acción manualmente:

   ![Introducción manual de una acción de SAP](media/logic-apps-using-sap-connector/manual-enter-SAP-action-trigger.png) 

   Este es un ejemplo que muestra la apariencia de la acción cuando se configura el desencadenador para recibir varios mensajes.

   ![Ejemplo de desencadenador](media/logic-apps-using-sap-connector/example-trigger.png)  

   Para obtener más información sobre las operaciones de SAP, consulte [Esquemas de mensaje para las operaciones de IDOC](https://docs.microsoft.com/biztalk/adapters-and-accelerators/adapter-sap/message-schemas-for-idoc-operations)

1. Ahora guarde la aplicación lógica para poder empezar a recibir mensajes del sistema SAP.
En la barra de herramientas del diseñador, haga clic en **Guardar**.

La aplicación lógica está preparada para recibir mensajes del sistema SAP.

> [!NOTE]
> El desencadenador de SAP no es un desencadenador de sondeo, sino un desencadenador basad en webhook. Solo se llama al desencadenador desde la puerta de enlace cuando existe un mensaje, por lo que no es necesario ningún sondeo.

### <a name="test-your-logic-app"></a>Comprobación de la aplicación lógica

1. Para desencadenar la aplicación lógica, envíe un mensaje desde el sistema SAP.

1. En el menú de la aplicación lógica, elija **Información general** y revise **Historial de ejecuciones** para conocer las nuevas ejecuciones de su aplicación lógica.

1. Abra la ejecución más reciente, que muestra el mensaje enviado desde su sistema SAP en la sección de salidas del desencadenador.

## <a name="generate-schemas-for-artifacts-in-sap"></a>Generar esquemas para artefactos en SAP

En este ejemplo, se utiliza una aplicación lógica que se puede desencadenar con una solicitud HTTP. La acción de SAP envía una solicitud a un sistema SAP para generar los esquemas de documento intermedio (IDoc) y BAPI especificados. Los esquemas que se devuelven en la respuesta se cargan en una cuenta de integración mediante el conector de Azure Resource Manager.

### <a name="add-http-request-trigger"></a>Adición de un desencadenador de solicitud HTTP

1. En Azure Portal, cree una aplicación lógica en blanco que abra el Diseñador de aplicación lógica.

1. En el cuadro de búsqueda, escriba "solicitud http" para el filtro. En la lista de desencadenadores, seleccione este desencadenador: **Cuando se recibe una solicitud HTTP**

   ![Adición de un desencadenador de solicitud HTTP](./media/logic-apps-using-sap-connector/add-trigger.png)

1. Ahora puede guardar la aplicación lógica para poder generar una dirección URL del punto de conexión para la aplicación lógica.
En la barra de herramientas del diseñador, haga clic en **Guardar**.

   La dirección URL del punto de conexión aparece ahora en el desencadenador, por ejemplo:

   ![Generación de una dirección URL para el punto de conexión](./media/logic-apps-using-sap-connector/generate-http-endpoint-url.png)

### <a name="add-sap-action-to-generate-schemas"></a>Agregar una acción de SAP para generar esquemas

1. En el Diseñador de aplicación lógica, en el desencadenador, elija **nuevo paso**.

   ![Elija "Nuevo paso"](./media/logic-apps-using-sap-connector/add-action.png)

1. En el cuadro de búsqueda, escriba "sap" como filtro. En la lista de acciones, seleccione esta acción: **Generate schemas** (Generar esquemas)
  
   ![Selección de la acción de envío de SAP](media/logic-apps-using-sap-connector/select-sap-schema-generator-action.png)

   Como alternativa, elija la pestaña **Enterprise** (Empresa) y seleccione la acción de SAP.

   ![Selección de la acción de envío de SAP desde la pestaña Enterprise](media/logic-apps-using-sap-connector/select-sap-schema-generator-ent-tab.png)

1. Si se le pide indicar los detalles de la conexión, cree ahora mismo su conexión de SAP. De lo contrario, si la conexión ya existe, continúe con el paso siguiente para poder configurar la acción de SAP.

   **Creación de la conexión de SAP local**

   1. Proporcione la información de conexión para el servidor SAP. Para la propiedad **Data Gateway** (Puerta de enlace de datos), seleccione la puerta de enlace de datos que creó en Azure Portal para la instalación de la puerta de enlace.

      Si la propiedad **Logon Type** (Tipo de inicio de sesión) está establecida en **Application Server** (Servidor de aplicaciones), estas propiedades, que suelen aparecer como opcionales, son obligatorias:

      ![Creación de conexiones del servidor de aplicaciones de SAP](media/logic-apps-using-sap-connector/create-SAP-application-server-connection.png)

      Si la propiedad **Logon Type** (Tipo de inicio de sesión) está establecida en **Group** (Grupo), estas propiedades, que suelen aparecer como opcionales, son obligatorias:

      ![Creación de conexiones del servidor de mensajes de SAP](media/logic-apps-using-sap-connector/create-SAP-message-server-connection.png)

   1. Cuando termine, seleccione **Crear**. Logic Apps configura y comprueba la conexión y se asegura de que esta funciona correctamente.

1. Proporcione la ruta de acceso del artefacto para el que quiere generar el esquema.

   Puede seleccionar la acción de SAP desde el selector de archivos:

   ![Selección de una acción de SAP](media/logic-apps-using-sap-connector/select-SAP-action-schema-generator.png)  

   O bien, puede escribir manualmente la acción:

   ![Introducción manual de una acción de SAP](media/logic-apps-using-sap-connector/manual-enter-SAP-action-schema-generator.png)

   Para generar esquemas de más de un artefacto, proporcione los detalles de la acción de SAP de cada artefacto, por ejemplo:

   ![Selección de la opción para agregar un elemento](media/logic-apps-using-sap-connector/schema-generator-array-pick.png)

   ![Visualización de dos elementos](media/logic-apps-using-sap-connector/schema-generator-example.png)

   Para obtener más información sobre la acción de SAP, consulte [Esquemas de mensaje para las operaciones de IDOC](https://docs.microsoft.com/biztalk/adapters-and-accelerators/adapter-sap/message-schemas-for-idoc-operations).

1. Guarde la aplicación lógica. En la barra de herramientas del diseñador, haga clic en **Guardar**.

### <a name="test-your-logic-app"></a>Comprobación de la aplicación lógica

1. En la barra de herramientas del diseñador, elija **Ejecutar** para desencadenar una ejecución de la aplicación lógica.

1. Abra la ejecución y compruebe los resultados para el **generar esquemas** acción.

   Las salidas muestran los esquemas generados para la lista de mensajes especificada.

### <a name="upload-schemas-to-integration-account"></a>Cargar esquemas en la cuenta de integración

Si lo desea, puede descargar o almacenar los esquemas generados en repositorios, como un blob, el almacenamiento o la cuenta de integración. Las cuentas de integración proporcionan una excelente experiencia con otras acciones de XML, por lo que en este ejemplo se muestra cómo cargar los esquemas en una cuenta de integración de la misma aplicación lógica mediante el conector de Azure Resource Manager.

1. En el desencadenador del Diseñador de aplicación lógica, seleccione **Nuevo paso**.

1. En el cuadro de búsqueda, escriba "Administrador de recursos" como filtro. Seleccione esta acción: **Crear o actualizar un recurso**

   ![Selección de la acción de Azure Resource Manager](media/logic-apps-using-sap-connector/select-azure-resource-manager-action.png)

1. Escriba los detalles de la acción, incluida su suscripción de Azure, el grupo de recursos de Azure y la cuenta de integración. Para agregar los tokens SAP en los campos, haga clic dentro de las casillas correspondientes a esos campos y seleccione en la lista de contenido dinámico que aparece.

   1. Abra el **Agregar nuevo parámetro** lista y seleccione el **ubicación** y **propiedades** campos.

   1. Proporcione detalles de estos campos como se muestra en este ejemplo.

      ![Introducción de detalles de la acción de Azure Resource Manager](media/logic-apps-using-sap-connector/azure-resource-manager-action.png)

   La acción **Generate schemas** (Generar esquemas) de SAP genera esquemas como una colección, de modo que el diseñador agrega automáticamente un bucle **For each** a la acción. El siguiente es un ejemplo que muestra cómo aparece esta acción:

   ![Acción de Azure Resource Manager con el bucle "for each"](media/logic-apps-using-sap-connector/azure-resource-manager-action-foreach.png)  

   > [!NOTE]
   > Los esquemas utilizan el formato codificado en Base64. Para cargar los esquemas en una cuenta de integración, deben descodificarse mediante la función `base64ToString()`. El siguiente es un ejemplo en el que se muestra el código del elemento `"properties"`:
   >
   > ```json
   > "properties": {
   >    "Content": "@base64ToString(items('For_each')?['Content'])",
   >    "ContentType": "application/xml",
   >    "SchemaType": "Xml"
   > }
   > ```

1. Guarde la aplicación lógica. En la barra de herramientas del diseñador, haga clic en **Guardar**.

### <a name="test-your-logic-app"></a>Comprobación de la aplicación lógica

1. En la barra de herramientas del diseñador, elija **Ejecutar** para desencadenar manualmente la aplicación lógica.

1. Después de una correcta ejecución, vaya a la cuenta de integración y comprobar que se encuentran los esquemas generados.

## <a name="enable-secure-network-communications-snc"></a>Habilitar las comunicaciones de red segura (SNC)

Antes de empezar, asegúrese de que se han cumplido enumeradas anteriormente [requisitos previos](#pre-reqs):

* La puerta de enlace de datos local se instala en un equipo que se encuentra en la misma red que su sistema SAP.

* Para Single Sign-On, la puerta de enlace se ejecuta como un usuario que está asignado al usuario SAP.

* Biblioteca SNC que proporciona las funciones de seguridad adicional se instaló en el mismo equipo como puerta de enlace de datos. Algunos de los ejemplos de estas incluyen [sapseculib](https://help.sap.com/saphelp_nw74/helpdata/en/7a/0755dc6ef84f76890a77ad6eb13b13/frameset.htm), Kerberos, NTLM, y así sucesivamente.

Para habilitar SNC para las solicitudes a o desde el sistema SAP, seleccione el **utilice SNC** casilla de verificación en la conexión de SAP y proporcionar estas propiedades:

   ![Configurar SAP SNC en conexión](media/logic-apps-using-sap-connector/configure-sapsnc.png)

   | Propiedad | DESCRIPCIÓN |
   |----------| ------------|
   | **Biblioteca SNC** | El nombre de la biblioteca SNC o ruta de acceso relativa a la ubicación de instalación NCo o ruta de acceso absoluta. Por ejemplo, `sapsnc.dll` o `.\security\sapsnc.dll` o `c:\security\sapsnc.dll` |
   | **INICIO DE SESIÓN ÚNICO DE SNC** | Al conectarse a través de SNC, la identidad SNC normalmente se usa para autenticar al llamador. Otra opción consiste en reemplazar para que se puede usar la información de usuario y contraseña para autenticar al llamador, pero todavía se cifra la línea. |
   | **Mi nombre SNC** | En la mayoría de los casos, se puede omitir esta propiedad. La solución instalada de SNC normalmente conoce su propio nombre SNC. Solo para las soluciones que admiten "identidades múltiples", es posible que necesite especificar la identidad que se utilizará para este destino concreto o un servidor. |
   | **Nombre del asociado de SNC** | El nombre para el back-end SNC |
   | **Calidad SNC de protección** | Calidad de servicio que se usará para la comunicación de SNC este particular o del servidor de destino. Valor predeterminado definido por el sistema back-end. El valor máximo es definido por el producto de seguridad utilizado para SNC. |
   |||

   > [!NOTE]
   > Variables de entorno SNC_LIB y SNC_LIB_64 no deben establecerse en el equipo donde haya puerta de enlace de datos y biblioteca SNC. Si se establece, tendría prioridad sobre el valor de la biblioteca de SNC pasado a través del conector.

## <a name="known-issues-and-limitations"></a>Problemas conocidos y limitaciones

Estos son los problemas y limitaciones actualmente conocidos para el conector SAP:

* Solo un envío único a una llamada o un mensaje de SAP funciona con tRFC. El patrón de confirmación de Business Application Programming Interface (BAPI), como realizar varias llamadas de tRFC en la misma sesión, no se admite.

* El desencadenador SAP no admite la recepción de IDOC por lotes de SAP. Esta acción puede producir un error de conexión de RFC entre el sistema SAP y la puerta de enlace de datos.

* El desencadenador SAP no admite clústeres de puerta de enlace de datos. En algunos casos de conmutación por error, el nodo de puerta de enlace de datos que se comunica con el sistema SAP puede diferir del nodo activo, lo que produce un comportamiento inesperado. Para los escenarios de envío, se admiten clústeres de puerta de enlace de datos.

* El conector SAP no admite actualmente las cadenas de enrutador SAP. La puerta de enlace de datos local debe existir en la misma LAN que el sistema SAP al que quiere conectarse.

## <a name="connector-reference"></a>Referencia de conectores

Para obtener información técnica acerca de los desencadenadores, acciones y los límites, que se describen mediante OpenAPI del conector (anteriormente Swagger) descripción, revise el [página de referencia del conector](/connectors/sap/).

## <a name="next-steps"></a>Pasos siguientes

* [Conexión a sistemas locales](../logic-apps/logic-apps-gateway-connection.md) desde aplicaciones lógicas
* Obtenga información acerca de cómo validar, transformar y realizar otras operaciones de mensaje con [Enterprise Integration Pack](../logic-apps/logic-apps-enterprise-integration-overview.md).
* Obtenga más información sobre otros [conectores de Logic Apps](../connectors/apis-list.md)
