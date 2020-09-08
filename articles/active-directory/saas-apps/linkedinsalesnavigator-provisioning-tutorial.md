---
title: 'Tutorial: aprovisionamiento de usuarios: LinkedIn Sales Navigator, Azure AD'
description: Aprenda a configurar Azure Active Directory para aprovisionar y cancelar el aprovisionamiento de cuentas de usuario de LinkedIn Sales Navigator automáticamente.
services: active-directory
author: ArvindHarinder1
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: article
ms.date: 03/28/2019
ms.author: arvinh
ms.openlocfilehash: ef87a1c7163825fcee8f4a27408719839d8a6d43
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88549711"
---
# <a name="tutorial-configure-linkedin-sales-navigator-for-automatic-user-provisioning"></a>Tutorial: Configuración de LinkedIn Sales Navigator para el aprovisionamiento automático de usuarios

El objetivo de este tutorial es explicar los pasos que debe realizar en LinkedIn Sales Navigator y Azure AD para aprovisionar y cancelar el aprovisionamiento de cuentas de usuario de Azure AD automáticamente para LinkedIn Sales Navigator.

## <a name="prerequisites"></a>Prerrequisitos

En la situación descrita en este tutorial se supone que ya cuenta con los elementos siguientes:

* Un inquilino de Azure Active Directory
* Un inquilino de LinkedIn Sales Navigator 
* Una cuenta de administrador de LinkedIn Sales Navigator con acceso al Centro de cuentas de LinkedIn

> [!NOTE]
> Azure Active Directory se integra con LinkedIn Sales Navigator mediante el protocolo [SCIM](http://www.simplecloud.info/).

## <a name="assigning-users-to-linkedin-sales-navigator"></a>Asignación de usuarios a LinkedIn Sales Navigator

Azure Active Directory usa un concepto que se denomina "asignaciones" para determinar qué usuarios deben recibir acceso a determinadas aplicaciones. En el contexto de aprovisionamiento de cuentas de usuario automático, solo se sincronizarán los usuarios y grupos que se han "asignado" a una aplicación de Azure AD.

Antes de configurar y habilitar el servicio de aprovisionamiento, debe decidir qué usuarios o grupos de Azure AD representan a los usuarios que necesitan acceder a LinkedIn Sales Navigator. Una vez decidido, puede asignar estos usuarios a LinkedIn Sales Navigator siguiendo estas instrucciones:

[Asignar un usuario o grupo a una aplicación empresarial](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-linkedin-sales-navigator"></a>Sugerencias importantes para asignar usuarios a LinkedIn Sales Navigator

* Se recomienda asignar un solo usuario de Azure AD a LinkedIn Sales Navigator para probar la configuración del aprovisionamiento. Más tarde, se pueden asignar otros usuarios o grupos.

* Al asignar un usuario a LinkedIn Sales Navigator, debe seleccionar el rol **Usuario** en el cuadro de diálogo de asignación. El rol "Acceso predeterminado" no funciona para realizar el aprovisionamiento.

## <a name="configuring-user-provisioning-to-linkedin-sales-navigator"></a>Configuración del aprovisionamiento de usuarios en LinkedIn Sales Navigator

Esta sección le guía a través de los pasos necesarios para conectar la API de aprovisionamiento de su cuenta de usuario de SCIM de LinkedIn Sales Navigator, así como para configurar el servicio de aprovisionamiento con el fin de crear, actualizar y deshabilitar cuentas de usuario asignadas de LinkedIn Sales Navigator en función de la asignación de grupos y usuarios en Azure AD.

> [!TIP]
> También puede decidir habilitar el inicio de sesión único basado en SAML para LinkedIn Sales Navigator siguiendo las instrucciones de [Azure Portal](https://portal.azure.com). El inicio de sesión único puede configurarse independientemente del aprovisionamiento automático, aunque estas dos características se complementan entre sí.

### <a name="to-configure-automatic-user-account-provisioning-to-linkedin-sales-navigator-in-azure-ad"></a>Para configurar el aprovisionamiento de cuentas de usuario automático para LinkedIn Sales Navigator en Azure AD, siga estos pasos:

El primer paso consiste en recuperar el token de acceso de LinkedIn. Si es administrador de organización, puede aprovisionar automáticamente un token de acceso. En el centro de cuentas, vaya a **Configuración &gt; Configuración global** y abra el panel **SCIM Setup** (Configuración de SCIM).

> [!NOTE]
> Si obtiene acceso al centro de cuentas directamente en lugar de a través de un vínculo, puede llegar a este panel mediante los pasos siguientes.

1. Inicie sesión en el centro de cuentas.

2. Seleccione **Administración &gt; Configuración de administración**.

3. Haga clic en **Advanced Integrations** (Integraciones avanzadas) en la barra lateral izquierda. Se le dirigirá al centro de cuentas.

4. Haga clic en **+ Add new SCIM configuration** (+ Agregar nueva configuración de SCIM) y rellene cada campo del procedimiento.

    > [!NOTE]
    > Si no está habilitada la asignación de licencias automática, solo se sincronizan los datos de usuario.

    ![Aprovisionamiento de LinkedIn Sales Navigator](./media/linkedinsalesnavigator-provisioning-tutorial/linkedin_1.PNG)

    > [!NOTE]
    > Si está habilitada la asignación de licencias automática, debe tomar nota de la instancia de la aplicación y del tipo de licencia. Las licencias se asignan por orden de llegada hasta que se obtienen todas las licencias.

    ![Aprovisionamiento de LinkedIn Sales Navigator](./media/linkedinsalesnavigator-provisioning-tutorial/linkedin_2.PNG)

5. Haga clic en **Generar token**. El token de acceso debería aparecer en el campo **Token de acceso**.

6. Guarde el token de acceso en el Portapapeles o en el equipo antes de abandonar la página.

7. Después, inicie sesión en [Azure Portal](https://portal.azure.com) y vaya a la sección **Azure Active Directory > Aplicaciones empresariales > Todas las aplicaciones**.

8. Si ya ha configurado LinkedIn Sales Navigator para el inicio de sesión único, busque la instancia de LinkedIn Sales Navigator mediante el campo de búsqueda. En caso contrario, seleccione **Agregar** y busque **LinkedIn Sales Navigator** en la galería de aplicaciones. Seleccione LinkedIn Sales Navigator en los resultados de búsqueda y agréguelo a la lista de aplicaciones.

9. Seleccione la instancia de LinkedIn Sales Navigator y la pestaña **Aprovisionamiento**.

10. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Aprovisionamiento de LinkedIn Sales Navigator](./media/linkedinsalesnavigator-provisioning-tutorial/linkedin_3.PNG)

11. Rellene los campos siguientes en **Credenciales de administrador**:

    * En el campo **URL de inquilino**, escriba https://developer.linkedin.com.

    * En el campo **Token secreto**, escriba el token de acceso que ha generado en el paso 1 y haga clic en **Probar conexión**.

    * Debería ver una notificación que le indica que el proceso se ha realizado correctamente en el lado superior derecho del portal.

12. Escriba la dirección de correo electrónico de una persona o grupo que debe recibir las notificaciones de error aprovisionamiento en el campo **Correo electrónico de notificación** y active la casilla que aparece a continuación.

13. Haga clic en **Save**(Guardar).

14. En la sección **Asignaciones de atributos**, revise los atributos de usuario y de grupo que se sincronizarán de Azure AD a LinkedIn Sales Navigator. Tenga en cuenta que los atributos seleccionados como propiedades **Matching** se usarán para establecer coincidencias con las cuentas de usuario y los grupos de LinkedIn Sales Navigator para las operaciones de actualización. Seleccione el botón Guardar para confirmar los cambios.

    ![Aprovisionamiento de LinkedIn Sales Navigator](./media/linkedinsalesnavigator-provisioning-tutorial/linkedin_4.PNG)

15. Para habilitar el servicio de aprovisionamiento de Azure AD para LinkedIn Sales Navigator, cambie el **Estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

16. Haga clic en **Save**(Guardar).

Esta acción iniciará la sincronización inicial de todos los usuarios y grupos asignados a LinkedIn Sales Navigator en la sección Usuarios y grupos. Tenga en cuenta que la sincronización inicial tardará más tiempo en realizarse que las posteriores, que se producen aproximadamente cada 40 minutos si se está ejecutando el servicio. Puede usar la sección **Detalles de sincronización** para supervisar el progreso y hacer clic en los vínculos a los registros de actividad de aprovisionamiento, que describen todas las acciones que ha llevado a cabo el servicio de aprovisionamiento en la aplicación LinkedIn Sales Navigator.

Para más información sobre cómo leer los registros de aprovisionamiento de Azure AD, consulte el tutorial de [Creación de informes sobre el aprovisionamiento automático de cuentas de usuario](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)
