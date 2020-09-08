---
title: 'Implementación de una herramienta de administración con una plantilla de Azure Resource Manager: Azure'
description: Cómo instalar una herramienta de interfaz de usuario con una plantilla de Azure Resource Manager para administrar los recursos de Windows Virtual Desktop (clásico).
author: Heidilohr
ms.topic: how-to
ms.date: 03/30/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: c261ea174f37de51a8fbd5b36aeb40c26d44e6a9
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88691401"
---
# <a name="deploy-a-windows-virtual-desktop-classic-management-tool-with-an-azure-resource-manager-template"></a>Implementación de una herramienta de administración de Windows Virtual Desktop (clásico) con una plantilla de Azure Resource Manager

>[!IMPORTANT]
>Este contenido se aplica a Windows Virtual Desktop (clásico), que no admite objetos de Windows Virtual Desktop para Azure Resource Manager.

Las instrucciones de este artículo le indicarán cómo implementar la interfaz de usuario mediante una plantilla de Azure Resource Manager.

## <a name="important-considerations"></a>Consideraciones importantes

Puesto que la aplicación requiere de su consentimiento para interactuar con Windows Virtual Desktop, esta herramienta no es compatible con escenarios de negocio a negocio (B2B). La suscripción de cada inquilino de Azure Active Directory (AAD) necesitará su propia implementación independiente de la herramienta de administración.

Esta herramienta de administración es un ejemplo. Microsoft proporcionará actualizaciones de calidad y seguridad importantes. El [código fuente está disponible en GitHub](https://github.com/Azure/RDS-Templates/tree/master/wvd-templates/wvd-management-ux/deploy). Se aconseja que los clientes y partners personalicen la herramienta para que se ajuste a sus necesidades empresariales.

Los siguientes exploradores son compatibles con la herramienta de administración:
- Google Chrome 68 o una versión posterior
- Microsoft Edge 40.15063 o una versión posterior
- Mozilla Firefox 52.0 o una versión posterior
- Safari 10 o una versión posterior (solo macOS)

## <a name="what-you-need-to-deploy-the-management-tool"></a>Requisitos para implementar la herramienta de administración

Antes de implementar la herramienta de administración, necesitará que un usuario de Azure Active Directory (Azure AD) cree un registro de aplicaciones e implemente la interfaz de usuario de administración. Dicho usuario debe:

- Deshabilitar la autenticación multifactor (MFA) de Azure.
- Tener los permisos para crear recursos en la suscripción de Azure.
- Tener los permisos para crear una aplicación de Azure AD. Siga los pasos que se describen en la sección [Permisos necesarios](../../active-directory/develop/howto-create-service-principal-portal.md#permissions-required-for-registering-an-app) para comprobar si el usuario tiene los permisos que necesita.

Después de implementar y configurar la herramienta de administración, se recomienda pedir a un usuario que inicie la interfaz de usuario de administración para asegurarse de que todo funciona correctamente. El usuario que inicie la interfaz de usuario de administración deberá tener una asignación de roles que le permita ver o editar el inquilino de Windows Virtual Desktop.

## <a name="deploy-the-management-tool"></a>Implementación de la herramienta de administración

Antes de empezar, asegúrese de que las aplicaciones cliente y servidor tienen su consentimiento al visitar la [página de consentimiento de Windows Virtual Desktop](https://rdweb.wvd.microsoft.com) de la instancia de Azure Active Directory (AAD) representada.

Siga las instrucciones para implementar la plantilla de Azure Resource Manager:

1. Vaya a la [página RDS-Templates de Azure de GitHub](https://github.com/Azure/RDS-Templates/tree/master/wvd-templates/wvd-management-ux/deploy).
2. Implemente la plantilla en Azure.
    - Si va a implementarla en una suscripción Enterprise, desplácese hacia abajo y seleccione **Implementar en Azure**.
    - Si va a realizar la implementación en una suscripción de proveedor de soluciones en la nube, siga estas instrucciones para implementar en Azure:
        1. Desplácese hacia abajo, haga clic con el botón derecho en **Implementar en Azure** y, después, seleccione **Copy Link Location** (Copiar ubicación del vínculo).
        2. Abra un editor de texto como el Bloc de notas y pegue el vínculo ahí.
        3. Justo después de <https://portal.azure.com/> y antes de la almohadilla (#), escriba una arroba (@) seguida del nombre de dominio del inquilino. Este es un ejemplo del formato: <https://portal.azure.com/@Contoso.onmicrosoft.com#create/>.
        4. Inicie sesión en el Azure Portal como un usuario con permisos de administrador o colaborador en la suscripción del proveedor de soluciones en la nube.
        5. Pegue el vínculo que ha copiado en el editor de texto en la barra de direcciones.
3. Al especificar los parámetros, haga lo siguiente:
    - Para el parámetro **isServicePrincipal** seleccione **false**.
    - Para las credenciales, escriba sus credenciales de Azure AD con la autenticación multifactor deshabilitada. Estas se usarán para crear la aplicación de Azure AD y los recursos de Azure. Para obtener más información, consulte los [requisitos para implementar la herramienta de administración](#what-you-need-to-deploy-the-management-tool).
    - En el caso de **applicationName**, use para la aplicación un nombre único que se registrará en Azure Active Directory. Este nombre también se usará para la dirección URL de la aplicación web. Por ejemplo, puede usar un nombre como "Apr3UX".
4. Una vez que haya especificado los parámetros, acepte los términos y condiciones y seleccione **Comprar**.

## <a name="provide-consent-for-the-management-tool"></a>Consentimiento para la herramienta de administración

Después de completar la plantilla de Azure Resource Manager de GitHub, encontrará un grupo de recursos que contiene dos instancias de App Services junto con un plan de App Service en Azure Portal.

Antes de iniciar sesión y usar la herramienta de administración, debe dar su consentimiento para la nueva aplicación de Azure AD asociada con la herramienta de administración. Al dar su consentimiento, la herramienta de administración puede realizar llamadas de administración a Windows Virtual Desktop en nombre del usuario que ha iniciado sesión en la herramienta.

> [!div class="mx-imgBorder"]
> ![Captura de pantalla en la que se muestran los permisos proporcionados al dar su consentimiento a la herramienta de administración de la interfaz de usuario.](../media/management-ui-delegated-permissions.png)

Para determinar qué usuario puede usarse para iniciar sesión en la herramienta, vaya a la [página de configuración de usuario de Azure Active Directory](https://portal.azure.com/#blade/Microsoft_AAD_IAM/StartboardApplicationsMenuBlade/UserSettings/menuId/) y tome nota del valor de **Los usuarios pueden permitir que las aplicaciones accedan a los datos de la compañía en su nombre**.

> [!div class="mx-imgBorder"]
> ![Captura de pantalla en la que se muestra si los usuarios pueden otorgar consentimiento a las aplicaciones solo para su usuario.](../media/management-ui-user-consent-allowed.png)

- Si el valor está establecido en **Sí**, puede iniciar sesión con cualquier cuenta de usuario en Azure Active Directory y dar su consentimiento solo para ese usuario. Sin embargo, si más adelante inicia sesión en la herramienta de administración con un usuario diferente, debe otorgar nuevamente el mismo consentimiento.
- Si el valor está establecido en **No**, debe iniciar sesión como administrador global en Azure Active Directory y proporcionar el consentimiento del administrador para todos los usuarios en el directorio. Ningún otro usuario se encontrará con una petición de consentimiento.


Una vez que decida qué usuario usará para dar su consentimiento, siga estas instrucciones para otorgar consentimiento a la herramienta:

1. Vaya a los recursos de Azure, seleccione el recurso de Azure App Services con el nombre que especificó en la plantilla (por ejemplo, Apr3UX) y vaya a la dirección URL asociada a él; por ejemplo, `https://rdmimgmtweb-210520190304.azurewebsites.net`.
2. Inicie sesión con la cuenta de usuario de Azure Active Directory adecuada.
3. Si se ha autenticado con un administrador global, ahora puede marcar la casilla de verificación para otorgar su **Consentimiento en nombre de la organización**. Seleccione **Aceptar** para dar su consentimiento.

   > [!div class="mx-imgBorder"]
   > ![Captura de pantalla en la que se muestra la página completa de consentimiento que verá el usuario o administrador.](../media/management-ui-consent-page.png)

Ahora se abrirá la herramienta de administración.

## <a name="use-the-management-tool"></a>Uso de la herramienta de administración

Después de proporcionar su consentimiento para la organización o para un usuario específico, puede tener acceso a la herramienta de administración en cualquier momento.

Siga estas instrucciones para iniciar la herramienta:

1. Seleccione el recurso de Azure App Services con el nombre proporcionado en la plantilla (por ejemplo, Apr3UX) y vaya a la dirección URL asociada a él. Por ejemplo, <https://rdmimgmtweb-210520190304.azurewebsites.net>.
2. Inicie sesión con sus credenciales de Windows Virtual Desktop.
3. Cuando se le pida que elija un grupo de inquilinos, seleccione **Default Tenant Group** (grupo de inquilinos predeterminado) en la lista desplegable.
4. Al seleccionar **Default Tenant Group** (grupo de inquilinos predeterminado), debería aparecer un menú en la parte izquierda de la ventana. En este menú, busque el nombre de su grupo de inquilinos y selecciónelo.

  > [!NOTE]
  > Si tiene un grupo de inquilinos personalizado, escriba el nombre manualmente en lugar de elegirlo en la lista desplegable.

## <a name="report-issues"></a>Informar de problemas

Si experimenta algún problema con la herramienta de administración o con cualquier otra herramienta de Windows Virtual Desktop, siga las instrucciones que encontrará en el artículo sobre [plantillas de Azure Resource Manager para Servicios de Escritorio remoto](https://github.com/Azure/RDS-Templates/blob/master/README.md) para notificarlo en GitHub.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido a implementar y conectarse a la herramienta de administración, puede obtener más información sobre cómo usar Azure Service Health para supervisar problemas de servicio y avisos de estado. Para más información, vea nuestro tutorial [Configuración de alertas de servicio](set-up-service-alerts-2019.md).