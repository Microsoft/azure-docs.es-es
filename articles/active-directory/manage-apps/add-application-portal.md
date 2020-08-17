---
title: 'Inicio rápido: Incorporación de una aplicación al inquilino de Azure Active Directory (Azure AD)'
description: Esta guía de inicio rápido usa Azure Portal para agregar una aplicación de la galería a su inquilino de Azure Active Directory (Azure AD).
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.topic: quickstart
ms.workload: identity
ms.date: 10/29/2019
ms.author: kenwith
ms.openlocfilehash: 0818ab782710e6a102d2034790ff8d997cd54f8e
ms.sourcegitcommit: 85eb6e79599a78573db2082fe6f3beee497ad316
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87808446"
---
# <a name="quickstart-add-an-application-to-your-azure-active-directory-azure-ad-tenant"></a>Inicio rápido: Incorporación de una aplicación al inquilino de Azure Active Directory (Azure AD)

Azure Active Directory (Azure AD) incluye una galería que contiene miles de aplicaciones previamente integradas. Muchas de las aplicaciones que su organización usa probablemente estén ya en la galería.

Después de agregar una aplicación a su inquilino de Azure AD, puede:

- Configurar las propiedades de la aplicación.
- Administrar el acceso de usuario a la aplicación con una directiva de acceso condicional.
- Configurar el inicio de sesión único para que los usuarios puedan iniciar sesión en la aplicación con sus credenciales de Azure AD.

## <a name="prerequisites"></a>Requisitos previos

Para agregar una aplicación al inquilino de Azure AD, necesita:

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Uno de los siguientes roles: Administrador global, Administrador de aplicaciones en la nube, Administrador de aplicaciones o Propietario de la entidad de servicio.
- (Opcional: finalización de [Visualización de sus aplicaciones](view-applications-portal.md)).

>[!IMPORTANT]
>Para probar los pasos de este inicio rápido, se recomienda usar un entorno que no sea de producción.

## <a name="add-an-app-to-your-azure-ad-tenant"></a>Incorporación de una aplicación al inquilino de Azure AD

Para agregar una aplicación a un inquilino de Azure AD:

1. En [Azure Portal](https://portal.azure.com), en el panel de navegación izquierdo, seleccione **Azure Active Directory**.
2. En el panel **Azure Active Directory**, seleccione **Aplicaciones empresariales**. Se abre el panel **Todas las aplicaciones**, en el que se ve una muestra aleatoria de las aplicaciones que hay en su inquilino de Azure AD.
3. En el panel **Aplicaciones empresariales**, seleccione **Nueva aplicación**. 
    ![Selección de Nueva aplicación para agregar una aplicación de la galería al inquilino](media/add-application-portal/new-application.png)
4. Cambie a la nueva experiencia de versión preliminar de la galería: En el banner situado en la parte superior de la **página para agregar una aplicación**, seleccione el vínculo que indica **Haga clic aquí para probar la nueva y mejorada experiencia de la galería de aplicaciones.** .
5. Se abre el panel **Examinar la Galería de Azure AD (versión preliminar)** , que muestra iconos para plataformas en la nube, aplicaciones locales y aplicaciones destacadas. Las aplicaciones que aparecen en la sección **Aplicaciones destacadas** incluyen iconos que indican si admiten el inicio de sesión único federado (SSO) y el aprovisionamiento.
    ![Búsqueda de una aplicación por nombre o categoría](media/add-application-portal/browse-gallery.png)
6. Puede buscar en la galería la aplicación que quiere agregar o escribir su nombre en el cuadro de búsqueda para localizarla. Después, seleccione la aplicación en los resultados. En el formulario, puede editar el nombre de la aplicación para que se ajuste a las necesidades de su organización. En este ejemplo, hemos seleccionado GitHub y hemos cambiado el nombre a **GitHub-test**.
    ![Se muestra cómo agregar una aplicación de la galería](media/add-application-portal/create-application.png).
    >[!TIP]
    >Si la aplicación que está buscando no está en la galería, puede hacer clic en el vínculo **Cree su propia aplicación** y, a continuación, en **¿Cuál es el objetivo de utilizar la aplicación?** elija **Integrar cualquier otra aplicación que no se encuentre en la galería**. Microsoft ha trabajado con muchos desarrolladores de aplicaciones para configurar estas previamente con el fin de que funcionen con Azure AD. Estas son las aplicaciones que se muestran en la galería. Pero si la aplicación que quiere agregar no aparece en la lista, puede crear una nueva, genérica y configurarla usted mismo o siguiendo las instrucciones del desarrollador que la haya creado.
7. Seleccione **Crear**. Aparece una página de introducción con las opciones para configurar la aplicación para su organización.

Ha terminado de agregar una aplicación. El siguiente inicio rápido muestra cómo cambiar el logotipo y modificar otras propiedades de la aplicación.

> [!TIP]
> La administración de aplicaciones se puede automatizar mediante Graph API, consulte el artículo sobre la [automatización de la administración de aplicaciones con Microsoft Graph API](https://docs.microsoft.com/graph/application-saml-sso-configure-api).

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no va a continuar con la serie de inicios rápidos, considere la posibilidad de eliminar la aplicación para limpiar el inquilino de prueba. La eliminación de la aplicación se trata en el último inicio rápido de esta serie, consulte [Eliminación de una aplicación](delete-application-portal.md).

## <a name="next-steps"></a>Pasos siguientes

Pase al siguiente artículo, donde aprenderá a configurar una aplicación.
> [!div class="nextstepaction"]
> [Configuración de una aplicación](add-application-portal-configure.md)
