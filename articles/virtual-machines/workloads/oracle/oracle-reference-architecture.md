---
title: Arquitecturas de referencia para bases de datos de Oracle en Azure | Microsoft Docs
description: Arquitecturas de referencia para ejecutar bases de datos de Oracle Database Enterprise Edition en Microsoft Azure Virtual Machines.
services: virtual-machines-linux
author: rgardler
manager: ''
tags: ''
ms.service: virtual-machines
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 12/13/2019
ms.author: rogardle
ms.custom: ''
ms.openlocfilehash: 8feede515cf7ed861f3219fdf5f4642a33c9e83e
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88690364"
---
# <a name="reference-architectures-for-oracle-database-enterprise-edition-on-azure"></a>Arquitecturas de referencia para Oracle Database Enterprise Edition en Azure

En esta guía se detalla la información sobre la implementación de una base de datos de alta disponibilidad de Oracle en Azure. Además, en esta guía se profundiza en las consideraciones sobre la recuperación ante desastres. Estas arquitecturas se han creado en función de las implementaciones de los clientes. Esta guía solo se aplica a Oracle Database Enterprise Edition.

Si le interesa obtener más información sobre cómo maximizar el rendimiento de Oracle Database, consulte [Diseño de una instancia de Oracle DB](oracle-design.md).

## <a name="assumptions"></a>Supuestos

- Reconoce los distintos conceptos de Azure, como las [zonas de disponibilidad](../../../availability-zones/az-overview.md).
- Ejecuta Oracle Database Enterprise Edition 12c o una versión posterior.
- Es consciente de las implicaciones de las licencias y las reconoce al usar las soluciones de este artículo.

## <a name="high-availability-for-oracle-databases"></a>Alta disponibilidad para bases de datos de Oracle

Lograr una alta disponibilidad en la nube es una parte importante de la planificación y el diseño de cada organización. Microsoft Azure ofrece [zonas de disponibilidad](../../../availability-zones/az-overview.md) y conjuntos de disponibilidad (que se pueden usar en las regiones en las que las zonas de disponibilidad no están disponibles). Obtenga más información sobre la [administración de la disponibilidad de las máquinas virtuales](../../../virtual-machines/linux/manage-availability.md) en el diseño de la nube.

Además de las ofertas y las herramientas nativas de la nube, Oracle proporciona soluciones para la alta disponibilidad, como [Oracle Data Guard](https://docs.oracle.com/en/database/oracle/oracle-database/18/sbydb/introduction-to-oracle-data-guard-concepts.html#GUID-5E73667D-4A56-445E-911F-1E99092DD8D7), [Data Guard con FSFO](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/dgbkr/index.html), [Sharding](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/admin/sharding-overview.html) y [GoldenGate](https://www.oracle.com/middleware/technologies/goldengate.html) que se pueden configurar en Azure. En esta guía se describen las arquitecturas de referencia para cada una de estas soluciones.

Por último, al migrar o crear aplicaciones para la nube, es importante retocar el código de aplicación para agregar patrones nativos de la nube, como el [patrón de reintento](/azure/architecture/patterns/retry) y el [patrón de interruptor](/azure/architecture/patterns/circuit-breaker). Los patrones adicionales definidos en la [guía Patrones de diseño en la nube](/azure/architecture/patterns/) pueden ayudar a que su aplicación sea más resistente.

### <a name="oracle-rac-in-the-cloud"></a>Oracle RAC en la nube

Oracle Real Application Cluster (RAC) es una solución de Oracle que ayuda a los clientes a lograr un alto rendimiento al tener muchas instancias que obtienen acceso a un almacenamiento de base de datos (patrón de arquitectura de uso compartido de todo). Aunque Oracle RAC también puede usarse para la alta disponibilidad en el entorno local, no se puede usar Oracle RAC por sí solo para la alta disponibilidad en la nube, ya que solo protege frente a errores de nivel de instancia y no frente a errores de nivel de bastidor o de centro de datos. Por este motivo, Oracle recomienda usar Oracle Data Guard con la base de datos (ya sea de una sola instancia o RAC) para lograr la alta disponibilidad. Por lo general, los clientes requieren un SLA alto para ejecutar sus aplicaciones críticas. Actualmente, Oracle RAC no está certificado o no es compatible con Oracle en Azure. Sin embargo, Azure ofrece características como Availability Zones y ventanas de mantenimiento planeado para ayudar en la protección frente a errores de nivel de instancia. Además, los clientes pueden usar tecnologías como Oracle Data Guard, Oracle GoldenGate y Oracle Sharding para conseguir un rendimiento y una resistencia altos mediante la protección de sus bases de datos contra los errores de nivel de bastidor, de nivel de centro de datos y de políticas geográficas.

Cuando se ejecutan instancias de Oracle Database en varias [zonas de disponibilidad](../../../availability-zones/az-overview.md) junto con Oracle Data Guard o GoldenGate, los clientes pueden obtener un SLA de tiempo de actividad del 99,99 %. En las regiones de Azure en las que todavía no hay zonas de disponibilidad, los clientes pueden usar los [conjuntos de disponibilidad](../../linux/manage-availability.md#configure-multiple-virtual-machines-in-an-availability-set-for-redundancy) y lograr un SLA de tiempo de actividad del 99,95 %.

>NOTA:  Se puede tener un objetivo de tiempo de actividad mucho mayor que el SLA de tiempo de actividad proporcionado por Microsoft.

## <a name="disaster-recovery-for-oracle-databases"></a>Recuperación ante desastres para bases de datos de Oracle

Al hospedar las aplicaciones críticas en la nube, es importante diseñar la alta disponibilidad y la recuperación ante desastres.

En Oracle Database Enterprise Edition, Oracle Data Guard es una característica útil para la recuperación ante desastres. Puede configurar una instancia de base de datos en espera en una [región de Azure emparejada](../../../best-practices-availability-paired-regions.md) y configurar la conmutación por error de Data Guard para la recuperación ante desastres. Para evitar la pérdida de datos, se recomienda implementar una instancia de Oracle Data Guard Far Sync, además de Active Data Guard. 

Considere la posibilidad de configurar la instancia de Data Guard Far Sync en una zona de disponibilidad distinta de la de la base de datos principal de Oracle si la aplicación permite la latencia (se requiere una prueba exhaustiva). Use un modo de **disponibilidad máxima** para configurar el transporte sincrónico de los archivos de fase de puesta al día en la instancia de Far Sync. Estos archivos se transfieren de forma asincrónica a la base de datos en espera. 

Si la aplicación no permite la pérdida de rendimiento al configurar una instancia de Far Sync en otra zona de disponibilidad en modo de **disponibilidad máxima** (sincrónico), puede configurar una instancia de Far Sync en la misma zona de disponibilidad que la base de datos principal. Para obtener una mayor disponibilidad, considere la posibilidad de configurar varias instancias de Far Sync cerca de su base de datos principal y al menos una instancia cercana a la base de datos en espera (si el rol realiza la transición). Obtenga más información sobre Oracle Data Guard Far Sync en estas [notas del producto Oracle Active Data Guard Far Sync](https://www.oracle.com/technetwork/database/availability/farsync-2267608.pdf).

Cuando se usan bases de datos de Oracle Standard Edition, hay soluciones ISV, como DBVisit Standby, que le permiten configurar la alta disponibilidad y la recuperación ante desastres.

## <a name="reference-architectures"></a>Arquitecturas de referencia

### <a name="oracle-data-guard"></a>Protección de datos de Oracle

Oracle Data Guard garantiza la alta disponibilidad, la protección de datos y la recuperación ante desastres para los datos empresariales. Data Guard mantiene las bases de datos en espera como copias coherentes con las transacciones de la base de datos principal. En función de la distancia entre las bases de datos principal y secundaria y la tolerancia de la aplicación para la latencia, puede configurar la replicación sincrónica o asincrónica. Después, si la base de datos principal no está disponible debido a una interrupción planeada o no planeada, Data Guard puede cambiar cualquier base de datos en espera al rol principal, lo que minimiza el tiempo de inactividad. 

Al usar Oracle Data Guard, también puede abrir la base de datos secundaria para solo lectura. Esta configuración se conoce como Active Data Guard. Oracle Database 12c presentó una característica denominada instancia de Data Guard Far Sync. Esta instancia le permite configurar una configuración de pérdida de datos cero de la base de datos de Oracle, sin tener que poner en peligro el rendimiento.

> [!NOTE]
> Active Data Guard requiere una licencia adicional. Esta licencia también es necesaria para usar la característica Far Sync. Póngase en contacto con su representante de Oracle para analizar las implicaciones de las licencias.

#### <a name="oracle-data-guard-with-fsfo"></a>Oracle Data Guard con FSFO
Oracle Data Guard con Fast-Start Failover (FSFO) puede proporcionar resistencia adicional mediante la configuración del agente en una máquina independiente. Data Guard Broker y la base de datos secundaria ejecutan el observador y observan si se produce un tiempo de inactividad en la base de datos principal. Esto permite también la redundancia en la configuración del observador de Data Guard. 

Con Oracle Database versión 12.2 y versiones posteriores, también es posible configurar varios observadores con una única configuración de Oracle Data Guard Broker. Esta configuración proporciona disponibilidad adicional, en caso de que un observador y la base de datos secundaria experimenten tiempo de inactividad. Data Guard Broker es ligero y se puede hospedar en una máquina virtual relativamente pequeña. Para obtener más información acerca de Data Guard Broker y sus ventajas, visite la [documentación de Oracle](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/dgbkr/oracle-data-guard-broker-concepts.html) sobre este tema.

En el siguiente diagrama se presenta una arquitectura recomendada para usar Oracle Data Guard en Azure con zonas de disponibilidad. Esta arquitectura le permite obtener un SLA de tiempo de actividad de VM del 99,99 %.

![Oracle Database que usa zonas de disponibilidad con Data Guard Broker: FSFO](./media/oracle-reference-architecture/oracledb_dg_fsfo_az.png)

En el diagrama anterior, el sistema cliente tiene acceso a una aplicación personalizada con el back-end de Oracle a través de la web. El front-end de la web está configurado en un equilibrador de carga. El front-end de la web realiza una llamada al servidor de aplicaciones adecuado para administrar el trabajo. El servidor de aplicaciones consulta la base de datos principal de Oracle. La base de datos de Oracle se ha configurado con una [máquina virtual con optimización para memoria](../../sizes-memory.md) de hiperproceso con [vCPU principales restringidas](../../../virtual-machines/constrained-vcpu.md) para ahorrar en los costos de licencia y maximizar el rendimiento. Se usan varios discos Premium o Ultra (Managed Disks) para el rendimiento y la alta disponibilidad.

Las bases de datos de Oracle se colocan en varias zonas de disponibilidad para lograr la alta disponibilidad. Cada zona consta de uno o varios centros de datos equipados con alimentación, refrigeración y redes independientes. Para garantizar la resistencia, se configuran un mínimo de tres zonas independientes en todas las regiones habilitadas. La separación física de las zonas de disponibilidad dentro de una región protege los datos frente a los errores del centro de datos. Además, se configuran dos observadores FSFO en dos zonas de disponibilidad para iniciar y conmutar por error la base de datos a la secundaria cuando se produce una interrupción. 

Puede configurar observadores adicionales y/o bases de datos en espera en una zona de disponibilidad diferente (AZ 1, en este caso) a la zona mostrada en la arquitectura anterior. Por último, el Administrador corporativo de Oracle (OEM) supervisa el tiempo de actividad y el rendimiento de las bases de datos de Oracle. El OEM también le permite generar varios informes de rendimiento y uso.

En las regiones en las que no se admiten las zonas de disponibilidad, puede usar conjuntos de disponibilidad para implementar Oracle Database con alta disponibilidad. Los conjuntos de disponibilidad permiten lograr un tiempo de actividad de VM del 99,95 %. El siguiente diagrama es una arquitectura de referencia de este uso:

![Oracle Database que usa conjuntos de disponibilidad con Data Guard Broker: FSFO](./media/oracle-reference-architecture/oracledb_dg_fsfo_as.png)

> [!NOTE]
> * No es necesario colocar la máquina virtual del Administrador corporativo de Oracle en un conjunto de disponibilidad, ya que solo se implementa una instancia de OEM.
> * Los discos Ultra no se admiten actualmente en una configuración de conjunto de disponibilidad.

#### <a name="oracle-data-guard-far-sync"></a>Oracle Data Guard Far Sync

Oracle Data Guard Far Sync proporciona la funcionalidad de protección de pérdida de datos cero para Oracle Database. Esta funcionalidad le permite protegerse frente a la pérdida de datos en el caso de que se produzca un error en la máquina de base de datos. Oracle Data Guard Far Sync tiene que estar instalado en una máquina virtual independiente. Far Sync es una instancia ligera de Oracle que solo tiene un archivo de control, un archivo de contraseña, spfile y registros en espera. No hay archivos de datos ni archivos de registro rego. 

Para una protección de pérdida de datos cero, debe haber comunicación sincrónica entre la base de datos principal y la instancia de Far Sync. La instancia de Far Sync recibe la fase de puesta al día de la principal de forma sincrónica y la reenvía inmediatamente a todas las bases de datos en espera de forma asincrónica. Esta configuración también reduce la sobrecarga en la base de datos principal, ya que solo tiene que enviar la fase de puesta al día a la instancia de Far Sync, en lugar de a todas las bases de datos en espera. Si se produce un error en una instancia de Far Sync, Data Guard usa automáticamente el transporte asincrónico a la base de datos secundaria desde la base de datos principal para mantener una protección de pérdida de datos casi cero. Para una mayor resistencia, los clientes pueden implementar varias instancias de Far Sync por cada instancia de base de datos (principal y secundarias).

En el siguiente diagrama se presenta una arquitectura de alta disponibilidad que usa Oracle Data Guard Far Sync:

![Oracle Database que usa zonas de disponibilidad con Oracle Data Guard Far Sync y Broker: FSFO](./media/oracle-reference-architecture/oracledb_dg_fs_az.png)

En la arquitectura anterior, hay una instancia de Far Sync implementada en la misma zona de disponibilidad que la instancia de base de datos para reducir la latencia entre las dos. En los casos en los que la aplicación sea sensible a la latencia, considere la posibilidad de implementar la base de datos y la instancia o instancias de Far Sync en un [grupo de selección de ubicación de proximidad](../../../virtual-machines/linux/proximity-placement-groups.md).

En el siguiente diagrama se presenta una arquitectura que usa Oracle Data Guard FSFO y Far Sync para lograr la alta disponibilidad y la recuperación ante desastres:

![Oracle Database que usa zonas de disponibilidad para la recuperación ante desastres con Data Guard Far Sync y Broker: FSFO](./media/oracle-reference-architecture/oracledb_dg_fs_az_dr.png)

### <a name="oracle-goldengate"></a>Oracle GoldenGate

GoldenGate permite el intercambio y la manipulación de datos en el nivel de transacción entre varias plataformas heterogéneas en toda la empresa. Mueve las transacciones confirmadas con integridad de transacción y mínima sobrecarga de la infraestructura existente. Su arquitectura modular le ofrece la flexibilidad de extraer y replicar registros de datos seleccionados, cambios transaccionales y cambios en DDL (lenguaje de definición de datos) en una variedad de topologías.

Oracle GoldenGate le permite configurar la base de datos para la alta disponibilidad al proporcionar replicación bidireccional. Esto le permite configurar una **arquitectura multimaestro** o una **configuración activo-activo**. En el siguiente diagrama se presenta una arquitectura recomendada para una configuración activo-activo de Oracle GoldenGate en Azure. En la siguiente arquitectura, la base de datos de Oracle se ha configurado con una [máquina virtual con optimización para memoria](../../sizes-memory.md) de hiperproceso con [vCPU principales restringidas](../../../virtual-machines/constrained-vcpu.md) para ahorrar en los costos de licencia y maximizar el rendimiento. Se usan varios discos Premium o Ultra (Managed Disks) para el rendimiento y la disponibilidad.

![Oracle Database que usa zonas de disponibilidad con Data Guard Broker: FSFO](./media/oracle-reference-architecture/oracledb_gg_az.png)

> [!NOTE]
> Se puede configurar una arquitectura similar con conjuntos de disponibilidad en regiones en las que las zonas de disponibilidad no están disponibles actualmente.

Oracle GoldenGate tiene procesos como la extracción, el bombeo y la replicación que le ayudan a replicar de forma asincrónica los datos de un servidor de base de datos de Oracle en otro. Estos procesos le permiten configurar una replicación bidireccional para garantizar la alta disponibilidad de la base de datos si hay tiempo de inactividad de nivel de zona de disponibilidad. En el diagrama anterior, el proceso de extracción se ejecuta en el mismo servidor que la base de datos de Oracle, mientras que los procesos de bombeo de datos y replicación se ejecutan en un servidor independiente en la misma zona de disponibilidad. El proceso de replicación se usa para recibir datos de la base de datos en la otra zona de disponibilidad y confirmar los datos en la base de datos de Oracle de su zona de disponibilidad. Del mismo modo, el proceso de bombeo de datos envía los datos extraídos por el proceso de extracción al proceso de replicación de la otra zona de disponibilidad. 

Aunque en el diagrama de arquitectura anterior se muestran los procesos de bombeo de datos y replicación configurados en un servidor independiente, puede configurar todos los procesos de Oracle GoldenGate en el mismo servidor, en función de la capacidad y el uso del servidor. Consulte siempre el informe AWR y las métricas de Azure para comprender el patrón de uso del servidor.

Al configurar la replicación bidireccional de Oracle GoldenGate en distintas zonas de disponibilidad o en distintas regiones, es importante asegurarse de que la latencia entre los distintos componentes es aceptable para su aplicación. La latencia entre zonas de disponibilidad y regiones puede variar y depende de varios factores. Se recomienda configurar pruebas de rendimiento entre la capa de aplicación y la capa de base de datos de distintas zonas de disponibilidad o regiones para confirmar que cumple los requisitos de rendimiento de la aplicación.

La capa de aplicación se puede configurar en su propia subred y la capa de base de datos se puede separar en su propia subred. Cuando sea posible, considere la posibilidad de usar [Azure Application Gateway](../../../application-gateway/overview.md) para equilibrar la carga del tráfico entre los servidores de aplicaciones. Azure Application Gateway es un equilibrador de carga sólido del tráfico web. Proporciona la afinidad de sesiones basada en cookies que mantiene una sesión de usuario en el mismo servidor, por lo que minimiza los conflictos en la base de datos. Las alternativas a Application Gateway son [Azure Load Balancer](../../../load-balancer/load-balancer-overview.md) y [Azure Traffic Manager](../../../traffic-manager/traffic-manager-overview.md).

### <a name="oracle-sharding"></a>Oracle Sharding

Sharding es un patrón de capa de datos que se presentó en Oracle 12.2. Permite particionar y escalar horizontalmente los datos entre bases de datos independientes. Se trata de una arquitectura sin ningún uso compartido en la que cada base de datos está hospedada en una máquina virtual dedicada, lo que permite un alto rendimiento de lectura y escritura, además de resistencia y mayor disponibilidad. Este patrón elimina los únicos puntos de error, proporciona aislamiento de errores y permite las actualizaciones graduales sin tiempo de inactividad. El tiempo de inactividad de una partición o de un error de nivel de centro de datos no afecta al rendimiento o a la disponibilidad de las demás particiones en otros centros de datos. 

Sharding es adecuado para aplicaciones OLTP de alto rendimiento que no se pueden permitir ningún tiempo de inactividad. Se garantiza que todas las filas con la misma clave de particionamiento están siempre en la misma partición, lo que aumenta el rendimiento y proporciona una alta coherencia. Las aplicaciones que usan el particionamiento deben tener un modelo de datos y una estrategia de distribución de datos bien definidos (hash, intervalo, lista o compuesto coherentes) que accedan principalmente a los datos mediante una clave de particionamiento (por ejemplo, *idCliente* o *númCuenta*). Sharding también le permite almacenar conjuntos concretos de datos más cerca de los clientes finales, lo que ayuda a cumplir los requisitos de rendimiento y cumplimiento.

Se recomienda replicar las particiones para lograr la alta disponibilidad y la recuperación ante desastres. Esta configuración se puede realizar mediante tecnologías de Oracle como Oracle Data Guard u Oracle GoldenGate. Una unidad de replicación puede ser una partición, una parte de una partición o un grupo de particiones. La disponibilidad de una base de datos particionada no se ve afectada por una interrupción o ralentización de una o más particiones. Para lograr la alta disponibilidad, las particiones en espera se pueden colocar en la misma zona de disponibilidad donde se encuentran las particiones principales. En el caso de la recuperación ante desastres, las particiones en espera pueden encontrarse en otra región. También puede implementar particiones en varias regiones para ofrecer tráfico en esas regiones. Obtenga más información sobre la configuración de la alta disponibilidad y la replicación de la base de datos particionada en la [documentación de Oracle Sharding](https://docs.oracle.com/en/database/oracle/oracle-database/19/shard/sharding-high-availability.html).

Oracle Sharding consta principalmente de los siguientes componentes. Puede encontrar más información sobre estos componentes en la [documentación de Oracle Sharding](https://docs.oracle.com/en/database/oracle/oracle-database/19/shard/sharding-overview.html):

- **Catálogo de particiones**: base de datos de Oracle de propósito especial que es un almacén persistente de todos los datos de configuración de las bases de datos de particiones. Todos los cambios de configuración, como la adición o la eliminación de particiones, la asignación de los datos y DDL en una base de datos de particiones se inician en el catálogo de particiones. El catálogo de particiones también contiene la copia maestra de todas las tablas duplicadas en una SDB. 

  El catálogo de particiones usa vistas materializadas para replicar automáticamente los cambios en las tablas duplicadas en todas las particiones. La base de datos del catálogo de particiones también actúa como un coordinador de consultas que se usa para procesar consultas de varias particiones y consultas que no especifican una clave de particionamiento. 
  
  El uso de Oracle Data Guard junto con las zonas de disponibilidad o los conjuntos de disponibilidad para la alta disponibilidad del catálogo de particiones es un procedimiento recomendado. La disponibilidad del catálogo de particiones no afecta a la disponibilidad de la base de datos particionada. Un tiempo de inactividad en el catálogo de particiones solo afecta a las operaciones de mantenimiento y a las consultas de varias particiones durante el breve período en que se completa la conmutación por error de Data Guard. La SDB continúa redirigiendo y ejecutando las transacciones en línea, que no se ven afectadas por una interrupción del catálogo.

- **Directores de particiones**: servicios ligeros que deben implementarse en cada zona de disponibilidad o región en la que residen las particiones. Los directores de particiones son los administradores de servicios globales implementados en el contexto de Oracle Sharding. Para lograr la alta disponibilidad, se recomienda que implemente al menos un director de particiones en cada zona de disponibilidad en la que existan las particiones. 

  Al conectarse inicialmente a la base de datos, un director de particiones configura y almacena en caché la información de enrutamiento, por lo que en las solicitudes posteriores se omite el director de particiones. Una vez que la sesión se establece con una partición, se admiten todas las consultas SQL y DML y se ejecutan en el ámbito de la partición especificada. Este enrutamiento es rápido y se usa para todas las cargas de trabajo OLTP que realizan transacciones dentro de las particiones. Se recomienda usar el enrutamiento directo para todas las cargas de trabajo OLTP que requieran el mayor rendimiento y disponibilidad. La caché de enrutamiento se actualiza automáticamente cuando una partición deja de estar disponible o se producen cambios en la topología de particionamiento. 
  
  Para el enrutamiento dependiente de los datos de alto rendimiento, Oracle recomienda el uso de un grupo de conexiones al acceder a los datos de la base de datos particionada. Los grupos de conexiones de Oracle, las bibliotecas específicas del lenguaje y los controladores admiten Oracle Sharding. Consulte la [documentación de Oracle Sharding](https://docs.oracle.com/en/database/oracle/oracle-database/19/shard/sharding-overview.html#GUID-3D41F762-BE04-486D-8018-C7A210D809F9) para más información.

- **Servicio global**: el servicio global es similar al servicio de base de datos normal. Además de todas las propiedades de un servicio de base de datos, un servicio global tiene propiedades para las bases de datos particionadas, como la afinidad de regiones entre clientes y la tolerancia a retrasos de partición y replicación. Solo se debe crear un servicio global para leer y escribir datos en una base de datos particionada. Al usar Active Data Guard y configurar réplicas de solo lectura de las particiones, puede crear otro servicio global para cargas de trabajo de solo lectura. El cliente puede usar estos servicios globales para conectarse a la base de datos.

- **Bases de datos de particiones**: las bases de datos de particiones son las bases de datos de Oracle. Cada base de datos se replica mediante Oracle Data Guard en una configuración de Broker con la opción Fast-Start Failover (FSFO) habilitada. No es necesario configurar la conmutación por error y la replicación de Data Guard en cada partición. Se configura e implementa automáticamente cuando se crea la base de datos compartida. Si se produce un error en una partición determinada, Oracle Sharding conmuta por error automáticamente las conexiones de base de datos del servidor principal al modo en espera.

Puede implementar y administrar las bases de datos particionadas de Oracle con dos interfaces: La GUI Oracle Enterprise Manager Cloud Control o la utilidad de línea de comandos `GDSCTL`. Incluso puede supervisar la disponibilidad y el rendimiento de las distintas particiones mediante el control de la nube. El comando `GDSCTL DEPLOY` crea automáticamente las particiones y sus respectivos clientes de escucha. Además, este comando implementa automáticamente la configuración de replicación usada para la alta disponibilidad en el nivel de partición especificada por el administrador.

Existen distintas formas de particionar una base de datos:

* Particionamiento administrado por el sistema: distribuye automáticamente entre particiones mediante el particionamiento.
* Particionamiento definido por el usuario: le permite especificar la asignación de los datos a las particiones, lo que funciona bien cuando existen requisitos normativos o de localización de datos.
* Particionamiento compuesto: una combinación de particionamiento administrado por el sistema y definido por el usuario para diferentes _espacios de particiones_
* Subparticiones de tabla: similar a una tabla con particiones normal.

Obtenga más información sobre los diferentes [métodos de particionamiento](https://docs.oracle.com/en/database/oracle/oracle-database/19/shard/sharding-methods.html) en la documentación de Oracle.

Aunque una base de datos particionada puede parecer una única base de datos para las aplicaciones y los desarrolladores, al migrar desde una base de datos sin particiones a una base de datos particionada, se requiere una planificación cuidadosa para determinar qué tablas se van a duplicar y cuáles se van a particionar. 

Las tablas duplicadas se almacenan en todas las particiones, mientras que las tablas particionadas se distribuyen entre diferentes particiones. La recomendación es la duplicación de tablas pequeñas y dimensionales y la distribución o partición de las tablas de hechos. Los datos se pueden cargar en la base de datos particionada usando el catálogo de particiones como coordinador central o ejecutando el bombeo de datos en cada partición. Obtenga más información sobre la [migración de datos a una base de datos particionada](https://docs.oracle.com/en/database/oracle/oracle-database/19/shard/sharding-loading-data.html) en la documentación de Oracle.

#### <a name="oracle-sharding-with-data-guard"></a>Oracle Sharding con Data Guard

Oracle Data Guard se puede usar para el particionamiento con métodos de particionamiento compuestos, administrados por el sistema y definidos por el usuario.

En el siguiente diagrama se presenta una arquitectura de referencia para Oracle Sharding con Oracle Data Guard que se usa para la alta disponibilidad de cada partición. El diagrama de arquitectura muestra un _método de particionamiento compuesto_. Es probable que el diagrama de la arquitectura difiera en las aplicaciones con requisitos diferentes para la ubicación de datos, el equilibrio de carga, la alta disponibilidad, la recuperación ante desastres, etc., y puede usar un método diferente para el particionamiento. Oracle Sharding le permite cumplir estos requisitos y escalar horizontal y eficazmente mediante estas opciones. Incluso puede implementarse una arquitectura similar mediante Oracle GoldenGate.

![Oracle Database Sharding que usa zonas de disponibilidad con Data Guard Broker: FSFO](./media/oracle-reference-architecture/oracledb_dg_sh_az.png)

Aunque el particionamiento administrado por el sistema es el más fácil de configurar y administrar, el particionamiento definido por el usuario o el particionamiento compuesto son idóneos para escenarios en los que los datos y la aplicación están distribuidos geográficamente o en escenarios en los que es necesario tener control sobre la replicación de cada partición. 

En la arquitectura anterior, el particionamiento compuesto se usa para distribuir geográficamente los datos y escalar horizontalmente las capas de aplicación. El particionamiento compuesto es una combinación de particionamiento administrado por el sistema y definido por el usuario y, por tanto, proporciona las ventajas de ambos métodos. En el escenario anterior, en primer lugar, se particionan los datos entre varios espacios de particiones separados por región. Después, los datos se particionan por hash coherente entre varias particiones en el espacio de particiones. Cada espacio de particiones contiene varios grupos de particiones. Cada grupo de particiones tiene varias particiones y es una "unidad" de replicación, en este caso. Cada grupo de particiones contiene todos los datos del espacio de particiones. Los grupos de particiones A1 y B1 son grupos de particiones principales, mientras que los grupos de particiones A2 y B2 están en espera. Puede elegir que las particiones individuales sean la unidad de replicación, en lugar de un grupo de particiones.

En la arquitectura anterior, se implementa un director de particiones o GSM en cada zona de disponibilidad para alta disponibilidad. La recomendación es implementar al menos un director de particiones o GSM por centro de datos/región. Además, se implementa una instancia del servidor de aplicaciones en cada zona de disponibilidad que contenga un grupo de particiones. Esta configuración permite que la aplicación conserve la latencia baja entre el servidor de aplicaciones y la base de datos/grupo de particiones. Si se produce un error en una base de datos, el servidor de aplicaciones de la misma zona que la base de datos en espera puede controlar las solicitudes una vez que se produce la transición del rol de base de datos. Azure Application Gateway y el director de particiones realizan un seguimiento de la latencia de solicitud y respuesta y redirigen las solicitudes en consecuencia.

Desde el punto de vista de una aplicación, el sistema cliente realiza una solicitud a Azure Application Gateway (u otras tecnologías de equilibrio de carga en Azure) que redirige la solicitud a la región más cercana al cliente. Azure Application Gateway también admite sesiones temporales, por lo que las solicitudes procedentes del mismo cliente se redirigen al mismo servidor de aplicaciones. El servidor de aplicaciones usa una agrupación de conexiones en los controladores de acceso a datos. Esta característica está disponible en controladores como JDBC, ODP.NET, OCI, etc. Los controladores pueden reconocer las claves de particionamiento especificadas como parte de la solicitud. [Oracle Universal Connection Pool (UCP)](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/jjucp/ucp-database-sharding-overview.html) para los clientes JDBC puede permitir que los clientes de aplicaciones que no son de Oracle, como Apache Tomcat e IIS, funcionen con Oracle Sharding. 

Durante la solicitud inicial, el servidor de aplicaciones se conecta al director de particiones de su región para obtener información de enrutamiento de la partición a la que se debe redirigir la solicitud. En función de la clave de particionamiento pasada, el director redirige el servidor de aplicaciones a la partición correspondiente. El servidor de aplicaciones almacena en caché esta información mediante la creación de una asignación y, en las solicitudes posteriores, se omite el director de particiones y redirige las solicitudes directamente a la partición.

#### <a name="oracle-sharding-with-goldengate"></a>Oracle Sharding con GoldenGate

En el siguiente diagrama se muestra una arquitectura de referencia para Oracle Sharding con Oracle GoldenGate que se usa para la alta disponibilidad en la región de cada partición. En contraposición a la arquitectura anterior, esta arquitectura solo ilustra la alta disponibilidad dentro de una única región de Azure (varias zonas de disponibilidad). Se puede implementar una base de datos particionada de alta disponibilidad en varias regiones (similar al ejemplo anterior) mediante Oracle GoldenGate.

![Oracle Database Sharding que usa zonas de disponibilidad con GoldenGate](./media/oracle-reference-architecture/oracledb_gg_sh_az.png)

La arquitectura de referencia anterior usa el método de particionamiento _administrado por el sistema_ para particionar los datos. Dado que la replicación de Oracle GoldenGate se realiza en un nivel de fragmento, la mitad de los datos replicados en una partición se pueden replicar en otra partición. La otra mitad se puede replicar en una partición diferente. 

La forma en que se replican los datos depende del factor de replicación. Con un factor de replicación 2, tendrá dos copias de cada fragmento de datos en las tres particiones del grupo de particiones. De forma similar, con un factor de replicación 3 y tres particiones en el grupo de particiones, todos los datos de cada partición se replicarán en todas las demás particiones del grupo de particiones. Cada partición del grupo de particiones puede tener un factor de replicación diferente. Esta configuración le ayuda a definir el diseño de alta disponibilidad y recuperación ante desastres de forma eficaz dentro de un grupo de particiones y entre varios grupos de particiones.

En la arquitectura anterior, el grupo de particiones A y el grupo de particiones B contienen los mismos datos, pero residen en zonas de disponibilidad diferentes. Si tanto el grupo de particiones A como el grupo de particiones B tienen el mismo factor de replicación 3, cada fila o fragmento de la tabla con particiones se replicará 6 veces en los dos grupos de particiones. Si el grupo de particiones A tiene un factor de replicación 3 y el grupo de particiones B tiene un factor de replicación 2, cada fila o fragmento se replicará 5 veces en los dos grupos de particiones.

Esta configuración evita la pérdida de datos si se produce un error de nivel de instancia o de zona de disponibilidad. La capa de aplicación es capaz de leer y escribir en cada partición. Para minimizar los conflictos, Oracle Sharding designa un "fragmento maestro" para cada intervalo de valores hash. Esta característica garantiza que las solicitudes de escritura para un fragmento determinado se dirigen al fragmento correspondiente. Además, Oracle GoldenGate proporciona la detección y resolución automática de conflictos para controlar cualquier conflicto que pueda surgir. Para obtener más información y conocer las limitaciones de la implementación de GoldenGate con Oracle Sharding, vea la documentación de Oracle sobre el uso de [Oracle GoldenGate con una base de datos particionada](https://docs.oracle.com/en/database/oracle/oracle-database/19/shard/sharding-high-availability.html#GUID-4FC0AC46-0B8B-4670-BBE4-052228492C72).

En la arquitectura anterior, se implementa un director de particiones o GSM en cada zona de disponibilidad para alta disponibilidad. La recomendación es implementar al menos un director de particiones o GSM por centro de datos o región. Además, se implementa una instancia del servidor de aplicaciones en cada zona de disponibilidad que contenga un grupo de particiones. Esta configuración permite que la aplicación conserve la latencia baja entre el servidor de aplicaciones y la base de datos/grupo de particiones. Si se produce un error en una base de datos, el servidor de aplicaciones de la misma zona que la base de datos en espera puede controlar las solicitudes una vez que se produce la transición del rol de base de datos. Azure Application Gateway y el director de particiones realizan un seguimiento de la latencia de solicitud y respuesta y redirigen las solicitudes en consecuencia.

Desde el punto de vista de una aplicación, el sistema cliente realiza una solicitud a Azure Application Gateway (u otras tecnologías de equilibrio de carga en Azure) que redirige la solicitud a la región más cercana al cliente. Azure Application Gateway también admite sesiones temporales, por lo que las solicitudes procedentes del mismo cliente se redirigen al mismo servidor de aplicaciones. El servidor de aplicaciones usa una agrupación de conexiones en los controladores de acceso a datos. Esta característica está disponible en controladores como JDBC, ODP.NET, OCI, etc. Los controladores pueden reconocer las claves de particionamiento especificadas como parte de la solicitud. [Oracle Universal Connection Pool (UCP)](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/jjucp/ucp-database-sharding-overview.html) para los clientes JDBC puede permitir que los clientes de aplicaciones que no son de Oracle, como Apache Tomcat e IIS, funcionen con Oracle Sharding. 

Durante la solicitud inicial, el servidor de aplicaciones se conecta al director de particiones de su región para obtener información de enrutamiento de la partición a la que se debe redirigir la solicitud. En función de la clave de particionamiento pasada, el director redirige el servidor de aplicaciones a la partición correspondiente. El servidor de aplicaciones almacena en caché esta información mediante la creación de una asignación y, en las solicitudes posteriores, se omite el director de particiones y redirige las solicitudes directamente a la partición.

## <a name="patching-and-maintenance"></a>Aplicación de revisión y mantenimiento

Al implementar las cargas de trabajo de Oracle en Azure, Microsoft se encarga de todas las aplicaciones de revisión de nivel de sistema operativo del host. Cualquier mantenimiento planeado de nivel de sistema operativo se comunica a los clientes con anterioridad para facilitar el mantenimiento planeado del cliente. Nunca se aplican revisiones a dos servidores de dos Availability Zones diferentes simultáneamente. Consulte [Administración de la disponibilidad de las máquinas virtuales](../../../virtual-machines/linux/manage-availability.md) para obtener más detalles sobre el mantenimiento y la aplicación de revisión de VM. 

La aplicación de revisiones al sistema operativo de una máquina virtual se puede automatizar mediante [Azure Automation Update Management](../../../automation/update-management/update-mgmt-overview.md). La aplicación de revisiones y el mantenimiento de una base de datos de Oracle se pueden automatizar y programar mediante [Azure Pipelines](/azure/devops/pipelines/get-started/what-is-azure-pipelines?view=azure-devops) o [Azure Automation Update Management](../../../automation/update-management/update-mgmt-overview.md) para minimizar el tiempo de inactividad. Consulte [Entrega continua e implementaciones de azul/verde](/azure/devops/learn/what-is-continuous-delivery) para entender cómo se puede usar en el contexto de las bases de datos de Oracle.

## <a name="architecture-and-design-considerations"></a>Consideraciones sobre la arquitectura y el diseño

- Considere la posibilidad de usar una [máquina virtual con optimización para memoria](../../sizes-memory.md) de hiperproceso con [vCPU principales restringidas](../../../virtual-machines/constrained-vcpu.md) para VM de Oracle Database para ahorrar en los costos de licencia y maximizar el rendimiento. Use varios discos Premium o Ultra (Managed Disks) para el rendimiento y la disponibilidad.
- Al usar Managed Disks, el nombre del disco o dispositivo puede cambiar en los reinicios. Se recomienda usar el UUID del dispositivo en lugar del nombre para asegurarse de que se conservan los montajes entre los reinicios. Puede encontrar más información [aquí](../../../virtual-machines/linux/configure-raid.md#add-the-new-file-system-to-etcfstab).
- Use las zonas de disponibilidad para lograr una alta disponibilidad en la región.
- Considere la posibilidad de usar discos Ultra (si están disponibles) o discos Premium para la base de datos de Oracle.
- Considere la posibilidad de configurar una base de datos de Oracle en espera en otra región de Azure mediante Oracle Data Guard.
- Considere la posibilidad de usar [grupos de selección de ubicación de proximidad](../../../virtual-machines/linux/co-location.md#proximity-placement-groups) para reducir la latencia entre la aplicación y la capa de base de datos.
- Configure [Oracle Enterprise Manager](https://docs.oracle.com/en/enterprise-manager/) para la administración, la supervisión y el registro.
- Considere la posibilidad de usar la administración automática del almacenamiento (ASM) de Oracle para simplificar la administración del almacenamiento de la base de datos.
- Use [Azure Pipelines](/azure/devops/pipelines/get-started/what-is-azure-pipelines) para administrar la aplicación de revisión y las actualizaciones de la base de datos sin tiempo de inactividad.
- Retoque el código de la aplicación para agregar patrones nativos de la nube, como el [patrón de reintento](/azure/architecture/patterns/retry), el [patrón de interruptor](/azure/architecture/patterns/circuit-breaker) y otros patrones definidos en la [guía Patrones de diseño en la nube](/azure/architecture/patterns/) que pueden ayudar a que la aplicación sea más resistente.

## <a name="next-steps"></a>Pasos siguientes

Revise los siguientes artículos de referencia de Oracle que se aplican a su escenario.

- [Introducción a Oracle Data Guard](https://docs.oracle.com/en/database/oracle/oracle-database/18/sbydb/introduction-to-oracle-data-guard-concepts.html#GUID-5E73667D-4A56-445E-911F-1E99092DD8D7)
- [Conceptos de Oracle Data Guard Broker](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/dgbkr/oracle-data-guard-broker-concepts.html)
- [Configuración de Oracle GoldenGate para alta disponibilidad activo-activo](https://docs.oracle.com/goldengate/1212/gg-winux/GWUAD/wu_bidirectional.htm#GWUAD282)
- [Información general sobre Oracle Sharding](https://docs.oracle.com/en/database/oracle/oracle-database/19/shard/sharding-overview.html)
- [Pérdida de datos cero con Oracle Active Data Guard Far Sync a cualquier distancia](https://www.oracle.com/technetwork/database/availability/farsync-2267608.pdf)
