---
title: 'Tutorial: Configuración de Concur para el aprovisionamiento automático de usuarios con Azure Active Directory | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Concur.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: article
ms.date: 01/26/2018
ms.author: jeedes
ms.openlocfilehash: 678b7571d54c8938c163801a584ad813c6e5963f
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88551173"
---
# <a name="tutorial-configure-concur-for-automatic-user-provisioning"></a>Tutorial: Configuración de Concur para el aprovisionamiento automático de usuarios

El objetivo de este tutorial es explicar los pasos que hay que realizar en Concur y Azure AD para aprovisionar y cancelar automáticamente el aprovisionamiento de cuentas de usuario de Azure AD en Concur.

## <a name="prerequisites"></a>Prerrequisitos

En la situación descrita en este tutorial se supone que ya cuenta con los elementos siguientes:

*   Un inquilino de Azure Active Directory.
*   Una suscripción habilitada para el inicio de sesión único en Concur.
*   Una cuenta de usuario de Concur con permisos de administrador de equipo.

## <a name="assigning-users-to-concur"></a>Asignación de usuarios a Concur

Azure Active Directory usa un concepto que se denomina "asignaciones" para determinar qué usuarios deben recibir acceso a determinadas aplicaciones. En el contexto del aprovisionamiento automático de cuentas de usuario, solo se sincronizarán los usuarios y grupos que se han "asignado" a una aplicación de Azure AD.

Antes de configurar y habilitar el servicio de aprovisionamiento, debe decidir qué usuarios o grupos de Azure AD representan a los usuarios que necesitan acceso a la aplicación Concur. Una vez decidido, puede asignar estos usuarios a la aplicación de Concur siguiendo estas instrucciones:

[Asignar un usuario o grupo a una aplicación empresarial](https://docs.microsoft.com/azure/active-directory/active-directory-coreapps-assign-user-azure-portal)

### <a name="important-tips-for-assigning-users-to-concur"></a>Sugerencias importantes para asignar usuarios a Concur

*   Se recomienda asignar un solo usuario de Azure AD a Concur para probar la configuración de aprovisionamiento. Más tarde, se pueden asignar otros usuarios o grupos.

*   Al asignar a un usuario a Concur, debe seleccionar un rol de usuario válido. El rol "Acceso predeterminado" no funciona para realizar el aprovisionamiento.

## <a name="enable-user-provisioning"></a>Habilitación del aprovisionamiento de usuarios

Esta sección lo guía a través de los pasos necesarios para conectar la API de aprovisionamiento de cuentas de usuario de Concur, así como para configurar el servicio de aprovisionamiento con el fin de crear, actualizar y deshabilitar cuentas de usuario asignadas en Concur en función de la asignación de grupos y usuarios de Azure AD.

> [!Tip] 
> También puede habilitar el inicio de sesión único basado en Concur para Concur siguiendo las instrucciones de [Azure Portal](https://portal.azure.com). El inicio de sesión único puede configurarse independientemente del aprovisionamiento automático, aunque estas dos características se complementan entre sí.

### <a name="to-configure-user-account-provisioning"></a>Para configurar el aprovisionamiento automático de cuentas de usuario:

El objetivo de esta sección es describir cómo habilitar el aprovisionamiento de las cuentas de usuario de Active Directory en Concur.

Para habilitar las aplicaciones en el servicio de gastos, debe tener la configuración correcta y el uso de un perfil de administrador de servicio web. No agregue el rol de administrador de servicio web al perfil de administrador existente que se usa para las funciones administrativas de tiempo y gastos.

Los asesores de Concur o el administrador de cliente deben crear un perfil de administrador de servicios web distinto y el administrador de cliente debe usar este perfil para las funciones de administrador de servicios web (por ejemplo, para habilitar aplicaciones). Estos perfiles deben mantenerse separados a diario del perfil de administración de tiempos y gastos del administrador de cliente (el perfil de administrador de tiempos y gastos no debe tener el rol de administrador de servicios web asignado).

Al crear el perfil que se usará para habilitar la aplicación, escriba el nombre del administrador de cliente en los campos de perfil de usuario. Así se asigna la propiedad al perfil. Una vez creado el perfil, el cliente debe iniciar sesión con este perfil para hacer clic en el botón *Habilitar* para una aplicación asociada en el menú de servicios web.

Por las siguientes razones, esta acción no debe realizarse con el perfil que se usa en la administración normal de tiempo y gastos.

* El cliente tiene que ser el que hace clic en "*Sí*" en la ventana de cuadro de diálogo que aparece después de la habilitación de la aplicación. Este clic confirma que el cliente está autorizado por la aplicación asociada a tener acceso a sus datos, por lo que usted o el asociado no pueden hacer clic en el botón Sí.

* Si un administrador de cliente que ha habilitado una aplicación mediante el perfil de administrador de tiempo y gastos abandona la empresa (lo que da lugar a la desactivación del perfil), las aplicaciones habilitadas mediante ese perfil no funcionarán hasta que la aplicación se habilite con otro perfil de administrador de servicios web activo. Es por este motivo por lo hay que crear distintos perfiles de administrador servicios web.

* Si un administrador deja la compañía, el nombre asociado al perfil de administrador de servicios web puede cambiarse si se quiere por el del administrador de sustitución sin afectar a la aplicación habilitada, porque no necesita ese perfil desactivado.

**Siga estos pasos para configurar el aprovisionamiento de usuario:**

1. Inicie sesión en el inquilino de **Concur**.

2. En el menú **Administración**, seleccione **Servicios web**.
   
    ![Inquilino de Concur](./media/concur-provisioning-tutorial/IC721729.png "Inquilino de Concur")

3. En el lado izquierdo, en el panel **Servicios web**, seleccione **Habilitar aplicación de asociado**.
   
    ![Habilitar aplicación de asociado](./media/concur-provisioning-tutorial/ic721730.png "Habilitar aplicación de asociado")

4. En la lista **Habilitar aplicación**, seleccione **Azure Active Directory** y después haga clic en **Habilitar**.
   
    ![Microsoft Azure Active Directory](./media/concur-provisioning-tutorial/ic721731.png "Microsoft Azure Active Directory")

5. Haga clic en **Sí** para cerrar el cuadro de diálogo **Confirmar acción**.
   
    ![Confirmar acción](./media/concur-provisioning-tutorial/ic721732.png "Confirmar acción")

6. En [Azure Portal](https://portal.azure.com), vaya a la sección **Azure Active Directory > Aplicaciones empresariales > Todas las aplicaciones**.

7. Si ya ha configurado Concur para el inicio de sesión único, busque la instancia de Concur mediante el campo de búsqueda. En caso contrario, seleccione **Agregar** y busque **Concur** en la galería de aplicaciones. Seleccione Concur en los resultados de búsqueda y agréguelo a la lista de aplicaciones.

8. Seleccione la instancia de Concur y luego, la pestaña **Aprovisionamiento**.

9. Establezca el **modo de aprovisionamiento** en **Automático**. 
 
    ![Aprovisionamiento](./media/concur-provisioning-tutorial/provisioning.png)

10. En la sección **Credenciales de administrador**, escriba el **nombre de usuario** y la **contraseña** del administrador de Concur.

11. En Azure Portal, haga clic en **Probar conexión** para asegurarse de que Azure AD puede conectarse a la aplicación de Concur. Si la conexión no se establece, asegúrese de que la cuenta de Concur tiene permisos de administrador de equipo.

12. Escriba la dirección de correo electrónico de una persona o grupo que debe recibir las notificaciones de error de aprovisionamiento en el campo **Correo electrónico de notificación** y active la casilla.

13. Haga clic en **Guardar**.

14. En la sección Asignaciones, seleccione **Synchronize Azure Active Directory Users to Concur** (Sincronizar usuarios de Azure Active Directory con Concur).

15. En la sección **Attribute Mappings** (Asignaciones de atributos), revise los atributos de usuario que se sincronizan entre Azure AD y Concur. Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con las cuentas de usuario de Concur con el objetivo de realizar operaciones de actualización. Seleccione el botón Guardar para confirmar los cambios.

16. Para habilitar el servicio de aprovisionamiento de Azure AD para Concur, cambie el **Estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

17. Haga clic en **Guardar**.

Ahora puede crear una cuenta de prueba. Espere 20 minutos para comprobar que la cuenta se ha sincronizado con Concur.

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](tutorial-list.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)
* [Configuración del inicio de sesión único](concur-tutorial.md)

