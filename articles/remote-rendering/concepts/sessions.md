---
title: Sesiones de Remote Rendering
description: En este artículo se escribe qué es una sesión de Remote Rendering.
author: jakrams
ms.author: jakras
ms.date: 02/21/2020
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.openlocfilehash: 74fae6a8aa0c59043db0ab816e09b16affb63580
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89021840"
---
# <a name="remote-rendering-sessions"></a>Sesiones de Remote Rendering

En Azure Remote Rendering (ARR), una *sesión* es un concepto clave. En este artículo se explica qué es exactamente una sesión.

## <a name="overview"></a>Información general

Azure Remote Rendering funciona mediante la descarga de tareas de representación complejas en la nube. Estas tareas de representación no las puede completar cualquier servidor, ya que la mayoría de los servidores en la nube no tienen GPU. Debido a la cantidad de datos implicados y al requisito estricto de producir resultados en velocidades de fotogramas interactivas, la responsabilidad del servidor que atiende la solicitud del usuario tampoco se puede trasladar a otra máquina sobre la marcha, como sí que puede ser posible en el tráfico web más común.

Esto significa que cuando se usa Azure Remote Rendering, debe reservarse exclusivamente un servidor en la nube con las funcionalidades de hardware necesarias para atender las solicitudes de representación. Una *sesión* hace referencia a todo lo relacionado con la interacción con este servidor. Comienza con la solicitud inicial para reservar (*concesión*) una máquina para su uso, continúa con todos los comandos para cargar y manipular modelos y finaliza con la liberación de la concesión en el servidor en la nube.

## <a name="managing-sessions"></a>Administración de sesiones

Hay varias maneras de administrar e interactuar con las sesiones. Con la [API REST de administración de sesiones](../how-tos/session-rest-api.md) se pueden crear, actualizar y finalizar sesiones sin tener que depender de un lenguaje. En C# y C++, estas operaciones se exponen a través de las clases `AzureFrontend` y `AzureSession`. En el caso de las aplicaciones de Unity, existen otras funciones de utilidad que proporciona el componente `ARRServiceUnity`.

Una vez que se haya *conectado* a una sesión activa, las operaciones como [cargar modelos](models.md) e interactuar con la escena se exponen a través de la clase `AzureSession`.

### <a name="managing-multiple-sessions-simultaneously"></a>Administración de varias sesiones simultáneamente

No es posible por completo *conectarse* a varias sesiones desde un dispositivo. Sin embargo, puede crear, observar y finalizar tantas sesiones como desee desde una sola aplicación. Siempre que la aplicación no esté diseñada para conectarse a una sesión, tampoco es necesario que se ejecute en un dispositivo como HoloLens 2. Un caso de uso para este tipo de implementación puede ser si desea controlar las sesiones a través de un mecanismo central. Por ejemplo, puede compilar una aplicación web, en la que varias tabletas y dispositivos HoloLens pueden iniciar sesión. A continuación, la aplicación puede mostrar opciones en las tabletas, como qué modelo CAD se va a mostrar. Si un usuario realiza una selección, esta información se comunica con todos los dispositivos HoloLens para crear una experiencia compartida.

## <a name="session-phases"></a>Fases de las sesiones

Cada sesión experimenta varias fases.

### <a name="session-startup"></a>Inicio de una sesión

Cuando solicita a ARR que [cree una sesión](../how-tos/session-rest-api.md#create-a-session), lo primero que hace es devolver un [UUID](https://en.wikipedia.org/wiki/Universally_unique_identifier) de la sesión. Este UUID permite consultar información sobre la sesión. El UUID y la información básica sobre la sesión se conservan durante 30 días, por lo que puede consultar esa información incluso después de que se haya finalizado la sesión. En este momento, el **estado de sesión** se notificará como **Iniciando**.

A continuación, Azure Remote Rendering intenta encontrar un servidor que puede hospedar la sesión. Hay dos parámetros para esta búsqueda. Primero, solo reservará servidores que se encuentren en su [región](../reference/regions.md). Esto se debe a que la latencia de red entre regiones puede ser demasiado alta para garantizar una experiencia decente. El segundo factor es el *tamaño* deseado que haya especificado. En cada región hay un número limitado de servidores que pueden completar la solicitud de tamaño [*Estándar*](../reference/vm-sizes.md) o [*Premium*](../reference/vm-sizes.md). Por lo tanto, si todos los servidores del tamaño solicitado están usándose actualmente en su región, la sesión no podrá crearse. El motivo del error [se puede consultar](../how-tos/session-rest-api.md#get-sessions-properties).

> [!IMPORTANT]
> Si solicita un tamaño de servidor *Estándar* y se produce un error en la solicitud debido a una demanda elevada, no implica que no se vaya a poder solicitar tampoco un servidor *Premium*. Por lo tanto, si es viable, puede intentar revertir a un tamaño de servidor *Premium*.

Cuando el servicio encuentra un servidor adecuado, tiene que copiar la máquina virtual adecuada en él para convertirlo en un host de Azure Remote Rendering. Este proceso tarda varios minutos. Después, se arranca la máquina virtual y el **estado de sesión** pasa a ser **Listo**.

En este punto, el servidor espera que sea el usuario quien lleve a cabo una acción. Este es también el momento en el que se le factura el servicio.

### <a name="connecting-to-a-session"></a>Conexión a un servidor

Una vez que la sesión está *lista*, puede *conectarse* a ella. Mientras está conectado, el dispositivo puede enviar comandos para cargar y modificar modelos. Cada host de ARR solo atiende un dispositivo cliente cada vez, por lo que cuando un cliente se conecta a una sesión, este tiene un control exclusivo del contenido representado. Eso también significa que el rendimiento de la representación nunca variará por motivos ajenos al control.

> [!IMPORTANT]
> Aunque solo un cliente puede *conectarse* a una sesión, se puede consultar información básica sobre las sesiones, como el estado actual, sin necesidad de establecer una conexión.

Mientras un dispositivo está conectado a una sesión, se producirá un error por los intentos de conexión de otros dispositivos. Sin embargo, una vez que el dispositivo conectado se desconecta, ya sea de forma voluntaria o debido a algún tipo de error, la sesión aceptará otra solicitud de conexión. Todos los estados anteriores (modelos cargados, etc.) se descartan, de modo que el siguiente dispositivo se conecta desde cero. Por lo tanto, las sesiones se pueden reutilizar muchas veces, y por dispositivos diferentes. Asimismo, en la mayoría de los casos es posible ocultar la sobrecarga de inicio de las sesiones del usuario final.

> [!IMPORTANT]
> El servidor remoto nunca altera el estado de los datos del lado cliente. La aplicación cliente debe realizar todas las mutaciones de datos (como las actualizaciones de transformación y las solicitudes de carga). Todas las acciones actualizan inmediatamente el estado de cliente.

### <a name="session-end"></a>Finalización de una sesión

Cuando se solicita una nueva sesión, se especifica un *tiempo máximo de concesión*, normalmente en el intervalo de una a ocho horas. Esta es la duración durante la cual el host aceptará que el usuario realice alguna acción.

Hay dos razones habituales por las que se finaliza una sesión. Puede solicitar manualmente que se finalice la sesión o que expire el tiempo máximo de concesión. En ambos casos, cualquier conexión activa al host se cierra inmediatamente y el servicio se detiene en ese servidor. A continuación, el servidor se devuelve al grupo de Azure y puede solicitarse para otros fines. No se puede deshacer ni cancelar una sesión. La consulta del **estado de sesión** de una sesión finalizada devolverá **Detenida** o **Expirada**, en función de si se finalizó manualmente o porque se alcanzó el tiempo máximo de concesión.

Una sesión también puede finalizarse debido a un error.

En todos los casos, no se le facturará más una vez que se finalice una sesión.

> [!WARNING]
> El hecho de conectarse a una sesión, y el tiempo que dure, no afecta a la facturación. Lo que se paga por el servicio depende de la *duración de la sesión*, es decir, el tiempo que un servidor se reserva exclusivamente y las funcionalidades de hardware solicitadas (el [tamaño asignado](../reference/vm-sizes.md)). Si inicia una sesión, conéctese durante cinco minutos y, a continuación, no cierre la sesión, de modo que siga ejecutándose hasta que expire su concesión. De este modo, se le facturará el tiempo de la concesión de la sesión completa. Por el contrario, el *tiempo máximo de concesión* es principalmente una medida de seguridad. No importa si solicita una sesión con un tiempo de la concesión de ocho horas y, después, la usa solo durante cinco, si se finaliza manualmente la sesión después.

#### <a name="extend-a-sessions-lease-time"></a>Ampliación del tiempo de la concesión de una sesión

Puede [ampliar el tiempo de la concesión](../how-tos/session-rest-api.md#modify-and-query-session-properties) de una sesión activa, en caso de que deje de ser necesaria.

## <a name="example-code"></a>Ejemplo de código

En el código siguiente se muestra una implementación simple del inicio de una sesión, de la espera al estado *Listo*, de la conexión y, a continuación, de la desconexión y la finalización.

```cs
RemoteRenderingInitialization init = new RemoteRenderingInitialization();
// fill out RemoteRenderingInitialization parameters...

RemoteManagerStatic.StartupRemoteRendering(init);

AzureFrontendAccountInfo accountInfo = new AzureFrontendAccountInfo();
// fill out accountInfo details...

AzureFrontend frontend = new AzureFrontend(accountInfo);

RenderingSessionCreationParams sessionCreationParams = new RenderingSessionCreationParams();
// fill out sessionCreationParams...

AzureSession session = await frontend.CreateNewRenderingSessionAsync(sessionCreationParams).AsTask();

RenderingSessionProperties sessionProperties;
while (true)
{
    sessionProperties = await session.GetPropertiesAsync().AsTask();
    if (sessionProperties.Status != RenderingSessionStatus.Starting &&
        sessionProperties.Status != RenderingSessionStatus.Unknown)
    {
        break;
    }
    // REST calls must not be issued too frequently, otherwise the server returns failure code 429 ("too many requests"). So we insert the recommended delay of 10s
    await Task.Delay(TimeSpan.FromSeconds(10));
}

if (sessionProperties.Status != RenderingSessionStatus.Ready)
{
    // Do some error handling and either terminate or retry.
}

// Connect to server
Result connectResult = await session.ConnectToRuntime(new ConnectToRuntimeParams()).AsTask();

// Connected!

while(...)
{
    // per frame update

    session.Actions.Update();
}

// Disconnect
session.DisconnectFromRuntime();

// stop the session
await session.StopAsync().AsTask();

// shut down the remote rendering SDK
RemoteManagerStatic.ShutdownRemoteRendering();
```

Se pueden mantener, manipular y consultar varias instancias de `AzureFrontend` y `AzureSession` desde el código. Sin embargo, solo un dispositivo puede conectarse a una sesión `AzureSession` a la vez.

La duración de una máquina virtual no está asociada a la instancia de `AzureFrontend` ni a la instancia de `AzureSession`. Se debe llamar a `AzureSession.StopAsync` para finalizar una sesión.

El identificador de sesión persistente se puede consultar a través de `AzureSession.SessionUUID()` y almacenarse en caché localmente. Con este identificador, una aplicación puede llamar a `AzureFrontend.OpenSession` para enlazarse a esa sesión.

Cuando el valor de `AzureSession.IsConnected` es true, `AzureSession.Actions` devuelve una instancia de `RemoteManager`, que contiene las funciones para [cargar modelos](models.md), manipular [entidades](entities.md) y [consultar información](../overview/features/spatial-queries.md) sobre la escena representada.

## <a name="next-steps"></a>Pasos siguientes

* [Entidades](entities.md)
* [Modelos](models.md)
