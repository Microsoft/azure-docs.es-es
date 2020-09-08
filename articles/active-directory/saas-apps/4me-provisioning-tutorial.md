---
title: 'Tutorial: Configuración de 4me para el aprovisionamiento automático de usuarios con Azure Active Directory | Microsoft Docs'
description: Aprenda a configurar Azure Active Directory para aprovisionar y cancelar automáticamente el aprovisionamiento de cuentas de usuario en 4me.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: article
ms.date: 06/3/2019
ms.author: jeedes
ms.openlocfilehash: beb580a02e1db80cf2d74f8167a98c9ead170810
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88538780"
---
# <a name="tutorial-configure-4me-for-automatic-user-provisioning"></a>Tutorial: Configuración de 4me para el aprovisionamiento automático de usuarios

El objetivo de este tutorial es mostrar los pasos que se realizan en 4me y Azure Active Directory (Azure AD) para configurar Azure AD a fin de aprovisionar y cancelar automáticamente el aprovisionamiento de usuarios o grupos en 4me.

> [!NOTE]
> Este tutorial describe un conector que se crea sobre el servicio de aprovisionamiento de usuarios de Azure AD. Para obtener información importante acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Este conector está actualmente en versión preliminar pública. Para más información sobre los términos de uso generales de Microsoft Azure para las características en versión preliminar, consulte [Términos de uso complementarios para las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prerrequisitos

En el escenario descrito en este tutorial se supone que ya cuenta con los requisitos previos siguientes:

* Un inquilino de Azure AD
* [Un inquilino de 4me](https://www.4me.com/trial/)
* Una cuenta de usuario de 4me con permisos de administrador.

## <a name="add-4me-from-the-gallery"></a>Adición de 4me desde la galería

Antes de configurar 4me para el aprovisionamiento automático de usuarios con Azure AD, es preciso agregar 4me desde la galería de aplicaciones de Azure AD a la lista de aplicaciones SaaS administradas.

**Para agregar 4me desde la galería de aplicaciones de Azure AD, siga estos pasos:**

1. En **[Azure Portal](https://portal.azure.com)** , en el panel de navegación izquierdo, seleccione **Azure Active Directory**.

    ![Botón Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una nueva aplicación, seleccione el botón **Nueva aplicación** en la parte superior del panel.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **4me**, seleccione **4me** en el panel de resultados y luego haga clic en el botón **Agregar** para agregar la aplicación.

    ![4me en la lista de resultados](common/search-new-app.png)

## <a name="assigning-users-to-4me"></a>Asignación de usuarios a 4me

Azure Active Directory usa un concepto denominado *asignaciones* para determinar qué usuarios deben recibir acceso a determinadas aplicaciones. En el contexto del aprovisionamiento automático de usuarios, solo se sincronizan los usuarios y grupos que se han asignado a una aplicación en Azure AD.

Antes de configurar y habilitar el aprovisionamiento automático de usuarios, debe decidir qué usuarios o grupos de Azure AD necesitan acceder a 4me. Una vez que lo decida, puede seguir estas instrucciones para asignar dichos usuarios o grupos a 4me:

* [Asignar un usuario o grupo a una aplicación empresarial](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-4me"></a>Sugerencias importantes para asignar usuarios a 4me

* Se recomienda asignar un único usuario de Azure AD a 4me para probar la configuración de aprovisionamiento automático de usuarios. Más tarde, se pueden asignar otros usuarios o grupos.

* Al asignar un usuario a 4me, debe seleccionar un rol válido específico de la aplicación (si está disponible) en el cuadro de diálogo de asignación. Los usuarios con el rol de **Acceso predeterminado** quedan excluidos del aprovisionamiento.

## <a name="configuring-automatic-user-provisioning-to-4me"></a>Configuración del aprovisionamiento automático de usuarios en 4me 

Esta sección le guía por los pasos necesarios para configurar el servicio de aprovisionamiento de Azure AD para crear, actualizar y deshabilitar usuarios o grupos en 4me en función de las asignaciones de grupos y usuarios de Azure AD.

> [!TIP]
> También puede optar por habilitar el inicio de sesión único basado en SAML para 4me siguiendo las instrucciones del [tutorial de inicio de sesión único de 4me](4me-tutorial.md). El inicio de sesión único puede configurarse independientemente del aprovisionamiento automático de usuarios, aunque estas dos características se complementan entre sí.

### <a name="to-configure-automatic-user-provisioning-for-4me-in-azure-ad"></a>Para configurar el aprovisionamiento automático de usuarios para 4me en Azure AD:

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Aplicaciones empresariales** y luego **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **4me**.

    ![Vínculo a 4me en la lista de aplicaciones](common/all-applications.png)

3. Seleccione la pestaña **Aprovisionamiento**.

    ![Pestaña Aprovisionamiento](common/provisioning.png)

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Pestaña Aprovisionamiento](common/provisioning-automatic.png)

5. Para recuperar los valores de la **URL de inquilino** y el **token secreto** de su cuenta 4me, siga el tutorial, tal y como se describe en el paso 6.

6. Inicie sesión en la consola de administración de 4me. Navegue a **Settings** (Configuración).

    ![Configuración de 4me](media/4me-provisioning-tutorial/4me01.png)

    Escriba **apps** en la barra de búsqueda.

    ![Aplicaciones de 4me](media/4me-provisioning-tutorial/4me02.png)

    Abra la lista desplegable **SCIM** para recuperar el token secreto y el punto de conexión de SCIM.

    ![SCIM de 4me](media/4me-provisioning-tutorial/4me03.png)

7. Tras rellenar los campos que se muestran en el paso 5, haga clic en **Probar conexión** para asegurarse de que Azure AD puede conectarse a 4me. Si la conexión no se establece, asegúrese de que la cuenta de 4me tiene permisos de administrador y pruebe de nuevo.

    ![Token](common/provisioning-testconnection-tenanturltoken.png)

8. En el campo **Correo electrónico de notificación**, escriba la dirección de correo electrónico de una persona o grupo que debe recibir las notificaciones de error de aprovisionamiento y active la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Correo electrónico de notificación](common/provisioning-notification-email.png)

9. Haga clic en **Save**(Guardar).

10. En la sección **Asignaciones**, seleccione Synchronize **Azure Active Directory Users to 4me** (Sincronizar usuarios de Azure Active Directory con 4me).

    ![Asignaciones de usuario de 4me](media/4me-provisioning-tutorial/4me-user-mapping.png)
    
11. Examine los atributos de grupo que se sincronizan entre Azure AD y 4me en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades **Coincidentes** se usan para establecer correspondencia con las cuentas del usuario en 4me a fin de realizar operaciones de actualización. Compruebe que [4me admite el filtrado](https://developer.4me.com/v1/scim/users/) por el atributo coincidente que ha elegido. Seleccione el botón **Guardar** para confirmar los cambios.

    ![Asignaciones de usuario de 4me](media/4me-provisioning-tutorial/4me-user-attributes.png)
    
12. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Groups to 4me** (Sincronizar grupos de Azure Active Directory con 4me).

    ![Asignaciones de usuario de 4me](media/4me-provisioning-tutorial/4me-group-mapping.png)
    
13. Examine los atributos de grupo que se sincronizan entre Azure AD y 4me en la sección **Asignación de atributos**. Los atributos seleccionados como propiedades **Coincidentes** se usan para establecer correspondencia con las cuentas del usuario en 4me a fin de realizar operaciones de actualización. Seleccione el botón **Guardar** para confirmar los cambios.

    ![Asignaciones de grupos de 4me](media/4me-provisioning-tutorial/4me-group-attribute.png)

14. Para configurar filtros de ámbito, consulte las siguientes instrucciones, que se proporcionan en el artículo [Aprovisionamiento de aplicaciones basado en atributos con filtros de ámbito](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

15. Para habilitar el servicio de aprovisionamiento de Azure AD para 4me, cambie la opción **Estado de aprovisionamiento** a **Activado** en la sección **Configuración**.

    ![Estado de aprovisionamiento activado](common/provisioning-toggle-on.png)

16. Elija los valores deseados en **Ámbito**, en la sección **Configuración**, para definir los usuarios o grupos que desea que se aprovisionen en 4me.

    ![Ámbito del aprovisionamiento](common/provisioning-scope.png)

17. Cuando esté listo para realizar el aprovisionamiento, haga clic en **Guardar**.

    ![Guardar la configuración de aprovisionamiento](common/provisioning-configuration-save.png)

Esta operación inicia la sincronización inicial de todos los usuarios o grupos definidos en **Ámbito** en la sección **Configuración**. La sincronización inicial tarda más tiempo en realizarse que las posteriores, que se producen aproximadamente cada 40 minutos si el servicio de aprovisionamiento de Azure AD está ejecutándose. Puede usar la sección **Detalles de sincronización** para supervisar el progreso y seguir los vínculos al informe de actividad de aprovisionamiento, donde se describen todas las acciones que ha llevado a cabo el servicio de aprovisionamiento de Azure AD en 4me.

Para más información sobre cómo leer los registros de aprovisionamiento de Azure AD, consulte el tutorial de [Creación de informes sobre el aprovisionamiento automático de cuentas de usuario](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="connector-limitations"></a>Limitaciones del conector

* 4me tiene distintas direcciones URL de punto de conexión SCIM para entornos de prueba y producción. La primera finaliza con **.qa** y la última con **.com**.
* La fecha de expiración de los tokens secretos de 4me generados es de un mes a partir de su generación.
* 4me no admite operaciones **DELETE**.

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md)
