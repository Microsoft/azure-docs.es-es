---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Maxient Conduct Manager Software | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Maxient Conduct Manager Software.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 12/18/2019
ms.author: jeedes
ms.openlocfilehash: dd6872fd8dca3f29b61c6f1dffb5f219abac5cb8
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88518905"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-maxient-conduct-manager-software"></a>Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Maxient Conduct Manager Software

En este tutorial, obtendrá información sobre cómo integrar Maxient Conduct Manager Software con Azure Active Directory (Azure AD). Al integrar Maxient Conduct Manager Software con Azure AD, puede:

* Utilizar Azure AD a fin de autenticar a todos los usuarios para Maxient Conduct Manager Software.
* Permitir que los usuarios inicien sesión automáticamente en Maxient Conduct Manager Software con sus cuentas de Azure AD.


Para más información sobre la integración de aplicaciones SaaS con Azure AD, consulte [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

## <a name="prerequisites"></a>Prerrequisitos

Para empezar, necesita los siguientes elementos:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción habilitada para el inicio de sesión único (SSO) en Maxient Conduct Manager Software.

## <a name="scenario-description"></a>Descripción del escenario

En este tutorial, configurará Azure AD para usarlo con Maxient Conduct Manager Software.


* Maxient Conduct Manager Software admite el inicio de sesión único iniciado por **SP e IDP**

> [!NOTE]
> El identificador de esta aplicación es un valor de cadena fijo, por lo que solo se puede configurar una instancia en un inquilino.

## <a name="adding-maxient-conduct-manager-software-from-the-gallery"></a>Adición Maxient Conduct Manager Software desde la galería

Para configurar la integración de Maxient Conduct Manager Software en Azure AD, deberá agregarlo desde la galería a la lista de aplicaciones SaaS administradas.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. En el panel de navegación de la izquierda, seleccione el servicio **Azure Active Directory**.
1. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.
1. Para agregar una nueva aplicación, seleccione **Nueva aplicación**.
1. En la sección **Agregar desde la galería**, escriba **Maxient Conduct Manager Software** en el cuadro de búsqueda.
1. Seleccione **Maxient Conduct Manager Software** en el panel de resultados y agregue la aplicación. Espere unos segundos mientras la aplicación se agrega al inquilino.


## <a name="configure-and-test-azure-ad-single-sign-on-for-maxient-conduct-manager-software"></a>Configuración y prueba del inicio de sesión único de Azure AD para Maxient Conduct Manager Software

Configure y pruebe del inicio de sesión único de Azure AD con Maxient Conduct Manager Software. Para que el inicio de sesión único funcione, es necesario establecer una vinculación entre un usuario de Azure AD y el usuario correspondiente de Maxient Conduct Manager Software.

Para configurar y probar el inicio de sesión único de Azure AD con Maxient Conduct Manager Software, complete los siguientes bloques de creación:

1. **[Configuración del inicio de sesión único de Azure AD](#configure-azure-ad-sso)** : para permitir que los usuarios se autentiquen con el fin de utilizar Maxient Conduct Manager Software.
    1. **[Asignación de todos los usuarios que van a utilizar Maxient](#assign-all-users-to-be-able-to-authenticate-for-the-maxient-conduct-manager-software)** : para permitir que todos los usuarios de su institución puedan autenticarse.
1. **[Prueba de la configuración de Azure AD con Maxient](#test-with-maxient)** : para comprobar si la configuración funciona y se han lanzado los atributos correctos.

## <a name="configure-azure-ad-sso"></a>Configuración del inicio de sesión único de Azure AD

Siga estos pasos para habilitar el inicio de sesión único de Azure AD en Azure Portal.

1. En [Azure Portal](https://portal.azure.com/), en la página de integración de aplicaciones de **Maxient Conduct Manager Software**, busque la sección **Administrar** y seleccione **Inicio de sesión único**.
1. En la página **Seleccione un método de inicio de sesión único**, elija **SAML**.
1. En la página **Configurar el inicio de sesión único con SAML**, haga clic en el icono de edición o con forma de lápiz para abrir el cuadro de diálogo **Configuración básica de SAML** y modificar la configuración.

   ![Edición de la configuración básica de SAML](common/edit-urls.png)

1. En la sección  **Configuración básica de SAML** , la aplicación está preconfigurada en el modo iniciado por  **IDP**  y las direcciones URL necesarias ya se han rellenado previamente con Azure. El usuario debe guardar la configuración, para lo que debe hacer clic en el botón  **Guardar** .

1. Haga clic en **Establecer direcciones URL adicionales** y siga este paso si desea configurar la aplicación en el modo iniciado por **SP**:

    En el cuadro de texto **URL de inicio de sesión**, escriba una dirección URL con el siguiente patrón: `https://cm.maxient.com/<SCHOOLCODE>`

    > [!NOTE]
    > Este valor no es real. Actualícelo con la dirección URL de inicio de sesión real. Consulte al representante de soporte técnico o implementación de Maxient para obtener el valor.

1. En la página **Configurar el inicio de sesión único con SAML**, en la sección **Certificado de firma de SAML**, haga clic en el botón de copia para copiar la **Dirección URL de metadatos de federación de aplicación** y guárdela en su equipo.  Tendrá que proporcionar esta dirección URL al representante de soporte técnico o implementación de Maxient.

    ![Vínculo de descarga del certificado](common/copy-metadataurl.png)

### <a name="assign-all-users-to-be-able-to-authenticate-for-the-maxient-conduct-manager-software"></a>Asignación de todos los usuarios para que puedan autenticarse en Maxient Conduct Manager Software.

En esta sección, proporcionará acceso para que todas las cuentas se autentiquen con el sistema de Azure en Maxient Conduct Manager Software.  Es importante tener en cuenta que este paso es **OBLIGATORIO** para que Maxient funcione correctamente.  Maxient utiliza el sistema de Azure AD para *autenticar* a los usuarios. La *autorización* de usuarios se realiza en el sistema de Maxient para la función concreta que están intentando realizar. Maxient no usa atributos de su directorio para tomar esas decisiones.

1. En Azure Portal, seleccione sucesivamente **Aplicaciones empresariales** y **Todas las aplicaciones**.
1. En la lista de aplicaciones, seleccione **Maxient Conduct Manager Software**.
1. En la página de información general de la aplicación, busque la sección **Administrar** y seleccione **Usuarios y grupos**.

   ![Vínculo "Usuarios y grupos"](common/users-groups-blade.png)

1. Seleccione **Agregar usuario**. A continuación, en el cuadro de diálogo **Agregar asignación**, seleccione **Usuarios y grupos**.

    ![Vínculo de Agregar usuario](common/add-assign-user.png)

1. En el cuadro de diálogo **Usuarios y grupos**, seleccione todos los usuarios (o los grupos adecuados) y **asígnelos** para que puedan autenticarse con Maxient.

## <a name="test-with-maxient"></a>Prueba con Maxient 

Si aún no se ha abierto una incidencia de soporte técnico con un representante de soporte técnico o implementación de Maxient, envíe un correo electrónico a [support@maxient.com](mailto:support@maxient.com) con el asunto "Campus Based Authentication/Azure Setup - \<\<School Name\>\>". En el cuerpo del correo electrónico, proporcione la **dirección URL de metadatos de federación de aplicación**. El personal de Maxient responderá con un vínculo de prueba para comprobar que se están lanzando los atributos adecuados.  
    
## <a name="additional-resources"></a>Recursos adicionales

- [Lista de tutoriales acerca de cómo integrar aplicaciones SaaS con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [¿Qué es el acceso a las aplicaciones y el inicio de sesión único con Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [¿Qué es el acceso condicional en Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Pruebe Maxient Conduct Manager Software con Azure AD](https://aad.portal.azure.com/)

