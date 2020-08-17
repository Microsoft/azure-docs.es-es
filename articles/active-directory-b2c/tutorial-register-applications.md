---
title: 'Tutorial: Registro de una aplicación'
titleSuffix: Azure AD B2C
description: Aprenda a registrar una aplicación web en Azure Active Directory B2C con Azure Portal.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: tutorial
ms.date: 04/10/2020
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 0f3ecb918aa1f830edeafbe57b365441e1e534f4
ms.sourcegitcommit: d7bd8f23ff51244636e31240dc7e689f138c31f0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87169342"
---
# <a name="tutorial-register-a-web-application-in-azure-active-directory-b2c"></a>Tutorial: Registro de una aplicación web en Azure Active Directory B2C

Para que sus [aplicaciones](application-types.md) puedan interactuar con Azure Active Directory B2C (Azure AD B2C), deben estar registradas en un inquilino que administre. En este tutorial se muestra cómo registrar una aplicación web mediante Azure Portal.

En este artículo aprenderá a:

> [!div class="checklist"]
> * Registro de una aplicación web
> * Creación de un secreto de cliente

Si usa una aplicación nativa (por ejemplo, iOS, Android, móvil y de escritorio), aprenda a [registrar una aplicación cliente nativa](add-native-application.md).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

Si aún no ha creado su propio [inquilino de Azure AD B2C](tutorial-create-tenant.md), cree una ahora. Puede usar un inquilino de Azure AD B2C existente.

## <a name="register-a-web-application"></a>Registro de una aplicación web

Para registrar una aplicación en su inquilino de Azure AD B2C, puede usar la nueva experiencia unificada **Registros de aplicaciones**, o bien la experiencia anterior **Aplicaciones (heredado)** . [Más información acerca de la nueva experiencia](https://aka.ms/b2cappregtraining).

#### <a name="app-registrations"></a>[Registros de aplicaciones](#tab/app-reg-ga/)

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione el icono **Directorio y suscripción** en la barra de herramientas del portal y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.
1. En Azure Portal, busque y seleccione **Azure AD B2C**.
1. Seleccione **Registros de aplicaciones** y luego **Nuevo registro**.
1. Escriba un **Nombre** para la aplicación. Por ejemplo, *webapp1*.
1. En **Tipos de cuenta compatibles**, seleccione **Cuentas de cualquier directorio de la organización o cualquier proveedor de identidades. Para autenticar a los usuarios con Azure AD B2C**.
1. En **URI de redirección**, seleccione **Web** y escriba `https://jwt.ms` en el cuadro de texto.

    El URI de redirección es el punto de conexión al que el servidor de autorización envía al usuario (Azure AD B2C, en este caso) después de completar su interacción con este, y al que se envía un token de acceso o un código de autorización tras una autorización correcta. En una aplicación de producción, suele ser un punto de conexión accesible públicamente donde se ejecuta la aplicación, como `https://contoso.com/auth-response`. Con fines de prueba como este tutorial, puede establecerlo en `https://jwt.ms`, una aplicación web propiedad de Microsoft que muestra el contenido descodificado de un token (el contenido del token nunca sale del explorador). Durante el desarrollo de aplicaciones, puede agregar el punto de conexión en el que la aplicación realiza escuchas localmente, como `https://localhost:5000`. Puede agregar y modificar los URI de redireccionamiento en las aplicaciones registradas en cualquier momento.

    Las siguientes restricciones se aplican a los URI de redireccionamiento:

    * La dirección URL de respuesta debe comenzar con el esquema `https`.
    * La dirección URL de respuesta distingue mayúsculas de minúsculas. Sus mayúsculas o minúsculas deben coincidir con las de la ruta de acceso de la dirección URL de la aplicación en ejecución. Por ejemplo, si la aplicación incluye como parte de su ruta de acceso `.../abc/response-oidc`, no especifique `.../ABC/response-oidc` en la dirección URL de respuesta. Dado que el explorador web tiene en cuenta las mayúsculas y minúsculas de la ruta de acceso, se pueden excluir las cookies asociadas con `.../abc/response-oidc` si se redirigen a la dirección URL `.../ABC/response-oidc` con mayúsculas y minúsculas no coincidentes.

1. En **Permisos**, active la casilla *Conceda consentimiento del administrador a los permisos openid y offline_access*.
1. Seleccione **Registrar**.

Una vez completado el registro de la aplicación, habilite el flujo de concesión implícita:

1. En el menú izquierdo, en **Administrar**, seleccione **Autenticación**.
1. En **Concesión implícita**, active las casillas **Tokens de acceso** y **Tokens de id.**
1. Seleccione **Guardar**.

#### <a name="applications-legacy"></a>[Aplicaciones (heredado)](#tab/applications-legacy/)

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione el icono **Directorio y suscripción** en la barra de herramientas del portal y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.
1. En Azure Portal, busque y seleccione **Azure AD B2C**.
1. Seleccione **Aplicaciones (heredado)** y, luego, **Agregar**.
1. Escriba un nombre para la aplicación. Por ejemplo, *webapp1*.
1. En **Incluir aplicación web o API web** y **Permitir flujo implícito**, seleccione **Sí**.
1. En **Dirección URL de respuesta**, escriba un punto de conexión donde Azure AD B2C devolverá los tokens que solicite la aplicación. Por ejemplo, se puede establecer para la escucha local en `https://localhost:44316`. Si aún no conoce el número de puerto, puede especificar un valor de marcador de posición y cambiarlo más adelante.

    Con fines de prueba (como los de este tutorial) puede establecerlo en `https://jwt.ms`, que muestra el contenido de un token para su inspección. Para este tutorial, establezca la **URL de respuesta** en `https://jwt.ms`.

    Las siguientes restricciones se aplican a las direcciones URL de respuesta:

    * La dirección URL de respuesta debe comenzar con el esquema `https`.
    * La dirección URL de respuesta distingue mayúsculas de minúsculas. Sus mayúsculas o minúsculas deben coincidir con las de la ruta de acceso de la dirección URL de la aplicación en ejecución. Por ejemplo, si la aplicación incluye como parte de su ruta de acceso `.../abc/response-oidc`, no especifique `.../ABC/response-oidc` en la dirección URL de respuesta. Dado que el explorador web tiene en cuenta las mayúsculas y minúsculas de la ruta de acceso, se pueden excluir las cookies asociadas con `.../abc/response-oidc` si se redirigen a la dirección URL `.../ABC/response-oidc` con mayúsculas y minúsculas no coincidentes.

1. Seleccione **Crear** para completar el registro de la aplicación.

* * *

## <a name="create-a-client-secret"></a>Creación de un secreto de cliente

Si la aplicación intercambia un código de autorización para un token de acceso, debe crear un secreto de aplicación.


#### <a name="app-registrations"></a>[Registros de aplicaciones](#tab/app-reg-ga/)

1. En la página **Azure AD B2C: Registros de aplicaciones**, seleccione la aplicación que ha creado, por ejemplo, *webapp1*.
1. En el menú de la izquierda, en **Administrar**, seleccione **Certificados y secretos**.
1. Seleccione **Nuevo secreto de cliente**.
1. Escriba una descripción para el secreto de cliente en el cuadro **Descripción**. Por ejemplo, *clientsecret1*.
1. En **Expira**, seleccione el tiempo durante el cual el secreto es válido y, a continuación, seleccione **Agregar**.
1. Registre el **Valor** del secreto. Use este valor como secreto de aplicación en el código de la aplicación.

#### <a name="applications-legacy"></a>[Aplicaciones (heredado)](#tab/applications-legacy/)

1. En la página de **aplicaciones de Azure AD B2C**, seleccione la aplicación que ha creado, por ejemplo, *webapp1*.
1. Seleccione **Claves** y haga clic en **Generar clave**.
1. Seleccione **Guardar** para ver la clave. Anote el valor de la **Clave de la aplicación**. Use este valor como secreto de aplicación en el código de la aplicación.

* * *

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido cómo:

> [!div class="checklist"]
> * Registro de una aplicación web
> * Creación de un secreto de cliente

A continuación, aprenda a crear flujos de usuario para permitir que los usuarios se registren, inicien sesión y administren sus perfiles.

> [!div class="nextstepaction"]
> [Creación de flujos de usuario en Azure Active Directory B2C >](tutorial-create-user-flows.md)
