---
title: Tutorial para configurar Azure Active Directory B2C con Onfido
titleSuffix: Azure AD B2C
description: Obtenga información sobre cómo integrar la autenticación de Azure AD B2C con Onfido para la verificación de biometría facial y el id. de documento.
services: active-directory-b2c
author: gargi-sinha
manager: martinco
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 08/03/2020
ms.author: gasinh
ms.subservice: B2C
ms.openlocfilehash: 580d3cea94e3c550954fc0fa1aa6f2c3a73e9386
ms.sourcegitcommit: 1b2d1755b2bf85f97b27e8fbec2ffc2fcd345120
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87554738"
---
# <a name="tutorial-for-configuring-onfido-with-azure-active-directory-b2c"></a>Tutorial para configurar Onfido con Azure Active Directory B2C

En este tutorial de ejemplo, se proporcionan instrucciones sobre cómo integrar Azure AD B2C con [Onfido](https://onfido.com/). Onfido es una aplicación de verificación de biométrica facial e id. de documento. Permite a las empresas satisfacer los requisitos de identidad y de *Conozca a su cliente* en tiempo real. Onfido usa la verificación de identidad sofisticada basada en IA, que primero comprueba un identificador de foto y, a continuación, lo hace coincidir con su biometría facial. Esta solución vincula una identidad digital a la persona real y proporciona una experiencia de incorporación segura a la vez que reduce el fraude.

En este ejemplo, se conecta el servicio de Onfido en el flujo de inicio de sesión o de registro para realizar la verificación de identidad. Las decisiones informadas sobre el producto y servicio al que puede obtener acceso el usuario se basan en los resultados de Onfido.

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesitará lo siguiente:

- Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).

- [Un inquilino de Azure AD B2C](https://docs.microsoft.com/azure/active-directory-b2c/tutorial-create-tenant) vinculado a la suscripción de Azure.

- Una [cuenta de prueba](https://onfido.com/signup/) de Onfido.

## <a name="scenario-description"></a>Descripción del escenario

La integración de Onfido incluye los siguientes componentes:

- Inquilino de Azure AD B2C: el servidor de autorización, responsable de comprobar las credenciales del usuario en función de las directivas personalizadas definidas en el inquilino. También se conoce como proveedor de identidades. Hospeda la aplicación cliente de Onfido, que recopila los documentos de usuario y los transmite al servicio de API de Onfido.

- Cliente de Onfido: utilidad de recopilación de documentos de cliente JavaScript configurable implementada en otras páginas web. Recopila los documentos y realiza comprobaciones preliminares como el tamaño y la calidad del documento.

- API REST intermedia: proporciona puntos de conexión para que el inquilino de Azure AD B2C se comunique con el servicio de API de Onfido, controle el procesamiento de datos y cumpla los requisitos de seguridad de ambos.

- Servicio de API de Onfido: servicio de back-end proporcionado por Onfido, que guarda y comprueba los documentos proporcionados por el usuario.

En el siguiente diagrama de arquitectura se muestra la implementación.

![Captura de pantalla del diagrama de arquitectura de Onfido](media/partner-onfido/onfido-architecture-diagram.png)

|Paso | Descripción |
|:-----| :-----------|
| 1. | El usuario llega a una página de inicio de sesión. El usuario inicia sesión para crear una nueva cuenta y escribir información en la página. Azure AD B2C recopila los atributos de usuario. La aplicación cliente de Onfido hospedada en Azure AD B2C realiza comprobaciones preliminares de la información del usuario.
| 2. | Azure AD B2C llama a la API de nivel intermedio y pasa los atributos de usuario.
| 3. | La API de nivel intermedio recopila atributos de usuario y los transforma en un formato que la API de Onfido puede consumir. A continuación, los envía a Onfido.
| 4. | Onfido consume la información y la procesa para validar la identificación del usuario. A continuación, devuelve el resultado a la API de nivel intermedio.
| 5. | La API de nivel intermedio procesa la información y devuelve la información pertinente a Azure AD B2C en el formato JSON correcto.
| 6. | Azure AD B2C recibe información de la API de nivel intermedio. Si aparece la respuesta Error, se muestra un mensaje de error al usuario. Si aparece la respuesta Correcto, el usuario se autentica y se escribe en el directorio.

## <a name="onboard-with-onfido"></a>Incorporación con Onfido

1. Para crear una cuenta de Onfido, póngase en contacto con [Onfido](https://onfido.com/signup/).

2. Una vez creada la cuenta, cree una [clave de API](https://documentation.onfido.com/). Las claves publicadas son facturables, pero puede usar las [claves de espacio aislado para probar](https://documentation.onfido.com/?javascript#sandbox-and-live-differences) la solución. Las claves de espacio aislado producen la misma estructura de resultados que las claves publicadas. Sin embargo, los resultados siempre están predeterminados. Los documentos no se procesan ni se guardan.

>[!NOTE]
> Necesitará la clave más adelante.

Para obtener más información sobre Onfido, consulte la [documentación de la API de Onfido](https://documentation.onfido.com) y el [Centro de desarrolladores de Onfido](https://developers.onfido.com).  

## <a name="configure-azure-ad-b2c-with-onfido"></a>Configuración de Azure AD B2C con Onfido

### <a name="part-1---deploy-the-api"></a>Parte 1: implementación de la API

- Implemente el [código de la API](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/OnFido-Combined/API/Onfido.Api) proporcionado en un servicio de Azure. El código se puede publicar desde Visual Studio siguiendo estas [instrucciones](https://docs.microsoft.com/visualstudio/deployment/quickstart-deploy-to-azure?view=vs-2019).
- Configure CORS y agregue **Origen permitido** como https://{your_tenant_name}.b2clogin.com.

>[!NOTE]
>Necesitará la dirección URL del servicio implementado para configurar Azure AD con la configuración necesaria.

#### <a name="adding-sensitive-configuration-settings"></a>Adición de valores de configuración confidenciales

La configuración de la aplicación se puede configurar en [App Service en Azure](https://docs.microsoft.com/azure/app-service/configure-common#configure-app-settings). App Service permite configurar los valores de forma segura sin registrarlos en un repositorio. La API REST necesita la siguiente configuración:

| Nombre de la configuración de la aplicación | Source | Notas |
|:-------------------------|:-------|:-------|
|OnfidoSettings:AuthToken| Cuenta de Onfido |

### <a name="part-2---deploy-the-ui"></a>Parte 2: implementación de la interfaz de usuario

#### <a name="configure-your-storage-location"></a>Configuración de la ubicación de almacenamiento

1. Configure un [contenedor de blobs en la cuenta de almacenamiento](https://docs.microsoft.com/azure/storage/blobs/storage-quickstart-blobs-portal#create-a-container).

2. Almacene los archivos de la interfaz de usuario de la carpeta de la interfaz de usuario en el contenedor de blobs.

3. Permita el acceso a CORS al contenedor de almacenamiento que creó siguiendo estas instrucciones:

   a. Vaya a **Configuración** >**Origen permitido** y escriba `https://{your_tenant_name}.b2clogin.com`. Reemplace el nombre del inquilino por el nombre del inquilino de Azure AD B2C. Por ejemplo: https://fabrikam.b2clogin.com. Al escribir su nombre de inquilino, use solo minúsculas.

   b. En **Métodos permitidos**, seleccione `GET` y `PUT`.

   c. Seleccione **Guardar**.

#### <a name="update-ui-files"></a>Actualización de los archivos de interfaz de usuario

1. En los archivos de la interfaz de usuario, vaya a la carpeta [**ocean_blue**](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/OnFido-Combined/UI/ocean_blue).

2. Abra los archivos HTML.

3. Busque {your-ui-blob-container-url} y reemplácelo por la dirección URL donde se encuentran las carpetas **ocean_blue**, **dist** y **assets** de la interfaz de usuario.

4. Busque {your-intermediate-api-url} y reemplácelo por la dirección URL del servicio de aplicaciones de API intermedias.

#### <a name="upload-your-files"></a>Carga de los archivos

1. Almacene los archivos de la interfaz de usuario de la carpeta de la interfaz de usuario en el contenedor de blobs.

2. Use [Explorador de Azure Storage](https://docs.microsoft.com/azure/virtual-machines/windows/disks-use-storage-explorer-managed-disks) para administrar los archivos y permisos de acceso.

### <a name="part-3---configure-azure-ad-b2c"></a>Parte 3: configuración de Azure AD B2C

#### <a name="replace-the-configuration-values"></a>Reemplazo de los valores de configuración

En las directivas personalizadas proporcionadas, busque los siguientes marcadores de posición y reemplácelos por los valores correspondientes de la instancia.

| Marcador de posición | Reemplazar por el valor | Ejemplo  |
|:---------------|:----------------|:-------------------|
| {your_tenant_name}  | Su nombre corto de inquilino |  "yourtenant" de yourtenant.onmicrosoft.com |
| {your_tenantID} | TenantID de su inquilino de Azure AD B2C | 01234567-89ab-cdef-0123-456789abcdef           |
| {your_tenant_IdentityExperienceFramework_appid}        | Identificador de aplicación de la aplicación IdentityExperienceFramework configurada en el inquilino de Azure AD B2C      | 01234567-89ab-cdef-0123-456789abcdef         |
| {your_tenant_ ProxyIdentityExperienceFramework _appid} | Identificador de aplicación de la aplicación ProxyIdentityExperienceFramework configurada en el inquilino de Azure AD B2C | 01234567-89ab-cdef-0123-456789abcdef         |
| {your_tenant_extensions_appid}                         | Identificador de aplicación de la aplicación de almacenamiento del inquilino                                      | 01234567-89ab-cdef-0123-456789abcdef         |
| {your_tenant_extensions_app_objectid}                  | Identificador de objeto de la aplicación de almacenamiento del inquilino                                   | 01234567-89ab-cdef-0123-456789abcdef         |
| {your_app_insights_instrumentation_key} | Clave de instrumentación de la instancia de App Insights*| 01234567-89ab-cdef-0123-456789abcdef|
|{your_ui_file_base_url}| Dirección URL de la ubicación donde se encuentran las carpetas **ocean_blue**, **dist** y **assets** de la interfaz de usuario | https://yourstorage.blob.core.windows.net/UI/|
| {your_app_service_URL}                                 | Dirección URL del servicio de aplicaciones que ha configurado                                             | <https://yourapp.azurewebsites.net>          |

*App Insights puede estar en un inquilino diferente. Este paso es opcional. Quite los TechnicalProfiles y OrchestrationSteps correspondientes si no son necesarios.

### <a name="part-4---configure-the-azure-ad-b2c-policy"></a>Parte 4: configuración de la directiva de Azure AD B2C

Consulte este [documento](https://docs.microsoft.com/azure/active-directory-b2c/custom-policy-get-started?tabs=applications#custom-policy-starter-pack) para obtener instrucciones sobre cómo configurar el inquilino de Azure AD B2C y las directivas.

>[!NOTE]
> Como procedimiento recomendado, se sugiere que los clientes agreguen una notificación de consentimiento en la página de colección de atributos. Notifique a los usuarios que la información se enviará a servicios de terceros para la verificación de identidad.

## <a name="test-the-user-flow"></a>Prueba del flujo de usuario

1. Abra el inquilino de Azure AD B2C y, en Directivas, seleccione **Identity Experience Framework**.

2. Seleccione el valor de **SignUpSignIn** que creó anteriormente.

3. Seleccione **Ejecutar flujo de usuario** y elija la configuración:

   a. **Aplicación**: seleccione la aplicación registrada (el ejemplo es JWT).

   b. **URL de respuesta**: seleccione la **dirección URL de redireccionamiento**.

   c. Seleccione **Ejecutar flujo de usuario**.

4. Recorra el flujo de registro y cree una cuenta.

5. Se llamará al servicio de Onfido durante el flujo, una vez creado el atributo de usuario. Si el flujo está incompleto, compruebe que el usuario no esté guardado en el directorio.

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte los artículos siguientes:

- [Directivas personalizadas de Azure AD B2C](https://docs.microsoft.com/azure/active-directory-b2c/custom-policy-overview)

- [Introducción a las directivas personalizadas en Azure AD B2C](https://docs.microsoft.com/azure/active-directory-b2c/custom-policy-get-started?tabs=applications)
