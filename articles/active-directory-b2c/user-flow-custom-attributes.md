---
title: Definición de atributos personalizados en Azure Active Directory B2C | Microsoft Docs
description: Defina atributos personalizados para su aplicación en Azure Active Directory B2C a fin de recopilar información sobre sus clientes.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 11/30/2018
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 1d631c47225fac7d8a95541313593333a1399e07
ms.sourcegitcommit: 0820c743038459a218c40ecfb6f60d12cbf538b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87115962"
---
# <a name="define-custom-attributes-in-azure-active-directory-b2c"></a>Definición de atributos personalizados en Azure Active Directory B2C

 Todas las aplicaciones orientadas al cliente tienen requisitos únicos para la información que debe recopilarse. El inquilino de Azure Active Directory B2C (Azure AD B2C) incluye un conjunto integrado de información almacenada en atributos, como el nombre propio, los apellidos, la ciudad y el código postal. Con Azure AD B2C, puede ampliar el conjunto de atributos que se almacenan en cada cuenta de cliente.

 Puede crear atributos personalizados en [Azure Portal](https://portal.azure.com/) y usarlos en sus flujos de usuario de registro, de inicio de sesión o de edición de perfiles. También puede leer y escribir estos atributos mediante [Microsoft Graph API](manage-user-accounts-graph-api.md).

## <a name="create-a-custom-attribute"></a>Creación de un atributo personalizado

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) como administrador global del inquilino de Azure AD B2C.
2. Asegúrese de que está usando el directorio que contiene su inquilino de Azure AD B2C cambiando a él en la esquina superior derecha de Azure Portal. Seleccione la información de la suscripción y, después, seleccione **Cambiar directorio**.

    ![Cambiar al inquilino de Azure AD B2C](./media/user-flow-custom-attributes/switch-directories.png)

    Elija el directorio que contiene el inquilino.

    ![Inquilino de B2C resaltado en el filtro de directorio y suscripción](./media/user-flow-custom-attributes/select-directory.PNG)

3. Elija **Todos los servicios** en la esquina superior izquierda de Azure Portal, busque y seleccione **Azure AD B2C**.
4. Seleccione **Atributos de usuario** y, a continuación, seleccione **Agregar**.
5. Proporcione un **Nombre** para el atributo personalizado (por ejemplo, "ShoeSize").
6. Elija un **Tipo de datos**. Solo están disponibles **String**, **Boolean** e **Int**.
7. Si lo desea, escriba una **Descripción** con fines informativos.
8. Haga clic en **Crear**.

El atributo personalizado ahora está disponible en la lista de **Atributos de usuario** y puede usarlo en los flujos de usuario. Solo se crea la primera vez que se utiliza en cualquier flujo de usuario y no cuando se agrega a la lista de **atributos de usuario**.

## <a name="use-a-custom-attribute-in-your-user-flow"></a>Uso de un atributo personalizado en el flujo de usuario

1. En el inquilino de Azure AD B2C, seleccione **Flujos de usuario**.
1. Haga clic en la directiva (por ejemplo, "B2C_1_SignupSignin") para abrirla.
1. Seleccione **Atributos de usuario** y, después, seleccione el atributo personalizado (por ejemplo, "ShoeSize"). Haga clic en **Save**(Guardar).
1. Seleccione **Notificaciones de aplicación** y, después, seleccione el atributo personalizado.
1. Haga clic en **Save**(Guardar).

Después de crear un usuario mediante un flujo de usuario que usa el atributo personalizado recién creado, el objeto se puede consultar en el [Explorador de Microsoft Graph](https://developer.microsoft.com/graph/graph-explorer). Como alternativa, puede usar la característica [Ejecutar flujo de usuario](https://docs.microsoft.com/azure/active-directory-b2c/tutorial-create-user-flows) del flujo de usuario para comprobar la experiencia del cliente. Ahora debe ver **ShoeSize** en la lista de atributos que se recopilan durante el viaje de suscripción y en el token enviado de vuelta a la aplicación.
