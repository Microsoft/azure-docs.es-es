---
title: Habilitación de Azure Multi-Factor Authentication
description: En este tutorial aprenderá a habilitar Azure Multi-Factor Authentication para un grupo de usuarios y a probar la solicitud de factor secundario durante un evento de inicio de sesión.
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: tutorial
ms.date: 07/13/2020
ms.author: iainfou
author: iainfoulds
ms.reviewer: michmcla
ms.collection: M365-identity-device-management
ms.openlocfilehash: b9072c5611f5bd5a4b8cca082cb2bfd7a1e3f1b2
ms.sourcegitcommit: 6fc156ceedd0fbbb2eec1e9f5e3c6d0915f65b8e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88718907"
---
# <a name="tutorial-secure-user-sign-in-events-with-azure-multi-factor-authentication"></a>Tutorial: Protección de eventos de inicio de sesión de usuario con Azure Multi-Factor Authentication

La autenticación multifactor (MFA) es un proceso por el que, durante un evento de inicio de sesión, se solicitan a un usuario otras formas de identificación. En esta solicitud se puede pedir la introducción de un código en el teléfono móvil o el escaneado de la huella digital. Al exigir una segunda forma de autenticación, aumenta la seguridad, ya que este factor adicional no es algo que resulte fácil de obtener o duplicar para un atacante.

Azure Multi-Factor Authentication y las directivas de acceso condicional proporcionan la flexibilidad necesaria para habilitar la autenticación multifactor en los usuarios durante eventos de inicio de sesión específicos.

> [!IMPORTANT]
> Este tutorial muestra al administrador cómo habilitar Azure Multi-Factor Authentication.
>
> Si el equipo de TI no ha habilitado la función para usar Azure Multi-Factor Authentication o si tiene problemas para iniciar sesión, póngase en contacto con el departamento de soporte técnico para obtener ayuda adicional.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear una directiva de acceso condicional para habilitar Azure Multi-Factor Authentication para un grupo de usuarios
> * Configurar las condiciones de la directiva que solicitan la autenticación multifactor
> * Probar el proceso de autenticación multifactor como usuario

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, necesitará los siguientes recursos y privilegios:

* Un inquilino de Azure AD activo con al menos una licencia de Azure AD Premium P1 o de prueba habilitada.
    * Si es preciso, [cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Una cuenta con privilegios de *administrador global*.
* Un usuario que no sea administrador con una contraseña que conozca, como *usuarioDePrueba*. Utilizará esta cuenta para probar la experiencia de Azure Multi-Factor Authentication por parte del usuario final en este tutorial.
    * Si necesita crear un usuario, consulte [Inicio rápido: Incorporación de nuevos usuarios a Azure Active Directory](../fundamentals/add-users-azure-active-directory.md).
* Un grupo del que sea miembro el usuario que no es administrador, como *Grupo-Prueba-MFA*. En este tutorial, habilitará Azure Multi-Factor Authentication para este grupo.
    * Si necesita crear un grupo, consulte [Creación de un grupo y adición de miembros en Azure Active Directory](../fundamentals/active-directory-groups-create-azure-portal.md).

## <a name="create-a-conditional-access-policy"></a>Creación de una directiva de acceso condicional

Se recomienda habilitar y usar Azure Multi-Factor Authentication a través de directivas de acceso condicional. El acceso condicional permite crear y definir directivas que reaccionan a los eventos de inicio de sesión y solicitan acciones adicionales antes de que se conceda a un usuario el acceso a una aplicación o un servicio.

![Diagrama general sobre el funcionamiento del acceso condicional para proteger el proceso de inicio de sesión](media/tutorial-enable-azure-mfa/conditional-access-overview.png)

Las directivas de acceso condicional pueden ser pormenorizadas y específicas, con el objetivo de permitir que los usuarios sean productivos en todo lugar y momento, pero también de proteger la organización. En este tutorial, crearemos una directiva de acceso condicional básica para solicitar la autenticación multifactor cuando un usuario inicie sesión en Azure Portal. En un tutorial posterior de esta serie, se configurará Azure Multi-Factor Authentication mediante una directiva de acceso condicional basado en riesgos.

En primer lugar, cree una directiva de acceso condicional y asigne el grupo de usuarios de prueba de la siguiente manera:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) mediante una cuenta con permisos de *administrador global*.
1. Busque y seleccione **Azure Active Directory** y, a continuación, elija **Seguridad** en el menú del lado izquierdo.
1. Seleccione **Acceso condicional** y, a continuación, elija **+ Nueva directiva**.
1. Escriba un nombre para la directiva, como *Piloto de MFA*.
1. En **Asignaciones**, elija **Usuarios y grupos** y, a continuación, el botón de radio **Seleccionar usuarios y grupos**.
1. Active la casilla de **Usuarios y grupos**, y elija la opción **Seleccionar** para examinar los usuarios y grupos de Azure AD disponibles.
1. Busque y seleccione el grupo de Azure AD, como *Grupo-Prueba-MFA* y, a continuación, elija **Seleccionar**.

    [ ![Selección del grupo de Azure AD que se usará con la directiva de acceso condicional](media/tutorial-enable-azure-mfa/select-group-for-conditional-access-cropped.png) ](media/tutorial-enable-azure-mfa/select-group-for-conditional-access.png#lightbox)

1. Para aplicar la directiva de acceso condicional para el grupo, seleccione **Listo**.

## <a name="configure-the-conditions-for-multi-factor-authentication"></a>Configuración de las condiciones para la autenticación multifactor

Ahora que ya ha creado la directiva de acceso condicional y ha asignado un grupo de usuarios de prueba, definirá las aplicaciones en la nube o las acciones que desencadenarán la directiva. Estas aplicaciones en la nube o acciones son los escenarios para los que decide que se necesita un procesamiento adicional, como la solicitud de autenticación multifactor. Por ejemplo, puede decidir que el acceso a una aplicación financiera o al uso de las herramientas de administración requiere una solicitud de verificación adicional.

En este tutorial, configurará la directiva de acceso condicional para que se pida la autenticación multifactor cuando un usuario inicie sesión en Azure Portal.

1. Seleccione **Aplicaciones en la nube o acciones**. Puede decidir si aplica la directiva de acceso condicional a *Todas las aplicaciones en la nube* o bien si va a elegir la opción para *Seleccionar aplicaciones*. Para proporcionar flexibilidad, también puede excluir determinadas aplicaciones de la directiva.

    En este tutorial, en la página *Incluir*, elija el botón de radio **Seleccionar aplicaciones**.

1. Elija **Seleccionar** y examine la lista de eventos de inicio de sesión disponibles que se pueden usar.

    En este tutorial, elija **Microsoft Azure Management** para que la directiva se aplique a los eventos de inicio de sesión en Azure Portal.

1. Para aplicar la selección de aplicaciones, elija **Seleccionar** y, después, haga clic en **Listo**.

    ![Selección de la aplicación Microsoft Azure Management para incluir en la directiva de acceso condicional](media/tutorial-enable-azure-mfa/select-azure-management-app.png)

Los controles de acceso permiten definir los requisitos para que se conceda acceso a un usuario, como la necesidad de una aplicación cliente aprobada o el uso de un dispositivo unido a Azure AD híbrido. En este tutorial, se configurarán los controles de acceso para que se solicite la autenticación multifactor durante un evento de inicio de sesión en Azure Portal.

1. En *Controles de acceso*, elija **Conceder** y asegúrese de que el botón de radio **Conceder acceso** esté seleccionado.
1. Active la casilla **Requerir autenticación multifactor** y, después, elija **Seleccionar**.

Las directivas de acceso condicional se pueden establecer en *Solo informe*, si desea ver cómo afectaría la configuración a los usuarios, o en *Desactivada*, si no desea usar la directiva en este momento. Como este tutorial se ha destinado a un grupo de prueba de usuarios, vamos a habilitar la directiva y, después, a probar Azure Multi-Factor Authentication.

1. Establezca la opción *Habilitar directiva* en **Activada**.
1. Para aplicar la directiva de acceso condicional, seleccione **Crear**.

## <a name="test-azure-multi-factor-authentication"></a>Prueba de Azure Multi-Factor Authentication

Veamos la directiva de acceso condicional y Azure Multi-Factor Authentication en acción. En primer lugar, inicie sesión en un recurso que no requiera la autenticación multifactor, como se indica a continuación:

1. Abra una nueva ventana del explorador en modo de incógnito o InPrivate y vaya a [https://account.activedirectory.windowsazure.com](https://account.activedirectory.windowsazure.com).
1. Inicie sesión con el usuario de prueba que no es administrador, como *usuarioDePrueba*. No se le pide que complete la autenticación multifactor.
1. Cierre la ventana del explorador.

Inicie sesión en Azure Portal. Dado que Azure Portal se configuró en la directiva de acceso condicional para requerir una verificación adicional, aparece una solicitud de Azure Multi-Factor Authentication.

1. Abra una nueva ventana del explorador en modo de incógnito o InPrivate, y vaya a [https://portal.azure.com](https://portal.azure.com).
1. Inicie sesión con el usuario de prueba que no es administrador, como *usuarioDePrueba*. Se le pide que se registre y use Azure Multi-Factor Authentication. Siga las indicaciones para completar el proceso y verifique que inicia sesión correctamente en Azure Portal.

    ![Siga las indicaciones del explorador y de la solicitud de autenticación multifactor registrada para iniciar sesión.](media/tutorial-enable-azure-mfa/azure-multi-factor-authentication-browser-prompt.png)

1. Cierre la ventana del explorador.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si ya no desea usar la directiva de acceso condicional para habilitar la opción de Azure Multi-Factor Authentication configurada como parte de este tutorial, elimine la directiva siguiendo estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Busque y seleccione **Azure Active Directory** y, a continuación, elija **Seguridad** en el menú del lado izquierdo.
1. Seleccione **Acceso condicional** y, a continuación, elija la directiva que creó, como *Piloto de MFA*.
1. Elija **Eliminar** y confirme que desea eliminar la directiva.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha habilitado Azure Multi-Factor Authentication mediante directivas de acceso condicional para un grupo seleccionado de usuarios. Ha aprendido a:

> [!div class="checklist"]
> * Crear una directiva de acceso condicional para habilitar Azure Multi-Factor Authentication en un grupo de usuarios de Azure AD
> * Configurar las condiciones de la directiva que solicitan la autenticación multifactor
> * Probar el proceso de autenticación multifactor como usuario

> [!div class="nextstepaction"]
> [Habilitación de la escritura diferida de contraseñas para el autoservicio de restablecimiento de contraseñas (SSPR)](./tutorial-enable-sspr-writeback.md)