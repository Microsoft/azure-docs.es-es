---
title: Valoraciones de máquinas virtuales de Azure en el Server Assessment de Azure Migrate
description: Obtenga información sobre las evaluaciones en Server Assessment de Azure Migrate.
ms.topic: conceptual
ms.date: 05/27/2020
ms.openlocfilehash: 7664c8296f0d47f37f9542dee82d3c718be40126
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87825997"
---
# <a name="azure-vm-assessments-in-azure-migrate-server-assessment"></a>Valoraciones de máquinas virtuales de Azure en Azure Migrate: Server Assessment

Este artículo sirve de introducción a las evaluaciones de la herramienta [Azure Migrate: Server Assessment](migrate-services-overview.md#azure-migrate-server-assessment-tool). La herramienta puede evaluar las máquinas virtuales de VMware locales, las máquinas virtuales de Hyper-V y los servidores físicos para migrarlos a Azure.

## <a name="whats-an-assessment"></a>¿Qué es una evaluación?

Una evaluación con la herramienta Server Assessment mide el grado de preparación de los servidores locales para la migración a Azure y calcula su impacto.

> [!NOTE]
> En Azure Government, revise las ubicaciones de evaluación de [destino compatibles](migrate-support-matrix.md#supported-geographies-azure-government). Tenga en cuenta que en las recomendaciones de tamaño de máquina virtual de las evaluaciones se usará la serie de máquina virtual específica para las regiones de la nube de administración pública. [Más información](https://azure.microsoft.com/global-infrastructure/services/?regions=usgov-non-regional,us-dod-central,us-dod-east,usgov-arizona,usgov-iowa,usgov-texas,usgov-virginia&products=virtual-machines) sobre los tamaños de máquina virtual.

## <a name="types-of-assessments"></a>Tipos de evaluaciones

Con Azure Migrate: Server Assessment se pueden crear dos Server Assessment.

**Tipo de evaluación** | **Detalles**
--- | --- 
**MV de Azure** | Evaluaciones para la migración de los servidores locales a máquinas virtuales de Azure. <br/><br/> Puede evaluar las [máquinas virtuales de VMware](how-to-set-up-appliance-vmware.md), las [máquinas virtuales de Hyper-V](how-to-set-up-appliance-hyper-v.md) y los [servidores físicos](how-to-set-up-appliance-physical.md) locales para la migración a Azure con este tipo de evaluación.
**Azure VMware Solution (AVS)** | Evaluaciones para la migración de los servidores locales a [Azure VMware Solution (AVS)](../azure-vmware/introduction.md). <br/><br/> Puede evaluar las [máquinas virtuales VMware](how-to-set-up-appliance-vmware.md) locales para la migración a Azure VMware Solution (AVS) con este tipo de evaluación. [Más información](concepts-azure-vmware-solution-assessment-calculation.md)

Las evaluaciones que se crean con Server Assessment son una instantánea puntual de los datos. Una valoración de máquinas virtuales de Azure en Server Assessment proporciona dos opciones de criterios de dimensionamiento:

**Tipo de evaluación** | **Detalles** | **Data**
--- | --- | ---
**Basada en el rendimiento** | Evaluaciones que realizan recomendaciones basadas en datos de rendimiento recopilados | La recomendación del tamaño de máquina virtual se basa en los datos de uso de CPU y memoria RAM.<br/><br/> La recomendación del tipo de disco se basa en las operaciones de entrada/salida por segundo (IOPS) y en el rendimiento de los discos locales. Los tipos de disco son Azure HDD estándar, Azure SSD estándar y Azure Premium.
**Tal cual en el entorno local** | Evaluaciones que no usan los datos de rendimiento para hacer recomendaciones | La recomendación de tamaño de máquina virtual se basa en el tamaño de la máquina virtual local.<br/><br> El tipo de disco recomendado se basa en el tipo de almacenamiento para la evaluación.

## <a name="how-do-i-run-an-assessment"></a>¿Cómo se ejecuta una evaluación?

Existen dos formas de realizar una evaluación:

- Evaluar las máquinas con los metadatos de servidor recopilados por un dispositivo de Azure Migrate ligero. El dispositivo detecta las máquinas locales. A continuación, envía continuamente los metadatos y los datos de rendimiento de las máquinas a Azure Migrate.
- Evaluar las máquinas mediante los metadatos de servidor importados en un formato de valores separados por comas (CSV).

## <a name="how-do-i-assess-with-the-appliance"></a>¿Cómo se evalúa con el dispositivo?

Haga lo siguiente si va a implementar un dispositivo de Azure Migrate para detectar servidores locales:

1. Configure Azure y el entorno local para que funcionen con Server Assessment.
1. Para realizar la primera evaluación, cree un proyecto de Azure y, luego, agregue la herramienta Server Assessment.
1. Implemente un dispositivo de Azure Migrate ligero. Este dispositivo detecta las máquinas locales de forma constante y envía a Azure Migrate los metadatos y los datos de rendimiento de las máquinas. Implemente el dispositivo como una máquina virtual o física. No es necesario instalar nada en las máquinas que quiera evaluar.

Una vez que el dispositivo inicia la detección de las máquinas, puede reunir aquellas que quiera evaluar en un grupo y realizar una valoración del grupo con tipo de valoración de **máquinas virtuales de Azure**.

Para probar estos pasos, siga nuestros tutoriales de [VMware](tutorial-prepare-vmware.md), [Hyper-V](tutorial-prepare-hyper-v.md) o [servidores físicos](tutorial-prepare-physical.md).

## <a name="how-do-i-assess-with-imported-data"></a>¿Cómo se evalúa con datos importados?

Si va a evaluar los servidores con un archivo CSV, no necesita un dispositivo. En su lugar, siga estos pasos:

1. Configure Azure para que funcione con Server Assessment.
1. Para realizar la primera evaluación, cree un proyecto de Azure y, luego, agregue la herramienta Server Assessment.
1. Descargue una plantilla CSV y agregue los datos de los servidores.
1. Importe la plantilla en Server Assessment.
1. Detecte los servidores agregados con la importación, reúnalos en un grupo y realice una valoración del grupo con tipo de valoración de **máquinas virtuales de Azure**.

## <a name="what-data-does-the-appliance-collect"></a>¿Qué datos recopila el dispositivo?

Si usa el dispositivo Azure Migrate para la evaluación, obtenga información sobre los metadatos y los datos de rendimiento que se recopilan en [VMware](migrate-appliance.md#collected-data---vmware) y [Hyper-V](migrate-appliance.md#collected-data---hyper-v).

## <a name="how-does-the-appliance-calculate-performance-data"></a>¿Cómo calcula el dispositivo los datos de rendimiento?

Si usa el dispositivo para la detección, los datos de rendimiento de la configuración de proceso se recopilan de la siguiente manera:

1. El dispositivo recopila un punto de ejemplo en tiempo real.

    - **VM de VMware**: se recopila un punto de ejemplo cada 20 segundos.
    - **VM de Hyper-V**: se recopila un punto de ejemplo cada 30 segundos.
    - **Servidores físicos**: se recopila un punto de ejemplo cada cinco minutos.

1. El dispositivo combina los puntos de ejemplo para crear un único punto de datos cada 10 minutos. Para crear el punto de datos, el dispositivo selecciona los valores máximos de todos los ejemplos. A continuación, envía el punto de datos a Azure.
1. Server Assessment almacena todos los puntos de ejemplo de 10 minutos del último mes.
1. Al crear una evaluación, Server Assessment identifica el punto de datos adecuado que se va a usar para elegir el tamaño adecuado. La identificación se basa en los valores de percentil de *historial de rendimiento* y *uso de percentil*.

    - Por ejemplo, si el historial de rendimiento es de una semana y el uso de percentil es el percentil 95, Server Assessment ordena los puntos de muestra de 10 minutos de la última semana. Los clasifica en orden ascendente y selecciona el valor de percentil 95 para la elección del tamaño adecuado.
    - El valor del percentil 95 garantiza que se van a omitir los valores atípicos, que podrían incluirse si eligiera el percentil 99.
    - Si quiere elegir el uso máximo en el período y no quiere omitir ningún valor atípico, seleccione el percentil 99 como uso de percentil.

1. Este valor se multiplica por el factor de confort para obtener los datos de uso efectivos del rendimiento para estas métricas que recopila el dispositivo:

    - Uso de CPU
    - Uso de RAM
    - IOPS de disco (lectura y escritura)
    - Rendimiento del disco (lectura y escritura)
    - Rendimiento de la red (entrada y salida)

## <a name="how-are-azure-vm-assessments-calculated"></a>¿Cómo se calculan las valoraciones de máquina virtual?

Server Assessment usa los datos de rendimiento y los metadatos de las máquinas locales para calcular las evaluaciones. Si implementa el dispositivo de Azure Migrate, la evaluación usa los datos que este recopila. Sin embargo, si ejecuta una evaluación importada mediante un archivo CSV, se proporcionan los metadatos para el cálculo.

Los cálculos se producen en tres fases:

1. **Cálculo de la preparación para Azure**: se evalúa si las máquinas son adecuadas para la migración a Azure.
1. **Cálculo de la recomendaciones de tamaño**: se calcula el tamaño de red, almacenamiento y proceso.
1. **Cálculo de los costos mensuales**: Se calculan los costos mensuales estimados de proceso y almacenamiento por ejecutar las máquinas en Azure tras la migración.

Los cálculos se encuentran en el orden anterior. Un servidor de máquina solo se mueve a una fase posterior si supera la anterior. Por ejemplo, si un servidor no supera la fase de preparación de Azure, se marca como no adecuado para Azure. No se realizan los cálculos de tamaño y costo para ese servidor.

## <a name="whats-in-an-azure-vm-assessment"></a>¿Qué es una valoración de máquinas virtuales de Azure?

Esto es lo que se incluye en una evaluación de máquinas virtuales de Azure en Server Assessment:

**Propiedad** | **Detalles**
--- | ---
**Ubicación de destino** | La ubicación de Azure a la que quiere realizar la migración. Server Assessment admite actualmente estas regiones de Azure de destino:<br/><br/> Este de Australia, Sudeste de Australia, Sur de Brasil, Centro de Canadá, Este de Canadá, Centro de la India, Centro de EE. UU., Este de China, Norte de China, Este de Asia, Este de EE. UU., Este de EE. UU. 2, Сentro de Alemania, Nordeste de Alemania, Este de Japón, Oeste de Japón, Centro de Corea del Sur, Corea del Sur, Centro y norte de EE. UU., Norte de Europa, Centro y sur de EE. UU., Sudeste de Asia, Sur de la India, Sur de Reino Unido, Oeste del Reino Unido, US Gov Arizona, US Gov Texas, US Gov Virginia, Centro-oeste de EE. UU., Oeste de Europa, Oeste de la India, Oeste de EE. UU. y Oeste de EE. UU. 2.
**Disco de almacenamiento de destino (tamaño tal cual)** | Tipo de disco que se va a usar para el almacenamiento en Azure. <br/><br/> Especifica el disco de almacenamiento de destino como administrado Premium, administrado SSD estándar o administrado HDD estándar.
**Disco de almacenamiento de destino (tamaño basado en el rendimiento)** | Especifica el tipo de disco de almacenamiento de destino como automático, administrado Premium, administrado HDD estándar o administrado SSD estándar.<br/><br/> **Automático**: la recomendación de disco se basa en los datos de rendimiento de los discos, es decir, el número de IOPS y el rendimiento.<br/><br/>**Premium o Standard**:  La evaluación recomienda una SKU de disco dentro del tipo de almacenamiento seleccionado.<br/><br/> Si quiere un contrato de nivel de servicio (SLA) de máquina virtual de una sola instancia del 99,9 %, considere el uso de discos administrados Premium. Este uso garantiza que todos los discos de la evaluación se recomienden como discos administrados Premium.<br/><br/> Azure Migrate solo admite discos administrados para la evaluación de la migración.
**Azure Reserved VM Instances** | Especifica las [instancias reservadas](https://azure.microsoft.com/pricing/reserved-vm-instances/) para que se tengan en cuenta en los cálculos de los costos de la evaluación.<br/><br/> Al seleccionar ' instancias reservadas ', el ' descuento (%) ' y las propiedades de "tiempo de actividad de la máquina virtual" no son aplicables.<br/><br/> Azure Migrate admite actualmente Azure Reserved VM Instances solo en ofertas de pago por uso.
**Criterio de tamaño** | Se usa para elegir el tamaño adecuado de la máquina virtual de Azure.<br/><br/> Use el tamaño tal cual o basado en el rendimiento.
**Historial de rendimiento** | Se usa el ajuste de tamaño basado en el rendimiento. El historial de rendimiento especifica la duración que se usa cuando se evalúan los datos de rendimiento.
**Uso de percentil** | Se usa el ajuste de tamaño basado en el rendimiento. El uso de percentil especifica el valor de percentil de la muestra de rendimiento que se usa para elegir el tamaño adecuado.
**Series de VM** | La serie de máquinas virtuales de Azure que quiere tener en cuenta para elegir el tamaño adecuado. Por ejemplo, si no tiene un entorno de producción que necesite máquinas virtuales de la serie A en Azure, puede excluir esta serie de la lista.
**Factor de confort** | Búfer usado durante la evaluación. Se aplica a los datos de uso de CPU, RAM, disco y red de las máquinas virtuales. Tiene en cuenta problemas como el uso estacional, el historial de rendimiento corto y los posibles aumentos en el uso futuro.<br/><br/> Por ejemplo, una VM de 10 núcleos con un uso del 20 % normalmente genera una VM de dos núcleos. Con un factor de confort de 2.0, el resultado es una máquina virtual de cuatro núcleos.
**Oferta** | La [oferta de Azure](https://azure.microsoft.com/support/legal/offer-details/) en la que está inscrito. Server Assessment calcula el costo de esa oferta.
**Moneda** | Moneda de facturación de la cuenta.
**Descuento (%)** | Cualquier descuento específico de la suscripción que recibe además de la oferta de Azure. La configuración predeterminada es 0 %.
**Tiempo de actividad de VM** | La duración en días por mes y horas al día de las máquinas virtuales de Azure que no se ejecutarán continuamente. Los cálculos de los costos se basan en esa duración.<br/><br/> Los valores predeterminados son 31 días al mes y 24 horas al día.
**Ventaja híbrida de Azure** | Especifique si dispone de Software Assurance y tiene derecho a la [Ventaja híbrida de Azure](https://azure.microsoft.com/pricing/hybrid-use-benefit/). Si la opción tiene el valor predeterminado "Sí", los precios de Azure para sistemas operativos que no son Windows se tienen en cuenta para las máquinas virtuales Windows.
**Suscripción a Contrato Enterprise** | Especifica que se usa una suscripción a Contrato Enterprise (EA) para la estimación de costos. Tiene en cuenta el descuento aplicable a la suscripción. <br/><br/> Deje la configuración de instancias reservadas, el descuento (%) y las propiedades de tiempo de actividad de la máquina virtual con su configuración predeterminada.


[Revise los procedimientos recomendados](best-practices-assessment.md) para crear una evaluación con Server Assessment.

## <a name="calculate-readiness"></a>Cálculo de la preparación

No todas las máquinas son adecuadas para ejecutarse en Azure. Una valoración de máquinas virtuales de Azure evalúa todas las máquinas locales y les asigna una categoría de preparación.

- **Preparada para Azure**: la máquina se puede migrar como está a Azure sin realizar ningún cambio. Se iniciará en Azure con soporte técnico de Azure completo.
- **Condicionalmente preparada para Azure**: es posible que la máquina se inicie en Azure, pero puede que no tenga soporte técnico de Azure completo. Por ejemplo, Azure no admite una máquina que ejecute una versión anterior de Windows Server. Debe tener cuidado antes de migrar estas máquinas a Azure. Para solucionar cualquier problema de preparación, siga las instrucciones de corrección que sugiere la evaluación.
- **No está preparado para Azure**: la máquina no se iniciará en Azure. Por ejemplo, si el disco de una máquina local almacena más de 64 TB, Azure no puede hospedar la máquina. Siga las instrucciones de corrección para solucionar el problema antes de la migración.
- **Preparación desconocida**: Azure Migrate no puede determinar la preparación de la máquina debido a que no hay metadatos suficientes.

Para calcular la preparación, Server Assessment revisa las propiedades de la máquina y la configuración del sistema operativo resumidas en las tablas siguientes.

### <a name="machine-properties"></a>Propiedades de la máquina

En el caso de una valoración de máquinas virtuales de Azure, Server Assessment revisa las siguientes propiedades de una máquina virtual local para determinar si se puede ejecutar en las máquinas virtuales de Azure.

Propiedad | Detalles | Estado de la preparación para Azure
--- | --- | ---
**Tipo de arranque** | Azure admite máquinas virtuales con un tipo de arranque de BIOS, no UEFI. | Condicionalmente preparada si el tipo de arranque es UEFI.
**Núcleos** | Cada máquina no debe tener más de 128 núcleos, que es el número máximo compatible que admite una máquina virtual de Azure.<br/><br/> Si el historial de rendimiento está disponible, Azure Migrate tiene en cuenta los núcleos utilizados para la comparación. Si se especifica un factor de confort en la configuración de evaluación, el número de núcleos usados se multiplica por este.<br/><br/> Si no hay ningún historial de rendimiento, Azure Migrate usa los núcleos asignados sin aplicar el factor de confort. | Preparada si el número de núcleos está dentro del límite.
**RAM** | Cada máquina no debe tener más de 3892 GB de RAM, que es el tamaño máximo que admite una máquina virtual Standard_M128m de la serie M de Azure&nbsp;<sup>2</sup>. [Más información](../virtual-machines/sizes.md).<br/><br/> Si el historial de rendimiento está disponible, Azure Migrate tiene en cuenta la RAM utilizada para la comparación. Si se especifica un factor de confort, la RAM utilizada se multiplica por este.<br/><br/> Si no hay ningún historial, se usa la RAM asignada sin aplicar el factor de confort.<br/><br/> | Preparada si la cantidad de RAM está dentro del límite.
**Disco de almacenamiento** | El tamaño asignado de un disco no debe ser superior a 32 TB. Aunque Azure admite discos de 64 TB con discos Azure SSD Ultra, Azure Migrate: Server Assessment utiliza actualmente 32 TB como límite de tamaño del disco, ya que todavía no admite SSD Ultra. <br/><br/> El número de discos conectados a la máquina debe ser 65 o menos, incluido el disco del sistema operativo. | Preparada si el tamaño y el número de los discos están dentro de los límites.
**Redes** | Una máquina no debe tener más de 32 interfaces de red (NIC) conectadas. | Preparada si el número de NIC está dentro del límite.

### <a name="guest-operating-system"></a>Sistema operativo invitado

En el caso de una valoración de máquinas virtuales de Azure, junto con la revisión de las propiedades de la máquina virtual, Server Assessment examina el sistema operativo invitado de las máquinas para determinar si se puede ejecutar en Azure.

> [!NOTE]
> Para realizar el análisis del sistema operativo invitado en las máquinas virtuales de VMware, Server Assessment usa el sistema operativo especificado para la máquina virtual en vCenter Server. Sin embargo, vCenter Server no proporciona la versión de kernel para los sistemas operativos de máquinas virtuales Linux. Para detectar la versión, debe configurar [la detección de aplicaciones](./how-to-discover-applications.md). A continuación, el dispositivo detecta información de versión mediante las credenciales de invitado que se especifican al configurar la detección de aplicaciones.


Server Assessment emplea la siguiente lógica para identificar la preparación para Azure en función del sistema operativo:

**Sistema operativo** | **Detalles** | **Estado de la preparación para Azure**
--- | --- | ---
Windows Server 2016 y todos los Service Pack | Azure proporciona compatibilidad completa. | Preparada para Azure.
Windows Server 2012 R2 y todos los Service Pack | Azure proporciona compatibilidad completa. | Preparada para Azure.
Windows Server 2012 y todos los Service Pack | Azure proporciona compatibilidad completa. | Preparada para Azure.
Windows Server 2008 R2 con todos los Service Pack | Azure proporciona compatibilidad completa.| Preparada para Azure.
Windows Server 2008 (32 bits y 64 bits) | Azure proporciona compatibilidad completa. | Preparada para Azure.
Windows Server 2003 y Windows Server 2003 R2 | Estos sistemas operativos alcanzaron la fecha de finalización del soporte técnico y necesitan un [contrato de soporte técnico personalizado (CSA)](https://aka.ms/WSosstatement) para recibir soporte técnico en Azure. | Condicionalmente preparada para Azure. Considere la posibilidad de actualizar el sistema operativo antes de migrar a Azure.
Windows 2000, Windows 98, Windows 95, Windows NT, Windows 3.1 y MS-DOS | Estos sistemas operativos han superado la fecha de finalización del soporte técnico. La máquina puede iniciarse en Azure, pero Azure no proporciona compatibilidad con el sistema operativo. | Condicionalmente preparada para Azure. Se recomienda actualizar el sistema operativo antes de migrar a Azure.
Windows 7, Windows 8 y Windows 10 | Azure solo proporciona soporte técnico con una [suscripción de Visual Studio.](../virtual-machines/windows/client-images.md) | Condicionalmente preparada para Azure.
Windows 10 Pro | Azure proporciona compatibilidad con [derechos de hospedaje multiinquilino.](../virtual-machines/windows/windows-desktop-multitenant-hosting-deployment.md) | Condicionalmente preparada para Azure.
Windows Vista y Windows XP Professional | Estos sistemas operativos han superado la fecha de finalización del soporte técnico. La máquina puede iniciarse en Azure, pero Azure no proporciona compatibilidad con el sistema operativo. | Condicionalmente preparada para Azure. Se recomienda actualizar el sistema operativo antes de migrar a Azure.
Linux | Consulte los [sistemas operativos Linux](../virtual-machines/linux/endorsed-distros.md) que Azure aprueba. Es posible que se inicien otros sistemas operativos Linux en Azure. Sin embargo, se recomienda que actualice el sistema operativo a una versión aprobada antes de migrar a Azure. | Preparada para Azure si la versión está aprobada.<br/><br/>Condicionalmente preparada si la versión no se ha aprobado.
Otros sistemas operativos, como Oracle Solaris, Apple macOS y FreeBSD. | Azure no aprueba estos sistemas operativos. La máquina puede iniciarse en Azure, pero Azure no proporciona compatibilidad con el sistema operativo. | Condicionalmente preparada para Azure. Se recomienda instalar un sistema operativo compatible antes de migrar a Azure.  
Sistema operativo especificado como **otro** en vCenter Server. | Azure Migrate no puede identificar el sistema operativo en este caso. | Preparación desconocida. Asegúrese de que Azure admite el sistema operativo que se ejecuta dentro de la máquina virtual.
Sistemas operativos de 32 bits | Es posible que la máquina se inicie en Azure, pero que Azure no proporcione soporte técnico completo. | Condicionalmente preparada para Azure. Considere la posibilidad de actualizar a un sistema operativo de 64 bits antes de migrar a Azure.

## <a name="calculating-sizing"></a>Cálculo del tamaño

Una vez que la máquina se marca como preparada para Azure, Server Assessment realiza recomendaciones de dimensionamiento en la valoración de máquinas virtuales de Azure. Estas recomendaciones identifican la máquina virtual de Azure y la SKU del disco. Los cálculos de tamaño dependen de si se usa el tamaño local tal cual o el basado en el rendimiento.

### <a name="calculate-sizing-as-is-on-premises"></a>Cálculo del ajuste de tamaño (local tal cual)

 Si usa el ajuste de tamaño local tal cual, Server Assessment no tiene en cuenta el historial de rendimiento de las máquinas virtuales ni los discos en la valoración de máquinas virtuales de Azure.

- **Ajuste de tamaño de proceso** : Server Assessment asigna una SKU de máquina virtual de Azure en función del tamaño asignado en el entorno local.
- **Tamaño de almacenamiento y disco**: Server Assessment examina el tipo de almacenamiento especificado en las propiedades de evaluación y recomienda el tipo de disco adecuado. Los posibles tipos de almacenamiento son HDD estándar, SSD estándar y Premium. El tipo de almacenamiento predeterminado es Premium.
- **Ajuste de tamaño de red**: Server Assessment considera el adaptador de red en la máquina local.

### <a name="calculate-sizing-performance-based"></a>Cálculo del ajuste de tamaño (basado en el rendimiento)

Si usa el dimensionamiento basado en el rendimiento en una valoración de máquinas virtuales de Azure, Server Assessment realiza recomendaciones de dimensionamiento de la siguiente manera:

- Server Assessment tiene en cuenta el historial de rendimiento de la máquina para identificar el tamaño y el tipo de disco de la VM en Azure.
- Si importa servidores mediante un archivo CSV, se usan los valores que especifique. Este método es especialmente útil si ha asignado en exceso la máquina local, el uso es bajo, y quiere elegir el tamaño adecuado de la máquina virtual de Azure para ahorrar costos.
- Si no quiere usar los datos de rendimiento, restablezca el criterio de ajuste de tamaño a local tal cual según se describe en la sección anterior.

#### <a name="calculate-storage-sizing"></a>Cálculo del ajuste de tamaño de almacenamiento

Para determinar el dimensionamiento del almacenamiento en una valoración de máquinas virtuales de Azure, Azure Migrate intenta asignar cada disco asociado a la máquina a un disco de Azure. El ajuste de tamaño funciona como se indica a continuación:

1. Server Assessment suma los valores de IOPS de lectura y escritura de un disco para obtener el valor total de IOPS necesario. Del mismo modo, suma los valores de rendimiento de lectura y escritura para obtener el rendimiento total de cada disco. En el caso de las evaluaciones basadas en la importación, tiene la opción de proporcionar el número total de IOPS, el rendimiento total y el número total. de discos en el archivo importado sin especificar la configuración de disco individual. Si lo hace, se omite el tamaño de cada disco y los datos proporcionados se usan directamente para calcular el tamaño y seleccionar una SKU de máquina virtual adecuada.

1. Si ha especificado el tipo de almacenamiento como automático, el tipo seleccionado se basa en los valores efectivos de IOPS y rendimiento. Server Assessment determina si se debe asignar el disco a un disco HDD estándar, SSD estándar o Premium de Azure. Si el tipo de almacenamiento se establece en uno de esos tipos de disco, Server Assessment intenta encontrar una SKU de disco en el tipo de almacenamiento seleccionado.
1. Los discos se seleccionan de la siguiente manera:
    - Si Server Assessment no puede encontrar un disco con los valores de rendimiento e IOPS necesarios, marca la máquina como no adecuada para Azure.
    - Si Server Assessment encuentra un conjunto de discos adecuados, selecciona aquellos que admiten la ubicación especificada en la configuración de evaluación.
    - Si hay varios discos coincidentes, Server Assessment selecciona el disco con el costo más bajo.
    - Si no están disponibles los datos de rendimiento de ningún disco, se usa el tamaño de disco de la configuración para buscar un disco SSD estándar en Azure.

#### <a name="calculate-network-sizing"></a>Cálculo del ajuste de tamaño de red

En el caso de una valoración de máquinas virtuales de Azure, Server Assessment intenta encontrar una máquina virtual de Azure que admita el número y el rendimiento requerido de los adaptadores de red conectados a la máquina local.

- Para obtener el rendimiento de red efectivo de la máquina virtual local, Server Assessment suma la velocidad de transmisión de los datos fuera de la máquina (salida de red) entre todos los adaptadores de red. Luego, aplica el factor de confort. El valor resultante se usa para buscar una máquina virtual de Azure que pueda cumplir el rendimiento de red requerido.
- Junto con el rendimiento de red, Server Assessment también tiene en cuenta si la máquina virtual de Azure puede admitir el número de adaptadores de red necesarios.
- Si no hay datos de rendimiento de red disponibles, Server Assessment solo tiene en cuenta el número de adaptadores de red para determinar el tamaño de la máquina virtual.

#### <a name="calculate-compute-sizing"></a>Cálculo del ajuste de tamaño de proceso

Después de calcular los requisitos de almacenamiento y red, Server Assessment tiene en cuenta los requisitos de CPU y memoria para encontrar un tamaño de máquina virtual adecuado en Azure.

- Azure Migrate examina los núcleos y la memoria RAM utilizados efectivos para encontrar un tamaño de máquina virtual adecuado de Azure.
- Si no encuentra ningún tamaño adecuado, la máquina se marca como no adecuada para Azure.
- Si encuentra un tamaño adecuado, Azure Migrate aplica los cálculos de red y almacenamiento. Después, aplica la configuración de ubicación y plan de tarifa para realizar la recomendación final de tamaño de máquina virtual.
- Si hay varios tamaños de máquina virtual de Azure adecuados, se recomienda el que tiene el costo más bajo.

## <a name="confidence-ratings-performance-based"></a>Clasificaciones de confianza (basadas en el rendimiento)

Cada valoración basada en el rendimiento de Azure en Azure Migrate está asociada a una clasificación de confianza. La clasificación oscila entre una (más baja) y cinco estrellas (más alta). La clasificación de confianza sirve de ayuda para calcular la confiabilidad de las recomendaciones de tamaño que proporciona Azure Migrate.

- La clasificación de confianza se asigna a una evaluación. Esta clasificación se basa en la disponibilidad de los puntos de datos necesarios para calcular tal evaluación.
- Para ajustes de tamaño según el rendimiento, Server Assessment necesita lo siguiente:
    - Los datos de uso de la CPU y la RAM de la máquina virtual.
    - Los datos de IOPS y rendimiento de cada disco conectado a la máquina virtual.
    - La entrada o la salida de red de cada adaptador de red conectado a una máquina virtual.

Si alguna de las cifras de uso anteriores no está disponible, las recomendaciones de tamaño podrían no ser confiables.

> [!NOTE]
> No se asignan clasificaciones de confianza en los servidores evaluados con un archivo CSV importado. Las clasificaciones tampoco son aplicables a las evaluaciones locales tal cual.

### <a name="ratings"></a>Clasificaciones

En esta tabla se muestran las clasificaciones de confianza de la evaluación, que dependen del porcentaje de puntos de datos disponibles:

   **Disponibilidad de puntos de datos** | **Clasificación de confianza**
   --- | ---
   De 0 a 20 % | 1 estrella
   De 21 a 40 % | 2 estrellas
   De 41 a 60 % | 3 estrellas
   De 61 a 80 % | 4 estrellas
   De 81 a 100 % | 5 estrellas

### <a name="low-confidence-ratings"></a>Clasificaciones de confianza bajas

Estos son algunos de los motivos por los que una evaluación puede obtener una clasificación de confianza baja:

- No ha generado un perfil de su entorno durante el tiempo que ha estado creando la evaluación. Por ejemplo, si crea la evaluación con la duración de rendimiento establecida en un día, debe esperar al menos un día después de empezar la detección para que se recopilen todos los puntos de datos.
- Se apagaron algunas máquinas virtuales en el período durante el que se calculó la evaluación. Si alguna máquina virtual se apaga durante un tiempo, Server Assessment no puede recopilar los datos de rendimiento de ese período.
- Se crearon algunas máquinas virtuales en el período durante el que se calculó la evaluación. Por ejemplo, supongamos que ha creado una evaluación para el historial de rendimiento del último mes, pero algunas máquinas virtuales se han creado hace solo una semana. El historial de rendimiento de las nuevas máquinas virtuales no existirá durante todo el tiempo.

> [!NOTE]
> Si la clasificación de confianza de una evaluación no llega a las cinco estrellas, se recomienda que espere al menos un día para que el dispositivo genere el perfil del entorno y luego recalcule la evaluación. De lo contrario, el tamaño basado en el rendimiento podría ser poco confiable. En ese caso, se recomienda cambiar la evaluación a un ajuste de tamaño local.

## <a name="calculate-monthly-costs"></a>Cálculo de los costos mensuales

Una vez completadas las recomendaciones de dimensionamiento, una valoración de máquinas virtuales de Azure en Azure Migrate calcula los costos de almacenamiento y proceso para después de la migración.

- **Costo de proceso**: Azure Migrate usa el tamaño recomendado de máquina virtual de Azure y Azure Billing API para calcular el costo mensual de la máquina virtual.

    En el cálculo se tiene en cuenta lo siguiente:
    - Sistema operativo
    - Software Assurance
    - Instancias reservadas
    - Tiempo de actividad de VM
    - Location
    - Configuración de moneda

    Para calcular el costo de proceso mensual total, Server Assessment suma el costo de todas las máquinas.

- **Costo de almacenamiento**: el costo de almacenamiento mensual de una máquina se calcula sumando el costo mensual de todos los discos conectados a la máquina.

    Server Assessment calcula los costos de almacenamiento mensuales totales sumando los costos de almacenamiento de todas las máquinas. Actualmente, el cálculo no tiene en cuenta las ofertas especificadas en la configuración de evaluación.

Los costos se muestran en la moneda especificada en la configuración de evaluación.

## <a name="next-steps"></a>Pasos siguientes

[Revise](best-practices-assessment.md) los procedimientos recomendados para crear evaluaciones. 

- Obtenga información sobre la ejecución de evaluaciones para [VM de VMware](tutorial-prepare-vmware.md), [VM de Hyper-V](tutorial-prepare-hyper-v.md) y [servidores físicos](tutorial-prepare-physical.md).
- Obtenga información sobre la evaluación de servidores [importados con un archivo CSV](tutorial-assess-import.md).
- Información acerca de la configuración de [visualización de dependencias](concepts-dependency-visualization.md).