---
title: Inicio de sesión único de SAML en aplicaciones locales mediante el proxy de aplicación de Azure AD
description: Aprenda a proporcionar un inicio de sesión único para aplicaciones locales que estén protegidas con la autenticación de SAML. Proporcione acceso remoto a aplicaciones locales con Application Proxy.
services: active-directory
documentationcenter: ''
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 10/24/2019
ms.author: kenwith
ms.reviewer: japere
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2c706de8a7638c16f3778d0d5295069c13b79d98
ms.sourcegitcommit: 5b8fb60a5ded05c5b7281094d18cf8ae15cb1d55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87387275"
---
# <a name="saml-single-sign-on-for-on-premises-applications-with-application-proxy"></a>Inicio de sesión único de SAML para aplicaciones en el entorno local con Application Proxy

Puede proporcionar el inicio de sesión único (SSO) para las aplicaciones en el entorno local que están protegidas con la autenticación de SAML y acceso remoto a estas aplicaciones mediante Application Proxy. Con el inicio de sesión único de SAML, Azure Active Directory (Azure AD) se autentica en la aplicación mediante el uso de la cuenta de Azure AD del usuario. Azure AD comunica la información de inicio de sesión a la aplicación a través de un protocolo de conexión. También puede asignar los usuarios a roles de aplicación específicos según las reglas que defina en las notificaciones de SAML. Al habilitar Application Proxy además de SSO de SAML, los usuarios tendrán acceso externo a la aplicación y una experiencia fluida de inicio de sesión único.

Las aplicaciones deben ser capaces de consumir los tokens SAML emitidos por **Azure Active Directory**. Esta configuración no es aplicable a las aplicaciones que usan un proveedor de identidades en el entorno local. Para estos escenarios, se recomienda revisar los [recursos para migrar aplicaciones a Azure AD](migration-resources.md).

SSO de SAML con Application Proxy también funciona con la característica de cifrado de tokens de SAML. Para más información, consulte [Configuración del cifrado de tokens SAML de Azure AD](howto-saml-token-encryption.md).

Los diagramas de protocolos siguientes describen la secuencia de inicio de sesión único para un flujo iniciado por el proveedor de servicios y un flujo iniciado por el proveedor de identidades. Application Proxy funciona con el inicio de sesión único de SAML mediante el almacenamiento en caché de la solicitud y respuesta de SAML en la aplicación local y desde esta.

  ![Flujo iniciado por el proveedor de servicios de SAML](./media/application-proxy-configure-single-sign-on-on-premises-apps/saml-sp-initiated-flow.png)

  ![Flujo iniciado por el proveedor de servicios de SAML](./media/application-proxy-configure-single-sign-on-on-premises-apps/saml-idp-initiated-flow.png)

## <a name="create-an-application-and-set-up-saml-sso"></a>Creación de una aplicación y configuración del inicio de sesión único de SAML

1. En Azure Portal, seleccione **Azure Active Directory > Aplicaciones empresariales** y seleccione **Nueva aplicación**.

2. Escriba el nombre para mostrar de la nueva aplicación, seleccione **Integrar cualquier otra aplicación que no se encuentre en la galería** y, después, seleccione **Crear**.

3. En la página **Información general** de la aplicación, seleccione **Inicio de sesión único**.

4. Seleccione **SAML** como método de inicio de sesión único.

5. En primer lugar, configure el inicio de sesión único de SAML para que funcione en la red corporativa, consulte la sección configuración básica de SAML de [Configuración del inicio de sesión único basado en SAML](configure-saml-single-sign-on.md) para configurar la autenticación basada en SAML para la aplicación.

6. Agregue al menos un usuario a la aplicación y asegúrese de que la cuenta de prueba tenga acceso a la aplicación. Mientras esté conectado a la red corporativa, utilice la cuenta de prueba para ver si tiene inicio de sesión único en la aplicación. 

   > [!NOTE]
   > Después de configurar Application Proxy, volverá y actualizará la **dirección URL de respuesta** de SAML.

## <a name="publish-the-on-premises-application-with-application-proxy"></a>Publicación de la aplicación en el entorno local con Application Proxy

Para proporcionar SSO para aplicaciones en el entorno local, tiene que habilitar Application Proxy e instalar un conector. Consulte el tutorial [Adición de una aplicación local para el acceso remoto mediante Application Proxy en Azure Active Directory](application-proxy-add-on-premises-application.md) para más información sobre cómo preparar el entorno local, instalar y registrar un conector, y probarlo. A continuación, siga estos pasos para publicar la nueva aplicación con Application Proxy. Para otras opciones de configuración que no se mencionan a continuación, consulte la sección [Adición de una aplicación local a Azure AD](application-proxy-add-on-premises-application.md#add-an-on-premises-app-to-azure-ad) del tutorial.

1. Con la aplicación todavía abierta en Azure Portal, seleccione **Application Proxy**. Proporcione la **dirección URL interna** para la aplicación. Si utiliza un dominio personalizado, también debe cargar el certificado TLS/SSL para la aplicación. 
   > [!NOTE]
   > Como procedimiento recomendado, use dominios personalizados siempre que sea posible para obtener la mejor experiencia de usuario. Más información acerca del [Uso de dominios personalizados en Azure AD Application Proxy](application-proxy-configure-custom-domain.md).

2. Seleccione **Azure Active Directory** como método de **autenticación previa** para la aplicación.

3. Copie la **dirección URL externa** para la aplicación. Necesitará esta dirección URL para completar la configuración de SAML.

4. Mediante la cuenta de prueba, puede probar a abrir la aplicación con la **dirección URL externa** para comprobar que Application Proxy se ha configurado correctamente. Si hay problemas, consulte [Solución de problemas y mensajes de error de Application Proxy](application-proxy-troubleshoot.md).

## <a name="update-the-saml-configuration"></a>Actualización de la configuración de SAML

1. Con la aplicación todavía abierta en Azure Portal, seleccione **Inicio de sesión único**. 

2. En la página **Configurar el inicio de sesión único con SAML**, vaya al encabezado **Configuración básica de SAML** y seleccione el icono de **edición** (un lápiz). Asegúrese de que la **dirección URL externa** que configuró en Application Proxy rellenará los campos **Identificador**, **URL de respuesta** y **URL de cierre de sesión**. Estas direcciones URL son necesarias para que Application Proxy funcione correctamente. 

3. Edite la **Dirección URL de respuesta** configurada anteriormente para que Application Proxy pueda acceder a su dominio a través de Internet. Por ejemplo, si la **dirección URL externa** es `https://contosotravel-f128.msappproxy.net` y la **URL de respuesta** original era `https://contosotravel.com/acs`, deberá actualizar la **URL de respuesta** original a `https://contosotravel-f128.msappproxy.net/acs`.

    ![Especificación de la configuración básica de SAML](./media/application-proxy-configure-single-sign-on-on-premises-apps/basic-saml-configuration.png)


4. Seleccione la casilla situada junto a la **URL de respuesta** actualizada para marcarla como predeterminada.

   * Después de marcar la **Dirección URL de respuesta** como el valor predeterminado, también puede eliminar la **Dirección URL de respuesta** configurada anteriormente que usó la dirección URL interna.

   * Con un flujo iniciado por el proveedor de servicios, asegúrese de que la aplicación de back-end especifica el valor de **URL de respuesta** correcto o la URL del Servicio de consumidor de aserciones que se utilizará para recibir el token de autenticación.

    > [!NOTE]
    > Si la aplicación de back-end espera que la **URL de respuesta** sea la dirección URL interna, necesitará usar [dominios personalizados](application-proxy-configure-custom-domain.md) para tener direcciones URL internas y externas que coincidan o instalar la extensión de inicio de sesión segura de Mis aplicaciones en los dispositivos de los usuarios. Esta extensión le redireccionará automáticamente al servicio Application Proxy adecuado. Para instalar la extensión, consulte [	Extensión de inicio de sesión seguro de mis aplicaciones](../user-help/my-apps-portal-end-user-access.md#download-and-install-the-my-apps-secure-sign-in-extension).
    
## <a name="test-your-app"></a>Prueba de la aplicación

Cuando haya completado todos los pasos, la aplicación debería estar en funcionamiento. Para probar la aplicación:

1. Abra un explorador y vaya a la **dirección URL externa** que creó al publicar la aplicación. 
1. Inicie sesión con la cuenta de prueba que asignó a la aplicación. Debe ser capaz de cargar la aplicación y tener el inicio de sesión único en la aplicación.

## <a name="next-steps"></a>Pasos siguientes

- [¿Cómo permite el proxy de aplicación de Azure AD el inicio de sesión único?](application-proxy-single-sign-on.md)
- [Solucionar problemas de Proxy de aplicación](application-proxy-troubleshoot.md)
