---
title: itsme OpenID Connect con Azure Active Directory B2C
titleSuffix: Azure AD B2C
description: Obtenga información sobre cómo integrar la autenticación de Azure AD  B2C con itsme OIDC mediante la directiva de flujo de usuario client_secret. itsme es una aplicación de identificación digital. Permite iniciar sesión de forma segura sin lectores de tarjetas, contraseñas, autenticación en dos fases y códigos PIN múltiples.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 07/30/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: ba7875caa6a1db7638bfeafcfea1efa7b2462152
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87489522"
---
# <a name="configure-itsme-openid-connect-oidc-with-azure-active-directory-b2c"></a>Configuración de itsme OpenID Connect (OIDC) con Azure Active Directory B2C

La aplicación de identificación digital itsme le permite iniciar sesión de forma segura sin lectores de tarjetas, contraseñas, autenticación en dos fases ni códigos PIN múltiples. La aplicación itsme proporciona una autenticación sólida de cliente con una identidad comprobada. En este artículo, aprenderá a integrar la autenticación de Azure AD B2C con itsme OpenID Connect (OIDC) mediante una directiva de flujo de usuario de secreto de cliente.

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesitará lo siguiente:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* [Un inquilino de Azure AD B2C](tutorial-create-tenant.md) vinculado a la suscripción de Azure.
* El identificador de cliente, también conocido como código de asociado, proporcionado por itsme.
* El código de servicio proporcionado por itsme.
* El secreto de cliente de la cuenta de itsme.

## <a name="scenario-description"></a>Descripción del escenario

![Diagrama de la arquitectura de itsme](media/partner-itsme/itsme-architecture-diagram.png)

<!--
Please clarify step 1 in the description below - we don't have steps in this tutorial for "adapting in the Azure AD B2C Custom Policy- User Journeys" - should this be added somewhere?
-->

| Paso | Descripción |
|------|------|
|1     | En el sitio web o la aplicación, adapte el flujo de usuario de Azure AD B2C para incluir el botón **Iniciar sesión con itsme**. El flujo de interacción se inicia cuando el usuario hace clic en este botón.  |
|2     | Azure AD B2C inicia el flujo de OpenID Connect mediante el envío de una solicitud de autorización a la API de secreto de cliente de itsme. Un punto de conexión de configuración de OpenID o conocido está disponible y contiene información sobre los puntos de conexión.  |
|3     | El entorno itsme redirige al usuario a la página de autoidentificación de itsme, que permite al usuario rellenar su número de teléfono.  |
|4     | El entorno itsme recibe el número de teléfono del usuario y valida la corrección.  |
|5     | Si el número de teléfono pertenece a un usuario de itsme activo, se crea una acción para la aplicación itsme.  |
|6     | El usuario abre la aplicación itsme, comprueba la solicitud y confirma la acción.  |
|7     |  La aplicación indica al entorno itsme que se ha confirmado la acción. |
|8     |  El entorno itsme devuelve el código de autorización de OAuth a Azure AD B2C. |
|9     |  Con el código de autorización, Azure AD B2C realiza una solicitud de token. |
| 10 | El entorno itsme comprueba la solicitud de token y, si sigue siendo válida, devuelve el token de acceso de OAuth y el token de identificador que contiene la información de usuario solicitada. |
| 11 | Por último, se redirige al usuario a la URL de redireccionamiento como un usuario autenticado.  |
|   |   |

## <a name="onboard-with-itsme"></a>Incorporación con itsme

1. Para crear una cuenta con itsme, visite itsme en [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace).

2. Active su cuenta de itsme mediante el envío de un correo electrónico a onboarding@itsme.be. Recibirá un **código de asociado** y un **código de servicio** que se necesitarán para la configuración de B2C.

3. Después de la activación de la cuenta de asociado de itsme, recibirá un correo electrónico con un vínculo de un solo uso al **secreto de cliente**.

4. Siga las instrucciones que encontrará en [itsme](https://business.itsme.be/en) para completar la configuración.

## <a name="integrate-with-azure-ad-b2c"></a>Integración con Azure AD B2C

### <a name="set-up-a-new-identity-provider-in-azure-ad-b2c"></a>Configuración de un nuevo proveedor de identidades en Azure AD B2C

> [!NOTE]
> Si todavía no tiene uno, [cree un inquilino de Azure AD B2C](tutorial-create-tenant.md) vinculado a la suscripción de Azure.

1. Asegúrese de que usa el directorio que contiene el inquilino de Azure AD B2C. Seleccione el filtro **Directorio y suscripción** del menú superior y el directorio que contiene el inquilino de Azure AD B2C.

2. En **Servicios de Azure**, seleccione **Azure AD B2C** (o seleccione **Más servicios** y utilice el cuadro de búsqueda **Todos los servicios** para buscar *Azure AD B2C*).

3. Seleccione **Proveedores de identidades** y luego **Nuevo proveedor de OpenID Connect**.

4. Rellene el formulario con la siguiente información:

   |Propiedad | Value |
   |------------ |------- |
   | Nombre | itsme |
   | URL de metadatos | `https://oidc.<environment>.itsme.services/clientsecret-oidc/csapi/v0.1/.well-known/openid-configuration` <br>donde `<environment>` es `e2e` (entorno de prueba) o `prd` (producción)  |
   | ClientID     | Su **Id. de cliente**, también conocido como **Código de asociado**  |
   | Secreto del cliente | Su valor de **client_secret** |
   | Ámbito  | Servicio OpenID: correo electrónico del perfil de YOURSERVICECODE [teléfono] [dirección]  |
   |Tipo de respuesta | código |
   |Modo de respuesta | Query |
   |Sugerencia de dominio | *Se puede dejar vacío* |
   |UserID | sub |
   |Display Name (Nombre para mostrar) | name |
   |Nombre propio | given_name |
   |Surname | family_name |
   |Email | email|

5. Seleccione **Guardar**.

### <a name="configure-a-user-flow"></a>Configuración de un flujo de usuario

1. En el inquilino de Azure AD B2C, en **Directivas**, seleccione **Flujos de usuario**.

2. Seleccione **Nuevo flujo de usuario**.

3. Seleccione **Registrarse e iniciar sesión**, seleccione una versión y, a continuación, seleccione **Crear**.

4. Escriba un **nombre**.

5. En la sección **Proveedores de identidades**, seleccione **itsme**.

6. Seleccione **Crear**.

7. Seleccione el nombre del flujo de usuario para abrir el flujo de usuario recién creado.

8. Seleccione **Propiedades** y ajuste los valores siguientes:

   * Cambie el valor de **Acceso y duración del token de id. (minutos)** a **5**.
   * Cambie el valor de **Vigencia de la ventana deslizante del token de actualización** a **Sin expiración**.

### <a name="register-an-application"></a>Registro de una aplicación

1. En el inquilino de B2C, en **Administrar**, seleccione **Registros de aplicaciones** > **Nuevo registro**.

2. Proporcione un **Nombre** para la aplicación y escriba su **URI de redirección**. Con fines de prueba, escriba `https://jwt.ms`.

3. Asegúrese de que el estado de la autenticación multifactor es **Deshabilitado**.

4. Seleccione **Registrar**.

   a. Con fines de prueba, seleccione **Autenticación** y, en **Concesión implícita**, active las casillas **Tokens de acceso** y **Tokens de id.**  

   b. Seleccione **Guardar**.

## <a name="test-the-user-flow"></a>Prueba del flujo de usuario

1. En su inquilino de B2C, en **Directivas**, seleccione**Flujos de usuario**.

2. Seleccione el flujo de usuario que creó anteriormente.

3. Seleccione **Ejecutar flujo de usuario**.

   a. **Aplicación**: *seleccione la aplicación registrada*.

   b. **URL de respuesta**: *seleccione la dirección URL de redireccionamiento*.

4. Aparece la página **Identifíquese** de itsme.  

5. Indique su número de teléfono móvil y seleccione **Enviar**.

6. Confirme la acción en la aplicación itsme.

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte los artículos siguientes:

* [Directivas personalizadas de Azure AD B2C](custom-policy-overview.md)

* [Introducción a las directivas personalizadas en Azure AD B2C](custom-policy-get-started.md?tabs=applications)