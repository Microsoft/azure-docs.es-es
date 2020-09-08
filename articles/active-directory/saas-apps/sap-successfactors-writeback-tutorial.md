---
title: 'Tutorial: Configuración de la escritura diferida de SAP SuccessFactors en Azure Active Directory | Microsoft Docs'
description: Aprenda a configurar la escritura diferida de atributos en SAP SuccessFactors desde Azure AD.
services: active-directory
author: cmmdesai
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.topic: article
ms.workload: identity
ms.date: 08/05/2020
ms.author: chmutali
ms.openlocfilehash: 4b048053a553176f73b5bd199bcb6e28bc74cc6c
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88534003"
---
# <a name="tutorial-configure-attribute-write-back-from-azure-ad-to-sap-successfactors"></a>Tutorial: Configuración de la escritura diferida de atributos de Azure AD en SAP SuccessFactors
El objetivo de este tutorial es mostrar los pasos para la escritura diferida de atributos de Azure AD en Employee Central de SAP SuccessFactors. 

## <a name="overview"></a>Información general

Puede configurar la aplicación de escritura diferida de SAP SuccessFactors para escribir atributos específicos de Azure Active Directory en Employee Central de SAP SuccessFactors. La aplicación de aprovisionamiento de escritura diferida de SuccessFactors admite la asignación de valores a los siguientes atributos de Employee Central:

* Correo electrónico del trabajo
* Nombre de usuario
* Número de teléfono del trabajo (incluidos el código de país, el código de área, el número y la extensión)
* Marca principal del número de teléfono del trabajo
* Número móvil (incluidos el código de país, el código de área y el número)
* Marca principal del teléfono móvil 
* Atributos custom01-custom15 del usuario
* Atributo loginMethod

> [!NOTE]
> Esta aplicación no tiene ninguna dependencia en las aplicaciones de integración de aprovisionamiento de usuarios de entrada de SuccessFactors. Puede configurarla de forma independiente de la aplicación de aprovisionamiento de [SuccessFactors en AD local](sap-successfactors-inbound-provisioning-tutorial.md) o de la aplicación de aprovisionamiento de [SuccessFactors para Azure AD](sap-successfactors-inbound-provisioning-cloud-only-tutorial.md).

### <a name="who-is-this-user-provisioning-solution-best-suited-for"></a>¿Para quién es más adecuada esta solución de aprovisionamiento de usuarios?

La solución de aprovisionamiento de usuarios SuccessFactors Writeback es idónea para:

* Organizaciones que usan Office 365 y que quieren realizar la escritura diferida de atributos autoritativos administrados por TI (como la dirección de correo electrónico, el teléfono o el nombre de usuario) en Employee Central de SuccessFactors.

## <a name="configuring-successfactors-for-the-integration"></a>Configuración de SuccessFactors para la integración

Todos los conectores de aprovisionamiento de SuccessFactors requieren credenciales de una cuenta de SuccessFactors con los permisos correctos para invocar las API OData de Employee Central. En esta sección se describen los pasos para crear la cuenta de servicio de SuccessFactors y conceder los permisos adecuados. 

* [Creación e identificación de una cuenta de usuario de API en SuccessFactors](#createidentify-api-user-account-in-successfactors)
* [Creación de un rol de permisos de API](#create-an-api-permissions-role)
* [Creación de un grupo de permisos para el usuario de la API](#create-a-permission-group-for-the-api-user)
* [Concesión del rol de permisos al grupo de permisos](#grant-permission-role-to-the-permission-group)

### <a name="createidentify-api-user-account-in-successfactors"></a>Creación e identificación de una cuenta de usuario de API en SuccessFactors
Trabaje con el equipo de administración de SuccessFactors o con el asociado de implementación para crear o identificar una cuenta de usuario en SuccessFactors que se usará para invocar las API OData. Las credenciales de nombre de usuario y contraseña de esta cuenta se necesitarán al configurar las aplicaciones de aprovisionamiento en Azure AD. 

### <a name="create-an-api-permissions-role"></a>Creación de un rol de permisos de API

1. Inicie sesión en SAP SuccessFactors con una cuenta de usuario que tenga acceso al centro de administración.
1. Busque *Manage Permission Roles* (Administrar roles de permisos) y, después, seleccione **Manage Permission Roles** (Administrar roles de permisos) en los resultados de la búsqueda.

   ![Administración de roles de permisos](./media/sap-successfactors-inbound-provisioning/manage-permission-roles.png)

1. En Permission Role List (Lista de roles de permisos), haga clic en **Create New** (Crear).

   > [!div class="mx-imgBorder"]
   > ![Creación de un rol de permisos](./media/sap-successfactors-inbound-provisioning/create-new-permission-role-1.png)

1. Agregue valores en **Role Name** (Nombre de rol) y **Description** (Descripción) para el nuevo rol de permisos. El nombre y la descripción deben indicar que el rol se utilizará para los permisos de uso de la API.

   > [!div class="mx-imgBorder"]
   > ![Detalle de rol de permisos](./media/sap-successfactors-inbound-provisioning/permission-role-detail.png)

1. En Permission settings (Configuración de los permisos), haga clic en **Permission...** (Permiso), desplácese hacia abajo en la lista de permisos y haga clic en **Manage Integration Tools** (Administrar herramientas de integración). Active la casilla **Allow Admin to Access to OData API through Basic Authentication** (Permitir que el administrador acceda a la API OData mediante autenticación básica).

   > [!div class="mx-imgBorder"]
   > ![Administración de herramientas de integración](./media/sap-successfactors-inbound-provisioning/manage-integration-tools.png)

1. Desplácese hacia abajo en el mismo cuadro y seleccione **Employee Central API**. Agregue permisos, tal como se muestra a continuación, de lectura y edición mediante la API ODATA. Seleccione la opción de edición si tiene previsto usar la misma cuenta para el escenario de escritura diferida en SuccessFactors. 

   > [!div class="mx-imgBorder"]
   > ![Permisos de lectura y escritura](./media/sap-successfactors-inbound-provisioning/odata-read-write-perm.png)

1. Haga clic en **Done** (Acabado). Haga clic en **Guardar cambios**.

### <a name="create-a-permission-group-for-the-api-user"></a>Creación de un grupo de permisos para el usuario de la API

1. En el centro de administración de SuccessFactors, busque *Manage Permission Groups* (Administrar grupos de permisos) y, después, seleccione **Manage Permission Groups** (Administrar grupos de permisos) en los resultados de la búsqueda.

   > [!div class="mx-imgBorder"]
   > ![Administración de grupos de permisos](./media/sap-successfactors-inbound-provisioning/manage-permission-groups.png)

1. En la ventana Manage Permission Groups (Administrar grupos de permisos), haga clic en **Create New** (Crear).

   > [!div class="mx-imgBorder"]
   > ![Incorporación de un nuevo grupo](./media/sap-successfactors-inbound-provisioning/create-new-group.png)

1. Agregue un valor en Group Name (Nombre de grupo) para el nuevo grupo. El nombre del grupo debe indicar que se utilizará para los usuarios de la API.

   > [!div class="mx-imgBorder"]
   > ![Nombre del grupo de permisos](./media/sap-successfactors-inbound-provisioning/permission-group-name.png)

1. Agregue miembros al grupo. Por ejemplo, puede seleccionar **Username** (Nombre de usuario) en el menú desplegable People Pool (Grupo de personas) y, a continuación, escribir el nombre de usuario de la cuenta de API que se usará para la integración. 

   > [!div class="mx-imgBorder"]
   > ![Adición de miembros del grupo](./media/sap-successfactors-inbound-provisioning/add-group-members.png)

1. Haga clic en **Done** (Listo) para terminar de crear el grupo de permisos.

### <a name="grant-permission-role-to-the-permission-group"></a>Concesión del rol de permisos al grupo de permisos

1. En el centro de administración de SuccessFactors, busque *Manage Permission Roles* (Administrar roles de permisos) y, después, seleccione **Manage Permission Roles** (Administrar roles de permisos) en los resultados de la búsqueda.
1. En **Permission Role List** (Lista de roles de permisos), seleccione el rol que creó para los permisos de uso de la API.
1. En **Grant this role to...** (Conceder este rol a), haga clic en el botón **Add...** (Agregar).
1. Seleccione **Permission Group...** (Grupo de permisos) en el menú desplegable y, a continuación, haga clic en **Select...** (Seleccionar) para abrir la ventana Groups (Grupos), en la que deberá buscar y seleccionar el grupo creado anteriormente. 

   > [!div class="mx-imgBorder"]
   > ![Incorporación del grupo de permisos](./media/sap-successfactors-inbound-provisioning/add-permission-group.png)

1. Revise la concesión del rol de permisos al grupo de permisos. 
   > [!div class="mx-imgBorder"]
   > ![Detalles de rol y grupo de permisos](./media/sap-successfactors-inbound-provisioning/permission-role-group.png)

1. Haga clic en **Guardar cambios**.

## <a name="preparing-for-successfactors-writeback"></a>Preparación de la escritura diferida de SuccessFactors

La aplicación de aprovisionamiento de reescritura de SuccessFactors usa ciertos valores de *código* para configurar el correo electrónico y los números de teléfono en Employee Central. Estos valores de *código* se establecen como valores constantes en la tabla de asignación de atributos y son diferentes para cada instancia de SuccessFactors. En esta sección se usa [Postman](https://www.postman.com/downloads/) para capturar los valores de código. Puede usar [cURL](https://curl.haxx.se/), [Fiddler](https://www.telerik.com/fiddler) o cualquier otra herramienta similar para enviar solicitudes HTTP. 

### <a name="download-and-configure-postman-with-your-successfactors-tenant"></a>Descarga y configuración de Postman con el inquilino de SuccessFactors

1. Descargue [Postman](https://www.postman.com/downloads/).
1. Cree una "nueva colección" en la aplicación Postman. Asígnele el siguiente nombre: "SuccessFactors". 

   > [!div class="mx-imgBorder"]
   > ![Nueva colección de Postman](./media/sap-successfactors-inbound-provisioning/new-postman-collection.png)

1. En la pestaña "Autorización", escriba las credenciales del usuario de la API configuradas en la sección anterior. Configure el tipo como "Autenticación básica". 

   > [!div class="mx-imgBorder"]
   > ![Autorización de Postman](./media/sap-successfactors-inbound-provisioning/postman-authorization.png)

1. Guarde la configuración. 

### <a name="retrieve-constant-value-for-emailtype"></a>Recuperación del valor constante para emailType

1. En Postman, haga clic en los puntos suspensivos (...) asociados a la colección de SuccessFactors y agregue una "nueva solicitud" denominada "Obtener tipos de correo electrónico" como se muestra a continuación. 

   > [!div class="mx-imgBorder"]
   > ![Solicitud de correo electrónico de Postman ](./media/sap-successfactors-inbound-provisioning/postman-email-request.png)

1. Abra el panel de solicitudes "Obtener tipo de correo electrónico". 
1. En Obtener la dirección URL, agregue la dirección URL siguiente, reemplazando `successFactorsAPITenantName` por el inquilino de API para la instancia de SuccessFactors. 
   `https://<successfactorsAPITenantName>/odata/v2/Picklist('ecEmailType')?$expand=picklistOptions&$select=picklistOptions/id,picklistOptions/externalCode&$format=json`

   > [!div class="mx-imgBorder"]
   > ![Tipo de correo electrónico de Postman](./media/sap-successfactors-inbound-provisioning/postman-get-email-type.png)

1. La pestaña "Autorización" heredará la autenticación configurada para la recopilación. 
1. Haga clic en "Enviar" para invocar la llamada API. 
1. En el cuerpo de la respuesta, vea el conjunto de resultados JSON y busque el identificador correspondiente a `externalCode = B`. 

   > [!div class="mx-imgBorder"]
   > ![Respuesta de tipo de correo electrónico de Postman](./media/sap-successfactors-inbound-provisioning/postman-email-type-response.png)

1. Anote este valor como la constante que se va a usar con *emailType* en la tabla de asignación de atributos.

### <a name="retrieve-constant-value-for-phonetype"></a>Recuperación del valor constante para phoneType

1. En Postman, haga clic en los puntos suspensivos (...) asociados a la colección de SuccessFactors y agregue una "nueva solicitud" denominada "Obtener tipos de teléfono" como se muestra a continuación. 

   > [!div class="mx-imgBorder"]
   > ![Solicitud de teléfono de Postman](./media/sap-successfactors-inbound-provisioning/postman-phone-request.png)

1. Abra el panel de solicitudes "Obtener tipo de teléfono". 
1. En Obtener la dirección URL, agregue la dirección URL siguiente, reemplazando `successFactorsAPITenantName` por el inquilino de API para la instancia de SuccessFactors. 
   `https://<successfactorsAPITenantName>/odata/v2/Picklist('ecPhoneType')?$expand=picklistOptions&$select=picklistOptions/id,picklistOptions/externalCode&$format=json`

   > [!div class="mx-imgBorder"]
   > ![Obtener tipo de teléfono de Postman](./media/sap-successfactors-inbound-provisioning/postman-get-phone-type.png)

1. La pestaña "Autorización" heredará la autenticación configurada para la recopilación. 
1. Haga clic en "Enviar" para invocar la llamada API. 
1. En el cuerpo de la respuesta, vea el conjunto de resultados JSON y busque el *identificador* correspondiente a `externalCode = B` y `externalCode = C`. 

   > [!div class="mx-imgBorder"]
   > ![Teléfono de Postman](./media/sap-successfactors-inbound-provisioning/postman-phone-type-response.png)

1. Anote estos valores como las constantes que se van a usar con *businessPhoneType* y *cellPhoneType* en la tabla de asignación de atributos.

## <a name="configuring-successfactors-writeback-app"></a>Configuración de la aplicación de escritura diferida de SuccessFactors

En esta sección se proporcionan los pasos necesarios para realizar las siguientes actividades: 

* [Incorporación de la aplicación del conector de aprovisionamiento y configuración de la conectividad con SuccessFactors](#part-1-add-the-provisioning-connector-app-and-configure-connectivity-to-successfactors)
* [Configuración de las asignaciones de atributos](#part-2-configure-attribute-mappings)
* [Habilitar e iniciar el aprovisionamiento de usuarios](#enable-and-launch-user-provisioning)

### <a name="part-1-add-the-provisioning-connector-app-and-configure-connectivity-to-successfactors"></a>Parte 1: Incorporación de la aplicación del conector de aprovisionamiento y configuración de la conectividad con SuccessFactors

**Para configurar SuccessFactors Writeback:**

1. Vaya a <https://portal.azure.com>.

2. En la barra de navegación de la izquierda, seleccione **Azure Active Directory**.

3. Seleccione **Enterprise Applications** (Aplicaciones empresariales) y **All Applications** (Todas las aplicaciones).

4. Seleccione **Add an application** (Agregar una aplicación) y seleccione la categoría **All** (Todos).

5. Busque **SuccessFactors Writeback** y agregue esa aplicación desde la galería.

6. Una vez que se haya agregado la aplicación y se muestre la pantalla de detalles de la aplicación, seleccione **Aprovisionamiento**.

7. Cambie el **Modo de** **aprovisionamiento** a **Automático**.

8. Cumplimente la sección **Credenciales de administrador** del siguiente modo:

   * **Nombre de usuario de administrador**: escriba el nombre de usuario de la cuenta de usuario de SuccessFactors API, con el identificador de empresa anexado. El formato es: **username\@companyID**

   * **Contraseña de administrador:** escriba la contraseña de la cuenta de usuario de SuccessFactors API. 

   * **URL de inquilino:** escriba el nombre del punto de conexión de servicios de la API OData de SuccessFactors. Especifique solo el nombre de host del servidor sin http o https. Este valor debería ser similar al siguiente: `api4.successfactors.com`.

   * **Correo electrónico de notificación**: escriba su dirección de correo electrónico y marque la casilla "Send email if failure occurs" (Enviar una notificación por correo electrónico cuando se produzca un error).
    > [!NOTE]
    > El servicio de aprovisionamiento de Azure AD envía la notificación por correo electrónico si el trabajo de aprovisionamiento entra en un estado de[cuarentena](/azure/active-directory/manage-apps/application-provisioning-quarantine-status).

   * Haga clic en el botón **Probar conexión**. Si la prueba de conexión se lleva a cabo correctamente, haga clic en el botón **Guardar** situado en la parte superior. Si se produce un error, compruebe que las credenciales y la dirección URL de SuccessFactors son válidas.
    >[!div class="mx-imgBorder"]
    >![Azure Portal](./media/sap-successfactors-inbound-provisioning/sfwb-provisioning-creds.png)

   * Una vez que las credenciales se guardan correctamente, en la sección **Asignaciones** se mostrará la asignación predeterminada. Actualice la página si las asignaciones de atributos no son visibles.  

### <a name="part-2-configure-attribute-mappings"></a>Parte 2: configuración de las asignaciones de atributos

En esta sección, configurará cómo fluyen los datos de los usuarios de SuccessFactors a Active Directory.

1. En la pestaña Aprovisionamiento, en **Asignaciones**, haga clic en **Aprovisionar usuarios de Azure Active Directory**.

1. En el campo **Ámbito de objeto de origen** puede seleccionar los conjuntos de usuarios de Azure AD que deben tenerse en cuenta para la escritura diferida; para ello debe definir un conjunto de filtros basados en atributos. El ámbito predeterminado es "todos los usuarios de Azure AD". 
   > [!TIP]
   > Al configurar la aplicación de aprovisionamiento por primera vez, deberá probar y verificar las expresiones y asignaciones de atributos para asegurarse de que ofrece el resultado deseado. Microsoft recomienda usar los filtros de ámbito en **Ámbito de objeto de origen** para probar las asignaciones con algunos usuarios de prueba de Azure AD. Una vez haya verificado que las asignaciones funcionan, puede quitar el filtro o expandirlo gradualmente para incluir más usuarios.

1. El campo **Acciones del objeto de destino** solo admite la operación **Actualizar**.

1. En la tabla de asignación en la sección **Asignaciones de atributos**, puede asignar los siguientes atributos de Azure Active Directory a SuccessFactors. En la tabla siguiente se proporcionan instrucciones sobre cómo asignar los atributos de escritura diferida. 

   | \# | Atributo de Azure AD | Atributo de SuccessFactors | Observaciones |
   |--|--|--|--|
   | 1 | employeeId | personIdExternal | De manera predeterminada, este atributo es el identificador correspondiente. En lugar de employeeId, puede usar cualquier otro atributo de Azure AD que pueda almacenar el valor igual a personIdExternal en SuccessFactors.    |
   | 2 | mail | email | Asigne el origen de atributo de correo electrónico. Con fines de prueba, puede asignar userPrincipalName al correo electrónico. |
   | 3 | 8448 | emailType | Este valor constante es el valor de identificador de SuccessFactors asociado con el correo electrónico empresarial. Actualice este valor para que coincida con el entorno de SuccessFactors. Vea la sección [Recuperación del valor constante para emailType](#retrieve-constant-value-for-emailtype) para conocer los pasos para establecer este valor. |
   | 4 | true | emailIsPrimary | Use este atributo para establecer el correo electrónico empresarial como principal en SuccessFactors. Si correo electrónico empresarial no es el principal, establezca esta marca en false. |
   | 5 | userPrincipalName | [custom01 – custom15] | Si usa **Agregar nueva asignación**, puede escribir userPrincipalName o cualquier atributo de Azure AD en un atributo personalizado disponible en el objeto de usuario de SuccessFactors.  |
   | 6 | on-prem-samAccountName | username | Con **Agregar nueva asignación**, puede asignar de forma opcional samAccountName local en el atributo de nombre de usuario de SuccessFactors. |
   | 7 | SSO | loginMethod | Si el inquilino de SuccessFactors se configura para [SSO parcial](https://apps.support.sap.com/sap/support/knowledge/en/2320766) y, después, usa Agregar nueva asignación, puede establecer loginMethod en un valor constante de "SSO" o "PWD". |
   | 8 | telephoneNumber | businessPhoneNumber | Use esta asignación para realizar el flujo de *telephoneNumber* de Azure AD a número de teléfono del trabajo o de la empresa de SuccessFactors. |
   | 9 | 10605 | businessPhoneType | Este valor constante es el valor de identificador de SuccessFactors asociado con el teléfono del trabajo. Actualice este valor para que coincida con el entorno de SuccessFactors. Vea la sección [Recuperación del valor constante para phoneType](#retrieve-constant-value-for-phonetype) para conocer los pasos para establecer este valor. |
   | 10 | true | businessPhoneIsPrimary | Use este atributo para establecer la marca principal para el número de teléfono del trabajo. Los valores válidos son "true" y "false". |
   | 11 | mobile | cellPhoneNumber | Use esta asignación para realizar el flujo de *telephoneNumber* de Azure AD a número de teléfono del trabajo o de la empresa de SuccessFactors. |
   | 12 | 10606 | cellPhoneType | Este valor constante es el valor de identificador de SuccessFactors asociado con el teléfono móvil. Actualice este valor para que coincida con el entorno de SuccessFactors. Vea la sección [Recuperación del valor constante para phoneType](#retrieve-constant-value-for-phonetype) para conocer los pasos para establecer este valor. |
   | 13 | false | cellPhoneIsPrimary | Use este atributo para establecer la marca principal para el número de teléfono móvil. Los valores válidos son "true" y "false". |
 
1. Valide y revise las asignaciones de atributos. 
 
    >[!div class="mx-imgBorder"]
    >![Asignación de atributos de escritura diferida](./media/sap-successfactors-inbound-provisioning/writeback-attribute-mapping.png)

1. Haga clic en **Guardar** para guardar las asignaciones. A continuación, actualizaremos las expresiones de API de ruta de acceso JSON para usar los códigos de phoneType en la instancia de SuccessFactors. 
1. Seleccione **Mostrar opciones avanzadas**. 

    >[!div class="mx-imgBorder"]
    >![Mostrar opciones avanzadas](./media/sap-successfactors-inbound-provisioning/show-advanced-options.png)

1. Haga clic en **Editar lista de atributos para SuccessFactors**. 

   > [!NOTE] 
   > Si la opción **Editar lista de atributos para SuccessFactors** no se muestra en Azure Portal, use la dirección URL *https://portal.azure.com/?Microsoft_AAD_IAM_forceSchemaEditorEnabled=true* para acceder a la página. 

1. La columna **Expresión de API** de esta vista muestra las expresiones de ruta de acceso JSON que usa el conector. 
1. Actualice las expresiones de ruta de acceso JSON para el teléfono del trabajo y el teléfono móvil para usar el valor de identificador (*businessPhoneType* y *cellPhoneType*) correspondiente a su entorno. 

    >[!div class="mx-imgBorder"]
    >![Cambio de la ruta de acceso JSON del teléfono](./media/sap-successfactors-inbound-provisioning/phone-json-path-change.png)

1. Haga clic en **Guardar** para guardar las asignaciones.

## <a name="enable-and-launch-user-provisioning"></a>Habilitar e iniciar el aprovisionamiento de usuarios

Una vez completadas las configuraciones de la aplicación de aprovisionamiento de SuccessFactors, puede activar el servicio de aprovisionamiento en Azure Portal.

> [!TIP]
> De forma predeterminada, al activar el servicio de aprovisionamiento, se iniciarán las operaciones de aprovisionamiento para todos los usuarios del ámbito. Si hay errores en la asignación o problemas con los datos, puede que se produzcan errores con el trabajo de aprovisionamiento y que entre en estado de cuarentena. Para evitar esto, como procedimiento recomendado, es conveniente configurar el filtro **Ámbito de objeto de origen** y probar las asignaciones de atributos con algunos usuarios de prueba antes de iniciar la sincronización completa de todos los usuarios. Una vez haya verificado que las asignaciones funcionan y que obtiene los resultados deseados, puede quitar el filtro o expandirlo gradualmente para incluir más usuarios.

1. En la pestaña **Aprovisionamiento**, establezca **Estado de aprovisionamiento** en **Activado**.

2. Haga clic en **Save**(Guardar).

3. Esta operación dará comienzo a la sincronización inicial, que puede tardar una cantidad de horas variable, según el número de usuarios que haya en el inquilino de SuccessFactors. Puede consultar en la barra de progreso el seguimiento del progreso del ciclo de sincronización. 

4. En cualquier momento, compruebe la pestaña **Registros de auditoría** en Azure Portal para ver las acciones que ha realizado el servicio de aprovisionamiento. Los registros de auditoría muestran todos los eventos de sincronización individuales realizados por el servicio de aprovisionamiento, por ejemplo, los usuarios que se leen de Employee Central y que luego se agregan o actualizan en Active Directory. 

5. Una vez completada la sincronización inicial, se escribe un informe resumido de auditoría en la pestaña **Aprovisionamiento**, tal y como se muestra a continuación.

   > [!div class="mx-imgBorder"]
   > ![Barra de progreso de aprovisionamiento](./media/sap-successfactors-inbound-provisioning/prov-progress-bar-stats.png)

## <a name="supported-scenarios-known-issues-and-limitations"></a>Escenarios admitidos, problemas conocidos y limitaciones

Consulte la [sección Escenarios de escritura diferida](../app-provisioning/sap-successfactors-integration-reference.md#writeback-scenarios) de la guía de referencia de integración de SAP SuccessFactors. 

## <a name="next-steps"></a>Pasos siguientes

* [Sumérjase en Referencia de la integración de Azure Active Directory y SAP SuccessFactors](../app-provisioning/sap-successfactors-integration-reference.md)
* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md)
* [Aprenda a configurar el inicio de sesión único entre SuccessFactors y Azure Active Directory](successfactors-tutorial.md)
* [Aprenda a integrar otras aplicaciones SaaS con Azure Active Directory](tutorial-list.md)
* [Aprenda a exportar e importar las configuraciones de aprovisionamiento](../app-provisioning/export-import-provisioning-configuration.md)

