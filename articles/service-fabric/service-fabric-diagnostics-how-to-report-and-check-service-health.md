---
title: Notificación y comprobación del estado con Azure Service Fabric
description: Conozca cómo se pueden enviar informes de estado desde el código de servicio y comprobar el estado del servicio mediante las herramientas de supervisión de estado que proporciona Azure Service Fabric.
author: srrengar
ms.topic: conceptual
ms.date: 02/25/2019
ms.author: srrengar
ms.custom: devx-track-csharp
ms.openlocfilehash: 59c8202b03bf1be2be5a68b75a1d7c7404b2213d
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89020208"
---
# <a name="report-and-check-service-health"></a>Notificación y comprobación del estado del servicio
Cuando los servicios se encuentran con problemas, su capacidad para responder y corregir cualquier incidente e interrupción depende de la capacidad de detectar los problemas rápidamente. Si informa de problemas y errores en el administrador de estado de Azure Service Fabric desde el código de servicio, puede usar las herramientas estándar de seguimiento de estado que proporciona Service Fabric para comprobar el estado de mantenimiento.

Hay tres maneras de informar sobre el estado del servicio:

* Mediante los objetos [Partition](/dotnet/api/system.fabric.istatefulservicepartition) o [CodePackageActivationContext](/dotnet/api/system.fabric.codepackageactivationcontext).  
  Puede usar los objetos `Partition` y `CodePackageActivationContext` para informar sobre el estado de elementos que forman parte del contexto actual. Por ejemplo, el código que se ejecuta como parte de una réplica solo puede informar sobre el estado de esa réplica, la partición a la que pertenece y la aplicación de la que forma parte.
* Mediante `FabricClient`.   
  Puede usar `FabricClient` para informar sobre el estado del código de servicio si el clúster no es [seguro](service-fabric-cluster-security.md) o si el servicio se ejecuta con privilegios de administrador. La mayoría de los escenarios reales no usan clústeres no seguros ni proporcionan privilegios de administrador. Con `FabricClient`, puede notificar sobre el estado de cualquier entidad que forme parte del clúster. Sin embargo, lo ideal es que el código de servicio solo envíe informes relacionados con su propio estado.
* Use las API de REST en el nivel de clúster, aplicación, aplicación implementada, servicio, paquete de servicios, partición, réplica o nodo. Esto se puede usar para informar sobre el estado desde dentro de un contenedor.

Este artículo le guiará a través de un ejemplo que informa del estado del código de servicio. El ejemplo también muestra cómo se pueden usar las herramientas que ofrece Service Fabric para comprobar el estado de mantenimiento. Este artículo pretende ser una introducción rápida a las funcionalidades de supervisión del estado de Service Fabric. Para obtener más información, puede leer la serie de artículos detallados sobre el estado, empezando por el vínculo al final de este documento.

## <a name="prerequisites"></a>Prerrequisitos
Debe tener instalados los siguientes elementos:

* Visual Studio 2015 o Visual Studio 2019
* SDK de Service Fabric

## <a name="to-create-a-local-secure-dev-cluster"></a>Para crear un clúster de desarrollo seguro local
* Abra PowerShell con privilegios de administrador y ejecute los comandos siguientes:

![Comandos que muestran cómo crear un clúster de desarrollo seguro](./media/service-fabric-diagnostics-how-to-report-and-check-service-health/create-secure-dev-cluster.png)

## <a name="to-deploy-an-application-and-check-its-health"></a>Para implementar una aplicación y comprobar su estado
1. Abra Visual Studio como administrador.
1. Cree un proyecto mediante la plantilla **Servicio con estado** .
   
    ![Creación de una aplicación de Service Fabric con servicio con estado](./media/service-fabric-diagnostics-how-to-report-and-check-service-health/create-stateful-service-application-dialog.png)
1. Presione **F5** para ejecutar la aplicación en modo de depuración. La aplicación se implementa en el clúster local.
1. Después de ejecutarse la aplicación, haga clic con el botón derecho en el icono Administrador de clúster local en el área de notificación y seleccione **Administrar clúster local** en el menú de función rápida para abrir el explorador de Service Fabric.
   
    ![Apertura del explorador de Service Fabric desde el área de notificación](./media/service-fabric-diagnostics-how-to-report-and-check-service-health/LaunchSFX.png)
1. El estado de la aplicación debe aparecer como se muestra en esta imagen. En este momento, la aplicación debe ser correcta y no tener errores.
   
    ![Aplicación correcta en el explorador de Service Fabric](./media/service-fabric-diagnostics-how-to-report-and-check-service-health/sfx-healthy-app.png)
1. También puede comprobar el estado mediante PowerShell. Puede usar ```Get-ServiceFabricApplicationHealth``` y ```Get-ServiceFabricServiceHealth``` para comprobar el estado de una aplicación y un servicio, respectivamente. El informe de estado para la misma aplicación en PowerShell se muestra en esta imagen.
   
    ![Aplicación correcta en PowerShell](./media/service-fabric-diagnostics-how-to-report-and-check-service-health/ps-healthy-app-report.png)

## <a name="to-add-custom-health-events-to-your-service-code"></a>Para agregar eventos de estado personalizados a su código de servicio
Las plantillas de proyecto de Service Fabric en Visual Studio contienen código de ejemplo. Los pasos siguientes muestran cómo puede informar sobre eventos de estado personalizados desde el código de servicio. Estos informes aparecen automáticamente en las herramientas estándar para la supervisión del estado que proporciona Service Fabric, como Service Fabric Explorer, la vista del estado de Azure Portal y PowerShell.

1. Vuelva a abrir la aplicación que creó anteriormente en Visual Studio o cree otra con un servicio con estado mediante la plantilla de Visual Studio **Servicio con estado** .
1. Abra el archivo Stateful1.cs y busque la llamada `myDictionary.TryGetValueAsync` en el método `RunAsync`. Puede ver que este método devuelve un `result` que mantiene el valor actual del contador porque la lógica principal de esta aplicación es mantener el recuento en funcionamiento. Si se tratara de una aplicación real y si la falta de resultados representara un error, es posible que le interesara marcar ese evento.
1. Para informar sobre un evento de estado cuando la falta de resultados representa un error, agregue los siguientes pasos.
   
    a. Agregue este espacio de nombres `System.Fabric.Health` al archivo Stateful1.cs.
   
    ```csharp
    using System.Fabric.Health;
    ```
   
    b. Agregue el siguiente código después de la llamada `myDictionary.TryGetValueAsync` .
   
    ```csharp
    if (!result.HasValue)
    {
        HealthInformation healthInformation = new HealthInformation("ServiceCode", "StateDictionary", HealthState.Error);
        this.Partition.ReportReplicaHealth(healthInformation);
    }
    ```
    Informamos sobre el estado de la réplica porque se está informando desde un servicio con estado. El parámetro `HealthInformation` almacena información sobre el problema de estado del que se está informando.
   
    Si ha creado un servicio sin estado, use el código siguiente.
   
    ```csharp
    if (!result.HasValue)
    {
        HealthInformation healthInformation = new HealthInformation("ServiceCode", "StateDictionary", HealthState.Error);
        this.Partition.ReportInstanceHealth(healthInformation);
    }
    ```
1. Si el servicio se ejecuta con privilegios de administrador o si el clúster no es [seguro`FabricClient`, también puede usar ](service-fabric-cluster-security.md) para informar sobre el estado tal y como se muestra en los pasos siguientes.  
   
    a. Cree la instancia `FabricClient` después de la declaración `var myDictionary`.
   
    ```csharp
    var fabricClient = new FabricClient(new FabricClientSettings() { HealthReportSendInterval = TimeSpan.FromSeconds(0) });
    ```
   
    b. Agregue el siguiente código después de la llamada `myDictionary.TryGetValueAsync` .
   
    ```csharp
    if (!result.HasValue)
    {
       var replicaHealthReport = new StatefulServiceReplicaHealthReport(
            this.Context.PartitionId,
            this.Context.ReplicaId,
            new HealthInformation("ServiceCode", "StateDictionary", HealthState.Error));
        fabricClient.HealthManager.ReportHealth(replicaHealthReport);
    }
    ```
1. Simulemos este error y veamos cómo se muestra en las herramientas de seguimiento de estado. Para simular el error, comente la primera línea en el código de notificación de estado que agregó anteriormente. Después de comentar la primera línea, el código tendrá el aspecto del siguiente ejemplo.
   
    ```csharp
    //if(!result.HasValue)
    {
        HealthInformation healthInformation = new HealthInformation("ServiceCode", "StateDictionary", HealthState.Error);
        this.Partition.ReportReplicaHealth(healthInformation);
    }
    ```
   Este código activa el informe de estado cada vez que se ejecuta `RunAsync` . Después de realizar el cambio, presione **F5** para ejecutar la aplicación.
1. Después de que la aplicación se esté ejecutando, abra el explorador de Service Fabric para comprobar el estado de la aplicación. Esta vez, Service Fabric Explorer muestra que el estado de la aplicación no es correcto. La aplicación se muestra como no correcta debido al error que se notificó en el código que se agregó anteriormente.
   
    ![Aplicación no correcta en el explorador de Service Fabric](./media/service-fabric-diagnostics-how-to-report-and-check-service-health/sfx-unhealthy-app.png)
1. Si selecciona la réplica principal en la vista de árbol del explorador de Service Fabric, verá que **Health State** también indica un error. El explorador de Service Fabric también muestra los detalles del informe de estado que se agregaron al parámetro `HealthInformation` en el código. También puede ver los mismos informes de estado en PowerShell y en Azure Portal.
   
    ![Estado de réplica del explorador de Service Fabric](./media/service-fabric-diagnostics-how-to-report-and-check-service-health/replica-health-error-report-sfx.png)

Este informe permanece en el administrador de estado hasta que se sustituya por otro o se elimine esta réplica. Dado que no establecimos `TimeToLive` para este informe de estado en el objeto `HealthInformation`, el informe nunca expira.

Recomendamos notificar sobre el mantenimiento en el nivel más detallado, que en este caso es la réplica. También se puede informar sobre el estado en `Partition`.

```csharp
HealthInformation healthInformation = new HealthInformation("ServiceCode", "StateDictionary", HealthState.Error);
this.Partition.ReportPartitionHealth(healthInformation);
```

Para informar del estado en `Application`, `DeployedApplication` y `DeployedServicePackage`, use `CodePackageActivationContext`.

```csharp
HealthInformation healthInformation = new HealthInformation("ServiceCode", "StateDictionary", HealthState.Error);
var activationContext = FabricRuntime.GetActivationContext();
activationContext.ReportApplicationHealth(healthInformation);
```

## <a name="next-steps"></a>Pasos siguientes
* [Profundización en el estado de Service Fabric](service-fabric-health-introduction.md)
* [API de REST para informar el estado del servicio](/rest/api/servicefabric/report-the-health-of-a-service)
* [API de REST para informar el estado de la aplicación](/rest/api/servicefabric/report-the-health-of-an-application)
