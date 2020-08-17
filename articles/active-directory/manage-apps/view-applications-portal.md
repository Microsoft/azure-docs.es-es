---
title: 'Inicio rápido: Visualización de la lista de aplicaciones que usan el inquilino de Azure Active Directory (Azure AD) para la administración de identidades'
description: En este inicio rápido, use Azure Portal para ver la lista de aplicaciones registradas para usar el inquilino de Azure Active Directory (Azure AD) para la administración de identidades.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: quickstart
ms.date: 04/09/2019
ms.author: kenwith
ms.reviewer: arvinh
ms.custom: it-pro
ms.openlocfilehash: dce579596563855022f14bb2c07353b84d1e6127
ms.sourcegitcommit: 85eb6e79599a78573db2082fe6f3beee497ad316
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87808378"
---
# <a name="quickstart-view-the-list-of-applications-that-are-using-your-azure-active-directory-azure-ad-tenant-for-identity-management"></a>Inicio rápido: Visualización de la lista de aplicaciones que usan el inquilino de Azure Active Directory (Azure AD) para la administración de identidades

Comience a usar Azure AD como sistema de Administración de identidad y acceso (IAM) de las aplicaciones que usa su organización. En este inicio rápido, verá las aplicaciones que ya están configuradas para usar el inquilino de Azure AD como proveedor de identidades.

## <a name="prerequisites"></a>Requisitos previos

Para ver las aplicaciones que se han registrado en el inquilino de Azure AD, necesita:

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

>[!IMPORTANT]
>Para probar los pasos de este inicio rápido, se recomienda usar un entorno que no sea de producción.

## <a name="find-the-list-of-applications-in-your-tenant"></a>Búsqueda de la lista de aplicaciones del inquilino

Las aplicaciones registradas con el inquilino de Azure AD están visibles en la sección **Aplicaciones empresariales** de Azure Portal.

Para ver las aplicaciones registradas en el inquilino:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. En el panel de navegación izquierdo, seleccione **Azure Active Directory**.
3. En el panel **Azure Active Directory**, seleccione **Aplicaciones empresariales**.
4. En el menú desplegable **Tipo de aplicación**, seleccione **Todas las aplicaciones** y elija **Aplicar**. Aparece una muestra aleatoria de las aplicaciones del inquilino.
5. Para ver más aplicaciones, seleccione **Cargar más** en la parte inferior de la lista. Si hay varias aplicaciones en el inquilino, puede resultar más fácil buscar una aplicación específica en lugar de desplazarse por la lista. La búsqueda de una aplicación específica se trata más adelante en este inicio rápido.

## <a name="select-viewing-options"></a>Selección de las opciones de visualización

Seleccione las opciones en función de lo que busque.

1. Puede ver las aplicaciones por **tipo de aplicación**, **estado de aplicación** y **visibilidad de aplicación**.
2. En **Tipo de aplicación**, elija una de estas opciones:
    - **Aplicaciones empresariales** muestra las aplicaciones que no son de Microsoft.
    - **Aplicaciones de Microsoft** muestra las aplicaciones de Microsoft.
    - **Todas las aplicaciones** muestra las aplicaciones de Microsoft y las que no son de Microsoft.
3. En **Estado de la aplicación**, elija **Cualquiera**, **Deshabilitada** o **Habilitada**. La opción **Cualquiera** incluye tanto aplicaciones habilitadas como deshabilitadas.
4. En **Visibilidad de la aplicación**, elija **Cualquiera** u **Oculta**. La opción **Ocultas** muestra las aplicaciones que están en el inquilino, pero que los usuarios no pueden ver.
5. Después de elegir las opciones que desee, seleccione **Aplicar**.

## <a name="search-for-an-application"></a>Búsqueda de una aplicación

Para buscar una aplicación concreta:

1. En el menú **Tipo de aplicación**, seleccione **Todas las aplicaciones** y elija **Aplicar**.
2. Escriba el nombre de la aplicación que desea buscar. Si la aplicación se ha agregado al inquilino de Azure AD, aparece en los resultados de la búsqueda. En este ejemplo, se muestra que GitHub no se ha agregado a las aplicaciones del inquilino.
    ![En el ejemplo se muestra que una aplicación no se ha agregado al inquilino](media/view-applications-portal/search-for-tenant-application.png).
3. Pruebe a escribir las primeras letras del nombre de una aplicación. En este ejemplo se muestran todas las aplicaciones que empiezan por **Sales**.
    ![En el ejemplo se muestran todas las aplicaciones que empiezan por Sales](media/view-applications-portal/search-by-prefix.png).


> [!TIP]
> La administración de aplicaciones se puede automatizar mediante Graph API, consulte el artículo sobre la [automatización de la administración de aplicaciones con Microsoft Graph API](https://docs.microsoft.com/graph/application-saml-sso-configure-api).


## <a name="clean-up-resources"></a>Limpieza de recursos

En este inicio rápido no se han creado recursos, por lo que no hay nada que limpiar.

## <a name="next-steps"></a>Pasos siguientes

Pase al siguiente artículo, donde aprenderá a usar Azure AD como proveedor de identidades para una aplicación.
> [!div class="nextstepaction"]
> [Adición de una aplicación](add-application-portal.md)
