---
title: 'Tutorial: Configuración de StarLeaf para el aprovisionamiento automático de usuarios con Azure Active Directory | Microsoft Docs'
description: Obtenga información sobre cómo configurar Azure Active Directory para aprovisionar y cancelar automáticamente el aprovisionamiento de cuentas de usuario de StarLeaf.
services: active-directory
documentationcenter: ''
author: zchia
writer: zchia
manager: beatrizd
ms.assetid: na
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/19/2019
ms.author: zhchia
ms.openlocfilehash: 74e37bda28bed3ecd2c842c4659bcff3cb86eb6a
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87920694"
---
# <a name="tutorial-configure-starleaf-for-automatic-user-provisioning"></a>Tutorial: Configuración de StarLeaf para el aprovisionamiento automático de usuarios

El objetivo de este tutorial es mostrar los pasos que se realizan en StarLeaf y Azure Active Directory (Azure AD) para configurar Azure AD a fin de aprovisionar y cancelar automáticamente el aprovisionamiento de usuarios o grupos en StarLeaf.

> [!NOTE]
>  Este tutorial describe un conector que se crea sobre el servicio de aprovisionamiento de usuarios de Azure AD. Para obtener información importante acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Este conector está actualmente en versión preliminar. Para más información sobre los términos de uso generales de Microsoft Azure para las características en versión preliminar, consulte [Términos de uso complementarios para las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prerrequisitos

En el escenario descrito en este tutorial se supone que ya cuenta con los requisitos previos siguientes:

* Un inquilino de Azure AD.
* [Un inquilino de StarLeaf](https://starleaf.com/).
* Una cuenta de usuario de StarLeaf con permisos de administrador.

## <a name="assign-users-to-starleaf"></a>Asignación de usuarios a StarLeaf
Azure Active Directory usa el concepto de asignación para determinar qué usuarios deben recibir acceso a determinadas aplicaciones. En el contexto del aprovisionamiento automático de usuarios, solo se sincronizan los usuarios y grupos que se han asignado a una aplicación en Azure AD.

Antes de configurar y habilitar el aprovisionamiento automático de usuarios, debe decidir qué usuarios o grupos de Azure AD necesitan acceder a StarLeaf. Después, puede asignar los usuarios y grupos a StarLeaf mediante [estas instrucciones](../manage-apps/assign-user-or-group-access-portal.md).

## <a name="important-tips-for-assigning-users-to-starleaf"></a>Sugerencias importantes para asignar usuarios a StarLeaf

* Se recomienda asignar un único usuario de Azure AD a StarLeaf para probar la configuración de aprovisionamiento automático de usuarios. Más tarde, se pueden asignar otros usuarios y grupos.

* Cuando asigne un usuario a StarLeaf, debe seleccionar un rol específico de la aplicación válido (si está disponible) en el cuadro de diálogo de asignación. Los usuarios con el rol de acceso predeterminado quedan excluidos del aprovisionamiento.

## <a name="set-up-starleaf-for-provisioning"></a>Configuración de StarLeaf para el aprovisionamiento

Antes de configurar StarLeaf para aprovisionar automáticamente usuarios con Azure AD, deberá configurar el aprovisionamiento SCIM en StarLeaf.

1. Inicie sesión en la [consola de administración de StarLeaf](https://portal.starleaf.com/#page=login). Vaya a **Integraciones** > **Agregar integración**.

    ![StarLeaf: Agregar SCIM](media/starleaf-provisioning-tutorial/image00.png)

2. En **Tipo**, seleccione Microsoft Azure Active Directory. Escriba un nombre adecuado en **Nombre**. Haga clic en **Aplicar**.

    ![StarLeaf: Agregar SCIM](media/starleaf-provisioning-tutorial/image01.png)

3.  Se mostrarán los valores de **URL base de SCIM** y **Token de acceso**. Estos valores se escriben en los campos **URL de inquilino** y **Token secreto** de la pestaña Aprovisionamiento de la aplicación StarLeaf en Azure Portal. 

    ![StarLeaf: Crear token](media/starleaf-provisioning-tutorial/image02.png)

## <a name="add-starleaf-from-the-gallery"></a>Adición de StarLeaf desde la galería

Para configurar StarLeaf para el aprovisionamiento automático de usuarios con Azure AD, es preciso agregar la aplicación desde la galería de Azure AD a la lista de aplicaciones SaaS administradas.

**Para agregar StarLeaf desde la galería de aplicaciones de Azure AD, siga estos pasos:**

1. En **[Azure Portal](https://portal.azure.com)** , en el panel de navegación izquierdo, seleccione **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, seleccione el botón **Nueva aplicación** en la parte superior del panel.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **StarLeaf** y seleccione **StarLeaf** en el panel de resultados.
    ![StarLeaf en la lista de resultados](common/search-new-app.png)

## <a name="configure-automatic-user-provisioning-to-starleaf"></a>Configuración del aprovisionamiento automático de usuarios en StarLeaf

Esta sección le guía por los pasos necesarios para configurar el servicio de aprovisionamiento de Azure AD para crear, actualizar y deshabilitar usuarios o grupos en StarLeaf en función de las asignaciones de grupos y usuarios de Azure AD.

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Aplicaciones empresariales** y luego **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **StarLeaf**.

    ![Vínculo a StarLeaf en la lista de aplicaciones](common/all-applications.png)

3. Seleccione la pestaña **Aprovisionamiento**.

    ![Pestaña Aprovisionamiento](common/provisioning.png)

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Pestaña Aprovisionamiento](common/provisioning-automatic.png)

5. En la sección Credenciales de administrador, escriba los valores de **URL base de SCIM** y **Token de acceso** que recuperó anteriormente en **URL de inquilino** y **Token secreto** respectivamente. Haga clic en **Probar conexión** para asegurarse de que Azure AD puede conectarse a StarLeaf. Si la conexión no se establece, asegúrese de que la cuenta de StarLeaf tiene permisos de administrador e inténtelo de nuevo.

    ![URL de inquilino + Token](common/provisioning-testconnection-tenanturltoken.png)

6. En el campo **Correo electrónico de notificación**, escriba la dirección de correo electrónico de una persona o grupo que debe recibir las notificaciones de error de aprovisionamiento y active la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Correo electrónico de notificación](common/provisioning-notification-email.png)

7. Haga clic en **Save**(Guardar).

8. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Users to StarLeaf** (Sincronizar usuarios de Azure Active Directory con StarLeaf).

    ![StarLeaf: Crear token](media/starleaf-provisioning-tutorial/usermapping.png)

9. Revise los atributos de usuario que se sincronizan entre Azure AD y StarLeaf en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con las cuentas de usuario de StarLeaf con el objetivo de realizar operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

    ![StarLeaf: Crear token](media/starleaf-provisioning-tutorial/userattribute.png)


10. Para configurar filtros de ámbito, consulte las siguientes instrucciones, que se proporcionan en el artículo [Aprovisionamiento de aplicaciones basado en atributos con filtros de ámbito](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).


11. Para habilitar el aprovisionamiento del servicio de aprovisionamiento de Azure AD para StarLeaf, cambie el **estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

    ![Estado de aprovisionamiento activado](common/provisioning-toggle-on.png)

12. Elija los valores deseados en **Ámbito**, en la sección **Configuración**, para definir los usuarios o grupos que quiere que se aprovisionen en StarLeaf.

    ![Ámbito del aprovisionamiento](common/provisioning-scope.png)

13. Cuando esté listo para realizar el aprovisionamiento, haga clic en **Guardar**.

    ![Guardar la configuración de aprovisionamiento](common/provisioning-configuration-save.png)

Esta operación inicia la sincronización inicial de todos los usuarios o grupos definidos en **Ámbito** en la sección **Configuración**. La sincronización inicial tarda más tiempo en realizarse que las posteriores, que se producen aproximadamente cada 40 minutos si el servicio de aprovisionamiento de Azure AD está ejecutándose. Puede usar la sección **Detalles de sincronización** para supervisar el progreso y seguir los vínculos al informe de actividad de aprovisionamiento, donde se describen todas las acciones que ha llevado a cabo el servicio de aprovisionamiento de Azure AD en StarLeaf.

Para más información sobre cómo leer los registros de aprovisionamiento de Azure AD, consulte [Creación de informes sobre el aprovisionamiento automático de cuentas de usuario](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="connector-limitations"></a>Limitaciones del conector

* Actualmente StarLeaf no admite el aprovisionamiento de grupos. 
* StarLeaf requiere que los valores **email** y **userName** tengan el mismo valor de origen.

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

* Aprenda a [revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md).
