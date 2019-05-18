---
title: Crear un inquilino de Azure AD para Azure Red Hat OpenShift | Microsoft Docs
description: Aquí le mostramos cómo crear un inquilino de Azure Active Directory (Azure AD) para hospedar el clúster de Microsoft Azure Red Hat OpenShift.
author: tylermsft
ms.author: twhitney
ms.service: container-service
manager: jeconnoc
ms.topic: conceptual
ms.date: 05/13/2019
ms.openlocfilehash: 04d710f4d60b776f8059d87ea4d009bed6f7f8ba
ms.sourcegitcommit: f013c433b18de2788bf09b98926c7136b15d36f1
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2019
ms.locfileid: "65551706"
---
# <a name="create-an-azure-ad-tenant-for-azure-red-hat-openshift"></a>Crear a un inquilino de Azure AD para Azure Red Hat OpenShift

Microsoft Azure Red Hat OpenShift requiere un [Azure Active Directory (Azure AD)](https://docs.microsoft.com/azure/active-directory/develop/quickstart-create-new-tenant) inquilino en el que se va a crear el clúster. Un *inquilino* es una instancia dedicada de Azure AD que un desarrollador de la organización o una aplicación recibe cuando crea una relación con Microsoft registrándose en Azure, Microsoft Intune o Microsoft 365. Cada inquilino de Azure AD es distinto y distinto de Azure AD a los inquilinos y tiene su propio trabajo y las identidades del centro educativo y registros de aplicaciones.

Si aún no tiene un inquilino de Azure AD, siga estas instrucciones para crear uno.

## <a name="create-a-new-azure-ad-tenant"></a>Creación de un nuevo inquilino de Azure AD

Para crear a un inquilino:

1. Inicie sesión en el [portal Azure](https://portal.azure.com/) mediante la cuenta que desea asociar con el clúster de Azure Red Hat OpenShift.
2. Abra el [hoja Azure Active Directory](https://portal.azure.com/#create/Microsoft.AzureActiveDirectory) para crear un nuevo inquilino (también conocido como un nuevo *Azure Active Directory*).
3. Proporcione un **nombreDeOrganización**.
4. Proporcione un **nombre de dominio inicial**. Esto tendrá *onmicrosoft.com* anexado a él. Puede volver a usar el valor de *nombreDeOrganización* aquí.
5. Elija un país o región donde se creará el inquilino.
6. Haga clic en **Create**(Crear).
7. Una vez creado el inquilino de Azure AD, seleccione el **haga clic aquí para administrar el nuevo directorio** vínculo. El nuevo nombre del inquilino se debe mostrar la parte superior derecha de Azure portal:  

    ![Captura de pantalla del portal que muestra el nombre del inquilino en la esquina superior derecha][tenantcallout]  

8. Tome nota de la *Id. de inquilino* , más adelante, que permite especificar dónde desea crear el clúster de Azure Red Hat OpenShift. En el portal, ahora debería ver la hoja de información general de Azure Active Directory en el nuevo inquilino. Seleccione **propiedades** y copie el valor para su **Id. de directorio**. Nos referiremos a este valor como `TENANT` en el [crear un clúster de Azure Red Hat OpenShift](tutorial-create-cluster.md) tutorial.

[tenantcallout]: ./media/howto-create-tenant/tenant-callout.png

## <a name="resources"></a>Recursos

Desproteger [documentación de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/) para obtener más información sobre [inquilinos de Azure AD](https://docs.microsoft.com/azure/active-directory/develop/quickstart-create-new-tenant).

## <a name="next-steps"></a>Pasos siguientes

Aprenda a crear a una entidad de servicio, generar una URL de devolución de llamada de secreto y autenticación de cliente y crear un nuevo usuario de Active Directory para probar aplicaciones en un clúster de Azure Red Hat OpenShift.

[Crear un objeto de aplicación de Azure AD y el usuario](howto-aad-app-configuration.md)