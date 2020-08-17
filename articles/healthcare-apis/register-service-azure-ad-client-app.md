---
title: 'Registro de una aplicación de servicio en Azure AD: Azure API for FHIR'
description: Aprenda a registrar una aplicación cliente de servicio en Azure Active Directory que se puede usar para autenticar y obtener tokens.
services: healthcare-apis
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: conceptual
ms.date: 02/07/2019
ms.author: matjazl
ms.openlocfilehash: 34eec3ad0d2fc193744898b6f08cbe50c261c945
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87853031"
---
# <a name="register-a-service-client-application-in-azure-active-directory"></a>Registro de una aplicación cliente de servicio en Azure Active Directory

En este artículo aprenderá a registrar una aplicación cliente de servicio en Azure Active Directory. Los registros de aplicaciones cliente son representaciones de Azure Active Directory de aplicaciones que se pueden utilizar para autenticar y obtener tokens. Un cliente de servicio está destinado a ser utilizado por una aplicación para obtener un token de acceso sin la autenticación interactiva de un usuario. Tendrá ciertos permisos de aplicación y utilizará un secreto de aplicación (contraseña) al obtener los tokens de acceso.

Siga los pasos descritos a continuación para crear un nuevo cliente de servicio.

## <a name="app-registrations-in-azure-portal"></a>Registros de aplicaciones en Azure Portal

1. En el panel de navegación izquierdo de [Azure Portal](https://portal.azure.com), haga clic en **Azure Active Directory**.

2. En la hoja **Azure Active Directory**, haga clic en **Registros de aplicaciones**:

    ![Azure Portal. Nuevo registro de aplicaciones.](media/how-to-aad/portal-aad-new-app-registration.png)

3. Haga clic en **Nuevo registro**.

## <a name="service-client-application-details"></a>Detalles de la aplicación cliente de servicio

* El cliente de servicio necesita un nombre para mostrar y también puede proporcionar una dirección URL de respuesta, pero normalmente no se utilizará.

    ![Azure Portal. Nuevo registro de aplicaciones de cliente de servicio.](media/how-to-aad/portal-aad-register-new-app-registration-SERVICE-CLIENT-NAME.png)

## <a name="api-permissions"></a>Permisos de API

Deberá conceder los roles de aplicación cliente de servicio. 

1. Abra **Permisos de API** y seleccione su [registro de aplicaciones de recursos de API de FHIR](register-resource-azure-ad-client-app.md). Si usa Azure API for FHIR, agregará un permiso a las API de salud de Azure mediante la búsqueda de dichas API en **API usadas en mi organización**.

    ![Azure Portal. Permisos de la API del cliente de servicio](media/how-to-aad/portal-aad-register-new-app-registration-SERVICE-CLIENT-API-PERMISSIONS.png)

2. Seleccione los roles de aplicación que desee entre los que están definidos en la aplicación de recursos:

    ![Azure Portal. Permisos de aplicación cliente de servicio](media/how-to-aad/portal-aad-register-new-app-registration-SERVICE-CLIENT-APPLICATION-PERMISSIONS.png)

3. Dé su consentimiento a la aplicación. Si no tiene los permisos necesarios, consulte con el administrador de Azure Active Directory:

    ![Azure Portal. Consentimiento del administrador de cliente de servicio](media/how-to-aad/portal-aad-register-new-app-registration-SERVICE-CLIENT-ADMIN-CONSENT.png)

## <a name="application-secret"></a>Secreto de aplicación

El cliente de servicio necesita un secreto (contraseña), que se usa al obtener los tokens.

1. Haga clic en **Certificados&amp; y secretos**.

2. Haga clic en **Nuevo secreto de cliente**.

    ![Azure Portal. Secreto de cliente de servicio](media/how-to-aad/portal-aad-register-new-app-registration-SERVICE-CLIENT-SECRET.png)

3. Proporcione una duración del secreto.

4. Cuando se ha generado, solo se mostrará una vez en el portal. Tome nota del mismo y guárdelo de forma segura.

## <a name="next-steps"></a>Pasos siguientes

En este artículo ha aprendido cómo registrar una aplicación cliente de servicio en Azure Active Directory. A continuación, implemente una API FHIR en Azure.
 
>[!div class="nextstepaction"]
>[Implementación de FHIR Server de código abierto](fhir-oss-powershell-quickstart.md)