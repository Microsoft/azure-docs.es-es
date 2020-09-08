---
title: 'Azure Premium Storage: Diseño para alto rendimiento'
description: Diseñe aplicaciones de alto rendimiento mediante discos administrados SSD Prémium de Azure. Premium Storage le ofrece compatibilidad con discos de alto rendimiento y baja latencia para cargas de trabajo con un uso intensivo de E/S, que se ejecutan en Azure Virtual Machines.
author: roygara
ms.service: virtual-machines
ms.topic: conceptual
ms.date: 06/27/2017
ms.author: rogarana
ms.subservice: disks
ms.openlocfilehash: a13fa7c819dcccc101c23015214bac55d2ab26c9
ms.sourcegitcommit: b33c9ad17598d7e4d66fe11d511daa78b4b8b330
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88855548"
---
# <a name="azure-premium-storage-design-for-high-performance"></a>Azure Premium Storage: diseño de alto rendimiento

Este artículo proporciona instrucciones para crear aplicaciones de alto rendimiento con Azure Premium Storage. Puede usar las instrucciones proporcionadas en este documento junto con los procedimientos recomendados de rendimiento aplicables a las tecnologías usadas por la aplicación. Para ilustrar las directrices, hemos usado SQL Server en Premium Storage como ejemplo en este documento.

Si bien en este artículo se tratan los escenarios de rendimiento de la capa de almacenamiento, deberá optimizar la capa de la aplicación. Por ejemplo, si hospeda una granja de SharePoint en Azure Premium Storage, puede usar los ejemplos de SQL Server de este artículo para optimizar el servidor de bases de datos. Además, optimice el servidor web y el servidor de aplicaciones de la granja de SharePoint para obtener el máximo rendimiento.

Este artículo le ayudará a responder a las siguientes preguntas habituales acerca de cómo optimizar el rendimiento de las aplicaciones en Azure Premium Storage:

* ¿Cómo medir el rendimiento de las aplicaciones?  
* ¿Por qué no se ve el alto rendimiento esperado?  
* ¿Qué factores influyen en el rendimiento de las aplicaciones en Premium Storage?  
* ¿Cómo influyen estos factores en el rendimiento de las aplicaciones en Premium Storage?  
* ¿Cómo puede optimizar para IOPS, el ancho de banda y la latencia?  

Proporcionamos estas directrices específicamente para Premium Storage porque las cargas de trabajo que se ejecutan en Premium Storage dependen mucho del rendimiento. Se proporcionan ejemplos donde corresponda. También puede aplicar algunas de estas instrucciones a las aplicaciones que se ejecutan en máquinas virtuales de IaaS con discos de Standard Storage.

> [!NOTE]
> A veces, lo que parece ser un problema de rendimiento del disco es realmente un cuello de botella de red. En estos casos, debería optimizarse el [rendimiento de la red](../virtual-network/virtual-network-optimize-network-bandwidth.md).
>
> Si desea realizar un banco de pruebas del disco, consulte nuestros artículos sobre cómo ejecutar bancos de pruebas de un disco:
>
> * Para Linux: [Ejecución de un banco de pruebas de la aplicación en Azure Disk Storage](linux/disks-benchmarks.md).
> * Para Windows: [Ejecución de un banco de pruebas de un disco](windows/disks-benchmarks.md).
>
> Si la VM admite redes aceleradas, debe asegurarse de que esta opción esté habilitada. Si no está habilitada, puede habilitarla en VM ya implementadas, tanto en [Windows](../virtual-network/create-vm-accelerated-networking-powershell.md#enable-accelerated-networking-on-existing-vms) como en [Linux](../virtual-network/create-vm-accelerated-networking-cli.md#enable-accelerated-networking-on-existing-vms).

Antes de comenzar, si es la primera vez que usa Premium Storage, lea antes los artículos acerca de la [Selección de un tipo de disco para máquinas virtuales IaaS de Azure](disks-types.md) y [Objetivos de escalabilidad de las cuentas de almacenamiento de blob en páginas Premium](../storage/blobs/scalability-targets-premium-page-blobs.md).

## <a name="application-performance-indicators"></a>Indicadores del rendimiento de las aplicaciones

Evaluamos si una aplicación tiene un buen rendimiento o no mediante indicadores de rendimiento como: la rapidez con la que una aplicación procesa una solicitud del usuario, la cantidad de datos que una aplicación procesa por solicitud, cuántas solicitudes procesa una aplicación en un determinado período, cuánto tiempo tiene que esperar un usuario para obtener una respuesta después de enviar su solicitud. Los términos técnicos para estos indicadores de rendimiento son IOPS, latencia y rendimiento o ancho de banda.

En esta sección, trataremos los indicadores de rendimiento comunes en el contexto de Premium Storage. En la siguiente sección, Reunión de los requisitos de rendimiento de las aplicaciones, obtendrá información sobre cómo medir estos indicadores de rendimiento de las aplicaciones. Más adelante, en Optimización del rendimiento de las aplicaciones, obtendrá información acerca de los factores que afectan a estos indicadores de rendimiento y recomendaciones para optimizarlos.

## <a name="iops"></a>E/S

IOPS, o el número de operaciones de E/S por segundo, es el número de solicitudes que la aplicación envía a los discos de almacenamiento en un segundo. Una operación de entrada y salida puede ser de lectura o escritura, secuencial o aleatoria. Las aplicaciones de procesamiento de transacciones en línea (OLTP), como un sitio web de venta directa en línea, necesitan procesar muchas solicitudes de usuario simultáneas inmediatamente. Las solicitudes de usuario suponen insertar y actualizar transacciones con un uso intensivo de las bases de datos y que la aplicación debe procesar rápidamente. Por lo tanto, las aplicaciones OLTP requieren IOPS muy alta. Dichas aplicaciones controlan millones de solicitudes de E/S pequeñas y aleatorias. Si tiene este tipo de aplicación, debe diseñar la infraestructura de aplicaciones para optimizar la IOPS. En la sección siguiente, *Optimización del rendimiento de las aplicaciones*, analizaremos en detalle todos los factores que debe tener en cuenta para obtener una IOPS alta.

Cuando conecte un disco de Almacenamiento premium a la máquina virtual a gran escala, Azure aprovisiona automáticamente un número garantizado de IOPS según la especificación del disco. Por ejemplo, un disco P50 aprovisiona 7500 IOPS. Cada tamaño de máquina virtual a gran escala también tiene un límite de IOPS específico que puede admitir. Por ejemplo, una máquina virtual GS5 estándar tiene un límite de 80.000 IOPS.

## <a name="throughput"></a>Throughput

El rendimiento o ancho de banda es la cantidad de datos que la aplicación envía a los discos de almacenamiento en un intervalo especificado. Si la aplicación está realizando operaciones de entrada y salida con tamaños de unidad de E/S grandes, requiere un alto rendimiento. Las aplicaciones de almacenamiento de datos tienden a emitir operaciones con un uso intensivo de análisis que acceden a grandes porciones de datos a la vez y suelen llevar a cabo operaciones masivas. En otras palabras, estas aplicaciones requieren un mayor rendimiento. Si tiene este tipo de aplicación, debe diseñar su infraestructura para optimizar el rendimiento. En la sección siguiente, analizamos en detalle los factores que se deben optimizar para lograrlo.

Cuando conecte un disco de Premium Storage a una máquina virtual a gran escala, Azure aprovisiona el rendimiento según la especificación del disco. Por ejemplo, un disco P50 aprovisiona 250 MB por capacidad de proceso del segundo disco. Cada tamaño de máquina virtual a gran escala también tiene un límite de rendimiento específico que puede admitir. Por ejemplo, la máquina virtual GS5 estándar tiene un rendimiento máximo de 2.000 MB por segundo.

Hay una relación entre el rendimiento y el número de IOPS, tal como se muestra en la siguiente fórmula.

![Relación entre IOPS y el rendimiento](linux/media/premium-storage-performance/image1.png)

Por lo tanto, es importante determinar los valores óptimos de rendimiento y de IOPS que requiere una aplicación. Si intenta optimizar uno, el otro también se ve afectado. En una sección posterior, *Optimización del rendimiento de las aplicaciones*, trataremos con más detalle cómo optimizar el rendimiento y la IOPS.

## <a name="latency"></a>Latencia

La latencia es el tiempo que tarda una aplicación en recibir una sola solicitud, enviarla a los discos de almacenamiento y enviar la respuesta al cliente. Se trata de una medida crítica del rendimiento de una aplicación, además de la IOPS y el rendimiento. La latencia de un disco de almacenamiento premium es el tiempo que tarda en recuperar la información de una solicitud y comunicarla de nuevo a la aplicación. Premium Storage proporciona latencias bajas coherentes. Los discos Premium están diseñados para proporcionar latencias de milisegundos de un solo dígito en la mayoría de las operaciones de E/S. Si habilita el almacenamiento en caché de host ReadOnly en discos de almacenamiento premium, puede obtener una latencia de lectura mucho menor. Trataremos el almacenamiento en caché de disco con más detalle en la sección *Optimización del rendimiento de las aplicaciones*.

Cuando se optimiza la aplicación para obtener un valor de IOPS y un rendimiento mayores, afectará a su latencia. Después de ajustar el rendimiento de las aplicaciones, siempre se evalúa la latencia de la aplicación para evitar un comportamiento inesperado con alta latencia.

Las siguientes operaciones de plano de control en Managed Disks pueden implicar el movimiento del disco de una ubicación de almacenamiento a otra. Esto se orquesta mediante una copia en segundo plano de los datos que puede tardar varias horas en completarse, normalmente menos de 24 horas en función de la cantidad de datos en los discos. Durante ese tiempo, la aplicación puede experimentar una latencia de lectura más alta de lo habitual, ya que algunas lecturas pueden redirigirse a la ubicación original y pueden tardar más tiempo en completarse. No hay ningún impacto en la latencia de escritura durante este período.

- Actualizar el tipo de almacenamiento.
- Asociar o desasociar un disco de una VM a otra.
- Crear un disco administrado a partir de un VHD.
- Crear un disco administrado a partir de una instantánea.
- Convertir discos no administrados en discos administrados.

## <a name="performance-application-checklist-for-disks"></a>Lista de comprobación del rendimiento de las aplicaciones para los discos

El primer paso para diseñar aplicaciones de alto rendimiento que se ejecutan en Azure Premium Storage es entender los requisitos de rendimiento de las aplicaciones. Después de reunir los requisitos de rendimiento, puede optimizar la aplicación para lograr un rendimiento óptimo.

En la sección anterior, explicamos los indicadores de rendimiento comunes: IOPS, rendimiento y latencia. Debe identificar cuál de estos indicadores de rendimiento son fundamentales para que la aplicación proporcione la experiencia de usuario deseada. Por ejemplo, una IOPS alta es más importante para las aplicaciones OLTP que procesan millones de transacciones en un segundo. Por otra parte, un alto rendimiento es fundamental para las aplicaciones de Almacenamiento de datos que procesan grandes cantidades de datos en un segundo. Una latencia extremadamente baja es fundamental para las aplicaciones en tiempo real, como sitios web de streaming de vídeo en directo.

A continuación, mida los requisitos para obtener el máximo rendimiento de sus aplicaciones a lo largo de toda su duración. Use la siguiente lista de comprobación de ejemplo como punto de partida. Registre los requisitos para obtener el máximo rendimiento durante períodos de carga de trabajo normales, pico y valle. Al identificar los requisitos para todos los niveles de carga de trabajo, podrá determinar los requisitos de rendimiento generales de la aplicación. Por ejemplo, la carga de trabajo normal de un sitio web de comercio electrónico serán las transacciones atendidas durante la mayoría de los días en un año. La carga de trabajo máxima del sitio web serán las transacciones atendidas durante la temporada de vacaciones o eventos de ventas especiales. La carga de trabajo máxima normalmente se experimenta durante un período limitado, pero puede que la aplicación deba escalar su funcionamiento normal de dos o más veces. Descubra los requisitos de percentil 50, percentil 90 y percentil 99. Esto ayuda a filtrar los valores atípicos en los requisitos de rendimiento, por lo que puede centrar sus esfuerzos en optimizar para los valores correctos.

## <a name="application-performance-requirements-checklist"></a>Lista de comprobación de requisitos de rendimiento de las aplicaciones

| **Requisitos de rendimiento** | **Percentil 50** | **Percentil 90** | **Percentil 99** |
| --- | --- | --- | --- |
| Máx. Transacciones por segundo | | | |
| Porcentaje de operaciones de lectura | | | |
| Porcentaje de operaciones de escritura | | | |
| Porcentaje de operaciones aleatorias | | | |
| Porcentaje de operaciones secuenciales | | | |
| Tamaño de las solicitudes de E/S | | | |
| Rendimiento medio | | | |
| Máx. Throughput | | | |
| Mín. Latencia | | | |
| Latencia media | | | |
| Máx. CPU | | | |
| Promedio de CPU | | | |
| Máx. Memoria | | | |
| Promedio de memoria | | | |
| Profundidad de la cola | | | |

> [!NOTE]
> Debe considerar la posibilidad de escalar estos números en función del crecimiento futuro previsto de la aplicación. Es buena idea para planificar el crecimiento antes de tiempo, porque podría ser más difícil cambiar la infraestructura para mejorar el rendimiento más adelante.

Si tiene una aplicación existente y desea cambiar a Premium Storage, primero prepare la lista de comprobación anterior para la aplicación existente. A continuación, cree un prototipo de la aplicación en Premium Storage y diseñe la aplicación de acuerdo con las directrices descritas en *Optimización del rendimiento de las aplicaciones* , en una sección posterior de este documento. En el siguiente artículo se describen las herramientas que puede usar para recopilar las mediciones de rendimiento.

### <a name="counters-to-measure-application-performance-requirements"></a>Contadores para medir los requisitos de rendimiento de las aplicaciones

La mejor forma de medir los requisitos de rendimiento de las aplicaciones es usar las herramientas de supervisión del rendimiento proporcionadas por el sistema operativo del servidor. Puede usar PerfMon para Windows e iostat para Linux. Estas herramientas capturan contadores correspondientes a cada medida explicada en la sección anterior. Debe capturar los valores de estos contadores cuando la aplicación funciona con cargas de trabajo normales, pico y valle.

Los contadores de rendimiento están disponibles para el procesador y la memoria, así como en cada disco lógico y físico del servidor. Al usar discos de Almacenamiento premium con una máquina virtual, los contadores del disco físico son para cada disco de Almacenamiento premium y los contadores del disco lógico son para cada volumen creado en los discos de Almacenamiento premium. Debe capturar los valores de los discos que hospedan la carga de trabajo de la aplicación. Si hay una asignación uno a uno entre los discos lógicos y físicos, puede hacer referencia a los contadores del disco físico; de lo contrario, haga referencia a los contadores del disco lógico. En Linux, el comando iostat genera un informe de uso de CPU y disco. El informe de uso del disco proporciona estadísticas por cada dispositivo físico o partición. Si tiene un servidor de bases de datos con sus datos y registros en discos independientes, recopile estos datos para ambos discos. En la tabla siguiente se describen los contadores para discos, procesadores y memoria:

| Contador | Descripción | PerfMon | Iostat |
| --- | --- | --- | --- |
| **E/S por segundo o transacciones por segundo** |Número de solicitudes de E/S emitidas en el disco de almacenamiento por segundo. |Lecturas de disco/s <br> Escrituras en disco/s |tps <br> r/s <br> w/s |
| **Escrituras y lecturas de disco** |Porcentaje de operaciones de lectura y escritura realizadas en el disco. |% de tiempo de lectura de disco <br> % de tiempo de escritura de disco |r/s <br> w/s |
| **Rendimiento** |Cantidad de datos que se leen o escriben en el disco por segundo. |Bytes de lectura de disco/s <br> Bytes de escritura en disco/s |kB_read/s <br> kB_wrtn/s |
| **Latency** |Tiempo total para completar una solicitud de E/S del disco. |Promedio de segundos de disco/lectura <br> Promedio de segundos de disco/escritura |await <br> svctm |
| **Tamaño de E/S** |El tamaño de E/S las solicitudes en los discos de almacenamiento. |Promedio de bytes de disco/lectura <br> Promedio de bytes de disco/escritura |avgrq-sz |
| **Profundidad de la cola** |Número de solicitudes de E/S pendientes de lectura de o escritura en el disco de almacenamiento. |Current Disk Queue Length |avgqu-sz |
| **Máx. memoria** |Cantidad de memoria necesaria para ejecutar la aplicación sin problemas |% de bytes confirmados en uso |Use vmstat |
| **Máx. CPU** |Cantidad de CPU necesaria para ejecutar la aplicación sin problemas |% de tiempo de procesador |%util |

Obtenga más información sobre [iostat](https://linux.die.net/man/1/iostat) y [PerfMon](https://msdn.microsoft.com/library/aa645516.aspx).



## <a name="optimize-application-performance"></a>Optimización del rendimiento de las aplicaciones

Los principales factores que influyen en el rendimiento de una aplicación que se ejecuta en Premium Storage son la naturaleza de las solicitudes de E/S, el tamaño de la máquina virtual, el tamaño del disco, el número de discos, la caché de disco, el multithreading y la profundidad de la cola. Puede controlar algunos de estos factores con mecanismos proporcionados por el sistema. Es posible que la mayoría de las aplicaciones no le de opción de modificar el tamaño de E/S y la profundidad de la cola directamente. Por ejemplo, si usa SQL Server, no puede elegir la profundidad de la cola y el tamaño de E/S. SQL Server selecciona los valores de tamaño de E/S y profundidad de la cola óptimos para obtener el máximo rendimiento. Es importante comprender los efectos de ambos tipos de factores en rendimiento de su aplicación para poder aprovisionar los recursos adecuados para satisfacer las necesidades de rendimiento.

En esta sección, consulte la lista de comprobación de los requisitos de la aplicación que creó para averiguar la cantidad que necesita para optimizar el rendimiento de las aplicaciones. En función de ello, podrá determinar qué factores de esta sección debe optimizar. Para ver los efectos de cada factor en el rendimiento de las aplicaciones, ejecute las herramientas de pruebas comparativas en la configuración de su aplicación. Vea el artículo Pruebas comparativas haciendo clic en el vínculo que aparece al final de este documento para conocer los pasos para ejecutar las herramientas de pruebas comparativas comunes en las máquinas virtuales de Windows y de Linux.

### <a name="optimize-iops-throughput-and-latency-at-a-glance"></a>Optimización de IOPS, rendimiento y latencia de un vistazo

En la tabla siguiente se resumen los factores de rendimiento y los pasos necesarios para optimizar la IOPS, el rendimiento y la latencia. Las secciones que siguen a este resumen describen cada factor con mucha más profundidad.

Para obtener más información sobre los tamaños de máquina virtual y la IOPS, el rendimiento y la latencia disponibles para cada tipo de máquina virtual, vea [Tamaños de máquinas virtuales en Azure](sizes.md).

| | **E/S** | **Rendimiento** | **Latency** |
| --- | --- | --- | --- |
| **Escenario de ejemplo** |Aplicación OLTP empresarial que requiere una tasa muy alta de transacciones por segundo. |Aplicación de Almacenamiento de datos de empresa que procesa grandes cantidades de datos. |Aplicaciones casi en tiempo real que requieren respuestas instantáneas a solicitudes de usuario, como juegos en línea. |
| **Factores de rendimiento** | &nbsp; | &nbsp; | &nbsp; |
| **Tamaño de E/S** |Un tamaño de E/S menor produce una mayor IOPS. |Un tamaño de E/S mayor produce un mayor rendimiento. | &nbsp;|
| **Tamaño de VM** |Use un tamaño de máquina virtual que ofrezca una IOPS mayor que los requisitos de la aplicación. |Use un tamaño de máquina virtual con un límite de rendimiento mayor que los requisitos de la aplicación. |Use un tamaño de máquina virtual que ofrezca una escala de límites mayor que los requisitos de la aplicación. |
| **Tamaño del disco** |Use un tamaño de disco que ofrezca una IOPS mayor que los requisitos de la aplicación. |Use un tamaño de disco con un límite de rendimiento mayor que los requisitos de la aplicación. |Use un tamaño de disco que ofrezca una escala de límites mayor que los requisitos de la aplicación. |
| **Máquina virtual y límites de escala de disco** |El límite de IOPS del tamaño de la máquina virtual elegido debe ser mayor que la IOPS total controlada por los discos de almacenamiento conectados. |El límite de rendimiento del tamaño de la máquina virtual elegido debe ser mayor que el rendimiento total controlado por los discos de almacenamiento premium conectados. |Los límites de la escala del tamaño de la máquina virtual elegidos deben ser mayores que los límites de escala total de los discos de almacenamiento premium conectados. |
| **Almacenamiento en caché de disco** |Habilite la caché de solo lectura en los discos de almacenamiento premium con operaciones intensivas de lectura para obtener una mayor IOPS de lectura. | &nbsp; |Habilite la caché de solo lectura en los discos de almacenamiento premium con operaciones intensivas de lectura para obtener latencias de lectura muy bajas. |
| **Seccionamiento del disco** |Use varios discos y secciónelos conjuntamente para conseguir un límite de IOPS y rendimiento combinado superior. El límite combinado por máquina virtual debe ser mayor que los límites combinados de los discos premium conectados. | &nbsp; | &nbsp; |
| **Tamaño de franja** |Un menor tamaño de franja para un patrón de E/S pequeño y aleatorio visto en las aplicaciones OLTP. Por ejemplo, puede usar un tamaño de franja de 64 KB para aplicaciones OLTP de SQL Server. |Un mayor tamaño de franja para un patrón de E/ grande y secuencial visto en las aplicaciones de Almacenamiento de datos. Por ejemplo, puede usar un tamaño de franja de 256 KB para aplicaciones de Almacenamiento de datos de SQL Server. | &nbsp; |
| **Multithreading** |Use el multithreading para insertar un mayor número de solicitudes en Premium Storage, lo que dará lugar a una mayor IOPS y rendimiento. Por ejemplo, en SQL Server establezca un valor MAXDOP alto para asignar más CPU a SQL Server. | &nbsp; | &nbsp; |
| **Profundidad de la cola** |Una profundidad de la cola mayor produce una IOPS mayor. |Una profundidad de la cola mayor produce un mayor rendimiento. |Una profundidad de la cola menor produce latencias más bajas. |

## <a name="nature-of-io-requests"></a>Naturaleza de las solicitudes de E/S

Una solicitud de E/S es una unidad de operación de entrada y salida que la aplicación va a realizar. La identificación de la naturaleza de las solicitudes de E/S, aleatorias o secuenciales, lectura o escritura, pequeñas o grandes, ayudará a determinar los requisitos de rendimiento de la aplicación. Es importante comprender la naturaleza de las solicitudes de E/S para tomar las decisiones correctas al diseñar la infraestructura de las aplicaciones. IOs se debe distribuir uniformemente para lograr el mejor rendimiento posible.

El tamaño de E/S es uno de los factores más importantes. El tamaño de E/S es el tamaño de la solicitud de operación de entrada/salida generada por la aplicación. El tamaño de E/S tiene una repercusión considerable en el rendimiento, especialmente en la IOPS y el ancho de banda que la aplicación es capaz de lograr. La fórmula siguiente muestra la relación entre IOPS, tamaño de E/S y ancho de banda y rendimiento.  
    ![Un diagrama que muestra la ecuación de IOPS multiplicado por el tamaño de IO es igual al rendimiento.](media/premium-storage-performance/image1.png)

Algunas aplicaciones permiten modificar su tamaño de E/S, mientras que otras aplicaciones no lo permiten. Por ejemplo, SQL Server determina el tamaño de E/S óptimo en sí; no proporciona a los usuarios ningún botón para cambiarlo. Por otro lado, Oracle proporciona un parámetro llamado [DB\_BLOCK\_SIZE](https://docs.oracle.com/cd/B19306_01/server.102/b14211/iodesign.htm#i28815) con el que puede configurar el tamaño de la solicitud de E/S de la base de datos.

Si usa una aplicación que no permite cambiar el tamaño de E/S, use las directrices de este artículo para optimizar el KPI de rendimiento que es más importante para su aplicación. Por ejemplo,

* Una aplicación OLTP genera millones de solicitudes de E/S pequeñas y aleatorias. Para controlar estos tipos de solicitudes de E/S, debe diseñar la infraestructura de la aplicación para obtener una mayor IOPS.  
* Una aplicación de almacenamiento de datos genera solicitudes de E/S grandes y secuenciales. Para controlar estos tipos de solicitudes de E/S, debe diseñar la infraestructura de la aplicación para obtener el mayor ancho de banda o el rendimiento.

Si usa una aplicación que le permite cambiar el tamaño de E/S, use esta regla general para el tamaño de E/S, además otras directrices de rendimiento.

* Un tamaño de E/S menor para obtener una mayor IOPS. Por ejemplo, 8 KB para una aplicación OLTP.  
* Un tamaño de E/S mayor para obtener un mayor ancho de banda y rendimiento. Por ejemplo, 1024 KB para una aplicación de Almacenamiento de datos.

Este es un ejemplo de cómo calcular la IOPS y el ancho de banda y el rendimiento de la aplicación. Considere una aplicación con un disco P30. El máximo rendimiento/ancho de banda e IOPS que un disco P30 puede lograr es 200 MB por segundo y 5000 IOPS respectivamente. Ahora, si la aplicación requiere la IOPS máxima en el disco P30 y usa un tamaño de E/S más pequeño, como 8 KB, el ancho de banda resultante que podrá obtener es de 40 MB por segundo. Sin embargo, si la aplicación requiere el máximo rendimiento/ancho de banda del disco P30 y usa un tamaño de E/S mayor, como 1024 KB, el número de IOPS resultante será menor, 200 IOPS. Por consiguiente, ajuste el tamaño de E/S para que cumpla los requisitos de IOPS y ancho de banda y rendimiento de la aplicación. En la siguiente tabla se resumen los distintos tamaños de E/S, así como sus IOPS y rendimiento correspondientes para un disco P30.

| Requisito de la aplicación | Tamaño de E/S | E/S | Rendimiento/ancho de banda |
| --- | --- | --- | --- |
| IOPS máx. |8 KB |5\.000 |40 MB por segundo |
| Rendimiento máx. |1024 KB |200 |200 MB por segundo |
| Rendimiento máx.+ IOPS alta |64 KB |3\.200 |200 MB por segundo |
| IOPS máx. + alto rendimiento |32 KB |5\.000 |160 MB por segundo |

Para obtener una IOPS y un ancho de banda mayores que el valor máximo de un solo disco de almacenamiento premium, use varios discos premium seccionados conjuntamente. Por ejemplo, seccione dos discos P30 para obtener una IOPS combinada de 10.000 IOPS o un rendimiento combinado de 400 MB por segundo. Como se explica en la sección siguiente, debe usar un tamaño de máquina virtual que admita la IOPS y el rendimiento de disco combinados.

> [!NOTE]
> a medida que aumente la IOPS o el rendimiento, el otro también aumenta, asegúrese de que no supera los límites de IOPS o rendimiento del disco o la máquina virtual al aumentar cualquiera de ellos.

Para ver los efectos del tamaño de E/S en el rendimiento de las aplicaciones, puede ejecutar las herramientas de pruebas comparativas en la máquina virtual y los discos. Cree varias ejecuciones de pruebas y use un tamaño de E/S diferente para cada ejecución para ver el impacto. Consulte el artículo Pruebas comparativas, cuyo vínculo aparece al final de este documento, para obtener más detalles.

## <a name="high-scale-vm-sizes"></a>Tamaños de máquina virtual a gran escala

Al empezar a diseñar una aplicación, una de las primeras cosas que hay que hacer es elegir una máquina virtual para hospedar la aplicación. Premium Storage viene con tamaños de máquina virtual a gran escala que pueden ejecutar aplicaciones que requieren una mayor capacidad de proceso y un alto rendimiento de E/S del disco local Estas máquinas virtuales proporcionan procesadores más rápidos, una mayor proporción de memoria a núcleo y una unidad de estado sólido (SSD) para el disco local. Algunos ejemplos de máquinas virtuales a gran escala que admiten Premium Storage son las de las series DS y GS.

Las máquinas virtuales a gran escala están disponibles en distintos tamaños con un número diferente de núcleos de CPU, memoria, sistema operativo y tamaño del disco temporal. Cada tamaño de máquina virtual también tiene el número máximo de discos de datos que se puede conectar a la máquina virtual. Por lo tanto, el tamaño de máquina virtual seleccionado afectará al procesamiento, la memoria y la capacidad de almacenamiento que están disponibles para su aplicación. También afecta al proceso y los costos de almacenamiento. Por ejemplo, a continuación se proporcionan las especificaciones del mayor tamaño de máquina virtual en una serie DS y una serie GS:

| Tamaño de VM | Núcleos de CPU | Memoria | Tamaños de disco de VM | Máx. discos de datos | Tamaño de memoria caché | E/S | Límites de E/S de la memoria caché de ancho de banda |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_DS14 |16 |112 GB |OS = 1.023 GB <br> SSD Local = 224 GB |32 |576 GB |50.000 E/S por segundo <br> 512 MB por segundo |4\.000 IOPS y 33 MB por segundo |
| Standard_GS5 |32 |448 GB |OS = 1.023 GB <br> SSD Local = 896 GB |64 |4224 GB |80.000 E/S por segundo <br> 2000 MB por segundo |5\.000 IOPS y 50 MB por segundo |

Para ver una lista completa de todos los tamaños disponibles de máquina virtual de Azure, consulte [Tamaños de las máquinas virtuales en Azure](sizes.md). Elija un tamaño de máquina virtual que puede cumplir y escale a los requisitos de rendimiento de las aplicaciones que desee. Además, tenga en cuenta que debe seguir consideraciones importantes al elegir los tamaños de las máquinas virtuales.

*Límites de escala*  
Los límites máximos de IOPS por máquina virtual y por disco son diferentes e independientes entre sí. Asegúrese de que la aplicación mantiene la IOPS dentro de los límites de la máquina virtual, así como los discos de premium conectados a ella. En caso contrario, el rendimiento de las aplicaciones experimentará una limitación.

Por ejemplo, suponga que el requisito de la aplicación es un máximo de 4.000 IOPS. Para lograrlo, aprovisiona un disco P30 en una máquina virtual DS1. El disco P30 puede proporcionar hasta 5.000 IOPS. Sin embargo, la máquina virtual DS1 está limitada a 3.200 IOPS. Por consiguiente, el rendimiento de las aplicaciones estará limitado a 3.200 IOPS por el límite de la máquina virtual y el rendimiento disminuirá. Para evitar esta situación, elija un tamaño de máquina virtual y de disco que cumplan los requisitos de la aplicación.

*Costo de operación*  
En muchos casos, es posible que el costo general de operación con Premium Storage sea inferior al uso de Standard Storage.

Por ejemplo, considere una aplicación que requiere más de 16.000 IOPS. Para obtener este rendimiento, necesitará una VM IaaS de Azure Standard\_D14, que puede proporcionar una IOPS máxima de 16.000 con 32 discos de 1 TB de almacenamiento estándar. Cada disco de almacenamiento estándar de 1 TB puede alcanzar un máximo de 500 IOPS. El costo estimado de esta máquina virtual por mes será de 1.570 USD. El costo mensual de 32 discos de almacenamiento estándar será de 1.638 USD. El costo mensual total estimado será de 3.208 USD.

Sin embargo, si hospeda la misma aplicación en Premium Storage, necesitará un tamaño de máquina virtual menor y menos discos de Premium Storage, lo que reduce el costo total. Una VM Standard\_DS13 puede cumplir los requisitos de 16.000 IOPS con cuatro discos P30. La máquina virtual DS13 tiene un máximo de 25.600 IOPS y cada disco P30 tiene un máximo de 5.000 IOPS. En general, esta configuración puede lograr 5.000 x 4 = 20.000 IOPS. El costo estimado de esta máquina virtual al mes será de 1.003 USD. El costo mensual de cuatro discos P30 de almacenamiento Premium será de 544,34 USD. El costo mensual total estimado será de 1,544 USD.

La tabla siguiente resume el análisis de costos de este escenario de Premium Storage y estándar.

| &nbsp; | **Estándar** | **Premium** |
| --- | --- | --- |
| **Costo de máquina virtual al mes** |1570,58 USD (Standard\_D14) |1003,66 USD (Standard\_DS13) |
| **Costo de discos al mes** |1638,40 USD (32 discos x 1 TB) |544,34 USD (4 discos x P30) |
| **Costo total al mes** |3\.208,98 USD |1\.544,34 USD |

*Linux Distros*  

Con Azure Premium Storage, obtendrá el mismo nivel de rendimiento para las máquinas virtuales de Windows y de Linux. Se admiten muchas versiones de las distribuciones de Linux; puede ver la lista completa [aquí](linux/endorsed-distros.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json). Es importante tener en cuenta que son adecuadas distintas distribuciones para diferentes tipos de carga de trabajo. Podrá ver diferentes niveles de rendimiento según la distribución en la que se ejecuta la carga de trabajo. Pruebe las distribuciones de Linux con su aplicación y elija la que mejor se adapte.

Cuando ejecute Linux con Premium Storage, compruebe las actualizaciones más recientes acerca de los controladores necesarios para garantizar un alto rendimiento.

## <a name="premium-storage-disk-sizes"></a>Tamaños de disco de Premium Storage

Azure Premium Storage ofrece varios tamaños, con el fin de que elija el que mejor se adapte a sus necesidades. Cada tamaño de disco tiene un límite de escala diferente de IOPS, ancho de banda y almacenamiento. Elija el tamaño de disco de Premium Storage adecuado según los requisitos de la aplicación y el tamaño de la máquina virtual a gran escala. En la tabla siguiente se muestran los tamaños de disco y sus capacidades. Los tamaños de disco P4, P6, P15, P60, P70 y P80 solo se admiten actualmente para Managed Disks.

[!INCLUDE [disk-storage-premium-ssd-sizes](../../includes/disk-storage-premium-ssd-sizes.md)]

El número de discos que elija depende del tamaño de disco elegido. Puede usar un único disco P50 o varios discos P10 para cubrir los requisitos de la aplicación. Tenga en cuenta las consideraciones enumeradas a continuación al realizar su elección.

*Límites de escala (IOPS y rendimiento)*  
Los límites de IOPS y rendimiento del tamaño de cada disco Premium son diferentes e independientes de los límites de escala de la máquina virtual. Asegúrese de que la IOPS y el rendimiento totales de los discos están dentro de los límites de escala del tamaño de máquina virtual elegido.

Por ejemplo, si un requisito de la aplicación es un máximo de 250 MB/s de rendimiento y usa una máquina virtual DS4 con un solo disco P30. La máquina virtual DS4 puede proporcionar un máximo rendimiento de 256 MB/s. Sin embargo, un solo disco P30 tiene un límite de rendimiento de 200 MB por segundo. Por lo tanto, la aplicación se restringirá a 200 MB/s debido al límite de disco. Para superar este límite, aprovisione más de un disco de datos para la máquina virtual o cambie el tamaño de los discos a P40 o P50.

> [!NOTE]
> Las lecturas atendidas por la caché no se incluyen en la IOPS y el rendimiento del disco, por lo que no están sujetas a los límites del disco. La caché tiene su límite de IOPS y rendimiento independiente de la máquina virtual.
>
> Por ejemplo, inicialmente sus lecturas y escrituras son 60MB/s y 40MB/s respectivamente. Con el tiempo, la memoria caché se prepara y atiende cada vez más y más lecturas de la memoria caché. Entonces, puede obtener un mayor rendimiento de escritura desde el disco.

*Número de discos*  
Para determinar el número de discos que necesitará, evalúe los requisitos de la aplicación. Cada tamaño de máquina virtual también tiene un límite en el número de discos que puede conectar a la máquina virtual. Normalmente, es dos veces el número de núcleos. Asegúrese de que el tamaño de la máquina virtual que elija puede admitir el número de discos necesario.

Recuerde que los discos de Premium Storage tienen capacidades de rendimiento superiores en comparación con los discos de Standard Storage. Por tanto, si va a migrar la aplicación de la máquina virtual IaaS de Azure Standard Storage a Premium Storage, probablemente necesitará menos discos premium para conseguir un rendimiento igual o superior para la aplicación.

## <a name="disk-caching"></a>Almacenamiento en caché de disco

Las máquinas virtuales a gran escala que aprovechan Azure Premium Storage tienen una tecnología de almacenamiento en caché de niveles múltiples denominada BlobCache. BlobCache usa una combinación de la RAM del host y del SSD local para almacenar en caché. Esta memoria caché está disponible para los discos de Premium Storage persistentes y los discos locales de la máquina virtual. De forma predeterminada, esta configuración de la caché se establece en lectura y escritura para los discos del sistema operativo y de solo lectura para los discos de datos hospedados en Premium Storage. Con la caché de disco habilitada en los discos de Premium Storage, la máquinas virtuales a gran escala pueden lograr niveles de rendimiento extremadamente altos que superan el rendimiento del disco subyacente.

> [!WARNING]
> No se admite el almacenamiento en caché de disco para discos de 4 TiB o más. Si varios discos están conectados a la máquina virtual, cada disco de menos de 4 TiB será compatible con el almacenamiento en caché.
>
> Al cambiar la configuración de caché de un disco de Azure, se desconecta y se vuelve a conectar el disco de destino. Si se trata del disco del sistema operativo, se reinicia la máquina virtual. Detenga todas las aplicaciones y todos los servicios que podrían verse afectados por esta interrupción antes de cambiar la configuración de caché de disco. El incumplimiento de estas recomendaciones podría provocar datos dañados.

Para más información acerca del funcionamiento de BlobCache, consulte la publicación de blog [Azure Premium Storage](https://azure.microsoft.com/blog/azure-premium-storage-now-generally-available-2/).

Es importante habilitar la memoria caché en el conjunto de discos correcto. Si debe habilitar el almacenamiento en caché de disco en un disco de premium o no dependerá del patrón de la carga de trabajo que el disco manejará. La tabla siguiente muestra el valor predeterminado de las opciones de caché para discos del sistema operativo y datos.

| **Tipo de disco** | **Configuración de la memoria caché predeterminada** |
| --- | --- |
| Disco del sistema operativo |ReadWrite |
| Disco de datos |ReadOnly |

A continuación se muestra la configuración de caché de disco recomendada para los discos de datos:

| **Configuración del almacenamiento en caché de disco** | **Recomendación sobre cuándo usar esta configuración** |
| --- | --- |
| None |Configure la caché de host como Ninguno para los discos de solo escritura y con escritura intensiva. |
| ReadOnly |Configure la caché de host como ReadOnly para discos de solo lectura y lectura y escritura. |
| ReadWrite |Configure la caché de host como ReadWrite solo si la aplicación maneja correctamente la escritura de datos en caché a discos persistentes cuando es necesario. |

*ReadOnly*  
Mediante la configuración del almacenamiento en caché ReadOnly en discos de datos de Premium Storage, puede lograr una baja latencia de lectura y obtener una IOPS de lectura y un rendimiento de la aplicación muy altos. Esto se debe a dos razones:

1. Las lecturas realizadas desde la memoria caché, que se encuentra en la memoria de la máquina virtual y el SSD local, son mucho más rápidas que las lecturas desde el disco de datos, que se encuentra en el almacenamiento de blobs de Azure.  
1. Premium Storage no cuenta las lecturas que se atienden desde la caché para la IOPS y el rendimiento del disco. Por lo tanto, la aplicación es capaz de lograr una IOPS y un rendimiento totales mayores.

*ReadWrite*  
De forma predeterminada, los discos del sistema operativo tienen habilitada la caché ReadWrite. Recientemente hemos agregado también compatibilidad para el almacenamiento en caché ReadWrite en los discos de datos. Si usa el almacenamiento en caché ReadWrite, debe tener una manera adecuada de escribir los datos de la memoria caché en discos persistentes. Por ejemplo, SQL Server administra por sí mismo la escritura de los datos en caché en los discos de almacenamiento persistentes. El uso de la memoria caché ReadWrite con una aplicación que no administre la persistencia de los datos necesarios puede provocar la pérdida de los datos, si se bloquea la máquina virtual.

*None*  
Actualmente, **None** solo se admite en discos de datos. No se admite en discos del SO. Si establece **None** en un disco del sistema operativo, se invalidará internamente y se establecerá en **ReadOnly**.

Por ejemplo, puede aplicar estas directrices a un SQL Server que funciona en Premium Storage del modo siguiente:

1. Configure la caché "ReadOnly" de los discos de Premium Storage que hospedan archivos de datos.  
   a.  Las rápidas lecturas de la caché reducen el tiempo de consulta de SQL Server, ya que las páginas de datos se recuperan mucho más rápido de la memoria caché que directamente desde los discos de datos.  
   b.  Atender las lecturas de la caché significa que hay un rendimiento adicional de los discos de datos premium. SQL Server puede usar este rendimiento adicional para recuperar más páginas de datos y otras operaciones, como copia de seguridad/restauración, cargas por lotes y volver a generar un índice.  
1. Configure la caché "None" en los discos de Premium Storage que hospedan los archivos de registro.  
   a.  Los archivos de registro tienen sobre todo muchas operaciones de escritura. Por lo tanto, no se benefician de la caché ReadOnly.

## <a name="optimize-performance-on-linux-vms"></a>Optimización del rendimiento en máquinas virtuales Linux

Para todos los discos ultra o SSD Premium con la memoria caché establecida en **ReadOnly** o **None**, debe deshabilitar las "barreras" cuando monte el sistema de archivos. En este escenario no se necesitan barreras, ya que las operaciones de escritura en los discos de Premium Storage son duraderas para esta configuración de la caché. Cuando la solicitud de escritura se completa correctamente, los datos están escritos en el almacenamiento permanente. Para deshabilitar las "barreras", utilice uno de los siguientes métodos. Elija uno para el sistema de archivos:
  
* Para **reiserFS**, para deshabilitar las barreras, utilice la opción de montaje `barrier=none`. (Para habilitar las barreras, use `barrier=flush`).
* Para **ext3/ext4**,, para deshabilitar las barreras, utilice la opción de montaje `barrier=0`. (Para habilitar las barreras, use `barrier=1`).
* Para **XFS**, para deshabilitar las barreras, utilice la opción de montaje `nobarrier`. (Para habilitar las barreras, use `barrier`).
* Para los discos de Premium Storage con la caché establecida en **ReadWrite**, habilite las barreras para la durabilidad de escritura.
* Para que las etiquetas de volumen persistan después de reiniciar la VM, debe actualizar /etc/fstab con las referencias del identificador único universal (UUID) en los discos. Para más información, vea [Adición de un disco administrado a una máquina virtual Linux](./linux/add-disk.md).

Las distribuciones de Linux siguientes han sido validadas para discos SSD. Para mejorar el rendimiento y la estabilidad con los discos SSD, se recomienda actualizar las máquinas virtuales a una de estas versiones o a una versión más reciente. 

Algunas de las versiones requieren Linux Integration Services (LIS) v4.0 para Azure. Para descargar e instalar una distribución, siga el vínculo que aparece en la tabla siguiente. Agregamos imágenes a la lista cuando se completa la validación. Nuestras validaciones muestran que el rendimiento varía en cada imagen. El rendimiento depende en las características de carga de trabajo y la configuración de la imagen. Se ajustan diferentes imágenes a los distintos tipos de cargas de trabajo.

| Distribución | Versión | Kernel compatible | Detalles |
| --- | --- | --- | --- |
| Ubuntu | 12.04 o más reciente| 3.2.0-75.110+ | &nbsp; |
| Ubuntu | 14.04 o más reciente| 3.13.0-44.73+  | &nbsp; |
| Debian | 7.x, 8.x o más reciente| 3.16.7-ckt4-1+ | &nbsp; |
| SUSE | SLES 12 o más reciente| 3.12.36-38.1+ | &nbsp; |
| SUSE | SLES 11 SP4 o más reciente| 3.0.101-0.63.1+ | &nbsp; |
| CoreOS | 584.0.0+ o más reciente| 3.18.4+ | &nbsp; |
| CentOS | 6.5, 6.6, 6.7, 7.0 o más reciente| &nbsp; | [LIS4 requerido](https://www.microsoft.com/download/details.aspx?id=55106) <br> *Vea la nota en la siguiente sección* |
| CentOS | 7.1+ o más reciente| 3.10.0-229.1.2.el7+ | [LIS4 recomendado](https://www.microsoft.com/download/details.aspx?id=55106) <br> *Vea la nota en la siguiente sección* |
| Red Hat Enterprise Linux (RHEL) | 6.8+, 7.2+ o más reciente | &nbsp; | &nbsp; |
| Oracle | 6.0+, 7.2+ o más reciente | &nbsp; | UEK4 o RHCK |
| Oracle | 7.0-7.1 o más reciente | &nbsp; | UEK4 o RHCK con[LIS4](https://www.microsoft.com/download/details.aspx?id=55106) |
| Oracle | 6.4-6.7 o más reciente | &nbsp; | UEK4 o RHCK con[LIS4](https://www.microsoft.com/download/details.aspx?id=55106) |

### <a name="lis-drivers-for-openlogic-centos"></a>Controladores de LIS para CentOS Openlogic

Si tiene máquinas virtuales de OpenLogic CentOS, ejecute el comando siguiente para instalar a los controladores más recientes:

```
sudo yum remove hypervkvpd  ## (Might return an error if not installed. That's OK.)
sudo yum install microsoft-hyper-v
sudo reboot
```

En algunos casos, el comando anterior también actualizará el kernel. Si se requiere una actualización del kernel, puede que tenga que volver a ejecutar los comandos anteriores después del reinicio para instalar completamente el paquete microsoft-hyper-v.


## <a name="disk-striping"></a>Seccionamiento del disco

Cuando una máquina virtual a gran escala esté conectada a varios discos persistente almacenamiento premium, los discos se pueden seccionar conjuntamente para agregar sus IOPS, ancho de banda y capacidad de almacenamiento.

En Windows, puede usar espacios de almacenamiento para seccionar discos conjuntamente. Debe configurar una columna para cada disco de un grupo. De lo contrario, el rendimiento general del volumen seccionado puede ser inferior al esperado debido a una distribución desigual del tráfico entre los discos.

Importante: Con la UI del Administrador del servidor, puede establecer el número total de columnas hasta en 8 para un volumen seccionado. Al conectar más de ocho discos, use PowerShell para crear el volumen. Mediante PowerShell, puede establecer un número de columnas igual al número de discos. Por ejemplo, si hay 16 discos en un solo conjunto de secciones; especifique 16 columnas en el parámetro *NumberOfColumns* del cmdlet de PowerShell *New-VirtualDisk*.

En Linux, use la utilidad MDADM para seccionar discos conjuntamente. Para ver los pasos detallados sobre cómo seccionar discos en Linux, consulte [Configuración del software RAID en Linux](linux/configure-raid.md).

*Tamaño de franja*  
Una configuración importante en el seccionamiento del disco es el tamaño de franja. El tamaño de franja o tamaño de bloque es el fragmento de datos más pequeño que la aplicación puede manejar en un volumen seccionado. El tamaño de franja que configurar depende del tipo de aplicación y su patrón de solicitudes. Si elije un tamaño de franja incorrecto, podría provocar la desalineación de E/S, lo que conduce a una disminución del rendimiento de la aplicación.

Por ejemplo, si una solicitud de E/S generada por la aplicación es mayor que el tamaño de franja del disco, el sistema de almacenamiento escribe a través de límites de la unidad de franja en más de un disco. Cuando llega el momento para tener acceso a esos datos, tendrá que buscar en las unidades con más de una franja para completar la solicitud. El efecto acumulativo de este comportamiento puede provocar una degradación del rendimiento considerable. Por otro lado, si el tamaño de la solicitud de E/S es menor que el tamaño de franja, y si es aleatoria por naturaleza, las solicitudes de E/S pueden acumularse en el mismo disco, causar un cuello de botella y, en última instancia, degradar el rendimiento de E/S.

Según el tipo de carga de trabajo que se ejecute la aplicación, elija un tamaño de franja adecuado. Para solicitudes de E/S pequeñas aleatorias, use un tamaño de franja más pequeño. Por otra parte, para solicitudes de E/S secuenciales grandes, use un tamaño de franja mayor. Descubra las recomendaciones de tamaño de franja para la aplicación que se ejecutará en Premium Storage. Para SQL Server, configure el tamaño de franja de 64 KB para cargas de trabajo OLTP y de 256 KB para cargas de trabajo de almacenamiento de datos. Vea [Procedimientos recomendados para SQL Server en máquinas virtuales de Azure](../azure-sql/virtual-machines/windows/performance-guidelines-best-practices.md#disks-guidance) para más información.

> [!NOTE]
> Puede seccionar conjuntamente un máximo de 32 discos de almacenamiento premium en una serie de máquinas virtuales DS y 64 discos de almacenamiento premium en una serie de máquinas virtuales GS.

## <a name="multi-threading"></a>Multithreading

Azure ha diseñado la plataforma Premium Storage para ser enormemente paralela. Por lo tanto, una aplicación multiproceso logra un rendimiento mucho mayor que una aplicación uniproceso. Una aplicación multiproceso divide sus tareas en varios subprocesos y aumenta la eficacia de su ejecución mediante el uso de los recursos de la máquina virtual y el disco al máximo.

Por ejemplo, si la aplicación se ejecuta en una máquina virtual de un solo núcleo que usa dos subprocesos, la CPU puede cambiar entre los dos subprocesos para lograr la eficiencia. Mientras un subproceso está esperando que se complete la E/S del disco, la CPU puede cambiar a otro subproceso. De esta manera, dos subprocesos pueden lograr más que un único subproceso. Si la máquina virtual tiene más de un núcleo, reduce aún más el tiempo de ejecución, ya que cada núcleo puede ejecutar tareas en paralelo.

No puede cambiar la forma en que una aplicación comercial implementa uniprocesos o multi-threading. Por ejemplo, SQL Server es capaz de manejar varios núcleos y CPU. Sin embargo, SQL Server determina en qué condiciones aprovechará uno o varios subprocesos para procesar una consulta. Puede ejecutar consultas y generar los índices mediante el multi-threading. Para una consulta que implica combinar tablas grandes y ordenar los datos antes de devolverlos al usuario, SQL Server probablemente usará varios subprocesos. Sin embargo, un usuario no puede controlar si SQL Server se ejecuta una consulta con un único subproceso o varios subprocesos.

Hay opciones de configuración que puede modificar para tener en cuenta este multi-threading o el procesamiento en paralelo de una aplicación. Por ejemplo, en el caso de SQL Server es la configuración con el máximo grado de paralelismo. Esta configuración, denominada MAXDOP, permite configurar el número máximo de procesadores que puede usar SQL Server cuando realiza el procesamiento en paralelo. Puede configurar MAXDOP para consultas individuales u operaciones de índice. Resulta especialmente útil cuando desea equilibrar los recursos del sistema para una aplicación crítica para el rendimiento.

Por ejemplo, supongamos que su aplicación con SQL Server ejecuta una consulta de gran tamaño y una operación de índice al mismo tiempo. Supongamos que desea que la operación de índice sea más eficaz en comparación con la consulta de gran tamaño. En tal caso, puede establecer el valor de MAXDOP de la operación de índice para que sea mayor que el valor de MAXDOP para la consulta. De este modo, SQL Server tiene un mayor número de procesadores que puede aprovechar para la operación de índice en comparación con el número de procesadores que puede dedicar a la consulta de gran tamaño. Recuerde que no controla el número de subprocesos que SQL Server usará para cada operación. Puede controlar el número máximo de procesadores que se dedicó a multi-threading.

Obtenga más información sobre [Grados de paralelismo](https://technet.microsoft.com/library/ms188611.aspx) en SQL Server. Descubra la configuración que influye en el multi-threading en la aplicación y sus configuraciones para optimizar el rendimiento.

## <a name="queue-depth"></a>Profundidad de la cola

La profundidad de la cola, la longitud de la cola o el tamaño de la cola es el número de solicitudes de E/S pendientes en el sistema. El valor de la profundidad de la cola determina cuántas operaciones de E/S, que procesarán los discos de almacenamiento, puede alinear la aplicación. Afecta a los tres indicadores de rendimiento de las aplicaciones que analizamos en este artículo: IOPS, rendimiento y latencia.

La profundidad de la cola y multi-threading están estrechamente relacionados. El valor de la profundidad de la cola indica el número de multi-threading que puede lograrse mediante la aplicación. Si la profundidad de la cola es grande, la aplicación puede ejecutar más operaciones simultáneamente, es decir, más multi-threading. Si la profundidad de la cola es pequeña, incluso si la aplicación es multiproceso, no tendrá suficientes solicitudes alineadas para la ejecución simultánea.

Normalmente, las aplicaciones comerciales no le permiten cambiar la profundidad de la cola, porque si se establece incorrectamente, hará más daño que beneficio. Las aplicaciones establecerán el valor correcto de profundidad de la cola para obtener un rendimiento óptimo. Sin embargo, es importante entender este concepto, para que pueda solucionar problemas de rendimiento con la aplicación. También puede observar los efectos de profundidad de la cola mediante la ejecución de herramientas de pruebas comparativas del sistema.

Algunas aplicaciones proporcionan opciones para influir en la profundidad de la cola. Por ejemplo, la configuración de MAXDOP (grado máximo de paralelismo) en SQL Server se explicó en la sección anterior. MAXDOP es una forma de influir en la profundidad de la cola y el multi-threading, aunque no cambia directamente el valor de Profundidad de la cola de SQL Server.

*Profundidad de cola alta*  
Una profundidad de la cola alta alinea más operaciones en el disco. El disco conoce la siguiente solicitud de su cola de antemano. Por lo tanto, el disco puede programar las operaciones de antemano y procesarlas en una secuencia óptima. Puesto que la aplicación está enviando solicitudes más al disco, el disco puede procesar más E/S paralelas. En última instancia, la aplicación podrá lograr una mayor IOPS. Puesto que la aplicación está procesando más solicitudes, también aumenta el rendimiento total de la aplicación.

Normalmente, una aplicación puede lograr un rendimiento máximo con 8-16+ E/S pendientes para cada disco conectado. Si la profundidad de la cola es uno, la aplicación no inserta suficientes E/S en el sistema y procesará menos cantidad en un período determinado. En otras palabras, menor rendimiento.

Por ejemplo, en SQL Server, si se establece el valor de MAXDOP en una consulta en "4", se informa a SQL Server de que puede usar un máximo de cuatro núcleos para ejecutar la consulta. SQL Server determinará el mejor valor de profundidad de la cola y el número de núcleos para la ejecución de la consulta.

*Profundidad de la cola óptima*  
Un valor de profundidad de la cola muy alto también tiene sus inconvenientes. Si el valor de profundidad de la cola es demasiado alto, la aplicación intentará manejar una IOPS muy alta. A menos que la aplicación tiene discos persistentes con suficientes IOPS aprovisionada, esto puede afectar negativamente a las latencias de la aplicación. La siguiente fórmula muestra la relación entre la E/S por segundo, la latencia y la profundidad de la cola.  
    ![Un diagrama que muestra la ecuación de IOPS multiplicado por la latencia es igual a la profundidad de la cola.](media/premium-storage-performance/image6.png)

No debe configurar la profundidad de la cola en cualquier valor alto, sino en un valor óptimo, que puede ofrecer suficientes IOPS para la aplicación sin afectar a las latencias. Por ejemplo, si la latencia de la aplicación debe ser 1 milisegundo, la profundidad de la cola necesaria para lograr 5.000 IOPS es QD = 5000 x 0,001 = 5.

*Profundidad de la cola para un volumen seccionado*  
Para un volumen seccionado, mantenga una profundidad de la cola lo suficientemente alta para que cada disco tenga una profundidad de la cola máxima individual. Por ejemplo, imagine una aplicación que inserta una profundidad de cola de 2 y hay cuatro discos en la franja. Las dos solicitudes de E/S irán a dos discos y los dos discos restantes estarán inactivos. Por lo tanto, configure la profundidad de la cola de modo que todos los discos puedan estar ocupados. La siguiente fórmula muestra cómo determinar la profundidad de la cola de volúmenes seccionados.  
    ![Un diagrama que muestra la ecuación de profundidad de la cola por disco multiplicado por el número de columnas por volumen es igual a la profundidad de la cola del volumen seleccionado.](media/premium-storage-performance/image7.png)

## <a name="throttling"></a>Limitaciones

Azure Premium Storage aprovisiona un número especificado de IOPS y rendimiento de acuerdo con los tamaños de la máquina virtual y de disco que elija. Cada vez que la aplicación intenta que la IOPS o el rendimiento estén por encima de los límites que puede administrar la máquina virtual o el disco, Premium Storage lo limitará. Esto se manifiesta en forma de una disminución del rendimiento de la aplicación. Esto puede significar una latencia mayor, un rendimiento menor o una IOPS menor. Si Premium Storage no lo limita, la aplicación podría fallar completamente al exceder lo que sus recursos son capaces de conseguir. Por lo tanto, para evitar problemas de rendimiento debido a la limitación,  aprovisione siempre suficientes recursos para su aplicación. Tenga en cuenta lo que hemos explicado en las secciones anteriores sobre los tamaños de la máquina virtual y el disco. Las pruebas comparativas son la mejor forma de averiguar qué recursos necesitará para hospedar su aplicación.

## <a name="next-steps"></a>Pasos siguientes

Si desea realizar un banco de pruebas del disco, consulte nuestros artículos sobre cómo ejecutar bancos de pruebas de un disco:

* Para Linux: [Ejecución de un banco de pruebas de la aplicación en Azure Disk Storage](linux/disks-benchmarks.md).
* Para Windows: [Ejecución de un banco de pruebas de un disco](windows/disks-benchmarks.md).

Más información sobre los tipos de disco disponibles:

* Para Linux: [Selección de un tipo de disco](disks-types.md)
* Para Windows: [Selección de un tipo de disco](disks-types.md)

Para los usuarios de SQL Server, lea artículos sobre procedimientos recomendados para SQL Server:

* [Procedimientos recomendados para SQL Server en Azure Virtual Machines](../azure-sql/virtual-machines/windows/performance-guidelines-best-practices.md)
* [Azure Premium Storage proporciona el máximo rendimiento para SQL Server en una máquina virtual de Azure](https://cloudblogs.microsoft.com/sqlserver/2015/04/23/azure-premium-storage-provides-highest-performance-for-sql-server-in-azure-vm/)
