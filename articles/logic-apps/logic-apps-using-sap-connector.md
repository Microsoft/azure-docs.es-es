---
title: Conexión a sistemas SAP
description: Cómo acceder a recursos de SAP y administrarlos mediante la automatización de flujos de trabajo con Azure Logic Apps
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: estfan, daviburg, logicappspm
ms.topic: article
ms.date: 07/21/2020
tags: connectors
ms.openlocfilehash: a0f6af706a81db537b9ed66dc49996282c4dbbaa
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87833902"
---
# <a name="connect-to-sap-systems-from-azure-logic-apps"></a>Conexión a sistemas SAP desde Azure Logic Apps

> [!IMPORTANT]
> Los conectores anteriores de SAP Application Server y SAP Message Server están en desuso desde el 29 de febrero de 2020. El conector de SAP actual consolida estos conectores de SAP anteriores para que no tenga que cambiar el tipo de conexión, es totalmente compatible con los conectores anteriores, proporciona muchas capacidades adicionales y sigue usando la biblioteca del conector SAP .NET ( SAP NCo).
>
> En el caso de las aplicaciones lógicas que usan los conectores más antiguos, [migre al conector más reciente](#migrate) antes de la fecha de desuso. De lo contrario, estas aplicaciones lógicas experimentarán errores de ejecución y no podrán enviar mensajes al sistema SAP.

En este artículo, se muestra cómo acceder a los recursos de SAP locales desde una aplicación lógica mediante el conector SAP. El conector funciona con las versiones clásicas de SAP, como los sistemas R/3 y ECC del entorno local. También permite la integración con los sistemas SAP más nuevos basados en HANA, como S/4 HANA, dondequiera que estén alojados, en el entorno local o en la nube. El conector SAP admite la integración de mensajes o datos hacia y desde sistemas basados en SAP NetWeaver a través de un documento intermedio (IDOC), de Business Application Programming Interface (BAPI) o de Remote Function Call (RFC).

El conector SAP emplea la [biblioteca de SAP .NET Connector (NCo)](https://support.sap.com/en/product/connectors/msnet.html) y proporciona estas acciones:

* **Enviar mensaje a SAP**: envío de IDOC a través de tRFC, llamada a funciones BAPI a través de RFC o llamada a sistemas SAP RFC/tRFC.

* **Cuando se recibe un mensaje de SAP**: recepción de IDOC a través de tRFC, llamada a funciones BAPI a través de tRFC o llamada a RFC/tRFC en sistemas SAP.

* **Generate schemas** (Generar esquemas): generación de esquemas de artefactos SAP para IDOC, BAPI o RFC.

En todas las operaciones anteriores, el conector SAP admite autenticación básica con nombre de usuario y contraseña. El conector también admite [comunicaciones de red seguras (SNC)](https://help.sap.com/doc/saphelp_nw70/7.0.31/e6/56f466e99a11d1a5b00000e835363f/content.htm?no_cache=true). SNC puede usarse con el inicio de sesión único (SSO) de SAP NetWeaver-o con funcionalidades de seguridad adicionales proporcionadas por un producto de seguridad externo.

En este artículo se muestra cómo crear aplicaciones lógicas que se integren con SAP y se tratan los escenarios de integración descritos anteriormente. En el caso de las aplicaciones lógicas que usan los conectores de SAP más antiguos, en este artículo se muestra cómo migrar las aplicaciones lógicas a la versión más reciente del conector de SAP.

<a name="pre-reqs"></a>

## <a name="prerequisites"></a>Requisitos previos

Para seguir con este artículo, necesita los siguientes elementos:

* Suscripción a Azure. Si aún no tiene ninguna suscripción de Azure, [regístrese para obtener una cuenta gratuita de Azure](https://azure.microsoft.com/free/).

* La aplicación lógica desde donde quiere obtener acceso al sistema SAP y un desencadenador que inicie el flujo de trabajo de la aplicación lógica. Si no está familiarizado con las aplicaciones lógicas, consulte [¿Qué es Azure Logic Apps?](../logic-apps/logic-apps-overview.md) y el [Inicio rápido: Creación de la primera aplicación lógica](../logic-apps/quickstart-create-first-logic-app-workflow.md).

* El [servidor de aplicaciones de SAP](https://wiki.scn.sap.com/wiki/display/ABAP/ABAP+Application+Server) o el [servidor de mensajes de SAP](https://help.sap.com/saphelp_nw70/helpdata/en/40/c235c15ab7468bb31599cc759179ef/frameset.htm).

* El contenido del mensaje que envía a su servidor SAP, como un archivo IDoc de ejemplo, debe estar en formato XML e incluir el espacio de nombres de la acción de SAP que quiera usar.

* Para usar el desencadenador **Cuando se recibe un mensaje de SAP**, también debe realizar estos pasos de configuración:
  
  > [!NOTE]
  > Este desencadenador utiliza la misma ubicación de URI para renovar y cancelar una suscripción de webhook. La operación de renovación usa el método `PATCH` de HTTP, mientras que la operación de cancelación usa el método `DELETE` de HTTP. Este comportamiento puede provocar que una operación de renovación aparezca como una operación de cancelación en el historial del desencadenador, pero la operación sigue siendo una renovación porque el desencadenador usa `PATCH` como método HTTP, no `DELETE`.

  * Configure los permisos de seguridad de la puerta de enlace de SAP con esta opción:

    `"TP=Microsoft.PowerBI.EnterpriseGateway HOST=<gateway-server-IP-address> ACCESS=*"`

  * Configure el registro de seguridad de la puerta de enlace de SAP, con el que es más fácil encontrar errores de lista de control de acceso (ACL) y no está habilitado de forma predeterminada. De lo contrario, obtendrá el siguiente error:

    `"Registration of tp Microsoft.PowerBI.EnterpriseGateway from host <host-name> not allowed"`

    Para más información, vea el tema de ayuda de SAP [Configurar el registro de la puerta de enlace](https://help.sap.com/erp_hcm_ias2_2015_02/helpdata/en/48/b2a710ca1c3079e10000000a42189b/frameset.htm).

<a name="multi-tenant"></a>

### <a name="multi-tenant-azure-prerequisites"></a>Requisitos previos de instancias multiinquilino de Azure

Estos requisitos previos se aplican cuando las aplicaciones lógicas se ejecutan en instancias multiinquilino de Azure y desea usar el conector de SAP administrado, que no se ejecuta de forma nativa en un [Entorno del servicio de integración (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md). En caso contrario, si usa un ISE de nivel Premium y quiere usar el conector de SAP que se ejecuta de forma nativa en el ISE, vea [Requisitos previos del Entorno del servicio de integración (ISE)](#sap-ise).

El conector de SAP administrado (que no sea ISE) se integra con los sistemas SAP locales a través de la [puerta de enlace de datos local](../logic-apps/logic-apps-gateway-connection.md). Por ejemplo, en escenarios de envío de mensajes, al enviar un mensaje desde una aplicación lógica hasta un sistema SAP, la puerta de enlace de datos actúa como cliente RFC y reenvía las solicitudes recibidas de la aplicación lógica a SAP. Del mismo modo, en escenarios de recepción de mensajes, la puerta de enlace de datos actúa como servidor RFC que recibe solicitudes de SAP y las reenvía a la aplicación lógica.

* [Descargue e instale la puerta de enlace de datos local](../logic-apps/logic-apps-gateway-install.md) en el equipo local. Después, [cree un recurso de puerta de enlace de Azure](../logic-apps/logic-apps-gateway-connection.md#create-azure-gateway-resource) para esa puerta de enlace en Azure Portal. La puerta de enlace le ayuda a acceder de forma segura a los datos y recursos que se encuentran en el entorno local.

  Como procedimiento recomendado, asegúrese de usar una versión compatible de la puerta de enlace de datos local. Microsoft publica una nueva versión cada mes. Actualmente, Microsoft admite las últimas seis versiones. Si tiene problemas con la puerta de enlace, intente [actualizar a la versión más reciente](https://aka.ms/on-premises-data-gateway-installer), que puede incluir actualizaciones para resolver el problema.

* [Descargue e instale la biblioteca más actualizada de clientes de SAP](#sap-client-library-prerequisites) en el mismo equipo que la puerta de enlace de datos local.

<a name="sap-ise"></a>

### <a name="integration-service-environment-ise-prerequisites"></a>Requisitos previos del Entorno del servicio de integración (ISE)

Estos requisitos previos se aplican cuando las aplicaciones lógicas se ejecutan en un [Entorno del servicio de integración (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md) de nivel Premium (no de nivel de desarrollador) y quiere usar el conector de SAP que se ejecuta de forma nativa en un ISE. Un ISE proporciona acceso a los recursos que están protegidos por una red virtual de Azure y ofrece otros conectores nativos del ISE que permiten a las aplicaciones lógicas acceder directamente a los recursos locales sin usar la puerta de enlace de datos local.

> [!NOTE]
> Aunque el conector del ISE de SAP está visible dentro de un ISE de nivel de desarrollador, los intentos para instalar el conector no se realizarán correctamente.

1. Si aún no tiene una cuenta de Azure Storage y un contenedor de blobs, cree ese contenedor mediante [Azure Portal](../storage/blobs/storage-quickstart-blobs-portal.md) o el [Explorador de Azure Storage](../storage/blobs/storage-quickstart-blobs-storage-explorer.md).

1. [Descargue e instale la biblioteca más actualizada de clientes de SAP](#sap-client-library-prerequisites) en el equipo local. Debe tener los siguientes archivos de ensamblado:

   * libicudecnumber.dll
   * rscp4n.dll
   * sapnco.dll
   * sapnco_utils.dll

1. Cree un archivo. zip que incluya estos ensamblados y cargue este paquete en el contenedor de blobs en Azure Storage.

1. En Azure Portal o el Explorador de Azure Storage, vaya a la ubicación del contenedor donde cargó el archivo. zip.

1. Copie la URL de esa ubicación y asegúrese de incluir el token de firma de acceso compartido (SAS).

   De lo contrario, el token de SAS no se autoriza y se produce un error en la implementación del conector del ISE de SAP.

1. Para poder usar el conector del ISE de SAP, debe instalar e implementar el conector en el ISE.

   1. En [Azure Portal](https://portal.azure.com), busque y abra el ISE.
   
   1. En el menú del ISE, seleccione **Managed connectors (Conectores administrados)**  > **Agregar**. En la lista de conectores, busque y seleccione **SAP**.
   
   1. En el panel **Add a new managed connector** (Agregar un nuevo conector administrado), en el cuadro **SAP package** (Paquete de SAP), pegue la dirección URL del archivo .zip que contiene los ensamblados de SAP. *Asegúrese de incluir el token de SAS.*

   1. Seleccione **Crear** cuando haya terminado.

   Para más información, vea [Adición de conectores del ISE](../logic-apps/add-artifacts-integration-service-environment-ise.md#add-ise-connectors-environment).

1. Si su instancia de SAP y el ISE se encuentran en distintas redes virtuales, también debe [emparejar esas redes](../virtual-network/tutorial-connect-virtual-networks-portal.md) para que la red virtual del ISE esté conectada a la red virtual de la instancia de SAP.

<a name="sap-client-library-prerequisites"></a>

### <a name="sap-client-library-prerequisites"></a>Requisitos previos de la biblioteca de cliente de SAP

* Instale la versión más reciente: [SAP Connector (NCo 3.0) para Microsoft .NET 3.0.22.0 compilado con .NET Framework 4.0  - Windows de 64 bits (x64)](https://softwaredownloads.sap.com/file/0020000001000932019). Las versiones anteriores pueden dar lugar a problemas de compatibilidad. Para más información, vea [Versiones de la biblioteca de cliente de SAP](#sap-library-versions).

* De forma predeterminada, el instalador de SAP coloca los archivos de ensamblado en la carpeta de instalación predeterminada. Debe copiar estos archivos de ensamblado en otra ubicación, en función del escenario que se indica aquí:

  * En el caso de las aplicaciones lógicas que se ejecutan en un ISE, siga los pasos descritos en [Requisitos previos del Entorno del servicio de integración (ISE)](#sap-ise). En el caso de las aplicaciones lógicas que se ejecutan en instancias multiinquilino de Azure y usan la puerta de enlace de datos local, copie los archivos de ensamblado de la carpeta de instalación predeterminada en la carpeta de instalación de la puerta de enlace de datos. Si surgen problemas con la puerta de enlace de datos, revise estos problemas:

  * Debe instalar la versión de 64 bits para la biblioteca de cliente de SAP, ya que la puerta de enlace de datos solo se ejecuta en sistemas de 64 bits. De lo contrario, obtendrá un error de "imagen incorrecta" porque el servicio de host de la puerta de enlace de datos no es compatible con los ensamblados de 32 bits.

  * Si se produce un error en la conexión de SAP con un mensaje que le pide que compruebe la información de la cuenta o los permisos y pruebe otra vez, es posible que los archivos de ensamblado estén en la ubicación incorrecta. Asegúrese de que ha copiado los archivos de ensamblado en la carpeta de instalación de la puerta de enlace de datos.

    Para ayudarle a solucionar problemas, [use el visor de registro de enlaces de ensamblados de .NET](/dotnet/framework/tools/fuslogvw-exe-assembly-binding-log-viewer), que permite comprobar que los archivos de ensamblado se encuentran en la ubicación correcta. Si lo prefiere, puede seleccionar la opción **Global Assembly Cache registration** (Registro de la caché global de ensamblados) al instalar la biblioteca de clientes de SAP.

<a name="sap-library-versions"></a>

#### <a name="sap-client-library-versions"></a>Versiones de la biblioteca de clientes de SAP

Las versiones anteriores de SAP NCo pueden experimentar un interbloqueo cuando se envía más de un mensaje de IDOC al mismo tiempo. Esta condición bloquea todos los mensajes posteriores que se envían al destino de SAP, lo que hace que se agote el tiempo de espera de los mensajes.

Estas son las relaciones entre la biblioteca de clientes de SAP, .NET Framework, el entorno de ejecución de .NET y la puerta de enlace:

* Tanto Microsoft SAP Adapter como el servicio de host de la puerta de enlace usan .NET Framework 4.5.

* NCo de SAP para .NET Framework 4.0 funciona con procesos que usan las versiones 4.0 a 4.7.1 del runtime de .NET.

* NCo de SAP para .NET Framework 2.0 funciona con procesos que usan las versiones 2.0 a 3.5 del entorno de ejecución de .NET y ya no funciona con la puerta de enlace más reciente.

### <a name="secure-network-communications-prerequisites"></a>Requisitos previos de comunicaciones de red seguras

Si usa la puerta de enlace de datos local con las comunicaciones de red segura (SNC) opcionales, que solo se admiten en instancias multiinquilino de Azure, también debe configurar estas opciones:

* Si usa SNC con el inicio de sesión seguro (SSO), asegúrese de que la puerta de enlace de datos se ejecuta con un usuario asignado con el usuario de SAP. Para cambiar la cuenta predeterminada, seleccione **Cambiar cuenta** y escriba las credenciales de usuario.

  ![Cambio de cuenta de la puerta de enlace de datos](./media/logic-apps-using-sap-connector/gateway-account.png)

* Si habilita SNC con un producto de seguridad externo, copie la biblioteca o los archivos de SNC en el mismo equipo donde está instalada la puerta de enlace de datos. Algunos ejemplos de productos SNC incluyen [sapseculib](https://help.sap.com/saphelp_nw74/helpdata/en/7a/0755dc6ef84f76890a77ad6eb13b13/frameset.htm), Kerberos y NTLM.

Para más información sobre la habilitación de SNC para la puerta de enlace de datos, vea [Habilitación de las comunicaciones de red seguras](#secure-network-communications).

<a name="migrate"></a>

## <a name="migrate-to-current-connector"></a>Migración al conector actual

Para migrar desde un conector de SAP administrado (que no sea ISE) anterior al conector de SAP administrado actual, siga estos pasos:

1. Si todavía no lo ha hecho, actualice la [puerta de enlace de datos local](https://www.microsoft.com/download/details.aspx?id=53127) para que tenga la versión más reciente. Para más información, consulte [Instalación de una puerta de enlace de datos local para Azure Logic Apps](../logic-apps/logic-apps-gateway-install.md).

1. En la aplicación lógica que usa el conector de SAP anterior, elimine la acción **Enviar a SAP**.

1. En el conector de SAP más reciente, agregue la acción **Enviar mensaje a SAP**. Para poder usar esta acción, vuelva a crear la conexión al sistema SAP.

1. Cuando haya terminado, guarde la aplicación lógica.

<a name="add-trigger"></a>

## <a name="send-message-to-sap"></a>Enviar mensaje a SAP

En este ejemplo, se utiliza una aplicación lógica que se puede desencadenar con una solicitud HTTP. La aplicación lógica envía un IDOC a un servidor SAP y devuelve una respuesta al solicitante que llamó a la aplicación lógica.

### <a name="add-an-http-request-trigger"></a>Adición de un desencadenador de solicitud HTTP

En Azure Logic Apps, cada aplicación lógica debe comenzar con un [desencadenador](../logic-apps/logic-apps-overview.md#logic-app-concepts), que se activa cuando sucede un evento específico o cuando se cumple una condición determinada. Cada vez que el desencadenador se activa, el motor de Logic Apps crea una instancia de aplicación lógica y empieza a ejecutar el flujo de trabajo de la aplicación.

> [!NOTE]
> Cuando una aplicación lógica recibe paquetes IDoc de SAP, el [desencadenador de solicitudes](../connectors/connectors-native-reqres.md) no admite el esquema XML "sin formato" generado por la documentación de IDoc WE60 de SAP. Aun así, el esquema XML "sin formato" es compatible con los escenarios que envían mensajes de aplicaciones lógicas *a* SAP. Puede usar el desencadenador de solicitudes con el XML del IDoc de SAP, pero no con IDoc sobre RFC. Si lo prefiere, puede transformar el XML al formato necesario. 

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

1. Si la conexión ya existe, continúe con el paso siguiente para configurar la acción de SAP. Pero si se le piden los detalles de la conexión, proporciónelos para que pueda crear una conexión al servidor SAP local.

   1. Proporcione un nombre para la conexión.

   1. Si usa la puerta de enlace de datos, siga estos pasos:
   
      1. En la sección **Puerta de enlace de datos**, en **Suscripción**, primero seleccione la suscripción de Azure para el recurso de puerta de enlace de datos que creó en el Azure Portal para la instalación de su puerta de enlace de datos.
   
      1. En **Puerta de enlace de conexión**, seleccione su recurso de puerta de enlace de datos en Azure.

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

      Para más información sobre las operaciones de IDoc, vea [Esquemas de mensaje para operaciones de IDoc](/biztalk/adapters-and-accelerators/adapter-sap/message-schemas-for-idoc-operations).

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

#### <a name="add-rfc-request-response"></a>Incorporación de una solicitud-respuesta de RFC

> [!NOTE]
> El desencadenador de SAP recibe IDoc a través de tRFC, que no tiene un parámetro de respuesta por diseño. 

Debe crear un patrón de solicitud y respuesta si necesita recibir respuestas mediante una llamada de función remota (RFC) a Logic Apps de ABAP para SAP. Para recibir IDoc en la aplicación lógica, debe hacer que la primera acción sea una [solicitud HTTP](../connectors/connectors-native-reqres.md#add-a-response-action) con un código de estado `200 OK` y ningún contenido. Este paso recomendado completa la transferencia asincrónica de LUW para SAP a través de tRFC inmediatamente, lo que deja la conversación de CPIC para SAP disponible de nuevo. Después, puede agregar otras acciones en la aplicación lógica para procesar el IDoc recibido sin bloquear más transferencias adicionales.

Para implementar un patrón de solicitud y respuesta, primero debe detectar el esquema RFC mediante el [comando `generate schema`](#generate-schemas-for-artifacts-in-sap). El esquema generado tiene dos posibles nodos raíz: 

1. El nodo de solicitud, que es la llamada que recibe de SAP.
1. El nodo de respuesta, que es la respuesta a SAP.

En el ejemplo siguiente, se genera un patrón de solicitud y respuesta desde el módulo RFC `STFC_CONNECTION`. Se analiza el código XML de la solicitud para extraer un valor de nodo en el que SAP solicita `<ECHOTEXT>`. La respuesta inserta la marca de tiempo actual como un valor dinámico. Se recibirá una respuesta similar cuando envíe una llamada de función remota `STFC_CONNECTION` desde una aplicación lógica a SAP.

```http

<STFC_CONNECTIONResponse xmlns="http://Microsoft.LobServices.Sap/2007/03/Rfc/">
  <ECHOTEXT>@{first(xpath(xml(triggerBody()?['Content']), '/*[local-name()="STFC_CONNECTION"]/*[local-name()="REQUTEXT"]/text()'))}</ECHOTEXT>
  <RESPTEXT>Azure Logic Apps @{utcNow()}</RESPTEXT>


```

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

   1. Si usa la puerta de enlace de datos, siga estos pasos:

      1. En la sección **Puerta de enlace de datos**, en **Suscripción**, primero seleccione la suscripción de Azure para el recurso de puerta de enlace de datos que creó en el Azure Portal para la instalación de su puerta de enlace de datos.

      1. En **Puerta de enlace de conexión**, seleccione su recurso de puerta de enlace de datos en Azure.

   1. Siga proporcionando información acerca de la conexión. Para la propiedad **Tipo de inicio de sesión**, siga el paso en función de si la propiedad está establecida en **Servidor de aplicaciones** o **Grupo**:

      * Para **Servidos de aplicaciones**, estas propiedades, que generalmente parecen opcionales, son necesarias:

        ![Creación de conexiones del servidor de aplicaciones de SAP](media/logic-apps-using-sap-connector/create-SAP-application-server-connection.png)

      * Para **Grupo**, estas propiedades, que generalmente parecen opcionales, son necesarias:

        ![Creación de conexiones del servidor de mensajes de SAP](media/logic-apps-using-sap-connector/create-SAP-message-server-connection.png)

      De forma predeterminada, se usa el establecimiento inflexible de tipos para comprobar si hay valores no válidos, para lo cual se realiza la validación de XML con respecto al esquema. Este comportamiento puede ayudarlo a detectar problemas con antelación. La opción **Escritura segura** está disponible para la compatibilidad con versiones anteriores y solo comprueba la longitud de cadena. Más información sobre la opción [Escritura segura](#safe-typing).

   1. Cuando haya finalizado, seleccione **Crear**.

      Logic Apps configura y comprueba la conexión para asegurarse de que funciona correctamente.

1. Proporcione los [parámetros necesarios](#parameters) en función de la configuración del sistema SAP. 

   Puede [filtrar los mensajes que recibe desde el servidor SAP especificando una lista de acciones de SAP](#filter-with-sap-actions).

   Puede seleccionar una acción de SAP desde el selector de archivos:

   ![Agregar acción de SAP a la aplicación lógica](media/logic-apps-using-sap-connector/select-SAP-action-trigger.png)  

   Si lo prefiere, puede especificar una acción manualmente:

   ![Introducción manual de una acción de SAP](media/logic-apps-using-sap-connector/manual-enter-SAP-action-trigger.png)

   Este es un ejemplo que muestra la apariencia de la acción cuando se configura el desencadenador para recibir varios mensajes.

   ![Ejemplo de desencadenador que recibe múltiples mensajes](media/logic-apps-using-sap-connector/example-trigger.png)

   Para más información sobre la acción de SAP, vea [Esquemas de mensaje para operaciones de IDoc](/biztalk/adapters-and-accelerators/adapter-sap/message-schemas-for-idoc-operations).

1. Ahora guarde la aplicación lógica para poder empezar a recibir mensajes del sistema SAP. En la barra de herramientas del diseñador, seleccione **Save** (Guardar).

La aplicación lógica está preparada para recibir mensajes del sistema SAP.

> [!NOTE]
> El desencadenador de SAP no es un desencadenador de sondeo, sino un desencadenador basado en webhook. Si usa la puerta de enlace de datos, el desencadenador se llama desde la puerta de enlace de datos solo cuando existe un mensaje, por lo que no es necesario realizar ningún sondeo.

<a name="parameters"></a>

#### <a name="parameters"></a>Parámetros

Junto con entradas de cadena y número simples, el conector de SAP acepta estos parámetros de tabla (entradas `Type=ITAB`):

* Parámetros de dirección de tabla, tanto de entrada como de salida, para versiones anteriores de SAP.
* Parámetros de cambio, que reemplazan los parámetros de dirección de la tabla para las versiones más recientes de SAP.
* Parámetros de tabla jerárquica

<a name="filter-with-sap-actions"></a>

#### <a name="filter-with-sap-actions"></a>Filtrado con acciones de SAP

Opcionalmente, puede filtrar los mensajes que la aplicación lógica recibe del servidor SAP proporcionando una lista, o una matriz, con una o varias acciones de SAP. De forma predeterminada, esta matriz está vacía, lo que significa que la aplicación lógica recibe todos los mensajes del servidor SAP sin filtrar. 

Al configurar el filtro de matriz, el desencadenador solo recibe mensajes de los tipos de acción de SAP especificados y rechaza todos los demás mensajes del servidor SAP. Sin embargo, este filtro no afecta al establecimiento de tipos de la carga recibida, que puede ser flexible o inflexible.

Cualquier filtrado de acciones de SAP se produce en el nivel del adaptador de SAP para la puerta de enlace de datos local. Para obtener más información, vea [Envío de IDoc de prueba a Logic Apps desde SAP](#send-idocs-from-sap).

Si no puede enviar paquetes de IDoc desde SAP al desencadenador de la aplicación lógica, consulte el mensaje de rechazo de llamada de RFC transaccional (tRFC) en el cuadro de diálogo tRFC de SAP (T-CODE SM58). En la interfaz de SAP, puede recibir los siguientes mensajes de error, que se recortan debido a los límites de subcadena del campo **Status Text** (Texto de estado).

* `The RequestContext on the IReplyChannel was closed without a reply being`: Se producen errores inesperados cuando el controlador comodín del canal finaliza el canal debido a un error y vuelve a generar el canal para procesar otros mensajes.

  * Para confirmar que la aplicación lógica recibió el IDoc, [agregue una acción de respuesta](../connectors/connectors-native-reqres.md#add-a-response-action) que devuelva un código de estado `200 OK`. El IDoc se transporta a través de tRFC, que no permite una carga de respuesta.

  * Si en vez de eso tiene que rechazar el IDoc, responda con cualquier código de Estado HTTP distinto de `200 OK` para que el adaptador de SAP devuelva una excepción a SAP en su nombre. 

* `The segment or group definition E2EDK36001 was not found in the IDoc meta`: Los errores esperados se producen con otros errores, como el error de generación de una carga XML de IDoc provocado por el hecho de que SAP no publica sus segmentos, por lo que faltan los metadatos de tipo de segmento necesarios para la conversión. 

  * Para que SAP publique estos segmentos, póngase en contacto con el ingeniero de ABAP para su sistema SAP.

<a name="find-extended-error-logs"></a>

## <a name="find-extended-error-logs"></a>Búsqueda de registros de errores ampliados

Para ver los mensajes de error completos, recurra a los registros ampliados del adaptador de SAP. 

En el caso de las versiones de puerta de enlace de datos local de junio de 2020 y posteriores, puede [habilitar los registros de puerta de enlace en la configuración de la aplicación](/data-integration/gateway/service-gateway-tshoot#collect-logs-from-the-on-premises-data-gateway-app).

En el caso de las versiones de puerta de enlace de datos local de abril de 2020 y anteriores, los registros están deshabilitados de forma predeterminada. Para recuperar los registros ampliados, siga estos pasos:

1. En la carpeta de instalación de la puerta de enlace de datos local, abra el archivo `Microsoft.PowerBI.DataMovement.Pipeline.GatewayCore.dll.config`. 

1. Para el valor **SapExtendedTracing**, cambie el valor de **False** a **True**.

1. Opcionalmente, para menos eventos, cambie el valor **SapTracingLevel** de **Informativo** (valor predeterminado) a **Error** o **Advertencia**. O bien, para obtener más eventos, cambie de **Informativo** a **Detallado**.

1. Guarde el archivo de configuración.

1. Reinicie la puerta de enlace de datos. Abra la aplicación del instalador de puerta de enlace de datos local y vaya al menú **Valor de configuración del servicio**. En **Reinicio de la puerta de enlace**, seleccione **Reiniciar ahora**.

1. Reproduzca el problema.

1. Exporte los registros de puerta de enlace. En la aplicación del instalador de puerta de enlace de datos, vaya al menú **Diagnóstico**. En **Registros de la puerta de enlace**, seleccione **Exportar registros**. Estos archivos incluyen registros SAP organizados por fecha. En función del tamaño del registro, pueden existir varios archivos de registro para una sola fecha.

1. En el archivo de configuración, revierta el valor de **SapExtendedTracing** a **False**.

1. Reinicie el servicio de puerta de enlace.

### <a name="test-your-logic-app"></a>Comprobación de la aplicación lógica

1. Para desencadenar la aplicación lógica, envíe un mensaje desde el sistema SAP.

1. En el menú de la aplicación lógica, seleccione **Información general**. Revise el **historial de ejecuciones** para ver las nuevas ejecuciones de la aplicación lógica.

1. Abra la ejecución más reciente, que muestra el mensaje enviado desde su sistema SAP en la sección de salidas del desencadenador.

<a name="send-idocs-from-sap"></a>

### <a name="test-sending-idocs-from-sap"></a>Prueba de envío de IDoc desde SAP

Para enviar IDoc desde SAP a la aplicación lógica, necesita la siguiente configuración mínima:

> [!IMPORTANT]
> Siga estos pasos solo al probar la configuración de SAP con la aplicación lógica. Los entornos de producción requieren configuración adicional.

1. [Configuración de un destino RFC en SAP](#create-rfc-destination)

1. [Creación de una conexión ABAP con el destino RFC](#create-abap-connection)

1. [Creación de un puerto receptor](#create-receiver-port)

1. [Creación de un puerto remitente](#create-sender-port)

1. [Creación de un asociado del sistema lógico](#create-logical-system-partner)

1. [Creación de perfil de socio](#create-partner-profiles)

1. [Prueba de envío de mensajes](#test-sending-messages)

#### <a name="create-rfc-destination"></a>Creación de destino RFC

1. Para abrir la opción **Configuration of RFC Connections** (Configuración de las conexiones RFC), en la interfaz de SAP, use el código de transacción (código T) **sm59** con el prefijo **/n**.

1. Seleccione **TCP/IP Connections** > **Create** (Conexiones TCP/IP > Crear).

1. Cree un nuevo destino RFC con la siguiente configuración:
    
    * En **RFC Destination** (Destino RFC), escriba un nombre.
    
    * En la pestaña **Technical Settings** (Configuración técnica), en **Activation Type** (Tipo de activación), seleccione **Registered Server Program** (Programa de servidor registrado). En **Program ID** (Id. de programa), escriba un valor. En SAP, el desencadenador de la aplicación lógica se registrará con este identificador.
    
    * En la pestaña **Unicode**, para **Communication Type with Target System** (Tipo de comunicación con sistema de destino), seleccione **Unicode**.

1. Guarde los cambios.

1. Registre el nuevo **Program ID** (Id. de programa) con Azure Logic Apps.

1. Para probar la conexión, en la interfaz de SAP, en el nuevo **RFC Destination** (Destino RFC), seleccione **Connection Test** (Prueba de conexión).

#### <a name="create-abap-connection"></a>Creación de conexión ABAP

1. Para abrir los parámetros **Configuration of RFC Connections** (Configuración de las conexiones RFC), en la interfaz de SAP, use el código de transacción (código T) **sm59*** con el prefijo **/n**.

1. Seleccione **ABAP Connections** > **Create** (Conexiones ABAP > Crear).

1. **RFC Destination** (Destino RFC), escriba el identificador de [su sistema SAP de prueba](#create-rfc-destination).

1. Guarde los cambios.

1. Para probar la conexión, seleccione **Connection Test** (Prueba de conexión).

#### <a name="create-receiver-port"></a>Creación de un puerto receptor

1. Para abrir la configuración **Ports In IDOC processing** (Procesamiento de puertos en IDoc), en la interfaz de SAP, use el código de transacción (código T) **we21** con el prefijo **/n**.

1. Seleccione **Ports** > **Transactional RFC** > **Create** (Puertos > RFC transaccional > Crear).

1. En el cuadro de configuración que se abre, seleccione **own port name** (nombre propio del puerto). En el puerto de prueba, escriba un **nombre**. Guarde los cambios.

1. En la configuración del nuevo puerto receptor, para **RFC destination** (Destino RFC), escriba el identificador del [destino RFC de prueba](#create-rfc-destination).

1. Guarde los cambios.

#### <a name="create-sender-port"></a>Creación de un puerto remitente

1.  Para abrir la configuración **Ports In IDOC processing** (Procesamiento de puertos en IDoc), en la interfaz de SAP, use el código de transacción (código T) **we21** con el prefijo **/n**.

1. Seleccione **Ports** > **Transactional RFC** > **Create** (Puertos > RFC transaccional > Crear).

1. En el cuadro de configuración que se abre, seleccione **own port name** (nombre propio del puerto). En el puerto de prueba, escriba un **nombre** que empiece por **SAP**. Todos los nombres de puerto remitente deben comenzar con las letras **SAP**, por ejemplo, **SAPTEST**. Guarde los cambios.

1. En la configuración del nuevo puerto remitente, para **RFC destination** (Destino RFC), escriba el identificador de [la conexión de ABAP](#create-abap-connection).

1. Guarde los cambios.

#### <a name="create-logical-system-partner"></a>Creación de un asociado del sistema lógico

1. Para abrir la configuración para **cambiar la vista de sistemas lógicos referente al apartado de información general**, en la interfaz de SAP, use el código de transacción (código T) **bd54**.

1. Acepte el mensaje de advertencia que aparece: **Caution: The table is cross-client** (Precaución, la tabla afecta a varios clientes).

1. Encima de la lista que muestra los sistemas lógicos existentes, seleccione **New Entries** (Nuevas entradas).

1. En el nuevo sistema lógico, escriba un identificador de **Log.System** y una descripción breve del **nombre**. Guarde los cambios.

1. Cuando aparezca **Prompt for Workbench** (Solicitud de banco de trabajo), cree una nueva solicitud. Para ello, proporcione una descripción (si ya ha creado una solicitud, omita este paso).

1. Después de crear la solicitud de banco de trabajo, vincule esa solicitud a la solicitud de actualización de la tabla. Para confirmar que la tabla se ha actualizado, guarde los cambios.

#### <a name="create-partner-profiles"></a>Creación de perfiles de socios

En entornos de producción, debe crear dos perfiles de socio. El primer perfil es para el remitente, que es su organización y el sistema SAP. El segundo perfil es para el receptor, que es la aplicación lógica.

1. Para abrir la configuración **Partner profiles** (Perfiles de asociado), en la interfaz de SAP, use el código de transacción (código T) **we20** con el prefijo **/n**.

1. En **Partner Profiles** (Perfiles de socio), seleccione **Partner Type LS** > **Create** (Tipo de socio LS > Crear).

1. Cree un nuevo perfil de socio con la siguiente configuración:

    * En **Partner No.** (Núm. de socio), escriba el [identificador de socio del sistema lógico](#create-logical-system-partner).

    * En **Partn. Type** (Tipo de socio), escriba **LS**.

    * En **Agent** (Agente), escriba el identificador de la cuenta de usuario de SAP que utilizará al registrar identificadores de programa para Azure Logic Apps u otros sistemas que no sean de SAP.

1. Guarde los cambios. Si no ha [creado el socio del sistema lógico](#create-logical-system-partner), aparecerá el error **Enter a valid partner number** (Escriba un número de socio válido).

1. En la configuración del perfil de socio, en **Outbound parmtrs.** (Parámetros de salida), seleccione **Create outbound parameter** (Crear parámetro de salida).

1. Cree un nuevo parámetro de salida con la siguiente configuración:

    * Escriba su **tipo de mensaje**, por ejemplo, **CREMAS**.

    * Escriba el [identificador del puerto receptor](#create-receiver-port).

    * Escriba un tamaño de IDoc para **Pack. Tamaño**. O bien, para [enviar IDoc de uno en uno desde SAP](#receive-idoc-packets-from-sap), seleccione **Pass IDoc Immediately** (Pasar IDoc de inmediato).

1. Guarde los cambios.

#### <a name="test-sending-messages"></a>Prueba de envío de mensajes

1. Para abrir la configuración **Test Tool for IDoc Processing** (Procesamiento de la herramienta de prueba para IDoc), en la interfaz de SAP, use el código de transacción (código T) **we19** con el prefijo **/n**.

1. En **Template for test** (Plantilla para prueba), seleccione **Via message type** (Mediante tipo de mensaje) y escriba su tipo de mensaje; por ejemplo, **CREMAS**. Seleccione **Crear**.

1. Confirme el mensaje **Which IDoc type?** (¿Qué tipo de IDoc?) seleccionando **Continue** (Continuar).

1. Seleccione el nodo **EDIDC**. Escriba los valores adecuados para los puertos receptor y remitente. Seleccione **Continuar**.

1. Seleccione **Standard Outbound Processing** (Procesamiento de salida estándar).

1. Para iniciar el procesamiento de IDoc de salida, seleccione **Continue** (Continuar). Una vez finalizado el procesamiento, aparece el mensaje **IDoc sent to SAP system or external program** (IDoc enviado a sistema SAP o programa externo).

1.  Para comprobar si hay errores de procesamiento, use el código de transacción **sm58** con el prefijo **/n**.

## <a name="receive-idoc-packets-from-sap"></a>Recepción de paquetes IDoc de SAP

Puede configurar SAP para que [envíe IDoc en paquetes](https://help.sap.com/viewer/8f3819b0c24149b5959ab31070b64058/7.4.16/4ab38886549a6d8ce10000000a42189c.html), que son lotes o grupos de IDoc. Para recibir paquetes IDoc, el conector de SAP y, en concreto, el desencadenador, no necesitan una configuración adicional. Pero para procesar cada elemento en un paquete IDoc después de que el desencadenador reciba el paquete, se necesitan algunos pasos adicionales para dividir el paquete en IDoc individuales.

Este es un ejemplo que muestra cómo extraer IDoc individuales de un paquete mediante la [función `xpath()`](./workflow-definition-language-functions-reference.md#xpath):

1. Antes de empezar, necesita una aplicación lógica con un desencadenador de SAP. Si aún no tiene esta aplicación lógica, siga los pasos anteriores de este tema para configurar una [aplicación lógica con un desencadenador de SAP](#receive-from-sap).

   Por ejemplo:

   ![Agregar disparador SAP a la aplicación lógica](./media/logic-apps-using-sap-connector/first-step-trigger.png)

1. Obtenga el espacio de nombres raíz de IDoc XML que la aplicación lógica recibe de SAP. Para extraer este espacio de nombres del documento XML, agregue un paso que cree una variable de cadena local y almacene dicho espacio de nombres mediante una expresión `xpath()`:

   `xpath(xml(triggerBody()?['Content']), 'namespace-uri(/*)')`

   ![Obtención del espacio de nombres raíz del IDoc](./media/logic-apps-using-sap-connector/get-namespace.png)

1. Para extraer un IDoc individual, agregue un paso que cree una variable de matriz y almacene la colección de IDoc mediante otra expresión `xpath()`:

   `xpath(xml(triggerBody()?['Content']), '/*[local-name()="Receive"]/*[local-name()="idocData"]')`

   ![Obtención de una matriz de elementos](./media/logic-apps-using-sap-connector/get-array.png)

   La variable de matriz hace que cada IDoc esté disponible para que la aplicación lógica lo procese individualmente mediante la enumeración de la colección. En este ejemplo, la aplicación lógica transfiere cada IDoc a un servidor SFTP mediante un bucle:

   ![Envío del IDoc al servidor SFTP](./media/logic-apps-using-sap-connector/loop-batch.png)

   Cada IDoc debe incluir el espacio de nombres raíz, que es el motivo por el que el contenido del archivo se ajusta dentro de un elemento `<Receive></Receive` junto con el espacio de nombres raíz antes de enviar el IDoc a la aplicación de nivel inferior o servidor SFTP en este caso.

Puede usar la plantilla de inicio rápido para este patrón si selecciona esta plantilla en el diseñador de aplicaciones lógicas al crear una nueva aplicación lógica.

![Seleccionar plantilla de aplicación de lógica de lote](./media/logic-apps-using-sap-connector/select-batch-logic-app-template.png)

## <a name="generate-schemas-for-artifacts-in-sap"></a>Generar esquemas para artefactos en SAP

En este ejemplo, se utiliza una aplicación lógica que se puede desencadenar con una solicitud HTTP. Para generar los esquemas para los IDoc y BAPI especificados, la acción de SAP **Generate schema** (Generar esquema) envía una solicitud a un sistema SAP.

Esta acción de SAP devuelve un [esquema XML](#sample-xml-schemas), no el contenido o los datos del propio documento XML. Los esquemas que se devuelven en la respuesta se cargan en una cuenta de integración mediante el conector de Azure Resource Manager. Los esquemas contienen las siguientes partes:

* La estructura del mensaje de solicitud. Use esta información para formar la lista `get` de BAPI.
* La estructura del mensaje de respuesta. Utilice esta información para analizar la respuesta. 

Para enviar el mensaje de solicitud, use la acción genérica de SAP **Send message to SAP** (Enviar mensaje a SAP) o las acciones **Call BAPI** (Llamar a BAPI) de destino.

### <a name="sample-xml-schemas"></a>Esquemas XML de ejemplo

Si está aprendiendo a generar un esquema XML para usarlo en la creación de un documento de ejemplo, consulte los ejemplos siguientes. En estos ejemplos se muestra cómo puede trabajar con muchos tipos de cargas, entre las que se incluyen:

* [Solicitudes RFC](#xml-samples-for-rfc-requests)
* [Solicitudes BAPI](#xml-samples-for-bapi-requests)
* [Solicitudes IDoc](#xml-samples-for-idoc-requests)
* Tipos de datos de esquemas XML simples o complejos
* Parámetros de tabla
* Comportamientos XML opcionales

Puede comenzar el esquema XML con un prólogo XML opcional. El conector de SAP funciona con o sin el prólogo XML.

```xml

<?xml version="1.0" encoding="utf-8">

```

#### <a name="xml-samples-for-rfc-requests"></a>Ejemplos XML para solicitudes de RFC

El ejemplo siguiente es una llamada de RFC simple. El nombre de RFC es `STFC_CONNECTION`. Esta solicitud usa el espacio de nombres predeterminado `xmlns=`, pero puede asignar y utilizar alias de espacio de nombres como `xmmlns:exampleAlias=`. El valor del espacio de nombres es el espacio de nombres para todas las RFC en SAP para servicios de Microsoft. Hay un parámetro de entrada simple en la solicitud: `<REQUTEXT>`.

```xml

<STFC_CONNECTION xmlns="http://Microsoft.LobServices.Sap/2007/03/Rfc/">
  <REQUTEXT>exampleInput</REQUTEXT>
</STFC_CONNECTION>

```

El ejemplo siguiente es una llamada de RFC con un parámetro de tabla. Esta llamada de ejemplo y su grupo de RFC de prueba están disponibles como parte de todos los sistemas SAP. El nombre del parámetro de tabla es `TCPICDAT`. El tipo de línea de tabla es `ABAPTEXT` y este elemento se repite para cada fila de la tabla. Este ejemplo contiene una sola línea, denominada `LINE`. Las solicitudes con un parámetro de tabla pueden contener cualquier número de campos, donde el número es un entero positivo (*n*). 

```xml

<STFC_WRITE_TO_TCPIC xmlns="http://Microsoft.LobServices.Sap/2007/03/Rfc/">
  <RESTART_QNAME>exampleQName</RESTART_QNAME>
    <TCPICDAT>
      <ABAPTEXT xmlns="http://Microsoft.LobServices.Sap/2007/03/Rfc/">
        <LINE>exampleFieldInput1</LINE>
      <ABAPTEXT xmlns="http://Microsoft.LobServices.Sap/2007/03/Rfc/">
        <LINE>exampleFieldInput2</LINE>
      <ABAPTEXT xmlns="http://Microsoft.LobServices.Sap/2007/03/Rfc/">
        <LINE>exampleFieldInput3</LINE>
      </ABAPTEXT>
    </TCPICDAT>
</STFC_WRITE_TO_TCPIC>

```

El ejemplo siguiente es una llamada de RFC con un parámetro de tabla que tiene un campo anónimo. Un campo anónimo es cuando el campo no tiene asignado ningún nombre. Los tipos complejos se declaran en un espacio de nombres independiente, en el que la declaración establece un nuevo valor predeterminado para el nodo actual y todos sus elementos secundarios. En el ejemplo se usa el código hexadecimal `x002F` como carácter de escape para el símbolo */* , porque este símbolo está reservado en el nombre de campo de SAP.

```xml

<RFC_XML_TEST_1 xmlns="http://Microsoft.LobServices.Sap/2007/03/Rfc/">
  <IM_XML_TABLE>
    <RFC_XMLCNT xmlns="http://Microsoft.LobServices.Sap/2007/03/Rfc/">
      <_x002F_AnonymousField>exampleFieldInput</_x002F_AnonymousField>
    </RFC_XMLCNT>
  </IM_XML_TABLE>
</RFC_XML_TEST_1>

```

En el ejemplo siguiente se incluyen los prefijos de los espacios de nombres. Puede declarar todos los prefijos a la vez o puede declarar un número de ellos como atributos de un nodo. El alias del espacio de nombres de RFC `ns0` se utiliza como la raíz y los parámetros para el tipo básico.

> [!NOTE]
> Los tipos complejos se declaran en otro espacio de nombres para tipos de RFC con el alias `ns3` en lugar del espacio de nombres de RFC normal con el alias `ns0`.

```xml

<ns0:BBP_RFC_READ_TABLE xmlns:ns0="http://Microsoft.LobServices.Sap/2007/03/Rfc/" xmlns:ns3="http://Microsoft.LobServices.Sap/2007/03/Types/Rfc/">
  <ns0:DELIMITER>0</ns0:DELIMITER>
  <ns0:QUERY_TABLE>KNA1</ns0:QUERY_TABLE>
  <ns0:ROWCOUNT>250</ns0:ROWCOUNT>
  <ns0:ROWSKIPS>0</ns0:ROWSKIPS>
  <ns0:FIELDS>
    <ns3:RFC_DB_FLD>
      <ns3:FIELDNAME>KUNNR</ns3:FIELDNAME>
    </ns3:RFC_DB_FLD>
  </ns0:FIELDS>
</ns0:BBP_RFC_READ_TABLE>

```

#### <a name="xml-samples-for-bapi-requests"></a>Ejemplos XML para solicitudes de BAPI

> [!TIP]
> Si usa el diseñador de Logic Apps para editar la solicitud de BAPI, puede usar las siguientes funciones de búsqueda: 
> 
> * Seleccione un objeto en el diseñador para ver un menú desplegable de los métodos disponibles.
> * Filtre los tipos de objetos comerciales por palabra clave mediante la lista de búsqueda que la llamada API de BAPI proporciona.

> [!NOTE]
> SAP pone los objetos comerciales a disposición de los sistemas externos mediante su descripción en respuesta a RFC `RPY_BOR_TREE_INIT`, que Logic Apps emite sin ningún filtro de entrada. Logic Apps inspecciona la tabla de salida `BOR_TREE`. El campo `SHORT_TEXT` se utiliza para los nombres de objetos comerciales. Logic Apps no puede acceder a los objetos comerciales no devueltos por SAP en la tabla de salida.
> Si usa objetos comerciales personalizados, debe asegurarse de publicar y liberar estos objetos comerciales en SAP. De lo contrario, SAP no muestra los objetos comerciales personalizados en la tabla de salida `BOR_TREE`. No se puede acceder a los objetos comerciales personalizados en Logic Apps hasta que se exponen los objetos comerciales desde SAP. 

En el ejemplo siguiente se obtiene una lista de bancos mediante el método de BAPI `GETLIST`. Este ejemplo contiene el objeto comercial de un banco: `BUS1011`. 

```xml

<GETLIST xmlns="http://Microsoft.LobServices.Sap/2007/03/Bapi/BUS1011">
  <BANK_CTRY>US</BANK_CTRY>
  <MAX_ROWS>10</MAX_ROWS>
</GETLIST>

```

En el ejemplo siguiente se crea un objeto de banco mediante el método `CREATE`. En este ejemplo se usa el mismo objeto comercial que en el ejemplo anterior: `BUS1011`. Cuando use el método `CREATE` para crear un banco, asegúrese de confirmar los cambios porque este método no se confirma de forma predeterminada.

> [!TIP]
> Asegúrese de que el documento XML sigue las reglas de validación configuradas en el sistema SAP. Por ejemplo, en este documento de referencia, la clave del banco (`<BANK_KEY>`) debe ser un número de enrutamiento bancario, que en Estados Unidos también se conoce como ABA.

```xml

<CREATE xmlns="http://Microsoft.LobServices.Sap/2007/03/Bapi/BUS1011">
  <BANK_ADDRESS>
    <BANK_NAME xmlns="http://Microsoft.LobServices.Sap/2007/03/Types/Rfc">ExampleBankName</BANK_NAME>
    <REGION xmlns="http://Microsoft.LobServices.Sap/2007/03/Types/Rfc">ExampleRegionName</REGION>
    <STREET xmlns="http://Microsoft.LobServices.Sap/2007/03/Types/Rfc">ExampleStreetAddress</STREET>
    <CITY xmlns="http://Microsoft.LobServices.Sap/2007/03/Types/Rfc">Redmond</CITY>
  </BANK_ADDRESS>
  <BANK_COUNTRY>US</BANK_COUNTRY>
  <BANK_KEY>123456789</BANK_KEY>
</CREATE>

```

En el ejemplo siguiente se obtienen detalles de un banco mediante el número de enrutamiento bancario, el valor de `<BANK_KEY>`. 

```xml

<GETDETAIL xmlns="http://Microsoft.LobServices.Sap/2007/03/Bapi/BUS1011">
  <BANK_COUNTRY>US</BANK_COUNTRY>
  <BANK_KEY>123456789</BANK_KEY>
</GETDETAIL>

```

#### <a name="xml-samples-for-idoc-requests"></a>Ejemplos de XML para solicitudes de IDoc

Para generar un esquema XML de IDoc para SAP sin formato, use la aplicación **Inicio de sesión de SAP** y el código de transacción `WE-60`. Acceda a la documentación de SAP a través de la GUI y genere esquemas XML en formato XSD para sus tipos y extensiones de IDoc. Para obtener una explicación de los formatos y las cargas de SAP genéricos y sus cuadros de diálogo integrados, consulte la [documentación de SAP](https://help.sap.com/viewer/index).

En este ejemplo se declaran los espacios de nombres y el nodo raíz. El URI del código de ejemplo (`http://Microsoft.LobServices.Sap/2007/03/Idoc/3/ORDERS05//700/Send`) declara la siguiente configuración:

* `/IDoc` es la nota raíz para todos los IDoc
* `/3` es la versión de tipos de registro para definiciones de segmento comunes
* `/ORDERS05` es el tipo de IDoc
* `//` es un segmento vacío, porque no hay ninguna extensión IDoc
* `/700` es la versión de SAP
* `/Send` es la acción para enviar la información a SAP

```xml

<ns0:Send xmlns:ns0="http://Microsoft.LobServices.Sap/2007/03/Idoc/3/ORDERS05//700/Send" xmlns:ns3="http://schemas.microsoft.com/2003/10/Serialization" xmlns:ns1="http://Microsoft.LobServices.Sap/2007/03/Types/Idoc/Common/" xmlns:ns2="http://Microsoft.LobServices.Sap/2007/03/Idoc/3/ORDERS05//700">
  <ns0:idocData>

```

Puede repetir el nodo `idocData` para enviar un lote de IDoc en una sola llamada. En el ejemplo siguiente, hay un registro de control (`EDI_DC40`) y varios registros de datos.

```xml

<...>
  <ns0:idocData>
    <ns2:EDI_DC40>
      <ns1:TABNAM>EDI_DC40</ns1:TABNAM>
<...>
      <ns1:ARCKEY>Cor1908207-5</ns1:ARCKEY>
    </ns2:EDI_DC40>
    <ns2:E2EDK01005>
      <ns2:DATAHEADERCOLUMN_SEGNAM>E23DK01005</ns2:DATAHEADERCOLUMN_SEGNAM>
      <ns2:CURCY>USD</ns2:CURCY>
    </ns2:E2EDK01005>
    <ns2:E2EDK03>
<...>
  </ns0:idocData>

```

El ejemplo siguiente es un registro de control IDoc de ejemplo, que usa el prefijo `EDI_DC`. Debe actualizar los valores para que coincidan con la instalación de SAP y el tipo de IDoc. Por ejemplo, el código de cliente de IDoc puede no ser `800`. Póngase en contacto con su equipo de SAP para asegurarse de que está usando los valores correctos para la instalación de SAP.

```xml

<ns2:EDI_DC40>
  <ns:TABNAM>EDI_DC40</ns1:TABNAM>
  <ns:MANDT>800</ns1:MANDT>
  <ns:DIRECT>2</ns1:DIRECT>
  <ns:IDOCTYP>ORDERS05</ns1:IDOCTYP>
  <ns:CIMTYP></ns1:CIMTYP>
  <ns:MESTYP>ORDERS</ns1:MESTYP>
  <ns:STD>X</ns1:STD>
  <ns:STDVRS>004010</ns1:STDVRS>
  <ns:STDMES></ns1:STDMES>
  <ns:SNDPOR>SAPENI</ns1:SNDPOR>
  <ns:SNDPRT>LS</ns1:SNDPRT>
  <ns:SNDPFC>AG</ns1:SNDPFC>
  <ns:SNDPRN>ABAP1PXP1</ns1:SNDPRN>
  <ns:SNDLAD></ns1:SNDLAD>
  <ns:RCVPOR>BTSFILE</ns1:RCVPOR>
  <ns:RCVPRT>LI</ns1:RCVPRT>

```

El ejemplo siguiente es un registro de datos de ejemplo con segmentos sin formato. En este ejemplo se usa el formato de fecha de SAP. Los documentos fuertemente tipados pueden utilizar formatos de fecha XML nativos, como `2020-12-31 23:59:59`.

```xml

<ns2:E2EDK01005>
  <ns2:DATAHEADERCOLUMN_SEGNAM>E2EDK01005</ns2:DATAHEADERCOLUMN_SEGNAM>
    <ns2:CURCY>USD</ns2:CURCY>
    <ns2:BSART>OR</ns2:BSART>
    <ns2:BELNR>1908207-5</ns2:BELNR>
    <ns2:ABLAD>CC</ns2:ABLAD>
  </ns2>
  <ns2:E2EDK03>
    <ns2:DATAHEADERCOLUMN_SEGNAM>E2EDK03</ns2:DATAHEADERCOLUMN_SEGNAM>
      <ns2:IDDAT>002</ns2:IDDAT>
      <ns2:DATUM>20160611</ns2:DATUM>
  </ns2:E2EDK03>

```

El ejemplo siguiente es un registro de datos con segmentos agrupados. El registro incluye un nodo primario de grupo (`E2EDKT1002GRP`) y varios nodos secundarios, como `E2EDKT1002` y `E2EDKT2001`. 

```xml

<ns2:E2EDKT1002GRP>
  <ns2:E2EDKT1002>
    <ns2:DATAHEADERCOLUMN_SEGNAM>E2EDKT1002</ns2:DATAHEADERCOLUMN_SEGNAM>
      <NS2:TDID>ZONE</ns2:TDID>
  </ns2:E2EDKT1002>
  <ns2:E2EDKT2001>
    <ns2:DATAHEADERCOLUMN_SEGNAM>E2EDKT2001</ns2:DATAHEADERCOLUMN_SEGNAM>
      <ns2:TDLINE>CRSD</ns2:TDLINE>
  </ns2:E2EDKT2001>
</ns2:E2EDKT1002GRP>

```

El método recomendado es crear un identificador de IDoc para su uso con tRFC. Puede establecer este identificador de transacción (`tid`) mediante la [operación de envío de IDoc](/connectors/sap/#send-idoc) en la API del conector de SAP.

El ejemplo siguiente es un método alternativo para establecer el identificador de la transacción, o `tid`. En este ejemplo, el último nodo de segmento de registro de datos y el nodo de datos de IDoc están cerrados. A continuación, el GUID (`guid`) se usa como el identificador de tRFC para detectar duplicados. 

```xml

    </E2STZUM002GRP>
  </idocData>
  <guid>8820ea40-5825-4b2f-ac3c-b83adc34321c</guid>
</Send>

```

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

   1. En la sección **Puerta de enlace de datos**, en **Suscripción**, primero seleccione la suscripción de Azure para el recurso de puerta de enlace de datos que creó en el Azure Portal para la instalación de su puerta de enlace de datos. 
   
   1. En **Puerta de enlace de conexión**, seleccione su recurso de puerta de enlace de datos en Azure.

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

   Para más información sobre la acción de SAP, vea [Esquemas de mensaje para operaciones de IDoc](/biztalk/adapters-and-accelerators/adapter-sap/message-schemas-for-idoc-operations).

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

<a name="secure-network-communications"></a>

## <a name="enable-secure-network-communications"></a>Habilitación de las comunicaciones de red seguras

Antes de empezar, asegúrese de que cumple los [requisitos previos](#pre-reqs) enumerados anteriormente, que solo se aplican cuando se usa la puerta de enlace de datos y las aplicaciones lógicas se ejecutan en instancias multiinquilino de Azure:

* Asegúrese de que la puerta de enlace de datos local está instalada en un equipo que se encuentre en la misma red que el sistema SAP.

* En el caso del inicio de sesión único (SSO), la puerta de enlace de datos se ejecuta como un usuario asignado a un usuario de SAP.

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

### <a name="change-language-headers"></a>Cambio de los encabezados de idioma

Cuando se conecta a SAP desde Logic Apps, el idioma predeterminado de la conexión es el inglés. Puede establecer el idioma de la conexión mediante el [encabezado HTTP estándar `Accept-Language`](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.4) con las solicitudes entrantes.

> [!TIP]
> La mayoría de los exploradores web agregan un encabezado `Accept-Language` basado en la configuración del usuario. El explorador web aplica este encabezado cuando se crea una nueva conexión de SAP en el diseñador de Logic Apps. Si no desea crear conexiones de SAP en el idioma preferido del explorador web, actualice la configuración del explorador web para que use el idioma que prefiera o cree la conexión de SAP mediante Azure Resource Manager en lugar del diseñador de Logic Apps. 

Por ejemplo, puede enviar una solicitud con el encabezado `Accept-Language` a la aplicación lógica mediante el desencadenador de la **solicitud HTTP**. Todas las acciones de la aplicación lógica reciben el encabezado. A continuación, SAP usa los idiomas especificados en sus mensajes del sistema, como los mensajes de error de BAPI.

Los parámetros de conexión de SAP para una aplicación lógica no tienen una propiedad de idioma. Por lo tanto, si usa el encabezado `Accept-Language`, es posible que reciba el siguiente error: **Please check your account info and/or permissions and try again** (Compruebe la información de la cuenta o los permisos e inténtelo de nuevo). En este caso, compruebe los registros de errores del componente de SAP en su lugar. En realidad, el error se produce en el componente de SAP que usa el encabezado, por lo que podría obtener uno de estos mensajes de error:

* `"SAP.Middleware.Connector.RfcLogonException: Select one of the installed languages"`
* `"SAP.Middleware.Connector.RfcAbapMessageException: Select one of the installed languages"`

### <a name="confirm-transaction-explicitly"></a>Confirmación de la transacción explícitamente

Cuando se envían transacciones a SAP desde Logic Apps, este intercambio se produce en dos pasos, tal como se describe en el documento de SAP [Transactional RFC Server Programs](https://help.sap.com/doc/saphelp_nwpi71/7.1/22/042ad7488911d189490000e829fbbd/content.htm?no_cache=true) (Programas del servidor RFC transaccional). De manera predeterminada, la acción **Enviar a SAP** controla ambos pasos para la transferencia de la función y la confirmación de la transacción en una sola llamada. El conector de SAP ofrece la opción de desacoplar estos pasos. Puede enviar un IDoc y, en lugar de confirmar automáticamente la transacción, puede usar la acción explícita **Confirmar id. de transacción**.

Esta capacidad de desacoplar la confirmación del id. de transacción resulta útil cuando no quiere duplicar las transacciones en SAP, por ejemplo, en escenarios en los que se pueden producir errores debido a causas como problemas de red. Al confirmar el id. de la transacción por separado, la transacción solo se completa una vez en el sistema SAP.

Este es un ejemplo que muestra este patrón:

1. Cree una aplicación lógica en blanco y agregue un desencadenador HTTP.

1. En el conector de SAP, agregue la acción **Enviar IDOC**. Proporcione los detalles del IDoc que envía al sistema SAP.

1. Para confirmar explícitamente el ID de la transacción en un paso separado, en el campo **Confirmar TID**, seleccione **No**. Para el campo opcional **GUID del id. de transacción**, puede especificar manualmente el valor o hacer que el conector genere y devuelva automáticamente este GUID en la respuesta de la acción Enviar IDOC.

   ![Propiedades de la acción Enviar IDOC](./media/logic-apps-using-sap-connector/send-idoc-action-details.png)

1. Para confirmar explícitamente el id. de transacción, agregue la acción **Confirmar id. de transacción**, asegurándose de [evitar el envío de IDoc duplicados a SAP](#avoid-sending-duplicate-idocs). Haga clic en el cuadro **Id. de transacción** para que aparezca la lista de contenido dinámico. En esa lista, seleccione el valor de **Id. de transacción** que devuelve la acción **Enviar IDOC**.

   ![Acción Confirmar id. de transacción](./media/logic-apps-using-sap-connector/explicit-transaction-id.png)

   Después de ejecutar este paso, la transacción actual se marca como completa en ambos extremos, en el lado del conector de SAP y en el lado del sistema SAP.

#### <a name="avoid-sending-duplicate-idocs"></a>Evitar el envío de IDoc duplicados

Si experimenta un problema con IDoc duplicados que se envían a SAP desde la aplicación lógica, siga estos pasos para crear una variable de cadena que sirva como el identificador de la transacción de IDoc. La creación de este identificador de transacción ayuda a evitar transmisiones de red duplicadas cuando hay problemas, como interrupciones temporales, problemas de red o confirmaciones perdidas.

> [!NOTE]
> Los sistemas SAP olvidan un identificador de transacción después de un tiempo especificado, o 24 horas de forma predeterminada. Por consiguiente, SAP nunca no puede confirmar un identificador de transacción si se desconoce el id. o el GUID.
> Si se produce un error en la confirmación de un identificador de transacción, este error indica que hubo un problema en la comunicación con el sistema SAP antes de que SAP pudiera reconocer la confirmación.

1. En el diseñador de Logic Apps, agregue la acción **Inicializar la variable** a la aplicación lógica. 
1. En el editor de la acción **Inicializar la variable**, defina las siguientes configuraciones. A continuación, guarde los cambios.
    1. Para **Nombre**, escriba un nombre para la variable. Por ejemplo, `IDOCtransferID`.
    2. Para **Tipo**, seleccione **Cadena** como el tipo de variable.
    3. Para **Valor**, seleccione el cuadro de texto **Escriba el valor inicial** para abrir el menú de contenido dinámico. Seleccione la pestaña **Expresiones**. En la lista de funciones, escriba la función `guid()`. Después, seleccione **Aceptar** para guardar los cambios. El campo **Valor** se establece ahora en la función `guid()`, que genera un GUID.
1. Después de la acción **Inicializar la variable**, agregue la acción **Enviar de IDoc**.
1. En el editor de la acción **Enviar IDOC**, defina las siguientes configuraciones. A continuación, guarde los cambios.
    1. Para **Tipo de IDOC** seleccione el tipo de mensaje y, en **Mensaje de entrada de IDOC**, especifique el mensaje.
    1. Para **Versión de SAP**, seleccione los valores de la configuración de SAP.
    1. Para **Versión de tipos de registro**, seleccione los valores de la configuración de SAP.
    1. Para **Confirmar TID**, seleccione **No**.
    1. Seleccione **Add new parameter list** (Agregar nueva lista de parámetros) > **GUID del id. de transacción**. Seleccione el cuadro de texto para abrir el menú de contenido dinámico. En la pestaña **Variables**, seleccione el nombre de la variable que creó. Por ejemplo, `IDOCtransferID`.
1. En la barra de título de la acción **Enviar IDOC**, seleccione **...**  > **Configuración**. Para **Directiva de reintentos**, seleccione **Ninguna** > **Listo**.
1. Después de la acción **Enviar IDOC**, agregue la acción **Confirmar id. de transacción**.
1. En el editor de la acción **Confirmar id. de transacción**, defina las siguientes configuraciones. A continuación, guarde los cambios.
    1. Para **Id. de transacción**, escriba de nuevo el nombre de la variable. Por ejemplo, `IDOCtransferID`.

## <a name="known-issues-and-limitations"></a>Limitaciones y problemas conocidos

Estos son los problemas y limitaciones actualmente conocidos para el conector de SAP (que no sea ISE) administrado:

* El desencadenador SAP no admite clústeres de puerta de enlace de datos. En algunos casos de conmutación por error, el nodo de puerta de enlace de datos que se comunica con el sistema SAP puede diferir del nodo activo, lo que produce un comportamiento inesperado. En los escenarios de envío, se admiten clústeres de puerta de enlace de datos.

* El conector SAP no admite actualmente las cadenas de enrutador SAP. La puerta de enlace de datos local debe existir en la misma LAN que el sistema SAP al que quiere conectarse.

## <a name="connector-reference"></a>Referencia de conectores

Si necesita más detalles técnicos sobre este conector, como los desencadenadores, las acciones y los límites que se describen en el archivo de Swagger del conector, vea la [página de referencia del conector](/connectors/sap/).

> [!NOTE]
> En el caso de las aplicaciones lógicas de un [entorno de servicio de integración (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md), la versión con la etiqueta ISE de este conector usa en su lugar los [límites de mensajes de ISE](../logic-apps/logic-apps-limits-and-config.md#message-size-limits).

## <a name="next-steps"></a>Pasos siguientes

* [Conéctese a sistemas locales](../logic-apps/logic-apps-gateway-connection.md) desde Azure Logic Apps.
* Aprenda a validar, transformar y usar otras operaciones de mensaje con [Enterprise Integration Pack](../logic-apps/logic-apps-enterprise-integration-overview.md).
* Conozca otros [conectores de Logic Apps](../connectors/apis-list.md).
