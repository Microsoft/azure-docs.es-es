---
title: 'Creación de contraseñas de aplicaciones desde la página Información de seguridad (versión preliminar): Azure AD'
description: Cree contraseñas generadas automáticamente (contraseñas de aplicaciones) para utilizarlas con cualquier aplicación sin explorador o que no admita la verificación en dos fases en la organización. Esta contraseña de aplicación es independiente de una contraseña normal y se puede configurar desde la página Información de seguridad.
services: active-directory
author: curtand
manager: daveba
ms.reviewer: sahenry
ms.service: active-directory
ms.workload: identity
ms.subservice: user-help
ms.topic: end-user-help
ms.date: 02/13/2018
ms.author: curtand
ms.openlocfilehash: 9de054162a59b10fbd02bed9b3e84634ee12125b
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88799203"
---
# <a name="create-app-passwords-from-the-security-info-preview-page"></a>Creación de contraseñas de aplicaciones desde la página Información de seguridad (versión preliminar)

Determinadas aplicaciones, como Outlook 2010, no admiten la verificación en dos pasos. Esta falta de compatibilidad significa que si usa la verificación en dos pasos en la organización, la aplicación no funcionará. Para solucionar este problema, puede crear una contraseña generada automáticamente para usar con cada aplicación sin explorador, independiente de la contraseña normal.

[!INCLUDE [preview-notice](../../../includes/active-directory-end-user-preview-notice-security-info.md)]

>[!Important]
>Es posible que el administrador no le permita utilizar contraseñas de aplicaciones. Si no ve **Contraseñas de aplicaciones** como una opción, no están disponibles en la organización.

Al utilizar las contraseñas de aplicaciones, es importante recordar:

- Las contraseñas de aplicaciones se generan automáticamente y deben crearse y escribirse una sola vez por cada aplicación.

- Hay un límite de 40 contraseñas por usuario. Si intenta crear una después de ese límite, se le pedirá que elimine una contraseña existente antes de que se le permita crear una nueva.

    >[!Note]
    >Los clientes de Office 2013 (incluido Outlook) admiten nuevos protocolos de autenticación que se pueden usar con la verificación en dos pasos. Esta compatibilidad significa que después de activar la verificación en dos pasos, ya no necesitará contraseñas de aplicaciones para clientes de Office 2013. Para obtener más información, consulte el artículo [Cómo funciona la autenticación moderna para las aplicaciones de cliente de Office 2013 y Office 2016](https://support.office.com/article/how-modern-authentication-works-for-office-2013-and-office-2016-client-apps-e4c45989-4b1a-462e-a81b-2a13191cf517).

## <a name="create-new-app-passwords"></a>Creación de una nueva contraseña de aplicaciones

Si utiliza la verificación en dos pasos con su cuenta profesional o educativa y su administrador ha activado la experiencia de información de seguridad, puede crear y eliminar las contraseñas de aplicaciones mediante la página **Información de seguridad**.

>[!Note]
>Si el administrador no ha activado la experiencia de información de seguridad, debe seguir las instrucciones y la información de la sección [¿Qué son las contraseñas de aplicación en Azure Multi-Factor Authentication?](multi-factor-authentication-end-user-app-passwords.md).

### <a name="to-create-a-new-app-password"></a>Para crear una nueva contraseña de aplicación

1. Inicie sesión en su cuenta profesional o educativa y vaya a la página https://myaccount.microsoft.com/.

    ![Página Mi Perfil, que muestra los vínculos de Información de seguridad resaltados](media/security-info/securityinfo-myprofile-page.png)

2. Seleccione **Información de seguridad** en el panel de navegación izquierdo o en el vínculo del bloque **Información de seguridad** y, después, seleccione **Agregar método** en la página **Información de seguridad**.

    ![Página Información de seguridad con la opción Agregar método resaltada](media/security-info/securityinfo-myprofile-addmethod-page.png)

3. En la página **Agregar un método**, seleccione **Agregar contraseña** en la lista desplegable y, a continuación, seleccione **Agregar**.

    ![Cuadro Agregar un método, con la contraseña de aplicaciones seleccionada](media/security-info/securityinfo-myprofile-addpassword.png)

4. Escriba el nombre de la aplicación que requiere la contraseña de la aplicación y, a continuación, seleccione **Siguiente**.

    ![Página de contraseña de aplicación, con el nombre de la aplicación](media/security-info/securityinfo-myprofile-password-appname.png)

5. Copie el texto del cuadro **Contraseña**, pegue la contraseña en el área de contraseña de aplicaciones (en este ejemplo, Outlook 2010) y, a continuación, seleccione **Listo**.

    ![Página de contraseña de aplicación, con el nombre de la aplicación](media/security-info/securityinfo-myprofile-password-copytext.png)

    Se agrega la contraseña y puede iniciar sesión correctamente en la aplicación en el futuro.

## <a name="delete-your-app-passwords"></a>Eliminación de las contraseñas de aplicaciones

Si ya no necesita utilizar una aplicación que requiera una contraseña de aplicaciones, puede eliminar la contraseña de aplicaciones asociada. Si se elimina la contraseña de aplicaciones, se libera uno de los puntos de contraseñas de aplicaciones disponibles para su uso en el futuro.

>[!Important]
>Si elimina una contraseña de aplicación por error, no existe modo alguno de deshacerlo. Tendrá que crear una nueva contraseña de aplicación y volver a introducirla en la aplicación. Para ello, siga los pasos de la sección [Creación de una nueva contraseña de aplicaciones](#create-new-app-passwords) de este artículo.

### <a name="to-delete-an-app-password"></a>Para eliminar una contraseña de aplicación

1. En la página **Información de seguridad**, seleccione el vínculo **Eliminar** situado junto a la opción **Contraseña de aplicación** para la aplicación específica.

    ![Vínculo para eliminar el método de la contraseña de aplicación de la información de seguridad](media/security-info/securityinfo-myprofile-password-appdelete.png)

2. Seleccione **Sí** en el cuadro de confirmación para eliminar la **contraseña de aplicación**. Una vez eliminada la contraseña de aplicación, se quitará de su información de seguridad y desaparecerá de la página **Información de seguridad**.

## <a name="for-more-information"></a>Para obtener más información

- Para más información sobre la página **Información de seguridad** y sobre cómo configurarla, consulte [Introducción a la información de seguridad](./security-info-setup-signin.md).