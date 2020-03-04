---
title: Conexión a sistemas SAP
description: Cómo acceder a recursos de SAP y administrarlos mediante la automatización de flujos de trabajo con Azure Logic Apps
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: estfan, logicappspm
ms.topic: article
ms.date: 08/30/2019
tags: connectors
ms.openlocfilehash: 3990a9d53a8e53ddd29683440917031bc1bb6448
ms.sourcegitcommit: 67e9f4cc16f2cc6d8de99239b56cb87f3e9bff41
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/31/2020
ms.locfileid: "76904655"
---
# <a name="connect-to-sap-systems-from-azure-logic-apps"></a>Conexión a sistemas SAP desde Azure Logic Apps

> [!IMPORTANT]
> Los conectores anteriores de SAP Application Server y SAP Message Server están programados para dejar de usarse el 29 de febrero de 2020. El conector de SAP actual consolida estos conectores de SAP anteriores para que no tenga que cambiar el tipo de conexión, es totalmente compatible con los conectores anteriores, proporciona muchas capacidades adicionales y sigue usando la biblioteca del conector SAP .NET ( SAP NCo).
>
> En el caso de las aplicaciones lógicas que usan los conectores más antiguos, [migre al conector más reciente](#migrate) antes de la fecha de desuso. De lo contrario, estas aplicaciones lógicas experimentarán errores de ejecución y no podrán enviar mensajes al sistema SAP.

En este artículo, se muestra cómo acceder a los recursos de SAP locales desde una aplicación lógica mediante el conector SAP. El conector funciona con las versiones clásicas de SAP, como los sistemas R/3 y ECC del entorno local. También permite la integración con los sistemas SAP más nuevos basados en HANA, como S/4 HANA, dondequiera que estén alojados, en el entorno local o en la nube. El conector SAP admite la integración de mensajes o datos hacia y desde sistemas basados en SAP NetWeaver a través de un documento intermedio (IDOC), de Business Application Programming Interface (BAPI) o de Remote Function Call (RFC).

El conector SAP emplea la [biblioteca de SAP .NET Connector (NCo)](https://support.sap.com/en/product/connectors/msnet.html) y proporciona estas acciones:

* **Enviar mensaje a SAP**: envío de IDOC a través de tRFC, llamada a funciones BAPI a través de RFC o llamada a sistemas SAP RFC/tRFC.
* **Cuando se recibe un mensaje de SAP**: recepción de IDOC a través de tRFC, llamada a funciones BAPI a través de tRFC o llamada a RFC/tRFC en sistemas SAP.
* **Generate schemas** (Generar esquemas): generación de esquemas de artefactos SAP para IDOC, BAPI o RFC.

En todas las operaciones anteriores, el conector SAP admite autenticación básica con nombre de usuario y contraseña. El conector también admite [comunicaciones de red seguras (SNC)](https://help.sap.com/doc/saphelp_nw70/7.0.31/e6/56f466e99a11d1a5b00000e835363f/content.htm?no_cache=true). SNC puede usarse con el inicio de sesión único (SSO) de SAP NetWeaver-o con funcionalidades de seguridad adicionales proporcionadas por un producto de seguridad externo.

El conector SAP se integra con los sistemas SAP locales a través de la [puerta de enlace de datos local](../logic-apps/logic-apps-gateway-connection.md). En los escenarios de envío, por ejemplo, al enviar un mensaje desde una aplicación lógica hasta un sistema SAP, la puerta de enlace de datos actúa como cliente RFC y reenvía las solicitudes recibidas de la aplicación lógica a SAP. Del mismo modo, en los escenarios de recepción, la puerta de enlace de datos actúa como servidor RFC que recibe solicitudes de SAP y las reenvía a la aplicación lógica.

En este artículo se muestra cómo crear aplicaciones lógicas que se integren con SAP y se tratan los escenarios de integración descritos anteriormente. En el caso de las aplicaciones lógicas que usan los conectores de SAP más antiguos, en este artículo se muestra cómo migrar las aplicaciones lógicas a la versión más reciente del conector de SAP.

<a name="pre-reqs"></a>

## <a name="prerequisites"></a>Prerequisites

Para seguir con este artículo, necesita los siguientes elementos:

* Suscripción a Azure. Si aún no tiene ninguna suscripción de Azure, [regístrese para obtener una cuenta gratuita de Azure](https://azure.microsoft.com/free/).

* La aplicación lógica desde donde quiere obtener acceso al sistema SAP y un desencadenador que inicie el flujo de trabajo de la aplicación lógica. Si no está familiarizado con las aplicaciones lógicas, consulte [¿Qué es Azure Logic Apps?](../logic-apps/logic-apps-overview.md) y el [Inicio rápido: Creación de la primera aplicación lógica](../logic-apps/quickstart-create-first-logic-app-workflow.md).

* El [servidor de aplicaciones de SAP](https://wiki.scn.sap.com/wiki/display/ABAP/ABAP+Application+Server) o el [servidor de mensajes de SAP](https://help.sap.com/saphelp_nw70/helpdata/en/40/c235c15ab7468bb31599cc759179ef/frameset.htm).

* Descargue e instale la versión más reciente de la [puerta de enlace de datos local](https://www.microsoft.com/download/details.aspx?id=53127) en cualquier equipo local. Asegúrese de configurar la puerta de enlace en Azure Portal antes de continuar. La puerta de enlace le ayuda a acceder de forma segura a los datos y recursos que se encuentran en el entorno local. Para más información, consulte [Instalación de una puerta de enlace de datos local para Azure Logic Apps](../logic-apps/logic-apps-gateway-install.md).

* Si usa SNC con SSO, asegúrese de que la puerta de enlace se ejecuta con un usuario asignado con el usuario de SAP. Para cambiar la cuenta predeterminada, seleccione **Cambiar cuenta** y escriba las credenciales de usuario.

  ![Cambio de la cuenta de puerta de enlace](./media/logic-apps-using-sap-connector/gateway-account.png)

* Si habilita SNC con un producto de seguridad externo, copie la biblioteca o los archivos de SNC en la misma máquina donde está instalada la puerta de enlace. Algunos ejemplos de productos SNC incluyen [sapseculib](https://help.sap.com/saphelp_nw74/helpdata/en/7a/0755dc6ef84f76890a77ad6eb13b13/frameset.htm), Kerberos y NTLM.

> [!IMPORTANT]
> Una vez instalada la biblioteca del conector SAP .NET ( SAP NCo) hay que copiar las .dll de la carpeta de instalación a la carpeta de instalación de nuestra puerta de enlace. Si dichas .dll no son copiadas a la carpeta de instalación del Data Gateway obtendremos el siguiente error **Could not load file or assembly 'sapnco'**

* Descargue e instale la biblioteca cliente de SAP más reciente, que actualmente es [SAP Connector (NCo 3.0) para Microsoft .NET 3.0.22.0 compilado con .NET Framework 4.0 y Windows de 64 bits (x64)](https://softwaredownloads.sap.com/file/0020000001000932019), en el mismo equipo que la puerta de enlace de datos local. Debe instalar esta versión o una versión posterior por estos motivos:

  * Las versiones anteriores de SAP NCo pueden experimentar un interbloqueo cuando se envía más de un mensaje de IDOC al mismo tiempo. Esta condición bloquea todos los mensajes posteriores que se envían al destino de SAP, lo que hace que se agote el tiempo de espera de los mensajes.
  
  * La puerta de enlace de datos local solo se ejecuta en sistemas de 64 bits. De lo contrario, obtendrá un error de "imagen incorrecta" porque el servicio de host de la puerta de enlace de datos no es compatible con los ensamblados de 32 bits.
  
  * El servicio de host de la puerta de enlace de datos y el adaptador de SAP de Microsoft usan .NET Framework 4.5. NCo de SAP para .NET Framework 4.0 funciona con procesos que usan las versiones 4.0 a 4.7.1 del runtime de .NET. SAP NCo para .NET Framework 2.0 funciona con procesos que usan las versiones 2.0 a 3.5 del entorno de ejecución de .NET y ya no funciona con la puerta de enlace de datos local más reciente.

* El contenido del mensaje que puede enviar a su servidor SAP, como un archivo IDOC de ejemplo, debe estar en formato XML e incluir el espacio de nombres de la acción de SAP que quiera usar.

<a name="migrate"></a>

## <a name="migrate-to-current-connector"></a>Migración al conector actual

1. Si todavía no lo ha hecho, actualice la [puerta de enlace de datos local](https://www.microsoft.com/download/details.aspx?id=53127) para que tenga la versión más reciente. Para más información, consulte [Instalación de una puerta de enlace de datos local para Azure Logic Apps](../logic-apps/logic-apps-gateway-install.md).

1. En la aplicación lógica que usa el conector de SAP anterior, elimine la acción **Enviar a SAP**.

1. En el conector de SAP más reciente, agregue la acción **Enviar mensaje a SAP**. Para poder usar esta acción, vuelva a crear la conexión al sistema SAP.

1. Cuando haya terminado, guarde la aplicación lógica.

<a name="add-trigger"></a>

## <a name="send-message-to-sap"></a>Enviar mensaje a SAP

En este ejemplo, se utiliza una aplicación lógica que se puede desencadenar con una solicitud HTTP. La aplicación lógica envía un IDOC a un servidor SAP y devuelve una respuesta al solicitante que llamó a la aplicación lógica.

### <a name="add-an-http-request-trigger"></a>Adición de un desencadenador de solicitud HTTP

En Azure Logic Apps, cada aplicación lógica debe comenzar con un [desencadenador](../logic-apps/logic-apps-overview.md#logic-app-concepts), que se activa cuando sucede un evento específico o cuando se cumple una condición determinada. Cada vez que el desencadenador se activa, el motor de Logic Apps crea una instancia de aplicación lógica y empieza a ejecutar el flujo de trabajo de la aplicación.

En este ejemplo, cree una aplicación lógica con un punto de conexión en Azure para poder enviar *solicitudes HTTP POST* a la aplicación lógica. Cuando la aplicación lógica reciba estas solicitudes HTTP, el desencadenador se activará y ejecutará el paso siguiente en el flujo de trabajo.

1. En [Azure Portal](https://portal.azure.com), cree una aplicación lógica en blanco que abra el Diseñador de aplicación lógica.

1. En el cuadro de búsqueda, escriba `http request` como filtro. En la lista **Desencadenadores**, seleccione **Cuando se recibe una solicitud HTTP**.

   ![Adición de un desencadenador de solicitud HTTP](./media/logic-apps-using-sap-connector/add-http-trigger-logic-app.png)

1. Ahora, guarde la aplicación lógica para poder generar una dirección URL de punto de conexión para ella. En la barra de herramientas del diseñador, seleccione **Save** (Guardar).

   La dirección URL del punto de conexión aparece ahora en el desencadenador, por ejemplo:

   ![Generación de una dirección URL para el punto de conexión](./media/logic-apps-using-sap-connector/generate-http-endpoint-url.png)

<a name="add-action"></a>

### <a name="add-an-sap-action"></a>Adición de una acción de SAP

En Azure Logic Apps, una [acción](../logic-apps/logic-apps-overview.md#logic-app-concepts) es un paso del flujo de trabajo que sigue a un desencadenador u otra acción. Si aún no ha agregado un desencadenador a la aplicación lógica y quiere seguir este ejemplo, [agregue el desencadenador que se describe en esta sección](#add-trigger).

1. En el Diseñador de aplicación lógica, seleccione **New Step** (Nuevo paso).

   ![Agregar un nuevo paso a la aplicación lógica](./media/logic-apps-using-sap-connector/add-sap-action-logic-app.png)

1. En el cuadro de búsqueda, escriba `sap` como filtro. En la lista **Actions** (Acciones), seleccione **Send message to SAP** (Enviar mensajes a SAP).
  
   ![Seleccione la acción "Enviar mensaje a SAP"](media/logic-apps-using-sap-connector/select-sap-send-action.png)

   O bien, puede elegir la pestaña **Enterprise** y seleccionar la acción de SAP.

   ![Seleccione la acción "Enviar mensaje a SAP"en la pestaña Enterprise](media/logic-apps-using-sap-connector/select-sap-send-action-ent-tab.png)

1. Si la conexión ya existe, continúe con el paso siguiente para configurar la acción de SAP. Sin embargo, si se le solicitan detalles de conexión, proporcione la información para que pueda crear una conexión a su servidor SAP local ahora.

   1. Proporcione un nombre para la conexión.

   1. En la sección **Puerta de enlace de datos**, en **Suscripción**, primero seleccione la suscripción de Azure para el recurso de puerta de enlace que creó en el Azure Portal para la instalación de su puerta de enlace. 
   
   1. En **Puerta de enlace de conexión**, seleccione su recurso de puerta de enlace.

   1. Siga proporcionando información acerca de la conexión. Para la propiedad **Tipo de inicio de sesión**, siga el paso en función de si la propiedad está establecida en **Servidor de aplicaciones** o **Grupo**:
   
      * Para **Servidos de aplicaciones**, estas propiedades, que generalmente parecen opcionales, son necesarias:

        ![Creación de conexiones del servidor de aplicaciones de SAP](media/logic-apps-using-sap-connector/create-SAP-application-server-connection.png)

      * Para **Grupo**, estas propiedades, que generalmente parecen opcionales, son necesarias:

        ![Creación de conexiones del servidor de mensajes de SAP](media/logic-apps-using-sap-connector/create-SAP-message-server-connection.png)  

      De forma predeterminada, se usa el establecimiento inflexible de tipos para comprobar si hay valores no válidos, para lo cual se realiza la validación de XML con respecto al esquema. Este comportamiento puede ayudarlo a detectar problemas con antelación. La opción **Escritura segura** está disponible para la compatibilidad con versiones anteriores y solo comprueba la longitud de cadena. Más información sobre la opción [Escritura segura](#safe-typing).

   1. Cuando haya finalizado, seleccione **Crear**.

      Logic Apps configura y comprueba la conexión para asegurarse de que funciona correctamente.

1. Ahora, busque y seleccione una acción en el servidor SAP.

   1. En el cuadro **SAP Action** (Acción de SAP), elija el icono de carpeta. Desde la lista de archivos, busque y seleccione el mensaje de SAP que quiere utilizar. Para navegar por la lista, utilice las flechas.

      En este ejemplo se selecciona un IDOC con el tipo **Orders** (Pedidos).

      ![Búsqueda y selección de la acción IDoc](./media/logic-apps-using-sap-connector/SAP-app-server-find-action.png)

      Si no encuentra la acción que quiere, puede escribir manualmente una ruta de acceso, por ejemplo:

      ![Proporcionar manualmente la ruta de acceso a la acción IDoc](./media/logic-apps-using-sap-connector/SAP-app-server-manually-enter-action.png)

      > [!TIP]
      > Proporcione el valor de la **acción de SAP** mediante el editor de expresiones. De este modo, puede usar la misma acción para diferentes tipos de mensajes.

      Para obtener más información sobre las operaciones de IDoc, consulte [Esquemas de mensaje para operaciones de IDOC](https://docs.microsoft.com/biztalk/adapters-and-accelerators/adapter-sap/message-schemas-for-idoc-operations).

   1. Haga clic en el cuadro **Mensaje de entrada** para que aparezca la lista de contenido dinámico. En dicha lista, en **When a HTTP request is received** (Cuando se recibe una solicitud HTTP), seleccione el campo **Cuerpo**.

      En este paso, se incluye el contenido del cuerpo del desencadenador de la solicitud HTTP y se envía la salida al servidor SAP.

      ![Seleccione la propiedad "Cuerpo" en el desencadenador](./media/logic-apps-using-sap-connector/SAP-app-server-action-select-body.png)

      Cuando haya terminado, la acción SAP debe parecerse a este ejemplo:

      ![Completar la acción SAP](./media/logic-apps-using-sap-connector/SAP-app-server-complete-action.png)

1. Guarde la aplicación lógica. En la barra de herramientas del diseñador, seleccione **Save** (Guardar).

<a name="add-response"></a>

### <a name="add-an-http-response-action"></a>Adición de una acción de respuesta HTTP

Ahora, agregue una acción de respuesta al flujo de trabajo de la aplicación lógica e incluya el resultado de la acción SAP. De este modo, la aplicación lógica devuelve los resultados del servidor SAP al solicitante original.

1. En el Diseñador de aplicación lógica, en la acción de SAP, seleccione **New step**(Nuevo paso).

1. En el cuadro de búsqueda, escriba `response` como filtro. En la lista **Actions** (Acciones), seleccione **Response** (Respuesta).

1. Haga clic en el cuadro **Cuerpo** para que aparezca la lista de contenido dinámico. En dicha lista, en **Send message to SAP** (Enviar mensaje a SAP), seleccione el campo **Body** (Cuerpo).

   ![Completar una acción SAP](./media/logic-apps-using-sap-connector/select-sap-body-for-response-action.png)

1. Guarde la aplicación lógica.

### <a name="test-your-logic-app"></a>Comprobación de la aplicación lógica

1. Si la aplicación lógica no está aún habilitada, en el menú de la aplicación lógica, elija **Información general**. En la barra de herramientas, seleccione **Enable** (Habilitar).

1. En la barra de herramientas del diseñador, seleccione **Run** (Ejecutar). Con este paso, se inicia manualmente la aplicación lógica.

1. Desencadene la aplicación lógica mediante el envío de una solicitud HTTP POST a la dirección URL del desencadenador de la solicitud HTTP.
Incluya el contenido del mensaje con la solicitud. Para enviar la solicitud, puede usar una herramienta como [Postman](https://www.getpostman.com/apps).

   En el caso de este artículo, la solicitud envía un archivo de IDoc, que debe estar en formato XML e incluir el espacio de nombres de la acción de SAP que está utilizando, por ejemplo:

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <Send xmlns="http://Microsoft.LobServices.Sap/2007/03/Idoc/2/ORDERS05//720/Send">
      <idocData>
         <...>
      </idocData>
   </Send>
   ```

1. Después de enviar la solicitud HTTP, espere la respuesta de la aplicación lógica.

   > [!NOTE]
   > La aplicación lógica puede agotar el tiempo de espera si todos los pasos necesarios para la respuesta no finalizan dentro del [límite de tiempo de espera de la solicitud](./logic-apps-limits-and-config.md). Si se produce esta situación, es posible que se bloqueen las solicitudes. A fin de poder diagnosticar problemas, obtenga información sobre cómo puede [comprobar y supervisar las aplicaciones lógicas](../logic-apps/monitor-logic-apps.md).

Ahora ha creado una aplicación lógica que puede comunicarse con el servidor SAP. Ahora que ha configurado una conexión de SAP para la aplicación lógica, puede explorar otras acciones SAP disponibles, como RFC y BAPI.

<a name="receive-from-sap"></a>

## <a name="receive-message-from-sap"></a>Recibir un mensaje de SAP

En este ejemplo se usa una aplicación lógica que se desencadena cuando la aplicación recibe un mensaje de un sistema SAP.

### <a name="add-an-sap-trigger"></a>Adición de un desencadenador de SAP

1. En Azure Portal, cree una aplicación lógica en blanco que abra el Diseñador de aplicación lógica.

1. En el cuadro de búsqueda, escriba `sap` como filtro. En la lista **Desencadenadores**, seleccione **When a message is received from SAP** (Cuando se recibe un mensaje de SAP).

   ![Agregar un desencadenador de SAP](./media/logic-apps-using-sap-connector/add-sap-trigger-logic-app.png)

   O bien, puede elegir la pestaña **Enterprise** y después seleccionar la acción de desencadenar:

   ![Adición de un desencadenador de SAP desde la pestaña Enterprise (Empresa)](./media/logic-apps-using-sap-connector/add-sap-trigger-ent-tab.png)

1. Si la conexión ya existe, continúe con el paso siguiente para configurar la acción de SAP. Sin embargo, si se le solicitan detalles de conexión, proporcione la información para que pueda crear una conexión a su servidor SAP local ahora.

   1. Proporcione un nombre para la conexión.

   1. En la sección **Puerta de enlace de datos**, en **Suscripción**, primero seleccione la suscripción de Azure para el recurso de puerta de enlace que creó en el Azure Portal para la instalación de su puerta de enlace. 

   1. En **Puerta de enlace de conexión**, seleccione su recurso de puerta de enlace.

   1. Siga proporcionando información acerca de la conexión. Para la propiedad **Tipo de inicio de sesión**, siga el paso en función de si la propiedad está establecida en **Servidor de aplicaciones** o **Grupo**:

      * Para **Servidos de aplicaciones**, estas propiedades, que generalmente parecen opcionales, son necesarias:

        ![Creación de conexiones del servidor de aplicaciones de SAP](media/logic-apps-using-sap-connector/create-SAP-application-server-connection.png)

      * Para **Grupo**, estas propiedades, que generalmente parecen opcionales, son necesarias:

        ![Creación de conexiones del servidor de mensajes de SAP](media/logic-apps-using-sap-connector/create-SAP-message-server-connection.png)

      De forma predeterminada, se usa el establecimiento inflexible de tipos para comprobar si hay valores no válidos, para lo cual se realiza la validación de XML con respecto al esquema. Este comportamiento puede ayudarlo a detectar problemas con antelación. La opción **Escritura segura** está disponible para la compatibilidad con versiones anteriores y solo comprueba la longitud de cadena. Más información sobre la opción [Escritura segura](#safe-typing).

   1. Cuando haya finalizado, seleccione **Crear**.

      Logic Apps configura y comprueba la conexión para asegurarse de que funciona correctamente.

1. Proporcione los parámetros necesarios en función de la configuración del sistema SAP.

   También puede proporcionar una o varias acciones de SAP. Esta lista de acciones especifica los mensajes que recibe el desencadenador del servidor SAP a través de la puerta de enlace de datos. Una lista vacía especifica que el desencadenador recibe todos los mensajes. Si la lista tiene más de un mensaje, el desencadenador recibe solo los mensajes especificados en la lista. La puerta de enlace rechaza todos los demás mensajes enviados desde el servidor SAP.

   Puede seleccionar una acción de SAP desde el selector de archivos:

   ![Agregar acción de SAP a la aplicación lógica](media/logic-apps-using-sap-connector/select-SAP-action-trigger.png)  

   También puede especificar una acción manualmente:

   ![Introducción manual de una acción de SAP](media/logic-apps-using-sap-connector/manual-enter-SAP-action-trigger.png)

   Este es un ejemplo que muestra la apariencia de la acción cuando se configura el desencadenador para recibir varios mensajes.

   ![Ejemplo de desencadenador que recibe múltiples mensajes](media/logic-apps-using-sap-connector/example-trigger.png)

   Para obtener más información sobre las operaciones de SAP, consulte [Esquemas de mensaje para las operaciones de IDOC](https://docs.microsoft.com/biztalk/adapters-and-accelerators/adapter-sap/message-schemas-for-idoc-operations)

1. Ahora guarde la aplicación lógica para poder empezar a recibir mensajes del sistema SAP. En la barra de herramientas del diseñador, seleccione **Save** (Guardar).

La aplicación lógica está preparada para recibir mensajes del sistema SAP.

> [!NOTE]
> El desencadenador de SAP no es un desencadenador de sondeo, sino un desencadenador basado en webhook. Solo se llama al desencadenador desde la puerta de enlace cuando existe un mensaje, por lo que no es necesario ningún sondeo.

### <a name="test-your-logic-app"></a>Comprobación de la aplicación lógica

1. Para desencadenar la aplicación lógica, envíe un mensaje desde el sistema SAP.

1. En el menú de la aplicación lógica, seleccione **Información general**. Revise el **historial de ejecuciones** para ver las nuevas ejecuciones de la aplicación lógica.

1. Abra la ejecución más reciente, que muestra el mensaje enviado desde su sistema SAP en la sección de salidas del desencadenador.

## <a name="receive-idoc-packets-from-sap"></a>Recepción de paquetes IDOC de SAP

Puede configurar SAP para [enviar IDOC en paquetes](https://help.sap.com/viewer/8f3819b0c24149b5959ab31070b64058/7.4.16/en-US/4ab38886549a6d8ce10000000a42189c.html), que son lotes o grupos de IDOC. Para recibir paquetes de IDOC, el conector de SAP y, en concreto, el desencadenador, no necesita configuración adicional. Sin embargo, para procesar cada elemento en un paquete IDOC después de que el desencadenador reciba el paquete, se necesitan algunos pasos adicionales para dividir el paquete en IDOC individuales.

Este es un ejemplo que muestra cómo extraer IDOC individuales de un paquete mediante la [función `xpath()`](./workflow-definition-language-functions-reference.md#xpath):

1. Antes de empezar, necesita una aplicación lógica con un desencadenador de SAP. Si aún no tiene esta aplicación lógica, siga los pasos anteriores de este tema para configurar una [aplicación lógica con un desencadenador de SAP](#receive-from-sap).

   Por ejemplo:

   ![Agregar disparador SAP a la aplicación lógica](./media/logic-apps-using-sap-connector/first-step-trigger.png)

1. Obtiene el espacio de nombres raíz de IDOC XML que la aplicación lógica recibe de SAP. Para extraer este espacio de nombres del documento XML, agregue un paso que cree una variable de cadena local y almacene dicho espacio de nombres mediante una expresión `xpath()`:

   `xpath(xml(triggerBody()?['Content']), 'namespace-uri(/*)')`

   ![Obtener el espacio de nombres raíz del IDOC](./media/logic-apps-using-sap-connector/get-namespace.png)

1. Para extraer un IDOC individual, agregue un paso que cree una variable de matriz y almacene la colección de IDOC mediante otra expresión `xpath()`:

   `xpath(xml(triggerBody()?['Content']), '/*[local-name()="Receive"]/*[local-name()="idocData"]')`

   ![Obtención de una matriz de elementos](./media/logic-apps-using-sap-connector/get-array.png)

   La variable de matriz hace que cada IDOC esté disponible para que la aplicación lógica lo procese individualmente mediante la enumeración de la colección. En este ejemplo, la aplicación lógica transfiere cada IDOC a un servidor SFTP mediante un bucle:

   ![Enviar IDOC al servidor SFTP](./media/logic-apps-using-sap-connector/loop-batch.png)

   Cada IDOC debe incluir el espacio de nombres raíz, que es el motivo por el que el contenido del archivo se ajusta dentro de un elemento `<Receive></Receive` junto con el espacio de nombres raíz antes de enviar el IDOC a la aplicación de nivel inferior o servidor SFTP en este caso.

Puede usar la plantilla de inicio rápido para este patrón si selecciona esta plantilla en el diseñador de aplicaciones lógicas al crear una nueva aplicación lógica.

![Seleccionar plantilla de aplicación de lógica de lote](./media/logic-apps-using-sap-connector/select-batch-logic-app-template.png)

## <a name="generate-schemas-for-artifacts-in-sap"></a>Generar esquemas para artefactos en SAP

En este ejemplo, se utiliza una aplicación lógica que se puede desencadenar con una solicitud HTTP. La acción de SAP envía una solicitud a un sistema SAP para generar los esquemas de IDOC y BAPI especificados. Los esquemas que se devuelven en la respuesta se cargan en una cuenta de integración mediante el conector de Azure Resource Manager.

### <a name="add-an-http-request-trigger"></a>Adición de un desencadenador de solicitud HTTP

1. En Azure Portal, cree una aplicación lógica en blanco que abra el Diseñador de aplicación lógica.

1. En el cuadro de búsqueda, escriba `http request` como filtro. En la lista **Desencadenadores**, seleccione **Cuando se recibe una solicitud HTTP**.

   ![Adición de un desencadenador de solicitud HTTP](./media/logic-apps-using-sap-connector/add-http-trigger-logic-app.png)

1. Ahora puede guardar la aplicación lógica para poder generar una dirección URL del punto de conexión para la aplicación lógica.
En la barra de herramientas del diseñador, seleccione **Save** (Guardar).

   La dirección URL del punto de conexión aparece ahora en el desencadenador, por ejemplo:

   ![Generación de una dirección URL para el punto de conexión](./media/logic-apps-using-sap-connector/generate-http-endpoint-url.png)

### <a name="add-an-sap-action-to-generate-schemas"></a>Adición de una acción de SAP para generar esquemas

1. En el Diseñador de aplicación lógica, seleccione **New Step** (Nuevo paso).

   ![Agregar un nuevo paso a la aplicación lógica](./media/logic-apps-using-sap-connector/add-sap-action-logic-app.png)

1. En el cuadro de búsqueda, escriba `sap` como filtro. En la lista **Actions** (Acciones), seleccione **Generate schemas** (Generar esquemas).
  
   ![Agregar la acción "generar esquemas" a la aplicación lógica](media/logic-apps-using-sap-connector/select-sap-schema-generator-action.png)

   O bien, puede elegir la pestaña **Enterprise** y seleccionar la acción de SAP.

   ![Selección de la acción de envío de SAP desde la pestaña Enterprise](media/logic-apps-using-sap-connector/select-sap-schema-generator-ent-tab.png)

1. Si la conexión ya existe, continúe con el paso siguiente para configurar la acción de SAP. Sin embargo, si se le solicitan detalles de conexión, proporcione la información para que pueda crear una conexión a su servidor SAP local ahora.

   1. Proporcione un nombre para la conexión.

   1. En la sección **Puerta de enlace de datos**, en **Suscripción**, primero seleccione la suscripción de Azure para el recurso de puerta de enlace que creó en el Azure Portal para la instalación de su puerta de enlace. 
   
   1. En **Puerta de enlace de conexión**, seleccione su recurso de puerta de enlace.

   1. Siga proporcionando información acerca de la conexión. Para la propiedad **Tipo de inicio de sesión**, siga el paso en función de si la propiedad está establecida en **Servidor de aplicaciones** o **Grupo**:
   
      * Para **Servidos de aplicaciones**, estas propiedades, que generalmente parecen opcionales, son necesarias:

        ![Creación de conexiones del servidor de aplicaciones de SAP](media/logic-apps-using-sap-connector/create-SAP-application-server-connection.png)

      * Para **Grupo**, estas propiedades, que generalmente parecen opcionales, son necesarias:

        ![Creación de conexiones del servidor de mensajes de SAP](media/logic-apps-using-sap-connector/create-SAP-message-server-connection.png)  

      De forma predeterminada, se usa el establecimiento inflexible de tipos para comprobar si hay valores no válidos, para lo cual se realiza la validación de XML con respecto al esquema. Este comportamiento puede ayudarlo a detectar problemas con antelación. La opción **Escritura segura** está disponible para la compatibilidad con versiones anteriores y solo comprueba la longitud de cadena. Más información sobre la opción [Escritura segura](#safe-typing).

   1. Cuando haya finalizado, seleccione **Crear**.

      Logic Apps configura y comprueba la conexión para asegurarse de que funciona correctamente.

1. Proporcione la ruta de acceso del artefacto para el que quiere generar el esquema.

   Puede seleccionar la acción de SAP desde el selector de archivos:

   ![Selección de una acción de SAP](media/logic-apps-using-sap-connector/select-SAP-action-schema-generator.png)  

   O bien, puede escribir manualmente la acción:

   ![Introducción manual de una acción de SAP](media/logic-apps-using-sap-connector/manual-enter-SAP-action-schema-generator.png)

   Para generar esquemas de más de un artefacto, proporcione los detalles de la acción de SAP de cada artefacto, por ejemplo:

   ![Selección de la opción para agregar un elemento](media/logic-apps-using-sap-connector/schema-generator-array-pick.png)

   ![Visualización de dos elementos](media/logic-apps-using-sap-connector/schema-generator-example.png)

   Para más información sobre la acción de SAP, consulte [Esquemas de mensaje para operaciones de IDOC](https://docs.microsoft.com/biztalk/adapters-and-accelerators/adapter-sap/message-schemas-for-idoc-operations).

1. Guarde la aplicación lógica. En la barra de herramientas del diseñador, seleccione **Save** (Guardar).

### <a name="test-your-logic-app"></a>Comprobación de la aplicación lógica

1. En la barra de herramientas del diseñador, elija **Run** (Ejecutar) para desencadenar una ejecución de la aplicación lógica.

1. Abra la ejecución y compruebe las salidas de la acción **Generate schemas** (Generar esquemas).

   Las salidas muestran los esquemas generados para la lista de mensajes especificada.

### <a name="upload-schemas-to-an-integration-account"></a>Carga de esquemas en una cuenta de integración

Si lo desea, puede descargar o almacenar los esquemas generados en repositorios, como un blob, el almacenamiento o la cuenta de integración. Las cuentas de integración proporcionan una excelente experiencia con otras acciones de XML, por lo que en este ejemplo se muestra cómo cargar los esquemas en una cuenta de integración de la misma aplicación lógica mediante el conector de Azure Resource Manager.

1. En el Diseñador de aplicación lógica, seleccione **New Step** (Nuevo paso).

1. En el cuadro de búsqueda, escriba `Resource Manager` como filtro. Seleccione **Create or update a resource** (Crear o actualizar un recurso).

   ![Selección de la acción de Azure Resource Manager](media/logic-apps-using-sap-connector/select-azure-resource-manager-action.png)

1. Escriba los detalles, incluida su suscripción a Azure, el grupo de recursos de Azure y la cuenta de integración. Para agregar tokens de SAP a los campos, haga clic dentro del cuadro de esos campos y seleccione de la lista de contenido dinámico que aparece.

   1. Abra la lista **Agregar nuevo parámetro** y seleccione los campos **Ubicación** y **Propiedades**.

   1. Proporcione detalles de estos nuevos campos como se muestra en este ejemplo.

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

1. Guarde la aplicación lógica. En la barra de herramientas del diseñador, seleccione **Save** (Guardar).

### <a name="test-your-logic-app"></a>Comprobación de la aplicación lógica

1. En la barra de herramientas del diseñador, elija **Run** (Ejecutar) para desencadenar manualmente la aplicación lógica.

1. Tras ejecutarse correctamente, vaya a la cuenta de integración y compruebe que existen los esquemas generados.

## <a name="enable-secure-network-communications"></a>Habilitación de las comunicaciones de red seguras

Antes de comenzar, asegúrese de que se han cumplido los [requisitos previos](#pre-reqs) enumerados:

* La puerta de enlace de datos local está instalada en una máquina que se encuentra en la misma red que el sistema SAP.
* Para el inicio de sesión único, la puerta de enlace se ejecuta con un usuario que se asigna a un usuario de SAP.
* La biblioteca de SNC que proporciona las funciones de seguridad adicionales está instalada en la misma máquina que la puerta de enlace de datos. Algunos ejemplos incluyen [sapseculib](https://help.sap.com/saphelp_nw74/helpdata/en/7a/0755dc6ef84f76890a77ad6eb13b13/frameset.htm), Kerberos y NTLM.

   Para habilitar SNC en las solicitudes a o desde el sistema SAP, active la casilla **Usar SNC** en la conexión de SAP y proporcione estas propiedades:

   ![Configuración de SAP SNC en la conexión](media/logic-apps-using-sap-connector/configure-sapsnc.png)

   | Propiedad | Descripción |
   |----------| ------------|
   | **SNC Library Path** (Ruta de acceso a la biblioteca de SNC) | El nombre de la biblioteca de SNC o la ruta de acceso relativa a la ubicación de instalación de NCo o la ruta de acceso absoluta. Algunos ejemplos son `sapsnc.dll`, `.\security\sapsnc.dll` o `c:\security\sapsnc.dll`. |
   | **SNC SSO** (SSO de SNC) | Cuando se conecta mediante SNC, la identidad de SNC normalmente se usa para autenticar al autor de llamada. Otra opción consiste en invalidarla para que se pueda usar la información de usuario y contraseña para autenticar al autor de llamada, pero la línea siga cifrada. |
   | **SNC My Name** (Mi nombre de SNC) | En la mayoría de los casos, esta propiedad se puede omitir. La solución de SNC instalada conoce normalmente su propio nombre SNC. En el caso de soluciones que admitan varias identidades, puede que deba especificar la identidad que se usará para este destino o servidor en concreto. |
   | **SNC Partner Name** (Nombre del asociado de SNC) | El nombre del SNC back-end. |
   | **SNC Quality of Protection** (Calidad de protección de SNC) | La calidad de servicio que se usará para la comunicación SNC de este destino o servidor en particular. El sistema back-end define el valor predeterminado. El valor máximo lo define el producto de seguridad usado para SNC. |
   |||

   > [!NOTE]
   > No establezca las variables de entorno SNC_LIB y SNC_LIB_64 en la máquina donde tenga la puerta de enlace de datos y la biblioteca de SNC. Si lo hace, tienen prioridad sobre el valor de la biblioteca de SNC pasado a través del conector.

<a name="safe-typing"></a>

## <a name="safe-typing"></a>Escritura segura

De forma predeterminada, al crear la conexión de SAP se usa el establecimiento inflexible de tipos para comprobar si hay valores no válidos; para ello, se realiza la validación de XML con respecto al esquema. Este comportamiento puede ayudarlo a detectar problemas con antelación. La opción **Escritura segura** está disponible para la compatibilidad con versiones anteriores y solo comprueba la longitud de cadena. Si elige **Escritura segura**, los tipos DATS y TIMS en SAP se tratan como cadenas en lugar de como sus equivalentes XML, `xs:date` y `xs:time`, donde `xmlns:xs="http://www.w3.org/2001/XMLSchema"`. La escritura segura afecta al comportamiento de generación de todos los esquemas, al mensaje de envío de "se ha enviado" la carga y de "se ha recibido" la respuesta y al desencadenador. 

Cuando se usa el establecimiento inflexible de tipos (la opción **Escritura segura** no está habilitada), el esquema asigna los tipos DATS y TIMS a tipos XML más sencillos:

```xml
<xs:element minOccurs="0" maxOccurs="1" name="UPDDAT" nillable="true" type="xs:date"/>
<xs:element minOccurs="0" maxOccurs="1" name="UPDTIM" nillable="true" type="xs:time"/>
```

Cuando se envían mensajes con establecimiento inflexible de tipos, la respuesta DATS y TIMS se adapta al formato del tipo XML coincidente:

```xml
<DATE>9999-12-31</DATE>
<TIME>23:59:59</TIME>
```

Cuando la opción **Escritura segura** está habilitada, el esquema solo asigna los tipos DATS y TIMS a campos de cadena XML con restricciones de longitud, por ejemplo:

```xml
<xs:element minOccurs="0" maxOccurs="1" name="UPDDAT" nillable="true">
  <xs:simpleType>
    <xs:restriction base="xs:string">
      <xs:maxLength value="8" />
    </xs:restriction>
  </xs:simpleType>
</xs:element>
<xs:element minOccurs="0" maxOccurs="1" name="UPDTIM" nillable="true">
  <xs:simpleType>
    <xs:restriction base="xs:string">
      <xs:maxLength value="6" />
    </xs:restriction>
  </xs:simpleType>
</xs:element>
```

Cuando se envían mensajes con **Escritura segura** habilitada, la respuesta DATS y TIMS se parece a la de este ejemplo:

```xml
<DATE>99991231</DATE>
<TIME>235959</TIME>
```

## <a name="advanced-scenarios"></a>Escenarios avanzados

### <a name="confirm-transaction-explicitly"></a>Confirmación de la transacción explícitamente

Cuando se envían transacciones a SAP desde Logic Apps, este intercambio se produce en dos pasos, tal como se describe en el documento de SAP [Transactional RFC Server Programs](https://help.sap.com/doc/saphelp_nwpi71/7.1/en-US/22/042ad7488911d189490000e829fbbd/content.htm?no_cache=true) (Programas del servidor RFC transaccional). De manera predeterminada, la acción **Enviar a SAP** controla ambos pasos para la transferencia de la función y la confirmación de la transacción en una sola llamada. El conector de SAP ofrece la opción de desacoplar estos pasos. Puede enviar un IDOC y, en lugar de confirmar automáticamente la transacción, puede usar la acción explícita **Confirmar id. de transacción**.

Esta capacidad de desacoplar la confirmación del id. de transacción resulta útil cuando no quiere duplicar las transacciones en SAP, por ejemplo, en escenarios en los que se pueden producir errores debido a causas como problemas de red. Al confirmar el id. de la transacción por separado, la transacción solo se completa una vez en el sistema SAP.

Este es un ejemplo que muestra este patrón:

1. Cree una aplicación lógica en blanco y agregue un desencadenador HTTP.

1. En el conector de SAP, agregue la acción **Enviar IDOC**. Proporcione los detalles del IDOC que envía al sistema SAP.

1. Para confirmar explícitamente el ID de la transacción en un paso separado, en el campo **Confirmar TID**, seleccione **No**. Para el campo opcional **GUID del id. de transacción**, puede especificar manualmente el valor o hacer que el conector genere y devuelva automáticamente este GUID en la respuesta de la acción Enviar IDOC.

   ![Propiedades de la acción Enviar IDOC](./media/logic-apps-using-sap-connector/send-idoc-action-details.png)

1. Para confirmar explícitamente el id. de transacción, agregue la acción **Confirmar id. de transacción**. Haga clic en el cuadro **Id. de transacción** para que aparezca la lista de contenido dinámico. En esa lista, seleccione el valor de **Id. de transacción** que devuelve la acción **Enviar IDOC**.

   ![Acción Confirmar id. de transacción](./media/logic-apps-using-sap-connector/explicit-transaction-id.png)

   Después de ejecutar este paso, la transacción actual se marca como completa en ambos extremos, en el lado del conector de SAP y en el lado del sistema SAP.

## <a name="known-issues-and-limitations"></a>Limitaciones y problemas conocidos

Estos son los problemas y limitaciones actualmente conocidos para el conector SAP:

* El desencadenador SAP no admite clústeres de puerta de enlace de datos. En algunos casos de conmutación por error, el nodo de puerta de enlace de datos que se comunica con el sistema SAP puede diferir del nodo activo, lo que produce un comportamiento inesperado. En los escenarios de envío, se admiten clústeres de puerta de enlace de datos.

* El conector SAP no admite actualmente las cadenas de enrutador SAP. La puerta de enlace de datos local debe existir en la misma LAN que el sistema SAP al que quiere conectarse.

## <a name="connector-reference"></a>Referencia de conectores

Para obtener información técnica acerca de los desencadenadores, las acciones y los límites, que se detallan en la descripción de OpenAPI (antes Swagger) del conector, consulte la [página de referencia de los conectores](/connectors/sap/).

## <a name="next-steps"></a>Pasos siguientes

* [Conéctese a sistemas locales](../logic-apps/logic-apps-gateway-connection.md) desde Azure Logic Apps.
* Aprenda a validar, transformar y usar otras operaciones de mensaje con [Enterprise Integration Pack](../logic-apps/logic-apps-enterprise-integration-overview.md).
* Conozca otros [conectores de Logic Apps](../connectors/apis-list.md).
