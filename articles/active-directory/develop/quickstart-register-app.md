---
title: 'Inicio rápido: Registro de una aplicación en la plataforma de identidad de Microsoft | Azure'
description: En este inicio rápido, aprenderá cómo registrar una aplicación mediante la plataforma de identidad de Microsoft.
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: quickstart
ms.workload: identity
ms.date: 09/03/2020
ms.author: marsma
ms.custom: aaddev, identityplatformtop40, contperfq1
ms.reviewer: aragra, lenalepa, sureshja
ms.openlocfilehash: 5f34215d57bd5dae8c9a5e6e8f4630b7ed0c827e
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89436753"
---
# <a name="quickstart-register-an-application-with-the-microsoft-identity-platform"></a>Inicio rápido: Registro de una aplicación en la plataforma de identidad de Microsoft

En este inicio rápido, va a registrar una aplicación en Azure Portal con el fin de que la plataforma de identidad de Microsoft pueda proporcionar servicios de autenticación y autorización para la aplicación y sus usuarios.

Es necesario registrar todas las aplicaciones para las que desee realizar la administración de identidad y acceso (IAM) mediante la plataforma de identidad de Microsoft. Tanto si se trata de una aplicación cliente (por ejemplo, móvil o web) como de una API web que respalda una aplicación cliente, al registrarlas se establece una relación de confianza entre la aplicación y el proveedor de identidades, es decir, la plataforma de identidad de Microsoft.

## <a name="prerequisites"></a>Requisitos previos

* Una cuenta de Azure con una suscripción activa: [cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Finalización de [Inicio rápido: Configuración de un inquilino](quickstart-create-new-tenant.md)

## <a name="register-an-application"></a>Registro de una aplicación

El registro de la aplicación establece una relación de confianza entre la aplicación y la plataforma de identidad de Microsoft. La confianza es unidireccional: la aplicación confía en la plataforma de identidad de Microsoft y no al revés.

Siga estos pasos para crear el registro de la aplicación:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Si tiene acceso a varios inquilinos, use el filtro **Directorio + suscripción** :::image type="icon" source="./media/quickstart-register-app/portal-01-directory-subscription-filter.png" border="false"::: del menú superior para seleccionar el inquilino en el que desea registrar una aplicación.
1. Busque y seleccione **Azure Active Directory**.
1. En **Administrar**, seleccione **Registros de aplicaciones** y luego **Nuevo registro**.
1. Escriba el **nombre** de la aplicación. Los usuarios de la aplicación pueden ver este nombre, el cual se puede cambiar más tarde.
1. Especifique qué personas pueden usar la aplicación. A veces, se conoce a estas personas como *público de inicio de sesión*.

    | Tipos de cuenta admitidos | Descripción |
    |-------------------------|-------------|
    | **Solo las cuentas de este directorio organizativo** | Seleccione esta opción si va a desarrollar una aplicación para que la utilicen usuarios (o invitados) de *su* inquilino.<br><br>A menudo denominada aplicación de *línea de negocio*, se trata de una aplicación de **un solo inquilino** en la plataforma de identidad de Microsoft. |
    | **Cuentas en cualquier directorio organizativo** | Seleccione esta opción si desea que los usuarios de *cualquier* inquilino de Azure AD puedan usar la aplicación. Esta opción es adecuada si, por ejemplo, va a desarrollar una aplicación de software como servicio (SaaS) que desea proporcionar a varias organizaciones.<br><br>Esta aplicación se denomina **multiinquilino** en la plataforma de identidad de Microsoft. |
    | **Cuentas en cualquier directorio organizativo y cuentas Microsoft personales** | Seleccione esta opción para establecer como destino el mayor conjunto posible de clientes.<br><br>Al seleccionar esta opción, estará registrando una aplicación **multiinquilino** que también admite usuarios con **cuentas personales de Microsoft** (MSA). |
    | **Cuentas personales de Microsoft** | Seleccione esta opción si va a desarrollar una aplicación para que la utilicen usuarios con cuentas personales de Microsoft. Las cuentas personales de Microsoft abarcan las cuentas de Skype, Xbox, Live y Hotmail. |

1. No escriba ningún valor en **URI de redirección (opcional)** . Se configurará en la sección siguiente.
1. Seleccione **Registrar** para completar el registro inicial de la aplicación.

    :::image type="content" source="media/quickstart-register-app/portal-02-app-reg-01.png" alt-text="Captura de pantalla de Azure Portal en un explorador web que muestra el panel Registrar una aplicación":::.

Cuando se completa el registro, Azure Portal muestra el panel de **información general** de registro de la aplicación, que incluye el **Id. de aplicación (cliente)** . Este valor, también conocido como *Id. de cliente*, identifica de forma única la aplicación en la plataforma de identidad de Microsoft.

El código de la aplicación, o más habitualmente una biblioteca de autenticación empleada en la aplicación, también utiliza la identificación del cliente como un aspecto de la validación de los tokens de seguridad que recibe de la plataforma de identidad.

:::image type="content" source="media/quickstart-register-app/portal-03-app-reg-02.png" alt-text="Captura de pantalla de Azure Portal en un explorador web que muestra el panel de información general del registro de una aplicación":::

## <a name="add-a-redirect-uri"></a>Incorporación de un URI de redirección

Un URI de redirección es la ubicación a la que la plataforma de identidad de Microsoft redirige el cliente de un usuario y envía tokens de seguridad tras la autenticación.

En una aplicación web de producción, por ejemplo, el URI de redirección suele ser un punto de conexión público en el que se ejecuta la aplicación, por ejemplo, `https://contoso.com/auth-response`. Durante la fase de desarrollo, también es habitual incorporar el punto de conexión en el que se ejecuta la aplicación localmente, por ejemplo, `https://127.0.0.1/auth-response`.

Para agregar y modificar los URI de redirección de las aplicaciones registradas, especifique los parámetros en [Configuraciones de plataforma](#configure-platform-settings).

### <a name="configure-platform-settings"></a>Configuración de los valores de plataforma

Los valores de cada tipo de aplicación, incluidos los URI de redirección, se especifican en **Configuraciones de plataforma** en Azure Portal. Algunas plataformas, como las de **aplicaciones web** y **aplicaciones de página única**, requieren que se especifique manualmente un URI de redirección. Para otras plataformas, como las de aplicaciones móviles y de escritorio, es posible elegir entre los URI de redirección que se generan automáticamente cuando configura las demás opciones.

Para configurar los valores de la aplicación según la plataforma o el dispositivo de destino:

1. Seleccione la aplicación en **Registros de aplicaciones** en Azure Portal.
1. En **Administrar**, seleccione **Autenticación**.
1. En **Configuraciones de plataforma**, seleccione **Agregar una plataforma**.
1. En **Configurar plataformas**, seleccione el icono del tipo de aplicación (plataforma) para configurar los valores.

    :::image type="content" source="media/quickstart-register-app/portal-04-app-reg-03-platform-config.png" alt-text="Captura de pantalla del panel Configuración de la plataforma en Azure Portal" border="false":::

    | Plataforma | Parámetros de configuración |
    | -------- | ---------------------- |
    | **Web** | Escriba un **URI de redirección** para la aplicación, es decir, la ubicación a la que la plataforma de identidad de Microsoft redirigirá el cliente de un usuario y enviará los tokens de seguridad tras la autenticación.<br/><br/>Seleccione esta plataforma para las aplicaciones web estándar que se ejecuten en un servidor. |
    | **Aplicación de página única** | Escriba un **URI de redirección** para la aplicación, es decir, la ubicación a la que la plataforma de identidad de Microsoft redirigirá el cliente de un usuario y enviará los tokens de seguridad tras la autenticación.<br/><br/>Seleccione esta plataforma si va a desarrollar una aplicación web de cliente en JavaScript, o bien con un marco de trabajo como Angular, Vue.js, React.js o Blazor WebAssembly. |
    | **iOS/macOS** | Especifique el **identificador de paquete** de la aplicación, que está disponible en XCode en *Info.plist*, o bien en Configuración de compilación.<br/><br/>Al especificar un identificador de paquete, se genera un URI de redirección. |
    | **Android** | Especifique el **nombre del paquete** de la aplicación, que encontrará en el archivo *AndroidManifest.xml*. A continuación, genere e indique el **hash de firma**.<br/><br/>Al especificar estos valores, se genera un URI de redirección. |
    | **Aplicaciones móviles y de escritorio** | Seleccione uno de los **URI de redirección sugeridos** o especifique un **URI de redirección personalizado**.<br/>En el caso de las aplicaciones de escritorio, se recomienda utilizar:<br/>`https://login.microsoftonline.com/common/oauth2/nativeclient`<br/><br/>Seleccione esta plataforma para las aplicaciones móviles que no utilicen la biblioteca de autenticación de Microsoft (MSAL) más reciente o que no usen un agente. Seleccione también esta plataforma para las aplicaciones de escritorio. |
1. Seleccione **Configurar** para completar la configuración de la plataforma.

### <a name="redirect-uri-restrictions"></a>Restricciones aplicables a los URI de redirección

Existen ciertas restricciones con respecto al formato de los URI de redirección que agrega al registro de una aplicación. Para más información sobre estas restricciones, consulte [Restricciones y limitaciones del identificador URI de redirección (dirección URL de respuesta)](reply-url.md).

## <a name="add-credentials"></a>Adición de credenciales

Las credenciales se usan con las aplicaciones cliente confidenciales que acceden a una API web. Ejemplos de aplicaciones cliente confidenciales son, entre otras, las aplicaciones web, las API web o las aplicaciones demonio y de servicios. Las credenciales permiten que la aplicación se autentique a sí misma, por lo que no se requiere la interacción del usuario en tiempo de ejecución.

Puede agregar certificados y secretos de cliente (una cadena) como credenciales al registro de la aplicación cliente confidencial.

:::image type="content" source="media/quickstart-register-app/portal-05-app-reg-04-credentials.png" alt-text="Captura de pantalla de Azure Portal que muestra el panel de certificados y secretos durante el registro de una aplicación":::

### <a name="add-a-certificate"></a>Incorporación de un certificado

Los certificados, a veces denominados *claves públicas*, son el tipo de credenciales que se recomienda, ya que proporcionan un mayor nivel de seguridad que un secreto de cliente.

1. Seleccione la aplicación en **Registros de aplicaciones** en Azure Portal.
1. Seleccione **Certificados y secretos** > **Cargar certificado**.
1. Seleccione el archivo que desea cargar. Debe ser uno de los siguientes tipos de archivo: .cer, .pem, .crt.
1. Seleccione **Agregar**.

### <a name="add-a-client-secret"></a>Agregar un secreto de cliente

El secreto de cliente, conocido también como *contraseña de aplicación*, es un valor de cadena que la aplicación puede usar en lugar de un certificado a fin de identificarse. Este es el tipo de credenciales más fácil de usar y, con frecuencia, se emplea en la fase de desarrollo, pero se considera menos seguro que un certificado. Debería utilizar los certificados cuando las aplicaciones se ejecuten en el entorno de producción.

1. Seleccione la aplicación en **Registros de aplicaciones** en Azure Portal.
1. Seleccione **Certificados y secretos** >  **Nuevo secreto de cliente**.
1. Agregue una descripción para el secreto de cliente.
1. Seleccione una duración.
1. Seleccione **Agregar**.
1. **Tome nota del valor del secreto** para utilizarlo en el código de la aplicación cliente, ya que *no volverá a mostrarse* una vez que salga de esta página.

## <a name="next-steps"></a>Pasos siguientes

Las aplicaciones cliente suelen necesitar acceso a los recursos de una API web. Además de proteger la aplicación cliente con la plataforma de identidad de Microsoft, puede usar la plataforma para autorizar el acceso basado en permisos con ámbito a la API web.

Pase al siguiente inicio rápido de la serie con el fin de crear otro registro de aplicación para la API web y exponer sus ámbitos.

> [!div class="nextstepaction"]
> [Configuración de una aplicación para exponer una API web](quickstart-configure-app-expose-web-apis.md)
