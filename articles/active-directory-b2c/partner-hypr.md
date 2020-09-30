---
title: Tutorial para configurar Azure Active Directory B2C con HYPR
titleSuffix: Azure AD B2C
description: Tutorial para configurar Azure Active Directory B2C con HYPR para la verdadera autenticación sólida de clientes sin contraseña
services: active-directory-b2c
author: gargi-sinha
manager: martinco
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 08/27/2020
ms.author: gasinh
ms.subservice: B2C
ms.openlocfilehash: 4f6b09061a4aa98824e176af55efcedfab3df48c
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89051874"
---
# <a name="tutorial-for-configuring-hypr-with-azure-active-directory-b2c"></a>Tutorial para configurar HYPR con Azure Active Directory B2C

En este tutorial de ejemplo, se proporcionan instrucciones sobre cómo configurar Azure AD B2C con [HYPR](https://get.hypr.com). Con Azure AD B2C como proveedor de identidades, puede integrar HYPR con cualquiera de las aplicaciones de cliente para proporcionar una verdadera autenticación sin contraseña para los usuarios. HYPR reemplaza las contraseñas con cifrados de clave pública, lo que elimina el fraude, la suplantación de identidad y la reutilización de credenciales.

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesitará lo siguiente:

- Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).

- Un [inquilino de Azure AD B2C](https://docs.microsoft.com/azure/active-directory-b2c/tutorial-create-tenant). El inquilino está vinculado a la suscripción de Azure.

- Un inquilino en la nube de HYPR; obtenga una [cuenta de prueba](https://get.hypr.com/free-trial) gratuita.

- Un dispositivo móvil de un usuario registrado mediante las API REST de HYPR o el Administrador de dispositivos de HYPR en el inquilino de HYPR. Por ejemplo, puede usar el [SDK de Java de HYPR](https://docs.hypr.com/integratinghypr/docs/hypr-java-web-sdk) para realizar esta tarea.

## <a name="scenario-description"></a>Descripción del escenario

La integración de HYRP incluye los siguientes componentes:

- Azure AD B2C: el servidor de autorización, responsable de comprobar las credenciales del usuario, al que también se le conoce como proveedor de identidades.

- Aplicaciones web y móviles: las aplicaciones web o móviles que decide proteger con HYPR y Azure AD B2C. HYPR proporciona un sólido SDK móvil así como una aplicación móvil que puede usar en las plataformas iOS y Android para realizar una verdadera autenticación sin contraseña.

- La aplicación móvil de HYPR: la aplicación móvil HYPR se puede usar para ejecutar este ejemplo si prefiere no usar los SDK móviles en sus propias aplicaciones móviles.

- API REST de HYPR: puede usar las API de HYPR para realizar el registro de dispositivos de usuario y la autenticación. Estas API se encuentran [aquí](https://apidocs.hypr.com).

En el siguiente diagrama de arquitectura se muestra la implementación.

![Captura de pantalla del diagrama de arquitectura de HYPR](media/partner-hypr/hypr-architecture-diagram.png)

|Paso | Descripción |
|:-----| :-----------|
| 1. | El usuario llega a una página de inicio de sesión. Los inician sesión o se registran y escriben el nombre de usuario en la página.
| 2. | La aplicación envía los atributos de usuario a Azure AD B2C para la comprobación de identificación.
| 3. | Azure AD B2C recopila los atributos de usuario y los envía a HYPR para autenticar al usuario a través de la aplicación móvil de HYPR.
| 4. | HYPR envía una notificación push al dispositivo móvil de usuario registrado para una autenticación con certificación de Fast Identity Online (FIDO). Puede ser un PIN de huella digital, biométrico o descentralizado del usuario.  
| 5. | Una vez que el usuario confirme la notificación push, se le concederá o denegará el acceso a la aplicación del cliente en función de los resultados de la comprobación.

## <a name="configure-the-azure-ad-b2c-policy"></a>Configuración de la directiva de Azure AD B2C

1. Vaya a la [directiva de HYPR de Azure AD B2C](https://github.com/HYPR-Corp-Public/Azure-AD-B2C-HYPR-Sample/tree/master/policy) en la carpeta Directiva.

2. Siga este [documento](https://docs.microsoft.com/azure/active-directory-b2c/custom-policy-get-started?tabs=applications#custom-policy-starter-pack) para descargar el [paquete de inicio de LocalAccounts](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/master/LocalAccounts).

3. Configure la directiva para el inquilino de Azure AD B2C.

>[!NOTE]
>Actualice las directivas proporcionadas para que se relacionen con su inquilino específico.

## <a name="test-the-user-flow"></a>Prueba del flujo de usuario

1. Abra el inquilino de Azure AD B2C y, en Directivas, seleccione **Identity Experience Framework**.

2. Seleccione el valor de **SignUpSignIn** que creó anteriormente.

3. Seleccione **Ejecutar flujo de usuario** y elija la configuración:

   a. **Aplicación**: seleccione la aplicación registrada (el ejemplo es JWT).

   b. **URL de respuesta**: seleccione la **dirección URL de redireccionamiento**.

   c. Seleccione **Ejecutar flujo de usuario**.

4. Recorra el flujo de registro y cree una cuenta.

5. Se llamará a HYPR durante el flujo, una vez creado el atributo de usuario. Si el flujo está incompleto, compruebe que el usuario no esté guardado en el directorio.

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte los artículos siguientes:

- [Directivas personalizadas de Azure AD B2C](https://docs.microsoft.com/azure/active-directory-b2c/custom-policy-overview)

- [Introducción a las directivas personalizadas en Azure AD B2C](https://docs.microsoft.com/azure/active-directory-b2c/custom-policy-get-started?tabs=applications)
