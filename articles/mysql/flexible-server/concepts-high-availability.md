---
title: Información general de alta disponibilidad con redundancia de zona con el servidor flexible de Azure Database for MySQL
description: Información sobre los conceptos de alta disponibilidad con redundancia de zona con el servidor flexible de Azure Database for MySQL
author: kummanish
ms.author: manishku
ms.service: mysql
ms.topic: conceptual
ms.date: 09/21/2020
ms.openlocfilehash: 9db5776a4d2395baf03a5ed7cf05db49de8d0321
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90931872"
---
# <a name="high-availability-concepts-in-azure-database-for-mysql-flexible-server-preview"></a>Conceptos de alta disponibilidad en el servidor flexible de Azure Database for MySQL (versión preliminar)

> [!IMPORTANT] 
> Actualmente, la opción Servidor flexible de Azure Database for MySQL se encuentra en versión preliminar pública.

Servidor flexible de Azure Database for MySQL (versión preliminar) permite configurar la alta disponibilidad con conmutación automática por error mediante la opción de alta disponibilidad con **redundancia de zona**. Cuando se implementa en una configuración con redundancia de zona, el servidor flexible aprovisiona y administra de forma automática una réplica en espera en otra zona de disponibilidad. Con la replicación de nivel de almacenamiento, los datos se **replican sincrónicamente** en el servidor en espera en la zona secundaria para permitir que no haya pérdida de datos después de una conmutación por error. La conmutación por error es totalmente transparente desde la aplicación cliente y no requiere ninguna acción del usuario. El servidor en espera no está disponible para las operaciones de lectura o escritura, pero es un estado de espera pasivo para habilitar la conmutación por error rápida. Los tiempos de conmutación por error suelen oscilar entre 60-120 segundos.

La configuración de alta disponibilidad con redundancia de zona permite la conmutación por error automática durante eventos planeados, como operaciones de proceso de escalado iniciadas por el usuario, y eventos no planeados, como errores de hardware y software subyacentes, errores de red e incluso errores de zona de disponibilidad.

:::image type="content" source="./media/concepts-high-availability/1-flexible-server-overview-zone-redundant-ha.png" alt-text="Vista de alta disponibilidad con redundancia de zona":::

## <a name="zone-redundancy-architecture"></a>Arquitectura de la redundancia de zona

El servidor principal se implementa en la región y en una zona de disponibilidad concreta. Cuando se elige la alta disponibilidad, se implementa de forma automática un servidor de réplica en espera con la misma configuración que el servidor principal, incluidos el nivel y el tamaño de proceso, el tamaño de almacenamiento y la configuración de red. Los datos de registro se replican sincrónicamente en la réplica en espera para garantizar que no se produzca ninguna pérdida de datos en cualquier situación de error. Las copias de seguridad automáticas, tanto las instantáneas como las de registros, se realizan desde el servidor de bases de datos principal. 

El estado de la alta disponibilidad se supervisa y se registra de manera continuada en la página de información general.

A continuación se enumeran los distintos estados de replicación:

| **Estado** | **Descripción** |
| :----- | :------ |
| <b>NotEnabled | La alta disponibilidad con redundancia de zona no está habilitada |
| <b>CreatingStandby | En el proceso de crear un servidor en espera |
| <b>ReplicatingData | Una vez que se ha creado el servidor en espera, se pone al día con el servidor principal. |
| <b>FailingOver | El servidor de bases de datos está en proceso de conmutar por error desde el principal al de espera. |
| <b>Correcto | La alta disponibilidad con redundancia de zona está en estado estable y es correcta. |
| <b>RemovingStandby | En función de la acción del usuario, la réplica en espera se encuentra en proceso de eliminación.| 

## <a name="advantages"></a>Ventajas

Estas son algunas ventajas de usar la característica de alta disponibilidad con redundancia de zona: 

-   La réplica en espera se implementará en una configuración de máquina virtual exacta a la del servidor principal, como núcleos virtuales, almacenamiento, configuración de red (VNET, firewall), etc.
-   Se puede quitar la réplica en espera si se deshabilita la alta disponibilidad.
-   Las copias de seguridad automáticas se basan en instantáneas, se realizan desde el servidor de bases de datos principal y se almacenan en un almacenamiento con redundancia de zona.
-   Si hay un evento de conmutación por error, se aprovisiona una nueva réplica en espera en la zona de disponibilidad principal original.
-   Los clientes siempre se conectan al servidor de bases de datos principal.
-   Si hay un bloqueo de base de datos o un error de nodo, primero se intentará el reinicio en el mismo nodo. Si se produce un error, se desencadena la conmutación automática por error.
-   Capacidad de reiniciar el servidor para seleccionar cualquier cambio de parámetro de servidor estático.

## <a name="steady-state-operations"></a>Operaciones de estado estable

Las aplicaciones se conectan al servidor principal mediante el nombre del servidor de bases de datos. La información de la réplica en espera no se expone para el acceso directo. Las operaciones de confirmación y escritura se reconocen a la aplicación solo después de que los archivos de registro se conserven en el disco del servidor principal y en la réplica en espera de forma sincrónica. Debido a este requisito de ida y vuelta adicional, las aplicaciones pueden esperar una latencia elevada para las operaciones de escritura y confirmación. Puede supervisar el estado de la alta disponibilidad en el portal.

## <a name="failover-process"></a>Proceso de conmutación por error 
Para la continuidad empresarial, debe tener un proceso de conmutación por error para eventos planeados y no planeados. 

### <a name="planned-events"></a>Eventos planeados

Los eventos de tiempo de inactividad planeados incluyen actividades programadas por Azure, como actualizaciones de software periódicas, actualizaciones de versiones secundarias o iniciadas por clientes como las operaciones de escalado de proceso y almacenamiento. Todos estos cambios se aplican primero a la réplica en espera. Durante ese tiempo, las aplicaciones mantienen el acceso al servidor principal. Una vez que se actualiza la réplica en espera, se purgan las conexiones del servidor principal, se desencadena una conmutación por error que activa la réplica en espera para que sea la principal con el mismo nombre de servidor de bases de datos mediante la actualización del registro de DNS. Las conexiones de cliente se desconectan y tendrán que volver a conectarse y reanudar sus operaciones. Se establecerá un nuevo servidor en espera en la misma zona que el principal anterior. Se espera que el tiempo de conmutación por error global sea de 60-120 segundos. 

>[!NOTE]
> En el caso de la operación de escalado de proceso, se escala el servidor de réplica secundario seguido del servidor principal. No hay ninguna conmutación por error implicada.

### <a name="failover-process---unplanned-events"></a>Proceso de conmutación por error: eventos no planeados
Los tiempos de inactividad del servicio no planeados incluyen errores de software o de infraestructura, como los de procesos, redes y almacenamiento, o bien interrupciones de energía que afectan a la disponibilidad de la base de datos. En caso de que la base de datos no esté disponible, la replicación en la réplica en espera se anula y la réplica en espera se activa para ser la base de datos principal. El DNS se actualiza y, después, los clientes se vuelven a conectar al servidor de bases de datos y reanudan sus operaciones. Se espera que el tiempo de conmutación por error global sea de 60-120 segundos. Pero en función de la actividad del servidor de bases de datos principal en el momento de la conmutación por error (como las transacciones de gran tamaño y el tiempo de recuperación) la conmutación por error puede tardar más.

## <a name="schedule-maintenance-window"></a>Programación de la ventana de mantenimiento 

Los servidores flexibles ofrecen una capacidad de programación de mantenimiento en la que puede elegir un período de 30 minutos en un día concreto durante el cual el mantenimiento de Azure funciona como las revisiones o las actualizaciones de versiones secundarias. En el caso de los servidores flexibles configurados con alta disponibilidad, estas actividades de mantenimiento se realizan primero en la réplica en espera. 

## <a name="point-in-time-restore"></a>Restauración a un momento dado 
Mientras que el servidor flexible configurado en alta disponibilidad replique los datos de forma sincrónica, el servidor en espera estará actualizado con el principal. Los errores de usuario en el servidor principal, como la eliminación accidental de una tabla o las actualizaciones incorrectas, se replican fielmente en el servidor en espera. Por tanto, no puede usar el servidor en espera para recuperarse de estos errores lógicos. Para recuperarse de estos errores lógicos, tendrá que realizar una restauración a un momento dado justo antes de que se haya producido el error. Con la funcionalidad de restauración a un momento dado del servidor flexible, al restaurar la base de datos configurada con alta disponibilidad, se restaurará un nuevo servidor de bases de datos como un nuevo servidor flexible con un nombre proporcionado por el usuario. Después, puede exportar el objeto desde el servidor de bases de datos e importarlo en el servidor de bases de datos de producción. Del mismo modo, si quiere clonar el servidor de bases de datos con fines de prueba y desarrollo, o bien realizar la restauración para cualquier otro fin, puede elegir el punto de restauración más reciente o uno personalizado. La operación de restauración creará un servidor flexible de una sola zona.

## <a name="mitigate-downtime"></a>Mitigación del tiempo de inactividad 
Cuando no use la característica de alta disponibilidad con redundancia de zona, todavía podrá mitigar el tiempo de inactividad de la aplicación. La planeación de tiempos de inactividad del servicio, como revisiones programadas, actualizaciones de versiones secundarias o las operaciones iniciadas por el usuario, se puede realizar como parte de ventanas de mantenimiento programadas. Los tiempos de inactividad del servicio planeados, como las revisiones programadas, las actualizaciones de versiones secundarias o las operaciones iniciadas por el usuario como las de escalado de proceso, incurren en tiempo de inactividad. Para mitigar el impacto de la aplicación en las tareas de mantenimiento iniciadas por Azure, puede optar por programarlas durante el día de la semana y la hora que menos afecten a la aplicación. 

Durante los eventos de tiempo de inactividad no planeados, como el bloqueo de la base de datos o el error del servidor, el servidor afectado se reiniciará dentro de la misma zona. Aunque es poco frecuente, si toda la zona resulta afectada, puede restaurar la base de datos en otra zona dentro de la región. 

## <a name="things-to-know-with-zone-redundancy"></a>Aspectos que se deben conocer sobre la redundancia de zona 

Estas son algunas consideraciones que debe tener en cuenta al usar alta disponibilidad con redundancia de zona: 

-   La alta disponibilidad **solo** se puede establecer durante el tiempo de creación del servidor flexible.
-   La alta disponibilidad no se admite en el nivel de proceso flexible.
-   Debido a la replicación sincrónica en otra zona de disponibilidad, el servidor de bases de datos principal puede experimentar una latencia elevada de escritura y confirmación.
-   No se puede usar la réplica en espera para las consultas de solo lectura.
-   En función de la actividad del servidor principal en el momento de la conmutación por error, la conmutación por error puede tardar hasta 60-120 segundos o más en completarse.
-   Al reiniciar el servidor de bases de datos principal para aplicar los cambios de los parámetros estáticos también se reinicia la réplica en espera.
-   No se admite la configuración de réplicas de lectura para servidores de alta disponibilidad con redundancia de zona.
-   La configuración de las tareas de administración iniciadas por el cliente no se puede automatizar durante la ventana de mantenimiento administrado.
-   Los eventos planeados, como la escala de proceso y las actualizaciones de versiones secundarias, se producen tanto en el servidor principal como en el servidor en espera al mismo tiempo. 


## <a name="next-steps"></a>Pasos siguientes

-   Más información sobre la [continuidad empresarial](./concepts-business-continuity.md)
-   Más información sobre la  [alta disponibilidad con redundancia de zona](./concepts-high-availability.md)
-   Más información sobre [copia de seguridad y recuperación](./concepts-backup-restore.md)
