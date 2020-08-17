---
title: Inicio de sesión único en aplicaciones mediante el proxy de aplicación de Azure AD | Microsoft Docs
description: Active el inicio de sesión único para sus aplicaciones locales publicadas mediante el proxy de aplicación de Azure AD en Azure Portal.
services: active-directory
documentationcenter: ''
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: how-to
ms.date: 11/12/2018
ms.author: kenwith
ms.reviewer: japere
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 00b25f1b506228d2d80aecae3845467759a4bc4f
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88165046"
---
# <a name="password-vaulting-for-single-sign-on-with-application-proxy"></a>Almacén de contraseñas para el inicio de sesión único con el proxy de aplicación

El proxy de aplicación de Azure Active Directory le ayuda a mejorar la productividad mediante la publicación de aplicaciones locales de manera que los empleados remotos puedan tener acceso seguro a estas. En Azure Portal, puede configurar también el inicio de sesión único (SSO) para estas aplicaciones. Los usuarios solo necesitan autenticarse con Azure AD y podrán acceder a la aplicación de empresa sin necesidad de volver a iniciar sesión.

El proxy de aplicación admite varios [modos de inicio de sesión único](sso-options.md#choosing-a-single-sign-on-method). El inicio de sesión basado en contraseña se ha creado para aplicaciones que usan una combinación de nombre de usuario/contraseña para la autenticación. Al configurar el inicio de sesión basado en contraseña para su aplicación, los usuarios tendrán que iniciar sesión en la aplicación local una vez. Después de eso, Azure Active Directory almacena la información de inicio de sesión y la proporciona automáticamente a la aplicación cuando los usuarios acceden a ella de forma remota.

Debe haber publicado y probado ya la aplicación con el proxy de aplicación. Si no es así, siga los pasos de [Publish applications using Azure AD Application Proxy](application-proxy-add-on-premises-application.md) (Publicación de aplicaciones mediante el proxy de aplicación de Azure AD) y luego vuelva a este punto.

## <a name="set-up-password-vaulting-for-your-application"></a>Configuración del almacenamiento de contraseñas para la aplicación

1. Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador.
1. Seleccione **Azure Active Directory** > **Aplicaciones empresariales** > **Todas las aplicaciones**.
1. En la lista, seleccione la aplicación para la que desea configurar el SSO.  
1. Seleccione **Proxy de la aplicación**. 
1. Cambie el valor de **Tipo de preautenticación** a **Acceso directo** y seleccione **Guardar**. Más adelante podrá volver al tipo **Azure Active Directory**. 
1. Seleccione **Inicio de sesión único**.

   ![Selección de inicio de sesión único desde la página de introducción de la aplicación](./media/application-proxy-configure-single-sign-on-password-vaulting/select-sso.png)

1. Para el modo SSO, elija **Inicio de sesión basado en contraseña**.
1. Para la URL de inicio de sesión, escriba la dirección URL de la página donde los usuarios escriben su nombre de usuario y contraseña para iniciar sesión fuera de la red corporativa. Debe ser la dirección URL externa que creó al publicar la aplicación a través del proxy de aplicación.

   ![Elegir inicio de sesión basado en contraseña y escribir la URL](./media/application-proxy-configure-single-sign-on-password-vaulting/password-sso.png)

1. Seleccione **Guardar**.
1. Seleccione **Proxy de la aplicación**. 
1. Cambie el valor de **Tipo de preautenticación** a **Azure Active Directory** y seleccione **Guardar**. 
1. Seleccione **Usuarios y grupos**.
1. Asigne usuarios a la aplicación con la selección de **Agregar usuario**. 
1. Si desea predefinir las credenciales de un usuario, active la casilla del nombre de usuario y seleccione **Actualizar credenciales**.
1. Seleccione **Azure Active Directory** > **Registros de aplicaciones** > **Todas las aplicaciones**.
1. En la lista, seleccione la aplicación que ha configurado con el inicio de sesión único con contraseña.
1. Seleccione la **personalización de marca**. 
1. Actualice la **dirección URL de la página principal** con la **dirección URL de inicio de sesión** de la página de inicio de sesión único con contraseña y seleccione **Guardar**.  



<!-- Need to repro?
7. The page should tell you that a sign-in form was successfully detected at the provided URL. If it doesn't, select **Configure [your app name] Password Single Sign-on Settings** and choose **Manually detect sign-in fields**. Follow the instructions to point out where the sign-in credentials go. 
-->

## <a name="test-your-app"></a>Prueba de la aplicación

Vaya al portal Aplicaciones. Inicie sesión con sus credenciales (o las credenciales de una cuenta de prueba que configuró con acceso). Después de iniciar sesión correctamente, haga clic en el icono de la aplicación. Al hacerlo, podría desencadenar la instalación de la extensión del explorador para el inicio de sesión seguro de Aplicaciones. Si el usuario tenía credenciales predefinidas, la autenticación para la aplicación debería realizarse automáticamente; de lo contrario, debe especificar el nombre de usuario o la contraseña la primera vez. 

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre otras maneras de implementar el [inicio de sesión único](what-is-single-sign-on.md)
- Más información sobre [Consideraciones de seguridad al obtener acceso a aplicaciones de forma remota con el Proxy de aplicación de Azure AD](application-proxy-security.md)
