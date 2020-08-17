---
title: Conexión a un servidor FTP
description: Automatice tareas y flujos de trabajo que crean, supervisan y administran archivos en un servidor FTP mediante Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: conceptual
ms.date: 12/15/2019
tags: connectors
ms.openlocfilehash: f4cad2b658547d56d00efdd5e1496110f8e4a5e6
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87284020"
---
# <a name="create-monitor-and-manage-ftp-files-by-using-azure-logic-apps"></a>Creación, supervisión y administración de archivos FTP mediante Azure Logic Apps

Con Azure Logic Apps y el conector FTP, puede crear tareas y flujos de trabajo automatizados que crea, supervisan, envían y reciben archivos mediante su cuenta en un servidor FTP, junto con otras acciones, por ejemplo:

* Supervisar cuándo se agregan o cambian archivos
* Obtener, crear, copiar, actualizar, enumerar y eliminar archivos
* Obtener contenido de archivos y metadatos
* Extraer archivos en carpetas

Puede usar desencadenadores que obtengan respuestas de su servidor FTP y permitan que la salida esté disponible para otras acciones. Puede usar las acciones de ejecución en las aplicaciones lógicas para administrar los archivos del servidor FTP. También puede hacer que otras acciones usen la salida de las acciones de FTP. Por ejemplo, si recupera archivos del servidor FTP habitualmente, puede enviar un correo electrónico sobre esos archivos y su contenido mediante el conector Office 365 Outlook o el conector Outlook.com. Si no está familiarizado con las aplicaciones lógicas, vea [¿Qué es Azure Logic Apps?](../logic-apps/logic-apps-overview.md)

## <a name="limitations"></a>Limitaciones

* El conector de FTP solo admite FTP por TLS/SSL (FTPS) explícito y no es compatible con FTPS implícito.

* De forma predeterminada, las acciones de FTP pueden leer o escribir archivos con un tamaño de *50 MB o menos*. Para procesar archivos mayores de 50 MB, las acciones de FTP admiten la [fragmentación de mensajes](../logic-apps/logic-apps-handle-large-messages.md). La acción **Obtener contenido de archivo** usa implícitamente la fragmentación.

* Los desencadenadores de FTP no admiten la fragmentación. Cuando se solicita el contenido del archivo, los desencadenadores seleccionan solo los archivos que tienen un tamaño de 50 MB o menos. Para obtener archivos de más de 50 MB, siga este patrón:

  * Use un desencadenador de FTP que devuelva las propiedades de archivo, como **Cuando se agrega o modifica un archivo (solo propiedades)** .

  * Siga el desencadenador con la acción **Obtener contenido de archivo** de FTP, que lee el archivo completo y usa implícitamente la fragmentación.

* Si tiene un servidor FTP local, considere la posibilidad de crear un [entorno de servicio de integración (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md) o usar [conexiones híbridas de Azure App Service](../app-service/app-service-hybrid-connections.md); ambas opciones permiten obtener acceso a orígenes de datos locales sin usar una puerta de enlace de datos local.

## <a name="how-ftp-triggers-work"></a>Funcionamiento de desencadenadores de FTP

Los desencadenadores de FTP funcionan al sondear el sistema de archivos FTP y buscar archivos que se hayan modificado desde el último sondeo. Algunas herramientas le permiten conservar la marca de tiempo cuando cambian los archivos. En estos casos, tiene que deshabilitar esta característica para que el desencadenador funcione. Estas son algunas opciones de configuración habituales:

| Cliente SFTP | Acción |
|-------------|--------|
| WinSCP | Vaya a **Options** > **Preferences** > **Transfer** > **Edit** > **Preserve timestamp** > **Disable** (Opciones > Preferencias > Transferir > Editar > Conservar marca de tiempo >Deshabilitar). |
| FileZilla | Vaya a **Transfer** > **Preserve timestamps of transferred files** > **Disable** (Transferir > Conservar marca de tiempo de archivos transferidos > Deshabilitar). |
|||

Cuando un desencadenador encuentra un nuevo archivo, el desencadenador comprueba que el nuevo archivo se ha escrito por completo y no parcialmente. Por ejemplo, un archivo podría tener los cambios en curso cuando el desencadenador comprueba el servidor de archivos. Para evitar devolver un archivo parcialmente escrito, el desencadenador anota la marca de tiempo del archivo que tiene cambios recientes, pero no devuelve inmediatamente dicho archivo. El desencadenador devuelve el archivo solo al volver a sondear el servidor. En ocasiones, este comportamiento puede provocar un retraso de hasta dos veces el intervalo de sondeo del desencadenador.

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción a Azure. Si no tiene una suscripción de Azure, [regístrese para obtener una cuenta gratuita de Azure](https://azure.microsoft.com/free/).

* Las credenciales de la cuenta y la dirección del servidor host FTP

  El conector de FTP requiere que el servidor FTP sea accesible desde Internet y que esté configurado para poder funcionar en modo *pasivo*. Las credenciales permiten a la aplicación lógica crear una conexión a la cuenta de FTP y acceder a ella.

* Conocimientos básicos acerca de [cómo crear aplicaciones lógicas](../logic-apps/quickstart-create-first-logic-app-workflow.md)

* La aplicación lógica desde donde quiere acceder a la cuenta de FTP. Para comenzar con un desencadenador de FTP, [cree una aplicación lógica en blanco](../logic-apps/quickstart-create-first-logic-app-workflow.md). Para usar una acción de FTP, inicie la aplicación lógica con otro desencadenador, por ejemplo, el desencadenador **Recurrence**.

## <a name="connect-to-ftp"></a>Conexión a un FTP

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)]

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y abra la aplicación lógica en el Diseñador de aplicación lógica.

1. Para las aplicaciones lógicas en blanco, en el cuadro de búsqueda, escriba `ftp` como filtro. En la lista **Desencadenadores**, seleccione el desencadenador que quiera usar.

   O bien

   Para las aplicaciones lógicas existentes, en el último paso donde quiere agregar una acción, elija **Nuevo paso** y seleccione **Agregar una acción**. En el cuadro de búsqueda, escriba `ftp` como filtro. En la lista **Acciones**, seleccione la que quiera usar.

   Para agregar una acción entre un paso y otro, mueva el puntero sobre la flecha entre ellos. Seleccione el signo más ( **+** ) que aparece y, luego, seleccione **Agregar una acción**.

1. Proporcione su información de conexión y seleccione **Crear**.

1. Proporcione la información necesaria para el desencadenador o la acción seleccionados y continúe con la compilación del flujo de trabajo de la aplicación lógica.

## <a name="examples"></a>Ejemplos

<a name="file-added-modified"></a>

### <a name="add-ftp-trigger"></a>Agregar desencadenador FTP

El desencadenador **Cuando se agrega o modifica un archivo (solo propiedades)** inicia un flujo de trabajo de aplicación lógica cuando el desencadenador detecta que se ha agregado o cambiado un archivo en un servidor FTP. Por ejemplo, puede agregar una condición que compruebe el contenido del archivo y decida si obtener ese contenido, en función de si cumple una condición especificada. Por último, puede agregar una acción que obtenga el contenido del archivo y lo coloque en una carpeta diferente en el servidor FTP.

Por ejemplo, puede usar este desencadenador para supervisar los nuevos archivos de una carpeta FTP que describen los pedidos de los clientes. Después, puede usar una acción de FTP como **Obtener metadatos del archivo** para obtener las propiedades de ese nuevo archivo y, después, usar **Obtener contenido del archivo** para obtener el contenido de ese archivo para su posterior procesamiento y almacenarlo en una base de datos de pedidos.

Este es un ejemplo que muestra cómo usar el desencadenador **Cuando se agrega o modifica un archivo (solo propiedades)** .

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y abra la aplicación lógica en el diseñador de aplicaciones lógicas, si aún no lo ha hecho.

1. Para las aplicaciones lógicas en blanco, en el cuadro de búsqueda, escriba `ftp` como filtro. En la lista de desencadenadores, seleccione este desencadenador: **Cuando se agrega o modifica un archivo (solo propiedades)**

   ![Buscar y seleccionar el desencadenador de FTP](./media/connectors-create-api-ftp/select-ftp-trigger-logic-app.png)

1. Proporcione la información necesaria para la conexión y seleccione **Crear**.

   De forma predeterminada, este conector transfiere los archivos en formato de texto. Para transferir archivos en formato binario, por ejemplo, cuando se use codificación, seleccione **Transporte binario**.

   ![Creación de una conexión a un servidor FTP](./media/connectors-create-api-ftp/create-ftp-connection-trigger.png)

1. En el cuadro **Carpeta**, seleccione el icono de la carpeta para que aparezca una lista. Para buscar la carpeta donde quiere supervisar los archivos nuevos o modificados, seleccione la flecha de ángulo derecho ( **>** ), vaya a esa carpeta y, luego, seleccione la carpeta.

   ![Busque y seleccione la carpeta para supervisar](./media/connectors-create-api-ftp/select-folder-ftp-trigger.png)

   La carpeta seleccionada aparece en el cuadro **Carpeta**.

   ![La carpeta seleccionada aparece en el cuadro "Carpeta".](./media/connectors-create-api-ftp/selected-folder-ftp-trigger.png)

1. Guarde la aplicación lógica. En la barra de herramientas del diseñador, seleccione **Save** (Guardar).

Ahora que la aplicación lógica tiene un desencadenador, agregue las acciones que quiera ejecutar cuando la aplicación lógica encuentre un archivo nuevo o modificado. En este ejemplo, puede agregar una acción de FTP que obtiene el contenido nuevo o actualizado.

<a name="get-content"></a>

### <a name="add-ftp-action"></a>Agregar acción de FTP

La acción **Obtener metadatos del archivo** obtiene las propiedades de un archivo que está en el servidor FTP y la acción **Obtener contenido del archivo** obtiene el contenido del archivo según la información sobre ese archivo en el servidor FTP. Por ejemplo, puede agregar el desencadenador del ejemplo anterior y una acción que obtiene el contenido del archivo después de que se agregue o modifique el archivo.

1. En el desencadenador o en cualquiera de las otras acciones, seleccione **Nuevo paso**.

1. En el cuadro de búsqueda, escriba `ftp` como filtro. En la lista de acciones, seleccione esta acción: **Obtener metadatos de archivo**

   ![Seleccione la acción "Obtener metadatos del archivo".](./media/connectors-create-api-ftp/select-get-file-metadata-ftp-action.png)

1. Si ya tiene una conexión con el servidor FTP y la cuenta, vaya al paso siguiente. De lo contrario, proporcione la información necesaria para la conexión y luego seleccione **Crear**.

   ![Creación de una conexión al servidor FTP](./media/connectors-create-api-ftp/create-ftp-connection-action.png)

1. Después de mostrarse la acción **Obtener metadatos del archivo**, haga clic dentro del cuadro **Archivo** para que aparezca la lista de contenido dinámico. Ahora puede seleccionar propiedades para las salidas de los pasos anteriores. En la lista de contenido dinámico, en **Obtener metadatos del archivo**, seleccione la propiedad **List of Files Id** (Id. de lista de archivos), que hace referencia a la colección en la que se agregó o actualizó el archivo.

   ![Buscar y seleccionar la propiedad List of Files Id (Id. de lista de archivos)](./media/connectors-create-api-ftp/select-list-of-files-id-output.png)

   La propiedad **List of Files Id** (Id. de lista de archivos) aparece ahora en el cuadro **Archivo**.

   ![Propiedad List of Files Id (Id. de lista de archivos) seleccionada](./media/connectors-create-api-ftp/selected-list-file-ids-ftp-action.png)

1. Ahora agregue esta acción de FTP: **Obtener contenido de archivo**

   ![Busque y seleccione la acción "Obtener contenido del archivo".](./media/connectors-create-api-ftp/select-get-file-content-ftp-action.png)

1. Después de mostrarse la acción **Obtener contenido del archivo**, haga clic dentro del cuadro **Archivo** para que aparezca la lista de contenido dinámico. Ahora puede seleccionar propiedades para las salidas de los pasos anteriores. En la lista de contenido dinámico, en **Obtener metadatos del archivo**, seleccione la propiedad **Id.** , que hace referencia al archivo que se agregó o actualizó.

   ![Buscar y seleccionar la propiedad "Id."](./media/connectors-create-api-ftp/get-file-content-id-output.png)

   La propiedad **Id.** aparece ahora en el cuadro **Archivo**.

   ![Propiedad "Id." seleccionada](./media/connectors-create-api-ftp/selected-get-file-content-id-ftp-action.png)

1. Guarde la aplicación lógica.

## <a name="test-your-logic-app"></a>Comprobación de la aplicación lógica

Para comprobar que el flujo de trabajo devuelve el contenido que espera, agregue otra acción que envíe el contenido del archivo cargado o actualizado.

1. En la acción **Obtener contenido del archivo**, agregue una acción que pueda enviarle el contenido del archivo. En este ejemplo se agrega la acción **Enviar un correo electrónico** para Office 365 Outlook.

   ![Agregar una acción para enviar correo electrónico](./media/connectors-create-api-ftp/select-send-email-action.png)

1. Después de mostrarse la acción, proporcione la información e incluya las propiedades que quiere probar. Por ejemplo, incluya la propiedad **Archivo de contenido**, que aparece en la lista de contenido dinámico después de seleccionar **Ver más** en la sección **Obtener contenido del archivo**.

   ![Proporción de información sobre la acción del correo electrónico](./media/connectors-create-api-ftp/selected-send-email-action.png)

1. Guarde la aplicación lógica. Para ejecutar y desencadenar la aplicación lógica, seleccione **Ejecutar** en la barra de herramientas y, después, agregue un archivo a la carpeta FTP que la aplicación lógica supervisa ahora.

## <a name="connector-reference"></a>Referencia de conectores

Si necesita más detalles técnicos sobre este conector, como los desencadenadores, las acciones y los límites que se describen en el archivo de Swagger del conector, vea la [página de referencia del conector](/connectors/ftpconnector/).

> [!NOTE]
> En el caso de las aplicaciones lógicas de un [entorno de servicio de integración (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md), la versión con la etiqueta ISE de este conector usa en su lugar los [límites de mensajes de ISE](../logic-apps/logic-apps-limits-and-config.md#message-size-limits).

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre otros [conectores de Logic Apps](../connectors/apis-list.md)

