---
title: Tutorial para configurar Azure Active Directory B2C con Experian
titleSuffix: Azure AD B2C
description: Obtenga información sobre cómo integrar la autenticación de Azure AD B2C con Experian para la prueba y comprobación de identificación en función de los atributos de usuario para evitar fraudes.
services: active-directory-b2c
author: gargi-sinha
manager: martinco
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 07/22/2020
ms.author: gasinh
ms.subservice: B2C
ms.openlocfilehash: 2a69bc40090e74ad5885c2576c040693f90d5d03
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87095129"
---
# <a name="tutorial-for-configuring-experian-with-azure-active-directory-b2c"></a>Tutorial para configurar Experian con Azure Active Directory B2C

En este tutorial de ejemplo, se proporcionan instrucciones sobre cómo integrar Azure AD B2C con [Experian](https://www.experian.com/decision-analytics/account-opening-fraud/microsoft-integration). Experian proporciona numerosas soluciones, que puede encontrar [aquí](https://www.experian.com/).

En este ejemplo, se utiliza la plataforma de riesgo de fraude y la identidad digital integrada de Experian: **CrossCore**. CrossCore es un servicio de comprobación de identidad que se usa para comprobar la identificación del usuario. Realiza análisis de riesgos en función de varios fragmentos de información que proporciona el usuario durante el flujo de registro. CrossCore se usa para determinar si se debe permitir que el usuario continúe con el inicio de sesión o no. En el análisis de riesgos de CrossCore se pueden usar los siguientes atributos:

- Email
- Dirección IP
- Nombre propio
- Segundo nombre
- Surname
- Dirección
- City
- Estado o provincia
- Código postal
- País/región
- Número de teléfono

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesitará lo siguiente:

- Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).

- [Un inquilino de Azure AD B2C](https://docs.microsoft.com/azure/active-directory-b2c/tutorial-create-tenant) vinculado a la suscripción de Azure.

## <a name="scenario-description"></a>Descripción del escenario

La integración de Experian incluye los siguientes componentes:

- Azure AD B2C: el servidor de autorización, responsable de comprobar las credenciales del usuario, al que también se le conoce como proveedor de identidades.

- Experian: el servicio Experian toma la entrada que el usuario proporciona y comprueba la identidad de este.

- API REST personalizada: esta API implementa la integración entre Azure AD B2C y el servicio Experian.

En el siguiente diagrama de arquitectura se muestra la implementación.

![Captura de pantalla del diagrama de arquitectura de Experian](media/partner-experian/experian-architecture-diagram.png)

|Paso | Descripción |
|:-----| :-----------|
| 1. | El usuario llega a una página de inicio de sesión. El usuario opta por iniciar sesión para crear una nueva cuenta y escribir información en la página. Azure AD B2C recopila los atributos de usuario.
| 2. | Azure AD B2C llama a la API de nivel intermedio y pasa los atributos de usuario.
| 3. | La API de nivel intermedio recopila atributos de usuario y los transforma en un formato que la API de Experian puede consumir. A continuación, los envía a Experian.
| 4. | Experian consume la información y la procesa para validar la identificación del usuario en función del análisis de riesgos. A continuación, devuelve el resultado a la API de nivel intermedio.
| 5. | La API de nivel intermedio procesa la información y devuelve la información pertinente a Azure AD B2C en el formato JSON correcto.
| 6. | Azure AD B2C recibe información de la API de nivel intermedio. Si aparece la respuesta Error, se muestra un mensaje de error al usuario. Si aparece la respuesta Correcto, el usuario se autentica y se escribe en el directorio.

## <a name="onboard-with-experian"></a>Incorporación con Experian

1. Para crear una cuenta de Experian, póngase en contacto con [Experian](https://www.experian.com/decision-analytics/account-opening-fraud/microsoft-integration).

2. Una vez creada la cuenta, recibirá la información que necesita para la configuración de la API. El proceso se describe en las secciones siguientes.

## <a name="configure-azure-ad-b2c-with-experian"></a>Configuración de Azure AD B2C con Experian

### <a name="part-1---deploy-the-api"></a>Parte 1: implementación de la API

Implemente el código de la API proporcionado en un servicio de Azure. El código se puede publicar desde Visual Studio siguiendo estas [instrucciones](https://docs.microsoft.com/visualstudio/deployment/quickstart-deploy-to-azure?view=vs-2019).

>[!NOTE]
>Necesitará la dirección URL del servicio implementado para configurar Azure AD con la configuración necesaria.

### <a name="part-2---deploy-the-client-certificate"></a>Parte 2: implementación del certificado de cliente

La llamada a la API de Experian está protegida por un certificado de cliente. Este certificado de cliente lo proporcionará Experian. Siguiendo las instrucciones mencionadas en este [documento](https://docs.microsoft.com/azure/app-service/environment/certificates#private-client-certificate), el certificado se debe cargar en el servicio App de Azure. La directiva de ejemplo utiliza estos pasos de claves en el proceso:

- Carga del certificado

- Establezca la clave `WEBSITE_LOAD_ROOT_CERTIFICATES` con la huella digital del certificado.

### <a name="part-3---configure-the-api"></a>Parte 3: configuración de la API

La configuración de la aplicación se puede [configurar en el servicio App en Azure](https://docs.microsoft.com/azure/app-service/configure-common#configure-app-settings). Con este método, los valores se pueden configurar de forma segura sin registrarlos en un repositorio. Debe especificar la configuración siguiente para la API de REST:

| Configuración de la aplicación | Source | Notas |
| :-------- | :------------| :-----------|
|CrossCoreConfig:TenantId | Configuración de la cuenta de Experian |     |
|CrossCoreConfig:OrgCode | Configuración de la cuenta de Experian |     |
|CrossCore:ApiEndpoint |Configuración de la cuenta de Experian|  |
|CrossCore:ClientReference | Configuración de la cuenta de Experian | |
| CrossCore:ModelCode |Configuración de la cuenta de Experian|
| CrossCore:OrgCode | Configuración de la cuenta de Experian |
| CrossCore:SignatureKey  | Configuración de la cuenta de Experian |
| CrossCore:TenantId  | Configuración de la cuenta de Experian |
| CrossCore:CertificateThumbprint | Certificado de Experian |
| BasicAuth:ApiUsername | Defina un nombre de usuario para la API | Se usa en la configuración de ExtId |
| BasicAuth:ApiPassword | Defina una contraseña para la API | Se usa en la configuración de ExtId

### <a name="part-4---create-api-policy-keys"></a>Parte 4: creación de claves de directiva de API

Consulte este [documento](https://docs.microsoft.com/azure/active-directory-b2c/secure-rest-api#add-rest-api-username-and-password-policy-keys) para crear dos claves de directiva: una para el nombre de usuario de la API y otra para la contraseña de la API que definió anteriormente para la autenticación básica HTTP.

>[!NOTE]
>Necesitará las claves para configurar las directivas más adelante.

### <a name="part-5---replace-the-configuration-values"></a>Parte 5: reemplazo de los valores de configuración

En las directivas personalizadas proporcionadas, busque los siguientes marcadores de posición y reemplácelos por los valores correspondientes de la instancia.

|                      Marcador de posición                       |                                   Reemplazar por el valor                                 |                   Ejemplo                    |
| ------------------------------------------------------ | -------------------------------------------------------------------------------- | -------------------------------------------- |
| {your_tenant_name}                                     | Su nombre corto de inquilino                                                           | "yourtenant" de yourtenant.onmicrosoft.com |
| {your_trustframeworkbase_policy}                       | Nombre de Azure AD B2C de la directiva de TrustFrameworkBase                  | B2C_1A_experian_TrustFrameworkBase           |
| {your_tenant_IdentityExperienceFramework_appid}        | Identificador de aplicación de la aplicación IdentityExperienceFramework configurada en el inquilino de Azure AD B2C      | 01234567-89ab-cdef-0123-456789abcdef         |
| {your_tenant_ ProxyIdentityExperienceFramework _appid} | Identificador de aplicación de la aplicación ProxyIdentityExperienceFramework configurada en el inquilino de Azure AD B2C | 01234567-89ab-cdef-0123-456789abcdef         |
| {your_tenant_extensions_appid}                         | Identificador de aplicación de la aplicación de almacenamiento del inquilino                                      | 01234567-89ab-cdef-0123-456789abcdef         |
| {your_tenant_extensions_app_objectid}                  | Identificador de objeto de la aplicación de almacenamiento del inquilino                                   | 01234567-89ab-cdef-0123-456789abcdef         |
| {your_api_username_key_name}                           | Nombre de la clave del nombre de usuario que creó [aquí](#part-4---create-api-policy-keys)             | B2C\_1A\_RestApiUsername                     |
| {your_api_password_key_name}                           | Nombre de la clave de contraseña que creó [aquí](#part-4---create-api-policy-keys)             | B2C\_1A\_RestApiPassword                     |
| {your_app_service_URL}                                 | Dirección URL del servicio de aplicaciones que ha configurado                                             | <https://yourapp.azurewebsites.net>          |

### <a name="part-6---configure-the-azure-ad-b2c-policy"></a>Parte 6: configuración de la directiva de Azure AD B2C

Consulte este [documento](https://docs.microsoft.com/azure/active-directory-b2c/custom-policy-get-started?tabs=applications#custom-policy-starter-pack) para obtener instrucciones sobre cómo configurar el inquilino de Azure AD B2C y las directivas.

>[!NOTE]
>Esta directiva de ejemplo se basa en el [paquete de inicio de cuentas locales](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/master/LocalAccounts).

>[!NOTE]
> Como procedimiento recomendado, se sugiere que los clientes agreguen una notificación de consentimiento en la página de colección de atributos. Notifique a los usuarios que la información se enviará a servicios de terceros para la verificación de identidad.

## <a name="test-the-user-flow"></a>Prueba del flujo de usuario

1. Abra el inquilino de Azure AD B2C y, en Directivas, seleccione **Flujos de usuario**.

2. Seleccione el **flujo de usuario** que creó anteriormente.

3. Seleccione **Ejecutar flujo de usuario** y elija la configuración:

   a. **Aplicación**: seleccione la aplicación registrada (en el ejemplo es JWT).

   b. **URL de respuesta**: seleccione la **dirección URL** de redireccionamiento.

   c. Seleccione **Ejecutar flujo de usuario**.

4. Recorra el flujo de registro y cree una cuenta.

5. Cierre la sesión.

6. Recorra el flujo de inicio de sesión.  

7. El rompecabezas CrossCore aparecerá después de escribir **continuar**.

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte los artículos siguientes:

- [Directivas personalizadas de Azure AD B2C](https://docs.microsoft.com/azure/active-directory-b2c/custom-policy-overview)

- [Introducción a las directivas personalizadas en Azure AD B2C](https://docs.microsoft.com/azure/active-directory-b2c/custom-policy-get-started?tabs=applications)
