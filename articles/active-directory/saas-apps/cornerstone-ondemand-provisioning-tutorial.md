---
title: 'Tutorial: Configuración de Cornerstone OnDemand para el aprovisionamiento automático de usuarios con Azure Active Directory | Microsoft Docs'
description: Aprenda a configurar Azure Active Directory para aprovisionar y cancelar el aprovisionamiento de cuentas de usuario de Cornerstone OnDemand automáticamente.
services: active-directory
author: zhchia
writer: zhchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: article
ms.date: 03/27/2019
ms.author: jeedes
ms.openlocfilehash: 4200bc9879aba5e18282123be03576cc617a5e14
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88549236"
---
# <a name="tutorial-configure-cornerstone-ondemand-for-automatic-user-provisioning"></a>Tutorial: Configuración de Cornerstone OnDemand para el aprovisionamiento automático de usuarios

Este tutorial muestra los pasos que se deben realizar en Cornerstone OnDemand y Azure Active Directory (Azure AD) para configurar Azure AD para que aprovisione y desaprovisione automáticamente los usuarios o grupos en Cornerstone OnDemand.

> [!NOTE]
> En este tutorial se describe un conector que se basa en el servicio de aprovisionamiento de usuarios de Azure AD. Para obtener información acerca de lo que hace este servicio, cómo funciona y ver preguntas frecuentes al respecto, consulte [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md).

## <a name="prerequisites"></a>Prerrequisitos

En el escenario que se describe en este tutorial se supone que ya cuenta con los elementos siguientes:

* Un inquilino de Azure AD.
* Un inquilino de Cornerstone OnDemand.
* Una cuenta de usuario de Cornerstone OnDemand con permisos de administrador.

> [!NOTE]
> La integración de aprovisionamiento de Azure AD se basa en el [servicio web de Cornerstone OnDemand](https://www.cornerstoneondemand.com/). Este servicio está disponible para los equipos de Cornerstone OnDemand.

## <a name="add-cornerstone-ondemand-from-the-azure-marketplace"></a>Adición de Cornerstone OnDemand desde Azure Marketplace

Antes de configurar Cornerstone OnDemand para el aprovisionamiento automático de usuarios con Azure AD, agregue Cornerstone OnDemand desde Azure Marketplace para enumerar las aplicaciones SaaS administradas.

Para agregar Cornerstone OnDemand desde Marketplace, siga estos pasos.

1. En el panel de navegación de la izquierda de [Azure Portal](https://portal.azure.com), seleccione **Azure Active Directory**.

    ![Icono de Azure Active Directory](common/select-azuread.png)

2. Vaya a **Aplicaciones empresariales** y seleccione **Todas las aplicaciones**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

3. Para agregar una aplicación nueva, en la parte superior del cuadro de diálogo, seleccione **Nueva aplicación**.

    ![Botón Nueva aplicación](common/add-new-app.png)

4. En el cuadro de búsqueda, escriba **Cornerstone OnDemand** y seleccione **Cornerstone OnDemand** desde el panel de resultados. Para agregar la aplicación, seleccione **Agregar**.

    ![Cornerstone OnDemand en la lista de resultados](common/search-new-app.png)

## <a name="assign-users-to-cornerstone-ondemand"></a>Asignación de usuarios a Cornerstone OnDemand

Azure Active Directory usa un concepto denominado *asignaciones* para determinar qué usuarios deben recibir acceso a determinadas aplicaciones. En el contexto de aprovisionamiento automático de usuarios, solo se sincronizan los usuarios o grupos que se han asignado a una aplicación en Azure AD.

Antes de configurar y habilitar el aprovisionamiento automático de usuarios, decida qué usuarios o grupos de Azure AD necesitan acceder a Cornerstone OnDemand. Para asignar estos usuarios o grupos a Cornerstone OnDemand, siga las instrucciones de [Asignación de un usuario o un grupo a una aplicación empresarial](../manage-apps/assign-user-or-group-access-portal.md).

### <a name="important-tips-for-assigning-users-to-cornerstone-ondemand"></a>Sugerencias importantes para asignar usuarios a Cornerstone OnDemand

* Se recomienda asignar un único usuario de Azure AD a Cornerstone OnDemand para probar la configuración de aprovisionamiento automático de usuarios. Asigne usuarios o grupos adicionales más tarde.

* Cuando asigne un usuario a Cornerstone OnDemand, seleccione algún rol específico de la aplicación válido (si está disponible) en el cuadro de diálogo de asignación. Los usuarios con el rol de **Acceso predeterminado** quedan excluidos del aprovisionamiento.

## <a name="configure-automatic-user-provisioning-to-cornerstone-ondemand"></a>Configuración del aprovisionamiento automático de usuarios en Cornerstone OnDemand

Esta sección le guía por los pasos para configurar el servicio de aprovisionamiento de Azure AD. Úsela para crear, actualizar y deshabilitar usuarios o grupos en Cornerstone OnDemand en función de las asignaciones de usuarios o grupos en Azure AD.

Para configurar el aprovisionamiento automático de usuarios para Cornerstone OnDemand en Azure AD, siga estos pasos.

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Aplicaciones empresariales** > **Todas las aplicaciones** > **Cornerstone OnDemand**.

    ![Hoja Aplicaciones empresariales](common/enterprise-applications.png)

2. En la lista de aplicaciones, seleccione **Cornerstone OnDemand**.

    ![El vínculo de Cornerstone OnDemand en la lista de aplicaciones](common/all-applications.png)

3. Seleccione la pestaña **Aprovisionamiento**.

    ![Aprovisionamiento de Cornerstone OnDemand](./media/cornerstone-ondemand-provisioning-tutorial/ProvisioningTab.png)

4. Establezca el **modo de aprovisionamiento** en **Automático**.

    ![Modo de aprovisionamiento de Cornerstone OnDemand](./media/cornerstone-ondemand-provisioning-tutorial/ProvisioningCredentials.png)

5. En la sección **Credenciales de administrador**, escriba el nombre de usuario de administrador, la contraseña de administrador y el dominio de la cuenta de Cornerstone OnDemand:

    * En el cuadro **Nombre de usuario de administrador**, rellene el dominio o nombre de usuario de la cuenta de administrador de su inquilino de Cornerstone OnDemand. Un ejemplo en contoso\admin.

    * En el cuadro **Contraseña de administrador**, rellene la contraseña correspondiente al nombre de usuario de administrador.

    * En el cuadro **Dominio**, rellene la dirección URL del servicio web del inquilino de Cornerstone OnDemand. Por ejemplo, el servicio se encuentra en `https://ws-[corpname].csod.com/feed30/clientdataservice.asmx` y para Contoso el dominio es `https://ws-contoso.csod.com/feed30/clientdataservice.asmx`. Para obtener más información sobre cómo recuperar la dirección URL del servicio web, consulte [este PDF](https://help.csod.com/help/csod_0/Content/Resources/Documents/WebServices/CSOD_Web_Services_-_User-OU_Technical_Specification_v20160222.pdf).

6. Después de rellenar los cuadros que se muestran en el paso 5, seleccione **Probar conexión** para asegurarse de que Azure AD puede conectarse a Cornerstone OnDemand. Si la conexión no se establece, asegúrese de que la cuenta de Cornerstone OnDemand tiene permisos de administrador e inténtelo de nuevo.

    ![Prueba de conexión de Cornerstone OnDemand](./media/cornerstone-ondemand-provisioning-tutorial/TestConnection.png)

7. En el cuadro **Correo electrónico de notificación**, escriba la dirección de correo electrónico de la persona o el grupo que deben recibir las notificaciones de error de aprovisionamiento. Seleccione la casilla **Enviar una notificación por correo electrónico cuando se produzca un error**.

    ![Notificación por correo electrónico de Cornerstone OnDemand](./media/cornerstone-ondemand-provisioning-tutorial/EmailNotification.png)

8. Seleccione **Guardar**.

9. En la sección **Asignaciones**, seleccione **Synchronize Azure Active Directory Users to Cornerstone OnDemand (Sincronizar usuarios de Azure Active Directory con Cornerstone OnDemand)** .

    ![Sincronización de Cornerstone OnDemand](./media/cornerstone-ondemand-provisioning-tutorial/UserMapping.png)

10. En la sección **Asignaciones de atributos**, revise los atributos de usuario que se sincronizan entre Azure AD y Cornerstone OnDemand. Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con las cuentas de usuario de Cornerstone OnDemand con el objetivo de realizar operaciones de actualización. Para guardar los cambios, seleccione **Guardar**.

    ![Asignaciones de atributos de Cornerstone OnDemand](./media/cornerstone-ondemand-provisioning-tutorial/UserMappingAttributes.png)

11. Para configurar filtros de ámbito, consulte las instrucciones del [tutorial sobre filtros de ámbito](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

12. Para habilitar el servicio de aprovisionamiento de Azure AD para Cornerstone OnDemand, vaya a la sección **Configuración** y cambie el valor de **Estado de aprovisionamiento** a **Activado**.

    ![Estado de aprovisionamiento de Cornerstone OnDemand](./media/cornerstone-ondemand-provisioning-tutorial/ProvisioningStatus.png)

13. Defina los usuarios o grupos que quiere que se aprovisionen en Cornerstone OnDemand. En la sección **Configuración**, seleccione los valores que quiere incluir en el **Ámbito**.

    ![Ámbito de Cornerstone OnDemand](./media/cornerstone-ondemand-provisioning-tutorial/SyncScope.png)

14. Cuando esté listo para realizar el aprovisionamiento, seleccione **Guardar**.

    ![Guardado de Cornerstone OnDemand](./media/cornerstone-ondemand-provisioning-tutorial/Save.png)

Esta operación inicia la sincronización inicial de todos los usuarios o grupos definidos en **Ámbito** en la sección **Configuración**. La sincronización inicial tarda más tiempo en realizarse que las sincronizaciones posteriores. Se produce aproximadamente cada 40 minutos mientras se ejecuta el servicio de aprovisionamiento de Azure AD. 

Puede usar la sección **Detalles de sincronización** para supervisar el progreso y hacer clic en los vínculos al informe de actividad de aprovisionamiento. En el informe se describen todas las acciones que el servicio de aprovisionamiento de Azure AD realiza en Cornerstone OnDemand.

Para obtener información acerca de cómo leer los registros de aprovisionamiento de Azure AD, consulte [Creación de informes sobre el aprovisionamiento automático de cuentas de usuario](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="connector-limitations"></a>Limitaciones del conector

El atributo **Position** de Cornerstone OnDemand espera un valor que corresponde a los roles del portal de Cornerstone OnDemand. Para obtener una lista de valores de **Position** válidos, vaya a **Edit User Record > Organization Structure > Position** (Editar registro de usuario > Estructura de la organización > Posición) en el portal de Cornerstone OnDemand.

![Editar un registro de usuario en el aprovisionamiento de Cornerstone OnDemand](./media/cornerstone-ondemand-provisioning-tutorial/UserEdit.png)

![Posición en el aprovisionamiento de Cornerstone OnDemand](./media/cornerstone-ondemand-provisioning-tutorial/UserPosition.png)

![Lista de posiciones en el aprovisionamiento de Cornerstone OnDemand](./media/cornerstone-ondemand-provisioning-tutorial/PostionId.png)

## <a name="additional-resources"></a>Recursos adicionales

* [Administración del aprovisionamiento de cuentas de usuario para aplicaciones empresariales](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md)

<!--Image references-->
[1]: ./media/cornerstone-ondemand-provisioning-tutorial/tutorial_general_01.png
[2]: ./media/cornerstone-ondemand-provisioning-tutorial/tutorial_general_02.png
[3]: ./media/cornerstone-ondemand-provisioning-tutorial/tutorial_general_03.png
