---
title: Notificaciones de Reliable Services
description: Documentación conceptual para las notificaciones de Reliable Services de Service Fabric para el administrador de estado y el diccionario de confianza
author: mcoskun
ms.topic: conceptual
ms.date: 6/29/2017
ms.author: mcoskun
ms.custom: devx-track-csharp
ms.openlocfilehash: 4a336daf9bd7400d049233a22a04d64d561b42c9
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89021959"
---
# <a name="reliable-services-notifications"></a>Notificaciones de Reliable Services
Las notificaciones permiten que los clientes sigan los cambios que se están realizando en un objeto que les interesa. Dos tipos de objetos admiten notificaciones: *Reliable State Manager* y *Reliable Dictionary*.

Algunas causas habituales del uso de notificaciones son:

* Creación de vistas materializadas, como índices secundarios o vistas filtradas agregadas del estado de la réplica. Un ejemplo es un índice ordenado de todas las claves en un diccionario Reliable Dictionary.
* Envío de datos de supervisión, como el número de usuarios agregados en la última hora.

Las notificaciones se desencadenan como parte de la aplicación de operaciones. Por ese motivo, se deben controlar las notificaciones lo más rápido posible y los eventos sincrónicos no deben incluir operaciones costosas.

## <a name="reliable-state-manager-notifications"></a>Notificaciones de Reliable State Manager
Reliable State Manager proporciona notificaciones para los siguientes eventos:

* Transacción
  * Commit
* Administrador de estado
  * Volver a generar
  * Adición de un estado Reliable State
  * Eliminación de un estado Reliable State

Reliable State Manager realiza un seguimiento de las transacciones en proceso actuales. El único cambio en el estado de transacción que hace que se desencadene una notificación es la confirmación de una transacción.

Reliable State Manager mantiene una colección de estados Reliable State, como Reliable Dictionary y Reliable Queue. Reliable State Manager desencadena notificaciones cuando esta colección cambia: se agrega o quita un estado Reliable State o se recompila la colección completa.
La colección de Reliable State Manager se recompila en tres situaciones:

* Recuperación: cuando se inicia una réplica, recupera su estado anterior del disco. Al final de la recuperación, usa **NotifyStateManagerChangedEventArgs** para desencadenar un evento que contiene el conjunto de estados Reliable State recuperados.
* Copia completa: para que una réplica pueda unirse al conjunto de configuración, tiene que compilarse. En ocasiones, se requiere que se aplique una copia completa del estado de Reliable State Manager en la réplica principal a la réplica secundaria inactiva. En la réplica secundaria, Reliable State Manager usa **NotifyStateManagerChangedEventArgs** para desencadenar un evento que contiene el conjunto de estados Reliable State que adquirió de la réplica principal.
* Restauración: en escenarios de recuperación ante desastres, se puede restaurar el estado de la réplica desde una copia de seguridad con **RestoreAsync**. En estos casos, en la réplica principal, Reliable State Manager usa **NotifyStateManagerChangedEventArgs** para desencadenar un evento que contiene el conjunto de estados Reliable State que restauró de la copia de seguridad.

Para registrarse para las notificaciones de transacciones o notificaciones del administrador de estado, se debe registrar con los eventos **TransactionChanged** o **StateManagerChanged** en Reliable State Manager. Un lugar habitual donde registrarse con estos controladores de eventos es el constructor de su servicio con estado. Cuando se registre en el constructor, no perderá ninguna notificación causada por un cambio mientras dure **IReliableStateManager**.

```csharp
public MyService(StatefulServiceContext context)
    : base(MyService.EndpointName, context, CreateReliableStateManager(context))
{
    this.StateManager.TransactionChanged += this.OnTransactionChangedHandler;
    this.StateManager.StateManagerChanged += this.OnStateManagerChangedHandler;
}
```

El controlador de eventos **TransactionChanged** usa **NotifyTransactionChangedEventArgs** para proporcionar detalles sobre el evento. Contiene la propiedad de acción (por ejemplo, **NotifyTransactionChangedAction.Commit**) que especifica el tipo de cambio. También contiene la propiedad de transacción que proporciona una referencia a la transacción que ha cambiado.

> [!NOTE]
> En la actualidad, los eventos **TransactionChanged** solo se generan si se confirma la transacción. Entonces, la acción equivale a **NotifyTransactionChangedAction.Commit**. Sin embargo, es posible que en el futuro se generen eventos para otros tipos de cambios en el estado de la transacción. Se recomienda comprobar la acción y solo procesar el evento si es el que espera.
> 
> 

A continuación, se ofrece un ejemplo de controlador de eventos **TransactionChanged** .

```csharp
private void OnTransactionChangedHandler(object sender, NotifyTransactionChangedEventArgs e)
{
    if (e.Action == NotifyTransactionChangedAction.Commit)
    {
        this.lastCommitLsn = e.Transaction.CommitSequenceNumber;
        this.lastTransactionId = e.Transaction.TransactionId;

        this.lastCommittedTransactionList.Add(e.Transaction.TransactionId);
    }
}
```

El controlador de eventos **StateManagerChanged** usa **NotifyStateManagerChangedEventArgs** para proporcionar detalles sobre el evento.
**NotifyStateManagerChangedEventArgs** tiene dos subclases: **NotifyStateManagerRebuildEventArgs** y **NotifyStateManagerSingleEntityChangedEventArgs**.
Use la propiedad de acción en **NotifyStateManagerChangedEventArgs** para convertir la subclase **NotifyStateManagerChangedEventArgs** en la correcta:

* **NotifyStateManagerChangedAction.Rebuild**: **NotifyStateManagerRebuildEventArgs**
* **NotifyStateManagerChangedAction.Add** y **NotifyStateManagerChangedAction.Remove**: **NotifyStateManagerSingleEntityChangedEventArgs**

A continuación, se ofrece un ejemplo de controlador de notificación **StateManagerChanged** .

```csharp
public void OnStateManagerChangedHandler(object sender, NotifyStateManagerChangedEventArgs e)
{
    if (e.Action == NotifyStateManagerChangedAction.Rebuild)
    {
        this.ProcessStateManagerRebuildNotification(e);

        return;
    }

    this.ProcessStateManagerSingleEntityNotification(e);
}
```

## <a name="reliable-dictionary-notifications"></a>Notificaciones de Reliable Dictionary
Reliable Dictionary proporciona notificaciones para los siguientes eventos:

* Recompilación: se llama cuando **ReliableDictionary** ha recuperado su estado de una copia de seguridad o un estado local copiado o recuperado.
* Clear: se llama cuando se ha borrado el estado de **ReliableDictionary** mediante el método **ClearAsync**.
* Agregue: se llama cuando se ha agregado un elemento a **ReliableDictionary**.
* Actualización: se llama cuando se ha actualizado un elemento de **IReliableDictionary**.
* Remove: se llama cuando se ha eliminado un elemento de **IReliableDictionary**.

Para obtener notificaciones de Reliable Dictionary, se debe registrar con el controlador de eventos **DictionaryChanged** en **IReliableDictionary**. Un lugar habitual donde registrarse con estos controladores de eventos es en la notificación de adición de **ReliableStateManager.StateManagerChanged** .
Registrarse cuando se agrega **IReliableDictionary** a **IReliableStateManager** garantiza que no se pierda ninguna notificación.

```csharp
private void ProcessStateManagerSingleEntityNotification(NotifyStateManagerChangedEventArgs e)
{
    var operation = e as NotifyStateManagerSingleEntityChangedEventArgs;

    if (operation.Action == NotifyStateManagerChangedAction.Add)
    {
        if (operation.ReliableState is IReliableDictionary<TKey, TValue>)
        {
            var dictionary = (IReliableDictionary<TKey, TValue>)operation.ReliableState;
            dictionary.RebuildNotificationAsyncCallback = this.OnDictionaryRebuildNotificationHandlerAsync;
            dictionary.DictionaryChanged += this.OnDictionaryChangedHandler;
        }
    }
}
```

> [!NOTE]
> **ProcessStateManagerSingleEntityNotification** es el método de ejemplo al que se llama en el ejemplo anterior de **OnStateManagerChangedHandler**.
> 
> 

El código anterior establece la interfaz **IReliableNotificationAsyncCallback**, junto con **DictionaryChanged**. Dado que **NotifyDictionaryRebuildEventArgs** contiene una interfaz **IAsyncEnumerable**, que debe enumerarse de forma asincrónica, las notificaciones de recompilación se desencadenan mediante **RebuildNotificationAsyncCallback** en lugar de **OnDictionaryChangedHandler**.

```csharp
public async Task OnDictionaryRebuildNotificationHandlerAsync(
    IReliableDictionary<TKey, TValue> origin,
    NotifyDictionaryRebuildEventArgs<TKey, TValue> rebuildNotification)
{
    this.secondaryIndex.Clear();

    var enumerator = e.State.GetAsyncEnumerator();
    while (await enumerator.MoveNextAsync(CancellationToken.None))
    {
        this.secondaryIndex.Add(enumerator.Current.Key, enumerator.Current.Value);
    }
}
```

> [!NOTE]
> En el código anterior, como parte del procesamiento de la notificación de recompilación, primero se borra el estado agregado mantenido. Como se está recompilando la colección Reliable Collection con un estado nuevo, todas las notificaciones anteriores son irrelevantes.
> 
> 

El controlador de eventos **DictionaryChanged** usa **NotifyDictionaryChangedEventArgs** para proporcionar detalles sobre el evento.
**NotifyDictionaryChangedEventArgs** tiene cinco subclases. Use la propiedad de acción en **NotifyDictionaryChangedEventArgs** para convertir la subclase **NotifyDictionaryChangedEventArgs** en la correcta:

* **NotifyDictionaryChangedAction.Rebuild**: **NotifyDictionaryRebuildEventArgs**
* **NotifyDictionaryChangedAction.Clear**: **NotifyDictionaryClearEventArgs**
* **NotifyDictionaryChangedAction.Add**: **NotifyDictionaryItemAddedEventArgs**
* **NotifyDictionaryChangedAction.Update**: **NotifyDictionaryItemUpdatedEventArgs**
* **NotifyDictionaryChangedAction.Remove**: **NotifyDictionaryItemRemovedEventArgs**

```csharp
public void OnDictionaryChangedHandler(object sender, NotifyDictionaryChangedEventArgs<TKey, TValue> e)
{
    switch (e.Action)
    {
        case NotifyDictionaryChangedAction.Clear:
            var clearEvent = e as NotifyDictionaryClearEventArgs<TKey, TValue>;
            this.ProcessClearNotification(clearEvent);
            return;

        case NotifyDictionaryChangedAction.Add:
            var addEvent = e as NotifyDictionaryItemAddedEventArgs<TKey, TValue>;
            this.ProcessAddNotification(addEvent);
            return;

        case NotifyDictionaryChangedAction.Update:
            var updateEvent = e as NotifyDictionaryItemUpdatedEventArgs<TKey, TValue>;
            this.ProcessUpdateNotification(updateEvent);
            return;

        case NotifyDictionaryChangedAction.Remove:
            var deleteEvent = e as NotifyDictionaryItemRemovedEventArgs<TKey, TValue>;
            this.ProcessRemoveNotification(deleteEvent);
            return;

        default:
            break;
    }
}
```

## <a name="recommendations"></a>Recomendaciones
* *Complete* los eventos de notificación tan rápido como sea posible.
* *No ejecute* ninguna operación costosa (por ejemplo, operaciones de E/S) como parte de eventos sincrónicos.
* *Compruebe* el tipo de acción antes de procesar el evento. Es posible que se agreguen nuevos tipos de acción en el futuro.

Algunos aspectos que debe tener en cuenta:

* Las notificaciones se desencadenan como parte de la ejecución de una operación. Por ejemplo, se desencadena una notificación de restauración como el último paso de una operación de restauración. La restauración no finalizará hasta que se procese el evento de notificación.
* Ya que las notificaciones se desencadenan como parte de la aplicación de operaciones, los clientes solo ven notificaciones para las operaciones confirmadas localmente. Y como solo se garantiza que las operaciones se confirmen localmente (es decir, se registren), es posible que en el futuro se deshagan o no.
* En la ruta de acceso de puesta al día, se desencadena una única notificación para cada operación aplicada. Esto significa que, si una transacción T1 incluye Create(X), Delete(X) y Create(X), recibirá una notificación para la creación de X, una para la eliminación y otra para la creación de nuevo, en ese orden.
* Para las transacciones que contienen varias operaciones, se aplican las operaciones en el orden en que se recibieron en la réplica principal del usuario.
* Como parte del procesamiento de progreso falso, es posible que algunas operaciones se deshagan. Se generan notificaciones para estas operaciones de deshacer y se revierte el estado de la réplica a un punto estable. Una diferencia importante de las notificaciones de deshacer es que se agregan eventos con claves duplicadas. Por ejemplo, si se está deshaciendo la transacción T1, el usuario verá una única notificación para Delete(X).

## <a name="next-steps"></a>Pasos siguientes
* [Colecciones confiables](service-fabric-work-with-reliable-collections.md)
* [Introducción a Reliable Services de Service Fabric de Microsoft Azure](service-fabric-reliable-services-quick-start.md)
* [Copia de seguridad y restauración de Reliable Services (recuperación ante desastres)](service-fabric-reliable-services-backup-restore.md)
* [Referencia para desarrolladores de colecciones confiables](/dotnet/api/microsoft.servicefabric.data.collections?view=azure-dotnet#microsoft_servicefabric_data_collections)
