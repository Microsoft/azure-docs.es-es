---
title: Grupos de conmutación por error automática
titleSuffix: Azure SQL Database & SQL Managed Instance
description: Los grupos de conmutación por error automática permiten administrar la replicación y la conmutación por error automática o coordinada de un grupo de bases de datos en un servidor o de todas las bases de datos en una instancia administrada.
services: sql-database
ms.service: sql-db-mi
ms.subservice: high-availability
ms.custom: sqldbrb=2
ms.devlang: ''
ms.topic: conceptual
author: anosov1960
ms.author: sashan
ms.reviewer: mathoma, carlrab
ms.date: 07/09/2020
ms.openlocfilehash: ecc2925b52679c5807c9fa79a67d7b45411df526
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87014038"
---
# <a name="use-auto-failover-groups-to-enable-transparent-and-coordinated-failover-of-multiple-databases"></a>Uso de grupos de conmutación por error automática para permitir la conmutación por error de varias bases de datos de manera transparente y coordinada
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Los grupos de conmutación por error automática permiten administrar la replicación y la conmutación por error de un grupo de bases de datos en un servidor o de todas las bases de datos de una instancia administrada en otra región. Se trata de una abstracción declarativa sobre la característica de [replicación geográfica activa](active-geo-replication-overview.md) existente, diseñada para simplificar la implementación y administración de bases de datos con replicación geográfica a escala. Puede iniciar la conmutación por error manualmente o puede delegarla en el servicio de Azure según una directiva definida por el usuario. La última opción le permite recuperar automáticamente varias bases de datos relacionadas en una región secundaria después de errores catastróficos u otros eventos no planeados que generen una pérdida total o parcial de la disponibilidad de SQL Database o Instancia administrada de SQL en la región primaria. Un grupo de conmutación por error puede incluir una o varias bases de datos, utilizadas normalmente por la misma aplicación. Además, puede usar las bases de datos secundarias legibles para descargar las cargas de trabajo de consulta de solo lectura. Debido a que los grupos de conmutación por error automática implican varias bases de datos, se deben configurar en el servidor principal. Los grupos de conmutación por error automática admiten la replicación de todas las bases de datos en el grupo solo a una instancia o un servidor secundario en otra región.

> [!NOTE]
> Si desea tener varias instancias de Azure SQL Database secundarias en la misma región o en regiones diferentes, use la [replicación geográfica activa](active-geo-replication-overview.md).

Cuando se usan grupos de conmutación por error automática con una directiva de conmutación por error automática, cualquier interrupción que afecte a una o varias de las bases de datos del grupo tiene como resultado la conmutación por error automática. Normalmente, se trata de incidentes que no se pueden mitigar automáticamente mediante las operaciones de alta disponibilidad automáticas integradas. Los ejemplos de desencadenadores de conmutación por error incluyen los incidentes causados por un anillo de inquilino de SQL Database o un anillo de control que está inactivo debido a una fuga de memoria del kernel del sistema operativo en varios nodos de proceso, o bien los incidentes provocados por uno o varios anillos de inquilino que están inactivos porque se cortó un cable de red incorrecto durante la retirada rutinaria de hardware.  Para más información, consulte [Alta disponibilidad y Azure SQL Database](high-availability-sla.md).

Además, los grupos de conmutación por error automática proporcionan puntos de conexión de agentes de escucha de lectura-escritura y de solo lectura que no se modifican durante las conmutaciones por error. Ya sea que use la activación de conmutación por error automática o manual, la conmutación por error transforma todas las bases de datos secundarias del grupo en primarias. Después de que la conmutación por error de una base de datos finaliza, el registro de DNS se actualiza automáticamente para redirigir los puntos de conexión a la nueva región. Para información sobre los datos específicos de RPO y RTO, consulte [Introducción a la continuidad empresarial](business-continuity-high-availability-disaster-recover-hadr-overview.md).

Cuando se usan grupos de conmutación por error automática con la directiva de conmutación por error automática, cualquier interrupción que afecte a las bases de datos del servidor o de la instancia administrada tiene como resultado la conmutación por error automática. Puede administrar el grupo de conmutación por error automática mediante:

- [Azure Portal](geo-distributed-application-configure-tutorial.md)
- [CLI de Azure: grupo de conmutación por error](scripts/add-database-to-failover-group-cli.md)
- [PowerShell: grupo de conmutación por error](scripts/add-database-to-failover-group-powershell.md)
- [API REST: grupo de conmutación por error](/rest/api/sql/failovergroups)

Después de la conmutación por error, asegúrese de que los requisitos de autenticación para el servidor y la base de datos o la instancia estén configurados en el nuevo elemento principal. Para obtener más información, consulte [Administración de la seguridad de Azure SQL Database después de la recuperación ante desastres](active-geo-replication-security-configure.md).

Para lograr una verdadera continuidad empresarial, agregar redundancia de base de datos entre centros de datos es solo parte de la solución. Para recuperar una aplicación (un servicio) de un extremo a otro tras un error catastrófico, es necesario recuperar todos los componentes que constituyen el servicio y cualquier servicio dependiente. Algunos ejemplos de estos componentes son el software cliente (por ejemplo, un explorador con JavaScript personalizado), los front-end web, el almacenamiento y DNS. Es fundamental que todos los componentes sean resistentes a los mismos errores y que estén disponibles en el plazo del objetivo de tiempo de recuperación (RTO) de la aplicación. Por lo tanto, debe identificar todos los servicios dependientes y comprender las garantías y capacidades que ofrecen. A continuación, debe seguir los pasos adecuados para asegurarse de que el servicio funcione durante la conmutación por error de los servicios de los que depende. Para más información sobre el diseño de soluciones para la recuperación ante desastres, consulte [Diseño de soluciones de nube para la continuidad empresarial mediante la replicación geográfica activa](designing-cloud-solutions-for-disaster-recovery.md).

## <a name="terminology-and-capabilities"></a>Terminología y funcionalidades

- **Grupo de conmutación por error (FOG)**

  Un grupo de conmutación por error es un grupo con nombre de bases de datos administradas por un único servidor o en una instancia administrada que puede conmutar por error como una unidad a otra región en caso de que algunas o todas las bases de datos principales estén deshabilitadas debido a una interrupción en la región primaria. Cuando se crea para Instancia administrada de SQL, un grupo de conmutación por error contiene todas las bases de datos de usuario de la instancia, por lo que solo se puede configurar un grupo de conmutación por error en una instancia.
  
  > [!IMPORTANT]
  > El nombre del grupo de conmutación por error debe ser único globalmente en el dominio `.database.windows.net`.

- **Servidores**

     Con los servidores, algunas o todas las bases de datos de usuario de un único servidor pueden colocarse en un grupo de conmutación por error. Además, un servidor admite varios grupos de conmutación por error en un único servidor.

- **Principal**

  Servidor o instancia administrada que hospeda las bases de datos principales del grupo de conmutación por error.

- **Secundario**

  Servidor o instancia administrada que hospeda las bases de datos secundarias del grupo de conmutación por error. La base de datos secundaria no puede estar en la misma región que la principal.

- **Incorporación de bases de datos únicas a un grupo de conmutación por error**

  Puede poner varias bases de datos únicas en el mismo servidor e incluirlas en el mismo grupo de conmutación por error. Si agrega una base de datos única al grupo de conmutación por error, automáticamente se creará una base de datos secundaria con la misma edición y el mismo tamaño de proceso del servidor secundario.  Ese servidor es el que especificó cuando creó el grupo de conmutación por error. Si agrega una base de datos que ya tenga una base de datos secundaria en el servidor secundario, el grupo heredará el vínculo de replicación geográfica. Cuando se agrega una base de datos que ya tiene una base de datos secundaria en un servidor que no forma parte del grupo de conmutación por error, se crea otra base de datos secundaria en el servidor secundario.

  > [!IMPORTANT]
  > Asegúrese de que el servidor secundario no tiene una base de datos con el mismo nombre, a menos que sea una base de datos secundaria existente. En los grupos de conmutación por error de Instancia administrada de SQL se replican todas las bases de datos de usuario. No se puede seleccionar un subconjunto de bases de datos de usuario para la replicación en el grupo de conmutación por error.

- **Incorporación de bases de datos de un grupo elástico a un grupo de conmutación por error**

  Puede poner varias bases de datos en el mismo grupo elástico e incluirlas en el mismo grupo de conmutación por error. Si la base de datos principal está en un grupo elástico, la base de datos secundaria se creará automáticamente en un grupo elástico con el mismo nombre (grupo secundario). Debe asegurarse de que el servidor secundario contiene un grupo elástico con el mismo nombre exacto y suficiente capacidad libre para hospedar las bases de datos secundarias que va a crear el grupo de conmutación por error. Si agrega una base de datos en un grupo que ya tiene una base de datos secundaria en el grupo secundario, el grupo heredará el vínculo de replicación geográfica. Cuando se agrega una base de datos que ya tiene una base de datos secundaria en un servidor que no forma parte del grupo de conmutación por error, se crea otra base de datos secundaria en el grupo secundario.
  
- **Propagación inicial**

  Al agregar bases de datos, grupos elásticos o instancias administradas a un grupo de conmutación por error, hay una fase de propagación inicial antes de que se inicie la replicación de datos. Esta fase de propagación inicial es la operación más larga y costosa. Una vez completada, se sincronizan los datos y, luego, solo se replican los cambios de datos posteriores. El tiempo que tarda en completarse la propagación inicial depende del tamaño de los datos, del número de bases de datos replicadas y de la velocidad del vínculo entre las entidades del grupo de conmutación por error. En circunstancias normales, la velocidad de propagación típica es de 50 a 500 GB por hora para SQL Database, y de 18 a 35 GB por hora en el caso Instancia administrada de SQL. La propagación se realiza en paralelo para todas las bases de datos. Puede usar la velocidad de propagación indicada, junto con el número de bases de datos y el tamaño total de los datos para calcular cuánto tiempo tardará la fase de propagación inicial antes de que se inicie la replicación de los datos.

  En el caso de Instancia administrada de SQL, al estimar el tiempo de la fase de propagación inicial también se debe tener en cuenta la velocidad del vínculo de ExpressRoute entre las dos instancias. Si la velocidad del vínculo entre las dos instancias es más lenta de lo necesario, es probable que el tiempo de propagación resulte muy afectado. Puede usar la velocidad de propagación indicada, el número de bases de datos, el tamaño total de los datos y la velocidad del vínculo para calcular cuánto tardará la fase de propagación inicial antes de que se inicie la replicación de los datos. Por ejemplo, en el caso de una sola base de datos de 100 GB, la fase de propagación inicial tardaría entre 2,8 y 5,5 horas si el vínculo es capaz de insertar 35 GB por hora. Si el vínculo solo puede transferir 10 GB por hora, la propagación de una base de datos de 100 GB tardará aproximadamente 10 horas. Si hay que replicar varias bases de datos, la propagación se ejecutará en paralelo y, cuando se combina con una velocidad de vínculo baja, la fase de propagación inicial puede tardar mucho más tiempo, en especial si la propagación paralela de los datos de todas las bases de datos supera el ancho de banda de vínculo disponible. Si el ancho de banda de red entre dos instancias es limitado y va a agregar varias instancias administradas a un grupo de conmutación por error, considere la posibilidad de hacerlo de manera secuencial, una por una.

- **Zona DNS**

  Identificador único que se genera automáticamente cuando se crea una Instancia administrada de SQL. Se aprovisiona un certificado de varios dominios (SAN) para esta instancia a fin de autenticar las conexiones de cliente a cualquier instancia de la misma zona DNS. Las dos instancias administradas en el mismo grupo de conmutación por error deben compartir la zona DNS.
  
  > [!NOTE]
  > No se requiere un identificador de zona DNS para los grupos de conmutación por error creados para SQL Database.

- **Agente de escucha de lectura-escritura de grupo de conmutación por error**

  Un registro CNAME de DNS que apunta a la dirección URL de la base de datos principal actual. Se crea automáticamente cuando se crea el grupo de conmutación por error y permite que la carga de trabajo de lectura y escritura se vuelva a conectar de forma transparente a la base de datos principal cuando esta cambia después de la conmutación por error. Cuando se crea el grupo de conmutación por error en un servidor, el registro CNAME de DNS para la dirección URL del cliente de escucha tiene el formato `<fog-name>.database.windows.net`. Cuando se crea el grupo de conmutación por error en Instancia administrada de SQL, el registro CNAME de DNS para la dirección URL del cliente de escucha tiene el formato `<fog-name>.zone_id.database.windows.net`.

- **Agente de escucha de solo lectura de grupo de conmutación por error**

  Un registro CNAME de DNS formado que apunta al agente de escucha de solo lectura que apunta a la dirección URL de la base de datos secundaria. Se crea automáticamente cuando se crea el grupo de conmutación por error y permite que la carga de trabajo de SQL de solo lectura se vuelva a conectar de forma transparente a la secundaria con las reglas de equilibrio de carga especificadas. Cuando se crea el grupo de conmutación por error en un servidor, el registro CNAME de DNS para la dirección URL del cliente de escucha tiene el formato `<fog-name>.secondary.database.windows.net`. Cuando se crea el grupo de conmutación por error en Instancia administrada de SQL, el registro CNAME de DNS para la dirección URL del cliente de escucha tiene el formato `<fog-name>.zone_id.secondary.database.windows.net`.

- **Directiva de conmutación por error automática**

  De manera predeterminada, un grupo de conmutación por error se configura con una directiva de conmutación por error automática. Azure desencadena la conmutación por error cuando se detecta el error y el período de gracia ha expirado. El sistema debe comprobar que la interrupción no se pueda mitigar mediante la [infraestructura de alta disponibilidad](high-availability-sla.md) integrada debido a la escala del impacto. Si desea controlar el flujo de trabajo de la conmutación por error desde la aplicación, puede desactivar la conmutación por error automática.
  
  > [!NOTE]
  > Dado que la comprobación de la escala de la interrupción y la rapidez con que se puede mitigar conllevan acciones humanas por parte del equipo de operaciones, el período de gracia no se puede establecer por debajo de una hora. Esta limitación se aplica a todas las bases de datos del grupo de conmutación por error, independientemente de su estado de sincronización de datos.

- **Directiva de conmutación por error de solo lectura**

  De forma predeterminada, se deshabilita la conmutación por error del agente de escucha de solo lectura. Se asegura de que el rendimiento de la réplica principal no se ve afectado cuando la base de datos secundaria está sin conexión. En cambio, también significa que las sesiones de solo lectura no podrán conectarse hasta que se recupere la base de datos secundaria. Si no puede tolerar tiempo de inactividad en las sesiones de solo lectura y le parece correcto usar temporalmente la base de datos principal para el tráfico de solo lectura y el de lectura y escritura a costa de la posible degradación del rendimiento de esta, puede habilitar la conmutación por error para el agente de escucha de solo lectura mediante la configuración de la propiedad `AllowReadOnlyFailoverToPrimary`. En ese caso, el tráfico de solo lectura se redirigirá automáticamente a la base de datos principal si la secundaria no está disponible.

- **Conmutación por error planeada**

   La conmutación por error planeada realiza una sincronización completa entre las bases de datos principales y secundarias antes de que el rol principal pase a la secundaria. De esta manera se garantiza que no hay pérdida de datos. La conmutación por error planeada se usa en los siguientes escenarios:

  - Realizar simulacros de recuperación ante desastres (DR) en producción cuando no es aceptable la pérdida de datos
  - Reubicar las bases de datos a una región diferente
  - Devolver las bases de datos a la región primaria después de que se ha corregido la interrupción (conmutación por recuperación)

- **Conmutación por error no planeada**

   La conmutación por error no planeada o forzada cambia inmediatamente el rol principal a la base de datos secundaria sin sincronización con la principal. Esta operación dará lugar a la pérdida de datos. La conmutación por error no planeada se usa como método de recuperación durante las interrupciones cuando no es posible acceder a la base de datos principal. Cuando la base de datos principal original vuelve a estar en línea, se conecta de nuevo automáticamente sin sincronización y se convierte en una nueva base de datos secundaria.

- **Conmutación por error manual**

  Puede iniciar manualmente la conmutación por error en cualquier momento, independientemente de la configuración de la conmutación por error automática. Si la directiva de conmutación por error automática no está configurada, se requiere la conmutación por error manual para recuperar las bases de datos del grupo de conmutación por error a la base de datos secundaria. Puede iniciar una conmutación por error forzada o sencilla (con sincronización total de los datos). El segundo caso se podría usar para reubicar la base de datos principal en la región secundaria. Cuando se completa la conmutación por error, los registros de DNS se actualizan automáticamente para garantizar la conectividad a la nueva base de datos principal.

- **Período de gracia con pérdida de datos**

  Como las bases de datos principal y secundaria se sincronizan con la replicación asincrónica, la conmutación por error puede provocar pérdida de datos. Puede personalizar la directiva de conmutación por error automática para que refleje la tolerancia de la aplicación a la pérdida de datos. Si configura `GracePeriodWithDataLossHours`, puede controlar durante cuánto tiempo espera el sistema antes de iniciar la conmutación por error que es probable que genere una pérdida de datos.

- **Varios grupos de conmutación por error**

  Puede configurar varios grupos de conmutación por error para el mismo par de servidores a fin de controlar la escala de conmutaciones por error. Cada grupo realiza la conmutación por error por separado. Si la aplicación multiinquilino usa grupos elásticos, puede usar esta funcionalidad para combinar bases de datos principales y secundarias en cada grupo. De este modo, puede reducir el impacto de una interrupción a solo la mitad de los inquilinos.

  > [!NOTE]
  > Instancia administrada de SQL no admite varios grupos de conmutación por error.
  
## <a name="permissions"></a>Permisos

Los permisos para un grupo de conmutación por error se administran a través del [control de acceso basado en rol (RBAC)](../../role-based-access-control/overview.md). El rol [Colaborador de SQL Server](../../role-based-access-control/built-in-roles.md#sql-server-contributor) tiene todos los permisos necesarios para administrar grupos de conmutación por error.

### <a name="create-failover-group"></a>Creación de un grupo de conmutación por error

Para crear un grupo de conmutación por error, necesita acceso de escritura RBAC a los servidores principales y secundarios, así como a todas las bases de datos del grupo de conmutación por error. Para una Instancia administrada de SQL, necesita acceso de escritura RBAC tanto a la instancia principal como a la secundaria de Instancia administrada de SQL, pero los permisos para las bases de datos individuales no son relevantes, puesto que no pueden agregarse bases de datos individuales de Instancia administrada de SQL a un grupo de conmutación por error ni eliminarse de este.

### <a name="update-a-failover-group"></a>Actualización de un grupo de conmutación por error

Para actualizar un grupo de conmutación por error, necesita acceso de escritura RBAC para el grupo de conmutación por error y para todas las bases de datos del servidor principal actual o instancia administrada.  

### <a name="fail-over-a-failover-group"></a>Conmutación por error de un grupo de conmutación por error

Para conmutar por error un grupo de conmutación por error, necesita acceso de escritura RBAC para el grupo de conmutación por error del nuevo servidor principal o instancia administrada.

## <a name="best-practices-for-sql-database"></a>Procedimientos recomendados para SQL Database

El grupo de conmutación por error automática debe estar configurado en el servidor principal y se conectará al servidor secundario de una región de Azure diferente. Los grupos pueden incluir todas las bases de datos de estos servidores o algunas de ellas. En el siguiente diagrama se ilustra una configuración típica de una aplicación de nube con redundancia geográfica que usa varias bases de datos y un grupo de conmutación por error automática.

![conmutación por error automática](./media/auto-failover-group-overview/auto-failover-group.png)

> [!NOTE]
> Consulte [Adición de SQL Database a un grupo de conmutación por error](failover-group-add-single-database-tutorial.md) para obtener un tutorial detallado paso a paso sobre la adición de una base de datos de SQL Database a un grupo de conmutación por error.

Al diseñar un servicio teniendo en cuenta la continuidad empresarial, siga estas directrices:

### <a name="using-one-or-several-failover-groups-to-manage-failover-of-multiple-databases"></a>Uso de uno o varios grupos de conmutación por error para administrar la conmutación por error de varias bases de datos

Se pueden crear uno o más grupos de conmutación por error entre dos servidores en regiones distintas (servidor principal y servidor secundario). Cada grupo puede incluir una o varias bases de datos que se recuperan como unidad en caso de que una o todas las bases de datos principales dejen de estar disponibles debido a una interrupción en la región primaria. El grupo de conmutación por error crea la base de datos geográfica secundaria con el mismo objetivo de servicio que la principal. Si agrega una relación de replicación geográfica existente al grupo de conmutación por error, asegúrese de que la base de datos geográfica secundaria esté configurada con el mismo nivel de servicio y tamaño de proceso que la principal.
  
> [!IMPORTANT]
> La creación de grupos de conmutación por error entre dos servidores en distintas suscripciones no se admite actualmente en Azure SQL Database. Si migra el servidor principal o secundario a otra suscripción una vez creado el grupo de conmutación por error, podría dar lugar a errores en las solicitudes de conmutación por error y en otras operaciones.

### <a name="using-read-write-listener-for-oltp-workload"></a>Uso del agente de escucha de lectura-escritura para la carga de trabajo de OLTP

Al realizar operaciones OLTP, use `<fog-name>.database.windows.net` como dirección URL del servidor y las conexiones se redirigirán automáticamente a la base de datos principal. Esta dirección URL no cambia después de la conmutación por error. Tenga en cuenta que la conmutación por error implica actualizar el registro DNS para que las conexiones de cliente se redirijan a la nueva base de datos principal cuando la caché DNS del cliente se haya actualizado.

### <a name="using-read-only-listener-for-read-only-workload"></a>Uso del agente de escucha de solo lectura para la carga de trabajo de solo lectura

Si tiene una carga de trabajo de solo lectura que es tolerante a una cierta obsolescencia de los datos, puede usar la base de datos secundaria en la aplicación. Para las sesiones de solo lectura, use `<fog-name>.secondary.database.windows.net` como dirección URL del servidor y la conexión se redirigirá automáticamente a la base de datos secundaria. También se recomienda indicar en la cadena de conexión la intención de lectura mediante `ApplicationIntent=ReadOnly`. Si quiere asegurarse de que la carga de trabajo de solo lectura se puede volver a conectar después de la conmutación por error o en caso de que el servidor secundario se quede sin conexión, asegúrese de configurar la propiedad `AllowReadOnlyFailoverToPrimary` de la directiva de conmutación por error.

### <a name="preparing-for-performance-degradation"></a>Preparación para la degradación del rendimiento

Una aplicación de Azure típica usa varios servicios de Azure y consta de varios componentes. La conmutación por error automatizada del grupo de conmutación por error se desencadena en función del estado de los componentes de Azure SQL. Es posible que otros servicios de Azure de la región primaria no se vean afectados por la interrupción y que sus componentes sigan estando disponibles en esa región. Una vez que las bases de datos principales cambien a la región de DR, la latencia entre los componentes dependientes puede aumentar. Para evitar el impacto de una mayor latencia en el rendimiento de la aplicación, compruebe la redundancia de todos los componentes de la aplicación en la región de DR y siga estas [instrucciones de seguridad de red](#failover-groups-and-network-security).

### <a name="preparing-for-data-loss"></a>Preparación para la pérdida de datos

Si se detecta una interrupción,Azure espera durante el período especificado por `GracePeriodWithDataLossHours`. El valor predeterminado es 1 hora. Si no puede permitirse perder datos, asegúrese de establecer `GracePeriodWithDataLossHours` en un número lo suficientemente grande, por ejemplo, 24 horas. Use un grupo de conmutación por error manual para conmutar por recuperación la base de datos secundaria a la principal.

> [!IMPORTANT]
> Los grupos elásticos con 800 o menos DTU y más de 250 bases de datos que utilizan la replicación geográfica pueden encontrar problemas, como conmutaciones por error planeadas más prolongadas y un menor rendimiento.  Es más probable que estos problemas sucedan con cargas de trabajo intensivas de escritura, cuando los puntos de conexión de replicación geográfica están separados por región geográfica, o cuando se utilizan varios puntos de conexión secundarias para cada base de datos.  Los síntomas de estos problemas aparecen si el intervalo de replicación geográfica aumenta con el tiempo.  Este retardo puede supervisarse con [sys.dm_geo_replication_link_status](/sql/relational-databases/system-dynamic-management-views/sys-dm-geo-replication-link-status-azure-sql-database).  Si se producen estos problemas, entre las posibles soluciones está el aumento del número de DTU del grupo o la reducción del número de bases de datos con replicación geográfica en el mismo grupo.

### <a name="changing-secondary-region-of-the-failover-group"></a>Cambio de la región secundaria del grupo de conmutación por error

Para ilustrar la secuencia de cambios, supondremos que el servidor A es el principal, el servidor B es el servidor secundario existente y el servidor C es la nueva instancia secundaria de la tercera región.  Para realizar la transición, siga estos pasos:

1. Cree instancias secundarias adicionales de cada base de datos del servidor A en el servidor C mediante la [replicación geográfica activa](active-geo-replication-overview.md). Cada base de datos del servidor A tendrá dos secundarios, uno en el servidor B y otro en el servidor C. Esto garantizará que las bases de datos principales permanezcan protegidas durante la transición.
2. Elimine el grupo de conmutación por error. En este momento, se producirá un error en los inicios de sesión. Esto se debe a que se han eliminado los alias de SQL para los agentes de escucha del grupo de conmutación por error y la puerta de enlace no reconoce el nombre del grupo de conmutación por error.
3. Vuelva a crear el grupo de conmutación por error con el mismo nombre entre los servidores A y C. En este momento, se dejarán de producir errores en los inicios de sesión.
4. Agregue todas las bases de datos principales del servidor A al nuevo grupo de conmutación por error.
5. Quite el servidor B. Todas las bases de datos de dicho servidor se eliminarán automáticamente.

### <a name="changing-primary-region-of-the-failover-group"></a>Cambio de la región primaria del grupo de conmutación por error

Para ilustrar la secuencia de cambios, supondremos que el servidor A es el servidor principal, el servidor B es el servidor secundario existente y el servidor C es la nueva instancia principal de la tercera región.  Para realizar la transición, siga estos pasos:

1. Realice una conmutación por error planeada para cambiar el servidor principal a B. El servidor A se convertirá en el nuevo servidor secundario. La conmutación por error puede producir varios minutos de tiempo de inactividad. El tiempo real dependerá del tamaño del grupo de conmutación por error.
2. Cree instancias secundarias adicionales de cada base de datos del servidor B en el servidor C mediante la [replicación geográfica activa](active-geo-replication-overview.md). Cada base de datos del servidor B tendrá dos secundarios, uno en el servidor A y otro en el servidor C. Esto garantizará que las bases de datos principales permanezcan protegidas durante la transición.
3. Elimine el grupo de conmutación por error. En este momento, se producirá un error en los inicios de sesión. Esto se debe a que se han eliminado los alias de SQL para los agentes de escucha del grupo de conmutación por error y la puerta de enlace no reconoce el nombre del grupo de conmutación por error.
4. Vuelva a crear el grupo de conmutación por error con el mismo nombre entre los servidores A y C. En este momento, se dejarán de producir errores en los inicios de sesión.
5. Agregue todas las bases de datos principales del servidor B al nuevo grupo de conmutación por error.
6. Realice una conmutación por error planeada del grupo de conmutación por error para cambiar los servidores B y C. Ahora, el servidor C será el principal y el B será el secundario. Todas las bases de datos secundarias del servidor A se vincularán automáticamente a las instancias principales de C. Como en el paso 1, la conmutación por error puede producir varios minutos de tiempo de inactividad.
7. Quite el servidor A. Todas las bases de datos de dicho servidor se eliminarán automáticamente.

> [!IMPORTANT]
> Cuando se elimina el grupo de conmutación por error, también se eliminan los registros de DNS de los puntos de conexión del agente de escucha. En ese momento, hay una probabilidad distinta de cero de otra persona cree un grupo de conmutación por error o un alias de servidor con el mismo nombre, lo que le impedirá volver a usarlo. Para minimizar el riesgo, no utilice nombres de grupo de conmutación por error genéricos.

## <a name="best-practices-for-sql-managed-instance"></a>Procedimientos recomendados para Instancia administrada de SQL

El grupo de conmutación por error automática debe estar configurado en la instancia principal y se conectará a la instancia secundaria de una región de Azure diferente.  Todas las bases de datos de la instancia se replicarán en la instancia secundaria.

En el siguiente diagrama se ilustra una configuración típica de una aplicación de nube con redundancia geográfica que usa instancia administrada y un grupo de conmutación por error automática.

![conmutación por error automática](./media/auto-failover-group-overview/auto-failover-group-mi.png)

> [!NOTE]
> Consulte [Adición de una instancia administrada a un grupo de conmutación por error](../managed-instance/failover-group-add-instance-tutorial.md) para obtener un tutorial detallado paso a paso para la adición de una Instancia administrada de SQL para usar un grupo de conmutación por error.

Si la aplicación usa Instancia administrada de SQL como capa de datos, siga estas directrices generales al realizar el diseño para la continuidad empresarial:

### <a name="creating-the-secondary-instance"></a>Creación de la instancia secundaria

Para garantizar la conectividad sin interrupciones a la Instancia administrada de SQL principal después de la conmutación por error, las instancias principales y secundarias deben estar en la misma zona DNS. Esto garantizará que se pueda usar el mismo certificado de varios dominios (SAN) para autenticar las conexiones de cliente a cualquiera de las dos instancias del grupo de conmutación por error. Cuando la aplicación esté lista para la implementación en producción, cree una Instancia administrada de SQL secundaria en una región distinta y asegúrese de que comparte la zona DNS con la Instancia administrada de SQL principal. Para ello, especifique un parámetro `DNS Zone Partner` opcional mediante Azure Portal, PowerShell o la API de REST.

> [!IMPORTANT]
> La primera Instancia administrada creada en la subred determina la zona DNS de todas las instancias posteriores de la misma subred. Esto significa que dos instancias de la misma subred no pueden pertenecer a zonas DNS diferentes.

Para obtener más información sobre cómo crear la Instancia administrada de SQL secundaria en la misma zona DNS que la instancia principal, consulte [Creación de una instancia administrada secundaria](../managed-instance/failover-group-add-instance-tutorial.md#3---create-a-secondary-managed-instance).

### <a name="enabling-replication-traffic-between-two-instances"></a>Habilitación del tráfico de replicación entre dos instancias

Dado que cada instancia está aislada en su propia red virtual, se debe permitir el tráfico bidireccional entre estas redes virtuales. Consulte [Azure VPN Gateway](../../vpn-gateway/vpn-gateway-about-vpngateways.md).

### <a name="creating-a-failover-group-between-managed-instances-in-different-subscriptions"></a>Creación de un grupo de conmutación por error entre instancias administradas en distintas suscripciones

Puede crear un grupo de conmutación por error entre instancias de SQL Managed Instance en dos suscripciones diferentes, siempre que las suscripciones estén asociadas al mismo [inquilino de Azure Active Directory](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-whatis#terminology). Al usar la API de PowerShell, puede hacerlo especificando el parámetro `PartnerSubscriptionId` de la instancia secundaria de SQL Managed Instance. Al usar la API REST, cada identificador de instancia incluido en el parámetro `properties.managedInstancePairs` puede tener su propio subscriptionID.
  
> [!IMPORTANT]
> Azure Portal no admite la creación de grupos de conmutación por error en distintas suscripciones. Además, para los grupos de conmutación por error existentes en distintas suscripciones o grupos de recursos, no se puede iniciar manualmente la conmutación por error a través del portal desde la Instancia administrada de SQL principal. En su lugar, se inicia desde la instancia de la base de datos geográfica secundaria.

### <a name="managing-failover-to-secondary-instance"></a>Administración de la conmutación por error en la instancia secundaria

El grupo de conmutación por error administrará la conmutación por error de todas las bases de datos de la Instancia administrada de SQL. Cuando se crea un grupo, cada base de datos de la instancia se replicará geográficamente de manera automática a la Instancia administrada de SQL. No se pueden usar grupos de conmutación por error para iniciar una conmutación por error parcial de un subconjunto de las bases de datos.

> [!IMPORTANT]
> Si se quita una base de datos de la Instancia administrada de SQL principal, también se eliminará automáticamente en la Instancia administrada de SQL secundaria de replicación geográfica.

### <a name="using-read-write-listener-for-oltp-workload"></a>Uso del agente de escucha de lectura-escritura para la carga de trabajo de OLTP

Al realizar operaciones OLTP, use `<fog-name>.zone_id.database.windows.net` como dirección URL del servidor y las conexiones se redirigirán automáticamente a la base de datos principal. Esta dirección URL no cambia después de la conmutación por error. La conmutación por error implica actualizar el registro DNS para que las conexiones de cliente se redirijan a la nueva base de datos principal cuando la caché DNS del cliente se haya actualizado. Dado que la instancia secundaria comparte la zona DNS con la principal, la aplicación cliente podrá volver a conectarse a ella con el mismo certificado de SAN.

### <a name="using-read-only-listener-to-connect-to-the-secondary-instance"></a>Uso del agente de escucha de solo lectura para conectarse a la instancia secundaria

Si tiene una carga de trabajo de solo lectura que es tolerante a una cierta obsolescencia de los datos, puede usar la base de datos secundaria en la aplicación. Para conectarse directamente a la base de datos secundaria con replicación geográfica, use `server.secondary.zone_id.database.windows.net` como dirección URL del servidor y la conexión se realizará directamente a dicha base de datos.

> [!NOTE]
> En determinados niveles de servicio, SQL Database admite el uso de [réplicas de solo lectura](read-scale-out.md) para equilibrar las cargas de trabajo de consultas de solo lectura mediante la capacidad de una réplica de solo lectura y el uso del parámetro `ApplicationIntent=ReadOnly` en la cadena de conexión. Cuando se ha configurado una base de datos secundaria con replicación geográfica, puede usar esta funcionalidad para conectarse a una réplica de solo lectura de la ubicación principal o de la ubicación con replicación geográfica.
>
> - Para conectarse a una réplica de solo lectura en la ubicación principal, use `<fog-name>.zone_id.database.windows.net`.
> - Para conectarse a una réplica de solo lectura en la ubicación secundaria, use `<fog-name>.secondary.zone_id.database.windows.net`.

### <a name="preparing-for-performance-degradation"></a>Preparación para la degradación del rendimiento

Una aplicación de Azure típica usa varios servicios de Azure y consta de varios componentes. La conmutación por error automatizada del grupo de conmutación por error se desencadena en función del estado de los componentes de Azure SQL. Es posible que otros servicios de Azure de la región primaria no se vean afectados por la interrupción y que sus componentes sigan estando disponibles en esa región. Una vez que las bases de datos principales cambien a la región de DR, la latencia entre los componentes dependientes puede aumentar. Para evitar el impacto de una mayor latencia en el rendimiento de la aplicación, compruebe la redundancia de todos los componentes de la aplicación en la región de DR y siga estas [instrucciones de seguridad de red](#failover-groups-and-network-security).

### <a name="preparing-for-data-loss"></a>Preparación para la pérdida de datos

Si se detecta una interrupción, se desencadena automáticamente una conmutación por error de lectura/escritura si nos consta que hay cero pérdida de datos. En caso contrario, espera el período especificado. En caso contrario, espera el período especificado por `GracePeriodWithDataLossHours`. Si especificó `GracePeriodWithDataLossHours`, prepárese para la pérdida de datos. Por lo general, durante las interrupciones, Azure favorece la disponibilidad. Si no puede permitirse perder datos, asegúrese de establecer GracePeriodWithDataLossHours en un número lo suficientemente grande, por ejemplo, 24 horas.

La actualización de DNS del agente de escucha de lectura y escritura sucederá inmediatamente después de que se inicie la conmutación por error. Esta operación no ocasionará pérdida de datos. Sin embargo, el proceso de conmutación de roles de base de datos puede tardar hasta 5 minutos en condiciones normales. Hasta que se complete, algunas bases de datos de la nueva instancia principal seguirán siendo de solo lectura. Si se inicia la conmutación por error mediante PowerShell, toda la operación es sincrónica. Si se inicia mediante Azure Portal, la interfaz de usuario indicará el estado de finalización. Si se inicia mediante la API REST, use el mecanismo de sondeo estándar de Azure Resource Manager para supervisar la finalización.

> [!IMPORTANT]
> Use un grupo de conmutación por error manual para mover las bases de datos principales de vuelta a la ubicación original. Cuando se corrija la interrupción que causó la conmutación por error, puede mover las bases de datos principales a la ubicación original. Para ello, debe iniciar la conmutación por error manual del grupo.
  
### <a name="changing-secondary-region-of-the-failover-group"></a>Cambio de la región secundaria del grupo de conmutación por error

Supongamos que la instancia A es la principal, la B es la instancia secundaria existente y la C es la nueva instancia secundaria de la tercera región.  Para realizar la transición, siga estos pasos:

1. Cree la instancia C con el mismo tamaño que la instancia A y en la misma zona DNS.
2. Elimine el grupo de conmutación por error entre las instancias A y B. En este momento, se producirá un error en los inicios de sesión porque se han eliminado los alias de SQL de los agentes de escucha del grupo de conmutación por error y la puerta de enlace no reconocerá el nombre del grupo de conmutación por error. Las bases de datos secundarias se desconectarán de las principales y se convertirán en bases de datos de lectura y escritura.
3. Cree un grupo de conmutación por error con el mismo nombre entre las instancias A y C. Siga las instrucciones del [tutorial sobre el grupo de conmutación por error con Instancia administrada de SQL](../managed-instance/failover-group-add-instance-tutorial.md). Se trata de una operación de tamaño de datos, que se completará cuando todas las bases de datos de la instancia A se inicialicen y se sincronicen.
4. Si no es necesario, elimine la instancia B para evitar cargos innecesarios.

> [!NOTE]
> Después del paso 2 y hasta que se complete el paso 3, las bases de datos de la instancia A permanecerán desprotegidas frente a un error catastrófico en la instancia A.

### <a name="changing-primary-region-of-the-failover-group"></a>Cambio de la región primaria del grupo de conmutación por error

Supongamos que la instancia A es la principal, la B es la instancia secundaria existente y la C es la nueva instancia principal de la tercera región.  Para realizar la transición, siga estos pasos:

1. Cree la instancia C con el mismo tamaño que la instancia B y en la misma zona DNS.
2. Conéctese a la instancia B y conmute por error manualmente para cambiar la instancia principal a B. La instancia A se convertirá en la nueva instancia secundaria automáticamente.
3. Elimine el grupo de conmutación por error entre las instancias A y B. En este momento, se producirá un error en los inicios de sesión porque se han eliminado los alias de SQL de los agentes de escucha del grupo de conmutación por error y la puerta de enlace no reconocerá el nombre del grupo de conmutación por error. Las bases de datos secundarias se desconectarán de las principales y se convertirán en bases de datos de lectura y escritura.
4. Cree un grupo de conmutación por error con el mismo nombre entre las instancias A y C. Siga las instrucciones del [tutorial sobre el grupo de conmutación por error con instancia administrada](../managed-instance/failover-group-add-instance-tutorial.md). Se trata de una operación de tamaño de datos, que se completará cuando todas las bases de datos de la instancia A se inicialicen y se sincronicen.
5. Si no es necesario, elimine la instancia A para evitar cargos innecesarios.

> [!CAUTION]
> Después del paso 3 y hasta que se complete el paso 4, las bases de datos de la instancia A permanecerán desprotegidas frente a un error catastrófico en la instancia A.

> [!IMPORTANT]
> Cuando se elimina el grupo de conmutación por error, también se eliminan los registros de DNS de los puntos de conexión del agente de escucha. En ese momento, hay una probabilidad distinta de cero de otra persona cree un grupo de conmutación por error o un alias de servidor con el mismo nombre, lo que le impedirá volver a usarlo. Para minimizar el riesgo, no utilice nombres de grupo de conmutación por error genéricos.

### <a name="enable-scenarios-dependent-on-objects-from-the-system-databases"></a>Habilitación de escenarios que dependen de objetos de las bases de datos del sistema
Las bases de datos del sistema no se replican en la instancia secundaria de un grupo de conmutación por error. Para habilitar escenarios que dependen de objetos de las bases de datos del sistema, asegúrese de crear los mismos objetos en la instancia secundaria. Por ejemplo, si tiene previsto utilizar los mismos inicios de sesión en la instancia secundaria, asegúrese de crearlos con el SID idéntico. 
```SQL
-- Code to create login on the secondary instance
CREATE LOGIN foo WITH PASSWORD = '<enterStrongPasswordHere>', SID = <login_sid>;
``` 


## <a name="failover-groups-and-network-security"></a>Grupos de conmutación por error y la seguridad de red

En algunas aplicaciones, las reglas de seguridad requieren que se restrinja el acceso de red a la capa de datos para un componente específico o a componentes como una máquina virtual, un servicio web, etc. Este requisito presenta algunos desafíos para el diseño de la continuidad empresarial y el uso de grupos de conmutación por error. Considere las siguientes opciones al implementar dicho acceso restringido.

### <a name="using-failover-groups-and-virtual-network-rules"></a>Uso de grupos de conmutación por error y reglas de red virtual

Si usa [reglas y puntos de conexión de servicio de Virtual Network](vnet-service-endpoint-rule-overview.md) para restringir el acceso a la base de datos en SQL Database o Instancia administrada de SQL, tenga en cuenta que cada punto de conexión de servicio de red virtual solo se aplica a una región de Azure. El punto de conexión no permite que otras regiones acepten la comunicación de la subred. Por lo tanto, solo las aplicaciones de cliente implementadas en la misma región pueden conectarse a la base de datos principal. Puesto que la conmutación por error provoca que las sesiones de cliente SQL Database se vuelvan a enrutar al servidor en una región diferente (secundaria), estas sesiones experimentarán un error si se originaron desde un cliente de fuera de esa región. Por ese motivo, la directiva de conmutación por error automática no se puede habilitar si se incluyen los servidores o las instancias implicados en las reglas de Virtual Network. Para admitir la conmutación por error manual, siga estos pasos:

1. Aprovisione las copias redundantes de los componentes front-end de la aplicación (servicio web, máquinas virtuales, etc.) en la región secundaria.
2. Configure las [reglas de red virtual](vnet-service-endpoint-rule-overview.md) individualmente para el servidor principal y secundario
3. Habilite la [conmutación por error front-end con una configuración de Traffic Manager](designing-cloud-solutions-for-disaster-recovery.md#scenario-1-using-two-azure-regions-for-business-continuity-with-minimal-downtime)
4. Inicie la conmutación por error manual cuando se detecte la interrupción. Esta opción está optimizada para las aplicaciones que requieren una latencia coherente entre la capa de datos y de front-end, y admite la recuperación cuando se ven afectadas por la interrupción la capa de datos, la capa front-end o ambas.

> [!NOTE]
> Si usa el **agente de escucha de solo lectura** para equilibrar una carga de trabajo de solo lectura, asegúrese de que esta carga de trabajo se ejecuta en una máquina virtual o en otro recurso en la región secundaria de modo que pueda conectarse a la base de datos secundaria.

### <a name="use-failover-groups-and-firewall-rules"></a>Uso de grupos de conmutación por error y reglas de firewall

Si el plan de continuidad empresarial requiere un proceso de conmutación por error mediante grupos con conmutación por error automática, puede restringir el acceso a la base de datos de SQL Database usando reglas de firewall tradicionales. Para admitir la conmutación por error automática, siga estos pasos:

1. [Cree una dirección IP pública](../../virtual-network/virtual-network-public-ip-address.md#create-a-public-ip-address).
2. [Cree un equilibrador de carga público](../../load-balancer/quickstart-load-balancer-standard-public-portal.md) y asígnele la dirección IP pública.
3. [Cree una red virtual y las máquinas virtuales](../../load-balancer/quickstart-load-balancer-standard-public-portal.md) para los componentes de front-end.
4. [Cree un grupo de seguridad de red](../../virtual-network/security-overview.md) y configure las conexiones entrantes.
5. Asegúrese de que las conexiones salientes están abiertas para Azure SQL Database usando la [etiqueta de servicio](../../virtual-network/security-overview.md#service-tags) "Sql".
6. Cree una [regla de firewall de SQL Database](firewall-configure.md) para permitir el tráfico entrante desde la dirección IP pública que creó en el paso 1.

Para más información acerca de cómo configurar el acceso de salida y qué dirección IP usar en las reglas de firewall, consulte [Conexiones salientes del equilibrador de carga](../../load-balancer/load-balancer-outbound-connections.md).

La configuración anterior garantizará que la conmutación por error automática no bloqueará las conexiones desde los componentes front-end y se da por supuesto que la aplicación puede tolerar la mayor latencia entre la capa de datos y el front-end.

> [!IMPORTANT]
> Para garantizar la continuidad empresarial ante interrupciones regionales, debe garantizar la redundancia geográfica tanto para los componentes front-end como para las bases de datos.

## <a name="enabling-geo-replication-between-managed-instances-and-their-vnets"></a>Habilitación de la replicación geográfica entre instancias administradas y sus redes virtuales

Al configurar un grupo de conmutación por error entre instancias administradas de SQL principales y secundarias en dos regiones diferentes, cada instancia se aísla mediante una red virtual independiente. Para permitir el tráfico de replicación entre estas redes virtuales, asegúrese de que se cumplen estos requisitos previos:

- Las dos instancias de Instancia administrada de SQL deben estar en diferentes regiones de Azure.
- Las dos instancias de Instancia administrada de SQL deben ser del mismo nivel de servicio y tener el mismo tamaño de almacenamiento.
- La instancia secundaria de Instancia administrada de SQL debe estar vacía (sin bases de datos de usuario).
- Las redes virtuales que se usan en las instancias de Instancia administrada de SQL deben estar conectadas mediante [VPN Gateway](../../vpn-gateway/vpn-gateway-about-vpngateways.md) o [ExpressRoute](../../expressroute/expressroute-howto-circuit-portal-resource-manager.md). Si dos redes virtuales se conectan a través de una red local, asegúrese de que no haya ninguna regla de firewall bloqueando los puertos 5022 y 11000-11999. No se admite el emparejamiento de VNet global.
- Las dos redes virtuales de Instancia administrada de SQL no pueden tener direcciones IP superpuestas.
- Debe configurar sus grupos de seguridad de red (NSG) de forma que los puertos 5022 y el intervalo 11000~12000 estén abiertos a las conexiones entrantes y salientes de la subred de la otra instancia administrada. Esto es para permitir el tráfico de replicación entre las instancias.

   > [!IMPORTANT]
   > Las reglas de seguridad de NSG mal configuradas dan lugar a operaciones de copia de base de datos bloqueadas.

- La Instancia administrada de SQL secundaria está configurada con el identificador de zona DNS correcto. La zona DNS es una propiedad de una Instancia administrada de SQL y un clúster virtual subyacente, cuyo identificador se incluye en la dirección del nombre de host. El identificador de zona se genera como una cadena aleatoria cuando se crea la primera Instancia administrada de SQL en cada red virtual, y este mismo identificador se asigna a todas las demás instancias de la misma subred. Una vez que se ha asignado, no se puede modificar la zona DNS. Las instancias de Instancia administrada de SQL incluidas en el mismo grupo de conmutación por error deben compartir la zona DNS. Para lograrlo, debe pasar el identificador de zona de la instancia principal como valor del parámetro DnsZonePartner al crear la instancia secundaria.

   > [!NOTE]
   > Consulte [Adición de una Instancia administrada de SQL a un grupo de conmutación por error](../managed-instance/failover-group-add-instance-tutorial.md) para obtener un tutorial detallado sobre la configuración de grupos de conmutación por error con Instancia administrada de SQL.

## <a name="upgrading-or-downgrading-a-primary-database"></a>Actualización o degradación de una base de datos principal

Puede actualizar o degradar una base de datos principal a un tamaño de proceso diferente (en el mismo nivel de servicio, no entre De uso general y Crítico para la empresa) sin desconectar las bases de datos secundarias. Al actualizar, se recomienda que actualice primero todas las bases de datos secundarias y, después, la principal. Al degradar, invierta el orden: degrade primero la base de datos principal y, luego, todas las secundarias. Cuando actualiza la base de datos a un nivel de servicio diferente, o la cambia a una versión anterior, se aplica esta recomendación.

Esta secuencia se recomienda específicamente para evitar que la base de datos secundaria de una SKU inferior se sobrecargue y deba propagarse durante un proceso de actualización o degradación. También puede evitar este problema si hace que la base de datos principal sea de solo lectura, a costa de afectar a todas las cargas de trabajo de lectura y escritura en la réplica principal.

> [!NOTE]
> Si creó una base de datos secundaria como parte de la configuración del grupo de conmutación por error, no se recomienda que la degrade. De este modo, se garantiza que la capa de datos tiene la capacidad suficiente para procesar la carga de trabajo habitual una vez que se activa la conmutación por error.

## <a name="preventing-the-loss-of-critical-data"></a>Evitar la pérdida de datos críticos

Debido a la elevada latencia de las redes de área extensa, la copia continua usa un mecanismo de replicación asincrónica. La replicación asincrónica hace inevitable la pérdida de cierta cantidad datos si se produce un error. Sin embargo, es posible que algunas aplicaciones exijan que no se pierdan datos. Para proteger estas actualizaciones críticas, un desarrollador de aplicaciones puede llamar al procedimiento del sistema [sp_wait_for_database_copy_sync](/sql/relational-databases/system-stored-procedures/active-geo-replication-sp-wait-for-database-copy-sync) inmediatamente después de confirmar la transacción. La llamada a `sp_wait_for_database_copy_sync` bloquea el subproceso de llamada hasta que se transmite la última transacción confirmada en la base de datos secundaria. Sin embargo, no espera para que las transacciones transmitidas se reproduzcan y se confirmen en la base de datos secundaria. `sp_wait_for_database_copy_sync` está en el ámbito de un vínculo de copia continua específico. Cualquier usuario con derechos de conexión para la base de datos principal puede llamar a este procedimiento.

> [!NOTE]
> `sp_wait_for_database_copy_sync` evita la pérdida de datos después de la conmutación por error, pero no garantiza la sincronización completa para el acceso de lectura. El retraso causado por una llamada al procedimiento `sp_wait_for_database_copy_sync` puede ser considerable y depende del tamaño del registro de transacciones en el momento de la llamada.

## <a name="failover-groups-and-point-in-time-restore"></a>Grupos de conmutación por error y restauración a un momento dado

Para información sobre el uso de la restauración a un momento dado con grupos de conmutación por error, consulte [Recuperación a un momento dado (PITR)](recovery-using-backups.md#point-in-time-restore).

## <a name="limitations-of-failover-groups"></a>Limitaciones de los grupos de conmutación por error

Tenga en cuenta las siguientes limitaciones:

- No se pueden crear grupos de conmutación por error entre dos servidores o instancias en las mismas regiones de Azure.
- No se puede cambiar el nombre de los grupos de conmutación por error. Tendrá que eliminar el grupo y volver a crearlo con un nombre diferente.
- No se admiten los cambios de nombre de base de datos para las instancias del grupo de conmutación por error. Tendrá que eliminar temporalmente el grupo de conmutación por error para poder cambiar el nombre a una base de datos.
- Las bases de datos del sistema no se replican en la instancia secundaria de un grupo de conmutación por error. Por lo tanto, los escenarios que dependen de objetos de las bases de datos del sistema serán imposibles en la instancia secundaria, a menos que los objetos se creen manualmente en la secundaria.

## <a name="programmatically-managing-failover-groups"></a>Administración mediante programación de grupos de conmutación por error

Como se ha mencionado antes, los grupos de conmutación automática por error y la replicación geográfica activa también pueden administrarse mediante programación con Azure PowerShell y la API REST. En las tablas siguientes se describe el conjunto de comandos disponibles. La replicación geográfica activa incluye un conjunto de API de Azure Resource Manager para la administración, en el que se incluyen la [API REST de Azure SQL Database](https://docs.microsoft.com/rest/api/sql/) y los [cmdlets de Azure PowerShell](https://docs.microsoft.com/powershell/azure/). Estas API requieren que se usen grupos de recursos y admiten la seguridad basada en roles (RBAC). Para más información sobre cómo implementar los roles de acceso, consulte [Control de acceso basado en roles de Azure](../../role-based-access-control/overview.md).

### <a name="manage-sql-database-failover"></a>Administración de la conmutación por error de SQL Database

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

| Cmdlet | Descripción |
| --- | --- |
| [New-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/new-azsqldatabasefailovergroup) |Este comando crea un grupo de conmutación por error y lo registra en el servidor principal y el servidor secundario|
| [Remove-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/remove-azsqldatabasefailovergroup) | Quita un grupo de conmutación por error del servidor. |
| [Get-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/get-azsqldatabasefailovergroup) | Recupera la configuración de un grupo de conmutación por error. |
| [Set-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/set-azsqldatabasefailovergroup) |Modifica la configuración de un grupo de conmutación por error. |
| [Switch-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/switch-azsqldatabasefailovergroup) | Desencadena la conmutación por error de un grupo de conmutación por error en el servidor secundario. |
| [Add-AzSqlDatabaseToFailoverGroup](/powershell/module/az.sql/add-azsqldatabasetofailovergroup)|Agrega una o varias bases de datos a un grupo de conmutación por error.|

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

| Get-Help | Descripción |
| --- | --- |
| [az sql failover-group create](/cli/azure/sql/failover-group#az-sql-failover-group-create) |Este comando crea un grupo de conmutación por error y lo registra en el servidor principal y el servidor secundario|
| [az sql failover-group delete](/cli/azure/sql/failover-group#az-sql-failover-group-delete) | Quita un grupo de conmutación por error del servidor. |
| [az sql failover-group show](/cli/azure/sql/failover-group#az-sql-failover-group-show) | Recupera la configuración de un grupo de conmutación por error. |
| [az sql failover-group update](/cli/azure/sql/failover-group#az-sql-failover-group-update) |Modifica la configuración de un grupo de conmutación por error, o bien agrega una o varias bases de datos a un grupo de conmutación por error.|
| [az sql failover-group set-primary](/cli/azure/sql/failover-group#az-sql-failover-group-set-primary) | Desencadena la conmutación por error de un grupo de conmutación por error en el servidor secundario. |

# <a name="rest-api"></a>[API DE REST](#tab/rest-api)

| API | Descripción |
| --- | --- |
| [Crear o actualizar grupo de conmutación por error](https://docs.microsoft.com/rest/api/sql/failovergroups/createorupdate) | Crea o actualiza un grupo de conmutación por error. |
| [Eliminar grupo de conmutación por error](https://docs.microsoft.com/rest/api/sql/failovergroups/delete) | Quita un grupo de conmutación por error del servidor. |
| [Conmutación por error (planeada)](https://docs.microsoft.com/rest/api/sql/failovergroups/failover) | Desencadena la conmutación por error del servidor principal actual en el servidor secundario con sincronización de datos completa.|
| [Forzar la conmutación por error ocasiona la pérdida de datos](https://docs.microsoft.com/rest/api/sql/failovergroups/forcefailoverallowdataloss) | Desencadena la conmutación por error del servidor principal actual en el servidor secundario sin sincronización de datos. Esta operación puede ocasionar pérdida de datos. |
| [Obtener grupo de conmutación por error](https://docs.microsoft.com/rest/api/sql/failovergroups/get) | Recupera la configuración de un grupo de conmutación por error. |
| [Enumerar grupos de conmutación por error por servidor](https://docs.microsoft.com/rest/api/sql/failovergroups/listbyserver) | Enumera los grupos de conmutación por error en un servidor. |
| [Actualizar grupo de conmutación por error](https://docs.microsoft.com/rest/api/sql/failovergroups/update) | Actualiza la configuración de un grupo de conmutación por error. |

---

### <a name="manage-sql-managed-instance-failover"></a>Administración de la conmutación por error de Instancia administrada de SQL


# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

| Cmdlet | Descripción |
| --- | --- |
| [New-AzSqlDatabaseInstanceFailoverGroup](/powershell/module/az.sql/new-azsqldatabaseinstancefailovergroup) |Este comando crea un grupo de conmutación por error y lo registra en las instancias principal y secundaria.|
| [Set-AzSqlDatabaseInstanceFailoverGroup](/powershell/module/az.sql/set-azsqldatabaseinstancefailovergroup) |Modifica la configuración de un grupo de conmutación por error.|
| [Get-AzSqlDatabaseInstanceFailoverGroup](/powershell/module/az.sql/get-azsqldatabaseinstancefailovergroup) |Recupera la configuración de un grupo de conmutación por error.|
| [Switch-AzSqlDatabaseInstanceFailoverGroup](/powershell/module/az.sql/switch-azsqldatabaseinstancefailovergroup) |Desencadena la conmutación por error de un grupo de conmutación por error en la instancia secundaria.|
| [Remove-AzSqlDatabaseInstanceFailoverGroup](/powershell/module/az.sql/remove-azsqldatabaseinstancefailovergroup) | Quita un grupo de conmutación por error.|


# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

| Get-Help | Descripción |
| --- | --- |
| [az sql failover-group create](/cli/azure/sql/failover-group#az-sql-failover-group-create) |Este comando crea un grupo de conmutación por error y lo registra en el servidor principal y el servidor secundario|
| [az sql failover-group delete](/cli/azure/sql/failover-group#az-sql-failover-group-delete) | Quita un grupo de conmutación por error del servidor. |
| [az sql failover-group show](/cli/azure/sql/failover-group#az-sql-failover-group-show) | Recupera la configuración de un grupo de conmutación por error. |
| [az sql failover-group update](/cli/azure/sql/failover-group#az-sql-failover-group-update) |Modifica la configuración de un grupo de conmutación por error, o bien agrega una o varias bases de datos a un grupo de conmutación por error.|
| [az sql failover-group set-primary](/cli/azure/sql/failover-group#az-sql-failover-group-set-primary) | Desencadena la conmutación por error de un grupo de conmutación por error en el servidor secundario. |

# <a name="rest-api"></a>[API DE REST](#tab/rest-api)

| API | Descripción |
| --- | --- |
| [Crear o actualizar grupo de conmutación por error](https://docs.microsoft.com/rest/api/sql/instancefailovergroups/createorupdate) | Crea o actualiza la configuración de un grupo de conmutación por error. |
| [Eliminar grupo de conmutación por error](https://docs.microsoft.com/rest/api/sql/instancefailovergroups/delete) | Quita un grupo de conmutación por error de la instancia. |
| [Conmutación por error (planeada)](https://docs.microsoft.com/rest/api/sql/instancefailovergroups/failover) | Desencadena la conmutación por error del servidor principal actual en esta instancia con sincronización de datos completa. |
| [Forzar la conmutación por error ocasiona la pérdida de datos](https://docs.microsoft.com/rest/api/sql/instancefailovergroups/forcefailoverallowdataloss) | Desencadena la conmutación por error de la instancia principal actual en la instancia secundaria sin sincronización de datos. Esta operación puede ocasionar pérdida de datos. |
| [Obtener grupo de conmutación por error](https://docs.microsoft.com/rest/api/sql/instancefailovergroups/get) | Recupera la configuración de un grupo de conmutación por error. |
| [Lista de grupos de conmutación por error: por ubicación](https://docs.microsoft.com/rest/api/sql/instancefailovergroups/listbylocation) | Enumera los grupos de conmutación por error de una ubicación. |

---

## <a name="next-steps"></a>Pasos siguientes

- Para ver tutoriales detallados, consulte
  - [Adición de SQL Database a un grupo de conmutación por error](failover-group-add-single-database-tutorial.md)
  - [Adición de un grupo elástico a un grupo de conmutación por error](failover-group-add-elastic-pool-tutorial.md)
  - [Adición de una Instancia administrada de SQL a un grupo de conmutación por error](../managed-instance/failover-group-add-instance-tutorial.md)
- Para los scripts de ejemplo, vea:
  - [Uso de PowerShell para configurar la replicación geográfica activa para Azure SQL Database](scripts/setup-geodr-and-failover-database-powershell.md)
  - [Uso de PowerShell para configurar la replicación geográfica activa para una base de datos agrupada en Azure SQL Database](scripts/setup-geodr-and-failover-elastic-pool-powershell.md)
  - [Uso de PowerShell para agregar una instancia de Azure SQL Database a un grupo de conmutación por error](scripts/add-database-to-failover-group-powershell.md)
- Para obtener una descripción general y los escenarios de la continuidad empresarial, consulte [Continuidad empresarial con Base de datos SQL de Azure](business-continuity-high-availability-disaster-recover-hadr-overview.md)
- Para saber en qué consisten las copias de seguridad automatizadas de Azure SQL Database, consulte [Información general: copias de seguridad automatizadas de SQL Database](automated-backups-overview.md).
- Si quiere saber cómo usar las copias de seguridad automatizadas para procesos de recuperación, consulte [Recuperación de una base de datos a partir de copias de seguridad iniciadas por un servicio](recovery-using-backups.md).
- Para obtener información acerca de los requisitos de autenticación para un nuevo servidor principal y la base de datos, consulte [Administración de la seguridad de Azure SQL Database después de la recuperación ante desastres](active-geo-replication-security-configure.md).
