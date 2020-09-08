---
title: Adición del acceso condicional a un flujo de usuario en Azure AD B2C
description: Obtenga información acerca de cómo agregar el acceso condicional a los flujos de usuario de Azure AD B2C. Configure las opciones de Multi-factor Authentication (MFA) y las directivas de acceso condicional en los flujos de usuario para aplicar directivas y corregir inicios de sesión de riesgo.
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: overview
ms.date: 09/01/2020
ms.author: mimart
author: msmimart
manager: celested
ms.collection: M365-identity-device-management
ms.openlocfilehash: 60bfac3b80e772e7b359b1e926d5fb84e447a8fb
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89270795"
---
# <a name="add-conditional-access-to-user-flows-in-azure-active-directory-b2c"></a>Adición del acceso condicional a los flujos de usuario en Azure AD B2C

[!INCLUDE [b2c-public-preview-feature](../../includes/active-directory-b2c-public-preview.md)]

Se puede agregar el acceso condicional a los flujos de usuario de Azure Active Directory B2C para administrar inicios de sesión de riesgo en las aplicaciones. La integración de Identity Protection y el acceso condicional en Azure AD B2C le permite configurar directivas que identifiquen comportamientos de inicio de sesión de riesgo y apliquen directivas que requieran una acción adicional por parte del usuario o el administrador. Estos son los componentes que habilitan el acceso condicional en los flujos de usuario de Azure AD B2C:

- **Flujo de usuario**. Cree un flujo de usuario que guíe al usuario en el proceso de inicio de sesión y registro. En la configuración del flujo de usuario, indique si desea activar las directivas de acceso condicional cuando un usuario sigue el flujo de usuario.
- **Aplicación que dirige a los usuarios al flujo de usuario**. Configure la aplicación para dirigir a los usuarios al flujo de usuario de inicio de sesión y registro adecuado; para ello, especifique el punto de conexión del flujo de usuario en la aplicación.
- **Directiva de acceso condicional**. [Cree una directiva de acceso condicional](conditional-access-identity-protection-setup.md) y especifique las aplicaciones a las que desea que se aplique la directiva. Cuando el usuario sigue el flujo de inicio de sesión o el flujo de registro de la aplicación, la directiva de acceso condicional usa las señales de Identity Protection para identificar los inicios de sesión de riesgo y presenta la acción correctiva adecuada si es necesario.

El acceso condicional es compatible con las versiones más recientes de los flujos de usuario. Puede agregar directivas de acceso condicional a los nuevos flujos de usuario a medida que se crean o bien puede agregarlas a flujos de usuarios existentes, siempre que la versión admita el acceso condicional. Al agregar el acceso condicional a un flujo de usuario existente, debe revisar dos configuraciones:

- **Multi-Factor Authentication (MFA)** : ahora los usuarios pueden usar un código de un solo uso mediante SMS o voz o una contraseña de un solo uso por correo electrónico para la autenticación multifactor. La configuración de MFA es independiente de la configuración del acceso condicional. Puede establecer MFA en **Siempre activado** para que MFA siempre sea necesario, independientemente de la configuración del acceso condicional. O bien, puede establecer MFA en **Condicional** para que MFA solo sea necesario cuando una directiva de acceso condicional activa lo requiera.

- **Acceso condicional**: esta configuración siempre se debe establecer en  **Activado**. Normalmente, esta opción solo se debe establecer en **Desactivado** durante la solución de problemas o la migración o para implementaciones heredadas.

Puede encontrar más información sobre [Identity Protection y acceso condicional](conditional-access-identity-protection-overview.md) en Azure AD B2C o puede consultar [cómo configurarlo](conditional-access-identity-protection-setup.md).

## <a name="add-conditional-access-to-a-new-user-flow"></a>Adición del acceso condicional a un nuevo flujo de usuario

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione el icono **Directorio y suscripción** en la barra de herramientas del portal y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.
1. En Azure Portal, busque y seleccione **Azure AD B2C**.
1. En **Directivas**, seleccione **Flujos de usuario** y **Nuevo flujo de usuario**.
1. En la página **Crear un flujo de usuario**, seleccione el tipo de flujo.
1. En **Seleccione una versión**, elija **Recomendada** y, luego, seleccione **Crear**. [Más información](user-flow-versions.md) sobre las versiones del flujo de usuario.

    ![Página Creación de flujo de usuario en Azure Portal con las propiedades resaltadas](./media/tutorial-create-user-flows/select-version.png)

1. Escriba un **nombre** para el flujo de usuario. Por ejemplo, *signupsignin1*.
1. En la sección **Proveedores de identidades**, seleccione los proveedores de identidades que desea permitir para este flujo de usuario.
2. En la sección **Multifactor authentication** (Autenticación multifactor), seleccione el valor de **MFA method** (Método de MFA) deseado y, a continuación, en **MFA enforcement** (Aplicación de MFA), seleccione **Conditional (Recommended)** (Condicional [Recomendado]).
 
   ![Configurar la autenticación multifactor](media/conditional-access-user-flow/configure-mfa.png)

1. En la sección **Conditional Access** (Acceso condicional), seleccione la casilla **Enforce conditional access policies** (Aplicar directivas de acceso condicional).

   ![Configuración del acceso condicional](media/conditional-access-user-flow/configure-conditional-access.png)

1. En la sección **User attributes and token claims** (Atributos de usuario y notificaciones de token), elija los atributos y las notificaciones que desea recopilar y enviar al usuario durante el registro. Por ejemplo, seleccione **Show more** (Mostrar más) y elija los atributos y las notificaciones de **Country/Region** (País/Región) y **Display Name** (Nombre para mostrar). Seleccione **Aceptar**.

    ![Página de selección de atributos y notificaciones con tres notificaciones seleccionadas](./media/conditional-access-user-flow/configure-user-attributes-claims.png)

1. Haga clic en **Crear** para agregar el flujo de usuario. El prefijo *B2C_1* se anexa automáticamente al nombre.

## <a name="add-conditional-access-to-an-existing-user-flow"></a>Adición del acceso condicional a un flujo de usuario existente

> [!NOTE]
> El flujo de usuario existente debe ser una versión que admita el acceso condicional. Estas versiones de flujo de usuario se etiquetan como **Recomendado**.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. Seleccione el icono **Directorio y suscripción** en la barra de herramientas del portal y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.

1. En Azure Portal, busque y seleccione **Azure AD B2C**.

1. En **Directivas**, seleccione**Flujos de usuario**. A continuación, seleccione el flujo de usuario.

1. Seleccione **Properties** (Propiedades) y asegúrese de que el flujo de usuario admite el acceso condicional; para ello, seleccione **Properties** (Propiedades) y busque la configuración con la etiqueta **Conditional Access** (Acceso condicional).
 
   ![Configuración de MFA y el acceso condicional en las propiedades](media/conditional-access-user-flow/add-conditional-access.png)

1. En la sección **Multifactor authentication** (Autenticación multifactor), seleccione el valor de **MFA method** (Método de MFA) deseado y, a continuación, en **MFA enforcement** (Aplicación de MFA), seleccione **Conditional (Recommended)** (Condicional [Recomendado]).
 
1. En la sección **Conditional Access** (Acceso condicional), seleccione la casilla **Enforce conditional access policies** (Aplicar directivas de acceso condicional).

1. Seleccione **Guardar**.

## <a name="test-the-user-flow"></a>Prueba del flujo de usuario

Para probar el acceso condicional en el flujo de usuario, debe [crear una directiva de acceso condicional](conditional-access-identity-protection-setup.md) y habilitar el acceso condicional en el flujo de usuario, como se ha descrito anteriormente. 

### <a name="prerequisites"></a>Requisitos previos

- Se requiere Azure AD B2C Premium 2 para crear directivas de inicio de sesión de riesgo. Los inquilinos Premium P1 pueden crear directivas basadas en grupos, aplicaciones o ubicaciones.
- Con fines de prueba, puede [registrar la aplicación web de prueba](tutorial-register-applications.md) `https://jwt.ms`, que es una aplicación web propiedad de Microsoft que muestra el contenido descodificado de un token (el contenido del token nunca sale del explorador). 
- Para simular un inicio de sesión de riesgo, descargue el explorador TOR e intente iniciar sesión en el punto de conexión del flujo de usuario.
- [Cree una directiva de acceso condicional](conditional-access-identity-protection-setup.md) con la siguiente configuración:
   
   - Para **Usuarios y grupos**, seleccione el usuario de prueba (no seleccione **Todos los usuarios** o podría bloquear su propio inicio de sesión).
   - Para **Aplicaciones en la nube o acciones**, elija **Seleccionar aplicaciones** y, después, elija la aplicación de usuario de confianza.
   - En Condiciones, seleccione **Riesgo de inicio de sesión** y los niveles de riesgo **Alto**, **Medio** y **Bajo**.
   - En **Conceder**, elija **Bloquear acceso**.

      ![Detecciones de riesgo](media/conditional-access-identity-protection-setup/test-conditional-access-policy.png)

### <a name="run-the-user-flow"></a>Ejecución del flujo de usuario

1. Seleccione el flujo de usuario que ha creado para abrir su página de información general y, luego, **Ejecutar flujo de usuario**. En **Aplicación**, seleccione *webapp1*. La **dirección URL de respuesta** debe mostrar `https://jwt.ms`.

   ![Página Ejecutar flujo de usuario del portal con el botón Ejecutar flujo de usuario resaltado](./media/tutorial-create-user-flows/signup-signin-run-now.PNG)

1. Copie la dirección URL de **Ejecutar punto de conexión de flujo de usuario**.

1. Para simular un inicio de sesión de riesgo, abra el [explorador Tor](https://www.torproject.org/download/) y use la dirección URL que copió en el paso anterior para iniciar sesión en la aplicación registrada.

1. Escriba la información solicitada en la página de inicio de sesión e intente iniciar sesión. El token se devuelve al `https://jwt.ms` y debe mostrarse. En el token descodificado de jwt.ms, debería ver que el inicio de sesión se ha bloqueado:

   ![Prueba de un inicio de sesión bloqueado](media/conditional-access-identity-protection-setup/test-blocked-sign-in.png)

## <a name="next-steps"></a>Pasos siguientes

[Personalización de la interfaz de usuario en un flujo de usuario de Azure AD B2C](customize-ui-overview.md)
