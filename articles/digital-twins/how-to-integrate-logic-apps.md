---
title: Integración con Logic Apps
titleSuffix: Azure Digital Twins
description: Obtenga información sobre cómo conectar Logic Apps a Azure Digital Twins mediante un conector personalizado.
author: baanders
ms.author: baanders
ms.date: 8/14/2020
ms.topic: how-to
ms.service: digital-twins
ms.reviewer: baanders
ms.openlocfilehash: 20959709854f8366cc067437fe86c245fcbc3ef0
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89401068"
---
# <a name="integrate-with-logic-apps-using-a-custom-connector"></a>Integración con Logic Apps mediante un conector personalizado

[Azure Logic Apps](../logic-apps/logic-apps-overview.md) es un servicio en la nube que le ayuda a automatizar flujos de trabajo entre aplicaciones y servicios. Al conectar Logic Apps a las API de Azure Digital Twins, puede crear esos flujos automatizados en torno a Azure Digital Twins y sus datos.

Azure Digital Twins actualmente no tiene un conector certificado (precompilado) para Logic Apps. En su lugar, el proceso actual para usar Logic Apps con Azure Digital Twins es crear un [**conector personalizado de Logic Apps**](../logic-apps/custom-connector-overview.md) mediante una instancia [personalizada de Swagger de Azure Digital Twins](https://docs.microsoft.com/samples/azure-samples/digital-twins-custom-swaggers/azure-digital-twins-custom-swaggers/) que se ha modificado para funcionar con Logic Apps.

En este artículo, usará [Azure Portal](https://portal.azure.com) para **crear un conector personalizado** que se puede usar para conectar Logic Apps a una instancia de Azure Digital Twins. A continuación, **creará una aplicación lógica** que usa esta conexión para un escenario de ejemplo, en el que los eventos desencadenados por un temporizador actualizarán automáticamente un gemelo de la instancia de Azure Digital Twins. 

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, **cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)** antes de empezar.

Inicie sesión en [Azure Portal](https://portal.azure.com) con esta cuenta.

### <a name="set-up-azure-digital-twins-instance"></a>Configuración de la instancia de Azure Digital Twins

Para conectar una instancia de Azure Digital Twins a Logic Apps en este artículo, debe tener configurada la **instancia de Azure Digital Twins**. 

Si necesita configurar una instancia nueva ahora, la manera más sencilla es ejecutar un ejemplo de script de implementación automatizada. Siga las instrucciones de [*Procedimiento: Configuración de una instancia y la autenticación (con script)* ](how-to-set-up-instance-scripted.md) para configurar una nueva instancia y el registro de aplicación de Azure AD necesario. Las instrucciones también contienen pasos para comprobar que ha completado cada paso correctamente y está listo para pasar a usar la nueva instancia.

En este tutorial, necesitará los valores siguientes que obtuvo al configurar la instancia. Si tiene que volver a recopilar estos valores, use los vínculos siguientes a las secciones correspondientes del artículo de instalación para encontrarlos en [Azure Portal](https://portal.azure.com).
* El **_nombre de host_** de la instancia de Azure Digital Twins ([que se encuentra en el portal](how-to-set-up-instance-portal.md#verify-success-and-collect-important-values))
* El **_identificador de la aplicación (cliente)_** del registro de aplicación de Azure AD ([que se encuentra en el portal](how-to-set-up-instance-portal.md#collect-important-values))
* El **_identificador de directorio (inquilino)_**  del registro de aplicación de Azure AD ([que se encuentra en el portal](how-to-set-up-instance-portal.md#collect-important-values))

### <a name="get-app-registration-client-secret"></a>Obtención del secreto de cliente del registro de la aplicación

También tendrá que crear un **_secreto de cliente_** para el registro de la aplicación de Azure AD. Para ello, vaya a la página [Registros de aplicaciones](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade) de Azure Portal (puede usar este vínculo o buscarlo en la barra de búsqueda del portal). Seleccione el registro en la lista para abrir sus detalles. 

Presione *Certificados y secretos* en el menú del registro y seleccione *+ Nuevo secreto de cliente*.

:::image type="content" source="media/how-to-integrate-logic-apps/client-secret.png" alt-text="Vista del portal de un registro de aplicaciones de Azure AD. Hay un resaltado alrededor de "Certificados y secretos" en el menú de recursos y un resaltado en la página alrededor de "Nuevo secreto de cliente".":::

Escriba los valores que desee para Descripción y Expiración y presione *Agregar*.
El secreto se agregará a la lista de secretos de cliente en la página *Certificados y secretos*. Tome nota de su valor para usarlo más adelante (también puede copiarlo en el Portapapeles con el icono de copia).

### <a name="add-a-digital-twin"></a>Adición de un gemelo digital

En este artículo se usa Logic Apps para actualizar un gemelo de la instancia de Azure Digital Twins. Para continuar, debe agregar al menos un gemelo a la instancia. 

Puede agregar gemelos mediante las [API de Digital Twins](how-to-use-apis-sdks.md), el [SDK de .NET (C#)](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/digitaltwins/Azure.DigitalTwins.Core) o la [CLI de Azure Digital Twins](how-to-use-cli.md). Para obtener pasos detallados sobre cómo crear gemelos con estos métodos, consulte [*Procedimiento: Administración de Digital Twins*](how-to-manage-twin.md).

Necesitará el **_Identificador de gemelo_** de un gemelo de la instancia que ha creado.

## <a name="create-custom-logic-apps-connector"></a>Creación de un conector personalizado de Logic Apps

En este paso va a crear un [conector personalizado de Logic Apps](../logic-apps/custom-connector-overview.md) para las API de Azure Digital Twins. Después de hacerlo, podrá enlazar Azure Digital Twins al crear una aplicación lógica en la sección siguiente.

Vaya a la página [Conector personalizado de Logic Apps](https://portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Web%2FcustomApis) de Azure Portal (puede usar este vínculo o buscarlo en la barra de búsqueda del portal). Presione *+ Agregar*.

:::image type="content" source="media/how-to-integrate-logic-apps/logic-apps-custom-connector.png" alt-text="Página "Conector personalizado de Logic Apps" de Azure Portal. Hay un resaltado alrededor del botón "Agregar".":::

En la página *Crear conector personalizado de Logic Apps* que se muestra a continuación, seleccione la suscripción y el grupo de recursos y un nombre y una ubicación de implementación para el nuevo conector. Presione *Revisar y crear*. Esto le llevará a la pestaña *Revisar y crear*, donde podrá presionar *Crear* en la parte inferior para crear el recurso.

:::image type="content" source="media/how-to-integrate-logic-apps/create-logic-apps-custom-connector.png" alt-text="Pestaña "Revisar y crear" de la página "Crear conector personalizado de Logic Apps" de Azure Portal. Hay un resaltado alrededor del botón "Crear".":::

Se le dirigirá a la página de implementación del conector. Cuando haya terminado la implementación, pulse el botón *Ir al recurso* para ver los detalles del conector en el portal.

### <a name="configure-connector-for-azure-digital-twins"></a>Configuración del conector para Azure Digital Twins

A continuación, va a configurar el conector que ha creado para la conexión con Azure Digital Twins.

En primer lugar, descargue un archivo de Swagger de Azure Digital Twins personalizado que se ha modificado para que funcione con Logic Apps. Descargue el ejemplo **Swaggers personalizados de Azure Digital Twins** desde [este vínculo](https://docs.microsoft.com/samples/azure-samples/digital-twins-custom-swaggers/azure-digital-twins-custom-swaggers/) mediante el botón *Descargar archivo ZIP*. Vaya a la carpeta descargada *Azure_Digital_Twins_Custom_Swaggers.zip* y descomprímala. La instancia de Swagger personalizada para este tutorial se encuentra en *Azure_Digital_Twins_Custom_Swaggers\LogicApps\preview\2020-05-31-preview\digitaltwins.json*.

A continuación, diríjase a la página de información general del conector en [Azure Portal](https://portal.azure.com) y pulse *Editar*.

:::image type="content" source="media/how-to-integrate-logic-apps/edit-connector.png" alt-text="Página de información general del conector creado en el paso anterior. Hay un resaltado alrededor del botón "Editar".":::

En la página *Editar conector personalizado de Logic Apps* que aparece a continuación, configure esta información:
* **Conectores personalizados**
    - Punto de conexión de API: REST (dejar el valor predeterminado)
    - Modo de importación: archivo OpenAPI (dejar el valor predeterminado)
    - Archivo: archivo de Swagger personalizado que descargó anteriormente. Pulse *Importar*, busque el archivo en el equipo (*Azure_Digital_Twins_Custom_Swaggers\LogicApps\preview\2020-05-31-preview\digitaltwins.json*) y seleccione *Abrir*.
* **Información general**
    - Icono, color de fondo del icono y descripción: rellene los valores que desee.
    - Esquema: HTTPS (dejar el valor predeterminado)
    - Host: *nombre de host* de la instancia de Azure Digital Twins.
    - URL base: / (dejar el valor predeterminado)

A continuación, presione el botón *Seguridad* situado en la parte inferior de la ventana para continuar con el siguiente paso de configuración.

:::image type="content" source="media/how-to-integrate-logic-apps/configure-next.png" alt-text="Captura de pantalla de la parte inferior de la página "Editar conector personalizado de Logic Apps". Hay un resaltado alrededor del botón para continuar con la sección Seguridad.":::

En el paso Seguridad, presione *Editar* y configure esta información:
* **Tipo de autenticación**: OAuth 2.0
* **OAuth 2.0**:
    - Proveedor de identidades: Azure Active Directory
    - Identificador de cliente: *identificador de la aplicación (cliente)* del registro de la aplicación de Azure AD
    - Secreto del cliente: *secreto de cliente* creado en la sección [*Requisitos previos*](#prerequisites) para el registro de la aplicación de Azure AD
    - Dirección URL de inicio de sesión: https://login.windows.net (dejar el valor predeterminado)
    - Identificador de inquilino: *identificador de directorio (inquilino)* del registro de aplicación de Azure AD
    - Dirección URL del recurso: 0b07f429-9f4b-4714-9392-cc5e8e80c8b0
    - Ámbito: Directory.AccessAsUser.All
    - URL de redireccionamiento: (dejar el valor predeterminado por ahora)

Observe que el campo dirección URL de redireccionamiento indica *Guardar el conector personalizado para generar la URL de redireccionamiento*. Para hacerlo ahora, presione *Actualizar conector* en la parte superior del panel para confirmar la configuración del conector.

:::image type="content" source="media/how-to-integrate-logic-apps/update-connector.png" alt-text="Captura de pantalla de la parte superior de la página "Editar conector personalizado de Logic Apps". Hay un resaltado alrededor del botón "Actualizar conector".":::

<!-- Success message? didn't see one -->

Vuelva al campo dirección URL de redireccionamiento y copie el valor que se ha generado. Lo usará en el paso siguiente.

:::image type="content" source="media/how-to-integrate-logic-apps/copy-redirect-url.png" alt-text="El campo dirección URL de redireccionamiento de la página "Editar conector personalizado de Logic Apps" tiene ahora el valor "https://logic-apis-westus2.consent.azure-apim.net/redirect". El botón para copiar el valor está resaltado.":::

Esta es toda la información necesaria para crear el conector (no es necesario continuar del paso de seguridad al paso de definición). Puede cerrar el panel *Editar conector personalizado de Logic Apps*.

>[!NOTE]
>De nuevo en la página de información general del conector donde presionó *Editar* originalmente, tenga en cuenta que al presionar *Editar* de nuevo se reiniciará todo el proceso de especificación de las opciones de configuración. No se rellenarán los valores desde la última vez que entró, por lo que si desea guardar una configuración actualizada con cualquier valor cambiado, debe volver a escribir todos los demás valores para evitar que los valores predeterminados los sobrescriban.

### <a name="grant-connector-permissions-in-the-azure-ad-app"></a>Concesión de permisos al conector en la aplicación de Azure AD

A continuación, use el valor de la *dirección URL de redireccionamiento* del conector personalizado que copió en el último paso para conceder los permisos al conector en el registro de la aplicación de Azure AD.

Vaya a la página [Registros de aplicaciones](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade) de Azure Portal y seleccione el registro en la lista. 

Seleccione *Autenticación* en el menú del registro y agregue un identificador URI.

:::image type="content" source="media/how-to-integrate-logic-apps/add-uri.png" alt-text="Página de autenticación del registro de la aplicación en Azure Portal. "Autenticación" en el menú está resaltado y, en la página, el botón "Agregar identificador URI" está resaltado."::: 

Escriba la *dirección URL de redireccionamiento* del conector personalizado en el campo nuevo y presione el icono *Guardar*.

:::image type="content" source="media/how-to-integrate-logic-apps/save-uri.png" alt-text="Página de autenticación del registro de la aplicación en Azure Portal. La nueva dirección URL de redireccionamiento y el botón "Guardar" de la página están resaltados.":::

Ya ha terminado de configurar un conector personalizado que puede acceder a las API de Azure Digital Twins. 

## <a name="create-logic-app"></a>Creación de la aplicación lógica

A continuación, va a crear una aplicación lógica que usará el nuevo conector para automatizar las actualizaciones de Azure Digital Twins.

Vaya a la página [Logic Apps](https://portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Logic%2Fworkflows) de Azure Portal (puede usar este vínculo o buscarlo en la barra de búsqueda del portal). Presione *Crear aplicación lógica*.

:::image type="content" source="media/how-to-integrate-logic-apps/create-logic-app.png" alt-text="Página "Logic Apps" en Azure Portal. Hay un resaltado alrededor del botón "Crear aplicación lógica".":::

En la página *Aplicación lógica* que se muestra a continuación, seleccione la suscripción y el grupo de recursos y un nombre y una ubicación de implementación para la nueva aplicación lógica. Presione *Revisar y crear*. Esto le llevará a la pestaña *Revisar y crear*, donde podrá presionar *Crear* en la parte inferior para crear el recurso.

Se le dirigirá a la página de implementación de la aplicación lógica. Cuando haya terminado la implementación, pulse el botón *Ir al recurso* para continuar con el *Diseñador de aplicaciones lógicas*, donde rellenará la lógica del flujo de trabajo.

### <a name="design-workflow"></a>Diseño del flujo de trabajo

En *Diseñador de aplicaciones lógicas*, en *Empezar con un desencadenador común*, seleccione _**Repetición**_.

:::image type="content" source="media/how-to-integrate-logic-apps/logic-apps-designer-recurrence.png" alt-text="Página "Diseñador de aplicaciones lógicas" en Azure Portal. Hay un resaltado alrededor del desencadenador común "Repetición".":::

En la página *Diseñador de aplicaciones lógicas* que aparece a continuación, cambie la frecuencia de **Repetición** a *Segundo*, de modo que el evento se desencadene cada 3 segundos. Esto hará que sea más fácil ver los resultados más adelante sin tener que esperar mucho tiempo.

Presione *+ Nuevo paso*.

Esto abrirá el cuadro *Elegir una acción*. Cambie a la pestaña *Personalizada*. Debería ver el conector personalizado anterior en el cuadro superior.

:::image type="content" source="media/how-to-integrate-logic-apps/custom-action.png" alt-text="Crear un flujo en el Diseñador de aplicaciones lógicas en Azure Portal. En el cuadro "Elegir una acción",está seleccionada la pestaña "Personalizada". El conector personalizado anterior del usuario se muestra en el cuadro, con un resaltado alrededor.":::

Selecciónelo para mostrar la lista de las API contenidas en ese conector. Use la barra de búsqueda o desplácese por la lista para seleccionar **DigitalTwins_Add**. (Esta es la API que se usa en este artículo, pero también puede seleccionar cualquier otra API como una opción válida para una conexión de Logic Apps).

Es posible que se le pida que inicie sesión con sus credenciales de Azure para conectarse al conector. Si aparece el cuadro de diálogo *Permisos solicitados*, siga las indicaciones para conceder consentimiento para la aplicación y acepte.

En el nuevo cuadro *DigitalTwinsAdd*, rellene los campos como se indica a continuación:
* id: rellene el *Identificador de gemelo* del gemelo digital de la instancia que desea que la aplicación lógica actualice.
* Elemento 1: en este campo especificará el cuerpo que requiere la solicitud de API seleccionada. En el caso de *DigitalTwinsUpdate*, el cuerpo tiene el formato de código de revisión JSON. Para más información sobre cómo estructurar una revisión JSON para actualizar el gemelo, consulte la sección [Actualización de un gemelo digital](how-to-manage-twin.md#update-a-digital-twin) de *Procedimiento: Administración de Digital Twins*.
* api-version: En la versión preliminar pública actual, este valor es *2020-05-31-preview*.

Presione *Guardar* en el Diseñador de aplicaciones lógicas.

:::image type="content" source="media/how-to-integrate-logic-apps/save-logic-app.png" alt-text="Vista finalizada de la aplicación en el conector de Logic apps. El cuadro DigitalTwinsAdd se rellena con los valores descritos anteriormente, incluido un cuerpo de revisión JSON de ejemplo. El botón "Guardar" de la ventana está resaltado.":::

## <a name="query-twin-to-see-the-update"></a>Consulta del gemelo para ver la actualización

Ahora que se ha creado la aplicación lógica, el evento de actualización del gemelo que definió en el Diseñador de aplicaciones lógicas debe aparecer con una periodicidad de cada tres segundos. Esto significa que, después de tres segundos, debería poder consultar el gemelo y ver reflejados los nuevos valores de revisión.

Puede consultar el gemelo mediante el método que elija (como una [aplicación cliente personalizada](tutorial-command-line-app.md), la [aplicación de ejemplo Explorador de Azure Digital Twins](https://docs.microsoft.com/samples/azure-samples/digital-twins-explorer/digital-twins-explorer/), los [SDK y las API](how-to-use-apis-sdks.md) o la [CLI](how-to-use-cli.md)). 

Para más información sobre cómo consultar la instancia de Azure Digital Twins, consulte [*Procedimiento: Consulta del grafo de gemelos*](how-to-query-graph.md).

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha creado una aplicación lógica que actualiza periódicamente un gemelo en la instancia de Azure Digital Twins con una revisión que ha proporcionado. Puede probar a seleccionar otras API en el conector personalizado para crear aplicaciones lógicas para varias acciones en la instancia.

Para más información sobre las operaciones de API disponibles y los detalles que requieren, consulte [*Procedimiento: las API y los SDK de Azure Digital Twins*](how-to-use-apis-sdks.md).