---
title: Control de los eventos de ciclo de vida del servicio en la nube | Microsoft Docs
description: Obtenga información sobre cómo usar los métodos de ciclo de vida de un rol de servicio en la nube en .NET, incluido RoleEntryPoint, que proporciona métodos para responder a los eventos de ciclo de vida.
services: cloud-services
documentationcenter: .net
author: tgore03
ms.service: cloud-services
ms.custom: devx-track-csharp
ms.topic: article
ms.date: 07/18/2017
ms.author: tagore
ms.openlocfilehash: d64414abfbc62e52b172a2c42796ec8d89d1719f
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88930067"
---
# <a name="customize-the-lifecycle-of-a-web-or-worker-role-in-net"></a>Personalizar el ciclo de vida de un rol web o de trabajo en .NET
Cuando cree un rol de trabajo, amplíe la clase [RoleEntryPoint](/previous-versions/azure/reference/ee758619(v=azure.100)) que ofrece métodos para invalidar que le permiten responder a eventos del ciclo de vida. Para los roles web esta clase es opcional, por lo que debe usarla para responder a eventos del ciclo de vida.

## <a name="extend-the-roleentrypoint-class"></a>Extender la clase RoleEntryPoint
La clase [RoleEntryPoint](/previous-versions/azure/reference/ee758619(v=azure.100)) incluye métodos a los que llama Azure cuando se **inicia**, **se ejecuta** o **se detiene** un rol web o de trabajo. Opcionalmente, puede invalidar estos métodos para administrar la inicialización de roles, las secuencias de apagado de rol o el subproceso de ejecución del rol. 

Al extender **RoleEntryPoint**, debe tener en cuenta los siguientes comportamientos de los métodos:

* El método [OnStart](/previous-versions/azure/reference/ee772851(v=azure.100)) devuelve un valor booleano, por lo que es posible devolver **false** desde este método.
  
   Si el código devuelve **false**, el proceso del rol finaliza precipitadamente, sin ejecutar ninguna secuencia de apagado que pueda tener implantada. En general, debería evitar devolver **false** del método **OnStart**.
* Cualquier excepción no detectada en una sobrecarga de un método **RoleEntryPoint** se trata como una excepción no controlada.
  
   Si se produce una excepción dentro de uno de los métodos del ciclo de vida, Azure generará el evento [UnhandledException](/dotnet/api/system.appdomain.unhandledexception) y luego finalizará el proceso. Tras la desconexión del rol, se reiniciará Azure. Cuando se produzca una excepción no controlada, no se generará el evento [Detener](/previous-versions/azure/reference/ee758136(v=azure.100)) y no se llamará al método **OnStop** .

Si el rol no se inicia, o se recicla entre los estados de inicialización, ocupado y detención, el código puede generar una excepción no controlada en uno de los eventos de ciclo de vida cada vez que se reinicie el rol. En este caso, use el evento [UnhandledException](/dotnet/api/system.appdomain.unhandledexception) para determinar la causa de la excepción y controlarla de manera adecuada. Puede que su rol también se pueda devolver desde el método [Ejecutar](/previous-versions/azure/reference/ee772746(v=azure.100)) , que hace que se reinicie el rol. Para obtener más información acerca de los estados de implementación, vea [Problemas comunes que hacen que se reciclen los roles](cloud-services-troubleshoot-common-issues-which-cause-roles-recycle.md).

> [!NOTE]
> Si está usando la **Azure Tools para Microsoft Visual Studio** para desarrollar su aplicación, las plantillas de proyecto de rol extienden automáticamente la clase **RoleEntryPoint**, en los archivos *WebRole.cs* y *WorkerRole.cs*.
> 
> 

## <a name="onstart-method"></a>Método OnStart
Al método **OnStart** se llama cuando Azure pone en línea la instancia de rol. Mientras se ejecuta el código OnStart, la instancia de rol estará marcada como **ocupada** y el equilibrador de carga no le dirigirá ningún tráfico externo. Puede invalidar este método para realizar trabajo de inicialización, como la implementación de controladores de eventos el inicio de [Diagnósticos de Azure](cloud-services-how-to-monitor.md).

Si **OnStart** devuelve **true**, la instancia se inicializa correctamente y Azure llama al método **RoleEntryPoint.Run**. Si **OnStart** devuelve **false**, el rol finaliza de inmediato, sin ejecutar ninguna secuencia de apagado planeado.

En el ejemplo de código siguiente se muestra cómo invalidar el método **OnStart** . Este método configura e inicia un monitor de diagnóstico cuando se inicia la instancia de rol y se configura una transferencia de datos de registro a una cuenta de almacenamiento:

```csharp
public override bool OnStart()
{
    var config = DiagnosticMonitor.GetDefaultInitialConfiguration();

    config.DiagnosticInfrastructureLogs.ScheduledTransferLogLevelFilter = LogLevel.Error;
    config.DiagnosticInfrastructureLogs.ScheduledTransferPeriod = TimeSpan.FromMinutes(5);

    DiagnosticMonitor.Start("DiagnosticsConnectionString", config);

    return true;
}
```

## <a name="onstop-method"></a>Método OnStop
Al método **OnStop** se llama después de que Azure haya desconectado una instancia de rol Azure y antes de que salga el proceso. Puede invalidar este método para llamar al código necesario para que la instancia de rol se desconecte sin errores.

> [!IMPORTANT]
> El código que se ejecuta en el método **OnStop** tiene un tiempo limitado para finalizar cuando se llama por motivos distintos del apagado iniciado por un usuario. Después de que transcurra este tiempo, el proceso finaliza, por lo que debe asegurarse de que el código del método **OnStop** puede ejecutarse rápidamente o de que no tolera que no haya ninguna ejecución para finalizar. Al método **OnStop** se le llama después de que se genere el evento **Deteniendo**.
> 
> 

## <a name="run-method"></a>Método Run
Puede invalidar el método **Run** para implementar un subproceso de larga ejecución para su instancia de rol.

No es necesaria la invalidación del método **Run** ; la implementación predeterminada inicia un subproceso que se mantiene en suspensión indefinidamente. Si invalida el método **Run** , el código debe bloquearse indefinidamente. Si vuelve el método **Ejecutar**, el rol se recicla automáticamente sin problemas; es decir, Azure genera el evento **Detener** y llama al método **OnStop** para que las secuencias de apagado se puedan ejecutar antes de que el rol se quede sin conexión.

### <a name="implementing-the-aspnet-lifecycle-methods-for-a-web-role"></a>Implementación de los métodos del ciclo de vida de ASP.NET para un rol web
Puede usar los métodos del ciclo de vida de ASP.NET, además de los proporcionados por la clase **RoleEntryPoint** , para administrar secuencias de inicialización y apagado para un rol web. Esto puede ser útil para compatibilidad si va a portar una aplicación de ASP.NET existente a Azure. Se llama a los métodos de ciclo de vida de ASP.NET desde los métodos **RoleEntryPoint** . Al método **Application\_Start** se llama después de que finalice el método **RoleEntryPoint.OnStart**. Al método **Application\_End** se llama después de que se llame al método **RoleEntryPoint.OnStop**.

## <a name="next-steps"></a>Pasos siguientes
Aprenda cómo [crear un paquete de servicio en la nube](cloud-services-model-and-package.md).




