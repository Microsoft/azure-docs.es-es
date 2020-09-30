---
title: Copia de seguridad y restauración en el Servidor flexible de Azure Database for MySQL
description: Obtenga información sobre los conceptos de la copia de seguridad y restauración con el Servidor flexible de Azure Database for MySQL
author: kummanish
ms.author: manishku
ms.service: mysql
ms.topic: conceptual
ms.date: 09/21/2020
ms.openlocfilehash: a5f11a6054358584a8511cb967860c89f6491beb
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932266"
---
# <a name="backup-and-restore-in-azure-database-for-mysql-flexible-server-preview"></a>Copia de seguridad y restauración en el Servidor flexible de Azure Database for MySQL (versión preliminar)

> [!IMPORTANT] 
> Actualmente, Azure Database for MySQL: Servidor flexible se encuentra en versión preliminar pública.

El Servidor flexible de Azure Database for MySQL crea automáticamente copias de seguridad del servidor y las almacena de forma segura en el almacenamiento con redundancia local en la región. Las copias de seguridad pueden utilizarse para restaurar el servidor a un momento dado. Las copias de seguridad y las restauraciones son una parte esencial de cualquier estrategia de continuidad del negocio, ya que protegen los datos frente a daños o eliminaciones accidentales.

## <a name="backup-overview"></a>Introducción a Backup

El Servidor flexible toma copias de seguridad de instantáneas de los archivos de datos y las almacena en un almacenamiento con redundancia local. El servidor también realiza copias de seguridad de registros de transacciones y los almacena en el almacenamiento con redundancia local. Estas copias de seguridad permiten restaurar un servidor a un momento dado dentro del período de retención de copias de seguridad configurado. El período de retención predeterminado es siete días. Opcionalmente, puede configurar la copia de seguridad de base de datos de 1 a 35 días. Todas las copias de seguridad se cifran mediante cifrado AES de 256 bits para los datos almacenados en reposo.

Estos archivos de copia de seguridad no se pueden exportar. Las copias de seguridad solo se pueden usar para operaciones de restauración en el Servidor flexible. También puede usar [mysqldump](https://docs.microsoft.com/azure/MySQL/howto-migrate-using-dump-and-restore) de un cliente MySQL para copiar una base de datos.

## <a name="backup-frequency"></a>Frecuencia de copia de seguridad

Las copias de seguridad en servidores flexibles se basan en instantáneas. La primera copia de seguridad de instantáneas completa, se programa inmediatamente después de la creación del servidor. Esa primera copia se conserva como la copia de seguridad base del servidor. Las copias de seguridad de instantáneas posteriores son solo copias de seguridad diferenciales.

Las copias de seguridad de instantáneas diferenciales se producen al menos una vez al día. Las copias de seguridad de instantáneas diferenciales no se realizan según una programación fija. Las copias de seguridad de instantáneas diferenciales se producen cada 24 horas, a menos que los registros binarios en MySQL superen los 50 GB desde la última copia de seguridad diferencial. En un día, se permite un máximo de seis instantáneas diferenciales. Las copias de seguridad del registro de transacciones tienen lugar cada cinco minutos.

## <a name="backup-retention"></a>Retención de copias de seguridad

Las copias de seguridad de base de datos se almacenan en un almacenamiento con redundancia local (LRS), que se almacena en tres copias en una región. Las copias de seguridad se conservan según el valor del período de retención de copia de seguridad en el servidor. Puede seleccionar un período de retención de 1 a 35 días con un período de retención predeterminado de siete días. Puede establecer el período de retención durante la creación del servidor o en otro momento actualizando la configuración de copia de seguridad mediante Azure Portal.

El período de retención de la copia de seguridad rige durante cuánto tiempo se puede realizar una restauración a un momento dado, porque se basa en las copias de seguridad disponibles. El período de retención de copia de seguridad también se puede tratar como un período de recuperación desde una perspectiva de restauración. Todas las copias de seguridad que se necesitan para realizar una restauración a un momento dado dentro del período de retención de copia de seguridad, se conservan en el almacenamiento de copia de seguridad. Por ejemplo, si el período de retención de la copia de seguridad se establece en siete días, la ventana de recuperación comprenderá los últimos siete días. En este escenario, se conservan todas las copias de seguridad necesarias para restaurar el servidor en los últimos siete días. Con una ventana de retención de copia de seguridad de siete días, las instantáneas de base de datos y las copias de seguridad del registro de transacciones se almacenan durante los últimos ocho días (1 día antes de la ventana).

## <a name="backup-storage-cost"></a>Costo del almacenamiento de copia de seguridad

El Servidor flexible proporciona hasta un 100 % del almacenamiento del servidor aprovisionado como almacenamiento de copia de seguridad, sin costos adicionales. El cargo de cualquier almacenamiento de copia de seguridad adicional que se use se realizará por GB/mes. Por ejemplo, si ha aprovisionado un servidor con 250 GB de almacenamiento, tiene 250 GB de almacenamiento disponible para las copias de seguridad del servidor sin ningún cargo adicional. Si la utilización de la copia de seguridad diaria es de 25 GB, puede tener disponible hasta 10 días de almacenamiento de copia de seguridad gratuito. El almacenamiento consumido para copias de seguridad que supere los 250 GB se cobra según el  [modelo de precios](https://azure.microsoft.com/pricing/details/mysql/).

Puede usar la métrica  [Almacenamiento de copia de seguridad utilizado](https://docs.microsoft.com/azure/mysql/concepts-monitoring) en Azure Monitor disponible en Azure Portal para supervisar el almacenamiento de copia de seguridad que consume un servidor. La métrica **Almacenamiento de copia de seguridad utilizado** representa la suma del almacenamiento que consumen todas las copias de seguridad de base de datos y las copias de seguridad de registros, que se conservan en función del período de retención de copia de seguridad establecido para el servidor. Una gran actividad transaccional en el servidor puede hacer que el uso del almacenamiento de copia de seguridad aumente, independientemente del tamaño total de la base de datos.

El medio principal para controlar el costo de almacenamiento de la copia de seguridad es establecer el período de retención de la copia de seguridad adecuado. Puede seleccionar un período de retención de 1 a 35 días.

> [!IMPORTANT]
> Las copias de seguridad de un servidor de base de datos configurado en una configuración de alta disponibilidad con redundancia de zona se producen desde el servidor de base de datos principal, ya que la sobrecarga es mínima con las copias de seguridad de instantáneas.

> [!IMPORTANT]
> Las copias de seguridad con redundancia geográfica no se admiten actualmente en el servidor flexible.

## <a name="point-in-time-restore"></a>Restauración a un momento dado

En el Servidor flexible de Azure Database for MySQL, al realizar una restauración a un momento dado, se crea un servidor nuevo a partir de las copias de seguridad del servidor flexible en la misma región que el servidor de origen. Se crea con la configuración del servidor original para el nivel de proceso, número de núcleos virtuales, tamaño de almacenamiento, período de retención de copia de seguridad y opción de redundancia de copia de seguridad. Además, las etiquetas y configuraciones como las de la red virtual y del firewall se heredan del servidor de origen. El nivel de proceso y la capa de almacenamiento del servidor restaurado, la configuración y los valores de seguridad se pueden cambiar una vez completada la restauración.

> [!NOTE]
> Hay dos parámetros de servidor que se restablecen a los valores predeterminados (y no se copian del servidor principal) después de la operación de restauración.
> *   time_zone: este valor se establece en DEFAULT value SYSTEM.
> *   event_scheduler: este parámetro se establece en OFF en el servidor restaurado.

La restauración a un momento dado es útil en diversos escenarios. Algunos de los casos de uso comunes incluyen los siguientes:
-   Cuando un usuario elimina accidentalmente datos de la base de datos.
-   El usuario quita una tabla o base de datos importante.
-   Una aplicación de usuario sobrescribe accidentalmente datos correctos por otros no válidos debido a un defecto en esta.

Puede elegir entre el último punto de restauración y un punto de restauración personalizado a través de [Azure Portal](how-to-restore-server-portal.md).

-   **Último punto de restauración**: el último punto de restauración ayuda a restaurar el servidor a la última copia de seguridad realizada en el servidor de origen. La marca de tiempo para la restauración también se mostrará en el portal. Esta opción es útil para restaurar rápidamente el servidor al estado más actualizado.
-   **Personalizar el punto de restauración**: esto le permitirá elegir cualquier momento dentro del período de retención definido para este servidor flexible. Esta opción es útil para restaurar el servidor en el momento preciso de recuperación de un error de usuario.

El tiempo estimado de recuperación depende de varios factores, entre los que se incluyen los tamaños de las bases de datos, el tamaño de la copia de seguridad del registro de transacciones, el tamaño de proceso de la SKU y la hora de la restauración. La recuperación del registro de transacciones es la parte más lenta del proceso de restauración. Si el momento de la restauración se elige más próximo a la programación de la copia de seguridad de instantáneas completa o diferencial, las restauraciones son más rápidas, ya que la aplicación del registro de transacciones es mínima. A fin de calcular el tiempo de recuperación preciso para el servidor, es muy recomendable probarla en el entorno, ya que tiene demasiadas variables específicas de entorno.

> [!IMPORTANT]
> Si va a restaurar un servidor flexible configurado con alta disponibilidad con redundancia de zona, el servidor restaurado se configurará en la misma región y zona que el servidor principal, y se implementará como un único servidor flexible en un modo sin alta disponibilidad. Consulte [alta disponibilidad con redundancia de zona](concepts-high-availability.md) para un servidor flexible.

> [!IMPORTANT]
> Los servidores eliminados **no se pueden** restaurar. Si elimina el servidor, todas las bases de datos que pertenecen al servidor también se eliminan y no se pueden recuperar. Para proteger los recursos del servidor, después de la implementación, de eliminaciones accidentales o cambios inesperados, los administradores pueden aprovechar los  [bloqueos de administración](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-lock-resources).

## <a name="perform-post-restore-tasks"></a>Tareas posteriores a la restauración

Cuando efectúe la restauración con el mecanismo de recuperación **último punto de restauración** o **punto de restauración personalizado**, deberá realizar las tareas siguientes para que los usuarios y las aplicaciones vuelvan a conectarse:

-   Si el servidor nuevo está destinado a reemplazar al servidor original, redirija a los clientes y las aplicaciones cliente al servidor nuevo.
-   Asegúrese de aplicar reglas de red virtual y de firewall de nivel de servidor adecuadas para que se conecten los usuarios.
-   No se olvide de emplear los permisos de nivel de base de datos y los inicios de sesión apropiados.
-   Configure las alertas según corresponda.

## <a name="next-steps"></a>Pasos siguientes

-   Más información sobre la [continuidad empresarial](./concepts-business-continuity.md)
-   Más información sobre la  [alta disponibilidad con redundancia de zona](./concepts-high-availability.md)
-   Más información sobre [copia de seguridad y recuperación](./concepts-backup-restore.md)