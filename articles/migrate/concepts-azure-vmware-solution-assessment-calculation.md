---
title: Cálculos de evaluación de Azure VMware Solution en Azure Migrate | Microsoft Docs
description: En este artículo se proporciona una introducción a los cálculos de evaluación de Azure VMware Solution en el servicio Azure Migrate.
author: rashi-ms
ms.service: azure-migrate
ms.topic: conceptual
ms.date: 06/25/2020
ms.author: mahain
ms.openlocfilehash: 3469b612081f9cb96beec98a065e0827d1c04b4c
ms.sourcegitcommit: 64ad2c8effa70506591b88abaa8836d64621e166
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/17/2020
ms.locfileid: "88261844"
---
# <a name="server-assessment-overview-migrate-to-azure-vmware-solution"></a>Introducción a la evaluación del servidor (migración a Azure VMware Solution)

[Azure Migrate](migrate-services-overview.md) proporciona un centro principal para realizar el seguimiento de la detección, evaluación y migración de las aplicaciones y cargas de trabajo locales. También realiza un seguimiento de las instancias de nube privada y pública en Azure. El centro proporciona herramientas de Azure Migrate para la evaluación y la migración, así como ofertas de proveedores de software independientes (ISV).

Server Assessment es una herramienta de Azure Migrate que evalúa los servidores locales para la migración a máquinas virtuales de IaaS de Azure y a Azure VMware Solution (AVS). En este artículo se proporciona información sobre cómo se calculan las evaluaciones de Azure VMware Solution (AVS).

> [!NOTE]
> La evaluación de Azure VMware Solution (AVS) se encuentra actualmente en versión preliminar y se puede crear solo para máquinas virtuales de VMware.

## <a name="types-of-assessments"></a>Tipos de evaluaciones

Las evaluaciones que se crean con Server Assessment son una instantánea puntual de los datos. Con Azure Migrate: Server Assessment se pueden crear dos Server Assessment.

**Tipo de evaluación** | **Detalles**
--- | --- 
**MV de Azure** | Evaluaciones para la migración de los servidores locales a máquinas virtuales de Azure. <br/><br/> Puede evaluar las [máquinas virtuales de VMware](how-to-set-up-appliance-vmware.md), las [máquinas virtuales Hyper-V](how-to-set-up-appliance-hyper-v.md) y los [servidores físicos](how-to-set-up-appliance-physical.md) locales para la migración a Azure con este tipo de evaluación. [Más información](concepts-assessment-calculation.md)
**Azure VMware Solution (AVS)** | Evaluaciones para la migración de los servidores locales a [Azure VMware Solution (AVS)](../azure-vmware/introduction.md). <br/><br/> Puede evaluar las [máquinas virtuales de VMware](how-to-set-up-appliance-vmware.md) locales para la migración a Azure VMware Solution (AVS) con este tipo de evaluación. [Más información](concepts-azure-vmware-solution-assessment-calculation.md)

Una evaluación de Azure VMware Solution (AVS) en Server Assessment proporciona dos opciones de criterios de dimensionamiento:

**Valoración** | **Detalles** | **Data**
--- | --- | ---
**Basada en el rendimiento** | Evaluaciones basadas en los datos de rendimiento recopilados de las máquinas virtuales locales. | **Tamaño del nodo recomendado**: se basa en los datos de uso de la CPU y de la memoria junto con el tipo de nodo, el tipo de almacenamiento y la configuración de FTT seleccionados para la evaluación.
**Como local** | Evaluaciones que se basan en el tamaño local. | **Tamaño del nodo recomendado**: se basa en el tamaño de la máquina virtual local junto con el tipo de nodo, el tipo de almacenamiento y la configuración de FTT seleccionados para la evaluación.

## <a name="how-do-i-run-an-assessment"></a>¿Cómo se ejecuta una evaluación?

Existen dos formas de realizar una evaluación:

- Evaluar las máquinas con los metadatos de servidor recopilados por un dispositivo de Azure Migrate ligero. El dispositivo detecta las máquinas locales. A continuación, envía continuamente los metadatos y los datos de rendimiento de las máquinas a Azure Migrate.
- Evaluar las máquinas mediante los metadatos de servidor importados en un formato de valores separados por comas (CSV).

## <a name="how-do-i-assess-with-the-appliance"></a>¿Cómo se evalúa con el dispositivo?

Haga lo siguiente si va a implementar un dispositivo de Azure Migrate para detectar servidores locales:

1. Configure Azure y el entorno local para que funcionen con Server Assessment.
2. Para realizar la primera evaluación, cree un proyecto de Azure y, luego, agregue la herramienta Server Assessment.
3. Implemente un dispositivo de Azure Migrate ligero. Este dispositivo detecta las máquinas locales de forma constante y envía a Azure Migrate los metadatos y los datos de rendimiento de las máquinas. Implemente el dispositivo como una máquina virtual. No es necesario instalar nada en las máquinas que quiera evaluar.

Una vez que el dispositivo inicia la detección de las máquinas, puede reunir aquellas que quiera evaluar en un grupo y realizar una evaluación del grupo con el tipo de evaluación de **Azure VMware Solution (AVS)** .

Cree la primera evaluación de Azure VMware Solution (AVS) mediante los pasos que se indican [aquí](how-to-create-azure-vmware-solution-assessment.md).

## <a name="how-do-i-assess-with-imported-data"></a>¿Cómo se evalúa con datos importados?

Si va a evaluar los servidores con un archivo CSV, no necesita un dispositivo. En su lugar, siga estos pasos:

1. Configure Azure para que funcione con Server Assessment.
2. Para realizar la primera evaluación, cree un proyecto de Azure y, luego, agregue la herramienta Server Assessment.
3. Descargue una plantilla CSV y agregue los datos de los servidores.
4. Importe la plantilla en Server Assessment.
5. Detecte los servidores agregados con la importación, reúnalos en un grupo y realice una evaluación del grupo con el tipo de evaluación de **Azure VMware Solution (AVS)** .

## <a name="what-data-does-the-appliance-collect"></a>¿Qué datos recopila el dispositivo?

Si usa el dispositivo Azure Migrate para la evaluación, obtenga información sobre los metadatos y los datos de rendimiento que se recopilan para [VMware](migrate-appliance.md#collected-data---vmware).

## <a name="how-does-the-appliance-calculate-performance-data"></a>¿Cómo calcula el dispositivo los datos de rendimiento?

Si usa el dispositivo para la detección, los datos de rendimiento de la configuración de proceso se recopilan de la siguiente manera:

1. El dispositivo recopila un punto de ejemplo en tiempo real.

    - **VM de VMware**: se recopila un punto de ejemplo cada 20 segundos.

2. El dispositivo combina los puntos de ejemplo para crear un único punto de datos cada 10 minutos. Para crear el punto de datos, el dispositivo selecciona los valores máximos de todos los ejemplos. A continuación, envía el punto de datos a Azure.
3. Server Assessment almacena todos los puntos de ejemplo de 10 minutos del último mes.
4. Al crear una evaluación, Server Assessment identifica el punto de datos adecuado que se va a usar para elegir el tamaño adecuado. La identificación se basa en los valores de percentil de *historial de rendimiento* y *uso de percentil*.

    - Por ejemplo, si el historial de rendimiento es de una semana y el uso de percentil es el percentil 95, Server Assessment ordena los puntos de muestra de 10 minutos de la última semana. Los clasifica en orden ascendente y selecciona el valor de percentil 95 para la elección del tamaño adecuado.
    - El valor del percentil 95 garantiza que se van a omitir los valores atípicos, que podrían incluirse si eligiera el percentil 99.
    - Si quiere elegir el uso máximo en el período y no quiere omitir ningún valor atípico, seleccione el percentil 99 como uso de percentil.

5. Este valor se multiplica por el factor de confort para obtener los datos de uso efectivos del rendimiento para estas métricas que recopila el dispositivo:

    - Uso de CPU
    - Uso de RAM
    - IOPS de disco (lectura y escritura)
    - Rendimiento del disco (lectura y escritura)
    - Rendimiento de la red (entrada y salida)

Se recopilan los siguientes datos de rendimiento, aunque no se usan, en las recomendaciones de dimensionamiento de las evaluaciones de AVS:
  - Los datos de IOPS y rendimiento de cada disco conectado a la máquina virtual.
  - La entrada o la salida de red de cada adaptador de red conectado a una máquina virtual.

## <a name="how-are-avs-assessments-calculated"></a>¿Cómo se calculan las evaluaciones de AVS?

Server Assessment usa los datos de rendimiento y los metadatos de las máquinas locales para calcular las evaluaciones. Si implementa el dispositivo de Azure Migrate, la evaluación usa los datos que este recopila. Sin embargo, si ejecuta una evaluación importada mediante un archivo CSV, se proporcionan los metadatos para el cálculo.

Los cálculos se producen en tres fases:

1. **Cálculo de la preparación de Azure VMware Solution (AVS)** : para ver si las máquinas virtuales locales son adecuadas para la migración a Azure VMware Solution (AVS).
2. **Cálculo del número de nodos de AVS y el uso de estos**: el número estimado de nodos de AVS necesarios para ejecutar las máquinas virtuales y el uso previsto de la CPU, la memoria y el almacenamiento en todos los nodos.
3. **Estimación del costo mensual**: los costos mensuales estimados de todos los nodos de Azure VMware Solution que ejecutan las máquinas virtuales locales.

Los cálculos se encuentran en el orden anterior. Un servidor de máquina solo se mueve a una fase posterior si supera la anterior. Por ejemplo, si un servidor no supera la fase de preparación de AVS, se marca como no adecuado para Azure. No se realizan los cálculos de dimensionamiento y costo para ese servidor.

## <a name="whats-in-an-azure-vmware-solution-avs-assessment"></a>¿En qué consiste una evaluación de Azure VMware Solution?

Esto es lo que se incluye en una evaluación de AVS en Server Assessment:


| **Propiedad** | **Detalles** 
| - | - 
| **Ubicación de destino** | Especifica la ubicación de la nube privada de AVS a la que se desea realizar la migración.<br/><br/> La evaluación de AVS en Server Assessment admite actualmente estas regiones de destino: Este de EE. UU., Oeste de Europa, Oeste de EE. UU. 
| **Tipo de almacenamiento** | Especifica el motor de almacenamiento que se va a utilizar en AVS.<br/><br/> Las evaluaciones de AVS solo admiten vSAN como tipo de almacenamiento predeterminado. 
**Instancias reservadas (RI)** | Esta propiedad le ayuda a especificar instancias reservadas de AVS. Las instancias reservadas no se admiten actualmente para los nodos de AVS. 
**Tipo de nodo** | Especifica el [tipo de nodo de AVS](../azure-vmware/concepts-private-clouds-clusters.md) que se usa para asignar las máquinas virtuales locales. El tipo de nodo predeterminado es AV36. <br/><br/> Azure Migrate le recomendará un número necesario de nodos para las máquinas virtuales que se vayan a migrar a AVS. 
**Configuración de errores tolerables, nivel de RAID** | Especifica el número de errores tolerables y las combinaciones de RAID. La opción de errores tolerables seleccionada junto con el requisito de disco de la máquina virtual local determinará el almacenamiento de vSAN total que se requiere en AVS. 
**Criterio de ajuste de tamaño** | Establece el criterio que se debe utilizar para *ajustar el tamaño* de las máquinas virtuales para AVS. Puede optar por un ajuste de tamaño *según el rendimiento* o *como en el entorno local*, sin tener en cuenta el historial de rendimiento. 
**Historial de rendimiento** | Establece la duración que se debe tener en cuenta para evaluar los datos de rendimiento de las máquinas. Esta propiedad solo es aplicable cuando el criterio de ajuste de tamaño está *basado en el rendimiento*. 
**Uso de percentil** | Especifica el valor de percentil del ejemplo de rendimiento establecido para determinar el tamaño adecuado. Esta propiedad solo es aplicable cuando el ajuste de tamaño se basa en el rendimiento.
**Factor de confort** | Azure Migrate Server Assessment tiene en cuenta un búfer (factor de confort) durante la evaluación. Dicho búfer se aplica además de los datos de uso de la máquina en las máquinas virtuales (CPU, memoria, disco y red). El factor de confort se tiene en cuenta en problemas como el uso estacional, un historial de rendimiento corto y los posibles aumentos en el uso futuro.<br/><br/> Por ejemplo, una máquina virtual de 10 núcleos con un uso del 20 % normalmente genera una máquina virtual de 2 núcleos. Sin embargo, con un factor de confort de 2.0 x, el resultado es una máquina virtual de 4 núcleos. 
**Oferta** | Muestra la [oferta de Azure](https://azure.microsoft.com/support/legal/offer-details/) en la que está inscrito. Azure Migrate calcula el costo en consecuencia.
**Moneda** | Muestra la moneda de facturación de la cuenta. 
**Descuento (%)** | Muestra cualquier descuento específico de la suscripción que recibe además de la oferta de Azure. La configuración predeterminada es 0 %. 
**Ventaja híbrida de Azure** | Especifique si dispone de Software Assurance y tiene derecho a la [Ventaja híbrida de Azure](https://azure.microsoft.com/pricing/hybrid-use-benefit/). Aunque no tiene ningún impacto en los precios de Azure VMware Solution debido a que el precio se basa en los nodos, los clientes pueden seguir utilizando las licencias de sistema operativo locales (basadas en Microsoft) en AVS mediante las ventajas híbridas de Azure. Otros proveedores de sistemas operativos de software tendrán que proporcionar sus propios términos de licencia como, por ejemplo, RHEL. 
**Suscripción excesiva de CPU virtual** | Especifica la relación entre el número de núcleos virtuales vinculados a un núcleo físico del nodo de AVS. El valor predeterminado en los cálculos es 4 vCPU: 1 núcleo físico en AVS. <br/><br/> Los usuarios de la API pueden establecer este valor como un entero. Tenga en cuenta que la suscripción excesiva de vCPU > 4:1 puede afectar a las cargas de trabajo según su uso de la CPU. 


## <a name="azure-vmware-solution-avs-suitability-analysis"></a>Análisis de idoneidad para Azure VMware Solution

Las evaluaciones de AVS en Server Assessment evalúan cada máquina virtual local para comprobar su idoneidad para AVS mediante la revisión de las propiedades de la máquina. También asigna cada máquina evaluada a una de las siguientes categorías de idoneidad:

- **Listo para AVS**: la máquina se puede migrar como está a Azure (AVS) sin realizar ningún cambio. Se iniciará en AVS con soporte técnico de AVS completo.
- **Preparado con condiciones**: Es posible que la máquina virtual tenga problemas de compatibilidad con la versión actual de vSphere y que, además, requiera herramientas de VMware y otras opciones de configuración antes de que pueda conseguirse la funcionalidad completa de la máquina virtual en AVS.
- **No está listo para AVS**: la máquina virtual no se iniciará en AVS. Por ejemplo, si la máquina virtual de VMware local tiene un dispositivo externo conectado, como un CD-ROM, se producirá un error en la operación de VMware VMotion (si se utiliza este).
- **Preparación desconocida**: Azure Migrate no ha podido encontrar la preparación de la máquina debido a una recopilación insuficiente de metadatos en el entorno local.

Server Assessment revisa las propiedades de la máquina para determinar la preparación para Azure de la máquina local.

### <a name="machine-properties"></a>Propiedades de la máquina

Server Assessment revisa la siguiente propiedad de la máquina virtual local para determinar si se puede ejecutar en Azure VMware Solution.


| **Propiedad** | **Detalles** | **Estado de preparación de AVS** 
| - | - | - 
| **Protocolo de Internet** | AVS no admite actualmente direcciones IPv6 de Internet.<br/><br/> Póngase en contacto con el equipo de MSFT AVS GBB local para obtener ayuda sobre las instrucciones de corrección si el equipo se detecta con IPv6.| Protocolo de Internet preparado condicionalmente


### <a name="guest-operating-system"></a>Sistema operativo invitado

Actualmente, las evaluaciones de AVS no revisan el sistema operativo como parte del análisis de idoneidad. Es probable que todos los sistemas operativos que se ejecutan en máquinas virtuales locales se ejecuten en Azure VMware Solution (AVS).

Junto con las propiedades de la máquina virtual, Server Assessment examina el sistema operativo invitado de las máquinas para determinar si se puede ejecutar en Azure.


## <a name="sizing"></a>Ajuste de tamaño

Una vez que una máquina está marcada como lista para AVS, la evaluación de AVS en Server Assessment realiza recomendaciones de dimensionamiento del nodo, lo cual implica la identificación de los requisitos adecuados de la máquina virtual local y la búsqueda del número total de nodos de AVS necesarios. Estas recomendaciones varían en función de las propiedades de evaluación especificadas.

- Si la evaluación utiliza el *dimensionamiento basado en el rendimiento*, Azure Migrate tiene en cuenta el historial de rendimiento de la máquina para realizar la recomendación de dimensionamiento adecuada para AVS. Este método es especialmente útil si ha asignado en exceso la máquina virtual local pero el uso es bajo, y desea ajustar adecuadamente el tamaño de la máquina virtual en AVS para ahorrar costos. Este método le ayudará a optimizar los tamaños durante la migración.
- Si no desea tener en cuenta los datos de rendimiento para el ajuste de tamaño de la máquina virtual y desea integrar las máquinas locales tal cual en AVS, puede establecer el criterio de dimensionamiento en *Como local*. Esto hará que Server Assessment ajuste el tamaño de las máquinas virtuales en función de la configuración local, sin tener en cuenta los datos de uso. 


### <a name="ftt-sizing-parameters"></a>Parámetros de ajuste de tamaño de los errores tolerables

El motor de almacenamiento que se usa en AVS es vSAN. Las directivas de almacenamiento de vSAN definen los requisitos de almacenamiento de las máquinas virtuales. Estas directivas garantizan el nivel de servicio requerido para las máquinas virtuales, ya que determinan cómo se asigna el almacenamiento a la máquina virtual. Estas son las combinaciones FTT-RAID disponibles: 

**Errores tolerables (FTT)** | **Configuración de RAID** | **Hosts mínimos requeridos** | **Consideración de dimensionamiento**
--- | --- | --- | --- 
1 | RAID-1 (Creación de reflejo) | 3 | Una máquina virtual de 100 GB consumiría 200 GB.
1 | RAID-5 (codificación de borrado) | 4 | Una máquina virtual de 100 GB consumiría 133,33 GB.
2 | RAID-1 (Creación de reflejo) | 5 | Una máquina virtual de 100 GB consumiría 300 GB.
2 | RAID-6 (codificación de borrado) | 6 | Una máquina virtual de 100 GB consumiría 150 GB.
3 | RAID-1 (Creación de reflejo) | 7 | Una máquina virtual de 100 GB consumirá 400 GB.

### <a name="performance-based-sizing"></a>Ajuste de tamaño según el rendimiento

En el ajuste de tamaño basado en el rendimiento, Server Assessment comienza con los discos conectados a la máquina virtual y sigue con los adaptadores de red. A continuación, asigna los requisitos de la máquina virtual a un número de nodos adecuado para AVS. El dispositivo de Azure Migrate realiza un perfil del entorno local para recopilar datos de rendimiento de la CPU, memoria, discos y adaptador de red.

**Pasos de la recopilación de datos de rendimiento:**

1. En el caso de las máquinas virtuales de VMware, el dispositivo de Azure Migrate recopila un punto de ejemplo en tiempo real en intervalos de 20 segundos. 
2. El dispositivo acumula los puntos de ejemplo recopilados cada 10 minutos y envía el valor máximo de los últimos 10 minutos a Server Assessment.
3. Server Assessment almacena todos los puntos de ejemplo de 10 minutos del último mes. Después, en función de las propiedades de evaluación especificadas para *Historial de rendimiento* y *Uso del percentil*, identifica el punto de datos adecuado que se debe usar para el ajuste de tamaño adecuado. Por ejemplo, si el historial de rendimiento está establecido en 1 día y el uso del percentil es 95, Server Assessment utiliza los puntos de ejemplo almacenados cada 10 minutos del último día, los clasifica en orden ascendente y selecciona el percentil 95 para el ajuste de tamaño adecuado.
4. Este valor se multiplica por el factor de confort para obtener los datos de uso de rendimiento efectivos para cada métrica (uso de CPU, uso de memoria, IOPS de disco [lectura y escritura], rendimiento de disco [lectura y escritura], rendimiento de red [entrada y salida]) que el dispositivo recopila.

Una vez determinado el valor de uso efectivo, el ajuste de tamaño del almacenamiento, la red y el proceso se determinan de la siguiente manera.

**Ajuste de tamaño de almacenamiento**: Azure Migrate usa el espacio de disco de la máquina virtual local total como parámetro de cálculo para determinar los requisitos de almacenamiento de vSAN de AVS además del valor de errores tolerables seleccionado por el cliente. FTT: el número de errores tolerables así como la necesidad de un número mínimo de nodos por opción de FTT determinarán el almacenamiento total de vSAN que se requiere junto con el requisito de disco de máquina virtual.

**Ajuste de tamaño de red**: Actualmente, Server Assessment no tiene en cuenta ninguna configuración de red para las evaluaciones de AVS.

**Ajuste de tamaño de proceso** : Después de calcular los requisitos de almacenamiento, Server Assessment tiene en cuenta los requisitos de CPU y de memoria para determinar el número de nodos necesarios para AVS según el tipo de nodo.

- Según los criterios de dimensionamiento, Server Assessment examina los datos de la máquina virtual basados en el rendimiento o la configuración de la máquina virtual local. La configuración del factor de confort permite especificar el factor de crecimiento del clúster. El hyperthreading está habilitado de forma predeterminada y, por lo tanto, un nodo de 36 núcleos tendrá 72 núcleos virtuales. Se utilizan cuatro núcleos virtuales por núcleo físico para determinar los umbrales de CPU por clúster mediante el estándar de VMware que no supera el 80 % de uso; el objetivo es permitir el mantenimiento o los errores que se van a controlar sin poner en peligro la disponibilidad del clúster. Actualmente no hay ninguna invalidación disponible para cambiar los valores de suscripción excesiva, aunque es posible que se incluya en versiones futuras.

### <a name="as-on-premises-sizing"></a>Ajuste de tamaño como local

Si usa *el ajuste de tamaño como local*, Server Assessment no tiene en cuenta el historial de rendimiento de las máquinas virtuales ni los discos. En su lugar, asigna nodos de AVS según el tamaño asignado en el entorno local. El tipo de almacenamiento predeterminado es vSAN en AVS.

## <a name="confidence-ratings"></a>Clasificaciones de confianza

Cada evaluación basada en el rendimiento de Azure Migrate se asocia a una clasificación de confianza comprendida entre una estrella (la más baja) y cinco estrellas (la más alta).

- La clasificación de confianza se asigna a una valoración que se basa en la disponibilidad de puntos de datos necesarios para calcular tal valoración.
- La clasificación de confianza de una valoración le ayuda a calcular la confiabilidad de las recomendaciones de tamaño que proporciona Azure Migrate.
- Las clasificaciones de confianza no son aplicables a evaluaciones *como en local*.
- Para dimensionamiento según el rendimiento, las evaluaciones de AVS en Server Assessment necesitan los datos de uso de la CPU y la memoria de la máquina virtual. Los siguientes datos se recopilan, pero no se usan en las recomendaciones de dimensionamiento de AVS:
  - Los datos de IOPS y rendimiento de cada disco conectado a la máquina virtual.
  - La entrada o la salida de red de cada adaptador de red conectado a una máquina virtual.

  Si alguna de las cifras de uso anteriores no está disponible en vCenter Server, la recomendación de tamaño podría no ser confiable.

A continuación se muestra la clasificación de confianza para la evaluación según el porcentaje de puntos de datos disponibles.


| **Disponibilidad de puntos de datos** | **Clasificación de confianza** |
| - | - |
| De 0 a 20 % | 1 estrella |
| De 21 a 40 % | 2 estrellas |
| De 41 a 60 % | 3 estrellas |
| De 61 a 80 % | 4 estrellas |
| De 81 a 100 % | 5 estrellas |

### <a name="low-confidence-ratings"></a>Clasificaciones de confianza bajas

Estos son algunos de los motivos por los que una evaluación puede obtener una clasificación de confianza baja:

- No ha generado un perfil de su entorno durante el tiempo que ha estado creando la evaluación. Por ejemplo, si crea la evaluación con la duración de rendimiento establecida en un día, debe esperar al menos un día después de empezar la detección para que se recopilen todos los puntos de datos.
- Se apagaron algunas máquinas virtuales en el período durante el que se calculó la valoración. Si alguna máquina virtual se apaga durante un tiempo, Server Assessment no puede recopilar los datos de rendimiento de ese período.
- Se crearon algunas máquinas virtuales en el período durante el que se calculó la valoración. Por ejemplo, si creó una evaluación para el historial de rendimiento del último mes, pero se crearon algunas máquinas virtuales en el entorno hace solo una semana, el historial de rendimiento de las nuevas máquinas virtuales no abarcará toda la duración.

> [!NOTE]
> Si la clasificación de confianza de una evaluación no llega a las cinco estrellas, se recomienda que espere al menos un día para que el dispositivo genere el perfil del entorno y luego recalcule la evaluación. En caso contrario, es posible que el ajuste de tamaño basado en el rendimiento no sea confiable. En ese caso, se recomienda cambiar la evaluación a un ajuste de tamaño local.

## <a name="monthly-cost-estimation"></a>Estimación del costo mensual

Una vez completadas las recomendaciones de dimensionamiento, Azure Migrate calcula el costo total de la ejecución de las cargas de trabajo locales en AVS multiplicando el número de nodos necesarios de AVS por el precio del nodo. El costo por máquina virtual se calcula dividiendo el costo total por el número de máquinas virtuales en la evaluación. 
- El cálculo tiene en cuenta el número de nodos necesario, el tipo de nodo y la ubicación.
- Para calcular el costo mensual total, suma el costo de todos los nodos.
- Los costos se muestran en la moneda especificada en la configuración de evaluación.

Como los precios de Azure VMware Solution (AVS) son por nodo, el costo total no tiene costo de proceso ni distribución de costos de almacenamiento. [Más información](../azure-vmware/introduction.md)

Tenga en cuenta que Azure VMware Solution está en versión preliminar, los precios de los nodos de la evaluación son los precios de la versión preliminar. Póngase en contacto con el equipo de MSFT AVS GBB local para obtener instrucciones.

## <a name="migration-tool-guidance"></a>Guía sobre la herramienta de migración

En el informe preparación para Azure para la evaluación de Azure VMware Solution (AVS), puede ver las siguientes herramientas sugeridas: 
- **VMware HCX o Enterprise** En el caso de las máquinas de VMware, la solución Hybrid Cloud Extension (HCX) de VMware es la herramienta de migración sugerida para migrar la carga de trabajo local a la nube privada de Azure VMware Solution (AVS). [Más información](../azure-vmware/hybrid-cloud-extension-installation.md).
- **Desconocido**: En el caso de las máquinas importadas mediante un archivo CSV, se desconoce la herramienta de migración predeterminada. Sin embargo, para las máquinas de VMware, se recomienda usar la solución Hybrid Cloud Extension (HCX) de VMware.

## <a name="next-steps"></a>Pasos siguientes

Cree una evaluación para [máquinas virtuales de VMware de Azure VMware Solution](how-to-create-azure-vmware-solution-assessment.md).
