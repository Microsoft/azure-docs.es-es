---
title: Actualización de la aplicación de Service Fabric
description: Este artículo proporciona una introducción a la actualización de una aplicación de Service Fabric, incluida la elección de los modos de actualización y las comprobaciones de estado.
ms.topic: conceptual
ms.date: 8/5/2020
ms.openlocfilehash: cb0c1c0049957244b94b59707b70e47dc53f6c9f
ms.sourcegitcommit: d8b8768d62672e9c287a04f2578383d0eb857950
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88067518"
---
# <a name="service-fabric-application-upgrade"></a>Actualización de la aplicación de Service Fabric
Una aplicación de Azure Service Fabric es una colección de servicios. Durante una actualización, Service Fabric compara el nuevo [manifiesto de aplicación](service-fabric-application-and-service-manifests.md) con la versión anterior y determina qué servicios de la aplicación requieren actualizaciones. Service Fabric compara los números de versión en los manifiestos de servicio con los números de versión en la versión anterior. Si un servicio no ha cambiado, no se actualiza.

> [!NOTE]
> [ApplicationParameter](https://docs.microsoft.com/dotnet/api/system.fabric.description.applicationdescription.applicationparameters?view=azure-dotnet#System_Fabric_Description_ApplicationDescription_ApplicationParameters)s no se conservan en una actualización de la aplicación. Para conservar los parámetros actuales de la aplicación, el usuario debe obtener primero los parámetros y luego pasarlos a la llamada API de actualización, como se indica a continuación:
```powershell
$myApplication = Get-ServiceFabricApplication -ApplicationName fabric:/myApplication
$appParamCollection = $myApplication.ApplicationParameters

$applicationParameterMap = @{}
foreach ($pair in $appParamCollection)
{
    $applicationParameterMap.Add($pair.Name, $pair.Value);
}

Start-ServiceFabricApplicationUpgrade -ApplicationName fabric:/myApplication -ApplicationTypeVersion 2.0.0 -ApplicationParameter $applicationParameterMap -Monitored -FailureAction Rollback
```

## <a name="rolling-upgrades-overview"></a>Información general de las actualizaciones graduales
En una actualización de la aplicación gradual, la actualización se realiza por etapas. En cada etapa, la actualización se aplica a un subconjunto de nodos del clúster, llamado dominio de actualización. Como resultado, la aplicación permanece disponible durante la actualización. Durante la actualización, el clúster puede contener una mezcla de las versiones anteriores y nuevas.

Por esa razón, las dos versiones deben ser compatibles con las versiones anteriores y nuevas. Si no son compatibles, el administrador de la aplicación es responsable del ensayo de una actualización multifase para mantener la disponibilidad. En una actualización multifase, el primer paso es actualizar a una versión intermedia de la aplicación que sea compatible con la versión anterior. El segundo paso es actualizar la versión final que interrumpe la compatibilidad con la versión anterior a la actualización, pero es compatible con la versión intermedia.

Los dominios de actualización se especifican en el manifiesto de clúster al configurar el clúster. Los dominios de actualización no reciben actualizaciones en un orden concreto. Un dominio de actualización es una unidad lógica de implementación para una aplicación. Los dominios de actualización permiten que la disponibilidad de los servicios se mantenga alta durante una actualización.

Las actualizaciones no graduales son posibles si la actualización se aplica a todos los nodos del clúster, que es el caso cuando la aplicación tiene un solo dominio de actualización. No se recomienda este enfoque, ya que el servicio deja de funcionar y no está disponible en el momento de la actualización. Además, Azure no ofrece ninguna garantía cuando se configura un clúster con un solo dominio de actualización.

Una vez completada la actualización, se mantendría la versión de todos los servicios y réplicas (instancias), es decir, si la actualización se realiza correctamente, se actualizarán a la nueva versión; si no se realiza correctamente y se revierte, se revertirían a la versión anterior.

## <a name="health-checks-during-upgrades"></a>Comprobaciones de estado durante las actualizaciones
Para realizar una actualización, las directivas de mantenimiento deben establecerse (o se pueden usar valores predeterminados). Una actualización se califica como correcta cuando se actualizan todos los dominios de actualización dentro de los tiempos de espera especificados, y dichos dominios se consideran correctos.  Un dominio de actualización correcto significa que el dominio de actualización pasa todas las comprobaciones de estado especificadas en la directiva de mantenimiento. Por ejemplo, una directiva de mantenimiento puede exigir que todos los servicios dentro de una instancia de la aplicación tienen que ser *correctos*, tal como Service Fabric define el estado correcto.

Las directivas de mantenimiento y comprobaciones de estado durante la actualización de Service Fabric son independientes de los servicios y aplicaciones. Es decir, no se realiza ninguna prueba específica del servicio.  Por ejemplo, el servicio podría tener un requisito de rendimiento, pero Service Fabric no tiene la información para comprobar el rendimiento. Consulte los [artículos sobre mantenimiento](service-fabric-health-introduction.md) para encontrar ver las comprobaciones que se llevan a cabo. Las comprobaciones que se producen durante una actualización incluyen pruebas para saber si el paquete de aplicación se copió correctamente, si se inició la instancia y así sucesivamente.

El estado de la aplicación es una agregación de las entidades secundarias de la aplicación. En resumen, Service Fabric evalúa el estado de la aplicación a través del estado notificado en la aplicación. También evalúa el estado de todos los servicios de la aplicación de este modo. Service Fabric evalúa aún más el estado de los servicios de aplicación al agregar el estado de sus elementos secundarios, como la réplica de servicio. Cuando se cumple la directiva de mantenimiento de la aplicación, se puede continuar con la actualización. Si se infringe la directiva de mantenimiento, se produce un error en la actualización de la aplicación.

## <a name="upgrade-modes"></a>Modos de actualización
El modo en que se recomienda que se actualice la aplicación es el modo supervisado, que es el de uso más común. El modo supervisado realiza la actualización en un dominio de actualización y, si se superan todas las comprobaciones de estado (según la directiva especificada), pasa al siguiente dominio de actualización automáticamente.  Si se produce un error en las comprobaciones de estado o los tiempos de espera llegan a su fin, la actualización se revierte para el dominio de actualización o el modo cambia a manual sin supervisión. Puede configurar la actualización para elegir uno de los dos modos en actualizaciones con errores. 

El modo manual sin supervisión requiere intervención manual tras cada actualización en un dominio de actualización para iniciar la actualización en el siguiente dominio de actualización. No se realiza ninguna comprobación de mantenimiento de Service Fabric. El administrador realiza las comprobaciones de estado antes de iniciar la actualización en el siguiente dominio de actualización.

## <a name="upgrade-default-services"></a>Actualización de servicios predeterminados
Algunos parámetros del servicio predeterminados definidos en el [manifiesto de aplicación](service-fabric-application-and-service-manifests.md) también se pueden actualizar como parte de una actualización de la aplicación. Solo se pueden cambiar como parte de una actualización los parámetros del servicio que admitan su modificación mediante [Update-ServiceFabricService](/powershell/module/servicefabric/update-servicefabricservice?view=azureservicefabricps). El comportamiento del cambio de los servicios predeterminados durante la actualización de la aplicación es el siguiente:

1. Se crean los servicios predeterminados del nuevo manifiesto de aplicación que no existen aún en el clúster.
2. Se actualizan los servicios predeterminados que existen tanto en el anterior manifiesto de aplicación como en el nuevo. Los parámetros del servicio predeterminados del nuevo manifiesto de aplicación sobrescriben los parámetros del servicio existente. La actualización de aplicaciones se revertirá automáticamente tras un error de actualización de un servicio predeterminado.
3. Se eliminan los servicios predeterminados que no existen en el nuevo manifiesto de aplicación si estos existen en el clúster. **Tenga en cuenta que eliminar un servicio predeterminado dará como resultado la eliminación del estado del servicio y no se puede deshacer.**

Cuando se revierte una actualización de la aplicación, los parámetros del servicio predeterminados se revertir a sus valores antiguos antes de iniciar la actualización, pero los servicios eliminados no se pueden volver a crear con su estado anterior.

> [!TIP]
> La opción de configuración [EnableDefaultServicesUpgrade](service-fabric-cluster-fabric-settings.md) del clúster debe ser *true* para habilitar las anteriores reglas 2 y 3 (actualización y eliminación del servicio predeterminado). Esta característica se admite a partir de Service Fabric versión 5.5.

## <a name="upgrading-multiple-applications-with-https-endpoints"></a>Actualización de varias aplicaciones con puntos de conexión HTTTPS
Debe tener cuidado de no usar el **mismo puerto** para distintas instancias de la misma aplicación cuando use HTTP**S**. El motivo es que Service Fabric no podrá actualizar el certificado para una de las instancias de la aplicación. Por ejemplo, si la aplicación 1 o 2 desean actualizar sus certificados 1 a 2. Una vez realizada la actualización, es posible que Service Fabric borre el registro del certificado 1 con http.sys, aunque la otra aplicación lo siga utilizando. Para evitar esto, Service Fabric detecta que hay ya otra instancia de la aplicación registrada en el puerto con el certificado (debido a http.sys) y se produce un error en la operación.

Por lo tanto, Service Fabric no admite la actualización de dos servicios diferentes que usen **el mismo puerto** en diferentes instancias de la aplicación. En otras palabras, no puede utilizar el mismo certificado en diferentes servicios en el mismo puerto. Si necesita tener un certificado compartido en el mismo puerto, tiene que asegurarse de que los servicios se encuentren en distintos equipos con restricciones de posición. También puede considerar la posibilidad de utilizar puertos dinámicos de Service Fabric para cada servicio en cada instancia de la aplicación. 

Si visualiza un error de actualización con https, aparecerá una advertencia de error que indica que "La API de servidor HTTP de Windows no admite varios certificados para las aplicaciones que comparten un puerto".

## <a name="application-upgrade-flowchart"></a>Diagrama de flujo de actualización de la aplicación
El siguiente diagrama de flujo puede ayudarle a comprender el proceso de actualización de una aplicación de Service Fabric. En concreto, el flujo describe cómo los tiempos de espera, incluidos *HealthCheckStableDuration*, *HealthCheckRetryTimeout* y *UpgradeHealthCheckInterval*, ayudan a controlar cuándo se considera un éxito o un fracaso la actualización en un dominio de actualización.

![El proceso de actualización de una aplicación de Service Fabric][image]

## <a name="next-steps"></a>Pasos siguientes
[actualización de aplicaciones usando Visual Studio](service-fabric-application-upgrade-tutorial.md) ofrece información para actualizar una aplicación mediante Visual Studio.

[Actualización de aplicaciones con PowerShell](service-fabric-application-upgrade-tutorial-powershell.md) explica en detalle lo que tiene que hacer para actualizar una aplicación mediante PowerShell.

Puede controlar cómo se actualiza una aplicación usando [parámetros de actualización](service-fabric-application-upgrade-parameters.md).

Consiga que sus actualizaciones de aplicaciones sean compatibles aprendiendo a usar la [serialización de datos](service-fabric-application-upgrade-data-serialization.md).

Aprenda a usar funcionalidades avanzadas para actualizar una aplicación. Para ello, consulte los [temas avanzados](service-fabric-application-upgrade-advanced.md).

Solucione problemas habituales en las actualizaciones de aplicaciones consultando los pasos que figuran en [Solución de problemas de las actualizaciones de aplicaciones](service-fabric-application-upgrade-troubleshooting.md).

[image]: media/service-fabric-application-upgrade/service-fabric-application-upgrade-flowchart.png
