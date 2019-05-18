---
title: Eliminación de una aplicación registrada con la plataforma de identidad de Microsoft | Azure
description: Aprenda a quitar una aplicación registrada con la plataforma de identidad de Microsoft.
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
ms.openlocfilehash: 271fe45082749afc310e8f217e8119d3931c3cac
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2019
ms.locfileid: "65413734"
---
# <a name="quickstart-remove-an-application-registered-with-the-microsoft-identity-platform"></a>Inicio rápido: Eliminación de una aplicación registrada con la plataforma de identidad de Microsoft

Es posible que los desarrolladores empresariales y proveedores de software como servicio (SaaS) que han registrado aplicaciones con la plataforma de identidad de Microsoft deban quitar el registro de una aplicación.

En esta guía de inicio rápido, aprenderá a hacer lo siguiente:

* Eliminación de una aplicación creada por el usuario o por la organización
* Eliminación de una aplicación creada por otra organización

## <a name="prerequisites"></a>Requisitos previos

Debe tener un inquilino que tenga las aplicaciones registradas en él. Para saber cómo agregar y registrar aplicaciones, consulte [Registro de una aplicación con la plataforma de identidad de Microsoft](quickstart-register-app.md).

## <a name="remove-an-application-authored-by-you-or-your-organization"></a>Eliminación de una aplicación creada por el usuario o por la organización

Las aplicaciones que el usuario o la empresa han registrado están representadas tanto por un objeto de aplicación como por un objeto de entidad de servicio en el inquilino. Para obtener más información, vea [Objetos de aplicación y objetos de entidad de servicio](active-directory-application-objects.md).

### <a name="to-remove-an-application"></a>Para quitar una aplicación

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
2. Si la cuenta proporciona acceso a más de un inquilino, haga clic en la cuenta en la esquina superior derecha y establezca la sesión del portal en el inquilino de Azure AD deseado.
3. En el panel de navegación izquierdo, seleccione el servicio **Azure Active Directory** y, después, seleccione **Registros de aplicaciones**. Busque y seleccione la aplicación que desea configurar. Cuando haya seleccionado la aplicación, verá la página **Introducción** de la aplicación.
4. En la página **Introducción**, seleccione **Eliminar**.
5. Seleccione **Sí** para confirmar que quiere eliminar la aplicación.

   > [!NOTE]
   > Para eliminar una aplicación, es necesario que aparezca como propietario de la aplicación o que tenga privilegios de administrador.

## <a name="remove-an-application-authored-by-another-organization"></a>Eliminación de una aplicación creada por otra organización

Si está viendo **Registros de aplicaciones** en el contexto de un inquilino, un subconjunto de las aplicaciones que aparecen en la pestaña **Todas las aplicaciones** son de otro inquilino y se registraron en su inquilino durante el proceso de consentimiento. Más específicamente, se representan solo mediante un objeto de entidad de servicio en su inquilino, sin un objeto aplicación correspondiente. Para más información sobre las diferencias entre objetos de aplicación y objetos de entidad de servicio, consulte [Objetos de aplicación y de entidad de servicio en Azure AD](active-directory-application-objects.md).

Para quitar el acceso de una aplicación a su directorio (después de concederle consentimiento), el administrador de la compañía debe eliminar su entidad de servicio. El administrador debe tener acceso de administrador global y puede quitar la aplicación mediante Azure Portal o usar los [cmdlets de PowerShell de Azure AD](https://go.microsoft.com/fwlink/?LinkId=294151) para eliminarlo.

## <a name="next-steps"></a>Pasos siguientes

Obtenga información acerca de estas otras guías de inicio rápido de administración de aplicaciones relacionadas:

* [Registro de una aplicación con la plataforma de identidad de Microsoft](quickstart-register-app.md)
* [Configuración de una aplicación cliente para tener acceso a las API web](quickstart-configure-app-access-web-apis.md)
* [Configuración de una aplicación para exponer las API web](quickstart-configure-app-expose-web-apis.md)
* [Modificación de las cuentas compatibles con una aplicación](quickstart-modify-supported-accounts.md)
