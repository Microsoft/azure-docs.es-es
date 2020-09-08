---
title: Configuración de Reliable Services en Azure Service Fabric
description: Conozca cómo configurar Reliable Services con estado en una aplicación de Azure Service Fabric de manera global y para un único servicio.
author: sumukhs
ms.topic: conceptual
ms.date: 10/02/2017
ms.author: sumukhs
ms.custom: devx-track-csharp
ms.openlocfilehash: cda0a9f988afae58a60bff051885a5eec8afe434
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89021976"
---
# <a name="configure-stateful-reliable-services"></a>Configurar Reliable Services con estado
Hay dos conjuntos de valores de configuración para los servicios de confianza. Un conjunto es global para todos los servicios de confianza del clúster, mientras que el otro conjunto es específico para un servicio de confianza determinado.

## <a name="global-configuration"></a>Configuración global
La configuración global de los servicios de confianza se especifica en el manifiesto de clúster para el clúster de la sección KtlLogger. Permite configurar el tamaño y la ubicación del registro compartido, así como los límites de memoria global que usa el registrador. El manifiesto de clúster es un único archivo XML que contiene los valores y las configuraciones que se aplican a todos los nodos y servicios del clúster. El archivo suele llamarse ClusterManifest.xml. Puede ver el manifiesto de clúster para su clúster usando el comando de PowerShell Get-ServiceFabricClusterManifest.

### <a name="configuration-names"></a>Nombres de la configuración
| Nombre | Unidad | Valor predeterminado | Observaciones |
| --- | --- | --- | --- |
| WriteBufferMemoryPoolMinimumInKB |Kilobytes |8388608 |Número mínimo de KB que se van a asignar en modo kernel al grupo de memoria del búfer de escritura del registrador. Este grupo de memoria se usa para almacenar en caché la información de estado antes de escribir en el disco. |
| WriteBufferMemoryPoolMaximumInKB |Kilobytes |Ilimitado |Tamaño máximo al que puede aumentar el grupo de memoria del búfer de escritura del registrador. |
| SharedLogId |GUID |"" |Especifica un GUID único que se usará para identificar el archivo de registro compartido predeterminado que usan todos los servicios de confianza en todos los nodos del clúster que no especifican SharedLogId en su configuración específica del servicio. Si se especifica SharedLogId, también se debe especificar SharedLogPath. |
| SharedLogPath |Nombre de ruta de acceso completo |"" |Especifica la ruta de acceso completa donde se encuentra el archivo de registro compartido que usan todos los servicios de confianza en todos los nodos del clúster que no especifican SharedLogPath en su configuración específica del servicio. Sin embargo, si se especifica SharedLogPath, también se debe especificar SharedLogId. |
| SharedLogSizeInMB |Megabytes |8192 |Especifica el número de MB de espacio en disco que se va a asignar estáticamente para el registro compartido. El valor deber ser 2048 o superior. |

En una plantilla de JSON o Azure ARM local, en el ejemplo siguiente se muestra cómo cambiar el registro de transacciones compartido que se crea para realizar copias de cualquier colección confiable de servicios con estado.

```json
"fabricSettings": [{
    "name": "KtlLogger",
    "parameters": [{
        "name": "SharedLogSizeInMB",
        "value": "4096"
    }]
}]
```

### <a name="sample-local-developer-cluster-manifest-section"></a>Sección de manifiesto de clúster del desarrollador local de ejemplo
Si desea cambiar este valor en su entorno de desarrollo local, debe editar el archivo clustermanifest.xml local.

```xml
   <Section Name="KtlLogger">
     <Parameter Name="SharedLogSizeInMB" Value="4096"/>
     <Parameter Name="WriteBufferMemoryPoolMinimumInKB" Value="8192" />
     <Parameter Name="WriteBufferMemoryPoolMaximumInKB" Value="8192" />
     <Parameter Name="SharedLogId" Value="{7668BB54-FE9C-48ed-81AC-FF89E60ED2EF}"/>
     <Parameter Name="SharedLogPath" Value="f:\SharedLog.Log"/>
   </Section>
```

### <a name="remarks"></a>Observaciones
El registrador tiene un grupo global de memoria asignado desde la memoria del kernel no paginada que está disponible para todos los servicios de confianza en un nodo para almacenar en caché los datos de estado antes de que se escriban en el registro específico asociado con la réplica del servicio de confianza. El tamaño del grupo se controla mediante las opciones WriteBufferMemoryPoolMinimumInKB y WriteBufferMemoryPoolMaximumInKB. WriteBufferMemoryPoolMinimumInKB especifica el tamaño inicial de este grupo de memoria y el tamaño mínimo al que se puede reducir el grupo de memoria. WriteBufferMemoryPoolMaximumInKB es el tamaño máximo que puede alcanzar el grupo de memoria. Cada réplica de un servicio de confianza que está abierta puede aumentar el tamaño del grupo de memoria en una cantidad que determina el sistema hasta WriteBufferMemoryPoolMaximumInKB. Si el grupo de memoria demanda más memoria de la que hay disponible, las solicitudes de memoria se retrasarán hasta que haya memoria disponible. Por lo tanto, si el grupo de memoria del búfer de escritura es demasiado pequeño para una configuración concreta, el rendimiento se puede ver afectado.

Los parámetros SharedLogId y SharedLogPath siempre se usan juntos para definir el GUID y la ubicación del registro compartido predeterminado de todos los nodos del clúster. El registro compartido predeterminado se usa para todos los servicios de confianza que no especifican los valores de configuración en el archivo settings.xml para el servicio específico. Para obtener un mejor rendimiento, los archivos de registro compartidos deben colocarse en discos que se usen únicamente para el archivo de registro compartido, de modo que se reduzca la contención.

SharedLogSizeInMB especifica la cantidad de espacio en disco que se va a preasignar para el registro compartido predeterminado en todos los nodos.  No es necesario especificar SharedLogId y SharedLogPath para poder especificar SharedLogSizeInMB.

## <a name="service-specific-configuration"></a>Configuración específica por servicio
Puede modificar la configuración predeterminada de Reliable Services con estado mediante el paquete de configuración (Config) o la implementación del servicio (código).

* **Config** : la configuración mediante el paquete de configuración se realiza cambiando el archivo Settings.xml generado en la raíz del paquete de Visual Studio, en la carpeta Config de cada servicio de la aplicación.
* **Código** : la configuración a través del código se consigue creando una clase ReliableStateManager con un objeto ReliableStateManagerConfiguration con el conjunto de opciones adecuado.

De forma predeterminada, el tiempo de ejecución de Azure Service Fabric busca los nombres de sección predefinidos en el archivo Settings.xml y usa los valores de configuración mientras crea los componentes en tiempo de ejecución subyacentes.

> [!NOTE]
> **No** elimine los nombres de sección de las siguientes configuraciones en el archivo Settings.xml que se genera en la solución de Visual Studio a menos que planee configurar el servicio mediante código.
> Cambiar los nombres del paquete de configuración o la sección requiere un cambio de código al configurar ReliableStateManager.
> 
> 

### <a name="replicator-security-configuration"></a>Configuración de seguridad del replicador
Las configuraciones de seguridad del replicador se utilizan para proteger el canal de comunicación que se usa durante la replicación. Esto significa que los servicios no podrán ver el tráfico de replicación del otro, lo que garantiza que los datos de alta disponibilidad también están seguros. De forma predeterminada, una sección de configuración de seguridad vacía impide la seguridad de la replicación.

> [!IMPORTANT]
> En los nodos Linux, los certificados deben tener el formato PEM. Para obtener más información sobre la ubicación y configuración de certificados para Linux, consulte [Configuración de certificados en Linux](./service-fabric-configure-certificates-linux.md). 
> 
> 

### <a name="default-section-name"></a>Nombre de sección predeterminado
ReplicatorSecurityConfig

> [!NOTE]
> Para cambiar este nombre de sección, reemplace el parámetro replicatorSecuritySectionName por el constructor ReliableStateManagerConfiguration al crear el ReliableStateManager para este servicio.
> 
> 

### <a name="replicator-configuration"></a>Configuración del replicador
Las configuraciones de replicador configuran el replicador responsable de hacer que el estado de Reliable Service con estado sea muy confiable replicando y conservando el estado localmente.
La configuración predeterminada es generada por la plantilla de Visual Studio y debe ser suficiente. En esta sección se habla sobre las configuraciones adicionales que están disponibles para optimizar el replicador.

### <a name="default-section-name"></a>Nombre de sección predeterminado
ReplicatorConfig

> [!NOTE]
> Para cambiar este nombre de sección, reemplace el parámetro replicatorSettingsSectionName por el constructor ReliableStateManagerConfiguration al crear el ReliableStateManager para este servicio.
> 
> 

### <a name="configuration-names"></a>Nombres de la configuración
| Nombre | Unidad | Valor predeterminado | Observaciones |
| --- | --- | --- | --- |
| BatchAcknowledgementInterval |Segundos |0.015 |Período de tiempo durante el que el replicador del secundario espera después de recibir una operación antes de enviar una confirmación al principal. El resto de confirmaciones que se enviarán para las operaciones que se procesan dentro de este intervalo se envían como una respuesta. |
| ReplicatorEndpoint |N/D |Ningún valor predeterminado: parámetro obligatorio |Dirección IP y puerto que usará el replicador principal y secundario para comunicarse con otros replicadores del conjunto de réplicas. Esto debe hacer referencia a un punto de conexión de recursos de TCP en el manifiesto de servicio. Consulte [Especificación de los recursos en un manifiesto de servicio](service-fabric-service-manifest-resources.md) para obtener más información sobre cómo definir los recursos de punto de conexión en un manifiesto de servicio. |
| MaxPrimaryReplicationQueueSize |Número de operaciones |8192 |Número máximo de operaciones de la cola principal. Una operación se libera después de que el replicador principal reciba una confirmación de todos los replicadores secundarios. Este valor debe ser mayor que 64 y una potencia de 2. |
| MaxSecondaryReplicationQueueSize |Número de operaciones |16384 |Número máximo de operaciones de la cola secundaria. Una operación se libera después de que su estado pase a ser de alta disponibilidad mediante persistencia. Este valor debe ser mayor que 64 y una potencia de 2. |
| CheckpointThresholdInMB |MB |50 |Cantidad del espacio del archivo de registro después de que se compruebe el estado. |
| MaxRecordSizeInKB |KB |1024 |Tamaño del registro de mayor tamaño el replicador que puede escribir en el registro. Este valor debe ser un múltiplo de 4 y superior a 16. |
| MinLogSizeInMB |MB |0 (sistema determinado) |Tamaño mínimo del registro transaccional. El registro no podrá truncarse a un tamaño inferior a este valor. 0 indica que el replicador determinará el tamaño mínimo del registro. Al aumentar este valor, incrementa la posibilidad de hacer copias parciales y copias de seguridad incrementales, ya que se reducen las posibilidades de que se trunquen registros pertinentes. |
| TruncationThresholdFactor |Factor |2 |Determina a qué tamaño del registro se activará el truncamiento. El umbral de truncamiento se determina multiplicando el valor de MinLogSizeInMB por TruncationThresholdFactor. El valor de TruncationThresholdFactor debe ser mayor que 1. El valor de MinLogSizeInMB * TruncationThresholdFactor debe ser menor que MaxStreamSizeInMB. |
| ThrottlingThresholdFactor |Factor |4 |Determina a qué tamaño del registro se empezará a limitar la réplica. El umbral de limitación (en MB) se determina mediante Max((MinLogSizeInMB ThrottlingThresholdFactor),(CheckpointThresholdInMB ThrottlingThresholdFactor)). El valor del umbral de limitación (en MB) debe ser mayor que el de truncamiento (en MB). El valor del umbral de truncamiento (en MB) debe ser menor que el de MaxStreamSizeInMB. |
| MaxAccumulatedBackupLogSizeInMB |MB |800 |El tamaño máximo acumulado (en MB) de los registros de copia de seguridad de una cadena de registros de copia de seguridad determinada. Las solicitudes de copia de seguridad incremental no se realizarán correctamente si esta genera un registro de copia de seguridad que provoque la acumulación de este tipo de elemento, ya que la copia de seguridad completa tendrá un tamaño superior. En tales casos, el usuario tiene que realizar una copia de seguridad completa. |
| SharedLogId |GUID |"" |Especifica un GUID único que debe usarse para identificar el archivo de registro compartido que se usa con esta réplica. Normalmente, los servicios no deben usar esta opción de configuración. Sin embargo, si se especifica SharedLogId, también se debe especificar SharedLogPath. |
| SharedLogPath |Nombre de ruta de acceso completo |"" |Especifica la ruta de acceso completa donde se creará el archivo de registro compartido para esta réplica. Normalmente, los servicios no deben usar esta opción de configuración. Sin embargo, si se especifica SharedLogPath, también se debe especificar SharedLogId. |
| SlowApiMonitoringDuration |Segundos |300 |Establece el intervalo de supervisión para las llamadas API administradas. Ejemplo: función de devolución de llamada de copia de seguridad proporcionada por el usuario. Una vez transcurrido el intervalo, se enviará un informe de mantenimiento de advertencias a Health Manager. |
| LogTruncationIntervalSeconds |Segundos |0 |Intervalo configurable en el que se iniciará el truncamiento del registro en cada réplica. Sirve para garantizar que el registro también se trunca según el tiempo en lugar de simplemente según el tamaño del registro. Esta opción obliga también a purgar las entradas eliminadas en un diccionario de confianza. Por tanto, se puede usar para asegurarse de que se purgan los elementos eliminados de manera oportuna. |
| EnableStableReads |Boolean |False |La habilitación de las lecturas estables restringe las réplicas secundarias para devolver valores con cuórum confirmado. |

### <a name="sample-configuration-via-code"></a>Ejemplo de configuración mediante código
```csharp
class Program
{
    /// <summary>
    /// This is the entry point of the service host process.
    /// </summary>
    static void Main()
    {
        ServiceRuntime.RegisterServiceAsync("HelloWorldStatefulType",
            context => new HelloWorldStateful(context, 
                new ReliableStateManager(context, 
        new ReliableStateManagerConfiguration(
                        new ReliableStateManagerReplicatorSettings()
            {
                RetryInterval = TimeSpan.FromSeconds(3)
                        }
            )))).GetAwaiter().GetResult();
    }
}    
```
```csharp
class MyStatefulService : StatefulService
{
    public MyStatefulService(StatefulServiceContext context, IReliableStateManagerReplica stateManager)
        : base(context, stateManager)
    { }
    ...
}
```


### <a name="sample-configuration-file"></a>Archivo de configuración de ejemplo
```xml
<?xml version="1.0" encoding="utf-8"?>
<Settings xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/2011/01/fabric">
   <Section Name="ReplicatorConfig">
      <Parameter Name="ReplicatorEndpoint" Value="ReplicatorEndpoint" />
      <Parameter Name="BatchAcknowledgementInterval" Value="0.05"/>
      <Parameter Name="CheckpointThresholdInMB" Value="512" />
   </Section>
   <Section Name="ReplicatorSecurityConfig">
      <Parameter Name="CredentialType" Value="X509" />
      <Parameter Name="FindType" Value="FindByThumbprint" />
      <Parameter Name="FindValue" Value="9d c9 06 b1 69 dc 4f af fd 16 97 ac 78 1e 80 67 90 74 9d 2f" />
      <Parameter Name="StoreLocation" Value="LocalMachine" />
      <Parameter Name="StoreName" Value="My" />
      <Parameter Name="ProtectionLevel" Value="EncryptAndSign" />
      <Parameter Name="AllowedCommonNames" Value="My-Test-SAN1-Alice,My-Test-SAN1-Bob" />
   </Section>
</Settings>
```


### <a name="remarks"></a>Observaciones
BatchAcknowledgementInterval controla la latencia de replicación. Un valor de "0" ofrecerá la menor latencia posible, a costa del rendimiento (como deben enviarse y procesarse más mensajes de confirmación, cada uno con menos confirmaciones).
Cuanto mayor sea el valor de BatchAcknowledgementInterval, mayor será el rendimiento general de la replicación a costa de una mayor latencia de la operación. Esto se traduce directamente en la latencia de transacciones confirmadas.

El valor de CheckpointThresholdInMB controla la cantidad de espacio en disco que el replicador puede usar para almacenar información de estado en el archivo de registro específico de la réplica. Aumentar este valor a un valor mayor que el predeterminado puede acelerar los tiempos de reconfiguración cuando se agrega una nueva réplica al conjunto. Esto se debe a la transferencia de estado parcial que tiene lugar debido a la disponibilidad de mayor cantidad de historial de operaciones en el registro. Esto podría aumentar el tiempo de recuperación de una réplica después de un error.

El parámetro MaxRecordSizeInKB define el tamaño máximo de un registro que el replicador puede escribir en el archivo de registro. En la mayoría de los casos, el tamaño predeterminado de 1024 KB del registro es óptimo. Sin embargo, si el servicio hace que elementos de datos de mayor tamaño formen parte de la información de estado, es posible que este valor se tenga que aumentar. Cambiar MaxRecordSizeInKB para que tenga un tamaño inferior a 1024 presenta pocas ventajas, ya que los registros más pequeños solamente usan el espacio necesario para el registro más pequeño. Se espera que este valor solo tenga que cambiarse en raras ocasiones.

Los parámetros SharedLogId y SharedLogPath siempre se usan en conjunto para obligar a un servicio a usar un registro compartido independiente del registro compartido predeterminado del nodo. Para obtener una mayor eficacia, todos los servicios posibles deben especificar el mismo registro compartido. Para reducir la contención del movimiento de encabezados, los archivos de registro compartidos deben colocarse en discos que se usen únicamente para el archivo de registro compartido. Se espera que este valor solo tenga que cambiarse en raras ocasiones.

## <a name="next-steps"></a>Pasos siguientes
* [Depuración de la aplicación del Service Fabric en Visual Studio](service-fabric-debugging-your-application.md)
* [Referencia para desarrolladores de servicios confiables](/previous-versions/azure/dn706529(v=azure.100))
