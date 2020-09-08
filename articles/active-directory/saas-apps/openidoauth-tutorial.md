---
title: Configuración de una aplicación OpenID/OAuth desde la galería de aplicaciones de Azure AD | Microsoft Docs
description: Pasos para configurar una aplicación OpenID/OAuth desde la galería de aplicaciones de Azure AD.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/30/2019
ms.author: jeedes
ms.custom: has-adal-ref
ms.openlocfilehash: 1934b6256ecf4f35c54bbc2ac497c331b2c5ee89
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88543931"
---
# <a name="configure-an-openidoauth-application-from-the-azure-ad-app-gallery"></a>Configuración de una aplicación OpenID/OAuth desde la galería de aplicaciones de Azure AD

## <a name="process-of-adding-an-openid-application-from-the-gallery"></a>Proceso de adición de una aplicación OpenID desde la galería

1. En el panel izquierdo de [Azure Portal](https://portal.azure.com), seleccione **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png))

2. Vaya a **Aplicaciones empresariales** > **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Seleccione **Nueva aplicación** en la parte superior del cuadro de diálogo.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba el nombre de la aplicación. Seleccione la aplicación que desee en el panel de resultados y regístrese en la aplicación.

    ![Openid en la lista de resultados](common/search-new-app.png)

    > [!NOTE]
    > Para las aplicaciones OpenID Connect y OAuth, el botón **Agregar** está deshabilitado de forma predeterminada. Aquí, el administrador de inquilinos debe seleccionar el botón de registro y proporcionar el consentimiento a la aplicación. La aplicación se agrega entonces al inquilino del cliente, donde puede configurar lo que desee. No es necesario agregar la aplicación de forma explícita.

    ![Botón Agregar](./media/openidoauth-tutorial/addbutton.png)

5. Cuando seleccione el vínculo de registro, se le redirigirá a la página de Azure Active Directory (Azure AD) para que indique las credenciales de inicio de sesión.

6. Una vez autenticado correctamente, acepte el consentimiento de la página de consentimiento. A continuación, aparece la página de inicio de la aplicación.

    > [!NOTE]
    > Puede agregar solo una instancia de la aplicación. Si ya ha agregado una y ha intentado proporcionar el consentimiento de nuevo, no se agregará otra vez en el inquilino. Por tanto, como es lógico, solo puede usar una instancia de la aplicación en el inquilino.

## <a name="authentication-flow-using-openid-connect"></a>Flujo de autenticación con OpenID Connect

El flujo de inicio de sesión más básico contiene los siguientes pasos:

![Flujo de autenticación con OpenID Connect](./media/openidoauth-tutorial/authenticationflow.png)

### <a name="multitenant-application"></a>Aplicación multiinquilino
El uso de una aplicación multiinquilino se destina a varias organizaciones, no solo a una. Suele tratarse de aplicaciones de software como servicio (SaaS) escritas por un proveedor de software independiente (ISV).

Las aplicaciones multiinquilino se deben aprovisionar en cada directorio en el que se van a utilizar. Requieren el consentimiento del usuario o administrador para su registro. El proceso de consentimiento se inicia cuando una aplicación se ha registrado en el directorio y se le proporciona acceso a Graph API o quizás a otra API web. Cuando un usuario o un administrador de otra organización se registra para usar la aplicación, un cuadro de diálogo indica los permisos requeridos por la aplicación.

El usuario o administrador puede entonces dar su consentimiento a la aplicación. El consentimiento da acceso a la aplicación a los datos indicados y, por último, registra la aplicación en el directorio.

> [!NOTE]
> Si está poniendo la aplicación a disposición de usuarios de varios directorios, necesitará un mecanismo para determinar en qué inquilino se encuentran. Para buscar un usuario, una aplicación de un único inquilino solo tiene que buscar en su propio directorio. Una aplicación multiinquilino debe identificar un usuario específico en todos los directorios de Azure AD.
>
> Para realizar esta tarea, Azure AD proporciona un punto de conexión de autenticación común donde cualquier aplicación multiinquilino puede dirigir solicitudes de inicio de sesión, en lugar de un punto de conexión específico del inquilino. Este punto de conexión es `https://login.microsoftonline.com/common` para todos los directorios de Azure AD. Un punto de conexión específico del inquilino podría ser `https://login.microsoftonline.com/contoso.onmicrosoft.com`.
>
> Es importante considerar el punto de conexión común a la hora de desarrollar su aplicación. Requerirá la lógica necesaria para administrar varios inquilinos durante el inicio de sesión, el cierre de sesión y la validación de tokens.

De forma predeterminada, Azure AD promueve las aplicaciones multiinquilino. Es fácil acceder a ellas entre organizaciones, y son sencillas de usar después de aceptar el consentimiento.

## <a name="consent-framework"></a>Marco de consentimiento

Puede usar el marco de consentimiento de Azure AD para desarrollar aplicaciones web multiinquilino y aplicaciones de cliente nativo. Estas aplicaciones permiten el inicio de sesión por cuentas de usuario de un inquilino de Azure AD diferente al que ha registrado la aplicación. Es posible que también necesiten acceder a las API web, como:
- Microsoft Graph API, para acceder a Azure AD, Intune y servicios en Office 365.
- Otras API de servicios de Microsoft.
- Sus propias API web.

El marco se basa en que un usuario o un administrador da consentimiento a una aplicación que solicita el registro en su directorio. El registro podría implicar el acceso a datos del directorio. Después de dar su consentimiento, la aplicación cliente puede llamar a Microsoft Graph API en nombre del usuario y usar la información según sea necesario.

[Microsoft Graph API](https://developer.microsoft.com/graph/) proporciona acceso a datos de Office 365, como:

- Calendarios y mensajes de Exchange.
- Sitios y listas de SharePoint.
- Documentos de OneDrive.
- Blocs de notas de OneNote.
- Tareas de Planner.
- Libros de Excel.

Graph API también proporciona acceso a usuarios y grupos desde Azure AD y otros objetos de datos desde más servicios en la nube de Microsoft.

Los siguientes pasos muestran cómo funciona la experiencia de consentimiento para el desarrollador de la aplicación y para el usuario:

1. Suponga que tiene una aplicación de cliente web que necesita solicitar permisos concretos para tener acceso a un recurso o API. Azure Portal se utiliza para declarar las solicitudes de permiso en el momento de la configuración. Al igual que otras opciones de configuración, pasan a formar parte del registro de Azure AD de la aplicación. Para la ruta de acceso de solicitud de permiso, siga los pasos que se indican a continuación:

    a. Haga clic en **Registros de aplicaciones** en el lado izquierdo del menú y abra la aplicación escribiendo el nombre de la aplicación en el cuadro de búsqueda.

    ![Graph API](./media/openidoauth-tutorial/application.png)

    b. Haga clic en **Ver permisos de API**.

    ![Graph API](./media/openidoauth-tutorial/api-permission.png)

    c. Haga clic en **Agregar un permiso**.

    ![Graph API](./media/openidoauth-tutorial/add-permission.png)

    d. Haga clic en **Microsoft Graph**.

    ![Graph API](./media/openidoauth-tutorial/microsoft-graph.png)

    e. Seleccione las opciones necesarias de **Permisos delegados** y **Permisos de la aplicación**.

    ![Graph API](./media/openidoauth-tutorial/graphapi.png)

2. Imagine que se han actualizado los permisos de la aplicación. La aplicación se está ejecutando y un usuario está a punto de usarla por primera vez. En primer lugar, la aplicación debe obtener un código de autorización del punto de conexión /authorize de Azure AD. El código de autorización, a continuación, se puede utilizar para adquirir un nuevo acceso y un token de actualización.

3. Si el usuario no está aún autenticado, el punto de conexión /authorize de Azure AD solicita el inicio de sesión.

    ![Authentication](./media/openidoauth-tutorial/authentication.png)

4. Cuando el usuario inicia sesión, Azure AD determina si se debe mostrar una página de consentimiento al usuario. Esta determinación se basa en que el usuario (o el administrador de la organización) ya concediese el consentimiento de la aplicación.

   Si no se ha concedido el consentimiento, Azure AD se lo pide al usuario y muestra los permisos necesarios para que funcione. Los permisos que aparecen en el cuadro de diálogo de consentimiento coinciden con los que se seleccionaron en los permisos delegados de Azure Portal.

    ![Página de consentimiento](./media/openidoauth-tutorial/consentpage.png)

Un usuario normal puede dar su consentimiento a algunos permisos. Otros permisos requieren el consentimiento de un administrador del inquilino.

## <a name="difference-between-admin-consent-and-user-consent"></a>Diferencia entre consentimiento del administrador y consentimiento del usuario

Como administrador, también puede dar su consentimiento para permisos delegados de una aplicación en nombre de todos los usuarios del inquilino. Con el consentimiento del administrador se evita que el cuadro de diálogo de consentimiento aparezca para cada usuario del inquilino. Los usuarios que tengan el rol de administrador pueden dar su consentimiento en Azure Portal. En la página **Configuración** de la aplicación, seleccione **Permisos necesarios** > **Conceder consentimiento de administrador**.

![Botón Conceder permisos](./media/openidoauth-tutorial/grantpermission.png)

> [!NOTE]
> Ahora es obligatorio conceder explícitamente el consentimiento con el botón **Conceder consentimiento de administrador** para las aplicaciones de página única (SPA) que usan ADAL.js. En caso contrario, se produce un error en la aplicación cuando se solicita el token de acceso.

Los permisos de solo aplicación siempre requieren el consentimiento de un administrador del inquilino. Si la aplicación solicita un permiso de solo aplicación y un usuario intenta iniciar sesión en la aplicación, aparece un mensaje de error. El mensaje indica que el usuario no puede dar su consentimiento.

Si la aplicación usa permisos que requieren el consentimiento del administrador, necesita tener un gesto, como un botón o un vínculo donde el administrador pueda iniciar la acción. La solicitud que la aplicación envía para esta acción es la solicitud de autorización habitual de OAuth2 o OpenID Connect. Esta solicitud incluye el parámetro de cadena de consulta *prompt=admin_consent*.

Una vez que el administrador ha dado su consentimiento y se crea la entidad de servicio en el inquilino del cliente, las posteriores solicitudes de inicio de sesión no necesitan el parámetro *prompt=admin_consent*. Dado que el administrador ha decido que los permisos solicitados son aceptables, en adelante no se solicitará consentimiento a ningún otro usuario.

Un administrador de inquilinos puede deshabilitar la posibilidad de que los usuarios normales den su consentimiento a las aplicaciones. Si esta capacidad está deshabilitada, para que la aplicación se use en el inquilino siempre se solicitará el consentimiento del administrador. Si quiere probar la aplicación con el consentimiento de usuario final deshabilitado, puede encontrar el modificador de configuración en [Azure Portal](https://portal.azure.com/). Específicamente, en la sección [Configuración de usuario](https://portal.azure.com/#blade/Microsoft_AAD_IAM/StartboardApplicationsMenuBlade/UserSettings/menuId/) en **Aplicaciones empresariales**.

El parámetro *prompt=admin_consent* también se puede utilizar en las aplicaciones que solicitan permisos que no requieren el consentimiento del administrador. Por ejemplo, una aplicación que requiere una experiencia en la que el administrador del inquilino se "registra" una vez y no se solicita a otros usuarios que den su consentimiento a partir de entonces.

Imagine que una aplicación requiere el consentimiento del administrador y un administrador inicia sesión sin que se envíe el parámetro *prompt=admin_consent*. Cuando el administrador dé correctamente su consentimiento a la aplicación, solo se aplicará para su cuenta de usuario. Los usuarios normales seguirán sin poder iniciar sesión ni dar su consentimiento a la aplicación. Esta característica resulta útil si desea brindar al administrador de inquilinos la posibilidad de explorar su aplicación antes de permitir el acceso a otros usuarios.
