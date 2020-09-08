---
title: Serialización de objetos de colección de confianza
description: Obtenga información sobre la serialización de objetos de colección de confianza de Azure Service Fabric, incluida la estrategia predeterminada y cómo definir la serialización personalizada.
ms.topic: conceptual
ms.date: 5/8/2017
ms.custom: devx-track-csharp
ms.openlocfilehash: 1d29f5d3391bdea0b21997a392d4d2e053b7ec65
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89018627"
---
# <a name="reliable-collection-object-serialization-in-azure-service-fabric"></a>Serialización de objetos de Reliable Collections en Azure Service Fabric
Reliable Collections replica y conserva sus elementos para garantizar su durabilidad en casos de errores de equipos e interrupciones del suministro eléctrico.
Tanto para la replicación como para la persistencia de elementos, Reliable Collections debe serializarlos.

Reliable Collections obtiene de Reliable State Manager el serializador apropiado para un tipo determinado.
Reliable State Manager contiene serializadores integrados y permite registrar serializadores personalizados para un tipo determinado.

## <a name="built-in-serializers"></a>Serializadores integrados

Reliable State Manager tiene serializadores integrados para algunos tipos comunes, a fin de poder serializarlos con eficacia de forma predeterminada. Para los demás tipos, Reliable State Manager recurre al uso de [DataContractSerializer](/dotnet/api/system.runtime.serialization.datacontractserializer?view=netcore-3.1).
Los serializadores integrados resultan más eficaces, ya que saben que sus tipos no pueden cambiar y no necesitan incluir información sobre el tipo, como el nombre de tipo.

Reliable State Manager tiene serializadores integrados para los siguientes tipos: 
- Guid
- bool
- byte
- sbyte
- byte[]
- char
- string
- Decimal
- double
- FLOAT
- int
- uint
- long
- ulong
- short
- ushort

## <a name="custom-serialization"></a>Serialización personalizada

Los serializadores personalizados se usan normalmente para aumentar el rendimiento o para cifrar los datos en la red y en disco. Entre otras razones, los serializadores personalizados suelen ser más eficaces que el serializador genérico, ya que no necesitan serializar la información sobre el tipo. 

[IReliableStateManager.TryAddStateSerializer\<T>](/dotnet/api/microsoft.servicefabric.data.ireliablestatemanager.tryaddstateserializer) se usa para registrar un serializador personalizado para un tipo determinado T. Este registro suele realizarse para la compilación de StatefulServiceBase, a fin de garantizar que, antes de que empiece la recuperación, todos los elementos de Reliable Collections puedan acceder al serializador pertinente para leer los datos persistentes.

```csharp
public StatefulBackendService(StatefulServiceContext context)
  : base(context)
  {
    if (!this.StateManager.TryAddStateSerializer(new OrderKeySerializer()))
    {
      throw new InvalidOperationException("Failed to set OrderKey custom serializer");
    }
  }
```

> [!NOTE]
> Los serializadores personalizados tienen prioridad sobre los serializadores integrados. Por ejemplo, cuando se registra un serializador personalizado para int, este se usa para serializar enteros en lugar del serializador integrado para int.

### <a name="how-to-implement-a-custom-serializer"></a>Cómo implementar un serializador personalizado

Un serializador personalizado debe implementar la interfaz [IStateSerializer\<T>](/dotnet/api/microsoft.servicefabric.data.istateserializer-1).

> [!NOTE]
> IStateSerializer\<T> incluye una sobrecarga para escritura y lectura que usa un valor base adicional denominado T. Esta API se usa para la serialización diferencial. Actualmente no se expone la característica de serialización diferencial. Por tanto, no se llama a estas dos sobrecargas hasta que se exponga y habilite la serialización diferencial.

A continuación se expone un ejemplo de tipo personalizado denominado OrderKey que contiene cuatro propiedades.

```csharp
public class OrderKey : IComparable<OrderKey>, IEquatable<OrderKey>
{
    public byte Warehouse { get; set; }

    public short District { get; set; }

    public int Customer { get; set; }

    public long Order { get; set; }

    #region Object Overrides for GetHashCode, CompareTo and Equals
    #endregion
}
```

A continuación se muestra una implementación de ejemplo de IStateSerializer\<OrderKey>.
Tenga en cuenta que las sobrecargas de lectura y escritura que usan baseValue llaman a su sobrecarga correspondiente para compatibilidad con versiones posteriores.

```csharp
public class OrderKeySerializer : IStateSerializer<OrderKey>
{
  OrderKey IStateSerializer<OrderKey>.Read(BinaryReader reader)
  {
      var value = new OrderKey();
      value.Warehouse = reader.ReadByte();
      value.District = reader.ReadInt16();
      value.Customer = reader.ReadInt32();
      value.Order = reader.ReadInt64();

      return value;
  }

  void IStateSerializer<OrderKey>.Write(OrderKey value, BinaryWriter writer)
  {
      writer.Write(value.Warehouse);
      writer.Write(value.District);
      writer.Write(value.Customer);
      writer.Write(value.Order);
  }
  
  // Read overload for differential de-serialization
  OrderKey IStateSerializer<OrderKey>.Read(OrderKey baseValue, BinaryReader reader)
  {
      return ((IStateSerializer<OrderKey>)this).Read(reader);
  }

  // Write overload for differential serialization
  void IStateSerializer<OrderKey>.Write(OrderKey baseValue, OrderKey newValue, BinaryWriter writer)
  {
      ((IStateSerializer<OrderKey>)this).Write(newValue, writer);
  }
}
```

## <a name="upgradability"></a>Capacidad de actualización
En una [actualización de la aplicación gradual](service-fabric-application-upgrade.md), la actualización se aplica a un subconjunto de nodos, un dominio de actualización a la vez. Durante este proceso, algunos dominios de actualización se incluirán en la versión más reciente de su aplicación, mientras que otros estarán en la versión anterior. Durante el lanzamiento, la nueva versión de la aplicación debe poder leer la versión anterior de sus datos, mientras que la versión anterior debe poder leer la nueva versión. Si el formato de datos no es compatible con las versiones anteriores y nuevas, es posible que se produzca un error en la actualización, o lo que es peor, que se pierdan o dañen datos.

Si usa el serializador integrado, no tiene que preocuparse por la cuestión de la compatibilidad.
No obstante, si usa un serializador personalizado o DataContractSerializer, los datos deben ser siempre compatibles con versiones anteriores y posteriores.
En otras palabras, todas las versiones del serializador deben tener la capacidad de serializar y deserializar cualquier versión del tipo.

Los usuarios con contrato de datos deben seguir reglas de versiones bien definidas para agregar, quitar y cambiar campos. El contrato de datos también tiene compatibilidad para tratar con campos desconocidos, enlazar con el proceso de serialización y deserialización, y para tratar con la herencia de clases. Para obtener más información, consulte [Uso de contrato de datos](/dotnet/framework/wcf/feature-details/using-data-contracts).

Los usuarios de serializadores personalizados deben atenerse a los criterios del serializador que usan, a fin de garantizar la compatibilidad con versiones anteriores y posteriores.
Una manera común de admitir todas las versiones es agregar información sobre el tamaño al principio y solo agregar propiedades opcionales.
De esta forma, cada versión puede leer todo lo posible y saltar por la parte restante de la secuencia.

## <a name="next-steps"></a>Pasos siguientes
  * [Serialización y actualización](service-fabric-application-upgrade-data-serialization.md)
  * [Referencia para desarrolladores de colecciones confiables](/dotnet/api/microsoft.servicefabric.data.collections?view=azure-dotnet#microsoft_servicefabric_data_collections)
  * [actualización de aplicaciones usando Visual Studio](service-fabric-application-upgrade-tutorial.md) ofrece información para actualizar una aplicación mediante Visual Studio.
  * [Actualización de aplicaciones de Service Fabric con PowerShell](service-fabric-application-upgrade-tutorial-powershell.md) se explica en detalle lo que tiene que hacer para actualizar una aplicación mediante PowerShell.
  * Puede controlar cómo se actualiza una aplicación usando [parámetros de actualización](service-fabric-application-upgrade-parameters.md).
  * Aprenda a usar funcionalidades avanzadas para actualizar una aplicación. Para ello, consulte los [temas avanzados](service-fabric-application-upgrade-advanced.md).
  * Solucione problemas habituales en las actualizaciones de aplicaciones consultando los pasos que figuran en [Solución de problemas de las actualizaciones de aplicaciones](service-fabric-application-upgrade-troubleshooting.md).
