---
title: Referencia de la integración de Azure Active Directory y SAP SuccessFactors
description: Información técnica detallada sobre el aprovisionamiento controlado por recursos humanos de SAP SuccessFactors
services: active-directory
author: cmmdesai
manager: celestedg
ms.service: active-directory
ms.subservice: app-provisioning
ms.topic: reference
ms.workload: identity
ms.date: 07/20/2020
ms.author: chmutali
ms.openlocfilehash: ea47f8a6fc29571a27f8976bd0ad9bbd30ed0ad9
ms.sourcegitcommit: 85eb6e79599a78573db2082fe6f3beee497ad316
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87808463"
---
# <a name="how-azure-active-directory-provisioning-integrates-with-sap-successfactors"></a>Integración del aprovisionamiento de Azure Active Directory con SAP SuccessFactors 

El [servicio de aprovisionamiento de usuarios de Azure Active Directory](../app-provisioning/user-provisioning.md) se integra con [SAP SuccessFactors Employee Central](https://www.successfactors.com/products-services/core-hr-payroll/employee-central.html) para administrar el ciclo de vida de identidad de los usuarios. Azure Active Directory ofrece tres integraciones precompiladas: 

* [Aprovisionamiento de usuarios de SuccessFactors a Active Directory en el entorno local](../saas-apps/sap-successfactors-inbound-provisioning-tutorial.md)
* [Aprovisionamiento de usuarios de SuccessFactors a Azure Active Directory](../saas-apps/sap-successfactors-inbound-provisioning-cloud-only-tutorial.md)
* [Escritura diferida de SuccessFactors](../saas-apps/sap-successfactors-writeback-tutorial.md)

En este artículo se explica cómo funciona la integración y cómo puede personalizar el comportamiento de aprovisionamiento para diferentes escenarios de recursos humanos. 

## <a name="establishing-connectivity"></a>Establecimiento de la conectividad 
El servicio de aprovisionamiento de Azure AD usa la autenticación básica para conectarse a los puntos de conexión de la API de OData de Employee Central. Al configurar la aplicación de aprovisionamiento de SuccessFactors, use el parámetro *URL de inquilino* de la sección *Credenciales de administrador* para configurar la [URL del centro de datos de la API](https://apps.support.sap.com/sap/support/knowledge/en/2215682). 

Para proteger mejor la conectividad entre el servicio de aprovisionamiento de Azure AD y SuccessFactors, puede agregar los intervalos IP de Azure AD a la lista de IP permitidas de SuccessFactors; para ello, siga los pasos que se describen a continuación:

1. Descargue los [intervalos IP más recientes](https://www.microsoft.com/download/details.aspx?id=56519) para la nube pública de Azure. 
1. Abra el archivo y busque la etiqueta **AzureActiveDirectory**. 

   >[!div class="mx-imgBorder"] 
   >![Intervalo IP de Azure AD](media/sap-successfactors-integration-reference/azure-active-directory-ip-range.png)

1. Copie todos los intervalos de direcciones IP que aparecen en el elemento *addressPrefixes* y use el intervalo para crear una lista de restricciones de direcciones IP.
1. Traduzca los valores CIDR a intervalos IP.  
1. Inicie sesión en el portal de administración de SuccessFactors para agregar los intervalos IP a la lista de permitidos. Consulte la [nota de soporte técnico de SAP 2253200](https://apps.support.sap.com/sap/support/knowledge/en/2253200). Ahora puede [especificar los intervalos IP](https://answers.sap.com/questions/12882263/whitelisting-sap-cloud-platform-ip-address-range-i.html) en esta herramienta. 

## <a name="supported-entities"></a>Entidades admitidas
El servicio de aprovisionamiento de Azure AD recupera las siguientes entidades para cada usuario de SuccessFactors. Cada entidad se expande mediante el parámetro de consulta *$expand* de la API de OData. Consulte la columna *Regla de recuperación* siguiente. Algunas entidades se expanden de forma predeterminada, mientras que otras se expanden solo si un atributo concreto está presente en la asignación. 

| \# | Entidad de SuccessFactors                  | Nodo de OData     | Regla de recuperación |
|----|----------------------------------------|------------------------------|------------------|
| 1  | PerPerson                              | *nodo raíz*                  | Siempre           |
| 2  | PerPersonal                            | personalInfoNav              | Siempre           |
| 3  | PerPhone                               | phoneNav                     | Siempre           |
| 4  | PerEmail                               | emailNav                     | Siempre           |
| 5  | EmpEmployment                          | employmentNav                | Siempre           |
| 6  | Usuario                                   | employmentNav/userNav        | Siempre           |
| 7  | EmpJob                                 | employmentNav/jobInfoNav     | Siempre           |
| 8  | EmpEmploymentTermination               | activeEmploymentsCount       | Siempre           |
| 9  | FOCompany                              | employmentNav/jobInfoNav/companyNav | Solo si está asignado el atributo `company` o `companyId` |
| 10 | FODepartment                           | employmentNav/jobInfoNav/companyNav | Solo si está asignado el atributo `department` o `departmentId` |
| 11 | FOBusinessUnit                         | employmentNav/jobInfoNav/businessUnitNav | Solo si está asignado el atributo `businessUnit` o `businessUnitId` |
| 12 | FOCostCenter                           | employmentNav/jobInfoNav/costCenterNav | Solo si está asignado el atributo `costCenter` o `costCenterId` |
| 13 | FODivision                             | employmentNav/jobInfoNav/divisionNav  | Solo si está asignado el atributo `division` o `divisionId` |
| 14 | FOJobCode                              | employmentNav/jobInfoNav/jobCodeNav  | Solo si está asignado el atributo `jobCode` o `jobCodeId` |
| 15 | FOPayGrade                             | employmentNav/jobInfoNav/payGradeNav  | Solo si está asignado el atributo `payGrade` |
| 16 | FOLocation                             | employmentNav/jobInfoNav/locationNav  | Solo si está asignado el atributo `location` |
| 17 | FOCorporateAddressDEFLT                | employmentNav/jobInfoNav/addressNavDEFLT  | Si la asignación contiene uno de los siguientes atributos: `officeLocationAddress,  officeLocationCity, officeLocationZipCode` |
| 18 | FOEventReason                          | employmentNav/jobInfoNav/eventReasonNav  | Solo si está asignado el atributo `eventReason` |
| 19 | EmpGlobalAssignment                    | employmentNav/empGlobalAssignmentNav | Solo si está asignado `assignmentType` |
| 20 | Lista desplegable EmploymentType                | employmentNav/jobInfoNav/employmentTypeNav | Solo si está asignado `employmentType` |
| 21 | Lista desplegable EmployeeClass                 | employmentNav/jobInfoNav/employeeClassNav | Solo si está asignado `employeeClass` |
| 22 | Lista desplegable EmplStatus                    | employmentNav/jobInfoNav/emplStatusNav | Solo si está asignado `emplStatus` |
| 23 | Lista desplegable AssignmentType                | employmentNav/empGlobalAssignmentNav/assignmentTypeNav | Solo si está asignado `assignmentType` |

## <a name="how-full-sync-works"></a>Cómo funciona la sincronización completa
En función de la asignación de atributos, durante la sincronización completa el servicio de aprovisionamiento de Azure AD envía la siguiente consulta "GET" de API de OData para capturar datos efectivos de todos los usuarios activos. 

> [!div class="mx-tdCol2BreakAll"]
>| Parámetro | Descripción |
>| ----------|-------------|
>| Host de la API de OData | Anexa https a la *URL del inquilino*. Ejemplo: `https://api4.successfactors.com` |
>| Punto de conexión de la API de OData | `/odata/v2/PerPerson` |
>| Parámetro de consulta $format de OData | `json` |
>| Parámetro de consulta $filter de OData | `(personEmpTerminationInfoNav/activeEmploymentsCount ge 1) and (lastModifiedDateTime le <CurrentExecutionTime>)` |
>| Parámetro de consulta $expand de OData | Este valor de parámetro depende de los atributos asignados. Ejemplo: `employmentNav/userNav,employmentNav/jobInfoNav,personalInfoNav,personEmpTerminationInfoNav,phoneNav,emailNav,employmentNav/jobInfoNav/companyNav/countryOfRegistrationNav,employmentNav/jobInfoNav/divisionNav,employmentNav/jobInfoNav/departmentNav` |
>| Parámetro de consulta customPageSize de OData | `100` |

> [!NOTE]
> Durante la primera sincronización completa inicial, el servicio de aprovisionamiento de Azure AD no extrae datos de trabajo inactivos o terminados.

Para cada usuario de SuccessFactors, el servicio de aprovisionamiento busca una cuenta en el destino (Azure AD o Active Directory en el entorno local) mediante el atributo coincidente que se definió en la asignación. Por ejemplo: si *personIdExternal* se asigna a *employeeId* y está configurado como atributo coincidente, el servicio de aprovisionamiento usa el valor *personIdExternal* para buscar el usuario con el filtro *employeeId*. Si se encuentra una coincidencia de usuario, se actualizan los atributos del destino. Si no se encuentra ninguna coincidencia, se crea una nueva entrada en el destino. 

Para validar los datos que devuelve el punto de conexión de la API de OData para un `personIdExternal` específico, actualice el `SuccessFactorsAPIEndpoint` en la consulta de API siguiente con la URL del servidor del centro de datos de la API y use una herramienta como [Postman](https://www.postman.com/downloads/) para invocar la consulta. 

```
https://[SuccessFactorsAPIEndpoint]/odata/v2/PerPerson?$format=json&
$filter=(personIdExternal in '[personIdExternalValue]')&
$expand=employmentNav/userNav,employmentNav/jobInfoNav,personalInfoNav,personEmpTerminationInfoNav,
phoneNav,phoneNav/phoneTypeNav,emailNav,employmentNav/jobInfoNav/businessUnitNav,employmentNav/jobInfoNav/companyNav,
employmentNav/jobInfoNav/companyNav/countryOfRegistrationNav,employmentNav/jobInfoNav/costCenterNav,
employmentNav/jobInfoNav/divisionNav,employmentNav/jobInfoNav/departmentNav,employmentNav/jobInfoNav/jobCodeNav,
employmentNav/jobInfoNav/locationNav,employmentNav/jobInfoNav/locationNav/addressNavDEFLT,employmentNav/jobInfoNav/payGradeNav,
employmentNav/empGlobalAssignmentNav,employmentNav/empGlobalAssignmentNav/assignmentTypeNav,employmentNav/jobInfoNav/emplStatusNav,
employmentNav/jobInfoNav/employmentTypeNav,employmentNav/jobInfoNav/employeeClassNav,employmentNav/jobInfoNav/eventReasonNav
```

## <a name="how-incremental-sync-works"></a>Cómo funciona la sincronización incremental

Después de una sincronización completa, el servicio de aprovisionamiento de Azure AD mantiene `LastExecutionTimestamp` y lo usa para crear consultas diferenciales para recuperar los cambios incrementales. Los atributos de marca de tiempo presentes en cada entidad de SuccessFactors, como `lastModifiedDateTime`, `startDate`, `endDate` y `latestTerminationDate`, se evalúan para comprobar si el cambio se encuentra entre `LastExecutionTimestamp` y `CurrentExecutionTime`. En caso afirmativo, se considera que el cambio de entrada es efectivo y se procesa para la sincronización. 

## <a name="reading-attribute-data"></a>Lectura de datos de atributo

Cuando el servicio de aprovisionamiento de Azure AD envía una consulta a SuccessFactors, recupera un conjunto de resultados JSON. El conjunto de resultados JSON incluye varios atributos almacenados en Employee Central. De forma predeterminada, el esquema de aprovisionamiento está configurado para recuperar solo un subconjunto de dichos atributos. 

Para recuperar atributos adicionales, siga los pasos que se indican a continuación:
    
1. Vaya a **Aplicaciones empresariales** -> **Aplicación SuccessFactors** -> **Aprovisionamiento** -> **Edit Provisioning** (Editar aprovisionamiento) -> **página asignación de atributos**.
1. Desplácese hacia abajo y haga clic en **Mostrar opciones avanzadas**.
1. Haga clic en **Editar lista de atributos para SuccessFactors**. 

   > [!NOTE] 
   > Si la opción **Editar lista de atributos para SuccessFactors** no se muestra en Azure Portal, use la dirección URL *https://portal.azure.com/?Microsoft_AAD_IAM_forceSchemaEditorEnabled=true* para acceder a la página. 

1. La columna **Expresión de API** de esta vista muestra las expresiones JSONPath que usa el conector.

   >[!div class="mx-imgBorder"] 
   >![Expresión de API](media/sap-successfactors-integration-reference/jsonpath-api-expressions.png#lightbox)  

1. Puede editar un valor de JSONPath existente o agregar un nuevo atributo con una expresión JSONPath válida al esquema. 

En la sección siguiente se proporciona una lista de los escenarios comunes de edición de los valores JSONPath. 

## <a name="handling-different-hr-scenarios"></a>Administración de distintos escenarios de recursos humanos

JSONPath es un lenguaje de consulta para JSON similar a XPath para XML. Al igual que XPath, JSONPath permite la extracción y el filtrado de datos a partir de una carga útil JSON.

Mediante la transformación JSONPath, puede personalizar el comportamiento de la aplicación de aprovisionamiento de Azure AD para recuperar atributos personalizados y administrar escenarios como la recontratación, la conversión de trabajos y las asignaciones globales. 

En esta sección se explica cómo puede personalizar la aplicación de aprovisionamiento para los siguientes escenarios de recursos humanos: 
* [Recuperación de atributos adicionales](#retrieving-additional-attributes)
* [Recuperando de atributos adicionales](#retrieving-custom-attributes)
* [Administración del escenario de conversión de trabajo](#handling-worker-conversion-scenario)
* [Administración del escenario de recontratación](#handling-rehire-scenario)
* [Administración del escenario de asignación global](#handling-global-assignment-scenario)
* [Administración del escenario de trabajos simultáneos](#handling-concurrent-jobs-scenario)

### <a name="retrieving-additional-attributes"></a>Recuperación de atributos adicionales

El esquema predeterminado de la aplicación de aprovisionamiento de SuccessFactors para Azure AD incluye [más de 90 atributos predefinidos](sap-successfactors-attribute-reference.md). Para agregar más atributos de SuccessFactors al esquema de aprovisionamiento, siga los pasos que se indican a continuación: 

1. Use la siguiente consulta de OData para recuperar datos de un usuario de prueba válido de Employee Central. 

   ```
    https://[SuccessFactorsAPIEndpoint]/odata/v2/PerPerson?$format=json&
    $filter=(personIdExternal in '[personIdExternalValue]')&
    $expand=employmentNav/userNav,employmentNav/jobInfoNav,personalInfoNav,personEmpTerminationInfoNav,
    phoneNav,phoneNav/phoneTypeNav,emailNav,employmentNav/jobInfoNav/businessUnitNav,employmentNav/jobInfoNav/companyNav,
    employmentNav/jobInfoNav/companyNav/countryOfRegistrationNav,employmentNav/jobInfoNav/costCenterNav,
    employmentNav/jobInfoNav/divisionNav,employmentNav/jobInfoNav/departmentNav,employmentNav/jobInfoNav/jobCodeNav,
    employmentNav/jobInfoNav/locationNav,employmentNav/jobInfoNav/locationNav/addressNavDEFLT,employmentNav/jobInfoNav/payGradeNav,
    employmentNav/empGlobalAssignmentNav,employmentNav/empGlobalAssignmentNav/assignmentTypeNav,employmentNav/jobInfoNav/emplStatusNav,
    employmentNav/jobInfoNav/employmentTypeNav,employmentNav/jobInfoNav/employeeClassNav,employmentNav/jobInfoNav/eventReasonNav
   ```

1. Determinación de la entidad de Employee Central asociada con el atributo
   * Si el atributo forma parte de la entidad *EmpEmployment*, búsquelo debajo del nodo *employmentNav*. 
   * Si el atributo forma parte de la entidad *User*, búsquelo debajo del nodo *employmentNav/userNav*.
   * Si el atributo forma parte de la entidad *EmpJob*, búsquelo debajo del nodo *employmentNav/jobInfoNav*. 
1. Construya la ruta de acceso JSON asociada con el atributo y agregue este nuevo atributo a la lista de atributos de SuccessFactors. 
   * Ejemplo 1: Supongamos que quiere agregar el atributo *okToRehire*, que forma parte de la entidad *employmentNav*, para luego usar el elemento JSONPath `$.employmentNav.results[0].okToRehire`
   * Ejemplo 2: Supongamos que quiere agregar el atributo *timeZone*, que forma parte de la entidad *userNav*, para luego usar el elemento JSONPath `$.employmentNav.results[0].userNav.timeZone`
   * Ejemplo 3: Supongamos que quiere agregar el atributo *flsaStatus*, que forma parte de la entidad *jobInfoNav*, para luego usar el elemento JSONPath `$.employmentNav.results[0].jobInfoNav.results[0].flsaStatus`
1. Guarde el esquema. 
1. Reinicie el aprovisionamiento.

### <a name="retrieving-custom-attributes"></a>Recuperando de atributos adicionales

De forma predeterminada, los siguientes atributos personalizados están predefinidos en la aplicación de aprovisionamiento de SuccessFactors para Azure AD: 
* *custom01-custom15* de la entidad User (userNav)
* *customString1-customString15* de la entidad EmpEmployment (employmentNav) denominados *empNavCustomString1-empNavCustomString15*
* *customString1-customString15* de la entidad EmpJobInfo (jobInfoNav) denominada *mpJobNavCustomString1-empNavJobCustomString15*

Supongamos que, en la instancia de Employee Central, el atributo *customString35* en *EmpJobInfo* almacena la descripción de la ubicación. Quiere transferir este valor al atributo *physicalDeliveryOfficeName* de Active Directory. Para configurar la asignación de atributos de este escenario, siga los pasos que se indican a continuación: 

1. Edite la lista de atributos de SuccessFactors para agregar un nuevo atributo llamado *empJobNavCustomString35*.
1. Configure la expresión de la API JSONPath para este atributo como: `$.employmentNav.results[0].jobInfoNav.results[0].customString35`
1. Guarde y vuelva a cargar el cambio de asignación en Azure Portal.  
1. En la hoja asignación de atributos, asigne *empJobNavCustomString35* a *physicalDeliveryOfficeName*.
1. Guarde la asignación.

Ampliación de este escenario: 
* Si quiere asignar el atributo *custom35* de la entidad *User*, use el JSONPath `$.employmentNav.results[0].userNav.custom35`.
* Si quiere asignar el atributo *customString35* de la entidad *EmpEmployment*, use el JSONPath `$.employmentNav.results[0].customString35`.

### <a name="handling-worker-conversion-scenario"></a>Administración del escenario de conversión de trabajo

La conversión de trabajos es el proceso para convertir un empleado a tiempo completo existente en un contratista o viceversa. En este escenario, Employee Central agregará una nueva entidad *EmpEmployment* junto con una nueva entidad *User* para la misma entidad *Person*. La entidad *User* anidada en la entidad *EmpEmployment* anterior está establecida en NULL. Para administrar este escenario de modo que los nuevos datos de empleo se muestren cuando se realice una conversión, puede actualizar de forma masiva el esquema de la aplicación de aprovisionamiento mediante los pasos que se indican a continuación:  

1. Abra la hoja de asignación de atributos de la aplicación de aprovisionamiento de SuccessFactors. 
1. Desplácese hacia abajo y haga clic en **Mostrar opciones avanzadas**.
1. Haga clic en el vínculo **Revise el esquema aquí** para abrir el editor de esquemas. 

   >![revisar el esquema](media/sap-successfactors-integration-reference/review-schema.png#lightbox)

1. Haga clic en el vínculo **Descargar** para guardar una copia del esquema antes de editarlo. 

   >![descargar el esquema](media/sap-successfactors-integration-reference/download-schema.png#lightbox)
1. En el editor de esquemas, presione las teclas Ctrl+H para abrir el control para buscar y reemplazar.
1. En el cuadro de texto de búsqueda, copie y pegue el valor. `$.employmentNav.results[0]`.
1. En el cuadro de texto de reemplazo, copie y pegue el valor. `$.employmentNav.results[?(@.userNav != null)]`. Tome nota del espacio en blanco que rodea al operador `!=`, es importante para el procesamiento correcto de la expresión JSONPath. 
   >![buscar y reemplazar para conversión](media/sap-successfactors-integration-reference/find-replace-conversion-scenario.png#lightbox)
1. Haga clic en la opción "reemplazar todo" para actualizar el esquema. 
1. Guarde el esquema. 
1. El proceso anterior actualiza todas las expresiones JSONPath como se indica a continuación: 
   * JSONPath anterior: `$.employmentNav.results[0].jobInfoNav.results[0].departmentNav.name_localized`
   * JSONPath nuevo: `$.employmentNav.results[?(@.userNav != null)].jobInfoNav.results[0].departmentNav.name_localized`
1. Reinicio del aprovisionamiento. 

### <a name="handling-rehire-scenario"></a>Administración del escenario de recontratación

Normalmente hay dos opciones para procesar las recontrataciones: 
* Opción 1: Crear un nuevo perfil de persona en Employee Central
* Opción 2: Reutilizar un perfil de persona existente en Employee Central

Si el proceso de recursos humanos usa la opción 1, no es necesario realizar ningún cambio en el esquema de aprovisionamiento. Si el proceso de recursos humanos usa la opción 2, Employee Central agregará una nueva entidad de *EmpEmployment* junto con una nueva entidad *User* para la misma entidad *Person*. A diferencia del escenario de conversión, la entidad *User* de la entidad *EmpEmployment* anterior no está establecida en NULL. 

Para administrar este escenario de recontratación (opción 2) de modo que los nuevos datos de empleo se muestren para los perfiles de recontratación, puede actualizar de forma masiva el esquema de la aplicación de aprovisionamiento mediante los pasos que se indican a continuación:  

1. Abra la hoja de asignación de atributos de la aplicación de aprovisionamiento de SuccessFactors. 
1. Desplácese hacia abajo y haga clic en **Mostrar opciones avanzadas**.
1. Haga clic en el vínculo **Revise el esquema aquí** para abrir el editor de esquemas.   
1. Haga clic en el vínculo **Descargar** para guardar una copia del esquema antes de editarlo.   
1. En el editor de esquemas, presione las teclas Ctrl+H para abrir el control para buscar y reemplazar.
1. En el cuadro de texto de búsqueda, copie y pegue el valor. `$.employmentNav.results[0]`.
1. En el cuadro de texto de reemplazo, copie y pegue el valor. `$.employmentNav.results[-1:]`. Esta expresión JSONPath devuelve el último registro de *EmpEmployment*.   
1. Haga clic en la opción "reemplazar todo" para actualizar el esquema. 
1. Guarde el esquema. 
1. El proceso anterior actualiza todas las expresiones JSONPath como se indica a continuación: 
   * JSONPath anterior: `$.employmentNav.results[0].jobInfoNav.results[0].departmentNav.name_localized`
   * JSONPath nuevo: `$.employmentNav.results[-1:].jobInfoNav.results[0].departmentNav.name_localized`
1. Reinicie el aprovisionamiento. 

Este cambio de esquema también es compatible con el escenario de conversión de trabajo. 

### <a name="handling-global-assignment-scenario"></a>Administración del escenario de asignación global

Cuando un usuario de Employee Central se procesa para una asignación global, SuccessFactors agrega una nueva entidad *EmpEmployment* y establece *assignmentClass* en "GA". También crea una nueva entidad *User*. Por lo tanto, el usuario ahora tiene:
* Una entidad *EmpEmployment* + *User* que corresponde a la asignación de inicio con *assignmentClass* establecida en "ST", y 
* Otra entidad *EmpEmployment* + *User* que corresponde a la asignación global con *assignmentClass* establecida en "GA".

Para capturar atributos que pertenecen al perfil de usuario de la asignación estándar y la asignación global, siga los pasos que se indican a continuación: 

1. Abra la hoja de asignación de atributos de la aplicación de aprovisionamiento de SuccessFactors. 
1. Desplácese hacia abajo y haga clic en **Mostrar opciones avanzadas**.
1. Haga clic en el vínculo **Revise el esquema aquí** para abrir el editor de esquemas.   
1. Haga clic en el vínculo **Descargar** para guardar una copia del esquema antes de editarlo.   
1. En el editor de esquemas, presione las teclas Ctrl+H para abrir el control para buscar y reemplazar.
1. En el cuadro de texto de búsqueda, copie y pegue el valor. `$.employmentNav.results[0]`.
1. En el cuadro de texto de reemplazo, copie y pegue el valor. `$.employmentNav.results[?(@.assignmentClass == 'ST')]`. 
1. Haga clic en la opción "reemplazar todo" para actualizar el esquema. 
1. Guarde el esquema. 
1. El proceso anterior actualiza todas las expresiones JSONPath como se indica a continuación: 
   * JSONPath anterior: `$.employmentNav.results[0].jobInfoNav.results[0].departmentNav.name_localized`
   * JSONPath nuevo: `$.employmentNav.results[?(@.assignmentClass == 'ST')].jobInfoNav.results[0].departmentNav.name_localized`
1. Vuelva a cargar la hoja de asignación de atributos de la aplicación. 
1. Desplácese hacia abajo y haga clic en **Mostrar opciones avanzadas**.
1. Haga clic en **Editar lista de atributos para SuccessFactors**.
1. Agregue nuevos atributos para capturar datos de la asignación global. Por ejemplo: si quiere capturar el nombre de departamento asociado a un perfil de asignación global, puede agregar el atributo *globalAssignmentDepartment* con la expresión JSONPath establecida en `$.employmentNav.results[?(@.assignmentClass == 'GA')].jobInfoNav.results[0].departmentNav.name_localized`. 
1. Ahora puede dirigir ambos valores de departamento a los atributos de Active Directory o direccionar de forma selectiva un valor mediante la asignación de expresiones. Ejemplo: la expresión siguiente establece el valor del atributo *department* de AD en *globalAssignmentDepartment*, si está presente; de lo contrario, establece el valor según el atributo *department* asociado a la asignación estándar. 
   * `IIF(IsPresent([globalAssignmentDepartment]),[globalAssignmentDepartment],[department])`

1. Guarde la asignación. 
1. Reinicie el aprovisionamiento. 

### <a name="handling-concurrent-jobs-scenario"></a>Administración del escenario de trabajos simultáneos

Cuando un usuario de Employee Central tiene varios trabajos simultáneos, hay dos entidades *EmpEmployment* y *User* con *assignmentClass* establecido en "ST". Para recuperar los atributos que pertenezcan a ambos trabajos, siga los pasos que se indican a continuación: 

1. Abra la hoja de asignación de atributos de la aplicación de aprovisionamiento de SuccessFactors. 
1. Desplácese hacia abajo y haga clic en **Mostrar opciones avanzadas**.
1. Haga clic en **Editar lista de atributos para SuccessFactors**.
1. Supongamos que quiere extraer el departamento asociado con el trabajo 1 y el trabajo 2. El atributo predefinido *department* ya captura el valor de department para el primer trabajo. Puede definir un nuevo atributo llamado *secondJobDepartment* y establecer la expresión JSONPath en `$.employmentNav.results[1].jobInfoNav.results[0].departmentNav.name_localized`
1. Ahora puede dirigir ambos valores de departamento a los atributos de Active Directory o direccionar de forma selectiva un valor mediante la asignación de expresiones. 
1. Guarde la asignación. 
1. Reinicie el aprovisionamiento. 

## <a name="writeback-scenarios"></a>Escenarios de escritura diferida

En esta sección se tratan diferentes escenarios de escritura diferida. Se recomiendan enfoques de configuración en función de cómo se configure el número de teléfono y el correo electrónico en SuccessFactors.

### <a name="supported-scenarios-for-phone-and-email-write-back"></a>Escenarios admitidos para la escritura diferida del correo electrónico y el teléfono 

| \# | Requisito del escenario | Valor de marca principal <br> de correo electrónico | Teléfono del trabajo <br> de teléfono del trabajo | Valor de marca principal <br> de teléfono móvil | Teléfono del trabajo <br> mapping | Asignación <br> mapping |
|--|--|--|--|--|--|--|
| 1 | * Establezca solo el correo empresarial como principal. <br> * No establezca números de teléfono. | true | true | false | \[Sin establecer\] | \[Sin establecer\] | 
| 2 | * En SuccessFactors, el correo empresarial y el teléfono del trabajo son el valor principal. <br> * Dirija siempre el número de teléfono de Azure AD al teléfono del trabajo y el móvil al teléfono móvil. | true | true | false | telephoneNumber | mobile | 
| 3 | * En SuccessFactors, el correo empresarial y el teléfono móvil son el valor principal. <br> * Dirija siempre el número de teléfono de Azure AD al teléfono del trabajo, y el móvil al teléfono móvil. | true | false | true |  telephoneNumber | mobile | 
| 4 | * En SuccessFactors, el correo empresarial es el principal. <br> * En Azure AD, compruebe si está presente el número de teléfono del trabajo; en caso afirmativo, compruebe si el número de teléfono móvil también está presente; marque el número de teléfono del trabajo como principal solo si no está presente el número de móvil. | true | Use la asignación de expresión: `IIF(IsPresent([telephoneNumber]), IIF(IsPresent([mobile]),"false", "true"), "false")` | Use la asignación de expresión: `IIF(IsPresent([mobile]),"false", "true")` | telephoneNumber | mobile | 
| 5 | * En SuccessFactors, el correo empresarial y el teléfono del trabajo son el principal. <br> * En Azure AD, si está disponible el móvil, establézcalo como teléfono del trabajo; de lo contrario, use telephoneNumber. | true | true | false | `IIF(IsPresent([mobile]), [mobile], [telephoneNumber])` | \[Sin establecer\] | 

* Si no hay ninguna asignación para el número de teléfono en la asignación de atributos de escritura diferida, solo se incluye el correo electrónico en la escritura diferida.
* Durante la incorporación de nuevos empleados en Employee Central, el correo empresarial y el número de teléfono del trabajo podría no estar disponibles. Si es obligatorio que establezca el correo empresarial y el teléfono del trabajo como principales durante la incorporación, puede establecer un valor ficticio para el teléfono y el correo electrónico del trabajo durante la creación de una nueva contratación, que posteriormente se actualizará con la aplicación de escritura diferida.
 
### <a name="unsupported-scenarios-for-phone-and-email-write-back"></a>Escenarios no admitidos para la escritura diferida del correo electrónico y el teléfono

* En Employee central, durante la incorporación, el correo electrónico personal y el teléfono personal se establecen como valores principales. La aplicación de escritura diferida no puede cambiar esta configuración y establecer el correo empresarial y el teléfono del trabajo como principal.
* En Employee Central, el teléfono del trabajo se establece como principal. La aplicación de escritura diferida no puede cambiarlo y establecer el teléfono móvil como principal.
* La aplicación de escritura diferida no puede leer la configuración de la marca principal actual y usar esos mismos valores para la operación de escritura. Siempre se usarán los valores de marca configurados en la asignación de atributos. 

## <a name="next-steps"></a>Pasos siguientes

* [Más información sobre configurar el aprovisionamiento de SuccessFactors a Active Directory](../saas-apps/sap-successfactors-inbound-provisioning-tutorial.md)
* [Más información sobre configurar la escritura diferida en SuccessFactors](../saas-apps/sap-successfactors-writeback-tutorial.md)
* [Más información sobre los atributos de SuccessFactors compatibles para el aprovisionamiento de entrada](sap-successfactors-attribute-reference.md)










