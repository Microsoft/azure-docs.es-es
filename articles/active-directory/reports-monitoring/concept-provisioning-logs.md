---
title: Registros de aprovisionamiento en el portal de Azure Active Directory (versión preliminar) | Microsoft Docs
description: Introducción a los informes de actividad de aprovisionamiento en el portal de Azure Active Directory
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.assetid: 4b18127b-d1d0-4bdc-8f9c-6a4c991c5f75
ms.service: active-directory
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 09/01/2020
ms.author: markvi
ms.reviewer: arvinh
ms.collection: M365-identity-device-management
ms.openlocfilehash: 16b2ab39e9bcd6dff44387edc60be9bfc649f224
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89229878"
---
# <a name="provisioning-reports-in-the-azure-active-directory-portal-preview"></a>Informes de aprovisionamiento en el portal de Azure Active Directory (versión preliminar)

La arquitectura de los informes de Azure Active Directory (Azure AD) consta de los siguientes componentes:

- **Actividad** 
    - **Inicios de sesión**: información sobre el uso de aplicaciones administradas y actividades de inicio de sesión de usuario.
    - **Registros de auditoría**: los [registros de auditoría](concept-audit-logs.md) proporcionan información de la actividad del sistema sobre la administración de usuarios y grupos, aplicaciones administradas y actividades de directorio. - 
    - **Registros de aprovisionamiento**: proporcionan la actividad del sistema sobre el usuario, los grupos y los roles aprovisionados por el servicio de aprovisionamiento de Azure AD. 

- **Seguridad** 
    - **Inicios de sesión de riesgo**: un [inicio de sesión de riesgo](../identity-protection/overview-identity-protection.md) es un indicador de un intento de inicio de sesión que puede haber realizado alguien que no es el propietario legítimo de una cuenta de usuario.
    - **Usuarios marcados en riesgo**: un [usuario en riesgo](../identity-protection/overview-identity-protection.md) es un indicador de una cuenta de usuario que puede haber estado en peligro.

Este tema ofrece una visión general del informe de aprovisionamiento.

## <a name="prerequisites"></a>Prerrequisitos

### <a name="who-can-access-the-data"></a>¿Quién puede acceder a los datos?
* Los usuarios con los roles Administrador de seguridad, Lector de seguridad, Lector de informes, Administrador de aplicaciones y Administrador de aplicaciones en la nube.
* Administradores globales


### <a name="what-azure-ad-license-do-you-need-to-access-provisioning-activities"></a>¿Qué licencia de Azure AD se necesita para acceder a las actividades de aprovisionamiento?

El inquilino debe tener una licencia de Azure AD Premium asociada para ver el informe de actividades de aprovisionamiento al completo. Consulte [Introducción a Azure Active Directory Premium](../fundamentals/active-directory-get-started-premium.md) para actualizar la edición de Azure Active Directory. 

## <a name="provisioning-logs"></a>Registros de aprovisionamiento

Los registros de aprovisionamiento proporcionan respuestas a las siguientes preguntas:

* ¿Qué grupos se han creado correctamente en ServiceNow?
* ¿Cómo se han importado los roles de Amazon Web Services?
* ¿Qué usuarios no se han creado correctamente en DropBox?

Se puede tener acceso a los registros de aprovisionamiento si se selecciona **Registros de aprovisionamiento** en la sección **Supervisión** de la hoja **Azure Active Directory** en [Azure Portal](https://portal.azure.com). Algunos registros de aprovisionamiento pueden tardar hasta dos horas en aparecer en el portal.

![Registros de aprovisionamiento](./media/concept-provisioning-logs/access-provisioning-logs.png "Registros de aprovisionamiento")


Un registro de aprovisionamiento tiene una vista de lista predeterminada que muestra:

- La identidad
- La acción
- El sistema de origen
- El sistema de destino
- El estado
- La fecha


![Columnas predeterminadas](./media/concept-provisioning-logs/default-columns.png "Columnas predeterminadas")

Puede personalizar la vista de lista, haga clic en **Columnas** en la barra de herramientas.

![Selector de columnas](./media/concept-provisioning-logs/column-chooser.png "Selector de columnas")

Esto le permite mostrar los campos adicionales o quitar los campos que ya se están mostrando.

![Columnas disponibles](./media/concept-provisioning-logs/available-columns.png "Columnas disponibles")

Seleccione un elemento de la vista de lista para obtener información más detallada.

![Información detallada](./media/concept-provisioning-logs/steps.png "Filter")


## <a name="filter-provisioning-activities"></a>Filtrado de las actividades de aprovisionamiento

Puede filtrar los datos de aprovisionamiento. Algunos valores de los filtros se rellenan dinámicamente en función del inquilino. Por ejemplo, si no tiene ningún evento de creación en el inquilino, no habrá una opción de filtro para la creación.
En la vista predeterminada, puede seleccionar los siguientes filtros:

- Identidad
- Date
- Status
- Acción


![Filter](./media/concept-provisioning-logs/default-filter.png "Filter")

El filtro **Identidad** le permite especificar el nombre o la identidad que le interesa. Esta identidad podría ser un usuario, un grupo, un rol u otro objeto. Puede buscar por nombre o por identificador de objeto. El identificador varía según el escenario. Por ejemplo, al aprovisionar un objeto de Azure AD en SalesForce, el identificador de origen es el identificador de objeto del usuario en Azure AD, mientras que el identificador de destino es el identificador del usuario en Salesforce. Al aprovisionar de Workday a Active Directory, el identificador de origen es el identificador de empleado del trabajador de Workday. Tenga en cuenta que el nombre del usuario puede no estar siempre presente en la columna de identidad. Siempre habrá un identificador. 


El filtro **Fecha** le permite definir un período de tiempo para los datos devueltos.  
Los valores posibles son:

- 1 mes
- 7 días
- 30 días
- 24 horas
- Intervalo de tiempo personalizado

Cuando se selecciona un período de tiempo personalizado, puede configurar una fecha de inicio y una fecha de finalización.


El filtro **Estado** le permite seleccionar:

- All
- Correcto
- Error
- Omitido



El filtro **Acción** permite filtrar:

- Crear 
- Actualizar
- Eliminar
- Disable
- Otros

Además, en los filtros de la vista predeterminada, también puede establecer los siguientes filtros:

- Id. del trabajo
- Id. de ciclo
- Id. de cambio
- Id. de origen
- Id. de destino
- Application


![Seleccionar un campo](./media/concept-provisioning-logs/add-filter.png "Seleccionar un campo")


- **Id. de trabajo**: es un identificador de trabajo único que está asociado a cada aplicación para la que se ha habilitado el aprovisionamiento.   

- **Id. de ciclo**: identifica de forma única el ciclo de aprovisionamiento. Puede compartir este identificador para permitir buscar el ciclo en el que se ha producido este evento.

- **Id. de cambio**: identificador único para el evento de aprovisionamiento. Puede compartir este identificador para permitir buscar el evento de aprovisionamiento.   


- **Sistema de origen**: permite especificar desde dónde se aprovisiona la identidad. Por ejemplo, al aprovisionar un objeto de Azure AD a ServiceNow, el sistema de origen es Azure AD. 

- **Sistema de destino**: permite especificar dónde se aprovisiona la identidad. Por ejemplo, al aprovisionar un objeto de Azure AD a ServiceNow, el sistema de destino es ServiceNow. 

- **Aplicación**: permite mostrar solo los registros de aplicaciones con un nombre para mostrar que contenga una cadena específica.

 

## <a name="provisioning-details"></a>Detalles de aprovisionamiento 

Al seleccionar un elemento en la vista de lista de aprovisionamiento, obtendrá más información sobre dicho elemento.
Los detalles se agrupan en función de las categorías siguientes:

- Pasos

- Solución de problemas y recomendaciones

- Propiedades modificadas

- Resumen


![Filter](./media/concept-provisioning-logs/provisioning-tabs.png "Pestañas")



### <a name="steps"></a>Pasos

En la pestaña **Pasos** se describen los pasos necesarios para aprovisionar un objeto. El aprovisionamiento de un objeto puede constar de cuatro pasos: 

- Importación del objeto
- Determinación de si el objeto está en el ámbito
- Coincidencia del objeto entre el origen y el destino
- Aprovisionamiento del objeto (realización de una acción, como crear, actualizar, eliminar o deshabilitar)



![Filter](./media/concept-provisioning-logs/steps.png "Filter")


### <a name="troubleshoot-and-recommendations"></a>Solución de problemas y recomendaciones


La pestaña **Solución de problemas y recomendaciones** proporciona el código de error y el motivo. La información del error solo está disponible en caso de error. 


### <a name="modified-properties"></a>Propiedades modificadas

En la pestaña **Propiedades modificadas** se muestran el valor anterior y el nuevo. En los casos en los que no hay ningún valor anterior, la columna con este valor está en blanco. 


### <a name="summary"></a>Resumen

En la pestaña **Resumen** se proporciona información general sobre lo que sucedió y los identificadores del objeto en el sistema de origen y de destino. 

## <a name="what-you-should-know"></a>Qué debería saber

- Azure Portal almacena durante 30 días los datos de aprovisionamiento detectados si tiene una edición Premium y durante siete días si tiene una edición gratuita.

- Puede usar el atributo Id. de cambio como identificador único. Esto es útil, por ejemplo, al hablar con el soporte técnico del producto.

- Actualmente no hay ninguna opción para descargar los datos de aprovisionamiento.

- Actualmente no se presta soporte técnico para el análisis de registros.

- Es posible que vea eventos omitidos para los usuarios que no estén en el ámbito. Esto es normal, especialmente cuando el ámbito de sincronización se establece en todos los usuarios y grupos. Nuestro servicio evaluará todos los objetos del inquilino, incluso los que estén fuera del ámbito. 

## <a name="error-codes"></a>Códigos de error

Use la tabla siguiente para entender mejor cómo resolver los errores que puede encontrar en los registros de aprovisionamiento. En el caso de los códigos de error que faltan, proporcione comentarios mediante el vínculo situado en la parte inferior de esta página. 

|Código de error|Descripción|
|---|---|
|Conflict, EntryConflict|Corrija los valores de atributo en conflicto en Azure AD o en la aplicación, o bien revise la configuración de atributo correspondiente si se supone que la cuenta de usuario en conflicto debía coincidir y tomarse. Revise la siguiente [documentación](../app-provisioning/customize-application-attributes.md) para obtener más información sobre cómo configurar atributos coincidentes.|
|TooManyRequests|La aplicación de destino rechazó este intento de actualizar el usuario porque está sobrecargado y recibe demasiadas solicitudes. No hay nada que hacer. Este intento se retirará automáticamente. También se ha notificado a Microsoft de este problema.|
|InternalServerError |La aplicación de destino devolvió un error inesperado. Es posible que haya un problema de servicio con la aplicación de destino que impide que esto funcione. Este intento se retirará automáticamente en 40 minutos.|
|InsufficientRights, MethodNotAllowed, NotPermitted, Unauthorized| Azure AD se pudo autenticar con la aplicación de destino, pero no tenía autorización para realizar la actualización. Revise las instrucciones proporcionadas por la aplicación de destino, así como el [tutorial](../saas-apps/tutorial-list.md) de la aplicación correspondiente.|
|UnprocessableEntity|La aplicación de destino devolvió una respuesta inesperada. Es posible que la configuración de la aplicación de destino no sea correcta o que haya un problema de servicio con la aplicación de destino que impide que esto funcione.|
|WebExceptionProtocolError |Error de protocolo HTTP al conectarse a la aplicación de destino. No hay nada que hacer. Este intento se retirará automáticamente en 40 minutos.|
|InvalidAnchor|Ya no existe un usuario que se había creado previamente o que coincidía con el servicio de aprovisionamiento. Compruebe que el usuario existe. Para forzar una nueva coincidencia de todos los usuarios, use MS Graph API para [reiniciar el trabajo](/graph/api/synchronization-synchronizationjob-restart?tabs=http&view=graph-rest-beta). Tenga en cuenta que al reiniciar el aprovisionamiento se desencadenará un ciclo inicial, que puede tardar en completarse. También se elimina la memoria caché que usa el servicio de aprovisionamiento para operar, lo que significa que todos los usuarios y grupos del inquilino tendrán que volver a evaluarse y se podrían quitar ciertos eventos de aprovisionamiento.|
|NotImplemented | La aplicación de destino devolvió una respuesta inesperada. Es posible que la configuración de la aplicación no sea correcta o que haya un problema de servicio con la aplicación de destino que impide que esto funcione. Revise las instrucciones proporcionadas por la aplicación de destino, así como el [tutorial](../saas-apps/tutorial-list.md) de la aplicación correspondiente. |
|MandatoryFieldsMissing, MissingValues |No se pudo crear el usuario porque faltan los valores necesarios. Corrija los valores de atributo que faltan en el registro de origen o revise la configuración de atributo coincidente para asegurarse de que no se omiten los campos obligatorios. [Más información](../app-provisioning/customize-application-attributes.md) sobre cómo configurar atributos coincidentes.|
|SchemaAttributeNotFound |No se pudo realizar la operación porque se especificó un atributo que no existe en la aplicación de destino. Consulte la [documentación](../app-provisioning/customize-application-attributes.md) sobre la personalización de atributos y asegúrese de que la configuración es correcta.|
|InternalError |Se produjo un error de servicio interno en el servicio de aprovisionamiento de Azure AD. No hay nada que hacer. Este intento se retirará automáticamente en 40 minutos.|
|InvalidDomain |No se pudo realizar la operación debido a un valor de atributo que contiene un nombre de dominio no válido. Actualice el nombre de dominio en el usuario o agréguelo a la lista de valores permitidos en la aplicación de destino. |
|Tiempo de espera |No se pudo completar la operación porque la aplicación de destino tardó demasiado tiempo en responder. No hay nada que hacer. Este intento se retirará automáticamente en 40 minutos.|
|LicenseLimitExceeded|No se pudo crear el usuario en la aplicación de destino porque no hay licencias disponibles para este usuario. Puede adquirir licencias adicionales para la aplicación de destino o revisar las asignaciones de usuario y la configuración de asignación de atributos para asegurarse de que se asignan los atributos correctos a los usuarios correctos.|
|DuplicateTargetEntries  |No se pudo completar la operación porque se encontró más de un usuario en la aplicación de destino con los atributos coincidentes configurados. Quite el usuario duplicado de la aplicación de destino o vuelva a configurar las asignaciones de atributos como se describe [aquí](../app-provisioning/customize-application-attributes.md).|
|DuplicateSourceEntries | No se pudo completar la operación porque se encontró más de un usuario con los atributos coincidentes configurados. Quite el usuario duplicado o vuelva a configurar las asignaciones de atributos como se describe [aquí](../app-provisioning/customize-application-attributes.md).|
|ImportSkipped | Cuando se evalúa a cada usuario, se intenta importar al usuario desde el sistema de origen. Este error suele producirse cuando al usuario que se importará le falta la propiedad coincidente definida en las asignaciones de atributos. Sin un valor presente en el objeto user para el atributo coincidente, no se pueden evaluar los cambios de ámbito, coincidencia o exportación. Tenga en cuenta que la presencia de este error no indica que el usuario está en el ámbito, ya que aún no se ha evaluado su ámbito.|
|EntrySynchronizationSkipped | El servicio de aprovisionamiento ha consultado correctamente el sistema de origen y ha identificado al usuario. No se realizó ninguna acción adicional en el usuario y se omitió. La omisión podría deberse a que el usuario está fuera del ámbito, o a que ya existe en el sistema de destino sin necesidad de realizar más cambios.|

## <a name="next-steps"></a>Pasos siguientes

* [Comprobación del estado de aprovisionamiento de usuarios](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md)
* [Problema al configurar el aprovisionamiento de usuarios para una aplicación de la galería de Azure AD](../app-provisioning/application-provisioning-config-problem.md)