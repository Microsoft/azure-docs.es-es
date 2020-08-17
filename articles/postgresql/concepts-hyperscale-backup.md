---
title: 'Copia de seguridad y restauración: Hiperescala (Citus) en Azure Database for PostgreSQL'
description: Protección de datos frente a daños o eliminaciones accidentales
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: conceptual
ms.date: 04/28/2020
ms.openlocfilehash: 22bc3d6efca24a88b28217b2e06ac79d33f16b2e
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87030086"
---
# <a name="backup-and-restore-in-azure-database-for-postgresql---hyperscale-citus"></a>Copia de seguridad y restauración en Azure Database for PostgreSQL: Hiperescala (Citus)

Azure Database for PostgreSQL: Hiperescala (Citus) crea automáticamente copias de seguridad de cada nodo y las almacena en un almacenamiento con redundancia local. Las copias de seguridad se pueden usar para restaurar el clúster de Hiperescala (Citus) a una hora especificada. Las copias de seguridad y las restauraciones son una parte esencial de cualquier estrategia de continuidad del negocio, ya que protegen los datos frente a daños o eliminaciones accidentales.

## <a name="backups"></a>Copias de seguridad

Al menos una vez al día, Azure Database for PostgreSQL realiza copias de seguridad de instantáneas de los archivos de datos y del registro de transacciones de la base de datos. Las copias de seguridad permiten restaurar un servidor a un momento dado dentro del período de retención. (El período de retención es actualmente de 35 días para todos los clústeres). Todas las copias de seguridad se cifran mediante cifrado AES de 256 bits.

En las regiones de Azure que admiten zonas de disponibilidad, las instantáneas de copia de seguridad se almacenan en tres zonas de disponibilidad. Siempre que haya al menos una zona de disponibilidad en línea, el clúster de Hiperescala (Citus) se puede restaurar.

Los archivos de copia de seguridad no se pueden exportar. Solo se pueden usar para operaciones de restauración en Azure Database for PostgreSQL.

### <a name="backup-storage-cost"></a>Costo del almacenamiento de copia de seguridad

Para obtener información sobre los precios del almacenamiento de copia de seguridad actual, consulte la [página de precios](https://azure.microsoft.com/pricing/details/postgresql/hyperscale-citus/) de Azure Database for PostgreSQL: Hiperescala (Citus).

## <a name="restore"></a>Restauración

En Azure Database for PostgreSQL, la restauración de un clúster de Hiperescala (Citus) crea un clúster nuevo a partir de las copias de seguridad de los nodos originales. 

> [!IMPORTANT]
>Solo puede restaurar el clúster de hiperescala dentro de la misma suscripción y el mismo grupo de recursos, con un nombre de clúster diferente.


> [!IMPORTANT]
> Los clústeres eliminados de Hiperescala (Citus) no se pueden restaurar. Si elimina el clúster, todos los nodos que pertenecen a este se eliminan y no se pueden recuperar. Para proteger los recursos de clúster, después de la implementación, de eliminaciones accidentales o cambios inesperados, los administradores pueden aprovechar los [bloqueos de administración](/azure/azure-resource-manager/management/lock-resources).

### <a name="point-in-time-restore-pitr"></a>Restauración a un momento dado

Puede restaurar un clúster a un momento dado cualquiera de los últimos 35 días.
La restauración a un momento dado es útil en diversos escenarios. Por ejemplo, cuando un usuario elimina accidentalmente los datos, elimina una tabla importante o la base de datos, o si una aplicación sobrescribe accidentalmente los datos correctos con datos incorrectos.

El proceso de restauración crea un nuevo clúster en la misma región, suscripción y grupo de recursos de Azure que el clúster original. El clúster tiene la misma configuración del original: el mismo número de nodos, de núcleos virtuales, tamaño de almacenamiento, roles de usuario, versión de PostgreSQL y versión de la extensión de Citus.

La configuración del firewall y los parámetros del servidor de PostgreSQL no se conservan del grupo de servidores original sino que se restablecen a los valores predeterminados. El firewall impedirá todas las conexiones. Tendrá que ajustar manualmente estos valores después de la restauración.

> [!IMPORTANT]
> Tendrá que abrir una solicitud de soporte técnico para realizar la restauración a un momento dado del clúster de Hiperescala (Citus).

### <a name="post-restore-tasks"></a>Tareas posteriores a la restauración

Cuando efectúe una restauración con cualquiera de los mecanismos de recuperación, debe hacer lo siguiente para que los usuarios y las aplicaciones vuelvan a conectarse:

* Si el nuevo servidor está destinado a reemplazar al servidor original, redirija los clientes y las aplicaciones de cliente al nuevo servidor.
* Asegúrese de aplicar reglas de red virtual y de firewall de nivel de servidor adecuadas para que se conecten los usuarios. Estas reglas no se copian del grupo de servidores original.
* Ajuste los parámetros del servidor PostgreSQL según sea necesario. Los parámetros no se copian del grupo de servidores original.
* No se olvide de emplear los permisos de nivel de base de datos y los inicios de sesión apropiados.
* Configure las alertas según corresponda.

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información acerca de las  [zonas de disponibilidad de Azure](/azure/availability-zones/az-overview).
* Establezca  [alertas sugeridas](/azure/postgresql/howto-hyperscale-alert-on-metric#suggested-alerts) en los grupos de servidores de Hiperescala (Citus).
