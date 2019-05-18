---
title: Restricción de una aplicación registrada en Azure Active Directory a un conjunto de usuarios
description: Aprenda a restringir el acceso a las aplicaciones registradas en Azure AD a un conjunto de usuarios seleccionado.
services: active-directory
documentationcenter: ''
author: kalyankrishna1
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 09/24/2018
ms.author: kkrishna
ms.reviewer: ''
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: e07d9f0fa6ec6b4abc7ce96279b7b02faae298fa
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65540197"
---
# <a name="how-to-restrict-your-app-to-a-set-of-users"></a>Procedimientos para: Restricción de la aplicación a un conjunto de usuarios

Las aplicaciones registradas en un inquilino de Azure Active Directory (Azure AD) están disponibles de forma predeterminada para todos los usuarios del inquilino que se autentica correctamente.

Igualmente, en el caso de una aplicación [multiinquilino](howto-convert-app-to-be-multi-tenant.md), todos los usuarios del inquilino de Azure AD donde se aprovisiona esta aplicación podrán acceder a ella una vez que se autentiquen correctamente en sus respectivos inquilinos.

Los desarrolladores y administradores de inquilinos tienen con frecuencia el requisito de restringir una aplicación a un determinado conjunto de usuarios. Los desarrolladores pueden conseguir lo mismo mediante patrones de autorización conocidos, como el control de acceso basado en rol (RBAC), pero este enfoque exige una importante cantidad de trabajo por parte del desarrollador.

Azure AD permite a los administradores y desarrolladores de inquilinos restringir una aplicación a un conjunto específico de usuarios o a los grupos de seguridad del inquilino.

## <a name="supported-app-configurations"></a>Configuraciones de aplicación admitidas

La opción para restringir una aplicación a un conjunto específico de usuarios o a los grupos de seguridad de un inquilino funciona con los siguientes tipos de aplicaciones:

- Aplicaciones configuradas para el inicio de sesión federado con autenticación basada en SAML
- Aplicaciones de proxy de aplicación que usan la autenticación previa de Azure AD
- Aplicaciones integradas directamente en la plataforma de aplicaciones de Azure AD que usan la autenticación de OAuth 2.0 u OpenID Connect después de que un usuario o un administrador han dado su consentimiento a esa aplicación.

     > [!NOTE]
     > Esta característica está disponible únicamente para aplicaciones web o API web y aplicaciones empresariales. Las aplicaciones que se registran como [nativas](quickstart-v1-integrate-apps-with-azure-ad.md) no se pueden restringir a un conjunto de usuarios o a los grupos de seguridad del inquilino.

## <a name="update-the-app-to-enable-user-assignment"></a>Actualización de la aplicación para permitir la asignación de usuarios

1. Vaya a [**Azure Portal**](https://portal.azure.com/) e inicie sesión como **administrador global.**
1. En la barra superior, seleccione la cuenta con sesión iniciada. 
1. En **Directory**, seleccione el inquilino de Azure AD donde se registrará la aplicación.
1. En el panel de navegación izquierdo, seleccione **Azure Active Directory**. Si Azure Active Directory no está disponible en el panel de navegación, siga estos pasos:

    1. Seleccione **Todos los servicios** en la parte superior del menú de navegación izquierdo principal.
    1. Escriba **Azure Active Directory** en el cuadro de búsqueda de filtrado y seleccione el elemento **Azure Active Directory** en el resultado.

1. En el panel **Azure Active Directory**, seleccione **Aplicaciones empresariales** en el menú de navegación izquierdo **Azure Active Directory**.
1. Seleccione **Todas las aplicaciones** para ver una lista de todas las aplicaciones.

     Si no ve la aplicación que desea, utilice los distintos filtros de la parte superior de la lista **Registros de aplicaciones** para restringir la lista o desplácese hacia abajo en la lista para buscar la aplicación.

1. Seleccione la aplicación que desea asignar a un usuario o a un grupo de seguridad.
1. En la página **Información general** de la aplicación, seleccione **Propiedades** en el menú de navegación izquierdo de la aplicación.
1. Busque el valor **¿Asignación de usuarios?** y establézcalo en **Sí**. Cuando esta opción se establece en **Sí**, los usuarios se deben asignar primero a esta aplicación antes de poder acceder a ella.
1. Para guardar este cambio de configuración, seleccione **Guardar**.

## <a name="assign-users-and-groups-to-the-app"></a>Asignación de usuarios y grupos a la aplicación

Una vez que ha configurado la aplicación para permitir la asignación de usuarios, puede seguir adelante y asignar usuarios y grupos a la aplicación.

1. Seleccione el panel **Usuarios y grupos** en el menú de navegación izquierdo de la aplicación.
1. En la parte superior de la lista **Usuarios y grupos**, seleccione el botón **Agregar usuario** para abrir el panel **Agregar asignación**.
1. Elija el selector **Usuarios** en el panel **Agregar asignación**. 

     Se mostrará una lista de usuarios y grupos de seguridad junto con un cuadro de texto para buscar un usuario o grupo determinado. Esta pantalla permite seleccionar varios usuarios y grupos en una sola operación.

1. Cuando haya terminado la selección de los usuarios y grupos, presione el botón **Seleccionar** en la parte inferior para desplazarse al apartado siguiente.
1. Presione el botón **Asignar** en la parte inferior para finalizar las asignaciones de usuarios y grupos a la aplicación. 
1. Confirme que los usuarios y grupos que agregó se muestran en la lista **Usuarios y grupos** actualizada.

