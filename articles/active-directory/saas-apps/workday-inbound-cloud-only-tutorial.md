---
title: 'Tutorial: Configuración del aprovisionamiento de entrada de Workday en Azure Active Directory | Microsoft Docs'
description: Aprenda a configurar el aprovisionamiento de entrada de Workday en Azure AD.
services: active-directory
author: cmmdesai
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.topic: article
ms.workload: identity
ms.date: 05/26/2020
ms.author: chmutali
ms.openlocfilehash: 69c3246c910a83d889151d6ad749e1be86340e8c
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88540973"
---
# <a name="tutorial-configure-workday-to-azure-ad-user-provisioning"></a>Tutorial: Configuración de Workday para el aprovisionamiento de usuarios en Azure AD
El objetivo de este tutorial es mostrar los pasos que debe dar para aprovisionar datos de trabajo desde Workday en Active Directory (AD). 

>[!NOTE]
>Use este tutorial si los usuarios que quiere aprovisionar desde Workday son usuarios que solo están en la nube y no necesitan una cuenta de AD local. Si los usuarios necesitan solo una cuenta de AD local o bien una cuenta de AD y de Azure AD, consulte el tutorial sobre [configuración del aprovisionamiento de usuarios de Workday en Active Directory](workday-inbound-tutorial.md). 

## <a name="overview"></a>Información general

El [servicio de aprovisionamiento de usuarios de Azure Active Directory](../app-provisioning/user-provisioning.md) se integra con la [API de recursos humanos de Workday](https://community.workday.com/sites/default/files/file-hosting/productionapi/Human_Resources/v21.1/Get_Workers.html) para poder aprovisionar cuentas de usuario. El flujo de trabajo de aprovisionamiento de usuarios de Workday compatible con el servicio de aprovisionamiento de usuarios de Azure AD permite la automatización de los siguientes escenarios de recursos humanos y de administración del ciclo de vida de identidades:

* **Contratación de nuevos empleados**: Cuando se agrega un nuevo empleado a Workday, se crea automáticamente una cuenta de usuario en Azure Active Directory y, opcionalmente, en Office 365 y en [otras aplicaciones SaaS compatibles con Azure AD](../app-provisioning/user-provisioning.md), con reescritura de la dirección de correo electrónico en Workday.

* **Actualizaciones de atributos y perfiles de empleados**: Si se actualiza un registro de empleado en Workday (por ejemplo, el nombre, el cargo o el jefe), su cuenta de usuario se actualizará automáticamente en Azure Active Directory y, opcionalmente, en Office 365 y en [otras aplicaciones SaaS compatibles con Azure AD](../app-provisioning/user-provisioning.md).

* **Despidos de empleados**: Cuando se prescinde de un empleado en Workday, su cuenta de usuario se deshabilita automáticamente en Azure Active Directory y, opcionalmente, en Office 365 y en [otras aplicaciones SaaS compatibles con Azure AD](../app-provisioning/user-provisioning.md).

* **Recontrataciones de empleados**: Cuando se vuelve a contratar a un empleado en Workday, se puede reactivar o volver a aprovisionar automáticamente su cuenta antigua (en función de las preferencias) en Azure Active Directory y, opcionalmente, en Office 365 y en [otras aplicaciones SaaS compatibles con Azure AD](../app-provisioning/user-provisioning.md).

### <a name="who-is-this-user-provisioning-solution-best-suited-for"></a>¿Para quién es más adecuada esta solución de aprovisionamiento de usuarios?

Esta solución de aprovisionamiento de usuarios de Workday en Azure Active Directory es adecuada para:

* Organizaciones que desean una solución basada en la nube precompilada para el aprovisionamiento de usuarios de Workday

* Organizaciones que requieren aprovisionamiento directo de usuarios de Workday en Azure Active Directory

* Organizaciones que necesitan aprovisionamiento de usuarios mediante los datos obtenidos de Workday

* Organizaciones que usan Office 365 para el correo electrónico

## <a name="solution-architecture"></a>Arquitectura de la solución

En esta sección se describe la arquitectura de la solución de aprovisionamiento de usuarios de un extremo a otro para usuarios que solo están en la nube. Hay dos flujos relacionados:

* **Flujo de datos de RR. HH. autoritativo, de Workday a Azure Active Directory:** En este flujo, los eventos de trabajadores (por ejemplo, nuevas contrataciones, traslados o despidos) se producen en primer lugar en Workday y, luego, los datos de eventos fluyen hacia Azure Active Directory. En función del evento, puede dar lugar a operaciones de creación, actualización, habilitación o deshabilitación en Azure AD.
* **Flujo de escritura diferida: de Active Directory local a Workday:** una vez completada la creación de la cuenta en Active Directory, se sincroniza con Azure AD mediante Azure AD Connect, y la información, como el correo electrónico, el nombre de usuario y el número de teléfono, puede escribirse de forma diferida en Workday.

  ![Información general](./media/workday-inbound-tutorial/workday-cloud-only-provisioning.png)

### <a name="end-to-end-user-data-flow"></a>Flujo de datos de usuarios de un extremo a otro

1. El equipo de RR. HH. realiza las transacciones de trabajadores (empleados que se incorporan a la empresa, se trasladan o la se van, o nuevas contrataciones, transferencias o terminaciones de contratos) en Workday Employee Central
2. El servicio de aprovisionamiento de Azure AD ejecuta las sincronizaciones programadas de las identidades de Workday EC e identifica los cambios que deben procesarse para la sincronización con Active Directory local.
3. El servicio de aprovisionamiento de Azure AD determina el cambio y llama a la operación de creación, actualización, habilitación o deshabilitación del usuario en Azure AD.
4. Si la aplicación de [escritura diferida de Workday](workday-writeback-tutorial.md) está configurada, recupera los atributos, como el correo electrónico, el nombre de usuario y el número de teléfono de Azure AD. 
5. El servicio de aprovisionamiento de Azure AD establece el correo electrónico, el nombre de usuario y el número de teléfono en Workday.

## <a name="planning-your-deployment"></a>Planeamiento de la implementación

La configuración del aprovisionamiento de usuarios controlado por RR. HH. en la nube desde Workday a Azure AD requiere un planeamiento considerable que abarca distintos aspectos, como:

* Determinación del identificador coincidente 
* Asignación de atributos
* Transformación de atributos 
* Filtros de ámbito

Consulte el [plan de implementación de RR. HH. en la nube](../app-provisioning/plan-cloud-hr-provision.md) para instrucciones detalladas sobre estos temas. 

## <a name="configure-integration-system-user-in-workday"></a>Configuración de un usuario de integración en Workday

Consulte la sección [Configuración de un usuario del sistema de integración](workday-inbound-tutorial.md#configure-integration-system-user-in-workday) para crear una cuenta de usuario del sistema de integración de Workday con permisos para recuperar datos de trabajo. 

## <a name="configure-user-provisioning-from-workday-to-azure-ad"></a>Configuración del aprovisionamiento de usuarios de Workday a Azure AD

En las secciones siguientes se describen los pasos para configurar el aprovisionamiento de usuarios desde Workday en Azure AD para implementaciones exclusivas en la nube.

* [Adición de la aplicación del conector de aprovisionamiento de Azure AD y creación de la conexión con Workday](#part-1-adding-the-azure-ad-provisioning-connector-app-and-creating-the-connection-to-workday)
* [Configuración de asignaciones de atributos de Workday y Azure AD](#part-2-configure-workday-and-azure-ad-attribute-mappings)
* [Habilitar e iniciar el aprovisionamiento de usuarios](#enable-and-launch-user-provisioning)

### <a name="part-1-adding-the-azure-ad-provisioning-connector-app-and-creating-the-connection-to-workday"></a>Parte 1: Agregación de la aplicación del conector de aprovisionamiento de Azure AD y creación de la conexión con Workday

**Para configurar el aprovisionamiento de Workday en Azure Active Directory para los usuarios que solo están en la nube:**

1. Ir a <https://portal.azure.com>.

2. En Azure Portal, busque y seleccione **Azure Active Directory**.

3. Seleccione **Enterprise Applications** (Aplicaciones empresariales) y **All Applications** (Todas las aplicaciones).

4. Seleccione **Agregar una aplicación** y, luego, seleccione la categoría **Todos**.

5. Busque **Workday to Azure AD user provisioning** y agregue esa aplicación desde la galería.

6. Una vez que se haya agregado la aplicación y se muestre la pantalla de detalles de la aplicación, seleccione **Aprovisionamiento**.

7. Cambie el **Modo** de **aprovisionamiento** a **Automático**.

8. Cumplimente la sección **Credenciales de administrador** del siguiente modo:

   * **Nombre de usuario de Workday**: escriba el nombre de usuario de la cuenta del sistema de integración de Workday, anexando el nombre de dominio del inquilino. Debe tener un aspecto similar al siguiente: username@contoso4

   * **Contraseña de Workday**: escriba la contraseña de la cuenta del sistema de integración de Workday.

   * **URL de la API Workday Web Services**: Escriba la dirección URL al punto de conexión de Workday Web Services de su inquilino. La dirección URL determina la versión de la API de Workday Web Services utilizada por el conector. 
   
     | Formato de dirección URL | Versión de la API WWS usada | Cambios de XPATH necesarios |
     |------------|----------------------|------------------------|
     | https://####.workday.com/ccx/service/tenantName | v21.1 | No |
     | https://####.workday.com/ccx/service/tenantName/Human_Resources | v21.1 | No |
     | https://####.workday.com/ccx/service/tenantName/Human_Resources/v##.# | v##.# | Sí |

      > [!NOTE]
     > Si no se especifica ninguna información de versión en la dirección URL, la aplicación usa Workday Web Services (WWS) v21.1 y no se requieren cambios en las expresiones de la API XPATH predeterminadas que se incluyen con la aplicación. Para usar una versión específica de la API WWS, especifique el número de versión en la dirección URL. <br>
     > Ejemplo: `https://wd3-impl-services1.workday.com/ccx/service/contoso4/Human_Resources/v34.0` <br>
     > <br> Si usa la API WWS v30.0 o posterior, antes de activar el trabajo de aprovisionamiento, debe actualizar las **expresiones de la API XPATH** en **Asignación de atributos -> Opciones avanzadas -> Editar lista de atributos para Workday** en la sección [Administración de la configuración](workday-inbound-tutorial.md#managing-your-configuration) y [Referencia de atributos de Workday](../app-provisioning/workday-attribute-reference.md#xpath-values-for-workday-web-services-wws-api-v30).  

   * **Correo electrónico de notificación**: escriba su dirección de correo electrónico y marque la casilla "Send email if failure occurs" (Enviar una notificación por correo electrónico cuando se produzca un error).

   * Haga clic en el botón **Probar conexión**.

   * Si la prueba de conexión se lleva a cabo correctamente, haga clic en el botón **Guardar** situado en la parte superior. Si se produce un error, compruebe que la dirección URL y las credenciales de Workday sean válidas en Workday.

### <a name="part-2-configure-workday-and-azure-ad-attribute-mappings"></a>Parte 2: Configuración de asignaciones de atributos de Workday y Azure AD

En esta sección configurará cómo fluyen los datos de los usuarios de Workday a Azure Active Directory para los usuarios que solo están en la nube.

1. En la pestaña Aprovisionamiento, en **Asignaciones**, haga clic en **Synchronize Workers to Azure AD** (Sincronizar trabajadores con Azure AD).

2. En el campo **Ámbito de objeto de origen** puede seleccionar los conjuntos de usuarios de Workday que deben estar en el ámbito para el aprovisionamiento en Azure AD; para ello debe definir un conjunto de filtros basados en atributos. El ámbito predeterminado es "todos los usuarios de Workday". Filtros de ejemplo:

   * Ejemplo: ámbito de los usuarios que tengan los id. de trabajador comprendidos entre 1000000 y 2000000

      * Atributo: WorkerID

      * Operador: REGEX Match

      * Valor: (1[0-9][0-9][0-9][0-9][0-9][0-9])

   * Ejemplo: solo los trabajadores temporales, no los empleados habituales

      * Atributo: ContingentID

      * Operador: IS NOT NULL

3. En el campo **Acciones del objeto de destino**, puede filtrar de forma global qué acciones se realizan en Azure AD. **Crear** y **Actualizar** son las más habituales.

4. En la sección **Asignaciones de atributos** puede definir cómo se asignan los distintos atributos de Workday a los atributos de Active Directory.

5. Haga clic en una asignación de atributos existente para actualizarla o haga clic en **Agregar nueva asignación** en la parte inferior de la pantalla para agregar asignaciones nuevas. Las asignaciones de atributos admiten estas propiedades:

   * **Tipo de asignación**

      * **Directo**: escribe el valor del atributo de Workday en el atributo de AD sin cambios.

      * **Constante**: escribe un valor de cadena estático y constante en el atributo de AD.

      * **Expresión**: le permite escribir un valor personalizado en el atributo de AD, en función de uno o varios atributos de Workday. [Para obtener más información, consulte este artículo sobre las expresiones](../app-provisioning/functions-for-customizing-application-data.md).

   * **Atributo de origen**: el atributo de usuario de Workday. Si el atributo que está buscando no está presente, consulte [Personalización de la lista de atributos de usuario de Workday](workday-inbound-tutorial.md#customizing-the-list-of-workday-user-attributes).

   * **Valor predeterminado**: opcional. Si el atributo de origen tiene un valor vacío, la asignación escribirá este valor.
            La configuración más habitual consiste en dejarlo en blanco.

   * **Atributo de destino**: atributo de usuario de Azure AD.

   * **Hacer coincidir objetos con este atributo**: especifica si se debe usar o no este atributo para identificar de forma unívoca a los usuarios entre Workday y Azure AD. Este valor se suele establecer en el campo Id. del trabajador de Workday, que se suele asignar al atributo de id. de empleado (nuevo) o a un atributo de extensión de Azure AD.

   * **Precedencia de coincidencia**: se pueden establecer varios atributos coincidentes. Si hay varios, se evalúan en el orden definido por este campo. En el momento en que se encuentre una coincidencia, no se evaluarán más atributos coincidentes.

   * **Aplicar esta asignación**

     * **Siempre**: esta asignación se aplica a las acciones de creación y actualización de usuarios

     * **Solo durante la creación**: esta asignación se aplica solo a las acciones de creación de usuarios

6. Para guardar las asignaciones, haga clic en **Guardar** en la parte superior de la sección Asignación de atributos.


## <a name="enable-and-launch-user-provisioning"></a>Habilitar e iniciar el aprovisionamiento de usuarios

Una vez completadas las configuraciones de la aplicación de aprovisionamiento de Workday, puede activar el servicio de aprovisionamiento en Azure Portal.

> [!TIP]
> De forma predeterminada, al activar el servicio de aprovisionamiento, se iniciarán las operaciones de aprovisionamiento para todos los usuarios del ámbito. Si hay errores en la asignación o problemas con los datos de Workday, puede que se produzcan errores con el trabajo de aprovisionamiento y que entre en estado de cuarentena. Para evitar esto, como procedimiento recomendado, es conveniente configurar el filtro **Ámbito de objeto de origen** y probar las asignaciones de atributos con algunos usuarios de prueba antes de iniciar la sincronización completa de todos los usuarios. Una vez haya verificado que las asignaciones funcionan y que obtiene los resultados deseados, puede quitar el filtro o expandirlo gradualmente para incluir más usuarios.

1. En la pestaña **Aprovisionamiento**, establezca **Estado de aprovisionamiento** en **Activado**.

2. Haga clic en **Save**(Guardar).

3. Esta operación iniciará la sincronización inicial, que puede tardar una cantidad de horas variable, según el número de usuarios que haya en el inquilino de Workday. Puede consultar en la barra de progreso el seguimiento del progreso del ciclo de sincronización. 

4. En cualquier momento, compruebe la pestaña **Registros de auditoría** en Azure Portal para ver las acciones que ha realizado el servicio de aprovisionamiento. Los registros de auditoría muestran todos los eventos de sincronización individuales realizados por el servicio de aprovisionamiento, por ejemplo, los usuarios que se leen fuera de Workday y que luego se agregan o actualizan en Azure Active Directory. 

5. Una vez completada la sincronización inicial, se escribe un informe resumido de auditoría en la pestaña **Aprovisionamiento**, tal y como se muestra a continuación.

   > [!div class="mx-imgBorder"]
   > ![Barra de progreso de aprovisionamiento](./media/sap-successfactors-inbound-provisioning/prov-progress-bar-stats.png)

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda sobre los atributos de Workday compatibles para el aprovisionamiento de entrada](../app-provisioning/workday-attribute-reference.md)
* [Aprenda a configurar la escritura diferida de Workday](workday-writeback-tutorial.md)
* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md)
* [Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Workday](workday-tutorial.md)
* [Aprenda a integrar otras aplicaciones SaaS con Azure Active Directory](tutorial-list.md)
* [Aprenda a exportar e importar las configuraciones de aprovisionamiento](../app-provisioning/export-import-provisioning-configuration.md)


