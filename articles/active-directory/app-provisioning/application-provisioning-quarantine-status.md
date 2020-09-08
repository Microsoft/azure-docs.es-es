---
title: Estado de aprovisionamiento de aplicaciones de cuarentena | Microsoft Docs
description: Cuando haya configurado una aplicación para el aprovisionamiento automático de usuarios, obtenga información sobre el estado de aprovisionamiento de los medios de cuarentena y cómo borrarlo.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-provisioning
ms.workload: identity
ms.topic: troubleshooting
ms.date: 04/28/2020
ms.author: kenwith
ms.reviewer: arvinh
ms.openlocfilehash: 54d02b3189825d08716b73b7250efd4e3f334aa0
ms.sourcegitcommit: 3bf69c5a5be48c2c7a979373895b4fae3f746757
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88234751"
---
# <a name="application-provisioning-in-quarantine-status"></a>Aprovisionamiento de aplicaciones en el estado de cuarentena

El servicio de aprovisionamiento de Azure AD supervisa el estado de la configuración y establece las aplicaciones incorrectas en un estado de "cuarentena". Si todas o la mayoría de las llamadas realizadas al sistema de destino no tienen éxito sistemáticamente debido a un error (como en el caso de credenciales de administrador no válidas), el trabajo de aprovisionamiento se establece en un estado de "cuarentena".

Mientras esté en cuarentena, la frecuencia de los ciclos incrementales se reduce gradualmente a una vez al día. El trabajo de aprovisionamiento se quita de la cuarentena después de que se hayan resuelto todos los errores y se inicie el siguiente ciclo de sincronización. Si el trabajo de aprovisionamiento permanece en cuarentena durante más de cuatro semanas, se deshabilita (deja de ejecutarse).

## <a name="how-do-i-know-if-my-application-is-in-quarantine"></a>¿Cómo puedo saber si mi aplicación está en cuarentena?

Hay tres maneras de comprobar si una aplicación está en cuarentena:
  
- En Azure Portal, vaya a **Azure Active Directory** > **Aplicaciones empresariales** > &lt;*nombre de la aplicación*&gt; > **Aprovisionamiento** y compruebe si la barra de progreso tienen algún mensaje de cuarentena.   

  ![Barra de estado de aprovisionamiento que muestra el estado de cuarentena](./media/application-provisioning-quarantine-status/progress-bar-quarantined.png)

- En Azure Portal, vaya a **Azure Active Directory** > **Registros de auditoría** > filtro en **Actividad: cuarentena** y revise el historial de cuarentena. Aunque la vista de la barra de progreso, tal como se ha descrito anteriormente, muestra si el aprovisionamiento está actualmente en cuarentena, los registros de auditoría permiten ver el historial de cuarentena de una aplicación. 

- Utilice la solicitud de Microsoft Graph [Get synchronizationJob](/graph/api/synchronization-synchronizationjob-get?tabs=http&view=graph-rest-beta) (Obtener trabajo de sincronización) para obtener, mediante programación, el estado del trabajo de aprovisionamiento:

```microsoft-graph
        GET https://graph.microsoft.com/beta/servicePrincipals/{id}/synchronization/jobs/{jobId}/
```

- Compruebe su correo electrónico. Cuando una aplicación se pone en cuarentena, se envía un correo electrónico de notificación de una sola vez. Si el motivo de la cuarentena cambia, se envía un correo electrónico actualizado que muestra la nueva razón para la cuarentena. Si no ve un correo electrónico:

  - Asegúrese de haber especificado un **correo electrónico de notificación** válido en la configuración de aprovisionamiento de la aplicación.
  - Asegúrese de que no haya ningún filtrado de correo no deseado en la bandeja de entrada del correo electrónico de notificación.
  - Asegúrese de que no ha cancelado la suscripción a los mensajes de correo electrónico.

## <a name="why-is-my-application-in-quarantine"></a>¿Por qué mi aplicación está en cuarentena?

|Descripción|Acción recomendada|
|---|---|
|**Problema de cumplimiento de SCIM:** se devolvió una respuesta HTTP/404 No encontrado en lugar de la respuesta HTTP/200 Correcto esperada. En este caso, el servicio de aprovisionamiento de Azure AD realizó una solicitud a la aplicación de destino y recibió una respuesta inesperada.|Consulte la sección Credenciales de administrador para ver si la aplicación requiere de especificar la dirección URL del inquilino. Además, asegúrese de que la dirección URL sea correcta. Si no experimenta ningún problema, póngase en contacto con el desarrollador de la aplicación para asegurarse de que su servicio sea conforme con SCIM. https://tools.ietf.org/html/rfc7644#section-3.4.2 |
|**Credenciales no válidas:** al intentar autorizar el acceso a la aplicación de destino, se recibió una respuesta de la aplicación de destino que indica que las credenciales proporcionadas no son válidas.|Vaya a la sección Credenciales de administrador de la interfaz de usuario de la configuración de aprovisionamiento y autorice el acceso de nuevo con credenciales válidas. Si la aplicación está en la galería, revise el tutorial de configuración correspondiente para ver los pasos adicionales necesarios.|
|**Roles duplicados:** los roles importados de ciertas aplicaciones como Salesforce y Zendesk deben ser únicos. |Vaya al [manifiesto](../develop/reference-app-manifest.md) de la aplicación en Azure Portal y quite el rol duplicado.|

 Una solicitud de Microsoft Graph para obtener el estado del trabajo de aprovisionamiento muestra la siguiente razón para la cuarentena:

- `EncounteredQuarantineException` indica que se proporcionaron credenciales no válidas. El servicio de aprovisionamiento no puede establecer una conexión entre el sistema de origen y el de destino.

- `EncounteredEscrowProportionThreshold` indica que el aprovisionamiento superó el umbral de custodia. Esta condición se crea cuando se produce un error en más del 60 % de los eventos de aprovisionamiento.

- `QuarantineOnDemand` significa que hemos detectado un problema con la aplicación y la hemos establecido manualmente en cuarentena.

## <a name="how-do-i-get-my-application-out-of-quarantine"></a>¿Cómo puedo sacar la aplicación de la cuarentena?

En primer lugar, resuelva el problema que provocó que la aplicación se pusiera en cuarentena.

- Compruebe la configuración de aprovisionamiento de la aplicación para asegurarse de que ha [escrito las credenciales de administrador válidas](../app-provisioning/configure-automatic-user-provisioning-portal.md#configuring-automatic-user-account-provisioning). Azure AD debe ser capaz de establecer una relación de confianza con la aplicación de destino. Asegúrese de que ha especificado las credenciales válidas y de que su cuenta tiene los permisos necesarios.

- Revise los [registros de aprovisionamiento](../reports-monitoring/concept-provisioning-logs.md) para investigar más a fondo qué errores están causando la cuarentena y así poder solucionarlos. Para acceder a los registros de aprovisionamiento en Azure Portal, seleccione **Azure Active Directory** &gt; **Aplicaciones empresariales** &gt; **Registros de aprovisionamiento (versión preliminar)** en la sección **Actividad**.

Una vez resuelto el problema, reinicie el trabajo de aprovisionamiento. Algunos cambios en la configuración de aprovisionamiento de la aplicación, como asignaciones de atributos o filtros de ámbito, reiniciarán automáticamente el proceso aprovisionamiento. La barra de progreso en la página **Provisioning** (Aprovisionamiento) de la aplicación indica cuándo comenzó el aprovisionamiento por última vez. Si necesita reiniciar el trabajo de aprovisionamiento manualmente, use uno de los métodos siguientes:  

- Use Azure Portal para reiniciar el trabajo de aprovisionamiento. En la página **Provisioning** (Aprovisionamiento) de la aplicación en **Settings** (Configuración), seleccione **Clear state and restart synchronization** (Borrar estado y reinicie la sincronización) y establezca el **estado de aprovisionamiento** en **On** (Activado). Esta acción reinicia completamente el servicio de aprovisionamiento, que puede tardar algún tiempo. Se volverá a ejecutar un ciclo inicial completo, que borrará los elementos en custodia, quitará la aplicación de la cuarentena y borrará las marcas de agua.

- Use Microsoft Graph para [reiniciar el trabajo de aprovisionamiento](/graph/api/synchronization-synchronizationjob-restart?tabs=http&view=graph-rest-beta). Así tendrá control total sobre lo que reinicie. Puede optar por borrar los elementos en custodia (para reiniciar el contador de custodia que se acumula en el estado de cuarentena), borrar la cuarentena (para quitar la aplicación de la cuarentena) o borrar las marcas de agua. Use la siguiente solicitud:
 
```microsoft-graph
        POST /servicePrincipals/{id}/synchronization/jobs/{jobId}/restart
```

Reemplace "{id}" con el valor de la ID de la aplicación y reemplace "{jobId}" con la [ID del trabajo de sincronización](/graph/api/resources/synchronization-configure-with-directory-extension-attributes?tabs=http&view=graph-rest-beta#list-synchronization-jobs-in-the-context-of-the-service-principal).