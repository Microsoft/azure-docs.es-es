---
title: Versión preliminar pública del punto de conexión de Azure AD Connect Sync V2 | Microsoft Docs
description: En este documento se tratan las actualizaciones de la API de los puntos de conexión de Azure AD Connect Sync v2.
services: active-directory
author: billmath
manager: daveba
editor: ''
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 05/20/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: ce7041cd74a6bfd3ac736d3ae774324122ed737b
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89277075"
---
# <a name="azure-ad-connect-sync-v2-endpoint-api-public-preview"></a>API de los puntos de conexión de Azure AD Connect Sync V2 (versión preliminar pública) 
Microsoft ha implementado un nuevo punto de conexión (API) para Azure AD Connect que mejora el rendimiento de las operaciones del servicio de sincronización para Azure Active Directory. Al usar el nuevo punto de conexión V2, experimentará mejoras de rendimiento notables en la exportación y la importación respecto a Azure AD. Este nuevo punto de conexión admite lo siguiente:
    
 -  Grupos de sincronización de hasta 250 000 miembros
 - Mejoras en el rendimiento de la exportación y la importación a Azure AD
 
> [!NOTE]
> Actualmente, el nuevo punto de conexión no tiene ningún límite de tamaño de grupo configurado para los grupos de O365 que se vuelven a escribir. Esto puede afectar a las latencias del ciclo de sincronización de Active Directory.  Se recomienda aumentar los tamaños de grupo por incrementos.  


## <a name="pre-requisites"></a>Requisitos previos  
Para poder usar el nuevo punto de conexión V2, debe usar [Azure AD Connect versión 1.5.30.0](https://www.microsoft.com/download/details.aspx?id=47594) o posterior y seguir los pasos de implementación que se proporcionan a continuación para habilitar el punto de conexión V2 para el servidor de Azure AD Connect.   

>[!NOTE]
>Actualmente, esta versión preliminar pública solo está disponible en la nube global de Azure y no está disponible para [nubes nacionales](../develop/authentication-national-cloud.md).

### <a name="public-preview-limitations"></a>Limitaciones de la vista preliminar pública  
Aunque esta versión se ha sometido a pruebas exhaustivas, es posible que se produzcan problemas. Uno de los objetivos de esta versión preliminar pública es encontrar y corregir dichos problemas.  

>[!IMPORTANT]
> Aunque se proporciona soporte para esta versión preliminar pública, es posible que Microsoft no siempre pueda corregir de inmediato todos los problemas que pueda encontrar. Por esta razón, se recomienda utilizar el mejor criterio antes de implementar esta versión en el entorno de producción. 

## <a name="deployment-guidance"></a>Guía para la implementación 
Tendrá que implementar [Azure AD Connect versión 1.5.30.0](https://www.microsoft.com/download/details.aspx?id=47594) o posterior para usar el punto de conexión V2. Use el vínculo que se proporciona para la descarga. 

Se recomienda seguir el método de [migración oscilante](./how-to-upgrade-previous-version.md#swing-migration) para implementar el nuevo punto de conexión en su entorno. Esto proporcionará un plan de contingencia claro en el caso de que sea necesario realizar una reversión importante. En el ejemplo siguiente se muestra cómo se puede utilizar una migración oscilante en este escenario. Para más información sobre el método de implementación de la migración oscilante, vea el vínculo proporcionado. 

### <a name="swing-migration-for-deploying-v2-endpoint"></a>Migración oscilante para implementar el punto de conexión V2
Los pasos siguientes le guiarán a través de la implementación del punto de conexión V2 mediante el método oscilante.

1. Implemente el punto de conexión V2 en el servidor provisional actual. Este servidor se conoce como **servidor V2** en los pasos siguientes. El servidor activo actual seguirá procesando la carga de trabajo de producción mediante el punto de conexión V1, al que se llamará **servidor V1** a continuación.
1. Compruebe que el **servidor V2** todavía procesa las importaciones según lo esperado. En esta fase, los grupos grandes no se aprovisionarán en Azure AD o AD local, pero podrá comprobar que la actualización no ha provocado ningún otro impacto inesperado en el proceso de sincronización existente. 
2. Una vez finalizada la validación, cambie el **servidor V2** para que sea el servidor activo y el **servidor V1** para que sea el servidor provisional. En este momento, los grupos grandes que se encuentran en el ámbito que se va a sincronizar se aprovisionarán en Azure AD, al igual que los grupos unificados grandes de O365 se aprovisionarán en AD, si está habilitada la escritura diferida de grupos.
3. Compruebe que el **servidor V2** está en ejecución y que procesa los grupos grandes correctamente. Puede optar por permanecer en este paso y supervisar el proceso de sincronización durante un período.
  >[!NOTE]
  > Si necesita volver a la configuración anterior, puede realizar una migración oscilante desde el **servidor V2** al **servidor V1**. Dado que el punto de conexión V1 no admite grupos con más de 50 000 miembros, se eliminarán posteriormente todos los grupos grandes aprovisionados por Azure AD Connect, tanto en Azure AD como en AD local. 
4. Una vez que esté seguro de usar el punto de conexión V2, actualice el **servidor V1** para empezar a usar el punto de conexión V2. 
 

## <a name="expectations-of-performance-impact"></a>Previsiones de impacto en el rendimiento  
Cuando se usa el punto de conexión V2, las mejoras de rendimiento dependen del número de grupos sincronizados, el tamaño de esos grupos y la renovación de un grupo (la actividad resultante de agregar y quitar usuarios como miembros del grupo). El uso del nuevo punto de conexión, sin aumentar el número, el tamaño o la renovación de los grupos sincronizados, debe resultar en tiempos más cortos de exportación e importación a Azure AD. 
 
Sin embargo, las mejoras de rendimiento pueden ser infructuosas por el procesamiento adicional necesario al sincronizar grupos grandes. Podría acabar aumentando el tiempo de sincronización total al agregar demasiados grupos grandes al proceso de sincronización.  

Para comprender mejor cómo la adición de los nuevos grupos afectará al rendimiento de la sincronización, se recomienda que empiece por sincronizar solo unos pocos grupos grandes con menos de 100 000 miembros. Después, puede aumentar el número y el tamaño de los grupos mediante la incorporación de más en el ámbito, a través de la unidad organizativa, el atributo o el filtrado del tamaño máximo del grupo. Las mejoras de rendimiento se producirán en las tareas de exportación e importación del conector de Azure AD, no en el conector de AD local. 

## <a name="deployment-step-by-step"></a>Implementación paso a paso 
Las tres fases siguientes son un ejemplo detallado de la implementación del nuevo punto de conexión V2.  Use las fases como guía para la implementación.

### <a name="phase-1--install-and-validate-azure-ad-connect"></a>Fase 1: Instalar y validar Azure AD Connect 
Se recomienda que realice primero los pasos para instalar [Azure AD Connect versión 1.5.30.0](https://www.microsoft.com/download/details.aspx?id=47594) o posterior o actualizar a esta versión, y que valide el proceso de sincronización antes de pasar a la segunda fase, en la que habilitará el punto de conexión V2. En el servidor de Azure AD Connect: 


1. [Opcional] Realice la copia de seguridad de la base de datos. 
2. Instale [Azure AD Connect versión 1.5.30.0](https://www.microsoft.com/download/details.aspx?id=47594) o posterior, o actualice a esta versión.
3. Validación de la instalación 

### <a name="phase-2--enable-the-v2-endpoint"></a>Fase 2: Habilitar el punto de conexión V2 
El paso siguiente es habilitar el punto de conexión V2. 

> [!NOTE]
> Una vez habilitado el punto de conexión V2 para el servidor, podrá observar algunas mejoras en el rendimiento de la carga de trabajo existente. Sin embargo, aún no podrá sincronizar grupos con más de 50 000 miembros. 

Para cambiar al punto de conexión V2, siga estos pasos: 

1. Abra un símbolo del sistema de PowerShell como administrador. 
2. Deshabilite el programador de sincronización después de comprobar que no se está ejecutando ninguna operación de sincronización: 
 
 `Set-ADSyncScheduler -SyncCycleEnabled $false`
 
3. Importe el módulo nuevo: 
 
 `Import-Module 'C:\Program Files\Microsoft Azure AD Sync\Extensions\AADConnector.psm1'` 
 
4.  Cambie al punto de conexión v2:

 `Set-ADSyncAADConnectorExportApiVersion 2` 
 
 `Set-ADSyncAADConnectorImportApiVersion 2` 

 ![PowerShell](media/how-to-connect-sync-endpoint-api-v2/endpoint1.png)
 
Ya ha habilitado el punto de conexión v2 para su servidor. Tómese un tiempo para comprobar que no hay resultados inesperados después de habilitar el punto de conexión V2 antes de pasar a la fase siguiente, donde aumentará el límite de tamaño del grupo. 
>[!NOTE]
>Las rutas de acceso al archivo o módulo pueden usar una letra de unidad diferente, dependiendo de la ruta de instalación proporcionada al instalar Azure AD Connect. 


### <a name="phase-3--increase-the-group-membership-limit"></a>Fase 3: Aumentar el límite de pertenencia a grupos 
Después de comprobar que el servicio se está ejecutando sin resultados inesperados, puede continuar con el aumento del límite de pertenencia a grupos. Se recomienda aumentar primero el límite de pertenencia a un valor ligeramente superior, por ejemplo, 75 000 miembros, para ver cómo se sincronizan los grupos más grandes en Azure AD. Una vez que esté satisfecho con los resultados, puede aumentar el límite de miembros.  

El límite máximo es 250 000 miembros por grupo. 

Los siguientes pasos se pueden usar para aumentar el límite de pertenencia:  

1. Abra el editor de reglas de sincronización de Azure AD. 
2. En el editor, elija **Saliente** para la dirección. 
3. Haga clic en la regla de sincronización **Out to AAD – Group Join**. 
4. Haga clic en el botón **Editar**. ![Editar regla de sincronización](media/how-to-connect-sync-endpoint-api-v2/endpoint2.png)

6. Haga clic en el botón **Sí** para deshabilitar la regla predeterminada y crear una copia editable.
 ![Editar regla de sincronización](media/how-to-connect-sync-endpoint-api-v2/endpoint3.png)

7. En la ventana emergente de la página **Description**, establezca la prioridad en un valor disponible entre 1 y 99. ![Editar regla de sincronización](media/how-to-connect-sync-endpoint-api-v2/endpoint4.png)

8. En la página **Transformations**, actualice el valor de **Source** por la transformación **member**, reemplazando "50000" por un valor comprendido entre 50001 y 250000. Este cambio aumentará el tamaño máximo de pertenencia de los grupos que se sincronizarán con Azure AD. Se recomienda comenzar con un número de 100 000 para saber el impacto que tendrá la sincronización de grupos grandes en el rendimiento de la sincronización. 
 
 **Ejemplo** 
 
 `IIF((ValueCount("member")> 75000),Error("Maximum Group member count exceeded"),IgnoreThisFlow)` 
 
 ![Editar regla de sincronización](media/how-to-connect-sync-endpoint-api-v2/endpoint5.png)

9. Haga clic en Guardar 
10. Abra un símbolo del sistema de PowerShell como administrador. 
11. Vuelva a habilitar el programador de sincronización. 
 
 `Set-ADSyncScheduler -SyncCycleEnabled $true` 
 
>[!NOTE]
> Si Azure AD Connect Health no está habilitado, cambie la configuración del registro de eventos de aplicación de Windows para archivar los registros, en lugar de sobrescribirlos. Los registros se pueden usar para ayudar en el futuro trabajo de solución de problemas. 

>[!NOTE]
> Después de habilitar el nuevo punto de conexión, puede ver errores de exportación adicionales en el conector de AAD con el nombre "dn-attributes-failure". Habrá una entrada del registro de eventos correspondiente para cada error con el identificador 6949. Los errores son informativos y no indican un problema con la instalación, sino que el proceso de sincronización no pudo agregar determinados miembros a un grupo en Azure AD porque el propio objeto miembro no se sincronizó con Azure AD. 

El nuevo código del punto de conexión V2 controla algunos tipos de errores de exportación de forma ligeramente distinta a como lo hizo el código de V1.  Es posible que vea más mensajes de error informativos al usar el punto de conexión V2. 

>[!NOTE]
> Al actualizar Azure AD Connect, asegúrese de que se vuelven a ejecutar los pasos de la fase 2, ya que los cambios no se conservan durante el proceso de actualización. 

Durante los próximos aumentos del límite de miembros de grupo en la regla de sincronización **Out to AAD – Group Join**, no es necesario realizar una sincronización completa, por lo que puede optar por suprimir la sincronización completa ejecutando el siguiente comando en PowerShell. 

 `Set-ADSyncSchedulerConnectorOverride -FullSyncRequired $false -ConnectorName "<AAD Connector Name>" `
 
>[!NOTE]
> Si tiene grupos unificados de O365 con más de 50 000 miembros, los grupos se leerán en Azure AD Connect y, si la escritura diferida de grupos está habilitada, se escribirán en la instancia de AD local. 

## <a name="rollback"></a>Reversión 
Si ha habilitado el punto de conexión V2 y necesita revertirlo, siga estos pasos: 

1. En el servidor de Azure AD Connect: a. [Opcional] Realice la copia de seguridad de la base de datos. 
2. Abra un símbolo del sistema de PowerShell como administrador.
3. Deshabilite el programador de sincronización después de comprobar que no se está ejecutando ninguna operación de sincronización:
 
 `Set-ADSyncScheduler -SyncCycleEnabled $false`

4. Cambie al punto de conexión V1*. 
 
 `Import-Module 'C:\Program Files\Microsoft Azure AD Sync\Extensions\AADConnector.psm1'` 

 `Set-ADSyncAADConnectorExportApiVersion 1`

 `Set-ADSyncAADConnectorImportApiVersion 1`
 
5. Abra el editor de reglas de sincronización de Azure AD. 
6. Elimine la copia editable de la regla de sincronización **Out to AAD – Group Join**. 
7. Habilite la copia predeterminada de la regla de sincronización **Out to AAD – Group Join**. 
8. Abra un símbolo del sistema de PowerShell como administrador. 
9. Vuelva a habilitar el programador de sincronización. 
 
 `Set-ADSyncScheduler -SyncCycleEnabled $true`
 
>[!NOTE]
> Al revertir de los puntos de conexión V2 a V1, los grupos sincronizados con más de 50 000 miembros se eliminarán después de ejecutar una sincronización completa, tanto en el caso de los grupos de AD aprovisionados en Azure AD como de los grupos unificados de O365 aprovisionados en AD. 

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes  
**P: ¿Puede un cliente usar esta característica en producción?**   
</br>Sí, se puede usar en entornos de producción, con la salvedad que se mencionó antes.
 
**P: ¿Con quién debe ponerse en contacto el cliente cuando hay algún problema?**   
</br>Si necesita soporte técnico al usar esta característica, debe abrir un caso de soporte técnico. 
 
**P: ¿Se prevén actualizaciones frecuentes de la versión preliminar pública?**   
</br>Hay un grado limitado de cambios continuos durante una versión preliminar pública. Debe evaluar este riesgo al implementar las características de la versión preliminar pública en producción.  
 
**P: ¿Cuánto tiempo hay hasta el siguiente hito?**   
</br>Las funcionalidades de la versión preliminar pública se pueden retirar y posiblemente rediseñar antes de llegar a hitos adicionales.  
 
## <a name="next-steps"></a>Pasos siguientes

* [Sincronización de Azure AD Connect: comprender y personalizar la sincronización](how-to-connect-sync-whatis.md)
* [Integración de las identidades locales con Azure Active Directory](whatis-hybrid-identity.md)