---
title: Registro de una aplicación con la plataforma de identidad de Microsoft (plataforma de identidad de Microsoft)
description: Aprenda a agregar y registrar una aplicación con la plataforma de identidad de Microsoft.
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
ms.date: 05/09/2019
ms.author: ryanwi
ms.custom: aaddev
ms.reviewer: aragra, lenalepa, sureshja
ms.collection: M365-identity-device-management
ms.openlocfilehash: 6943ad460bb5d852220e704492f892961992a59f
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2019
ms.locfileid: "65413602"
---
# <a name="quickstart-register-an-application-with-the-microsoft-identity-platform"></a>Inicio rápido: Registro de una aplicación en la plataforma de identidad de Microsoft

Los desarrolladores de la empresa y los proveedores de software como servicio (SaaS) pueden desarrollar aplicaciones de línea de negocio o servicios comerciales en la nube que se pueden integrar con la plataforma de identidad de Microsoft para ofrecer inicio de sesión seguro y autorización para los servicios.

En este inicio rápido se muestra cómo agregar y registrar una aplicación mediante la experiencia de **Registros de aplicaciones** en Azure Portal para que pueda integrarse con la plataforma de identidad de Microsoft. Para más información acerca de las nuevas características y mejoras en la nueva experiencia de los registros de aplicación, consulte [esta entrada de blog](https://developer.microsoft.com/graph/blogs/new-app-registration/).

## <a name="register-a-new-application-using-the-azure-portal"></a>Registro de una aplicación nueva mediante Azure Portal

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. Si la cuenta proporciona acceso a más de un inquilino, haga clic en la cuenta en la esquina superior derecha y establezca la sesión del portal en el inquilino de Azure AD que desee.
1. En el panel de navegación izquierdo, seleccione el servicio **Azure Active Directory** y, después, seleccione **Registros de aplicaciones > Nuevo registro**.
1. Cuando aparece la página **Registrar una aplicación**, escriba la información de registro de la aplicación:

   - **Nombre**: escriba un nombre significativo para la aplicación, que se mostrará a los usuarios de la aplicación.
   - **Tipos de cuentas compatibles**: seleccione qué cuentas desea que admita la aplicación.

       | Tipos de cuenta admitidos | DESCRIPCIÓN |
       |-------------------------|-------------|
       | **Solo las cuentas de este directorio organizativo** | Seleccione esta opción si va a crear una aplicación de línea de negocio (LOB). Esta opción no está disponible si no va a registrar la aplicación en un directorio.<br><br>Esta opción se asigna solo a un único inquilino de Azure AD.<br><br>Esta es la opción predeterminada a menos que registre la aplicación fuera de un directorio. En los casos en los que la aplicación se registra fuera de un directorio, el valor predeterminado son las cuentas personales de Microsoft y cuentas multiinquilino de Azure AD. |
       | **Cuentas en cualquier directorio organizativo** | Seleccione esta opción si desea tener como destino todos los clientes de negocios y del sector educativo.<br><br>Esta opción se asigna solo a un multiinquilino de Azure AD.<br><br>Si ha registrado la aplicación como solo para un único inquilino de Azure AD, puede actualizarla para que sea de multiinquilino de Azure AD y que vuelva a serlo para un solo inquilino mediante la hoja **Autenticación**. |
       | **Cuentas en cualquier directorio organizativo y cuentas Microsoft personales** | Seleccione esta opción para establecer como destino el mayor conjunto posible de clientes.<br><br>Esta opción asigna cuentas personales de Microsoft y cuentas multiinquilinos de Azure AD.<br><br>Si registró la aplicación como cuentas multiinquilino de Azure AD y cuentas personales de Microsoft, no puede cambiar esto en la interfaz de usuario. En su lugar, debe usar el editor de manifiestos de aplicación para cambiar los tipos de cuenta admitidos. |

   - **URI de redirección (opcional)**: seleccione el tipo de aplicación que se va a crear, **Web** o **Cliente público (móvil y escritorio)** y, a continuación, escriba el identificador URI de redireccionamiento (o la dirección URL de respuesta) para la aplicación.
       - Para aplicaciones web, proporcione la dirección URL base de la aplicación. Por ejemplo, `http://localhost:31544` podría ser la dirección URL de una aplicación web que se ejecuta en la máquina local. Los usuarios utilizan esta dirección URL para iniciar sesión en una aplicación cliente web.
       - Para aplicaciones cliente públicas, proporcione el identificador URI que utiliza Azure AD para devolver las respuestas de los tokens. Escriba un valor específico para la aplicación, como `myapp://auth`.

     Si desea ejemplos específicos de aplicaciones web o aplicaciones nativa, visite nuestras [guías de inicio rápido](https://docs.microsoft.com/azure/active-directory/develop/#quickstarts).

1. Cuando termine, seleccione **Registrar**.

    [![Registro de una aplicación nueva en Azure Portal](./media/quickstart-add-azure-ad-app-preview/new-app-registration-expanded.png)](./media/quickstart-add-azure-ad-app-preview/new-app-registration-expanded.png#lightbox)

Azure AD le asigna un identificador de aplicación único (cliente) a la aplicación y le lleva a la página **Información general** de la aplicación. Para agregar funcionalidades adicionales a la aplicación, puede seleccionar otras opciones de configuración, como personalización de marca, certificados y secretos, permisos de API y mucho más.

[![Página de introducción de la aplicación recién registrada](./media/quickstart-add-azure-ad-app-preview/new-app-overview-page-expanded.png)](./media/quickstart-add-azure-ad-app-preview/new-app-overview-page-expanded.png#lightbox)

## <a name="next-steps"></a>Pasos siguientes

- Más información acerca de [permisos y consentimiento](v2-permissions-and-consent.md).
- Para habilitar características de configuración adicionales en el registro de la aplicación, como credenciales y permisos, y para habilitar el inicio de sesión para usuarios de otros inquilinos, consulte estas guías de inicio rápido:
    - [Configuración de una aplicación cliente para tener acceso a las API web](quickstart-configure-app-access-web-apis.md)
    - [Configuración de una aplicación para exponer las API web](quickstart-configure-app-expose-web-apis.md)
    - [Modificación de las cuentas compatibles con una aplicación](quickstart-modify-supported-accounts.md)
- Elija una [guía de inicio rápido](https://docs.microsoft.com/azure/active-directory/develop/#quickstarts) para crear rápidamente una aplicación y agregar funcionalidad similar a obtener tokens, actualizar tokens, iniciar sesión de un usuario, mostrar información de usuario y mucho más.
- Para obtener más información acerca de los dos objetos de Azure AD que representan una aplicación registrada y la relación entre ellos, consulte [Objetos de aplicación y de entidad de servicio](app-objects-and-service-principals.md).
- Para más información acerca de las directrices de personalización de marca que hay que usar al desarrollar aplicaciones, consulte [Directrices de personalización de marca para aplicaciones](howto-add-branding-in-azure-ad-apps.md).
