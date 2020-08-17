---
title: 'Tutorial: Configuración del aprovisionamiento de entrada de SuccessFactors en Azure Active Directory | Microsoft Docs'
description: Aprenda a configurar el aprovisionamiento de entrada de SuccessFactors en Azure AD.
services: active-directory
author: cmmdesai
documentationcenter: na
manager: jodadzie
ms.assetid: fac4f61e-d942-4429-a298-9ba74db95077
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 08/05/2020
ms.author: chmutali
ms.openlocfilehash: 97bda09870949846e333cc2845286512fd5d6386
ms.sourcegitcommit: 85eb6e79599a78573db2082fe6f3beee497ad316
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87809908"
---
# <a name="tutorial-configure-sap-successfactors-to-azure-ad-user-provisioning"></a>Tutorial: Configuración del aprovisionamiento de usuarios de SAP SuccessFactors en Azure AD
El objetivo de este tutorial es mostrar los pasos que debe realizar para aprovisionar los datos de trabajadores de SuccessFactors Employee Central en Azure Active Directory, con escritura diferida opcional de la dirección de correo electrónico en SuccessFactors. 

>[!NOTE]
>Use este tutorial si los usuarios que quiere aprovisionar desde SuccessFactors son usuarios que solo están en la nube y no necesitan una cuenta de AD local. Si los usuarios necesitan solo una cuenta de AD local o bien una cuenta de AD y de Azure AD, consulte el tutorial sobre [configuración del aprovisionamiento de usuarios de SAP SuccessFactors a Active Directory](sap-successfactors-inbound-provisioning-tutorial.md#overview). 

## <a name="overview"></a>Información general

El [servicio de aprovisionamiento de usuarios de Azure Active Directory](../app-provisioning/user-provisioning.md) se integra con [SuccessFactors Employee Central](https://www.successfactors.com/products-services/core-hr-payroll/employee-central.html) para administrar el ciclo de vida de identidad de los usuarios. 

Los flujos de trabajo de aprovisionamiento de usuarios de SuccessFactors que admite el servicio de aprovisionamiento de usuarios de Azure AD permiten la automatización de los siguientes escenarios de recursos humanos y de administración del ciclo de vida de identidad:

* **Contratación de nuevos empleados**: cuando se agrega un nuevo empleado a SuccessFactors, se crea automáticamente una cuenta de usuario en Azure Active Directory y, opcionalmente, en Office 365 y en [otras aplicaciones SaaS compatibles con Azure AD](../app-provisioning/user-provisioning.md), con escritura diferida de la dirección de correo electrónico en SuccessFactors.

* **Actualizaciones de atributos y perfiles de empleados**: si se actualiza un registro de empleado en SuccessFactors (por ejemplo, el nombre, el cargo o el jefe), su cuenta de usuario se actualizará automáticamente en Azure Active Directory y, opcionalmente, en Office 365 y en [otras aplicaciones SaaS compatibles con Azure AD](../app-provisioning/user-provisioning.md).

* **Despidos de empleados**: cuando se prescinde de un empleado en SuccessFactors, su cuenta de usuario se deshabilita automáticamente en Azure Active Directory y, opcionalmente, en Office 365 y en [otras aplicaciones SaaS compatibles con Azure AD](../app-provisioning/user-provisioning.md).

* **Recontrataciones de empleados**: cuando se vuelve a contratar a un empleado en SuccessFactors, se puede reactivar o volver a aprovisionar automáticamente su cuenta antigua (en función de las preferencias) en Azure Active Directory y, opcionalmente, en Office 365 y en [otras aplicaciones SaaS compatibles con Azure AD](../app-provisioning/user-provisioning.md).

### <a name="who-is-this-user-provisioning-solution-best-suited-for"></a>¿Para quién es más adecuada esta solución de aprovisionamiento de usuarios?

Esta solución de aprovisionamiento de usuarios de SuccessFactors en Azure Active Directory es adecuada para:

* Organizaciones que desean una solución basada en la nube pregenerada para el aprovisionamiento de usuarios de SuccessFactors

* Organizaciones que requieren aprovisionamiento directo de usuarios de SuccessFactors en Azure Active Directory

* Organizaciones que requieren que los usuarios se aprovisionen con los datos obtenidos de [SuccessFactors Employee Central (EC)](https://www.successfactors.com/products-services/core-hr-payroll/employee-central.html)

* Organizaciones que usan Office 365 para el correo electrónico

## <a name="solution-architecture"></a>Arquitectura de la solución

En esta sección se describe la arquitectura de la solución de aprovisionamiento de usuarios de un extremo a otro para usuarios que solo están en la nube. Hay dos flujos relacionados:

* **Flujo de datos de RR. HH. autoritativos, de SuccessFactors a Azure Active Directory**: en este flujo, los eventos de los trabajadores (por ejemplo, nuevas contrataciones, traslados o despidos) se producen en primer lugar en la nube de SuccessFactors Employee Central y, luego, los datos de eventos fluyen hacia Azure Active Directory. En función del evento, puede dar lugar a operaciones de creación, actualización, habilitación o deshabilitación en Azure AD.
* **Flujo de escritura diferida de correo electrónico, de Active Directory local a SuccessFactors**: una vez finalizada la creación de la cuenta en Azure Active Directory, el valor del atributo de correo electrónico o el UPN generado en Azure AD puede escribirse de forma diferida en SuccessFactors.

  ![Información general](./media/sap-successfactors-inbound-provisioning/sf2aad-overview.png)

### <a name="end-to-end-user-data-flow"></a>Flujo de datos de usuarios de un extremo a otro

1. El equipo de RR. HH. realiza las transacciones relacionadas con los trabajadores (empleados que se incorporan a la empresa, se trasladan o se van, o nuevas contrataciones, traslados o despidos) en SuccessFactors Employee Central.
2. El servicio de aprovisionamiento de Azure AD ejecuta sincronizaciones programadas de identidades desde SuccessFactors EC e identifica los cambios que deben procesarse para la sincronización con Active Directory local.
3. El servicio de aprovisionamiento de Azure AD determina el cambio y llama a la operación de creación, actualización, habilitación o deshabilitación del usuario en Azure AD.
4. Si está configurada la [aplicación SuccessFactors Writeback](sap-successfactors-writeback-tutorial.md), se recupera la dirección de correo electrónico del usuario de Azure AD. 
5. El servicio de aprovisionamiento de Azure AD realiza la escritura diferida del atributo de correo electrónico en SuccessFactors, en función del atributo coincidente usado.

## <a name="planning-your-deployment"></a>Planeamiento de la implementación

La configuración del aprovisionamiento de usuarios controlado por RR. HH. en la nube desde SuccessFactors a Azure AD requiere un planeamiento considerable que abarca distintos aspectos, como:

* Determinación del identificador coincidente 
* Asignación de atributos
* Transformación de atributos 
* Filtros de ámbito

Consulte el [plan de implementación de RR. HH. en la nube](../app-provisioning/plan-cloud-hr-provision.md) para instrucciones detalladas sobre estos temas. Consulte [Referencia de integración de SAP SuccessFactors](../app-provisioning/sap-successfactors-integration-reference.md) para obtener información sobre las entidades admitidas, los detalles de procesamiento y cómo personalizar la integración para distintos escenarios de recursos humanos. 

## <a name="configuring-successfactors-for-the-integration"></a>Configuración de SuccessFactors para la integración

Un requisito común de todos los conectores de aprovisionamiento de SuccessFactors es que requieren credenciales de una cuenta de SuccessFactors con los permisos correctos para invocar las API OData de SuccessFactors. En esta sección se describen los pasos para crear la cuenta de servicio de SuccessFactors y conceder los permisos adecuados. 

* [Creación e identificación de una cuenta de usuario de API en SuccessFactors](#createidentify-api-user-account-in-successfactors)
* [Creación de un rol de permisos de API](#create-an-api-permissions-role)
* [Creación de un grupo de permisos para el usuario de la API](#create-a-permission-group-for-the-api-user)
* [Concesión del rol de permisos al grupo de permisos](#grant-permission-role-to-the-permission-group)

### <a name="createidentify-api-user-account-in-successfactors"></a>Creación e identificación de una cuenta de usuario de API en SuccessFactors
Trabaje con el equipo de administración de SuccessFactors o con el asociado de implementación para crear o identificar una cuenta de usuario en SuccessFactors que se usará para invocar las API OData. Las credenciales de nombre de usuario y contraseña de esta cuenta se necesitarán al configurar las aplicaciones de aprovisionamiento en Azure AD. 

### <a name="create-an-api-permissions-role"></a>Creación de un rol de permisos de API

* Inicie sesión en SAP SuccessFactors con una cuenta de usuario que tenga acceso al centro de administración.
* Busque *Manage Permission Roles* (Administrar roles de permisos) y, después, seleccione **Manage Permission Roles** (Administrar roles de permisos) en los resultados de la búsqueda.
  ![Administración de roles de permisos](./media/sap-successfactors-inbound-provisioning/manage-permission-roles.png)
* En Permission Role List (Lista de roles de permisos), haga clic en **Create New** (Crear).
  > [!div class="mx-imgBorder"]
  > ![Creación de un rol de permisos](./media/sap-successfactors-inbound-provisioning/create-new-permission-role-1.png)
* Agregue valores en **Role Name** (Nombre de rol) y **Description** (Descripción) para el nuevo rol de permisos. El nombre y la descripción deben indicar que el rol se utilizará para los permisos de uso de la API.
  > [!div class="mx-imgBorder"]
  > ![Detalle de rol de permisos](./media/sap-successfactors-inbound-provisioning/permission-role-detail.png)
* En Permission settings (Configuración de los permisos), haga clic en **Permission...** (Permiso), desplácese hacia abajo en la lista de permisos y haga clic en **Manage Integration Tools** (Administrar herramientas de integración). Active la casilla **Allow Admin to Access to OData API through Basic Authentication** (Permitir que el administrador acceda a la API OData mediante autenticación básica).
  > [!div class="mx-imgBorder"]
  > ![Administración de herramientas de integración](./media/sap-successfactors-inbound-provisioning/manage-integration-tools.png)
* Desplácese hacia abajo en el mismo cuadro y seleccione **Employee Central API**. Agregue permisos, tal como se muestra a continuación, de lectura y edición mediante la API ODATA. Seleccione la opción de edición si tiene previsto usar la misma cuenta para el escenario de escritura diferida en SuccessFactors. 
  > [!div class="mx-imgBorder"]
  > ![Permisos de lectura y escritura](./media/sap-successfactors-inbound-provisioning/odata-read-write-perm.png)
* Haga clic en **Done** (Acabado). Haga clic en **Guardar cambios**.

### <a name="create-a-permission-group-for-the-api-user"></a>Creación de un grupo de permisos para el usuario de la API

* En el centro de administración de SuccessFactors, busque *Manage Permission Groups* (Administrar grupos de permisos) y, después, seleccione **Manage Permission Groups** (Administrar grupos de permisos) en los resultados de la búsqueda.
  > [!div class="mx-imgBorder"]
  > ![Administración de grupos de permisos](./media/sap-successfactors-inbound-provisioning/manage-permission-groups.png)
* En la ventana Manage Permission Groups (Administrar grupos de permisos), haga clic en **Create New** (Crear).
  > [!div class="mx-imgBorder"]
  > ![Incorporación de un nuevo grupo](./media/sap-successfactors-inbound-provisioning/create-new-group.png)
* Agregue un valor en Group Name (Nombre de grupo) para el nuevo grupo. El nombre del grupo debe indicar que se utilizará para los usuarios de la API.
  > [!div class="mx-imgBorder"]
  > ![Nombre del grupo de permisos](./media/sap-successfactors-inbound-provisioning/permission-group-name.png)
* Agregue miembros al grupo. Por ejemplo, puede seleccionar **Username** (Nombre de usuario) en el menú desplegable People Pool (Grupo de personas) y, a continuación, escribir el nombre de usuario de la cuenta de API que se usará para la integración. 
  > [!div class="mx-imgBorder"]
  > ![Adición de miembros del grupo](./media/sap-successfactors-inbound-provisioning/add-group-members.png)
* Haga clic en **Done** (Listo) para terminar de crear el grupo de permisos.

### <a name="grant-permission-role-to-the-permission-group"></a>Concesión del rol de permisos al grupo de permisos

* En el centro de administración de SuccessFactors, busque *Manage Permission Roles* (Administrar roles de permisos) y, después, seleccione **Manage Permission Roles** (Administrar roles de permisos) en los resultados de la búsqueda.
* En **Permission Role List** (Lista de roles de permisos), seleccione el rol que creó para los permisos de uso de la API.
* En **Grant this role to...** (Conceder este rol a), haga clic en el botón **Add...** (Agregar).
* Seleccione **Permission Group...** (Grupo de permisos) en el menú desplegable y, a continuación, haga clic en **Select...** (Seleccionar) para abrir la ventana Groups (Grupos), en la que deberá buscar y seleccionar el grupo creado anteriormente. 
  > [!div class="mx-imgBorder"]
  > ![Incorporación del grupo de permisos](./media/sap-successfactors-inbound-provisioning/add-permission-group.png)
* Revise la concesión del rol de permisos al grupo de permisos. 
  > [!div class="mx-imgBorder"]
  > ![Detalles de rol y grupo de permisos](./media/sap-successfactors-inbound-provisioning/permission-role-group.png)
* Haga clic en **Guardar cambios**.

## <a name="configuring-user-provisioning-from-successfactors-to-azure-ad"></a>Configuración del aprovisionamiento de usuarios de SuccessFactors en Azure AD

En esta sección se proporcionan los pasos para el aprovisionamiento de cuentas de usuario de SuccessFactors en Azure AD.

* [Incorporación de la aplicación del conector de aprovisionamiento y configuración de la conectividad con SuccessFactors](#part-1-add-the-provisioning-connector-app-and-configure-connectivity-to-successfactors)
* [Configuración de las asignaciones de atributos](#part-2-configure-attribute-mappings)
* [Habilitar e iniciar el aprovisionamiento de usuarios](#enable-and-launch-user-provisioning)

### <a name="part-1-add-the-provisioning-connector-app-and-configure-connectivity-to-successfactors"></a>Parte 1: Incorporación de la aplicación del conector de aprovisionamiento y configuración de la conectividad con SuccessFactors

**Para configurar SuccessFactors para el aprovisionamiento a Azure AD:**

1. Vaya a <https://portal.azure.com>.

2. En la barra de navegación de la izquierda, seleccione **Azure Active Directory**.

3. Seleccione **Enterprise Applications** (Aplicaciones empresariales) y **All Applications** (Todas las aplicaciones).

4. Seleccione **Add an application** (Agregar una aplicación) y seleccione la categoría **All** (Todos).

5. Busque **SuccessFactors to Azure Active Directory User Provisioning** y agregue esa aplicación desde la galería.

6. Una vez que se haya agregado la aplicación y se muestre la pantalla de detalles de la aplicación, seleccione **Aprovisionamiento**.

7. Cambie el **Modo de** **aprovisionamiento** a **Automático**.

8. Cumplimente la sección **Credenciales de administrador** del siguiente modo:

   * **Nombre de usuario de administrador**: escriba el nombre de usuario de la cuenta de usuario de SuccessFactors API, con el identificador de empresa anexado. El formato es: **username\@companyID**

   * **Contraseña de administrador:** escriba la contraseña de la cuenta de usuario de SuccessFactors API. 

   * **URL de inquilino:** escriba el nombre del punto de conexión de servicios de la API OData de SuccessFactors. Especifique solo el nombre de host del servidor sin http o https. Este valor debería ser similar al siguiente: **nombre-servidor-api.successfactors.com**.

   * **Correo electrónico de notificación**: escriba su dirección de correo electrónico y marque la casilla "Send email if failure occurs" (Enviar una notificación por correo electrónico cuando se produzca un error).
    > [!NOTE]
    > El servicio de aprovisionamiento de Azure AD envía la notificación por correo electrónico si el trabajo de aprovisionamiento entra en un estado de[cuarentena](/azure/active-directory/manage-apps/application-provisioning-quarantine-status).

   * Haga clic en el botón **Probar conexión**. Si la prueba de conexión se lleva a cabo correctamente, haga clic en el botón **Guardar** situado en la parte superior. Si se produce un error, compruebe que las credenciales y la dirección URL de SuccessFactors son válidas.
    >[!div class="mx-imgBorder"]
    >![Azure Portal](./media/sap-successfactors-inbound-provisioning/sf2aad-provisioning-creds.png)

   * Una vez que las credenciales se guardan correctamente, en la sección **Asignaciones** se mostrará la asignación predeterminada **Synchronize SuccessFactors Users to Azure Active Directory** (Sincronizar usuarios de SuccessFactors con Azure Active Directory).

### <a name="part-2-configure-attribute-mappings"></a>Parte 2: configuración de las asignaciones de atributos

En esta sección, configurará cómo fluyen los datos de los usuarios de SuccessFactors a Active Directory.

1. En la pestaña Aprovisionamiento, en **Asignaciones**, haga clic en **Synchronize SuccessFactors Users to Azure Active Directory** (Sincronizar usuarios de SuccessFactors con Azure Active Directory).

1. En el campo **Ámbito de objeto de origen** puede seleccionar los conjuntos de usuarios de SuccessFactors que deben estar en el ámbito para el aprovisionamiento en Azure AD; para ello debe definir un conjunto de filtros basados en atributos. El ámbito predeterminado es "todos los usuarios de SuccessFactors". Filtros de ejemplo:

   * Ejemplo: Ámbito de los usuarios con personIdExternal entre 1 y 2 millones (excepto 2 millones)

      * Atributo: personIdExternal

      * Operador: REGEX Match

      * Valor: (1[0-9][0-9][0-9][0-9][0-9][0-9])

   * Ejemplo: solo los empleados, no los trabajadores temporales

      * Atributo: EmployeeID

      * Operador: IS NOT NULL

   > [!TIP]
   > Al configurar la aplicación de aprovisionamiento por primera vez, deberá probar y verificar las expresiones y asignaciones de atributos para asegurarse de que ofrece el resultado deseado. Microsoft recomienda usar los filtros de ámbito de **Ámbito de objeto de origen** para probar las asignaciones con algunos usuarios de prueba de SuccessFactors. Una vez haya verificado que las asignaciones funcionan, puede quitar el filtro o expandirlo gradualmente para incluir más usuarios.

   > [!CAUTION] 
   > El comportamiento predeterminado del motor de aprovisionamiento es deshabilitar o eliminar usuarios que salen del ámbito. Puede que esta no sea la situación deseable en la integración de SuccessFactors con Azure AD. Para invalidar este comportamiento predeterminado, consulte el artículo [Omisión de la eliminación de usuarios fuera del ámbito](../app-provisioning/skip-out-of-scope-deletions.md).
  
1. En el campo **Acciones del objeto de destino**, puede filtrar de forma global qué acciones se realizan en Active Directory. **Crear** y **Actualizar** son las más habituales.

1. En la sección **Asignaciones de atributos**, puede definir cómo se asignan los distintos atributos de SuccessFactors a los atributos de Active Directory.

  >[!NOTE]
  >Para obtener la lista completa de atributos de SuccessFactors compatibles con la aplicación, consulte [Referencia de atributos de SuccessFactors](../app-provisioning/sap-successfactors-attribute-reference.md).


1. Haga clic en una asignación de atributos existente para actualizarla o haga clic en **Agregar nueva asignación** en la parte inferior de la pantalla para agregar asignaciones nuevas. Las asignaciones de atributos admiten estas propiedades:

      * **Tipo de asignación**

         * **Directo**: escribe el valor del atributo de SuccessFactors en el atributo de AD, sin cambios.

         * **Constante**: escribe un valor de cadena estático y constante en el atributo de AD.

         * **Expresión**: permite escribir un valor personalizado en el atributo de AD, en función de uno o varios atributos de SuccessFactors. [Para obtener más información, consulte este artículo sobre las expresiones](../app-provisioning/functions-for-customizing-application-data.md).

      * **Atributo de origen**: el atributo de usuario de SuccessFactors.

      * **Valor predeterminado**: opcional. Si el atributo de origen tiene un valor vacío, la asignación escribirá este valor.
            La configuración más habitual consiste en dejarlo en blanco.

      * **Atributo de destino**: atributo de usuario de Active Directory.

      * **Hacer coincidir objetos con este atributo**: especifica si se debe usar o no esta asignación para identificar de forma unívoca a los usuarios entre SuccessFactors y Active Directory. Este valor se suele establecer en el campo Worker ID (Id. del trabajador) de SuccessFactors, que se suele asignar a uno de los atributos de identificador de empleado de Active Directory.

      * **Precedencia de coincidencia**: se pueden establecer varios atributos coincidentes. Si hay varios, se evalúan en el orden definido por este campo. En el momento en que se encuentre una coincidencia, no se evaluarán más atributos coincidentes.

      * **Aplicar esta asignación**

         * **Siempre**: esta asignación se aplica a las acciones de creación y actualización de usuarios

         * **Solo durante la creación**: esta asignación se aplica solo a las acciones de creación de usuarios

1. Para guardar las asignaciones, haga clic en **Guardar** en la parte superior de la sección Asignación de atributos.

Una vez completada la configuración de la asignación de atributos, ahora puede [habilitar e iniciar el servicio de aprovisionamiento de usuarios](#enable-and-launch-user-provisioning).

## <a name="enable-and-launch-user-provisioning"></a>Habilitar e iniciar el aprovisionamiento de usuarios

Una vez completadas las configuraciones de la aplicación de aprovisionamiento de SuccessFactors, puede activar el servicio de aprovisionamiento en Azure Portal.

> [!TIP]
> De forma predeterminada, al activar el servicio de aprovisionamiento, se iniciarán las operaciones de aprovisionamiento para todos los usuarios del ámbito. Si hay errores en la asignación o problemas con los datos de Workday, puede que se produzcan errores con el trabajo de aprovisionamiento y que entre en estado de cuarentena. Para evitar esto, como procedimiento recomendado, es conveniente configurar el filtro **Ámbito de objeto de origen** y probar las asignaciones de atributos con algunos usuarios de prueba antes de iniciar la sincronización completa de todos los usuarios. Una vez haya verificado que las asignaciones funcionan y que obtiene los resultados deseados, puede quitar el filtro o expandirlo gradualmente para incluir más usuarios.

1. En la pestaña **Aprovisionamiento**, establezca **Estado de aprovisionamiento** en **Activado**.

2. Haga clic en **Save**(Guardar).

3. Esta operación dará comienzo a la sincronización inicial, que puede tardar una cantidad de horas variable, según el número de usuarios que haya en el inquilino de SuccessFactors. Puede consultar en la barra de progreso el seguimiento del progreso del ciclo de sincronización. 

4. En cualquier momento, compruebe la pestaña **Registros de auditoría** en Azure Portal para ver las acciones que ha realizado el servicio de aprovisionamiento. Los registros de auditoría muestran todos los eventos de sincronización individuales realizados por el servicio de aprovisionamiento, por ejemplo, los usuarios que se leen fuera de Workday y que luego se agregan o actualizan en Active Directory. 

5. Una vez completada la sincronización inicial, se escribe un informe resumido de auditoría en la pestaña **Aprovisionamiento**, tal y como se muestra a continuación.

   > [!div class="mx-imgBorder"]
   > ![Barra de progreso de aprovisionamiento](./media/sap-successfactors-inbound-provisioning/prov-progress-bar-stats.png)

## <a name="next-steps"></a>Pasos siguientes

* [Más información sobre los atributos de SuccessFactors compatibles para el aprovisionamiento de entrada](../app-provisioning/sap-successfactors-attribute-reference.md)
* [Aprenda a configurar la escritura diferida de correo electrónico en SuccessFactors](sap-successfactors-writeback-tutorial.md)
* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md)
* [Aprenda a configurar el inicio de sesión único entre SuccessFactors y Azure Active Directory](successfactors-tutorial.md)
* [Aprenda a integrar otras aplicaciones SaaS con Azure Active Directory](tutorial-list.md)
* [Aprenda a exportar e importar las configuraciones de aprovisionamiento](../app-provisioning/export-import-provisioning-configuration.md)


