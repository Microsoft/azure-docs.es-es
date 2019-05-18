---
title: Modificación de las cuentas admitidas por una aplicación registrada con la plataforma de identidad de Microsoft | Azure
description: Configure una aplicación registrada en la plataforma de identidad de Microsoft para cambiar quién o qué cuentas pueden acceder a la aplicación.
services: active-directory
documentationcenter: ''
author: rwike77
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/08/2019
ms.author: ryanwi
ms.custom: aaddev
ms.reviewer: aragra, lenalepa, sureshja
ms.collection: M365-identity-device-management
ms.openlocfilehash: e2c52bd0bf6c1e385248bd379a8001756d28ac2d
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2019
ms.locfileid: "65413582"
---
# <a name="quickstart-modify-the-accounts-supported-by-an-application"></a>Inicio rápido: Modificación de las cuentas compatibles con una aplicación

Al registrar una aplicación en la plataforma de identidad de Microsoft, tal vez quiera que solo tengan acceso a la aplicación los usuarios de la organización. Como alternativa, también puede desear que la aplicación sea accesible para los usuarios de organizaciones externas, así como para los usuarios que no formen necesariamente parte de una organización (cuentas personales).

En esta guía de inicio rápido, aprenderá a modificar la configuración de la aplicación para cambiar quién o qué cuentas pueden acceder a la aplicación.

## <a name="prerequisites"></a>Requisitos previos

Para comenzar, asegúrese de que completa estos requisitos previos:

* Aprenda sobre los [permisos y consentimiento](v2-permissions-and-consent.md) admitidos, que son importantes comprender al crear aplicaciones que necesitan ser utilizadas por otros usuarios o aplicaciones.
* Tiene un inquilino que tenga las aplicaciones registradas en él.
  * Si no tiene aplicaciones registradas, [aprenda a registrar aplicaciones con la plataforma de identidad de Microsoft](quickstart-register-app.md).

## <a name="sign-in-to-the-azure-portal-and-select-the-app"></a>Inicio de sesión en Azure Portal y selección de la aplicación

Antes de configurar la aplicación, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. Si la cuenta proporciona acceso a más de un inquilino, haga clic en la cuenta en la esquina superior derecha y establezca la sesión del portal en el inquilino de Azure AD deseado.
1. En el panel de navegación izquierdo, seleccione el servicio **Azure Active Directory** y, después, seleccione **Registros de aplicaciones**.
1. Busque y seleccione la aplicación que desea configurar. Cuando haya seleccionado la aplicación, verá la página **Introducción** de la aplicación o la de registro principal.
1. Siga los pasos para [cambiar el registro de la aplicación para admitir distintas cuentas](#change-the-application-registration-to-support-different-accounts).
1. Si tiene una aplicación de página única, [habilite la concesión implícita de OAuth 2.0](#enable-oauth-20-implicit-grant-for-single-page-applications).

## <a name="change-the-application-registration-to-support-different-accounts"></a>Cambio del registro de la aplicación para admitir distintas cuentas

Si está escribiendo una aplicación que quiere poner a disposición de clientes o asociados externos a la organización, debe actualizar la definición de la aplicación en Azure Portal.

> [!IMPORTANT]
> Azure AD necesita que el URI del identificador de las aplicaciones multiinquilino sea único globalmente. El URI de id. de aplicación es una de las maneras en que una aplicación se identifica en los mensajes de protocolo. Cuando la aplicación es de un solo inquilino, es suficiente con que el URI de identificador de aplicación sea único en dicho inquilino. En el caso de una aplicación multiempresa, debe ser único a nivel global de forma que Azure AD pueda encontrar la aplicación entre todos los inquilinos. El carácter globalmente único viene impuesto por la necesidad de que el URI de id. de aplicación tenga un nombre de host que coincida con un dominio comprobado del inquilino de Azure AD. Por ejemplo, si el nombre del inquilino es contoso.onmicrosoft.com, un identificador URI de identificador de aplicación válido sería https://contoso.onmicrosoft.com/myapp. Si el inquilino tiene el dominio comprobado contoso.com, un URI del identificador de aplicación válido también sería https://contoso.com/myapp. Si el URI de id. de aplicación no sigue este patrón, la configuración de una aplicación como multiinquilino dará error.

### <a name="to-change-who-can-access-your-application"></a>Para cambiar quién puede acceder a la aplicación

1. Desde la página **Introducción** de la aplicación, seleccione la sección **Autenticación** y cambie el valor seleccionado en **Tipos de cuentas compatibles**.
    * Seleccione **Accounts in this directory only** (Solo las cuentas de este directorio) si va a crear una aplicación de línea de negocio (LOB). Esta opción no está disponible si la aplicación no está registrada en un directorio.
    * Seleccione **Cuentas en cualquier directorio organizativo** si desea tener como destino todos los clientes de negocios y del sector educativo.
    * Seleccione **Cuentas en cualquier directorio organizativo y cuentas Microsoft personales** para establecer como destino el mayor conjunto posible de clientes.
1. Seleccione **Guardar**.

## <a name="enable-oauth-20-implicit-grant-for-single-page-applications"></a>Habilitación de la concesión implícita de OAuth 2.0 en aplicaciones de página única

Las aplicaciones de página única (SPA) normalmente tienen una estructura con un front-end de JavaScript que se ejecuta en el explorador, que llama al back-end de la API web de la aplicación para llevar a cabo la lógica de negocios. Para las SPA hospedadas en Azure AD, se usa la concesión implícita de OAuth 2.0 para autenticar al usuario en Azure AD y obtener un token que puede usar para proteger las llamadas desde el cliente JavaScript de la aplicación hasta la API web de back-end.

Después de que el usuario haya dado su consentimiento, este mismo protocolo de autenticación se puede usar para obtener tokens para proteger las llamadas entre el cliente y otros recursos de API web configurados para la aplicación. Para saber más sobre la concesión de autorización implícita y ayudarle a decidir si es adecuada para el escenario de su aplicación, consulte el flujo de concesión implícita de OAuth 2.0 en Azure AD [v1.0](v1-oauth2-implicit-grant-flow.md) y [v2.0](v2-oauth2-implicit-grant-flow.md).

De forma predeterminada, la concesión implícita de OAuth 2.0 está deshabilitada para las aplicaciones. Para habilitar la concesión implícita de OAuth 2.0 para la aplicación, siga los pasos descritos a continuación.

### <a name="to-enable-oauth-20-implicit-grant"></a>Para habilitar la concesión implícita de OAuth 2.0

1. Desde la página **Introducción** de la aplicación, seleccione la sección **Autenticación**.
1. En **Configuración avanzada**, busque la sección **Concesión implícita**.
1. Seleccione **Tokens de identificador**, **Tokens de acceso** o ambos.
1. Seleccione **Guardar**.

## <a name="next-steps"></a>Pasos siguientes

Obtenga información acerca de estas otras guías de inicio rápido de administración de aplicaciones relacionadas para aplicaciones:

* [Registro de una aplicación con la plataforma de identidad de Microsoft](quickstart-register-app.md)
* [Configuración de una aplicación cliente para tener acceso a las API web](quickstart-configure-app-access-web-apis.md)
* [Configuración de una aplicación para exponer las API web](quickstart-configure-app-expose-web-apis.md)
* [Eliminación de una aplicación registrada con la plataforma de identidad de Microsoft](quickstart-remove-app.md)

Para obtener más información acerca de los dos objetos de Azure AD que representan una aplicación registrada y la relación entre ellos, consulte [Objetos de aplicación y de entidad de servicio](app-objects-and-service-principals.md).

Para obtener más información acerca de las directrices de personalización de marca que hay que usar al desarrollar aplicaciones con Azure Active Directory, consulte [Directrices de personalización de marca para aplicaciones](howto-add-branding-in-azure-ad-apps.md).
