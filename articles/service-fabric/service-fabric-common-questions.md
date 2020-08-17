---
title: Preguntas frecuentes sobre Microsoft Azure Service Fabric
description: Preguntas más frecuentes sobre Service Fabric, incluyendo funciones, casos de uso y escenarios habituales.
ms.topic: troubleshooting
ms.date: 08/18/2017
ms.author: pepogors
ms.openlocfilehash: 1655a8ed03b1f678cc5dba0a165e0bcca1d2517a
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87292849"
---
# <a name="commonly-asked-service-fabric-questions"></a>Preguntas frecuentes sobre Service Fabric

Hay muchas preguntas que se plantean con frecuencia acerca de qué puede hacer Service Fabric y cómo se debe usar. En este documento se incluyen muchas de esas preguntas y sus respuestas.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="cluster-setup-and-management"></a>Instalación y administración de clústeres

### <a name="how-do-i-roll-back-my-service-fabric-cluster-certificate"></a>¿Cómo se revierte el certificado de clúster de Service Fabric?

Revertir cualquier actualización de la aplicación requiere la detección de errores de mantenimiento antes de que el cuórum de clúster de Service Fabric confirme el cambio; solo se pueden poner al día los cambios confirmados. Puede ser necesario el escalamiento a otro ingeniero a través de los servicios de soporte técnico al cliente para recuperar el clúster, si se ha introducido un cambio de certificado importante no supervisado.  La [actualización de la aplicación de Service Fabric](./service-fabric-application-upgrade.md?branch=master) aplica [parámetros de actualización de la aplicación](./service-fabric-application-upgrade-parameters.md?branch=master), y ofrece la promesa de actualización sin tiempo de inactividad.  Al seguir el modo supervisado recomendado de actualización de la aplicación, el progreso automático de actualización de los dominios se basa en pasar comprobaciones de mantenimiento y se revierte automáticamente si se produce un error en la actualización de un servicio predeterminado.
 
Si el clúster todavía utiliza la propiedad clásica de huella digital del certificado en la plantilla de Resource Manager, se recomienda [cambiar el clúster de huella digital del certificado por el nombre común](./service-fabric-cluster-change-cert-thumbprint-to-cn.md), para aprovechar las modernas características de administración de secretos.

### <a name="can-i-create-a-cluster-that-spans-multiple-azure-regions-or-my-own-datacenters"></a>¿Puedo crear un clúster que abarque varias regiones de Azure o mis propios centros de datos?

Sí. 

La tecnología básica de agrupación en clústeres de Service Fabric puede utilizarse para combinar máquinas que se ejecutan en cualquier lugar del mundo, siempre y cuando tengan conectividad de red entre sí. Sin embargo, compilar y ejecutar clústeres de este tipo puede resultar complicado.

Si está interesado en este escenario, le animamos a que se ponga en contacto con nosotros, ya sea a través de la [lista de problemas de Service Fabric de GitHub](https://github.com/azure/service-fabric-issues) o a través de su representante de soporte técnico para obtener orientación adicional. El equipo de Service Fabric está trabajando para ofrecer mayor claridad, instrucciones y recomendaciones para este escenario. 

Hay varias cosas que deben tenerse en cuenta: 

1. En estos momentos el recurso de clúster de Service Fabric en Azure es regional, como los conjuntos de escalado de máquinas virtuales en los que el clúster está basado. Esto significa que si se produce un error regional puede perder la capacidad para administrar el clúster mediante Azure Resource Manager o Azure Portal. Esto puede ocurrir incluso aunque el clúster permanezca en ejecución y se pueda interactuar con él directamente. Además, actualmente Azure no ofrecen la capacidad de tener una única red virtual que se pueda usar en varias regiones. Esto significa que un clúster de varias regiones de Azure necesita [direcciones IP públicas para cada máquina virtual de los conjuntos de escalado de máquinas virtuales](../virtual-machine-scale-sets/virtual-machine-scale-sets-networking.md#public-ipv4-per-virtual-machine) o instancias de [Azure VPN Gateway](../vpn-gateway/vpn-gateway-about-vpngateways.md). Estas opciones de red tienen efectos diferentes en los costos, el rendimiento, y hasta cierto punto, en el diseño de la aplicación, por ello es necesario realizar un análisis y un planeamiento meticuloso antes de poner en marcha un entorno así.
2. El mantenimiento, administración y supervisión de estas máquinas puede llegar a complicarse, especialmente cuando se distribuye entre diferentes _tipos_ de entornos, como por ejemplo entre diferentes proveedores de nube o entre recursos locales y Azure. Debe tener cuidado para asegurarse de que tanto el clúster como las aplicaciones entienden las actualizaciones, supervisión, administración y diagnóstico antes de ejecutar cargas de trabajo de producción en un entorno de este tipo. Si ya tiene amplia experiencia en la resolución de estos problemas en Azure o dentro de sus propios centros de datos, es probable que esas mismas soluciones que utiliza se puedan aplicar al crear o ejecutar el clúster de Service Fabric. 

### <a name="do-service-fabric-nodes-automatically-receive-os-updates"></a>¿Los nodos de Service Fabric reciben automáticamente actualizaciones del sistema operativo?

Puede usar la [actualización de la imagen del SO automática del conjunto de escalado de máquinas virtuales](../virtual-machine-scale-sets/virtual-machine-scale-sets-automatic-upgrade.md) que desde hoy está disponible con carácter general.

Para los clústeres que NO se ejecutan en Azure, hemos [proporcionado una aplicación](service-fabric-patch-orchestration-application.md) que permite revisar los sistemas operativos debajo de los nodos de Service Fabric.

### <a name="can-i-use-large-virtual-machine-scale-sets-in-my-sf-cluster"></a>¿Puedo usar conjuntos de escalado de máquinas virtuales en el clúster SF? 

**Respuesta corta**: no. 

**Respuesta larga**: aunque los conjuntos de escalado grandes de máquinas virtuales permiten escalar un conjunto de escalado de máquinas virtuales hasta en 1000 instancias de VM, lo hace mediante el uso de grupos de selección de ubicación (PG). Los dominios de error (FD) y los dominios de actualización (UD) solo son coherentes dentro de un grupo de selección de ubicación. Service Fabric usa los FD y los UD para tomar decisiones de selección de ubicación de las réplicas o instancias del servicio. Puesto que los FD y los UD solo son comparables dentro de un grupo de selección de ubicación, SF no puede usarlo. Por ejemplo, si VM1 en PG1 tiene una topología de FD=0 y VM9 en PG2 tiene una topología de FD=4, esto no significa que VM1 y VM2 se encuentren en dos bastidores de hardware diferentes. Por tanto, SF no podrá usar los valores de FD en este caso para tomar decisiones de selección de ubicación.

Actualmente, hay otros problemas con los conjuntos de escalado grandes de máquinas virtuales, como la falta de compatibilidad con el equilibrio de carga del nivel 4. Consulte para obtener [detalles sobre conjuntos de escalado grandes](../virtual-machine-scale-sets/virtual-machine-scale-sets-placement-groups.md)



### <a name="what-is-the-minimum-size-of-a-service-fabric-cluster-why-cant-it-be-smaller"></a>¿Cuál es el tamaño mínimo de un clúster de Service Fabric? ¿Por qué no puede ser menor?

El tamaño mínimo admitido para un clúster de Service Fabric que ejecute cargas de trabajo de producción es cinco nodos. Para escenarios de desarrollo, se admite un nodo (optimizado para la experiencia de desarrollo rápido de Visual Studio) y clústeres de cinco nodos.

Se necesita un clúster de producción que tenga al menos cinco nodos debido a las tres razones siguientes:
1. Incluso cuando no hay servicios de usuario en ejecución, un clúster de Service Fabric ejecuta un conjunto de servicios del sistema con estado, incluidos el servicio de nomenclatura y el administrador de conmutación por error. Estos servicios del sistema son esenciales para que el clúster siga estando operativo.
2. Siempre se coloca una réplica de un servicio por nodo, por lo que el tamaño del clúster es el límite superior del número de réplicas que puede tener un servicio (en realidad una partición).
3. Puesto que una actualización del clúster desactivará al menos un nodo y querremos tener un búfer de al menos un nodo, un clúster de producción deberá tener al menos dos nodos *además* del mínimo. El mínimo es el tamaño del cuórum de un servicio del sistema, como se explica a continuación.  

Queremos que el clúster esté disponible en el caso de error simultáneo de dos nodos. Para que un clúster de Service Fabric esté disponible, los servicios del sistema deben estar disponibles. Los servicios del sistema con estado, como el servicio de nomenclatura y el administrador de conmutación por error, que realizan un seguimiento de qué servicios se han implementado en el clúster y en dónde se hospedan en la actualidad, dependen de la existencia de una fuerte coherencia. Esta fuerte coherencia depende a su vez de la capacidad de obtener *cuórum* para cualquier actualización del estado de esos servicios, en donde un cuórum representa una mayoría estricta de las réplicas (N/2 + 1) para un servicio dado. Por lo tanto, si queremos que sea resistente contra la pérdida simultánea de dos nodos (por lo tanto, pérdida simultánea de dos réplicas de un servicio de sistema), debemos tener TamañoDelClúster - TamañoDelCuórum > = 2, lo que fuerza el tamaño mínimo a cinco. Para ver esto, considere que el clúster tiene N nodos y hay N réplicas de un servicio del sistema: una en cada nodo. El tamaño del cuórum para un servicio del sistema es (N/2 + 1). La desigualdad anterior se parece a N - (N/2 + 1) > = 2. Hay dos casos a tener en cuenta: cuando N es par y cuando N es impar. Si N es par, digamos N = 2\*m, donde m > = 1, la desigualdad se parece a 2\*m - (2\*m/2 + 1) > = 2 o m > = 3. El valor mínimo para N es 6 y se logra cuando m = 3. Por otro lado, si N es impar, por ejemplo, N = 2\*m + 1, donde m > = 1, la desigualdad se parece a 2\*m+1 - ((2\*m+1) / 2 + 1) > = 2 o 2\*m+1 - (m+1) > = 2 o m > = 2. El valor mínimo para N es 5 y se logra cuando m = 2. Por lo tanto, entre todos los valores de N que satisfacen la desigualdad TamañoDelClúster - TamañoDelCuórum > = 2, el valor mínimo es 5.

Tenga en cuenta que en el argumento anterior se asume que todos los nodos tienen una réplica de un servicio del sistema, por lo tanto el tamaño del cuórum se calcula en función del número de nodos del clúster. Sin embargo, al cambiar *TargetReplicaSetSize*, podríamos hacer el tamaño del cuórum menor que (N/2+1), lo que podría dar la impresión de que podríamos tener un clúster menor de 5 nodos y que todavía tiene 2 nodos adicionales por encima del tamaño del cuórum. Por ejemplo, en un clúster de cuatro nodos, si se establece el valor de TargetReplicaSetSize en 3, el tamaño del cuórum según el valor de TargetReplicaSetSize es (3/2 + 1) o 2, por lo tanto, tenemos ClusterSize - QuorumSize = 4-2 > = 2. Sin embargo, no podemos garantizar que el servicio del sistema estará en o por encima del cuórum si se pierde cualquier par de nodos al mismo tiempo; es posible que los dos nodos que hemos perdido hospedaran dos réplicas, por lo que el servicio del sistema pasará a pérdida de cuórum (con solo una única réplica restante) y dejará de estar disponible.

Con esta información, examinemos algunas posibles configuraciones de clúster:

**Un nodo**: esta opción no proporciona alta disponibilidad, ya que la pérdida de este nodo único por cualquier razón, supone la pérdida de todo el clúster.

**Dos nodos**: un cuórum para un servicio implementado en dos nodos (N = 2) is 2 (2/2 + 1 = 2). Cuando se pierde una réplica, es imposible crear un cuórum. Teniendo en cuenta que realizar una actualización del servicio requiere desconectar una réplica, esta no es una configuración útil.

**Tres nodos**: con tres nodos (N=3), el requisito para crear un cuórum es aún dos nodos (3/2 + 1 = 2). Esto significa que puede perder un solo nodo y mantener el cuórum, pero un error simultáneo de dos nodos llevará los servicios del sistema a pérdida de cuórum y hará que el clúster deje de estar disponible.

**Cuatro nodos**: con cuatro nodos (N=4), el requisito para crear un cuórum es tres nodos (4/2 + 1 = 3). Esto significa que puede perder un solo nodo y mantener el cuórum, pero un error simultáneo de dos nodos llevará los servicios del sistema a pérdida de cuórum y hará que el clúster deje de estar disponible.

**Cinco nodos**: con cinco nodos (N=5), el requisito para crear un cuórum es aún de tres nodos (5/2 + 1 = 3). Esto significa que puede perder dos nodos al mismo tiempo y continuar manteniendo el cuórum para los servicios del sistema.

Para cargas de trabajo de producción, debe ser resistente a errores simultáneos de al menos dos nodos (por ejemplo, uno debido a una actualización del clúster y uno por otras razones), por lo que se necesitan cinco nodos.

### <a name="can-i-turn-off-my-cluster-at-nightweekends-to-save-costs"></a>¿Puedo apagar mi clúster por la noche o los fines de semana para ahorrar costos?

En general, no. Service Fabric almacena el estado en discos efímeros locales, lo que quiere decir que si la máquina virtual se mueve a un host distinto, los datos no se mueven con ella. En el funcionamiento normal, esto no es un problema ya que otros nodos ponen al día al nuevo nodo. Pero, si detiene todos los nodos y los reinicia más tarde, existe una posibilidad importante de que la mayoría de los nodos se inicien en nuevos hosts y hagan que el sistema no pueda recuperar.

Si quiere crear clústeres para probar su aplicación antes de implementarla, le recomendamos que cree dinámicamente esos clústeres como parte de su [integración continua/ canalización de implementación continua](service-fabric-tutorial-deploy-app-with-cicd-vsts.md).


### <a name="how-do-i-upgrade-my-operating-system-for-example-from-windows-server-2012-to-windows-server-2016"></a>¿Cómo se puede actualizar el sistema operativo (por ejemplo, de Windows Server 2012 a Windows Server 2016)?

Mientras trabajamos en una experiencia mejorada, por el momento, el usuario es el responsable de la actualización. Debe actualizar la imagen de SO en las máquinas virtuales del clúster, pero una por una. 

### <a name="can-i-encrypt-attached-data-disks-in-a-cluster-node-type-virtual-machine-scale-set"></a>¿Se pueden cifrar los discos de datos asociados en un tipo de nodo de clúster (conjunto de escalado de máquinas virtuales)?
Sí.  Para más información, consulte [Creación de un clúster con discos de datos asociados](../virtual-machine-scale-sets/virtual-machine-scale-sets-attached-disks.md#create-a-service-fabric-cluster-with-attached-data-disks) y [Azure Disk Encryption para conjuntos de escalado de máquinas virtuales](../virtual-machine-scale-sets/disk-encryption-overview.md).

### <a name="can-i-use-low-priority-vms-in-a-cluster-node-type-virtual-machine-scale-set"></a>¿Se pueden usar máquinas virtuales de prioridad baja en un tipo de nodo de clúster (conjunto de escalado de máquinas virtuales)?
No. No se admiten máquinas virtuales de prioridad baja. 

### <a name="what-are-the-directories-and-processes-that-i-need-to-exclude-when-running-an-anti-virus-program-in-my-cluster"></a>¿Qué directorios y procesos es necesario excluir al ejecutar un programa antivirus en un clúster?

| **Directorios excluidos del antivirus** |
| --- |
| Archivos de programa\Microsoft Service Fabric |
| FabricDataRoot (de la configuración del clúster) |
| FabricLogRoot (de la configuración del clúster) |

| **Procesos excluidos del antivirus** |
| --- |
| Fabric.exe |
| FabricHost.exe |
| FabricInstallerService.exe |
| FabricSetup.exe |
| FabricDeployer.exe |
| ImageBuilder.exe |
| FabricGateway.exe |
| FabricDCA.exe |
| FabricFAS.exe |
| FabricUOS.exe |
| FabricRM.exe |
| FileStoreService.exe |
 
### <a name="how-can-my-application-authenticate-to-key-vault-to-get-secrets"></a>¿Cómo se puede autenticar una aplicación en Key Vault para obtener secretos?
Estos son los distintos medios para que una aplicación obtenga las credenciales para autenticarse en Key Vault:

A. Durante el trabajo de compilación y empaquetado de las aplicaciones puede extraer un certificado en el paquete de datos de una aplicación de Service Fabric y usarlo para autenticarse en Key Vault.
B. En el caso de los hosts con MSI habilitado en el conjunto de escalado de máquinas virtuales, puede desarrollar un sencillo PowerShell SetupEntryPoint para que la aplicación de Service Fabric obtenga [un token de acceso del punto de conexión de MSI](../active-directory/managed-identities-azure-resources/how-to-use-vm-token.md) y después [recuperar los secretos de Key Vault](/powershell/module/azurerm.keyvault/get-azurekeyvaultsecret).

## <a name="application-design"></a>Diseño de aplicaciones

### <a name="whats-the-best-way-to-query-data-across-partitions-of-a-reliable-collection"></a>¿Cuál es la mejor forma de consultar datos entre las particiones de una instancia de Reliable Collection?

Las colecciones confiables están generalmente [particionadas](service-fabric-concepts-partitioning.md) para habilitar el escalado horizontal, y así conseguir un mayor rendimiento y capacidad de proceso. Esto significa que el estado de un servicio determinado puede distribuirse entre decenas o cientos de máquinas. Para realizar operaciones sobre ese conjunto de datos completo, tiene varias opciones:

- Crear un servicio que realiza consultas en todas las particiones de otro servicio para extraer los datos necesarios.
- Crear un servicio que puede recibir datos de todas las particiones de otro servicio.
- Insertar datos periódicamente desde cada servicio a un almacén externo. Este método es adecuado solamente si las consultas que va a realizar no forman parte de su lógica de negocios troncal, ya que los datos del almacén externo estarán obsoletos.
- También puede almacenar datos que deben admitir la consulta en todos los registros directamente en un almacén de datos, en lugar de en una colección de confianza. Esto elimina el problema con los datos obsoletos, pero no permite aprovechar las ventajas de las colecciones de confianza.


### <a name="whats-the-best-way-to-query-data-across-my-actors"></a>¿Cuál es la mejor forma de consultar los datos a través de mis actores?

Los actores están diseñados para ser unidades independientes de estado y cálculo, por lo que no se recomienda realizar consultas amplias de estado de actor en tiempo de ejecución. Si tiene una necesidad de consulta en el conjunto completo de estado de actor, debe considerar una de estas dos opciones:

- Reemplazar los servicios de actor por servicios confiables con estado, como el número de solicitudes de red para recopilar todos los datos del número de actores para el número de particiones en el servicio.
- Diseñar los actores para insertar periódicamente su estado en un almacén externo y así facilitar las consultas. Como anteriormente, este método solo es viable si las consultas que está realizando no son necesarias para su comportamiento en tiempo de ejecución.

### <a name="how-much-data-can-i-store-in-a-reliable-collection"></a>¿Qué cantidad de datos puedo almacenar en una instancia de Reliable Collection?

Reliable Services normalmente tienen particiones, por lo que la cantidad que se puede almacenar está limitada únicamente por el número de máquinas que tiene en el clúster y la cantidad de memoria disponible en esas máquinas.

Por ejemplo, suponga que tiene una colección confiable de un servicio con 100 particiones y 3 réplicas, en donde se almacenan objetos que tienen un tamaño medio de 1 kB. Ahora suponga que tiene un clúster de 10 máquinas con 16 GB de memoria por máquina. Por simplificar y para mantener un enfoque conservador, suponga que el sistema operativo y los servicios del sistema, el tiempo de ejecución de Service Fabric y sus servicios consumen 6 GB, y dejan 10 GB disponibles por máquina, o 100 GB para el clúster.

Teniendo en cuenta que cada objeto tiene que almacenarse tres veces (una principal y dos réplicas), debería tener suficiente memoria para aproximadamente 35 millones de objetos en la colección cuando se trabaja con la máxima capacidad. Sin embargo, se recomienda que cuente con la pérdida simultánea de un dominio de error y un dominio de actualización, lo que representa aproximadamente 1/3 de la capacidad y que podría reducir el número hasta aproximadamente a 23 millones.

En este cálculo también se da por supuesto:

- Que la distribución de datos en las particiones es aproximadamente uniforme o que informa sobre las métricas de carga a Cluster Resource Manager. De forma predeterminada, Service Fabric equilibra la carga según el número de réplicas. En el ejemplo anterior, se pondrían 10 réplicas principales y 20 réplicas secundarias en cada nodo del clúster. Esto funciona bien para las cargas que se distribuyen uniformemente en todas las particiones. Si carga no es uniforme, tiene que informar sobre ella para que Resource Manager pueda empaquetar juntas las réplicas más pequeñas y dejar que las réplicas mayores consuman más memoria en un nodo individual.

- Que el servicio confiable en cuestión es el único que almacena estado en el clúster. Dado que puede implementar varios servicios en un clúster, tiene que prestar atención a los recursos que cada uno de ellos necesita para ejecutar y administrar su estado.

- Que el propio clúster no experimente crecimiento o reducción. Si agrega más máquinas, Service Fabric reequilibra las réplicas para aprovechar la capacidad adicional hasta que el número de máquinas supere el número de particiones en el servicio, ya que una réplica individual no puede abarcar varias máquinas. Por el contrario, si reduce el tamaño del clúster mediante la eliminación de máquinas, las réplicas se empaquetan de forma más ajustada y tienen menos capacidad total.

### <a name="how-much-data-can-i-store-in-an-actor"></a>¿Cuántos datos puedo almacenar en un actor?

Al igual que con Reliable Services, la cantidad de datos que se pueden almacenar en un servicio de actor solo está limitada por el espacio total disponible en disco y memoria en todos los nodos del clúster. Sin embargo, los actores individuales son más eficaces cuando se utilizan para encapsular una pequeña cantidad de estado y la lógica de negocio asociada. Como norma general, un actor individual debe tener un estado que se mida en kilobytes.


### <a name="where-does-azure-service-fabric-resource-provider-store-customer-data"></a>¿Dónde almacena el proveedor de recursos de Azure Service Fabric los datos de los clientes?

El proveedor de recursos de Azure Service Fabric no mueve ni almacena los datos de los clientes fuera de la región en que están implementados.


## <a name="other-questions"></a>Otras preguntas

### <a name="how-does-service-fabric-relate-to-containers"></a>¿Cómo se relacionan Service Fabric con los contenedores?

Los contenedores ofrecen una manera sencilla de empaquetar servicios y sus dependencias, de modo que se ejecuten de forma coherente en todos los entornos y puedan funcionar de forma aislada en una única máquina. Service Fabric ofrece una manera de implementar y administrar servicios, incluidos los [servicios empaquetados en un contenedor](service-fabric-containers-overview.md).

### <a name="are-you-planning-to-open-source-service-fabric"></a>¿Tienen previsto abrir el código de Service Fabric?

Tenemos partes de código abierto de Service Fabric ([marco de Reliable Services](https://github.com/Azure/service-fabric-services-and-actors-dotnet), [marco de Reliable Actors](https://github.com/Azure/service-fabric-services-and-actors-dotnet), [bibliotecas de integración de ASP.NET Core](https://github.com/Azure/service-fabric-aspnetcore), [ Service Fabric Explorer](https://github.com/Azure/service-fabric-explorer) y [Service Fabric CLI](https://github.com/Azure/service-fabric-cli)) en GitHub y se aceptan contribuciones de la comunidad a estos proyectos. 

Se [anunció recientemente](https://techcommunity.microsoft.com/t5/azure-service-fabric/bg-p/Service-Fabric) que tenemos previsto ofrece el runtime de Service Fabric en código abierto. En este momento, tenemos el [repositorio de Service Fabric](https://github.com/Microsoft/service-fabric/) en GitHub con las herramientas de compilación y prueba de Linux, lo que significa que puede clonarlo, compilar Service Fabric para Linux, ejecutar pruebas básicas, abrir problemas y enviar solicitudes de incorporación de cambios. Estamos trabajando para también migrar el entorno de compilación de Windows, junto con un entorno completo de integración continua.

Siga el [blog de Service Fabric](https://techcommunity.microsoft.com/t5/azure-service-fabric/bg-p/Service-Fabric) para más información cuando se anuncie.

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre los [conceptos fundamentales de Service Fabric](service-fabric-technical-overview.md) y los [procedimientos recomendados](service-fabric-best-practices-overview.md).
