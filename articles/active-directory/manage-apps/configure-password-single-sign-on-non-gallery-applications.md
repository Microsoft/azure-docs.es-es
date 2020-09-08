---
title: Procedimiento para configurar el inicio de sesión único basado en contraseña para aplicaciones de Azure AD
description: Procedimiento para configurar el inicio de sesión único (SSO) basado en contraseña para sus aplicaciones de Azure AD que están en la plataforma de identidad de Microsoft (Azure AD)
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: how-to
ms.date: 07/29/2020
ms.author: kenwith
ms.openlocfilehash: be6a8a58f1d66df9d0fe557584c4731e42ae9c59
ms.sourcegitcommit: 628be49d29421a638c8a479452d78ba1c9f7c8e4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88640555"
---
# <a name="configure-password-based-single-sign-on"></a>Configuración del inicio de sesión único basado en contraseña

En la [serie de guías de inicio rápido](view-applications-portal.md) sobre la administración de aplicaciones, aprendió a usar Azure AD como proveedor de identidades (IdP) para una aplicación. En la guía de inicio rápido, configura el inicio de sesión único basado en SAML. Otra opción es el SSO basado en contraseña. Este artículo explica con más detalle la opción SSO basado en contraseña. 

Esta opción está disponible para cualquier sitio web con una página de inicio de sesión HTML. El SSO basado en contraseña también se conoce como almacenamiento de contraseñas. El SSO basado en contraseña permite administrar el acceso y las contraseñas de los usuarios en aplicaciones web que no admiten la federación de identidades. También es útil cuando varios usuarios necesitan compartir una sola cuenta, como las cuentas de aplicaciones de redes sociales de la organización.

El SSO basado en contraseña representa una excelente manera de empezar a integrar aplicaciones en Azure AD rápidamente, y le permite lo siguiente:

- Habilitar el inicio de sesión único para los usuarios almacenando y reproduciendo de forma segura los nombres de usuario y contraseñas

- Admitir aplicaciones que requieren varios campos de inicio de sesión para aplicaciones que no solo requieren los campos de nombre de usuario y contraseña para iniciar sesión

- Personalizar las etiquetas de los campos de nombre de usuario y contraseña que los usuarios ven en la página [Aplicaciones](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction) cuando escriben sus credenciales

- Permitir a los usuarios proporcionar sus propios nombres de usuario y contraseñas para las cuentas de aplicación existentes que están escribiendo manualmente

- Permitir que un miembro del grupo de negocios especifique los nombres de usuario y contraseñas que se asignan a un usuario mediante el uso de la característica [Acceso de autoservicio a las aplicaciones](https://docs.microsoft.com/azure/active-directory/active-directory-self-service-application-access)

-   Permitir que un administrador especifique un nombre de usuario y una contraseña que usarán los usuarios o grupos al iniciar sesión en la aplicación mediante la característica Actualizar credenciales 

## <a name="before-you-begin"></a>Antes de empezar

El uso de Azure AD como proveedor de identidades (IdP) y la configuración del inicio de sesión único (SSO) pueden ser algo sencillo o complejo en función de la aplicación utilizada. Algunas aplicaciones se pueden configurar con pocos pasos. Otras exigen una configuración detallada. Para empezar rápidamente, consulte la [serie de guías de inicio rápido](view-applications-portal.md) sobre la administración de aplicaciones. Si la aplicación que va a añadir es sencilla, probablemente no necesite leer este artículo. Si la aplicación que va a añadir requiere una configuración personalizada y tiene que usar SSO basado en contraseña, este artículo es para usted.

> [!IMPORTANT] 
> Hay escenarios en los que la opción **Inicio de sesión único** no estará al desplazarse por una aplicación en las **aplicaciones empresariales**. 
>
> Por ejemplo, si la aplicación se registró mediante **Registros de aplicaciones**, la característica de inicio de sesión único estará configurada para usar OAuth de OIDC de forma predeterminada. En este caso, la opción de **Inicio de sesión único** no se mostrará en la navegación en **Aplicaciones empresariales**. Cuando use **Registros de aplicaciones** para agregar su aplicación personalizada, configure las opciones en el archivo de manifiesto. Para obtener más información sobre el archivo de manifiesto, consulte [Manifiesto de aplicación de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/develop/reference-app-manifest). Para obtener más información sobre los estándares de SSO, consulte [Autenticación y autorización mediante la Plataforma de identidad de Microsoft](https://docs.microsoft.com/azure/active-directory/develop/authentication-vs-authorization#authentication-and-authorization-using-microsoft-identity-platform). 
>
> Otros escenarios en los que falta **inicio de sesión único** en la navegación incluyen cuándo se hospeda una aplicación en otro inquilino o si su cuenta no tiene los permisos necesarios (Administrador global, Administrador de aplicaciones en la nube, Administrador de la aplicación o propietario de la entidad de servicio). Los permisos también pueden provocar un escenario en el que puede abrir **Inicio de sesión único** pero no podrá guardar. Para más información acerca de los roles administrativos de Azure AD, consulte (https://docs.microsoft.com/azure/active-directory/users-groups-roles/directory-assign-admin-roles).


## <a name="basic-configuration"></a>Configuración básica

En la [serie de guías de inicio rápido](view-applications-portal.md), ha aprendido a agregar una aplicación al inquilino, lo que permite saber a Azure AD que se usa como proveedor de identidades (IdP) de la aplicación. Algunas aplicaciones ya están preconfiguradas y aparecen en la galería de Azure AD. Otras aplicaciones no están en la galería y tiene que crear una aplicación genérica y configurarla manualmente. Dependiendo de la aplicación, es posible que la opción SSO basado en contraseña no esté disponible. Si no ve la lista de opciones basadas en contraseña en la página de inicio de sesión único de la aplicación, no estará disponible.

La página de configuración para SSO basado en contraseña es sencilla. Incluye solo la dirección URL de la página de inicio de sesión que usa la aplicación. Esta cadena debe ser la página que incluye el campo de entrada del nombre de usuario.

Después de escribir la dirección URL, seleccione **Guardar**. Azure AD analiza el código HTML de la página de inicio de sesión para los campos de entrada de nombre de usuario y contraseña. Si el intento se realiza correctamente, habrá terminado.
 
El siguiente paso será la [Asignación de usuarios y grupos a la aplicación](methods-for-assigning-users-and-groups.md). Después de asignar usuarios y grupos, puede proporcionar las credenciales que se usarán para un usuario cuando inicie sesión en la aplicación. Seleccione **Usuarios y grupos**, active la casilla correspondiente a la fila del usuario o del grupo y, después, seleccione **Actualizar credenciales**. Por último, escriba el nombre de usuario y la contraseña que se usarán para el usuario o grupo. Si no lo hace, se solicitará a los usuarios que especifiquen ellos mismos las credenciales al inicio.
 

## <a name="manual-configuration"></a>Configuración manual

Si se produce un error en el intento de análisis de Azure AD, puede configurar el inicio de sesión manualmente.

1. En **\<application name> Configuración**, seleccione  **Configurar\<application name> Establecer configuración de inicio de sesión único con contraseña** para mostrar la página **Configurar inicio de sesión**. 

2. Seleccione **Detectar campos de inicio de sesión manualmente**. Aparecen instrucciones adicionales que describen la detección manual de los campos de inicio de sesión.

   ![Configuración manual del inicio de sesión único con contraseña](./media/configure-password-single-sign-on/password-configure-sign-on.png)
3. Seleccione **Capturar campos de inicio de sesión**. Se abre una página de estado de la captura en una nueva pestaña, que muestra el mensaje **Actualmente la captura de metadatos está en curso**.

4. Si aparece la casilla **My Apps Extension Required** (Se requiere la extensión Aplicaciones) en una nueva pestaña, seleccione **Instalar ahora** para instalar la extensión del explorador **Extensión de inicio de sesión seguro de mis aplicaciones**. (La extensión del explorador requiere Microsoft Edge, Chrome o Firefox). A continuación, instale, inicie y active la extensión, y actualice la página de estado de la captura.

   La extensión del explorador abre otra pestaña que muestra la dirección URL especificada.
5. En la pestaña con la dirección URL especificada, recorra el proceso de inicio de sesión. Rellene los campos de nombre de usuario y contraseña e intente iniciar sesión. (No es necesario proporcionar la contraseña correcta).

   Se le pedirá que guarde los campos de inicio de sesión capturados.
6. Seleccione **Aceptar**. La extensión del explorador actualiza la página de estado de la captura con el mensaje **Los metadatos se han actualizado para la aplicación**. La pestaña del explorador se cierra.

7. En la página **Configurar inicio de sesión** de Azure AD, seleccione **Bien. Pude iniciar sesión en la aplicación correctamente**.

8. Seleccione **Aceptar**.

## <a name="next-steps"></a>Pasos siguientes

- [Asignación de usuarios y grupos a la aplicación](methods-for-assigning-users-and-groups.md)
- [Configuración del aprovisionamiento automático de cuentas de usuario](../app-provisioning/configure-automatic-user-provisioning-portal.md)
