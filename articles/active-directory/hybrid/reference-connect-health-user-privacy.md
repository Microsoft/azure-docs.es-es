---
title: Azure AD Connect Health y privacidad del usuario | Microsoft Docs
description: En este documento, se describe la privacidad del usuario con Azure AD Connect Health.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: hybrid
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: reference
ms.date: 04/26/2018
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 58ecdfefa79716bcc0a69063293d5a1067693a78
ms.sourcegitcommit: 3fb5e772f8f4068cc6d91d9cde253065a7f265d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/31/2020
ms.locfileid: "89182367"
---
# <a name="user-privacy-and-azure-ad-connect-health"></a>Privacidad del usuario y Azure AD Connect Health 

[!INCLUDE [Privacy](../../../includes/gdpr-intro-sentence.md)]

>[!NOTE] 
>Este artículo trata sobre Azure AD Connect Health y la privacidad del usuario.  Para obtener información sobre Azure AD Connect y la privacidad del usuario, consulte el artículo que se indica [aquí](reference-connect-user-privacy.md).

## <a name="user-privacy-classification"></a>Clasificación de privacidad del usuario
Azure AD Connect Health se incluye dentro de la categoría de **procesador de datos** de la clasificación del GDPR. Como canalización del procesador de datos, el servicio proporciona servicios de procesamiento de datos a asociados clave y consumidores finales. Azure AD Connect Health no genera datos de usuario y no tiene control independiente sobre qué datos personales se recopilan y cómo se usan. La recuperación de datos, la agregación, el análisis y los informes en Azure AD Connect Health se basan en los datos locales existentes. 

## <a name="data-retention-policy"></a>Directiva de retención de datos
Azure AD Connect Health no genera informes, ni realiza análisis ni proporciona información después de 30 días. Por lo tanto, no almacena, procesa ni retiene datos más allá de ese período. Este diseño es compatible con el reglamento del GDPR, las regulaciones de cumplimiento de la privacidad de Microsoft y las directivas de retención de datos de Azure AD. 

El hecho de que aparezca en servidores la alerta de **error** **Los datos del servicio de mantenimiento no están actualizados** durante más de 30 días consecutivos sugiere que ningún dato ha llegado a Connect Health durante ese tiempo. Estos servidores se deshabilitarán y no aparecerán en el portal de Connect Health. Para volver a habilitar los servidores, debe desinstalar y [volver a instalar el agente de mantenimiento](how-to-connect-health-agent-install.md). Tenga en cuenta que esto no se aplica a las **advertencias** con el mismo tipo de alerta. Las advertencias indican que parte de los datos no está presente en el servidor para el que se generan las alertas. 
 
## <a name="disable-data-collection-and-monitoring-in-azure-ad-connect-health"></a>Deshabilitación de la recopilación de datos y la supervisión en Azure AD Connect Health
Azure AD Connect Health le permite detener la recopilación de datos de cada servidor supervisado individual o de una instancia de un servicio supervisado. Por ejemplo, puede detener la recopilación de datos de servidores de ADFS (Servicios de federación de Active Directory) individuales que se supervisan mediante Azure AD Connect Health. También puede detener la recopilación de datos de la instancia completa de ADFS que se supervisa mediante Azure AD Connect Health. Si decide hacerlo, los servidores correspondientes se eliminan del portal de Azure AD Connect Health después de detener la recopilación de datos. 

>[!IMPORTANT]
> Para eliminar servidores supervisados de Azure AD Connect Health, necesita privilegios de administrador global de Azure AD o tener el rol de colaborador en Azure RBAC.
>
> Quitar un servidor o una instancia de servicio de Azure AD Connect Health no es una acción reversible. 

### <a name="what-to-expect"></a>Qué cabe esperar
Si detiene la recopilación de datos y la supervisión de un servidor supervisado individual o de una instancia de un servicio supervisado, tenga en cuenta lo siguiente:

- Cuando se elimina una instancia de un servicio supervisado, se quita la instancia de la lista de servicios de supervisión de Azure AD Connect Health en el portal. 
- Cuando se elimina un servidor supervisado o una instancia de un servicio supervisado, no se desinstala el Agente de mantenimiento no se quita de los servidores. El Agente de mantenimiento se configura para no enviar datos a Azure AD Connect Health. Debe desinstalar manualmente el Agente de mantenimiento en los servidores supervisados previamente.
- Si no ha desinstalado el agente de mantenimiento antes de realizar este paso, es posible que vea eventos de error en los servidores relacionados con el agente de mantenimiento.
- Todos los datos que pertenecen a la instancia del servicio supervisado se eliminan según la directiva de retención de datos de Microsoft Azure.

### <a name="disable-data-collection-and-monitoring-for-an-instance-of-a-monitored-service"></a>Deshabilitación de la recopilación de datos y la supervisión en una instancia de un servicio supervisado
Consulte [cómo eliminar una instancia de servicio de Azure AD Connect Health](how-to-connect-health-operations.md#delete-a-service-instance-from-azure-ad-connect-health-service).

### <a name="disable-data-collection-and-monitoring-for-a-monitored-server"></a>Deshabilitación de la recopilación de datos y la supervisión en un servidor supervisado
Consulte [cómo eliminar un servidor de Azure AD Connect Health](how-to-connect-health-operations.md#delete-a-server-from-the-azure-ad-connect-health-service).

### <a name="disable-data-collection-and-monitoring-for-all-monitored-services-in-azure-ad-connect-health"></a>Deshabilitación de la recopilación de datos y la supervisión de todos los servicios supervisados en Azure AD Connect Health
Azure AD Connect Health también proporciona la opción de detener la recopilación de datos de **todos** los servicios registrados en el inquilino. Recomendamos que se evalúe con cuidado y que lo admitan todos los administradores globales antes de realizar ninguna acción. Una vez iniciado el proceso, el servicio de Connect Health dejará de recibir, procesar e informar de los datos de todos los servicios. Se conservarán los datos existentes en el servicio de Connect Health durante no más de 30 días.
Si quiere detener la recopilación de datos de un servidor específico, siga los pasos que se indican en la eliminación de servidores específicos. Para detener la recopilación de datos de todos los inquilinos, siga estos pasos para detener la recopilación de datos y eliminar todos los servicios del inquilino.

1. Haga clic en **Configuración general** en la configuración de la hoja principal. 
2. Haga clic en el botón **Detener recopilación de datos** en la parte superior de la hoja. Las demás opciones de configuración del inquilino se deshabilitarán una vez que se inicie el proceso.  
 
   ![Detener recopilación de datos](./media/reference-connect-health-user-privacy/gdpr4.png)
  
3. Compruebe la lista de servicios incorporados que se ven afectados al detener las recopilaciones de datos. 
4. Escriba el nombre exacto del inquilino para habilitar el botón de acción **Eliminar**.
5. Haga clic en **Eliminar** para desencadenar la eliminación de todos los servicios. Connect Health dejará de recibir, procesar e informar de todos los datos enviados desde los servicios incorporados. Todo el proceso puede tardar hasta 24 horas. Tenga en cuenta que este paso no es reversible. 
6. Una vez completado el proceso, ya no verá ningún servicio registrado en Connect Health. 

   ![Después de detener la recopilación de datos](./media/reference-connect-health-user-privacy/gdpr5.png)

## <a name="re-enable-data-collection-and-monitoring-in-azure-ad-connect-health"></a>Nueva habilitación de la recopilación de datos y la supervisión en Azure AD Connect Health
Para volver a habilitar la supervisión en Azure AD Connect Health en un servicio supervisado previamente eliminado, debe desinstalar y [volver a instalar el Agente de mantenimiento](how-to-connect-health-agent-install.md) en todos los servidores.

### <a name="re-enable-data-collection-and-monitoring-for-all-monitored-services"></a>Nueva habilitación de la recopilación de datos y la supervisión de todos los servidores supervisados

La recopilación de datos de todo el inquilino se puede reanudar en Azure AD Connect Health. Recomendamos que se evalúe con cuidado y que lo admitan todos los administradores globales antes de realizar ninguna acción.

>[!IMPORTANT]
> Los pasos siguientes estarán disponibles una vez transcurridas 24 horas tras la acción de deshabilitación.
> Después de habilitar la recopilación de datos, los datos de conclusión y supervisión presentados en Connect Health no mostrarán datos heredados recopilados antes. 

1. Haga clic en **Configuración general** en la configuración de la hoja principal. 
2. Haga clic en el botón **Habilitar recopilación de datos** en la parte superior de la hoja. 
 
   ![Habilitación de recolección de datos](./media/reference-connect-health-user-privacy/gdpr6.png)
 
3. Escriba el nombre exacto del inquilino para activar el botón **Habilitar**.
4. Haga clic en el botón **Habilitar** para conceder el permiso de recopilación de datos en el servicio de Connect Health. El cambio se aplicará en breve. 
5. Siga el [proceso de instalación](how-to-connect-health-agent-install.md) para volver a instalar el agente en los servidores que quiera supervisar y los servicios estarán en el portal.  


## <a name="next-steps"></a>Pasos siguientes
* [Revisar la directiva de privacidad de Microsoft en Trust Center](https://www.microsoft.com/trustcenter)
* [Azure AD Connect y privacidad del usuario](reference-connect-user-privacy.md)

