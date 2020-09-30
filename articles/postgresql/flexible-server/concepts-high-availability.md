---
title: 'Información general de alta disponibilidad con redundancia de zona con Azure Database for PostgreSQL: Servidor flexible (versión preliminar)'
description: 'Obtenga información sobre los conceptos de alta disponibilidad con redundancia de zona con Azure Database for PostgreSQL: Servidor flexible'
author: sr-msft
ms.author: srranga
ms.service: postgresql
ms.topic: conceptual
ms.date: 09/22/2020
ms.openlocfilehash: 7db9ac0eb624c2732295639d65e0311fcf459f71
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90931858"
---
# <a name="high-availability-concepts-in-azure-database-for-postgresql---flexible-server"></a>Conceptos de alta disponibilidad en Azure Database for PostgreSQL: Servidor flexible

> [!IMPORTANT]
> Azure Database for PostgreSQL: Servidor flexible está en versión preliminar.

Azure Database for PostgreSQL: Servidor flexible ofrece una configuración de alta disponibilidad con capacidad de conmutación automática por error mediante una implementación de servidor **con redundancia de zona**. Cuando se implementa en una configuración con redundancia de zona, el servidor flexible aprovisiona y administra de forma automática una réplica en espera en otra zona de disponibilidad. Mediante la replicación de streaming de PostgreSQL, los datos se replican en el servidor de réplica en espera en modo **sincrónico**. 

La configuración con redundancia de zona habilita la capacidad de conmutación por error automática sin pérdida de datos durante eventos planeados, como operaciones de proceso de escalado iniciadas por el usuario, y también durante eventos no planeados, como errores de hardware y software subyacentes, errores de red y errores de zona de disponibilidad. 

:::image type="content" source="./media/business-continuity/concepts-zone-redundant-high-availability-architecture.png" alt-text="Alta disponibilidad con redundancia de zona"::: 

## <a name="zone-redundant-high-availability-architecture"></a>Arquitectura de alta disponibilidad con redundancia de zona

Puede elegir la región y la zona de disponibilidad para implementar el servidor de bases de datos principal. Se aprovisiona un servidor de réplica en espera en una zona de disponibilidad distinta con la misma configuración que el servidor principal, incluidos el nivel y el tamaño de proceso, el tamaño de almacenamiento y la configuración de red. Los registros de transacciones se replican en modo sincrónico a la réplica en espera mediante la replicación de streaming de PostgreSQL. Las copias de seguridad automáticas se realizan periódicamente desde el servidor de bases de datos principal, mientras que los registros de transacciones se archivan continuamente en el almacenamiento de copia de seguridad de la réplica en espera. 

El estado de la configuración de alta disponibilidad se supervisa y se registra continuamente en el portal. A continuación se enumeran los estados de alta disponibilidad con redundancia de zona:

| **Estado** | **Descripción** |
| ------- | ------ |
| <b> Inicializando | En proceso de crear un servidor en espera |
| <b> Replicando datos | Una vez que se ha creado el servidor en espera, se pone al día con el servidor principal. |
| <b> Correcto | La replicación está en estado estable y es correcta. |
| <b> Conmutando por error | El servidor de bases de datos está en proceso de conmutar por error al de espera. |
| <b> Quitando en espera | En proceso de eliminar el servidor en espera. | 
| <b> Sin habilitar | La alta disponibilidad con redundancia de zona no está habilitada.  |

## <a name="steady-state-operations"></a>Operaciones de estado estable

Las aplicaciones cliente de PostgreSQL se conectan al servidor principal mediante el nombre del servidor de bases de datos. Las lecturas de aplicación se sirven directamente desde el servidor principal, mientras que las confirmaciones y escrituras se confirman en la aplicación solo después de que los datos se conserven en el servidor principal y en la réplica en espera. Debido a este requisito de ida y vuelta adicional, las aplicaciones pueden esperar una latencia elevada para las operaciones de escritura y confirmación. Puede supervisar el estado de la alta disponibilidad en el portal.

:::image type="content" source="./media/business-continuity/concepts-high-availability-steady-state.png" alt-text="Alta disponibilidad con redundancia de zona: estado estable"::: 

1. Los clientes se conectan al servidor flexible y realizan operaciones de escritura.
2. Los cambios se replican en el sitio en espera.
3. El servidor principal recibe confirmación.
4. Las escrituras o confirmaciones se confirman.

## <a name="failover-process---planned-downtimes"></a>Proceso de conmutación por error: tiempos de inactividad planificados

Los eventos de tiempo de inactividad planificados incluyen actualizaciones de software frecuentes programadas de Azure y actualizaciones de versiones secundarias. Cuando se configuran en alta disponibilidad, estas operaciones se aplican en primer lugar a la réplica en espera mientras las aplicaciones continúan accediendo al servidor principal. Una vez que se actualiza la réplica en espera, se purgan las conexiones del servidor principal y se desencadena una conmutación por error que activa la réplica en espera para que sea la principal con el mismo nombre de servidor de bases de datos. Las aplicaciones cliente tendrán que volver a conectarse con el mismo nombre de servidor de bases de datos al servidor principal nuevo y podrán reanudar sus operaciones. Se establecerá un nuevo servidor en espera en la misma zona que el servidor principal anterior. 

Para otras operaciones iniciadas por el usuario, como el proceso de escalado o almacenamiento de escalado, los cambios se aplican en primer lugar en el modo en espera y después en el principal. Actualmente, las conexiones no se han conmutado por error al modo de espera y, por lo tanto, incurre en tiempo de inactividad mientras la operación se lleva a cabo en el servidor principal.

### <a name="reducing-planned-downtime-with-managed-maintenance-window"></a>Reducción del tiempo de inactividad planificado con la ventana de mantenimiento administrado

 Puede programar actividades de mantenimiento iniciadas por Azure eligiendo una ventana de 30 minutos en un día de su preferencia, donde se espera que las actividades de las bases de datos sean bajas. Las tareas de mantenimiento de Azure, como la revisión o las actualizaciones de versiones secundarias, se producirían durante esa ventana.  En el caso de los servidores flexibles configurados con alta disponibilidad, estas actividades de mantenimiento se realizan primero en la réplica en espera y luego se activa. Las aplicaciones se vuelven a conectar al servidor principal nuevo y reanudan sus operaciones mientras se aprovisiona un nuevo servidor en espera.

## <a name="failover-process---unplanned-downtimes"></a>Proceso de conmutación por error: tiempos de inactividad no planificados

Las interrupciones no planificadas incluyen errores de software o de componentes de infraestructura que afectan a la disponibilidad de la base de datos. En caso de que el sistema de supervisión detecte la no disponibilidad del servidor, la replicación en la réplica en espera se anula y la réplica en espera se activa para ser el servidor de bases de datos principal. Los clientes pueden volver a conectarse al servidor de bases de datos mediante la misma cadena de conexión y reanudar sus operaciones. Se espera que el tiempo de conmutación por error global sea de 60 a 120 segundos. Pero en función de la actividad del servidor de bases de datos principal en el momento de la conmutación por error, como las transacciones de gran tamaño y el tiempo de recuperación, la conmutación por error puede tardar más.

:::image type="content" source="./media/business-continuity/concepts-high-availability-failover-state.png" alt-text="Alta disponibilidad con redundancia de zona: conmutación por error"::: 

1. El servidor de bases de datos principal está inactivo y los clientes pierden la conectividad de la base de datos. 
2. El servidor en espera se activa para convertirse en el servidor principal nuevo. El cliente se conecta al servidor principal nuevo con la misma cadena de conexión. Tener la aplicación cliente en la misma zona que el servidor de bases de datos principal reduce la latencia y mejora el rendimiento.
3. El servidor en espera se establece en la misma zona que el servidor principal anterior y se inicia la replicación de streaming. 
4. Una vez establecida la replicación de estado estable, las confirmaciones y escrituras de la aplicación cliente se confirman después de que los datos se conserven en ambos sitios.

## <a name="point-in-time-restore"></a>Restauración a un momento dado 

Los servidores flexibles que están configurados con alta disponibilidad replican los datos en tiempo real en el servidor en espera para mantenerlos actualizados. Los errores de usuario en el servidor principal, como la eliminación accidental de una tabla o las actualizaciones de datos incorrectas, se replican fielmente en la réplica en espera. Por tanto, no puede usar el servidor en espera para recuperarse de estos errores lógicos. Tiene que realizar una restauración a un momento dado de copias de seguridad para recuperarse de estos errores.  Con la capacidad de restauración a un momento dado del servidor flexible, puede restaurar hasta el momento antes de que se produjera el error. En el caso de las bases de datos configuradas con alta disponibilidad, un nuevo servidor de bases de datos se restaurará como un servidor flexible de una sola zona con un nombre proporcionado por el usuario. Después, puede exportar el objeto desde el servidor de bases de datos e importarlo en el servidor de bases de datos de producción. Del mismo modo, si quiere clonar el servidor de bases de datos con fines de prueba y desarrollo, o bien realizar la restauración para cualquier otro fin, puede realizar restauraciones a un momento dado.

## <a name="zone-redundant-high-availability---features"></a>Alta disponibilidad con redundancia de zona: características

-   La réplica en espera se implementará en una configuración de máquina virtual exacta a la del servidor principal, incluidos los núcleos virtuales, el almacenamiento, la configuración de red (VNET, firewall), etc.

-   Posibilidad de agregar alta disponibilidad a un servidor de bases de datos existente.

-   Posibilidad de quitar la réplica en espera si se deshabilita la alta disponibilidad.

-   Posibilidad de elegir la zona de disponibilidad para el servidor de bases de datos principal.

-   Posibilidad de detener, iniciar y reiniciar los servidores de bases de datos principales y en espera.

-   Las copias de seguridad automáticas se realizan desde el servidor de bases de datos principal y se almacenan en un almacenamiento con redundancia de zona.

-   Los clientes siempre se conectan al servidor de bases de datos principal.

-   Capacidad de reiniciar el servidor para seleccionar cualquier cambio de parámetro de servidor estático.
  
-   Las actividades de mantenimiento periódico, como las actualizaciones de versiones secundarias, se producen en primer lugar en el servidor en espera y el servicio se conmuta por error para reducir el tiempo de inactividad.  

## <a name="zone-redundant-high-availability---limitations"></a>Alta disponibilidad con redundancia de zona: limitaciones

-   La alta disponibilidad no se admite en el nivel de proceso expandible.
-   La alta disponibilidad solo se admite en regiones donde hay disponibles varias zonas.
-   Debido a la replicación sincrónica en otra zona de disponibilidad, las aplicaciones pueden experimentar una latencia elevada de escritura y confirmación.

-   No se puede usar la réplica en espera para las consultas de solo lectura.

-   En función de la actividad del servidor principal en el momento de la conmutación por error, esta puede tardar hasta dos minutos o más en completarse.

-   Al reiniciar el servidor de bases de datos principal para elegir los cambios de los parámetros estáticos también se reinicia la réplica en espera.

-   No se admite la configuración de réplicas de lectura adicionales.

-   La configuración de las tareas de administración iniciadas por el cliente no se puede programar durante la ventana de mantenimiento administrado.

-   Los eventos planificados, como la escala de proceso y el almacenamiento de escalado, se producen primero en el servidor en espera y luego en el servidor principal. El servicio no se conmuta por error. 

## <a name="next-steps"></a>Pasos siguientes

-   Más información sobre [continuidad empresarial](./concepts-business-continuity.md)
-   Más información sobre [administración de la alta disponibilidad](./how-to-manage-high-availability-portal.md)
-   Más información sobre [copia de seguridad y recuperación](./concepts-backup-restore.md)