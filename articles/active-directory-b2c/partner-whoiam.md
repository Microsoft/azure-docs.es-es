---
title: Tutorial para configurar Azure Active Directory B2C con WhoIAM
titleSuffix: Azure AD B2C
description: En este tutorial, aprenderá a integrar la autenticación de Azure AD B2C con WhoIAM para la verificación de usuarios.
services: active-directory-b2c
author: gargi-sinha
manager: martinco
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 08/20/2020
ms.author: gasinh
ms.subservice: B2C
ms.openlocfilehash: 718ccbaa57ffe9f4ebaf4e8df448b602ba8cc3fa
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89293158"
---
# <a name="tutorial-for-configuring-whoiam-with-azure-active-directory-b2c"></a>Tutorial para configurar WhoIAM con Azure Active Directory B2C

En este tutorial de ejemplo, se proporcionan instrucciones sobre cómo configurar Branded Identity Management System (BRIMS) de [WhoIAM](https://www.whoiam.ai/brims/) en el entorno y cómo integrarlo con Active Directory B2C (Azure AD B2C).

BRIMS es un conjunto de aplicaciones y servicios que se implementa en su entorno. Proporciona la verificación de voz, SMS y correo electrónico de la base de usuarios. BRIMS funciona junto con la solución de administración de identidades y acceso existente y es independiente de la plataforma.

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesitará lo siguiente:

- Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).

- [Un inquilino de Azure AD B2C](https://docs.microsoft.com/azure/active-directory-b2c/tutorial-create-tenant) vinculado a la suscripción de Azure.

- Una [cuenta de prueba](https://www.whoiam.ai/contact-us/) de WhoIAM.

## <a name="scenario-description"></a>Descripción del escenario

La integración de WhoIAM incluye los siguientes componentes:

- Un inquilino de Azure AD B2C. Es el servidor de autorización que comprueba las credenciales del usuario en función de las directivas personalizadas definidas en este. También se conoce como proveedor de identidades.

- Un portal de administración para administrar clientes y sus configuraciones.

- Un servicio de API que expone varias características mediante puntos de conexión.  

- Azure Cosmos DB, que actúa como back-end para el portal de administración de BRIMS y el servicio de API.

En el siguiente diagrama de arquitectura se muestra la implementación.

![Diagrama de la arquitectura de la integración de Azure AD B2C con WhoIAM.](media/partner-whoiam/whoiam-architecture-diagram.png)

|Paso | Descripción |
|:-----| :-----------|
| 1. | El usuario llega a una página para iniciar una solicitud de registro o inicio de sesión en una aplicación que usa Azure AD B2C como proveedor de identidades.
| 2. | Como parte de la autenticación, el usuario solicita verificar la propiedad de su correo electrónico o teléfono, o bien usar su voz como factor de verificación biométrico.  
| 3. | Azure AD B2C realiza una llamada al servicio de API de BRIMS y pasa la dirección de correo electrónico, el número de teléfono y la grabación de voz del usuario.
| 4. | BRIMS usa configuraciones predefinidas, como plantillas de correo electrónico y SMS completamente personalizables para interactuar con el usuario en su idioma respectivo de un modo coherente con el estilo de la aplicación.
| 5. | Una vez finalizada la verificación de la identidad de un usuario, BRIMS devuelve un token a Azure AD B2C para indicar el resultado de la verificación. A continuación, Azure AD B2C concede al usuario acceso a la aplicación o genera un error en el intento de autenticación.  

## <a name="sign-up-with-whoiam"></a>Registro con WhoIAM

1. Póngase en contacto con [WhoIAM](https://www.whoiam.ai/contact-us/) y cree una cuenta de BRIMS.

2. Use las directrices de registro que tiene a su disposición y configure los siguientes servicios de Azure:

    - [Azure Key Vault](https://azure.microsoft.com/services/key-vault/): se usa para el almacenamiento seguro de contraseñas como, por ejemplo, contraseñas del servicio de correo electrónico.

    - [Azure App Service](https://azure.microsoft.com/services/app-service/): se usa para hospedar la API y los servicios del portal de administración de BRIMS.

    - [Azure Active Directory](https://azure.microsoft.com/services/active-directory/): se usa para autenticar a los usuarios administrativos en el portal de administración.

    - [Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/): se usa para almacenar y recuperar la configuración.

    - [Application Insights](https://docs.microsoft.com/azure/azure-monitor/app/app-insights-overview#:~:text=Application%20Insights%2C%20a%20feature%20of%20Azure%20Monitor%2C%20is,professionals.%20Use%20it%20to%20monitor%20your%20live%20applications) (opcional): se usa para iniciar sesión tanto en la API como en el portal de administración.

3. Implemente la API de BRIMS y el portal de administración de BRIMS en su entorno de Azure.

4. Puede encontrar ejemplos de directivas personalizadas de Azure AD B2C en la documentación de registro de BRIMS. Siga la documentación para configurar la aplicación y use la plataforma BRIMS para la verificación de la identidad del usuario.  

Para más información sobre BRIMS de WhoIAM, consulte la [documentación del producto](https://www.whoiam.ai/brims/).

## <a name="test-the-user-flow"></a>Prueba del flujo de usuario

1. Abra el inquilino de Azure AD B2C. En **Directivas**, seleccione **Identity Experience Framework**.

2. Seleccione el valor de **SignUpSignIn** que creó anteriormente.

3. Seleccione **Ejecutar el flujo del usuario** y, a continuación:

   a. Para **Aplicación**, seleccione la aplicación registrada (la muestra es JWT).

   b. Para **URL de respuesta**, seleccione la **URL de redireccionamiento**.

   c. Seleccione **Ejecutar flujo de usuario**.

4. Recorra el flujo de registro y cree una cuenta.

5. Se llamará al servicio BRIMS durante el flujo, una vez creado el atributo de usuario. Si el flujo está incompleto, compruebe que el usuario no esté guardado en el directorio.

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte los artículos siguientes:

- [Directivas personalizadas de Azure AD B2C](https://docs.microsoft.com/azure/active-directory-b2c/custom-policy-overview)

- [Introducción a las directivas personalizadas en Azure AD B2C](https://docs.microsoft.com/azure/active-directory-b2c/custom-policy-get-started?tabs=applications)
