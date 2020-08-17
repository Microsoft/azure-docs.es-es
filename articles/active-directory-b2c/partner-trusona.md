---
title: Trusona y Azure Active Directory B2C
titleSuffix: Azure AD B2C
description: Obtenga información sobre cómo agregar Trusona como un proveedor de identidades en Azure AD B2C para habilitar la autenticación sin contraseña.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 07/30/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: a0d5b369e1c143b3df4157329bcf7d3a3f7142d7
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87489476"
---
# <a name="integrating-trusona-with-azure-active-directory-b2c"></a>Integración de Trusona con Azure Active Directory B2C

Trusona es un proveedor de software independiente (ISV) que ayuda a proteger el inicio de sesión, para lo cual habilita la autenticación sin contraseña, la autenticación multifactor y el examen de licencias digitales. En este artículo, aprenderá a agregar Trusona como un proveedor de identidades en Azure AD B2C para habilitar la autenticación sin contraseña.

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesitará lo siguiente:

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* [Un inquilino de Azure AD B2C](tutorial-create-tenant.md) vinculado a la suscripción de Azure.
* Una [cuenta de prueba](https://www.trusona.com/aadb2c) en Trusona.

## <a name="scenario-description"></a>Descripción del escenario

En este escenario, Trusona actúa como proveedor de identidades para Azure AD B2C a fin de habilitar la autenticación sin contraseña. En la solución, se incluyen los componentes siguientes:

* Una directiva de inicio de sesión y registro combinada de Azure AD B2C
* Trusona agregado a Azure AD B2C como un proveedor de identidades
* La aplicación Trusona descargable

![Diagrama de la arquitectura de Trusona](media/partner-trusona/trusona-architecture-diagram.png)

| Paso | Descripción |
|------|------|
|1     | El usuario intenta iniciar sesión o registrarse en la aplicación. El usuario se autentica mediante la directiva de registro e inicio de sesión de Azure AD B2C. Durante el registro, se usa la dirección de correo electrónico comprobada previamente del usuario de la aplicación Trusona.     |
|2     | Azure B2C redirige al usuario al proveedor de identidades OpenID Connect (OIDC) de Trusona mediante el flujo implícito.     |
|3     | En el caso de los inicios de sesión basados en PC de escritorio, Trusona muestra un código QR único, sin estado, animado y dinámico para el examen con la aplicación Trusona. En el caso de los inicios de sesión basados en móviles, Trusona usa un "vínculo profundo" para abrir la aplicación Trusona. Estos dos métodos se usan para la detección de dispositivos y, en última instancia, de usuarios.     |
|4     | El usuario examina el código QR mostrado con la aplicación Trusona.     |
|5     | La cuenta del usuario se encuentra en el servicio en la nube Trusona y la autenticación está preparada.     |
|6     | El servicio en la nube Trusona emite un desafío de autenticación al usuario a través de una notificación de inserción enviada a la aplicación Trusona:<br>a. Al usuario se le muestra un mensaje del desafío de autenticación. <br> b. El usuario decide aceptar o rechazar el desafío. <br> c. Al usuario se le pide que use las opciones de seguridad del sistema operativo (por ejemplo, biometría, código de acceso, PIN o patrón) para confirmar y firmar el desafío con una clave privada en el entorno de ejecución de confianza o enclave seguro. <br> d. La aplicación Trusona genera una carga útil antireproducción dinámica basada en los parámetros de la autenticación en tiempo real. <br> e. La respuesta completa está firmada (por segunda vez) con una clave privada en el entorno de ejecución de confianza o enclave seguro y se devuelve al servicio en la nube Trusona para su comprobación.      |
|7     |  El servicio en la nube Trusona redirige al usuario a la aplicación iniciadora con un id_token. Azure AD B2C comprueba el id_token con la configuración de OpenID publicada de Trusona tal y como se configuró durante la configuración del proveedor de identidades.    |
|  |  |

## <a name="onboard-with-trusona"></a>Incorporación a Trusona

1. Rellene el [formulario](https://www.trusona.com/aadb2c) para crear una cuenta de Trusona y comenzar.

2. Descargue la aplicación móvil de Trusona de la tienda de aplicaciones. Instale la aplicación y registre el correo electrónico.

3. Compruebe su correo electrónico mediante el "vínculo mágico" seguro que envía el software.  

4. Vaya al [panel del desarrollador de Trusona](https://dashboard.trusona.com) para acceder al autoservicio.

5. Seleccione **I'm Ready** (Estoy listo) y autentíquese con la aplicación Trusona.

6. En el panel de navegación izquierdo, seleccione **OIDC Integrations** (Integraciones de OIDC).

7. Seleccione **Create OpenID Connect Integration** (Crear una integración de OpenID Connect).

8. Proporcione un **Nombre** de su elección y use la información de dominio proporcionada anteriormente (por ejemplo, Contoso) en el **campo Client Redirect Host** (Host de redireccionamiento de cliente).  

   > [!NOTE]
   > El nombre de dominio inicial de Azure Active Directory se usa como host de redireccionamiento de cliente.

9. Siga las instrucciones de la [guía de integración de Trusona](https://docs.trusona.com/integrations/aad-b2c-integration/). Cuando se le solicite, use el nombre de dominio inicial (por ejemplo, Contoso) al que se hace referencia en el paso anterior.  

## <a name="integrate-with-azure-ad-b2c"></a>Integración con Azure AD B2C

### <a name="add-a-new-identity-provider"></a>Adición de un nuevo proveedor de identidades

> [!NOTE]
> Si todavía no tiene uno, [cree un inquilino de Azure AD B2C](tutorial-create-tenant.md) vinculado a la suscripción de Azure.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) como administrador global del inquilino de Azure AD B2C.

2. Asegúrese de usar el directorio que contiene el inquilino de Azure AD B2C. Para ello, seleccione el filtro **Directorio y suscripción** en el menú superior y luego el directorio que contiene el inquilino.

3. Elija **Todos los servicios** en la esquina superior izquierda de Azure Portal, busque y seleccione **Azure AD B2C**.

4. Vaya a **Panel** > **Azure Active Directory B2C** > **Proveedores de identidad**.

3. Seleccione **Proveedores de identidades**.

4. Seleccione **Agregar**.

### <a name="configure-an-identity-provider"></a>Configuración de un proveedor de identidades  

1. Seleccione **Tipo de proveedor de identidades** > **OpenID Connect (versión preliminar)** .

2. Rellene el formulario para configurar el proveedor de identidades:  

   | Propiedad | Value  |
   | :--- | :--- |
   | URL de metadatos | `https://gateway.trusona.net/oidc/.well-known/openid-configuration`|
   | Id. de cliente | Se le enviará por correo electrónico desde Trusona |
   | Ámbito | Correo electrónico del perfil de OpenID |
   | Tipo de respuesta | Id_token |
   | Modo de respuesta  | Form_post |

3. Seleccione **Aceptar**.  

4. Seleccione **Asignar las notificaciones de este proveedor de identidades**.  

5. Rellene el formulario para asignar el proveedor de identidades:

   | Propiedad | Value  |
   | :--- | :--- |
   | UserID | Sub  |
   | Nombre para mostrar | nickname |
   | Nombre propio | given_name |
   | Surname | Family_name |
   | Modo de respuesta | email |

6. Seleccione **Aceptar** para completar la instalación del nuevo proveedor de identidades de OIDC.

### <a name="create-a-user-flow-policy"></a>Creación de una directiva de flujo de usuario

Ahora debería ver Trusona como un **nuevo proveedor de identidades de OpenID Connect** enumerado en los proveedores de identidades de B2C.

1. En el inquilino de Azure AD B2C, en **Directivas**, seleccione **Flujos de usuario**.

1. Seleccione **Nuevo flujo de usuario**.

1. Seleccione **Registrarse e iniciar sesión**, seleccione una versión y, a continuación, seleccione **Crear**.

1. Escriba un **nombre** para la directiva.

1. En la sección **Proveedores de identidades** seleccione el **proveedor de identidades de Trusona** recién creado.

   > [!NOTE]
   > Dado que Trusona es intrínsecamente multifactor, es mejor dejar la autenticación multifactor deshabilitada.

1. Seleccione **Crear**.

1. En **Atributos y notificaciones de usuario**, haga clic en **Mostrar más**. En el formulario, seleccione al menos un atributo que haya especificado durante la instalación del proveedor de identidades en la sección anterior.

1. Seleccione **Aceptar**.  

### <a name="test-the-policy"></a>Prueba de la directiva

1. Seleccione la directiva que acaba de crear.

2. Seleccione **Ejecutar flujo de usuario**.

3. En el formulario, escriba la dirección URL de respuesta.

4. Seleccione **Ejecutar flujo de usuario**. Debe redirigirse a la puerta de enlace de OIDC de Trusona. En la puerta de enlace de Trusona, digitalice el código QR seguro que se muestra con la aplicación Trusona o con una aplicación personalizada mediante el SDK de Trusona móvil.

5. Después de digitalizar el código QR seguro, debe redirigirse a la URL de respuesta que definió en el paso 3.

## <a name="next-steps"></a>Pasos siguientes  

Para más información, consulte los artículos siguientes:

- [Directivas personalizadas de Azure AD B2C](custom-policy-overview.md)

- [Introducción a las directivas personalizadas en AAD B2C](custom-policy-get-started.md?tabs=applications)
