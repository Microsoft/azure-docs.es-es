---
title: 'Inicio rápido: Configuración de una aplicación para acceder a una API web | Azure'
titleSuffix: Microsoft identity platform
description: En este inicio rápido, configurará un registro de aplicación que representa una API web en la plataforma de identidad de Microsoft para que las aplicaciones cliente tengan acceso a los recursos con ámbito (permisos).
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: quickstart
ms.workload: identity
ms.date: 09/03/2020
ms.author: marsma
ms.custom: aaddev, contperfq1
ms.reviewer: lenalepa, aragra, sureshja
ms.openlocfilehash: fc2f3202ac88e3ee6c24db21dd9072a13a8deef9
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89442314"
---
# <a name="quickstart-configure-a-client-application-to-access-a-web-api"></a>Inicio rápido: Configuración de una aplicación cliente para acceder a las API web

En este inicio rápido proporcionará a una aplicación cliente registrada en la plataforma de identidad de Microsoft acceso basado en permisos con ámbito a una API web propia. También proporcionará a la aplicación cliente acceso a Microsoft Graph.

Al especificar los ámbitos de una API web en el registro de la aplicación cliente, esta puede obtener un token de acceso que contenga esos ámbitos de la plataforma de identidad de Microsoft. Dentro de su código, la API web puede proporcionar acceso basado en permisos a sus recursos en función de los ámbitos que se encuentran en el token de acceso.

## <a name="prerequisites"></a>Requisitos previos

* Una cuenta de Azure con una suscripción activa: [cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Finalización de [Inicio rápido: Registro de una aplicación](quickstart-register-app.md).
* Finalización de [Inicio rápido: Configuración de una aplicación para exponer una API web](quickstart-configure-app-expose-web-apis.md).

## <a name="add-permissions-to-access-your-web-api"></a>Incorporación de permisos para acceder a la API web

En el primer escenario, se concede acceso a la aplicación cliente a la API web propia; ambas deben haberse registrado como parte de los requisitos previos. Si aún no tiene registrada una aplicación cliente y una API web, complete los pasos descritos en los dos artículos de [Requisitos previos](#prerequisites).

En este diagrama se muestra cómo se relacionan entre sí los dos registros de aplicaciones. En esta sección se agregan permisos al registro de la aplicación cliente.

:::image type="content" source="media/quickstart-configure-app-access-web-apis/diagram-01-app-permission-to-api-scopes.svg" alt-text="Diagrama de líneas que muestra una API web con ámbitos expuestos a la derecha y una aplicación cliente a la izquierda con los ámbitos seleccionados como permisos" border="false":::

Una vez que haya registrado la aplicación cliente y la API web, y haya expuesto la API mediante la creación de ámbitos, puede configurar los permisos del cliente para la API siguiendo estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Si tiene acceso a varios inquilinos, use el filtro **Directorio + suscripción** :::image type="icon" source="./media/quickstart-configure-app-access-web-apis/portal-01-directory-subscription-filter.png" border="false"::: del menú superior para seleccionar el inquilino que contiene el registro de la aplicación cliente.
1. Seleccione **Azure Active Directory** > **Registros de aplicaciones** y la aplicación cliente (*no* la API web).
1. Seleccione **Permisos de API** > **Agregar permiso** > **Mis API**.
1. Seleccione la API web registrada como parte de los requisitos previos.

    La opción seleccionada de forma predeterminada es **Permisos delegados**. Los permisos delegados son adecuados para las aplicaciones cliente que acceden a una API web como usuario con sesión iniciada y cuyo acceso debe limitarse a los permisos que seleccione en el paso siguiente. Deje **Permisos delegados** seleccionado en este ejemplo.

    Los **Permisos de aplicación** son para las aplicaciones de tipo servicio o demonio que necesitan acceder a una API web por sí mismas, sin la interacción del usuario para el inicio de sesión o el consentimiento. A menos que haya definido los roles de aplicación para la API web, esta opción está deshabilitada.
1. En **Seleccionar permisos**, expanda el recurso cuyos ámbitos definió para la API web y seleccione los permisos que debe tener la aplicación cliente en nombre del usuario cuya sesión se ha iniciado.

    Si ha usado los nombres de ámbito de ejemplo especificados en el inicio rápido anterior, debería ver **Employees.Read.All** y **Employees.Write.All**.
    Seleccione **Employees.Read.All** u otro permiso que haya creado al completar los requisitos previos.
1. Seleccione **Agregar permisos** para completar el proceso.

Después de agregar permisos a la API, debería ver los permisos seleccionados en **Permisos configurados**. En la imagen siguiente se muestra el permiso delegado *Employees.Read.All* de ejemplo agregado al registro de la aplicación cliente.

:::image type="content" source="media/quickstart-configure-app-access-web-apis/portal-02-configured-permissions-pane.png" alt-text="Panel de permisos configurados en Azure Portal que muestra el permiso recién agregado":::

También puede observar el permiso *User.Read* para Microsoft Graph API. Este permiso se agrega automáticamente al registrar una aplicación en Azure Portal.

## <a name="add-permissions-to-access-microsoft-graph"></a>Incorporación de permisos para acceder a Microsoft Graph

Además de acceder a su API web propia en nombre del usuario que ha iniciado sesión, es posible que la aplicación también necesite acceder a los datos del usuario (u otros) almacenados en Microsoft Graph o modificarlos. O bien, puede que tenga una aplicación de servicio o demonio que necesite acceder a Microsoft Graph por sí misma y realizar operaciones sin la interacción del usuario.

### <a name="delegated-permission-to-microsoft-graph"></a>Permiso delegado para Microsoft Graph

Configure el permiso delegado a Microsoft Graph para permitir que la aplicación cliente realice operaciones en nombre del usuario que ha iniciado sesión, por ejemplo, leer el correo electrónico o modificar su perfil. De forma predeterminada, cuando los usuarios de la aplicación cliente inician sesión, se les pide su consentimiento para los permisos delegados configurados para ella.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Si tiene acceso a varios inquilinos, use el filtro **Directorio + suscripción** :::image type="icon" source="./media/quickstart-configure-app-access-web-apis/portal-01-directory-subscription-filter.png" border="false"::: del menú superior para seleccionar el inquilino que contiene el registro de la aplicación cliente.
1. Seleccione **Azure Active Directory** > **Registros de aplicaciones** y la aplicación cliente.
1. Seleccione **Permisos de API** > **Agregar un permiso** > **Microsoft Graph**.
1. Seleccione **Permisos delegados**. Microsoft Graph expone muchos permisos, los más frecuentes se muestran en la parte superior de la lista.
1. En **Seleccionar permisos**, seleccione los permisos siguientes:

    | Permiso       | Descripción                                         |
    |------------------|-----------------------------------------------------|
    | `email`          | Ver la dirección de correo electrónico de los usuarios                           |
    | `offline_access` | Mantener el acceso a los datos a los que se le ha concedido acceso |
    | `openid`         | Inicio de sesión de los usuarios                                       |
    | `profile`        | Ver el perfil básico de los usuarios                           |
1. Seleccione **Agregar permisos** para completar el proceso.

Una vez configurados los permisos, cuando los usuarios de la aplicación inician sesión, se les pide su consentimiento para que la aplicación pueda acceder a la API de recursos en su nombre.

Como administrador, también puede otorgar el consentimiento en nombre de *todos* los usuarios y, por lo tanto, a ellos no se les pedirá. El consentimiento del administrador se describe más adelante en la sección [Más información sobre los permisos de la API y el consentimiento del administrador](#more-on-api-permissions-and-admin-consent) de este artículo.

### <a name="application-permission-to-microsoft-graph"></a>Permiso de aplicación para Microsoft Graph

Configure los permisos de una aplicación que necesite autenticarse por sí misma sin la interacción ni el consentimiento del usuario. Los permisos de aplicación suelen usarse en los servicios en segundo plano o en las aplicaciones de demonio que acceden a una API "sin periféricos" y en las API web que acceden a otra API (de nivel inferior).

En los pasos siguientes se concede el permiso *Files.Read.All* de Microsoft Graph como ejemplo.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Si tiene acceso a varios inquilinos, use el filtro **Directorio + suscripción** :::image type="icon" source="./media/quickstart-configure-app-access-web-apis/portal-01-directory-subscription-filter.png" border="false"::: del menú superior para seleccionar el inquilino que contiene el registro de la aplicación cliente.
1. Seleccione **Azure Active Directory** > **Registros de aplicaciones** y la aplicación cliente.
1. Seleccione **Permisos de API** > **Agregar un permiso** > **Microsoft Graph** > **Permisos de la aplicación**.
1. Todos los permisos expuestos por Microsoft Graph se muestran en **Seleccionar permisos**.
1. Seleccione los permisos que desee conceder a la aplicación. Por ejemplo, podría tener una aplicación de demonio que examine los archivos de la organización para alertar sobre un tipo o nombre de archivo específico.

    En **Seleccionar permisos**, expanda **Archivos** y seleccione el permiso *Files.Read.All*.
1. Seleccione **Agregar permisos**.

Algunos permisos, como *Files.Read.All* de Microsoft Graph, requieren el consentimiento del administrador. Para conceder el consentimiento del administrador, seleccione el botón **Concesión de consentimiento del administrador**, que se describe más adelante en la sección [Botón Consentimiento del administrador](#admin-consent-button).

### <a name="configure-client-credentials"></a>Configuración de las credenciales del cliente

Las aplicaciones que usan permisos de aplicación se autentican por sí mismas con sus propias credenciales, sin necesidad de la interacción del usuario. Para que la aplicación (o API) pueda acceder a Microsoft Graph, a su propia API web o a cualquier otra API mediante permisos de aplicación, debe configurar antes las credenciales de la aplicación cliente.

Para más información sobre la configuración de las credenciales de una aplicación, consulte la sección [Agregar credenciales](quickstart-register-app.md#add-credentials) de [Inicio rápido: Registro de una aplicación en la plataforma de identidad de Microsoft](quickstart-register-app.md).

## <a name="more-on-api-permissions-and-admin-consent"></a>Más información sobre los permisos de la API y el consentimiento del administrador

El panel **Permisos de API** del registro de una aplicación contiene una tabla [Permisos configurados](#configured-permissions) y quizá también otra denominada [Otros permisos concedidos](#other-permissions-granted). En las secciones siguientes se describen las dos tablas y el [botón Consentimiento del administrador](#admin-consent-button).

### <a name="configured-permissions"></a>Permisos configurados

La tabla **Permisos configurados** del panel **Permisos de API** muestra la lista de permisos que la aplicación requiere para el funcionamiento básico: la lista de *acceso a los recursos requeridos* (RRA). Los usuarios o sus administradores deberán dar su consentimiento para estos permisos antes de usar la aplicación. Posteriormente se pueden solicitar otros permisos opcionales en tiempo de ejecución (mediante consentimiento dinámico).

Esta es la lista mínima de permisos para los que los usuarios tendrán que dar su consentimiento a la aplicación. Podría haber más, pero estos siempre serán necesarios. Por seguridad, y para que los usuarios y administradores se sientan más cómodos con la aplicación, no pida nunca nada que no necesite.

Puede agregar o quitar los permisos que aparecen en esta tabla siguiendo los pasos descritos anteriormente o desde [Otros permisos concedidos](#other-permissions-granted) (que se describen en la sección siguiente). Como administrador, puede conceder el consentimiento del administrador para el conjunto completo de permisos de una API que aparece en la tabla y revocar el consentimiento de permisos individuales.

### <a name="other-permissions-granted"></a>Otros permisos concedidos

También podría ver una tabla titulada **Otros permisos concedidos para {su inquilino}** en el panel **Permisos de API**. En la tabla **Otros permisos concedidos para {su inquilino}** se muestran los permisos concedidos para el inquilino que no se han configurado explícitamente en el objeto de aplicación. Estos permisos se solicitaron y consintieron dinámicamente. Esta sección solo aparece si hay al menos un permiso vigente.

Puede agregar el conjunto de permisos de API entero o permisos individuales de esta tabla a la tabla **Permisos configurados**. Como administrador, en esta sección puede revocar el consentimiento del administrador para las API o permisos individuales.

### <a name="admin-consent-button"></a>Botón Consentimiento del administrador

El botón **Conceder consentimiento de administrador para {su inquilino}** permite a un administrador conceder su consentimiento a los permisos configurados para la aplicación. Al seleccionar el botón, se muestra un cuadro de diálogo que le solicita que confirme el consentimiento.

:::image type="content" source="media/quickstart-configure-app-access-web-apis/portal-03-grant-admin-consent-button.png" alt-text="Botón Conceder consentimiento de administrador resaltado en el panel Permisos configurados de Azure Portal":::

Después de conceder el consentimiento, los permisos que requerían el consentimiento del administrador lo muestran como concedido:

:::image type="content" source="media/quickstart-configure-app-access-web-apis/portal-04-admin-consent-granted.png" alt-text="Tabla Permisos configurados en Azure Portal que muestra el consentimiento de administrador concedido para el permiso Files.Read.All":::

El botón **Conceder consentimiento de administrador** está *deshabilitado* si no es el administrador o si no se han configurado permisos para la aplicación. Si tiene permisos que se han concedido pero no configurado aún, el botón de consentimiento del administrador le pedirá que los administre. Puede agregarlos a los permisos configurados o quitarlos.

## <a name="next-steps"></a>Pasos siguientes

Avance al siguiente inicio rápido de la serie para aprender a configurar los tipos de cuenta que pueden acceder a la aplicación. Por ejemplo, puede que quiera limitar el acceso solo a los usuarios de su organización (un solo inquilino) o permitir a los usuarios de otros inquilinos de Azure AD (multiinquilino) y a quienes tengan cuentas personales de Microsoft (MSA).

> [!div class="nextstepaction"]
> [Modificación de las cuentas compatibles con una aplicación](quickstart-modify-supported-accounts.md)