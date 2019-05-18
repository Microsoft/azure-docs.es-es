---
title: 'Depuración del inicio de sesión único basado en SAML: Azure Active Directory | Microsoft Docs'
description: Depure el inicio de sesión único basado en SAML en aplicaciones de Azure Active Directory.
services: active-directory
author: rwike77
documentationcenter: na
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 02/18/2019
ms.author: ryanwi
ms.custom: aaddev
ms.reviewer: luleon, hirsin, smalser
ms.collection: M365-identity-device-management
ms.openlocfilehash: abbc0965973d58f7716abac0e9e9834f17c99203
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65540522"
---
# <a name="debug-saml-based-single-sign-on-to-applications-in-azure-active-directory"></a>Depuración del inicio de sesión único basado en SAML en aplicaciones de Azure Active Directory

Obtenga información acerca de cómo buscar y corregir problemas del [inicio de sesión único](../manage-apps/what-is-single-sign-on.md) en aplicaciones de Azure Active Directory (Azure AD) compatibles con el [Lenguaje de marcado de aserción de seguridad (SAML) 2.0](https://en.wikipedia.org/wiki/Security_Assertion_Markup_Language). 

## <a name="before-you-begin"></a>Antes de empezar

Se recomienda instalar la [Extensión de inicio de sesión seguro de mis aplicaciones](../user-help/my-apps-portal-end-user-troubleshoot.md#im-having-trouble-installing-the-my-apps-secure-sign-in-extension). Esta extensión de explorador facilita recopilar la solicitud SAML y la información de respuesta SAML que necesita para resolver problemas con el inicio de sesión único. En caso de que no pueda instalar la extensión, este artículo muestra cómo resolver problemas con y sin la extensión instalada.

Para descargar e instalar la extensión de inicio de sesión seguro de mis aplicaciones, utilice uno de los siguientes vínculos.

- [Chrome](https://go.microsoft.com/fwlink/?linkid=866367)
- [Microsoft Edge](https://go.microsoft.com/fwlink/?linkid=845176)
- [Firefox](https://go.microsoft.com/fwlink/?linkid=866366)


## <a name="test-saml-based-single-sign-on"></a>Prueba del inicio de sesión único basado en SAML

Para probar basado en SAML sesión único entre Azure AD y una aplicación de destino:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador global u otro administrador que esté autorizado para administrar aplicaciones.
1. En la hoja de la izquierda, seleccione **Azure Active Directory**y, a continuación, seleccione **aplicaciones empresariales**. 
1. En la lista de aplicaciones empresariales, seleccione la aplicación para que va a probar el inicio de sesión único y, a continuación, de las opciones de la izquierda, seleccione **inicio de sesión único**.
1. Para abrir la basado en SAML única inicio de sesión en experiencia de prueba, vaya a **probar el inicio de sesión único en** (paso 5). Si el **prueba** botón está atenuado, deberá rellenar y guarde los atributos necesarios en primer lugar el **configuración básica de SAML** sección.
1. En la hoja **Probar el inicio de sesión único**, use sus credenciales corporativas para iniciar sesión en la aplicación de destino. Puede iniciar sesión como el usuario actual o como otro usuario. Si inicia sesión como otro usuario, un mensaje le pedirá que se autentique.

    ![Prueba de página SAML](./media/howto-v1-debug-saml-sso-issues/test-single-sign-on.png)


Si ha iniciado sesión correctamente, ha superado la prueba. En este caso, Azure AD ha emitido un token de respuesta de SAML a la aplicación. La aplicación ha usado el token de SAML para su inicio de sesión correcto.

Si tiene un error en la página de inicio de sesión de la compañía o la página de la aplicación, use una de las secciones siguientes para resolver el error.


## <a name="resolve-a-sign-in-error-on-your-company-sign-in-page"></a>Resolver un error de inicio de sesión en la página de inicio de sesión de compañía

Al intentar iniciar sesión, verá un error en la página de inicio de sesión de compañía es similar al ejemplo siguiente.

![Error de inicio de sesión](./media/howto-v1-debug-saml-sso-issues/error.png)

Para depurar este error, se necesita el mensaje de error y la solicitud SAML. La Extensión de inicio de sesión seguro de mis aplicaciones recopila automáticamente esta información y muestra una guía para la resolución en Azure AD. 

### <a name="to-resolve-the-sign-in-error-with-the-my-apps-secure-sign-in-extension-installed"></a>Para resolver el error de inicio de sesión con la extensión de inicio de sesión seguro de mis aplicaciones instaladas

1. Cuando se produce un error, la extensión redirige a Azure AD **probar el inicio de sesión único en** hoja. 
1. En el **probar el inicio de sesión único en** hoja, seleccione **descargar la solicitud SAML**. 
1. Debería ver instrucciones específicas para la resolución en función del error y los valores de la solicitud SAML.
1. Verá un **corregirlo** botón para actualizar automáticamente la configuración de Azure AD para resolver el problema. Si no ve este botón, a continuación, el problema de inicio de sesión no es debido a una configuración incorrecta en Azure AD.

Si no se proporciona ninguna resolución del error de inicio de sesión, se recomienda que utilice el cuadro de texto de comentarios para informar a nosotros.

### <a name="to-resolve-the-error-without-installing-the-my-apps-secure-sign-in-extension"></a>Para resolver el error sin necesidad de instalar la extensión de inicio de sesión seguro de mis aplicaciones

1. Copie el mensaje de error de la esquina inferior derecha de la página. El mensaje de error incluye:
    - Un elemento CorrelationID y una marca de tiempo. Estos valores son importantes cuando se crea un caso de soporte técnico con Microsoft ya que ayudan a los ingenieros a identificar el problema y a proporcionar una solución más precisa y rápida para el problema.
    - Una instrucción que identifica la causa principal del problema.
1. Vuelva a Azure AD y busque la hoja **Probar el inicio de sesión único**.
1. En el cuadro de texto situado encima de **Get resolution guidance** (Obtener instrucciones para la resolución), pegue el mensaje de error.
1. Haga clic en **Get resolution guidance** (Obtener instrucciones para la resolución) para mostrar los pasos para resolver el problema. Las instrucciones pueden requerir información de la solicitud SAML o la respuesta de SAML. Si no usa la extensión de inicio de sesión seguro de mis aplicaciones, es posible que necesita una herramienta como [Fiddler](https://www.telerik.com/fiddler) para recuperar la solicitud SAML y la respuesta.
1. Compruebe que el destino en la solicitud SAML se corresponde con el SAML Single Sign-On Service URL obtenido de Azure AD.
1. Compruebe que el emisor en la solicitud SAML es el mismo identificador que se ha configurado para la aplicación en Azure AD. Azure AD usa al emisor para buscar una aplicación en el directorio.
1. Compruebe que assertionconsumerserviceurl es donde la aplicación espera recibir el token de SAML de Azure AD. Este valor se puede configurar en Azure AD, pero no es obligatorio si forma parte de la solicitud SAML.


## <a name="resolve-a-sign-in-error-on-the-application-page"></a>Resolución de un error de inicio de sesión único en la página de la aplicación

Puede iniciar sesión correctamente y luego ver un error en la página de la aplicación. Esto se produce cuando Azure AD ha emitido un token a la aplicación, pero la aplicación no acepta la respuesta.   

Para solucionar el error, siga estos pasos:

1. Si la aplicación está en la Galería de Azure AD, compruebe que ha seguido todos los pasos para integrar la aplicación con Azure AD. Para buscar las instrucciones de integración de la aplicación, consulte la [lista de tutoriales sobre la integración de aplicaciones de SaaS](../saas-apps/tutorial-list.md).
1. Recupere la respuesta SAML.
    - Si se ha instalado la extensión de inicio de sesión seguro de mis aplicaciones, en la hoja **Probar el inicio de sesión único**, haga clic en **download the SAML response** (Descargar respuesta SAML).
    - Si no se ha instalado la extensión, use una herramienta como [Fiddler](https://www.telerik.com/fiddler) para recuperar la respuesta SAML. 
1. Observe estos elementos en el token de la respuesta SAML:
   - Identificador único del usuario del valor NameID y formato
   - Notificaciones emitidas en el token
   - Certificado usado en el token de inicio de sesión. 

     Para más información sobre la respuesta SAML, vea [Protocolo SAML de inicio de sesión único](single-sign-on-saml-protocol.md).

1. Ahora que ha revisado la respuesta de SAML, consulte [Error en la página de la aplicación después de iniciar sesión](../manage-apps/application-sign-in-problem-application-error.md) para obtener instrucciones sobre cómo resolver el problema. 
1. Si le sigue sin poder iniciar sesión correctamente, puede pedir el proveedor de la aplicación, lo que no está presente en la respuesta de SAML.


## <a name="next-steps"></a>Pasos siguientes

Ahora que el inicio de sesión único está funcionando su aplicación, podría [automatización del aprovisionamiento y Desaprovisionamiento de usuarios para aplicaciones SaaS](../manage-apps/user-provisioning.md) o [empezar a trabajar con el acceso condicional](../conditional-access/app-based-conditional-access.md).
