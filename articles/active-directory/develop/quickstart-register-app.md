---
title: 'Inicio rápido: Registro de las aplicaciones con la Plataforma de identidad de Microsoft | Azure'
description: En este inicio rápido, aprenderá cómo agregar y registrar una aplicación con la Plataforma de identidad de Microsoft.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: quickstart
ms.workload: identity
ms.date: 03/12/2020
ms.author: ryanwi
ms.custom: aaddev, identityplatformtop40
ms.reviewer: aragra, lenalepa, sureshja
ms.openlocfilehash: 65fff06b4a2d28bbc276920ccbaba90d814d03f3
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88115363"
---
# <a name="quickstart-register-an-application-with-the-microsoft-identity-platform"></a>Inicio rápido: Registro de una aplicación en la plataforma de identidad de Microsoft

En este inicio rápido, se registra una aplicación mediante la experiencia **Registros de aplicaciones** en Azure Portal. 

La aplicación se integra con la Plataforma de identidad de Microsoft; para ello, se registra con un inquilino de Azure Active Directory. Los desarrolladores de empresas y los proveedores de software como servicio (SaaS) pueden desarrollar aplicaciones de línea de negocio o servicios comerciales en la nube que se pueden integrar con la Plataforma de identidad de Microsoft. La integración proporciona un inicio de sesión seguro y una autorización para estos servicios.

## <a name="prerequisites"></a>Prerrequisitos

* Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).
* Un [inquilino de Azure AD](quickstart-create-new-tenant.md).

## <a name="register-a-new-application-using-the-azure-portal"></a>Registro de una aplicación nueva mediante Azure Portal

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. Si la cuenta proporciona acceso a más de un inquilino, seleccione su cuenta en la esquina superior derecha. Establezca la sesión del portal en el inquilino de Azure AD que desee.
1. Busque y seleccione **Azure Active Directory**. En **Administrar**, seleccione **Registros de aplicaciones**.
1. Seleccione **Nuevo registro**.
1. En **Registrar una aplicación**, escriba un nombre de aplicación significativo para mostrar a los usuarios.
1. Especifique quién puede usar la aplicación, como se indica a continuación:

    | Tipos de cuenta admitidos | Descripción |
    |-------------------------|-------------|
    | **Solo las cuentas de este directorio organizativo** | Seleccione esta opción si va a crear una aplicación de línea de negocio (LOB). Esta opción no está disponible si no va a registrar la aplicación en un directorio.<br><br>Esta opción se asigna solo a un único inquilino de Azure AD.<br><br>Esta es la opción predeterminada, a menos que registre la aplicación fuera de un directorio. En los casos en los que la aplicación se registra fuera de un directorio, el valor predeterminado son las cuentas personales de Microsoft y cuentas multiinquilino de Azure AD. |
    | **Cuentas en cualquier directorio organizativo** | Seleccione esta opción si desea tener como destino todos los clientes de negocios y del sector educativo.<br><br>Esta opción se asigna solo a un multiinquilino de Azure AD.<br><br>Si ha registrado la aplicación como solo para un único inquilino de Azure AD, puede actualizarla para que sea de multiinquilino de Azure AD y que vuelva a serlo para un solo inquilino mediante la página **Autenticación**. |
    | **Cuentas en cualquier directorio organizativo y cuentas Microsoft personales** | Seleccione esta opción para establecer como destino el mayor conjunto posible de clientes.<br><br>Esta opción asigna cuentas personales de Microsoft y cuentas multiinquilinos de Azure AD.<br><br>Si registró la aplicación como cuentas multiinquilino de Azure AD y cuentas personales de Microsoft, no puede cambiar esta opción en la interfaz de usuario. En su lugar, debe usar el editor de manifiestos de aplicación para cambiar los tipos de cuenta admitidos. |

1. En **URI de redirección (opcional)** , seleccione el tipo de aplicación que va a crear: **Web** o **Cliente público (dispositivos móviles y escritorio)** . A continuación, escriba el identificador URI de redireccionamiento o la dirección URL de respuesta de la aplicación.

    * Para aplicaciones web, proporcione la dirección URL base de la aplicación. Por ejemplo, `https://localhost:31544` podría ser la dirección URL de una aplicación web que se ejecuta en la máquina local. Los usuarios utilizan esta dirección URL para iniciar sesión en una aplicación cliente web.
    * Para aplicaciones cliente públicas, proporcione el identificador URI que utiliza Azure AD para devolver las respuestas de los tokens. Escriba un valor específico para la aplicación, como `myapp://auth`.

    Para obtener ejemplos de aplicaciones web o aplicaciones nativas, consulte las guías de inicio rápido en la [Plataforma de identidad de Microsoft](./index.yml).

1. Cuando termine, seleccione **Registrar**.

    ![Muestra la pantalla para registrar una aplicación nueva en Azure Portal.](./media/quickstart-add-azure-ad-app-preview/new-app-registration.png)

Azure AD asigna un identificador de aplicación o cliente único a la aplicación. El portal abre la página de **información general** de la aplicación. Para agregar funcionalidades a la aplicación, puede seleccionar otras opciones de configuración, como personalización de marca, certificados y secretos, permisos de API y mucho más.

![Ejemplo de una página de introducción de la aplicación recién registrada](./media/quickstart-add-azure-ad-app-preview/new-app-overview-page-expanded.png)

## <a name="next-steps"></a>Pasos siguientes

* Para acceder a las API Web, consulte [Inicio rápido: Configuración de una aplicación cliente para tener acceso a las API web](quickstart-configure-app-access-web-apis.md)

* Para obtener información sobre los permisos, vea [Permisos y consentimiento en el punto de conexión de la Plataforma de identidad de Microsoft](v2-permissions-and-consent.md).

* Para exponer las API Web, consulte [Inicio rápido: Configuración de una aplicación para exponer las API web](quickstart-configure-app-expose-web-apis.md).

* Para administrar cuentas admitidas, consulte [Inicio rápido: Modificación de las cuentas compatibles con una aplicación](quickstart-modify-supported-accounts.md).

* Para compilar una aplicación y agregar funcionalidad, consulte las guías de inicio rápido en la [Plataforma de identidad de Microsoft](./index.yml).

* Para obtener más información acerca de los dos objetos de Azure AD que representan una aplicación registrada y la relación entre ellos, consulte [Objetos de aplicación y de entidad de servicio](app-objects-and-service-principals.md).

* Para más información acerca de las directrices de personalización de marca que hay que usar al desarrollar aplicaciones, consulte [Directrices de personalización de marca para aplicaciones](howto-add-branding-in-azure-ad-apps.md).