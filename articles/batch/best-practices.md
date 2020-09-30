---
title: Procedimientos recomendados
description: Obtenga información sobre los procedimientos recomendados y sugerencias útiles para desarrollar su solución de Azure Batch.
ms.date: 08/12/2020
ms.topic: conceptual
ms.openlocfilehash: ca6e491586fd653f39da7466ea116109000facd6
ms.sourcegitcommit: d7352c07708180a9293e8a0e7020b9dd3dd153ce
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/30/2020
ms.locfileid: "89146545"
---
# <a name="azure-batch-best-practices"></a>Procedimientos recomendados de Azure Batch

En este artículo se describe una colección de procedimientos recomendados para usar el servicio de Azure Batch de forma eficaz y eficiente, en función de la experiencia real con Batch. Lea este artículo para evitar los problemas de diseño, posibles problemas de rendimiento y los antipatrones durante el desarrollo para Batch y su uso.

## <a name="pools"></a>Grupos

Los [grupos](nodes-and-pools.md#pools) de Batch son los recursos de proceso para ejecutar trabajos en el servicio de Batch. En las secciones siguientes se proporcionan recomendaciones para trabajar con grupos de Batch.

### <a name="pool-configuration-and-naming"></a>Configuración y nomenclatura de grupos

- **Modo de asignación de grupo** Al crear una cuenta de Batch, puede elegir entre dos modos de asignación de grupo: **Servicio de Batch** o **Suscripción de usuario**. En la mayoría de los casos, debe usar el modo predeterminado servicio Batch, en el que los grupos se asignan en segundo plano en las suscripciones administradas por Batch. En el modo de suscripción de usuario alternativo, tanto las máquinas virtuales de Batch como otros recursos se crean directamente en su suscripción cuando se crea un grupo. Las cuentas de suscripción de usuario se utilizan principalmente para habilitar un subconjunto importante, pero pequeño, de escenarios. Puede obtener más información sobre el modo de suscripción de usuario en [Configuración adicional para el modo de suscripción de usuario](batch-account-create-portal.md#additional-configuration-for-user-subscription-mode).

- **Tenga en cuenta el tiempo de ejecución del trabajo y de la tarea al determinar la asignación del trabajo al grupo.**
    Si tiene trabajos compuestos principalmente de tareas de ejecución breve y los recuentos de tareas totales esperados son pequeños (por lo que el tiempo de ejecución global previsto del trabajo no es largo), no asigne un nuevo grupo a cada trabajo. El tiempo de asignación de los nodos acortará el tiempo de ejecución del trabajo.

- **Los grupos deben tener más de un nodo de proceso.**
    No se garantiza que los nodos individuales estén siempre disponibles. Aunque no es habitual, los errores de hardware, las actualizaciones del sistema operativo y un sinfín de otros problemas pueden hacer que los nodos individuales estén sin conexión. Si la carga de trabajo de Batch requiere un progreso determinista y garantizado, debe asignar grupos con varios nodos.

- **No reutilice los nombres de recursos.**
    Los recursos de Batch (trabajos, grupos, etc.) suelen ser inestables a lo largo del tiempo. Por ejemplo, puede crear un grupo el lunes, eliminarlo el martes y, después, crear otro grupo el jueves. Cada recurso nuevo que cree debe recibir un nombre único que no haya usado antes. Esto puede hacerse mediante el uso de un GUID (como el nombre de recurso completo o como parte del mismo) o insertando la hora en que se creó el recurso en el nombre del mismo. Batch admite la propiedad [DisplayName](/dotnet/api/microsoft.azure.batch.jobspecification.displayname), que se puede usar para dar a un recurso un nombre legible, incluso si el identificador de recurso real es algo que no es descriptivo. El uso de nombres únicos facilita la diferenciación de un recurso determinado en los registros y las métricas. También elimina la ambigüedad si alguna vez tiene que archivar un caso de soporte técnico para un recurso.

- **Continuidad durante el mantenimiento y errores del grupo.**
    Es mejor que los trabajos usen los grupos dinámicamente. Si los trabajos usan el mismo grupo para todo, existe la posibilidad de que los trabajos no se ejecuten si algo sale mal con el grupo. Esto es especialmente importante para las cargas de trabajo que dependen del tiempo. Para solucionar este error, seleccione o cree un grupo de forma dinámica cuando programe cada trabajo, o tenga una manera de invalidar el nombre del grupo para que pueda omitir un grupo incorrecto.

- **Continuidad empresarial durante el mantenimiento y errores del grupo** Hay muchas causas posibles por las que puede que un grupo no alcance el tamaño necesario que quiere, como errores internos, restricciones de capacidad, etc. Por esta razón, debe estar preparado para redestinar los trabajos a un grupo diferente (posiblemente con un tamaño de máquina virtual diferente: Batch admite esto a través de [UpdateJob](/dotnet/api/microsoft.azure.batch.protocol.joboperationsextensions.update)) si es necesario. Evite el uso de un identificador de grupo estático con la esperanza de que nunca se eliminará ni cambiará.

### <a name="pool-lifetime-and-billing"></a>Vigencia del grupo y facturación

La vigencia del grupo puede variar en función del método de asignación y de las opciones que se apliquen a la configuración del grupo. Los grupos pueden tener una vigencia arbitraria y un número de nodos de proceso variable en el grupo en cualquier momento. Es su responsabilidad administrar los nodos de proceso del grupo, ya sea explícitamente o a través de las características proporcionadas por el servicio (escalado automático o autogrupo).

- **Mantenga los grupos actualizados.**
    Debe cambiar el tamaño de los grupos a cero cada pocos meses para asegurarse de que obtiene las actualizaciones y las correcciones de errores del agente de nodo más recientes. El grupo no recibirá actualizaciones de agente de nodo a menos que se vuelva a crear o se cambie el tamaño a 0 nodos de proceso. Antes de volver a crear o cambiar el tamaño del grupo, se recomienda descargar los registros del agente de nodo con fines de depuración, como se describe en la sección [Nodos](#nodes).

- **Volver a crear el grupo** De forma similar, no se recomienda eliminar y volver a crear los grupos diariamente. En su lugar, cree un nuevo grupo y actualice los trabajos existentes para que apunten al nuevo grupo. Cuando se hayan pasado todas las tareas al nuevo grupo, elimine el grupo anterior.

- **Eficacia del grupo y facturación** Batch en sí no incurre en cargos adicionales, pero se incurre en cargos por los recursos de proceso utilizados. Se le facturará por cada nodo de proceso del grupo, independientemente del estado en el que se encuentre. Esto incluye los cargos necesarios para que se ejecute el nodo, como los costos de almacenamiento y de red. Para obtener más información sobre los procedimientos recomendados, consulte [Análisis de costos y presupuestos para Azure Batch](budget.md).

### <a name="pool-allocation-failures"></a>Errores de asignación de grupos

Los errores de asignación de grupos pueden producirse en cualquier momento durante la primera asignación o en los siguientes cambios de tamaño. Esto puede deberse a un agotamiento temporal de la capacidad en una región o a errores en otros servicios de Azure en los que se basa Batch. La cuota principal no es una garantía, sino un límite.

### <a name="unplanned-downtime"></a>Tiempo de inactividad no planeado

Los grupos de Batch pueden experimentar eventos de tiempo de inactividad en Azure. Tenga esto en cuenta al planear y desarrollar el escenario o el flujo de trabajo para Batch.

En caso de que se produzca un error en un nodo, Batch intentará recuperar automáticamente estos nodos de proceso en su nombre. De esta forma se puede desencadenar la reprogramación de cualquier tarea en ejecución en el nodo que se recupera. Consulte [Diseño de reintentos](#design-for-retries-and-re-execution) para obtener más información sobre las tareas interrumpidas.

### <a name="custom-image-pools"></a>Grupos de imágenes personalizados

Al crear un grupo en Azure Batch con Configuración de máquina virtual, se especifica una imagen de máquina virtual (VM) que proporciona el sistema operativo para cada nodo de proceso en el grupo. Puede crear el grupo con una imagen de Azure Marketplace compatible o [crear una imagen personalizada con una imagen de Shared Image Gallery](batch-sig-images.md). Aunque también puede usar una [imagen administrada](batch-custom-images.md) para crear un grupo de imágenes personalizadas, se recomienda crear imágenes personalizadas mediante Shared Image Gallery siempre que sea posible. Shared Image Gallery le ayuda a aprovisionar grupos más rápido, escalar cantidades más grandes de VM y mejorar la confiabilidad al aprovisionar las VM.

### <a name="third-party-images"></a>Imágenes de terceros

Pueden crearse grupos con imágenes de terceros publicadas en Azure Marketplace. Con cuentas de Batch en modo de suscripción de usuario, puede mostrarse el error "Allocation failed due to marketplace purchase eligibility check" (Error en la asignación debido a la comprobación de la validez de la compra en Marketplace) al crear un grupo con determinadas imágenes de terceros. Para resolver este error, acepte los términos establecidos por el publicador de la imagen. Puede hacerlo con [Azure PowerShell](https://docs.microsoft.com/powershell/module/azurerm.marketplaceordering/set-azurermmarketplaceterms) o la [CLI de Azure](https://docs.microsoft.com/cli/azure/vm/image/terms).

### <a name="azure-region-dependency"></a>Dependencia de la región de Azure

Es aconsejable no depender de una sola región de Azure si tiene una carga de trabajo que depende del tiempo o de la producción. Aunque es poco frecuente, hay problemas que pueden afectar a toda una región. Por ejemplo, si su procesamiento tiene que iniciarse en un momento determinado, considere la posibilidad de escalar verticalmente el grupo de la región principal *justo antes de la hora de inicio*. Si se produce un error en la escala del grupo, puede revertir la escala de un grupo verticalmente en una región (o regiones) de copia de seguridad. Los grupos en varias cuentas de distintas regiones proporcionan una copia de seguridad preparada y de fácil acceso si se produce algún problema con otro grupo. Para más información, consulte [Diseño de la aplicación para una alta disponibilidad](high-availability-disaster-recovery.md).

## <a name="jobs"></a>Trabajos

Un [trabajo](jobs-and-tasks.md#jobs) es un contenedor diseñado para contener cientos, miles o incluso millones de tareas. Siga estas instrucciones al crear trabajos.

### <a name="fewer-jobs-more-tasks"></a>Menos trabajos, más tareas

El uso de un trabajo para ejecutar una única tarea es ineficaz. Por ejemplo, es más eficaz usar un único trabajo que contenga 1000 tareas en lugar de crear 100 trabajos que contengan 10 tareas cada uno. La ejecución de 1000 trabajos, cada uno con una sola tarea, sería el enfoque menos eficaz, más lento y más costoso.

Por este motivo, asegúrese de no diseñar una solución de Batch que requiera miles de trabajos activos simultáneamente. No hay ninguna cuota de tareas, por lo que la ejecución de tantas tareas en el menor número de trabajos como sea posible utiliza de forma eficaz los [trabajos y las cuotas de programación de trabajos](batch-quota-limit.md#resource-quotas).

### <a name="job-lifetime"></a>Vigencia del trabajo

Un trabajo de Batch tiene una vigencia indefinida hasta que se elimine del sistema. Su estado designa si este puede aceptar más tareas para la programación.

Un trabajo no se mueve automáticamente al estado completado a menos que se termine explícitamente. Esta acción se puede desencadenar automáticamente mediante la propiedad [onAllTasksComplete](/dotnet/api/microsoft.azure.batch.common.onalltaskscomplete) o [maxWallClockTime](/rest/api/batchservice/job/add#jobconstraints).

Hay un [trabajo activo y cuota de programación de trabajo](batch-quota-limit.md#resource-quotas) predeterminados. Los trabajos y las programaciones de trabajos en estado completado no cuentan para esta cuota.

## <a name="tasks"></a>Tareas

Las [tareas](jobs-and-tasks.md#tasks) son unidades de trabajo individuales que componen un trabajo. El usuario envía las tareas y Batch las programa en nodos de proceso. Hay varias consideraciones de diseño que se deben tener en cuenta al crear y ejecutar tareas. En las siguientes secciones se explican los escenarios comunes y cómo diseñar las tareas para solventar los problemas y ejecutarlas de forma eficaz.

### <a name="save-task-data"></a>Almacenamiento de datos de tareas

Los nodos de proceso son efímeros por naturaleza. Hay muchas características en Batch, como el autogrupo y el escalado automático, que facilitan la desaparición de los nodos. Cuando los nodos dejan el grupo (debido a un cambio de tamaño o una eliminación del grupo), también se eliminan todos los archivos de dichos nodos. Por este motivo, una tarea debe mover su salida del nodo en el que se está ejecutando y en un almacén duradero antes de que se complete. Del mismo modo, si se produce un error en una tarea, debe trasladar los registros necesarios para diagnosticar el error en un almacén duradero.

Batch tiene compatibilidad integrada con Azure Storage para cargar datos a través de [OutputFiles](batch-task-output-files.md), así como una variedad de sistemas de archivos compartidos, o bien puede realizar la carga usted mismo en sus tareas.

### <a name="manage-task-lifetime"></a>Administración de la duración de la tarea

Elimine las tareas cuando ya no se necesiten o establezca una restricción de tarea [retentionTime](/dotnet/api/microsoft.azure.batch.taskconstraints.retentiontime). Si se establece un `retentionTime`, Batch limpia automáticamente el espacio en disco que usa la tarea cuando `retentionTime` expire.

La eliminación de tareas consigue dos cosas. Garantiza que no tiene una acumulación de tareas en el trabajo, lo que dificultaría la consulta y la búsqueda de la tarea que le interesa (porque tendría que filtrar por las tareas completadas). También limpia los datos de la tarea correspondiente en el nodo (siempre que todavía no se haya llegado al `retentionTime`). Esto ayuda a garantizar que los nodos no se llenen con los datos de la tarea y se quede sin espacio en disco.

### <a name="submit-large-numbers-of-tasks-in-collection"></a>Envío de un gran número de tareas en la recopilación

Las tareas se pueden enviar de forma individual o en colecciones. Envíe tareas en [colecciones](/rest/api/batchservice/task/addcollection) hasta un máximo de 100 a la vez al realizar el envío masivo de tareas para reducir la sobrecarga y el tiempo de envío.

### <a name="set-max-tasks-per-node-appropriately"></a>Establecimiento del número máximo de tareas por nodo correctamente

Batch admite tareas de sobresuscripción en nodos (que ejecutan más tareas que núcleos tiene un nodo). Depende de usted asegurarse de que las tareas se ajustan a los nodos del grupo. Por ejemplo, su experiencia podría verse degradada si intenta programar ocho tareas, cada una de las cuales consume un 25 % de uso de CPU en un nodo (en un grupo con `maxTasksPerNode = 8`).

### <a name="design-for-retries-and-re-execution"></a>Diseño de reintentos y reejecución

Batch puede reintentar automáticamente las tareas. Hay dos tipos de reintentos: controlados por el usuario e internos. Los reintentos controlados por el usuario los especifica el elemento [maxTaskRetryCount](/dotnet/api/microsoft.azure.batch.taskconstraints.maxtaskretrycount) de la tarea. Cuando un programa especificado en la tarea sale con un código de salida distinto de cero, la tarea se reintenta hasta el valor del `maxTaskRetryCount`.

Aunque es poco frecuente, se puede reintentar una tarea internamente debido a errores en el nodo de proceso, como no poder actualizar el estado interno o un error en el nodo mientras la tarea se está ejecutando. La tarea se reintentará en el mismo nodo de proceso, si es posible, hasta un límite interno antes de que se desista y se aplace la tarea que Batch va a reprogramar, potencialmente en un nodo de proceso diferente.

No hay ninguna diferencia de diseño al ejecutar las tareas en nodos dedicados o en nodos de prioridad baja. Tanto si una tarea se reemplaza mientras se ejecuta en un nodo de prioridad baja como si se interrumpe debido a un error en un nodo dedicado, ambas situaciones se mitigarían diseñando la tarea para resistir el error.

### <a name="build-durable-tasks"></a>Creación de tareas durables

Las tareas deben diseñarse para resistir errores y dar cabida al reintento. Esto es especialmente importante para las tareas de larga duración. Para ello, asegúrese de que las tareas generan el mismo resultado único, incluso si se ejecutan más de una vez. Una manera de lograrlo es hacer que las tareas sean de búsqueda de objetivos. Otra manera es asegurarse de que las tareas sean idempotentes (es decir, tendrán el mismo resultado independientemente del número de veces que se ejecuten).

Un ejemplo común es una tarea de copia archivos en un nodo de proceso. Un enfoque sencillo es una tarea que copia todos los archivos especificados cada vez que se ejecuta, lo que resulta ineficaz y no se ha creado para resistir errores. En su lugar, cree una tarea para asegurarse de que los archivos se encuentran en el nodo de proceso; una tarea que no vuelva a copiar los archivos que ya están presentes. De esta manera, la tarea se retomará en el punto en que se quedó si se interrumpió.

### <a name="avoid-short-execution-time"></a>Evitación de un breve tiempo de ejecución

Las tareas que solo se ejecutan durante uno o dos segundos no son ideales. Debe intentar realizar una cantidad significativa de trabajo en una tarea individual (un mínimo de 10 segundos y hasta horas o días). Si cada tarea se está ejecutando durante un minuto (o más), la sobrecarga de programación como una fracción del tiempo de proceso general es pequeña.

### <a name="use-pool-scope-for-short-tasks-on-windows-nodes"></a>Uso del ámbito de grupo para tareas cortas en nodos de Windows

Al programar una tarea en nodos de Batch, puede elegir si desea ejecutarlo con el ámbito de la tarea o con el ámbito del grupo. Si la tarea solo se va a ejecutar durante un breve período de tiempo, su ámbito puede ser ineficaz debido a los recursos necesarios para crear la cuenta de usuario automático para dicha tarea. Si desea mayor eficacia, considere la posibilidad de establecer estas tareas en el ámbito del grupo. Para más información, consulte [Ejecución de una tarea como usuario automático con ámbito de grupo](batch-user-accounts.md#run-a-task-as-an-auto-user-with-pool-scope).

## <a name="nodes"></a>Nodos

Un [nodo de proceso](nodes-and-pools.md#nodes) es una máquina virtual de Azure o una máquina virtual de servicio en la nube dedicada al proceso de una parte de la carga de trabajo de la aplicación. Siga estas instrucciones al trabajar con nodos.

### <a name="idempotent-start-tasks"></a>Tareas de inicio idempotentes

De forma similar a otras tareas, la [tarea de inicio](jobs-and-tasks.md#start-task) del nodo debe ser idempotente, ya que se volverá a ejecutar cada vez que se inicie el nodo. Una tarea idempotente es simplemente una que genera un resultado coherente cuando se ejecuta varias veces.

### <a name="manage-long-running-services-via-the-operating-system-services-interface"></a>Administración de servicios de larga duración a través de la interfaz de servicios del sistema operativo

A veces, es necesario ejecutar otro agente junto con el agente de Batch en el nodo. Por ejemplo, puede que desee recopilar datos del nodo y notificarlos. Se recomienda que estos agentes se implementen como servicios del sistema operativo, como un servicio de Windows o un servicio `systemd` de Linux.

Al ejecutar estos servicios, no deben realizar bloqueos de archivos en los directorios administrados por Batch del nodo, ya que, de lo contrario, Batch no podrá eliminar dichos directorios debido a los bloqueos de archivos. Por ejemplo, si instala un servicio de Windows en una tarea de inicio, en lugar de iniciar el servicio directamente desde el directorio de trabajo de la tarea de inicio, copie los archivos en otro lugar (o si los archivos existen simplemente omita la copia). Después, instale el servicio desde dicha ubicación. Cuando Batch vuelva a ejecutar la tarea de inicio, eliminará el directorio de trabajo de la tarea de inicio y la volverá a crear. Esto funciona porque el servicio tiene bloqueos de archivos en el otro directorio, no en el directorio de trabajo de la tarea de inicio.

### <a name="avoid-creating-directory-junctions-in-windows"></a>Evitación de la creación de uniones de directorio en Windows

Las uniones de directorios, que a veces se denominan vínculos físicos, son difíciles de tratar durante la limpieza de tareas y trabajos. Use vínculos simbólicos (soft-links) en lugar de vínculos físicos.

### <a name="collect-the-batch-agent-logs"></a>Recopilación de registros de agente de Batch

Si observa un problema relacionado con el comportamiento de un nodo o tareas en ejecución en un nodo, recopile los registros del agente de Batch antes de desasignar los nodos en cuestión. Puede recopilar los registros del agente de Batch mediante la API de registros del servicio de Batch de carga. Estos registros se pueden proporcionar como parte de una incidencia de soporte técnico a Microsoft y ayudarán a solucionar los problemas.

### <a name="manage-os-upgrades"></a>Administración de las actualizaciones del sistema operativo

En el caso de las cuentas de Batch del modo de suscripción de usuario, las actualizaciones automáticas del sistema operativo pueden interrumpir el progreso de las tareas, especialmente si estas son de larga duración. La [creación de tareas idempotentes](#build-durable-tasks) puede ayudar a reducir los errores causados por estas interrupciones. También se recomienda [programar actualizaciones de imágenes de sistema operativo para las horas en las que no se espera que las tareas se ejecuten](../virtual-machine-scale-sets/virtual-machine-scale-sets-automatic-upgrade.md#manually-trigger-os-image-upgrades).

## <a name="isolation-security"></a>Seguridad de aislamiento

Con respecto al aislamiento, si su escenario requiere aislar los trabajos entre sí, debe hacerlo colocándolos en grupos independientes. Un grupo es el límite de aislamiento de seguridad en Batch y, de forma predeterminada, dos grupos no son visibles ni pueden comunicarse entre sí. Evite el uso de cuentas de Batch independientes como medio de aislamiento.

## <a name="moving-batch-accounts-across-regions"></a>Movimiento de cuentas de Batch entre regiones

Hay escenarios en los que puede resultar útil trasladar una cuenta de Batch existente de una región a otra. También puede realizar el traslado a otra región como parte del planeamiento de la recuperación ante desastres.

Las cuentas de Azure Batch no se pueden trasladar directamente de una región a otra. Sin embargo, puede usar una plantilla de Azure Resource Manager para exportar la configuración actual de la cuenta de Batch. Después, para preparar el recurso en otra región, puede exportar la cuenta de Batch a una plantilla, modificar los parámetros para que coincidan con la región de destino y, a continuación, implementar la plantilla en la nueva región.

Después de cargar la plantilla en la nueva región, tendrá que volver a crear los certificados, las programaciones de trabajos y los paquetes de aplicación. Para confirmar los cambios y completar el traslado de la cuenta de Batch, recuerde eliminar la cuenta de Batch original o el grupo de recursos.

Para más información sobre Resource Manager y las plantillas, consulte [Inicio rápido: Creación e implementación de plantillas de Azure Resource Manager mediante Azure Portal](../azure-resource-manager/templates/quickstart-create-templates-use-the-portal.md).

## <a name="connectivity"></a>Conectividad

Revise las siguientes instrucciones a la hora de considerar la conectividad en las soluciones de Batch.

### <a name="network-security-groups-nsgs-and-user-defined-routes-udrs"></a>Grupos de seguridad de red (NSG) y Rutas definidas por el usuario (UDR)

Al aprovisionar [grupos de Batch en una red virtual](batch-virtual-network.md), asegúrese de seguir las instrucciones sobre el uso de la etiqueta de servicio `BatchNodeManagement`, los puertos, los protocolos y la dirección de la regla.
Se recomienda encarecidamente el uso de la etiqueta de servicio, en lugar de usar las direcciones IP subyacentes del servicio Batch. Esto se debe a que las direcciones IP pueden cambiar con el tiempo. El uso directo de las direcciones IP del servicio Batch puede provocar inestabilidad o interrupciones para los grupos de Batch.

En el caso de las rutas definidas por el usuario (UDR), asegúrese de que tiene un proceso implementado para actualizar las direcciones IP del servicio Batch periódicamente en la tabla de rutas, ya que estas direcciones cambian con el tiempo. Para más información sobre cómo obtener la lista de direcciones IP del servicio Batch, consulte [Etiquetas de servicio de red virtual](../virtual-network/service-tags-overview.md). Las direcciones IP del servicio Batch se asociarán a la etiqueta de servicio `BatchNodeManagement`, o la variante regional que coincida con la región de la cuenta de Batch.

### <a name="honoring-dns"></a>Respetar DNS

Asegúrese de que los sistemas respetan el período de vida (TTL) de DNS para la dirección URL del servicio de la cuenta de Batch. Además, asegúrese de que los clientes del servicio Batch y otros mecanismos de conectividad con el servicio Batch no se basan en direcciones IP (o bien [cree un grupo con direcciones IP públicas estáticas](create-pool-public-ip.md), como se describe a continuación).

Si las solicitudes reciben respuestas HTTP de nivel 5xx y hay un encabezado "Conexión: cerrar" en la respuesta, el cliente del servicio Batch debe observar la recomendación cerrando la conexión existente y volviendo a resolver DNS para la dirección URL del servicio de la cuenta de Batch, e intentar realizar las siguientes solicitudes en una nueva conexión.

### <a name="retry-requests-automatically"></a>Reintento automático de las solicitudes

Asegúrese de que los clientes del servicio Batch tienen las directivas de reintento adecuadas para volver a intentar automáticamente las solicitudes, incluso durante el funcionamiento normal y no exclusivamente durante períodos de tiempo de mantenimiento del servicio. Estas directivas de reintento deben abarcar un intervalo de al menos 5 minutos. Se proporcionan capacidades de reintento automático con varios SDK de Batch, como la [clase RetryPolicyProvider de .NET](/dotnet/api/microsoft.azure.batch.retrypolicyprovider).

### <a name="static-public-ip-addresses"></a>Direcciones IP públicas estáticas

Normalmente, se accede a las máquinas virtuales de un grupo de Batch mediante direcciones IP públicas que pueden cambiar a lo largo de la duración del grupo. Esto puede dificultar la interacción con una base de datos u otro servicio externo que limite el acceso a determinadas direcciones IP. Para asegurarse de que las direcciones IP públicas del grupo no cambien de forma inesperada, puede crear un grupo con un conjunto de direcciones IP públicas estáticas que controle. Para más información, consulte [Creación de un grupo de Azure Batch con direcciones IP públicas especificadas](create-pool-public-ip.md).

## <a name="batch-node-underlying-dependencies"></a>Dependencias subyacentes del nodo Batch

Tenga en cuenta las siguientes dependencias y restricciones al diseñar sus soluciones de Batch.

### <a name="system-created-resources"></a>Recursos creados por el sistema

Azure Batch crea y administra un conjunto de usuarios y grupos en la máquina virtual, que no se deben modificar. Los pasos son los siguientes:

#### <a name="windows"></a>Windows

- Un usuario llamado **PoolNonAdmin**
- Un grupo de usuarios denominado **WATaskCommon**

#### <a name="linux"></a>Linux

- Un usuario llamado **_azbatch**

### <a name="file-cleanup"></a>Limpieza de archivos

Batch intenta limpiar activamente el directorio de trabajo en el que se ejecutan las tareas, una vez que expira el tiempo de retención. Es [su responsabilidad limpiar](#manage-task-lifetime) cualquier archivo que se escriba fuera de este directorio para evitar llenar el espacio en disco. 

La limpieza automatizada del directorio de trabajo se bloqueará si ejecuta un servicio en Windows desde el directorio de trabajo startTask, debido a que la carpeta todavía está en uso. Esto dará lugar a un rendimiento degradado. Para solucionarlo, cambie el directorio de ese servicio a un directorio independiente que no esté administrado por Batch.
