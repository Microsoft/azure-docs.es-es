---
title: Exportación de una API hospedada en Azure a PowerApps y Microsoft Flow
description: Introducción a la exposición de una API que se hospeda en App Service a PowerApps y Microsoft Flow
ms.topic: conceptual
ms.date: 04/28/2020
ms.reviewer: sunayv
ms.openlocfilehash: 7d968c62afbfc92952f747e1e7627c98fe07436d
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87015092"
---
# <a name="exporting-an-azure-hosted-api-to-powerapps-and-microsoft-flow"></a>Exportación de una API hospedada en Azure a PowerApps y Microsoft Flow

[PowerApps](https://powerapps.microsoft.com/guided-learning/learning-introducing-powerapps/) es un servicio para generar y utilizar aplicaciones empresariales personalizadas que se conectan a los datos y funcionan en otras plataformas. [Microsoft Flow](/learn/modules/get-started-with-flow/index) facilita la automatización de los flujos de trabajo y los procesos empresariales en sus aplicaciones y servicios favoritos. Tanto PowerApps como Microsoft Flow vienen con una variedad de conectores integrados a los orígenes de datos, como Office 365, 365 Dynamics, Salesforce y mucho más. En algunos casos, los generadores de aplicaciones y flujos también desean conectarse a los orígenes de datos y las API generadas por su organización.

De igual forma, los desarrolladores que desean exponer su API de manera más amplia dentro de la organización pueden poner sus API a disposición de los generadores de aplicaciones y flujos. En este artículo se muestra cómo exportar una API generada con [Azure Functions](../azure-functions/functions-overview.md) o [Azure App Service](../app-service/overview.md). La API exportada se convierte en un *conector personalizado*, que se utiliza en PowerApps y Microsoft Flow igual que un conector integrado.

> [!IMPORTANT]
> La funcionalidad de la definición de la API que se muestra en este artículo solo se admite para la [versión 1.x del entorno de ejecución de Azure Functions](functions-versions.md#creating-1x-apps) y de las aplicaciones de App Services. La versión 2.x de Funciones se integra con API Management para crear y mantener las definiciones de OpenAPI. Para más información, consulte [Creación de una definición de OpenAPI para una función con Azure API Management](functions-openapi-definition.md). 

## <a name="create-and-export-an-api-definition"></a>Creación y exportación de una definición de API
Antes de exportar una API, debe describir la API con una definición de OpenAPI (anteriormente conocido como archivo [Swagger](https://swagger.io/)). Esta definición contiene información sobre las operaciones que están disponibles en una API y cómo se deben estructurar los datos de la solicitud y la respuesta para la API. PowerApps y Microsoft Flow pueden crear conectores personalizados para cualquier definición de Open API 2.0. Azure Functions y Azure App Service tienen compatibilidad integrada para crear, hospedar y administrar definiciones de OpenAPI. Para más información, consulte [Hospedaje de una API RESTful con CORS en Azure App Service](../app-service/app-service-web-tutorial-rest-api.md).

> [!NOTE]
> También puede generar conectores personalizados en la IU de PowerApps y Microsoft Flow sin usar una definición de OpenAPI. Para obtener más información, consulte [Registrar y usar conectores personalizados en PowerApps](https://powerapps.microsoft.com/tutorials/register-custom-api/) y [Registro y uso de conectores personalizados en Microsoft Flow](/power-automate/developer/register-custom-api).

Para exportar la definición de API, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com), vaya a la aplicación de funciones o a una aplicación de App Service.

    En el menú de la izquierda, en **API**, seleccione **Definición de la API**.

    :::image type="content" source="media/app-service-export-api-to-powerapps-and-flow/api-definition-function.png" alt-text="Definición de la API de Azure Functions":::

2. El botón **Exportar a PowerApps + Microsoft Flow** debe estar disponible (si no es así, primero debe crear una definición de OpenAPI). Seleccione este botón para iniciar la exportación.

    ![Botón Exportar a PowerApps + Microsoft Flow](media/app-service-export-api-to-powerapps-and-flow/export-apps-flow.png)

3. Seleccione el **modo de exportación**:

    **Rápida** permite crear el conector personalizado desde dentro Azure Portal. Esto requiere que haya iniciado sesión en PowerApps o Microsoft Flow y que tenga permiso para crear conectores en el entorno de destino. Este es el enfoque recomendado si se puede cumplir dicho requisito. Si usa este modo, siga las instrucciones de [Uso de la exportación rápida](#express) especificadas más abajo.

    **Manual** permite exportar la definición de la API, que puede importarse luego mediante los portales de PowerApps o Microsoft Flow. Este es el enfoque recomendado si el usuario de Azure y el usuario con permiso para crear conectores son personas diferentes o si el conector debe crearse en otro inquilino de Azure. Si usa este modo, siga las instrucciones de [Uso de la exportación manual](#manual) especificadas a continuación.

    ![Modo de exportación](media/app-service-export-api-to-powerapps-and-flow/export-mode.png)

> [!NOTE]
> El conector personalizado usa una *copia* de la definición de API, por lo que PowerApps y Microsoft Flow no sabrán de inmediato si realiza cambios en la aplicación y su definición de API. Si efectivamente realiza cambios, repita los pasos de exportación para la nueva versión.

<a name="express"></a>
## <a name="use-express-export"></a>Uso de la exportación rápida

Para completar la exportación en la modalidad **Rápida**, siga estos pasos:

1. Asegúrese de que ha iniciado sesión en el inquilino de PowerApps o Microsoft Flow al que desea exportar. 

2. Use la configuración que se especifica en la tabla.

    |Configuración|Descripción|
    |--------|------------|
    |**Entorno**|Seleccione el entorno en que se debe guardar el conector personalizado. Para obtener más información, consulte [Información general sobre los entornos](https://powerapps.microsoft.com/tutorials/environments-overview/).|
    |**Nombre de la API personalizada**|Escriba un nombre, el cual verán PowerApps y Microsoft Flow en su lista de conectores.|
    |**Preparación de la configuración de seguridad**|Si es necesario, proporcione los detalles de configuración de seguridad requeridos para conceder a los usuarios acceso a la API. En este ejemplo se muestra una clave de API. Para obtener más información, vea el apartado [Especificar el tipo de autenticación](#auth) más abajo.|
 
    ![Exportación rápida a PowerApps y Microsoft Flow](media/app-service-export-api-to-powerapps-and-flow/export-express.png)

3. Haga clic en **OK**. El conector personalizado se ha compilado y agregado al entorno especificado.

<a name="manual"></a>
## <a name="use-manual-export"></a>Uso de la exportación manual

Para completar la exportación en el modo **Manual**, siga estos pasos:

1. Haga clic en **Descargar** y guarde el archivo, o haga clic en el botón Copiar y guarde la dirección URL. Usará el archivo de descarga o la dirección URL durante la importación.
 
    ![Exportación manual a PowerApps y Microsoft Flow](media/app-service-export-api-to-powerapps-and-flow/export-manual.png)
 
2. Si la definición de la API incluye cualquier definición de seguridad, estas definiciones se indican en el paso 2. Durante la importación, PowerApps y Microsoft Flow detectan estas definiciones y solicitan información de seguridad. Recopile las credenciales relacionadas con cada definición para su uso en la sección siguiente. Para obtener más información, vea el apartado [Especificar el tipo de autenticación](#auth) más abajo.

    ![Seguridad para la exportación manual](media/app-service-export-api-to-powerapps-and-flow/export-manual-security.png)

    En este ejemplo se muestra la definición de seguridad de la clave de API que se incluyó en la definición de OpenAPI.

Ahora que ha exportado la definición de API, la importa para crear un conector personalizado en PowerApps y Microsoft Flow. Los conectores personalizados se comparten entre los dos servicios, por lo que solo tiene que importar la definición una vez.

Para importar la definición de la API en PowerApps y Microsoft Flow, siga estos pasos:

1. Vaya a [powerapps.com](https://web.powerapps.com) o [flow.microsoft.com](https://flow.microsoft.com).

2. En la esquina superior derecha, haga clic en el icono con forma de engranaje y, a continuación, haga clic en **Conectores personalizados**.

   ![Icono de engranaje en el servicio](media/app-service-export-api-to-powerapps-and-flow/icon-gear.png)

3. Haga clic en **Crear conector personalizado** y, a continuación, en **Importar una definición de OpenAPI**.

   ![Crear un conector personalizado](media/app-service-export-api-to-powerapps-and-flow/flow-apps-create-connector.png)

4. Escriba un nombre para el conector personalizado, navegue a la definición de OpenAPI que exportó y haga clic en **Continuar**.

   ![Cargar la definición de OpenAPI](media/app-service-export-api-to-powerapps-and-flow/flow-apps-upload-definition.png)

4. En la ficha **General**, revise la información que procede de la definición de OpenAPI.

5. En la pestaña **Seguridad**, si se le pide que proporcione detalles de autenticación, escriba los valores adecuados para el tipo de autenticación. Haga clic en **Continuar**.

    ![Pestaña Seguridad](media/app-service-export-api-to-powerapps-and-flow/tab-security.png)

    En este ejemplo se muestran los campos obligatorios para la autenticación de la clave de API. Los campos difieren según el tipo de autenticación.

6. En la pestaña **Definiciones**, todas las operaciones definidas en el archivo de OpenAPI se rellenan automáticamente. Si se han definido todas las operaciones necesarias, puede ir al paso siguiente. Si no es así, puede agregar y modificar las operaciones aquí.

    ![Pestaña Definiciones](media/app-service-export-api-to-powerapps-and-flow/tab-definitions.png)

    Este ejemplo tiene una sola operación, denominada `CalculateCosts`. Los metadatos, como **Descripción**, provienen en todo caso del archivo OpenAPI.

7. Haga clic en **Crear conector** en la parte superior de la página.

Ahora puede conectarse al conector personalizado en PowerApps y Microsoft Flow. Para obtener más información sobre la creación de conectores en los portales de PowerApps y Microsoft Flow, consulte [Registrar y usar conectores personalizados en PowerApps](https://powerapps.microsoft.com/tutorials/register-custom-api/#register-your-custom-connector) y [Registro y uso de conectores personalizados en Microsoft Flow](/power-automate/get-started-flow-dev#create-a-custom-connector).

<a name="auth"></a>
## <a name="specify-authentication-type"></a>Especificar el tipo de autenticación

PowerApps y Microsoft Flow admiten una colección de proveedores de identidad que proporciona autenticación para conectores personalizados. Si la API requiere autenticación, asegúrese de que se captura como _definición de seguridad_ en el documento Open API, como el ejemplo siguiente:

```json
"securityDefinitions": {
    "AAD": {
    "type": "oauth2",
    "flow": "accessCode",
    "authorizationUrl": "https://login.windows.net/common/oauth2/authorize",
    "scopes": {}
    }
}
``` 
Durante la exportación, proporcione los valores de configuración que permiten a PowerApps y Microsoft Flow autenticar a los usuarios.

En esta sección se describen los tipos de autenticación admitidos en el modo **Rápido**: clave de API, Azure Active Directory y OAuth 2.0 genérico. PowerApps y Microsoft Flow también admiten la autenticación básica y OAuth 2.0 para servicios específicos como Dropbox, Facebook y SalesForce.

### <a name="api-key"></a>Clave de API
Cuando se usa una clave de API, a los usuarios de su conector se les pide que proporcionen la clave cuando creen una conexión. Especifique un nombre para la clave de API para ayudarlos a entender qué clave necesitan. En el ejemplo anterior, se utiliza el nombre `API Key (contact meganb@contoso.com)` para que los usuarios sepan dónde obtener información acerca de la clave de API. En Azure Functions, la clave es normalmente una de las claves de host, que abarcan varias funciones en una aplicación de función.

### <a name="azure-active-directory-azure-ad"></a>Azure Active Directory (Azure AD)
Al usar Azure AD, se necesitan dos registros de aplicación de Azure AD: uno para la propia API y otro para el conector personalizado:

- Para configurar el registro de la API, utilice la característica [Autenticación/Autorización de App Service](../app-service/configure-authentication-provider-aad.md).

- Para configurar el registro del conector, siga los pasos de [Agregar una aplicación de Azure AD](../active-directory/develop/quickstart-register-app.md). El registro debe tener acceso delegado a la API y una dirección URL de respuesta de `https://msmanaged-na.consent.azure-apim.net/redirect`. 

Para más información, vea los ejemplos de registro de Azure AD para [PowerApps](https://powerapps.microsoft.com/tutorials/customapi-azure-resource-manager-tutorial/) y [Microsoft Flow](/connectors/custom-connectors/azure-active-directory-authentication). En estos ejemplos se usa Azure Resource Manager como API; sustituya la API si sigue esos pasos.

Se requieren los siguientes valores de configuración:
- **Id. de cliente**: el identificador de cliente del registro de Azure AD del conector
- **Secreto de cliente**: el secreto de cliente del registro de Azure AD del conector
- **Dirección URL de inicio de sesión**: la dirección URL base para Azure AD. En Azure, normalmente `https://login.windows.net`.
- **Identificador de inquilino**: el identificador del inquilino que se usará para el inicio de sesión. Este identificador debe ser "common" o el identificador del inquilino en el que se crea el conector.
- **URL de recursos**: la dirección URL del recurso del registro de Azure AD para la API

> [!IMPORTANT]
> Si otra persona va a importar la definición de la API en PowerApps y Microsoft Flow como parte del flujo manual, tendrá que proporcionarle el identificador y el secreto de cliente *del registro del conector*, así como la dirección URL de recursos de la API. Asegúrese de que estos secretos se administran de forma segura. **No comparta las credenciales de seguridad de la propia API.**

### <a name="generic-oauth-20"></a>OAuth 2.0 genérico
Al utilizar OAuth 2.0 genérico, puede realizar la integración con cualquier proveedor de OAuth 2.0. Esto le permite trabajar con proveedores personalizados que no se admiten de forma nativa.

Se requieren los siguientes valores de configuración:
- **Id. de cliente**: el identificador de cliente de OAuth 2.0
- **Secreto de cliente**: el secreto de cliente de OAuth 2.0
- **Dirección URL de autorización**: la dirección URL de autorización de OAuth 2.0
- **Dirección URL de token**: la dirección URL del token de OAuth 2.0
- **Dirección URL de actualización**: la dirección URL de actualización de OAuth 2.0
