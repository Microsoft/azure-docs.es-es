---
title: 'Tutorial: Configuración de Zscaler One para el aprovisionamiento automático de usuarios con Azure Active Directory | Microsoft Docs'
description: Obtenga información sobre cómo configurar Azure Active Directory para aprovisionar y cancelar automáticamente el aprovisionamiento de cuentas de usuario de Zscaler One.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: article
ms.date: 03/27/2019
ms.author: jeedes
ms.openlocfilehash: 978f8ff5e57a5a9a1df10152713c8c42901b3b6f
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88545869"
---
# <a name="tutorial-configure-zscaler-one-for-automatic-user-provisioning"></a>Tutorial: Configuración de Zscaler One para el aprovisionamiento automático de usuarios

Este tutorial muestra los pasos que se deben realizar en Zscaler One y Azure Active Directory (Azure AD) para configurar Azure AD con el objetivo de aprovisionar y desaprovisionar automáticamente usuarios y grupos en Zscaler One.

> [!NOTE]
> En este tutorial se describe un conector que se basa en el servicio de aprovisionamiento de usuarios de Azure AD. Para obtener información acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../active-directory-saas-app-provisioning.md).


## <a name="prerequisites"></a>Prerrequisitos

En el escenario que se describe en este tutorial se supone que ya cuenta con los elementos siguientes:

* Un inquilino de Azure AD.
* Un inquilino de Zscaler One.
* Una cuenta de usuario de Zscaler One con permisos de administrador.

> [!NOTE]
> La integración del aprovisionamiento de Azure AD se basa en la API SCIM de Zscaler One. Esta API está disponible para los desarrolladores de Zscaler One para las cuentas con el paquete Enterprise.

## <a name="add-zscaler-one-from-the-azure-marketplace"></a>Adición de Zscaler One desde Azure Marketplace

Antes de configurar Zscaler One para el aprovisionamiento automático de usuarios con Azure AD, agregue Zscaler One desde Azure Marketplace en su lista de aplicaciones SaaS administradas.

Para eliminar Zscaler One de Marketplace, siga estos pasos.

1. En el panel de navegación de la izquierda de [Azure Portal](https://portal.azure.com), seleccione **Azure Active Directory**.

    ![Icono de Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una aplicación nueva, en la parte superior del cuadro de diálogo, seleccione **Nueva aplicación**.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Zscaler One** y seleccione **Zscaler One** del panel de resultados. Para agregar la aplicación, seleccione **Agregar**.

    ![Zscaler One en la lista de resultados](common/search-new-app.png)

## <a name="assign-users-to-zscaler-one"></a>Asignación de usuarios a Zscaler One

Azure Active Directory usa un concepto denominado *asignaciones* para determinar qué usuarios deben recibir acceso a determinadas aplicaciones. En el contexto de aprovisionamiento automático de usuarios, solo se sincronizan los usuarios o grupos que se han asignado a una aplicación en Azure AD.

Antes de configurar y habilitar el aprovisionamiento automático de usuarios, decida qué usuarios o grupos de Azure AD necesitan acceder a Zscaler One. Para asignar estos usuarios o grupos a Zscaler One, siga las instrucciones de [Asignación de un usuario o un grupo a una aplicación empresarial](https://docs.microsoft.com/azure/active-directory/active-directory-coreapps-assign-user-azure-portal).

### <a name="important-tips-for-assigning-users-to-zscaler-one"></a>Sugerencias importantes para asignar usuarios a Zscaler One

* Se recomienda asignar primero un usuario individual de Azure AD a Zscaler One para probar la configuración del aprovisionamiento automático de usuarios. Asigne usuarios o grupos adicionales más tarde.

* Cuando asigne un usuario a Zscaler One, seleccione un rol específico de la aplicación válido (si está disponible) en el cuadro de diálogo de asignación. Los usuarios con el rol de **Acceso predeterminado** quedan excluidos del aprovisionamiento.

## <a name="configure-automatic-user-provisioning-to-zscaler-one"></a>Configuración del aprovisionamiento automático de usuarios en Zscaler One

Esta sección le guía por los pasos para configurar el servicio de aprovisionamiento de Azure AD. Úselo para crear, actualizar y deshabilitar usuarios o grupos en Zscaler One en función de las asignaciones de usuarios o grupos en Azure AD.

> [!TIP]
> También puede habilitar el inicio de sesión único basado en SAML para Zscaler One. Si lo hace, siga las instrucciones del [Tutorial del inicio de sesión único de Zscaler One](zscaler-One-tutorial.md). El inicio de sesión único puede configurarse independientemente del aprovisionamiento automático de usuarios, aunque estas dos características se complementan entre sí.

### <a name="configure-automatic-user-provisioning-for-zscaler-one-in-azure-ad"></a>Configuración del aprovisionamiento automático de usuarios para Zscaler One en Azure AD

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Aplicaciones empresariales** > **Todas las aplicaciones** > **Zscaler One**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Zscaler One**.

    ![Vínculo de Zscaler One en la lista de aplicaciones](common/all-applications.png)

3. Seleccione la pestaña **Aprovisionamiento**.

    ![Aprovisionamiento de Zscaler One](./media/zscaler-one-provisioning-tutorial/provisioning-tab.png)

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Modo de aprovisionamiento de Zscaler One](./media/zscaler-one-provisioning-tutorial/provisioning-credentials.png)

5. En la sección **Credenciales de administrador**, rellene los cuadros de **URL de inquilino** y **Token secreto** con las opciones de configuración para la cuenta de Zscaler One, como se describe en el paso 6.

6. Para obtener los valores de la URL de inquilino y el Token secreto, vaya a **Administración** > **Configuración de autenticación** en la interfaz de usuario del portal de Zscaler One. En **Tipo de autenticación**, seleccione **SAML**.

    ![Configuración de autenticación de Zscaler One](./media/zscaler-one-provisioning-tutorial/secret-token-1.png)

    a. Seleccione **Configure SAML** (Configurar SAML) para abrir las opciones de **Configure SAML** (Configurar SAML).

    ![Configuración de SAML de Zscaler One](./media/zscaler-one-provisioning-tutorial/secret-token-2.png)

    b. Seleccione **Enable SCIM-Based Provisioning** (Habilitar aprovisionamiento basado en SCIM) para recuperar la configuración de **URL base** y **Token de portador**. Después, guarde la configuración. Copie la configuración de **URL Base** en la **URL del inquilino** en Azure Portal. Copie la configuración de **Token de portador** en el **Token secreto** en Azure Portal.

7. Después de rellenar los cuadros que se muestran en el paso 5, seleccione **Probar conexión** para asegurarse de que Azure AD puede conectarse a Zscaler One. Si la conexión no se establece, asegúrese de que la cuenta de Zscaler One tiene permisos de administrador e inténtelo de nuevo.

    ![Prueba de conexión de Zscaler One](./media/zscaler-one-provisioning-tutorial/test-connection.png)

8. En el cuadro **Correo electrónico de notificación**, escriba la dirección de correo electrónico de la persona o el grupo que deben recibir las notificaciones de error de aprovisionamiento. Seleccione la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Notificación por correo electrónico de Zscaler One](./media/zscaler-one-provisioning-tutorial/notification.png)

9. Seleccione **Guardar**.

10. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Users to Zscaler One** (Sincronizar usuarios de Azure Active Directory con Zscaler One).

    ![Sincronización de usuarios de Zscaler One](./media/zscaler-one-provisioning-tutorial/user-mappings.png)

11. Examine los atributos de usuario que se sincronizan entre Azure AD y Zscaler One en la sección **Asignaciones de atributos**. Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias entre las cuentas de usuario de Zscaler One con el objetivo de realizar operaciones de actualización. Para guardar los cambios, seleccione **Guardar**.

    ![Atributos de usuario coincidentes de Zscaler One](./media/zscaler-one-provisioning-tutorial/user-attribute-mappings.png)

12. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Groups to Zscaler One** (Sincronizar grupos de Azure Active Directory con Zscaler One).

    ![Sincronización de grupos de Zscaler One](./media/zscaler-one-provisioning-tutorial/group-mappings.png)

13. Examine los atributos de grupo que se sincronizan entre Azure AD y Zscaler One en la sección **Asignaciones de atributos**. Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con los grupos de Zscaler One para las operaciones de actualización. Para guardar los cambios, seleccione **Guardar**.

    ![Atributos de grupo coincidentes de Zscaler One](./media/zscaler-one-provisioning-tutorial/group-attribute-mappings.png)

14. Para configurar filtros de ámbito, consulte las instrucciones del [tutorial sobre filtros de ámbito](./../active-directory-saas-scoping-filters.md).

15. Para habilitar el servicio de aprovisionamiento de Azure AD para Zscaler One, vaya a la sección **Configuración** y cambie el valor de **Estado de aprovisionamiento** a **Activado**.

    ![Estado de aprovisionamiento de Zscaler One](./media/zscaler-one-provisioning-tutorial/provisioning-status.png)

16. Defina los usuarios o grupos que quiere que se aprovisionen en Zscaler One. En la sección **Configuración**, seleccione los valores que quiere incluir en el **Ámbito**.

    ![Ámbito de Zscaler One](./media/zscaler-one-provisioning-tutorial/scoping.png)

17. Cuando esté listo para realizar el aprovisionamiento, seleccione **Guardar**.

    ![Ámbito de Zscaler One](./media/zscaler-one-provisioning-tutorial/save-provisioning.png)

Esta operación inicia la sincronización inicial de todos los usuarios o grupos definidos en **Ámbito** en la sección **Configuración**. La sincronización inicial tarda más tiempo en realizarse que las sincronizaciones posteriores. Se produce aproximadamente cada 40 minutos mientras se ejecuta el servicio de aprovisionamiento de Azure AD. 

Puede usar la sección **Detalles de sincronización** para supervisar el progreso y hacer clic en los vínculos al informe de actividad de aprovisionamiento. En el informe se describen todas las acciones que el servicio de aprovisionamiento de Azure AD realiza en Zscaler One.

Para obtener información acerca de cómo leer los registros de aprovisionamiento de Azure AD, consulte [Creación de informes sobre el aprovisionamiento automático de cuentas de usuario](../active-directory-saas-provisioning-reporting.md).

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../active-directory-saas-provisioning-reporting.md)

<!--Image references-->
[1]: ./media/zscaler-one-provisioning-tutorial/tutorial-general-01.png
[2]: ./media/zscaler-one-provisioning-tutorial/tutorial-general-02.png
[3]: ./media/zscaler-one-provisioning-tutorial/tutorial-general-03.png
