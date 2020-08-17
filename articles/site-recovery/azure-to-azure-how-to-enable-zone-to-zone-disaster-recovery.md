---
title: Habilitación de la recuperación ante desastres de zona a zona para máquinas virtuales de Azure
description: En este artículo se describe cuándo y cómo usar la recuperación ante desastres de zona a zona para máquinas virtuales de Azure.
author: sideeksh
manager: gaggupta
ms.service: site-recovery
ms.topic: article
ms.date: 04/28/2020
ms.author: sideeksh
ms.openlocfilehash: a1952f6dccf12de4cb1571dacabecf78c65cd01b
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87021654"
---
# <a name="enable-zone-to-zone-disaster-recovery-for-azure-virtual-machines"></a>Habilitación de la recuperación ante desastres de zona a zona para máquinas virtuales de Azure

En este artículo se describe cómo replicar, conmutar por error y conmutar por recuperación máquinas virtuales de Azure de una zona de disponibilidad a otra dentro de la misma región de Azure.

>[!NOTE]
>
>- La compatibilidad para la recuperación ante desastres de zona a zona está limitada actualmente a dos regiones: Sudeste de Asia y Sur de Reino Unido.  
>- Site Recovery no mueve ni almacena los datos de los clientes fuera de la región en la que se ha implementado cuando el cliente usa la recuperación ante desastres de zona a zona. Los clientes pueden seleccionar un almacén de Recovery Services de otra región si así lo deciden. El almacén de Recovery Services contiene metadatos, pero ningún dato real de los clientes.

El servicio Site Recovery contribuye a la estrategia de continuidad empresarial y recuperación ante desastres al mantener sus aplicaciones empresariales al día y disponibles durante interrupciones planeadas y no planeadas. Es la opción recomendada de recuperación ante desastres para mantener sus aplicaciones en funcionamiento si hay interrupciones regionales.

Las zonas de disponibilidad son ubicaciones físicas exclusivas dentro de una región de Azure. Cada zona tiene uno o varios centros de datos. 

## <a name="using-availability-zones-for-disaster-recovery"></a>Uso de las zona de disponibilidad para la recuperación ante desastres 

Normalmente, las zonas de disponibilidad se usan para implementar VM en una configuración de alta disponibilidad. Puede que se encuentren demasiado cerca entre sí como para servir de solución de recuperación ante desastres en caso de desastre natural.

Sin embargo, en algunos escenarios, las zonas de disponibilidad pueden aprovecharse para la recuperación ante desastres:

- Muchos clientes que tenían una estrategia metro de recuperación ante desastres al hospedar aplicaciones en el entorno local a veces buscaban imitar esta estrategia una vez que migraban aplicaciones a Azure. Estos clientes reconocen el hecho de que la estrategia metro de recuperación ante desastres puede no funcionar en caso de que se produzca un desastre físico a gran escala y aceptan este riesgo. Para estos clientes, la recuperación ante desastres de zona a zona se puede usar como opción de recuperación ante desastres.

- Muchos otros clientes tienen una complicada infraestructura de red y no quieren volver a crearla en una región secundaria debido al costo y la complejidad que esto conlleva. La recuperación ante desastres de zona a zona reduce la complejidad, ya que aprovecha los conceptos de redes redundantes entre las zonas de disponibilidad, lo que hace que la configuración sea mucho más sencilla. Estos clientes prefieren la simplicidad y también pueden usar las zonas de disponibilidad para la recuperación ante desastres.

- En algunas regiones que no tienen una región emparejada dentro de la misma jurisdicción legal (por ejemplo, Sudeste de Asia), la recuperación ante desastres de zona a zona puede servir como solución de facto de recuperación ante desastres, ya que ayuda a garantizar el cumplimiento jurídico, debido a que las aplicaciones y los datos no cruzan las fronteras nacionales. 

- La recuperación ante desastres de zona a zona implica la replicación de los datos entre distancias menores en comparación con la recuperación ante desastres de Azure a Azure y, por lo tanto, podrá obtener una menor latencia y, en consecuencia, una reducción en el RPO.

Aunque estas son grandes ventajas, existe la posibilidad de que la recuperación ante desastres de zona a zona no alcance los requisitos de resistencia en caso de que se produzca un desastre natural en toda la región.

## <a name="networking-for-zone-to-zone-disaster-recovery"></a>Redes para la recuperación ante desastres de zona a zona

Como se ha mencionado, la recuperación ante desastres de zona a zona reduce la complejidad, ya que aprovecha los conceptos de redes redundantes entre las zonas de disponibilidad, lo que hace que la configuración sea mucho más sencilla. A continuación se describe el comportamiento de los componentes de red en el escenario de recuperación ante desastres de zona a zona: 

- Red virtual: Puede usar la misma red virtual que la red de origen para las conmutaciones por error reales. Use una red virtual diferente de la red virtual de origen para las conmutaciones por error de prueba.

- Subred: Se admite la conmutación por error en la misma subred.

- Dirección IP privada: Si utiliza direcciones IP estáticas, puede usar las mismas direcciones IP en la zona de destino si decide configurarlas de esta manera.

- Redes aceleradas: De forma similar a la recuperación ante desastres de Azure a Azure, puede habilitar Redes aceleradas si la SKU de la VM lo admite.

- Dirección IP pública: Puede conectar una dirección IP pública estándar creada previamente en la misma región a la VM de destino. Las direcciones IP públicas básicas no admiten los escenarios relacionados con zonas de disponibilidad.

- Equilibrador de carga: El equilibrador de carga estándar es un recurso regional y, por lo tanto, la VM de destino se puede conectar al grupo de back-end del mismo equilibrador de carga. No se requiere un nuevo equilibrador de carga.

- Grupo de seguridad de red: Puede usar los mismos grupos de seguridad de red que se aplican a la VM de origen.

## <a name="pre-requisites"></a>Requisitos previos

Antes de implementar la recuperación ante desastres de zona en zona para las VM, es importante asegurarse de que otras características habilitadas en las VM sean interoperables con la recuperación ante desastres de zona a zona.

|Característica  | Declaración de compatibilidad  |
|---------|---------|
|Máquinas virtuales clásicas   |     No compatible    |
|VM de ARM    |    Compatible    |
|Azure Disk Encryption v1 (pase doble, con AAD)     |     Compatible |
|Azure Disk Encryption v2 (pase único, sin AAD)    |    Compatible    |
|Discos no administrados    |    No compatible    |
|Discos administrados    |    Compatible    |
|Claves administradas por el cliente    |    Compatible    |
|Grupos de selección de ubicación de proximidad    |    Compatible    |
|Interoperabilidad de copias de seguridad    |    Se admiten la copia de seguridad y restauración a nivel de archivo. No se admiten la copia de seguridad y restauración a nivel de disco y VM.    |
|Adición o eliminación en caliente    |    Se pueden agregar discos después de habilitar la replicación de zona a zona. No se admite la eliminación de discos después de habilitar la replicación de zona a zona.    | 

## <a name="set-up-site-recovery-zone-to-zone-disaster-recovery"></a>Configuración de la recuperación ante desastres de zona a zona de Site Recovery

### <a name="log-in"></a>Registro

Inicie sesión en Azure Portal.

### <a name="enable-replication-for-the-zonal-azure-virtual-machine"></a>Habilite la replicación para la máquina virtual zonal de Azure.

1. En el menú de Azure Portal, seleccione Máquinas virtuales o busque y seleccione Máquinas virtuales en cualquier página. Seleccione la máquina virtual que quiere replicar. Para la recuperación ante desastres de zona a zona, esta VM ya debe estar en una zona de disponibilidad.

2. En Operaciones, seleccione Recuperación ante desastres.

3. Como se muestra a continuación, en la pestaña Aspectos básicos, seleccione "Sí" para "¿Recuperación ante desastres entre zonas de disponibilidad?".

    ![Página Configuración básica](./media/azure-to-azure-how-to-enable-zone-to-zone-disaster-recovery/zonal-disaster-recovery-basic-settings-blade.png)

4. Si acepta todos los valores predeterminados, haga clic en "Revisar + iniciar replicación" seguido de "Iniciar replicación".

5. Si desea realizar cambios en la configuración de replicación, haga clic en "Siguiente: Configuración avanzada".

6. Cambie la configuración del valor predeterminado siempre que sea adecuado. En el caso de los usuarios de recuperación ante desastres de Azure a Azure, es posible que esta página le resulte familiar. [Aquí](./azure-to-azure-tutorial-enable-replication.md) puede encontrar más información sobre las opciones que se presentan en esta hoja.

    ![Página Configuración avanzada](./media/azure-to-azure-how-to-enable-zone-to-zone-disaster-recovery/zonal-disaster-recovery-advanced-settings-blade.png)

7. Haga clic en "Siguiente: Revise + iniciar replicación" y, a continuación, "Iniciar replicación".

## <a name="faqs"></a>Preguntas más frecuentes

**1. ¿Cómo funcionan los precios para la recuperación ante desastres de zona a zona?**
Los precios de la recuperación ante desastres de zona a zona son idénticos a los precios de la recuperación ante desastres de Azure a Azure. Puede encontrar más detalles en la página de precios [aquí](https://azure.microsoft.com/pricing/details/site-recovery/) y también [aquí](https://azure.microsoft.com/blog/know-exactly-how-much-it-will-cost-for-enabling-dr-to-your-azure-vm/). Tenga en cuenta que los cargos de salida que podría ver en la recuperación ante desastres de zona a zona serían menores que para la recuperación ante desastres de región a región.

**2. ¿Cuál es el SLA para RTO y RPO?**
El SLA de RTO es el mismo que para Site Recovery en general. Prometemos un RTO de hasta 2 horas. No se ha definido un SLA para el RPO.

**3. ¿Se garantiza la capacidad en la zona secundaria?**
El equipo de Site Recovery y el equipo de administración de la capacidad de Azure planean que haya una capacidad de infraestructura suficiente. Cuando se inicia una conmutación por error, los equipos también ayudan a garantizar que las instancias que están protegidas mediante Site Recovery se implementarán en la zona de destino.

**4. ¿Qué sistemas operativos se admiten?**
La recuperación ante desastres de zona a zona admite los mismos sistemas operativos que la recuperación ante desastres de Azure a Azure. Consulte la matriz de compatibilidad [aquí](./azure-to-azure-support-matrix.md).

**5. ¿Pueden los grupos de recursos de origen y de destino ser los mismos?**
No, debe conmutar por error a otro grupo de recursos.

## <a name="next-steps"></a>Pasos siguientes

Los pasos que deben seguirse para ejecutar un simulacro de recuperación ante desastres, conmutación por error, reprotección y conmutación por recuperación son los mismos que los del escenario de recuperación ante desastres de Azure en Azure.

Para realizar un simulacro de recuperación ante desastres, siga los pasos descritos [aquí](./azure-to-azure-tutorial-dr-drill.md).

Para realizar una conmutación por error y volver a proteger las VM de la zona secundaria, siga los pasos descritos [aquí](./azure-to-azure-tutorial-failover-failback.md).

Para conmutar por recuperación a la zona principal, siga los pasos descritos [aquí](./azure-to-azure-tutorial-failback.md).
