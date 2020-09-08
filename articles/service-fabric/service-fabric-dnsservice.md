---
title: Servicio DNS en Azure Service Fabric
description: Use el servicio DNS de Service Fabric para detectar microservicios desde dentro del clúster.
ms.topic: conceptual
ms.date: 7/20/2018
ms.custom: devx-track-csharp
ms.openlocfilehash: a05669bbd6de44447d7eb11a0b9941d18e8048d1
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89021279"
---
# <a name="dns-service-in-azure-service-fabric"></a>Servicio DNS en Azure Service Fabric
El servicio DNS es un servicio de sistema opcional que se puede habilitar en el clúster para detectar otros servicios que usan el protocolo DNS. 

Muchos servicios, especialmente los servicios de contenedor, son direccionables a través de una dirección URL ya existente. Resulta conveniente tener la capacidad de resolver estos servicios mediante el protocolo de DNS estándar, en lugar de con el protocolo del Servicio de nombres de Service Fabric. El servicio DNS permite asignar nombres DNS a un nombre de servicio y, por tanto, resolver direcciones IP del punto de conexión. Esta funcionalidad mantiene la portabilidad de los servicios de contenedor en diferentes plataformas y puede facilitar escenarios de "Levantar y mover", ya que permite usar las direcciones URL de servicio existentes en lugar de tener que volver a escribir el código para aprovechar el servicio de nombres. 

El servicio DNS asigna nombres DNS a nombres de servicio que el protocolo Servicio de nombres resuelve para devolver el punto de conexión del servicio. El nombre DNS para el servicio se proporciona en el momento de la creación. En el diagrama siguiente se muestra cómo funciona el servicio DNS para servicios sin estado.

![puntos de conexión del servicio](./media/service-fabric-dnsservice/stateless-dns.png)

A partir de la versión 6.3 de Service Fabric, el protocolo de DNS de Service Fabric se ha ampliado para incluir un esquema de direccionamiento de servicios con estado particionados. Estas extensiones permiten resolver direcciones IP de partición específicas mediante una combinación de nombre DNS del servicio con estado y el nombre de partición. Se admiten los tres esquemas de partición:

- Particiones con nombre
- Particiones por rangos
- Particiones de singleton

En el diagrama siguiente se muestra cómo funciona el servicio DNS para servicios con estado particionados.

![puntos de conexión de servicio con estado](./media/service-fabric-dnsservice/stateful-dns.png)

El servicio DNS no admite puertos dinámicos. Para resolver servicios expuestos en puertos dinámicos, use el [servicio de proxy inverso](./service-fabric-reverseproxy.md).

## <a name="enabling-the-dns-service"></a>Habilitar el servicio DNS
> [!NOTE]
> El servicio DNS para servicios de Service Fabric no se admite todavía en Linux.

Cuando se crea un clúster mediante el portal, el servicio DNS está activado de manera predeterminada en la casilla **Incluir servicio DNS** en el menú **Configuración del clúster**:

![Habilitación del servicio DNS a través del portal](./media/service-fabric-dnsservice/enable-dns-service.png)

Si no usa el portal para crear el clúster o va a actualizar un clúster existente, tendrá que habilitar el servicio DNS en una plantilla:

- Para implementar un nuevo clúster, puede usar las [plantillas de ejemplo](https://github.com/Azure/azure-quickstart-templates/tree/master/service-fabric-secure-cluster-5-node-1-nodetype) o crear su propia plantilla de Resource Manager. 
- Para actualizar un clúster existente, navegue al grupo de recursos del clúster en el portal y haga clic en **Script de Automation** para trabajar con una plantilla que refleje el estado actual del clúster y otros recursos del grupo. Para obtener más información, vea [Exportación de la plantilla desde el grupo de recursos](../azure-resource-manager/templates/export-template-portal.md).

Siga estos pasos para habilitar el servicio DNS:

1. Compruebe que la `apiversion` está establecida en `2017-07-01-preview` o una versión posterior para el recurso `Microsoft.ServiceFabric/clusters` y, si no es así, actualícela tal y como se muestra en el siguiente ejemplo:

    ```json
    {
        "apiVersion": "2017-07-01-preview",
        "type": "Microsoft.ServiceFabric/clusters",
        "name": "[parameters('clusterName')]",
        "location": "[parameters('clusterLocation')]",
        ...
    }
    ```

2. Ahora, habilite el servicio DNS de alguna de las maneras siguientes:

   - Para habilitar el servicio DNS con la configuración predeterminada, agréguelo a la sección `addonFeatures` dentro de la sección `properties`, tal como se muestra en el ejemplo siguiente:

        ```json
          "properties": {
            ...
            "addonFeatures": [
              "DnsService"
              ],
            ...
          }
        ```

   - Para habilitar el servicio con otra configuración que no sea la predeterminada, agregue una sección `DnsService` a la sección `fabricSettings` dentro de la sección `properties`. En este caso, no es necesario agregar DnsService a `addonFeatures`. Para más información sobre las propiedades que se pueden establecer para el servicio DNS, vea la [configuración del servicio DNS](./service-fabric-cluster-fabric-settings.md#dnsservice).

       ```json
           "properties": {
             ...  
             "fabricSettings": [
               ...
               {
                 "name": "DnsService",
                 "parameters": [
                   {
                     "name": "IsEnabled",
                     "value": "true"
                   },
                   {
                     "name": "PartitionSuffix",
                     "value": "--"
                   },
                   {
                     "name": "PartitionPrefix",
                     "value": "--"
                   }
                 ]
               },
               ...
              ]
            }
       ```
3. Una vez actualizada la plantilla del clúster con sus cambios, aplíquelos para completar la actualización. Cuando se lleva a cabo la actualización, el servicio del sistema DNS comienza a ejecutarse en el clúster. El nombre del servicio es `fabric:/System/DnsService`, que puede encontrar en la sección de servicio del **sistema** del Service Fabric Explorer. 

> [!NOTE]
> Al actualizar DNS de deshabilitado a habilitado, es posible que Service Fabric Explorer no refleje el nuevo estado. Para solucionarlo, reinicie los nodos modificando UpgradePolicy en la plantilla de Azure Resource Manager. Para más información, consulte la [referencia de la plantilla de Service Fabric](/azure/templates/microsoft.servicefabric/2019-03-01/clusters/applications).

> [!NOTE]
> Si habilita el servicio DNS al desarrollar en un equipo local, se invalidará parte de la configuración de DNS. Si tiene problemas para conectarse a Internet, compruebe la configuración de DNS.

## <a name="setting-the-dns-name-for-your-service"></a>Definición del nombre DNS para el servicio
Puede establecer un nombre DNS para los servicios, ya sea mediante declaración para los servicios predeterminados en el archivo ApplicationManifest.xml o con comandos de PowerShell.

El nombre DNS para el servicio se puede resolver en todo el clúster, por lo que es importante garantizar la exclusividad del nombre DNS en el clúster. 

Es muy recomendable que utilice un esquema de nomenclatura de `<ServiceDnsName>.<AppInstanceName>`; por ejemplo, `service1.application1`. Si una aplicación se implementa con Docker Compose, los nombres DNS se asignan automáticamente a los servicios mediante este esquema de nomenclatura.

### <a name="setting-the-dns-name-for-a-default-service-in-the-applicationmanifestxml"></a>Definición del nombre DNS de un servicio predeterminado en ApplicationManifest.xml
Abra el proyecto en Visual Studio o en su editor favorito y luego abra el archivo ApplicationManifest.xml. Vaya a la sección de servicios predeterminados y, para cada servicio, agregue el atributo `ServiceDnsName`. En el ejemplo siguiente se muestra cómo establecer el nombre DNS del servicio en `service1.application1`.

```xml
    <Service Name="Stateless1" ServiceDnsName="service1.application1">
      <StatelessService ServiceTypeName="Stateless1Type" InstanceCount="[Stateless1_InstanceCount]">
        <SingletonPartition />
      </StatelessService>
    </Service>
```
Una vez implementada la aplicación, la instancia de servicio de Service Fabric Explorer muestra el nombre DNS de esta instancia, como se muestra en la siguiente imagen: 

![puntos de conexión del servicio](./media/service-fabric-dnsservice/service-fabric-explorer-dns.png)

En el ejemplo siguiente se establece el nombre DNS para un servicio con estado en `statefulsvc.app`. El servicio utiliza un esquema de partición con nombre. Tenga en cuenta que los nombres de particiones están en minúscula. Se trata de un requisito para las particiones de destino en las consultas de DNS; para más información, vea [Realización de consultas de DNS en una partición de servicio con estado](#preview-making-dns-queries-on-a-stateful-service-partition).

```xml
    <Service Name="Stateful1" ServiceDnsName="statefulsvc.app" />
      <StatefulService ServiceTypeName="ProcessingType" TargetReplicaSetSize="2" MinReplicaSetSize="2">
        <NamedPartition>
         <Partition Name="partition1" />
         <Partition Name="partition2" />
        </NamedPartition>
      </StatefulService>
    </Service>
```

### <a name="setting-the-dns-name-for-a-service-using-powershell"></a>Definición del nombre DNS de un servicio mediante PowerShell
Puede establecer el nombre DNS de un servicio cuando se crea mediante el comando de PowerShell `New-ServiceFabricService`. En el ejemplo siguiente se crea un servicio sin estado nuevo con el nombre DNS `service1.application1`.

```powershell
    New-ServiceFabricService `
    -Stateless `
    -PartitionSchemeSingleton `
    -ApplicationName `fabric:/application1 `
    -ServiceName fabric:/application1/service1 `
    -ServiceTypeName Service1Type `
    -InstanceCount 1 `
    -ServiceDnsName service1.application1
```

## <a name="preview-making-dns-queries-on-a-stateful-service-partition"></a>[Versión preliminar] Realización de consultas de DNS en una partición de servicio con estado
A partir de la versión 6.3 de Service Fabric, el servicio DNS de Service Fabric admite las consultas para las particiones del servicio.

Para las particiones que se usarán en las consultas de DNS, se aplican las restricciones de nomenclatura siguientes:

   - Los nombres de partición deben ser compatibles con DNS.
   - Los nombres de partición con varias etiquetas (que incluyen el punto, ".", en el nombre) no deben usarse.
   - Los nombres de partición deben estar en minúscula.

Las consultas de DNS destinadas a una partición presentan el siguiente formato:

```
    <First-Label-Of-Partitioned-Service-DNSName><PartitionPrefix><Target-Partition-Name>< PartitionSuffix>.<Remaining- Partitioned-Service-DNSName>
```
Donde:

- *First-Label-Of-Partitioned-Service-DNSName* es la primera parte de su nombre DNS del servicio.
- *PartitionPrefix* es un valor que se puede establecer en la sección DnsService del manifiesto de clúster o con la plantilla de Resource Manager del clúster. El valor predeterminado es "--". Para más información, consulte la [configuración del servicio DNS](./service-fabric-cluster-fabric-settings.md#dnsservice).
- *Target-Partition-Name* es el nombre de la partición. 
- *PartitionSuffix* es un valor que se puede establecer en la sección DnsService del manifiesto de clúster o con la plantilla de Resource Manager del clúster. El valor predeterminado es una cadena vacía. Para más información, consulte la [configuración del servicio DNS](./service-fabric-cluster-fabric-settings.md#dnsservice).
- *Remaining-Partitioned-Service-DNSName* es la parte restante del nombre DNS del servicio.

En los ejemplos siguientes se muestran las consultas de DNS para servicios con particiones que se ejecutan en un clúster que tiene una configuración predeterminada para `PartitionPrefix` y `PartitionSuffix`: 

- Para resolver la partición "0" de un servicio con el nombre DNS `backendrangedschemesvc.application` que utiliza un esquema de particiones por rangos, utilice `backendrangedschemesvc-0.application`.
- Para resolver la partición "primera" de un servicio con el nombre DNS `backendnamedschemesvc.application` que utiliza un esquema de particiones con nombre, utilice `backendnamedschemesvc-first.application`.

El servicio DNS devuelve la dirección IP de la réplica principal de la partición. Si no se especifica ninguna partición, el servicio devuelve la dirección IP de la réplica principal de una partición seleccionada al azar.

## <a name="using-dns-in-your-services"></a>Uso de DNS en los servicios
Si implementa más de un servicio, puede buscar los puntos de conexión de otros servicios con los que comunicarse mediante un nombre DNS. El servicio DNS funciona para los servicios sin estado y, en la versión 6.3 y versiones posteriores de Service Fabric, para los servicios con estado. En el caso de los servicios con estado que se ejecutan en versiones de Service Fabric anteriores a la 6.3, puede usar el [servicio de proxy inverso](./service-fabric-reverseproxy.md) integrado en llamadas HTTP para llamar a una partición de servicio determinada. 

El servicio DNS no admite puertos dinámicos. Puede usar el servicio de proxy inverso para resolver servicios expuestos en puertos dinámicos.

El código siguiente muestra cómo llamar a un servicio sin estado mediante DNS. Se trata simplemente de una llamada HTTP normal donde se proporcionan el nombre DNS, el puerto y cualquier ruta de acceso opcional como parte de la dirección URL.

```csharp
public class ValuesController : Controller
{
    // GET api
    [HttpGet]
    public async Task<string> Get()
    {
        string result = "";
        try
        {
            Uri uri = new Uri("http://service1.application1:8080/api/values");
            HttpClient client = new HttpClient();
            var response = await client.GetAsync(uri);
            result = await response.Content.ReadAsStringAsync();
            
        }
        catch (Exception e)
        {
            Console.Write(e.Message);
        }

        return result;
    }
}
```

El código siguiente muestra una llamada en una partición específica de un servicio con estado. En este caso, el nombre DNS contiene el nombre de partición (partition1). La llamada asume un clúster con valores predeterminados para `PartitionPrefix` y `PartitionSuffix`.

```csharp
public class ValuesController : Controller
{
    // GET api
    [HttpGet]
    public async Task<string> Get()
    {
        string result = "";
        try
        {
            Uri uri = new Uri("http://service2-partition1.application1:8080/api/values");
            HttpClient client = new HttpClient();
            var response = await client.GetAsync(uri);
            result = await response.Content.ReadAsStringAsync();
            
        }
        catch (Exception e)
        {
            Console.Write(e.Message);
        }

        return result;
    }
}
```

## <a name="known-issues"></a>Problemas conocidos
* En las versiones 6.3, y posteriores, de Service Fabric hay un problema con las búsquedas de DNS de los nombres de servicio que contienen un guión en el nombre DNS. Para más información acerca de este problema, realiza un seguimiento del siguiente [problema de GitHub](https://github.com/Azure/service-fabric-issues/issues/1197). En la próxima actualización, la 6.3, se solucionará el problema. 

* El servicio DNS para servicios de Service Fabric no se admite todavía en Linux. El servicio DNS es compatible con los contenedores en Linux. La resolución manual mediante el cliente de Fabric/ServicePartitionResolver es la alternativa disponible.

## <a name="next-steps"></a>Pasos siguientes
Para más información sobre la comunicación de servicios dentro del clúster, vea [Conexión y comunicación con servicios](service-fabric-connect-and-communicate-with-services.md).
