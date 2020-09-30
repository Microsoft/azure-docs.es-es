---
title: 'Copia de seguridad y restauración en Azure Database for PostgreSQL: servidor flexible'
description: 'Obtenga información sobre los conceptos de la copia de seguridad y restauración con Azure Database for PostgreSQL: servidor flexible.'
author: sr-msft
ms.author: srranga
ms.service: postgresql
ms.topic: conceptual
ms.date: 09/22/2020
ms.openlocfilehash: bed196d1be101ffa75affc389d390ec0fa764b05
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90931859"
---
# <a name="backup-and-restore-in-azure-database-for-postgresql---flexible-server"></a>Copia de seguridad y restauración en Azure Database for PostgreSQL: servidor flexible

> [!IMPORTANT]
> Azure Database for PostgreSQL: servidor flexible está en versión preliminar.

Las copias de seguridad forman una parte esencial de cualquier estrategia de continuidad empresarial. Ayudan a proteger los datos en caso de daños o eliminaciones accidentales. Azure Database for PostgreSQL: servidor flexible realiza automáticamente una copia de seguridad del servidor y conserva las copias de seguridad durante un período de hasta 35 días. Durante la restauración, puede especificar la fecha y la hora en las que quiera realizar la restauración en el período de retención. El tiempo total de restauración y recuperación depende del tamaño de los archivos de la base de datos y de las dimensiones de la recuperación que se va a realizar. 

### <a name="backup-process-in-flexible-server"></a>Proceso de copia de seguridad en un servidor flexible
La primera copia de seguridad de instantáneas completa se programa inmediatamente después de la creación del servidor flexible. Posteriormente, se realiza una copia de seguridad diaria de las instantáneas de los archivos de datos. Las copias de seguridad se almacenan en un almacenamiento con redundancia de zona dentro de una región. Los registros de transacciones (registros de escritura previa) también se archivan continuamente, por lo que podrá restaurar a la última transacción confirmada. Estas copias de seguridad de registros y datos permiten restaurar un servidor a un momento dado dentro del período de retención de copias de seguridad configurado. Todas las copias de seguridad se cifran mediante cifrado AES de 256 bits.

Si la base de datos se configura con alta disponibilidad, las instantáneas diarias se realizarán desde el servidor principal, y las copias de seguridad continuas del registro, desde el modo de espera.

> [!IMPORTANT]
>En los servidores detenidos no se realizan copias de seguridad. Sin embargo, las copias de seguridad se reanudan cuando la base de datos se inicia automáticamente transcurridos siete días o cuando lo hace el usuario.

Las copias de seguridad solo se pueden usar para operaciones de restauración en el servidor flexible. Si quiere exportar datos del servidor flexible o importar datos en él, use la metodología de [volcado y restauración](https://docs.microsoft.com/azure/postgresql/howto-migrate-using-dump-and-restore).


### <a name="backup-retention"></a>Retención de copia de seguridad

Las copias de seguridad se conservan según el valor del período de retención de copia de seguridad en el servidor. Puede seleccionar un período de retención de entre 7 y 35 días. El período de retención predeterminado es de siete días. Puede establecer el período de retención durante la creación del servidor, o bien puede actualizarlo más adelante. Las copias de seguridad de los servidores detenidos también se conservan.

El período de retención de copia de seguridad rige durante cuánto tiempo se puede realizar una restauración a un momento dado, porque se basa en las copias de seguridad disponibles. El período de retención de copia de seguridad también se puede tratar como un período de recuperación desde una perspectiva de restauración. Todas las copias de seguridad que se necesitan para realizar una restauración a un momento dado dentro del período de retención de copia de seguridad se conservan en el almacenamiento de copia de seguridad. Por ejemplo, si el período de retención de la copia de seguridad se establece en siete días, el período de recuperación comprendería los últimos siete días. En este escenario, se conservan todos los datos y los registros necesarios para restaurar y recuperar el servidor en los últimos siete días. 


### <a name="backup-storage-cost"></a>Costo del almacenamiento de copia de seguridad

El Servidor flexible proporciona hasta un 100 % del almacenamiento del servidor aprovisionado como almacenamiento de copia de seguridad, sin costos adicionales. El cargo de cualquier almacenamiento de copia de seguridad adicional que se use se realizará por GB/mes. Por ejemplo, si ha aprovisionado un servidor con 250 GiB de almacenamiento, tendrá 250 GiB de capacidad para almacenar copias de seguridad sin costos adicionales. Si el uso de la copia de seguridad diaria es de 25 GiB, puede tener hasta diez días de almacenamiento de copia de seguridad. El consumo de almacenamiento de copia de seguridad que supere los 250 GiB se cobrará siguiendo el [modelo de precios](https://azure.microsoft.com/pricing/details/postgresql/).

Puede usar la métrica [Almacenamiento de copia de seguridad utilizado](https://docs.microsoft.com/azure/postgresql/concepts-monitoring) en Azure Portal para supervisar el almacenamiento de copia de seguridad que usa un servidor. La métrica Almacenamiento de copia de seguridad utilizado representa la suma del almacenamiento consumido por todas las copias de seguridad de base de datos y copias de seguridad de registros, que se conservan durante el período de retención de copia de seguridad establecido para el servidor.  Una gran actividad transaccional en el servidor puede hacer que el uso del almacenamiento de copia de seguridad aumente, independientemente del tamaño total de la base de datos.

El medio principal para controlar el costo de almacenamiento de copia de seguridad es establecer el período de retención apropiado y elegir las opciones adecuadas de redundancia de copia de seguridad para satisfacer los objetivos de recuperación deseados.

> [!IMPORTANT]
> Las copias de seguridad con redundancia geográfica no se admiten actualmente en el servidor flexible.

## <a name="point-in-time-restore-overview"></a>Información general sobre la restauración a un momento dado

En un servidor flexible, al realizar una restauración a un momento dado, se crea un nuevo servidor a partir de las copias de seguridad del servidor flexible en la misma región que el de origen. Se crea con la configuración del servidor de origen relativa al plan de tarifa, generación de procesos, número de núcleos virtuales, tamaño de almacenamiento, período de retención de copia de seguridad y opción de redundancia de copia de seguridad. Además, las etiquetas y configuraciones como la configuración de la red virtual y del firewall se heredan del servidor de origen. 

 > [!IMPORTANT]
> Si va a restaurar un servidor flexible configurado con alta disponibilidad y redundancia geográfica, el servidor restaurado se configurará sin alta disponibilidad y en la misma región que el principal. 

 ### <a name="restore-process"></a>Proceso de restauración

Los archivos físicos de base de datos se restauran primero a partir de las copias de seguridad de instantáneas en la ubicación de datos del servidor. Se elige y restaura automáticamente la copia de seguridad pertinente, realizada antes del momento indicado. A continuación, se inicia un proceso de recuperación con archivos WAL para llevar la base de datos en un estado coherente. 

 Por ejemplo, supongamos que las copias de seguridad se realizan todas las noches a las 23:00. Si el punto de restauración es el 15 de agosto de 2020 a las 10:00, se restaurará la copia de seguridad diaria del 14 de agosto de 2020. Se recuperará la base de datos hasta las 10:00 del 25 de agosto de 2020 con la copia de seguridad del registro de transacciones entre el 24 de agosto a las 23:00 y el 25 de agosto a las 10:00. 

 Siga [estos pasos](./how-to-restore-server-portal.md) para restaurar el servidor de base de datos.

> [!IMPORTANT]
> Las operaciones de restauración en un servidor flexible crean un nuevo servidor de base de datos y no sobrescriben el servidor de base de datos existente.

La restauración a un momento dado es útil en diversos escenarios. Por ejemplo, cuando un usuario elimina accidentalmente los datos, elimina una tabla importante o la base de datos, o si una aplicación sobrescribe accidentalmente los datos correctos con datos incorrectos debido a un defecto de la aplicación. Gracias a la copia de seguridad continua de los registros de transacciones, podrá restaurar el servidor a la última transacción.

Puede elegir entre el primer punto de restauración y uno personalizado.

-   **Punto de restauración más antiguo**: en función del período de retención, será el registro más antiguo que se pueda restaurar. Se seleccionará automáticamente la hora de la copia de seguridad más antigua y se mostrará en el portal. Esto resulta útil si quiere llevar a cabo investigaciones o pruebas a partir de ese momento.

-   **Punto de restauración personalizado**: esta opción le permite elegir cualquier momento dentro del período de retención definido para este servidor flexible. De forma predeterminada, se selecciona automáticamente la última hora UTC, que resulta útil para realizar la restauración a la última transacción confirmada con fines de prueba. También puede elegir otros días y horas. 

El tiempo estimado de recuperación depende de varios factores, como el tamaño de la base de datos, el volumen de los registros de transacciones que se van a procesar, el ancho de banda de red y el número total de bases de datos que se están recuperando en la misma región al mismo tiempo. Normalmente, el tiempo de recuperación general tarda entre unos minutos y unas horas.


> [!IMPORTANT]
> Los servidores eliminados **no** se pueden restaurar. Si elimina el servidor, todas las bases de datos que pertenecen al servidor también se eliminan y no se pueden recuperar. Para proteger los recursos del servidor, después de la implementación, de eliminaciones accidentales o cambios inesperados, los administradores pueden aprovechar los  [bloqueos de administración](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-lock-resources).

## <a name="perform-post-restore-tasks"></a>Tareas posteriores a la restauración

Después de restaurar la base de datos, puede realizar las siguientes tareas para que los usuarios y las aplicaciones vuelvan a conectarse:

-   Si el nuevo servidor está destinado a reemplazar al original, redirija a los clientes y las aplicaciones cliente al nuevo servidor.

-   Asegúrese de aplicar reglas de red virtual y de firewall de nivel de servidor adecuadas para que se conecten los usuarios. Estas reglas no se copian desde el servidor original.
  
-   El proceso del servidor restaurado se puede escalar vertical u horizontalmente según sea necesario.

-   No se olvide de emplear los permisos de nivel de base de datos y los inicios de sesión apropiados.

-   Configure las alertas según corresponda.
  
-  Si ha restaurado la base de datos con alta disponibilidad y quiere configurar el servidor restaurado con alta disponibilidad, siga [estos pasos](./how-to-manage-high-availability-portal.md).


## <a name="next-steps"></a>Pasos siguientes

-   Más información sobre la [continuidad empresarial](./concepts-business-continuity.md)
-   Obtenga información sobre la [alta disponibilidad con redundancia geográfica](./concepts-high-availability.md).
-   Obtenga información sobre [cómo llevar a cabo una restauración](./how-to-restore-server-portal.md).

