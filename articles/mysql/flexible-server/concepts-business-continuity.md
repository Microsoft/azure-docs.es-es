---
title: 'Introducción a la continuidad empresarial con Azure Database for MySQL: servidor flexible'
description: 'Obtenga información sobre los conceptos de la continuidad empresarial Azure Database for MySQL: servidor flexible.'
author: kummanish
ms.author: manishku
ms.service: mysql
ms.topic: conceptual
ms.date: 09/21/2020
ms.openlocfilehash: 62ca7ea885605b3b5590342b6786dcdc63f3a00b
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932112"
---
# <a name="overview-of-business-continuity-with-azure-database-for-mysql---flexible-server-preview"></a>Introducción a la continuidad empresarial con Azure Database for MySQL: servidor flexible (versión preliminar)

> [!IMPORTANT]
> Azure Database for MySQL: servidor flexible está actualmente en versión preliminar pública.

Azure Database for MySQL: servidor flexible habilita capacidades de continuidad empresarial que protegen las bases de datos en caso de una interrupción, ya sea planeada o no. Características como las copias de seguridad automatizadas y la alta disponibilidad abordan distintos niveles de protección ante errores con diferentes tiempos de recuperación y riesgos de pérdida de datos. Cuando diseñe la aplicación para protegerse frente a errores, debe tener en cuenta el objetivo de tiempo de recuperación (RTO) y el objetivo de punto de recuperación (RPO) de cada aplicación. El RTO es la tolerancia al tiempo de inactividad y el RPO es la tolerancia de pérdida de datos después de una interrupción del servicio de base de datos.

En la tabla siguiente se muestran las características que ofrece un servidor flexible.


| **Característica** | **Descripción** | **Restricciones** |
| ---------- | ----------- | ------------ |
| **Copia de seguridad y recuperación** | Un servidor flexible realiza automáticamente copias de seguridad diarias de los archivos de base de datos y copias de seguridad de los registros de transacciones de forma continuada. Las copias de seguridad se pueden conservar durante cualquier período entre 1 y 35 días. Es posible restaurar el servidor de bases de datos a un momento dado dentro del período de retención de la copia de seguridad. El tiempo de recuperación depende del tamaño de los datos que se van a restaurar más el tiempo necesario para realizar la recuperación del registro. Vea [Conceptos: copia de seguridad y restauración](./concepts-backup-restore.md) para obtener más detalles. |Datos de copia de seguridad conservados dentro de la región |
| **Copia de seguridad con redundancia local** | Las copias de seguridad de un servidor flexible se almacenan de forma automática y segura en un almacenamiento con redundancia local dentro de una región y la misma zona de disponibilidad. Las copias de seguridad con redundancia local replican los archivos de datos de la copia de seguridad del servidor tres veces dentro de una única ubicación física de la región primaria. El almacenamiento de copia de seguridad con redundancia local proporciona una durabilidad mínima del 99,999999999 % (once nueves) de los objetos a lo largo de un año determinado. Vea [Conceptos: copia de seguridad y restauración](./concepts-backup-restore.md) para obtener más detalles.| Aplicable en todas las regiones |
| **Alta disponibilidad con redundancia de zona** | Un servidor flexible se puede implementar en modo de alta disponibilidad, que implementa los servidores principal y en espera en dos zonas de disponibilidad diferentes dentro de una región. Esto protege frente a errores de nivel de zona y, además, ayuda a reducir el tiempo de inactividad de la aplicación durante eventos de tiempo de inactividad, ya sean planeados o no. Los datos del servidor principal se replican de forma sincrónica en la réplica en espera. Durante cualquier evento de tiempo de inactividad, el servidor de bases de datos se conmuta por error automáticamente en la réplica en espera. Vea [Conceptos: alta disponibilidad](./concepts-high-availability.md) para obtener más detalles. | Es compatible con los niveles de proceso de uso general y optimizado para memoria. Está disponible únicamente en regiones con varias zonas.|
| **Recursos compartidos de archivos Premium** | Los archivos de base de datos se almacenan en recursos compartidos Premium de Azure de gran durabilidad y confiabilidad que proporcionan redundancia de datos con tres copias de réplica almacenadas en una zona de disponibilidad con capacidades de recuperación de datos automáticas. Vea [Recursos compartidos de archivos Premium](../../storage/files/storage-how-to-create-premium-fileshare.md) para obtener más detalles. | Datos almacenados en una zona de disponibilidad |

> [!IMPORTANT]
> No se ofrece ningún contrato de nivel de servicio de tiempo de actividad, RTO ni RPO durante el período de versión preliminar. Los detalles que se proporcionan en esta página solo tienen fines informativos y de planeación.

## <a name="planned-downtime-mitigation"></a>Mitigación del tiempo de inactividad planeado
Estos son algunos escenarios de mantenimiento planeado que conllevan tiempo de inactividad:

| **Escenario** | **Process**|
| :------------ | :----------- |
| **Escalado de proceso (usuario)**| Cuando se realiza una operación de escalado de proceso, se aprovisiona un nuevo servidor flexible con la configuración del proceso escalado. Antes de apagar el servidor de bases de datos existente, se permite que finalicen los puntos de comprobación activos, se purgan las conexiones de cliente y se cancelan las transacciones sin confirmar. Después, el almacenamiento se conecta al nuevo servidor y se inicia la base de datos, que, en caso necesario, realiza una recuperación antes de aceptar conexiones de cliente. |
| **Nueva implementación de software (Azure)** | La implementación de nuevas características o correcciones de errores se producen automáticamente como parte del mantenimiento planeado del servicio, y es posible programar cuándo se realizan. Para obtener más información, vea la [documentación](https://aka.ms/servicehealthpm) y el [portal](https://aka.ms/servicehealthpm). |
| **Actualizaciones de versiones secundarias (Azure)** | Azure Database for MySQL revisa automáticamente los servidores de bases de datos según la versión secundaria determinada por Azure. Se produce como parte del mantenimiento planeado del servicio. Esto provocaría un breve tiempo de inactividad en términos de segundos y el servidor de base de datos se reiniciará automáticamente con la nueva versión secundaria. Para obtener más información, vea la [documentación](https://docs.microsoft.com/azure/mysql/concepts-monitoring#planned-maintenance-notification) y el [portal](https://aka.ms/servicehealthpm).|

Cuando el servidor flexible se configura con **alta disponibilidad con redundancia de zona**, primero realiza operaciones en el servidor en espera y, después, en el principal sin conmutación por error. Vea [Conceptos: alta disponibilidad](./concepts-high-availability.md) para obtener más detalles.

##  <a name="unplanned-downtime-mitigation"></a>Mitigación del tiempo de inactividad no planeado

Se pueden producir tiempos de inactividad no planeados como resultado de errores imprevistos, incluidos errores subyacentes de hardware, problemas de red y errores de software. Si el servidor de bases de datos deja de funcionar de forma inesperada, en caso de estar configurado con alta disponibilidad (HA), se activa la réplica en espera. Si no es así, se aprovisiona automáticamente un nuevo servidor de bases de datos. Aunque no es posible evitar un tiempo de inactividad no planeado, el servidor flexible lo reduce gracias a las operaciones de recuperación automáticas tanto en el servidor de bases de datos como en las capas de almacenamiento sin necesidad de intervención humana.

### <a name="unplanned-downtime-failure-scenarios-and-service-recovery"></a>Tiempo de inactividad no planeado: escenarios de error y recuperación de servicio

Estos son algunos escenarios de error no planeados y el proceso de recuperación:

| **Escenario** | **Proceso de recuperación (sin alta disponibilidad)** | **Proceso de recuperación (con HA)** |
| :---------- | ---------- | ------- |
| **Error de servidor de bases de datos** | Si el servidor de base de datos está inactivo debido a un error de hardware subyacente, se quitan las conexiones activas y se anulan las transacciones inactivas. Azure intentará reiniciar el servidor de bases de datos. Si lo hace correctamente, se realizará la recuperación de la base de datos. Si se produce un error de reinicio, el servidor de bases de datos se intentará reiniciar en otro nodo físico.  <br /> <br /> El tiempo de recuperación (RTO) depende de varios factores, por ejemplo, la actividad en el momento del error, como una transacción de gran tamaño y la cantidad de recuperación que se va a realizar durante el proceso de inicio del servidor de bases de datos. <br /> <br /> Las aplicaciones que usan bases de datos de MySQL se deben crear de forma que detecten y reintenten las conexiones eliminadas y las transacciones erróneas.  Cuando la aplicación vuelva a intentarlo, las conexiones se dirigirán al servidor de bases de datos recién creado. | Si se detecta un error del servidor de bases de datos, se activará el servidor de bases de datos en espera, lo que reducirá el tiempo de inactividad. Vea la [página de conceptos de alta disponibilidad](concepts-high-availability.md) para obtener más detalles. Se espera que el RTO sea de entre 60 y 120 s, con RPO = 0. |
| **Error de almacenamiento** | Las aplicaciones no verán ningún impacto por los problemas relacionados con el almacenamiento, como un error de disco o un daño de bloque físico. A medida que los datos se almacenan en tres copias, el almacenamiento sobreviviente proporciona la copia de los datos. Los daños en bloques se corrigen automáticamente. Si se pierde una copia de los datos, se crea automáticamente una nueva. | En el caso de los errores no recuperables, el servidor flexible se conmuta por error en la réplica en espera para reducir el tiempo de inactividad. Vea la [página de conceptos de alta disponibilidad](../concepts-high-availability.md) para obtener más detalles. |
| **Errores de usuario o lógicos** | La recuperación de los errores de usuario, como las tablas eliminadas accidentalmente o los datos actualizados incorrectamente, implica la realización de una [recuperación a un momento dado](https://docs.microsoft.com/azure/MySQL/concepts-backup) (PITR), de modo que se restauran y recuperan los datos hasta el momento justo antes de que se produjera el error.<br> <br>  Si quiere restaurar únicamente un subconjunto de bases de datos o tablas específicas en lugar de todas las bases de datos del servidor de bases de datos, puede restaurar el servidor de base de datos en una nueva instancia, exportar las tablas a través de [pg_dump](https://www.MySQL.org/docs/11/app-pgdump.html) y, a continuación, usar [pg_restore](https://www.MySQL.org/docs/11/app-pgrestore.html) para restaurar esas tablas en la base de datos. | Estos errores de usuario no están protegidos con alta disponibilidad debido al hecho de que todas las operaciones de usuario se replican también en el servidor en espera. |
| **Error de zona de disponibilidad** | Aunque se trata de un evento poco frecuente, si quiere recuperarse de un error de nivel de zona, puede realizar una recuperación a un momento dado mediante la copia de seguridad y elegir un punto de restauración personalizado para obtener los datos más recientes. Se implementará un nuevo servidor flexible en otra zona. El tiempo necesario para la restauración dependerá de la copia de seguridad anterior y del número de registros de transacciones que se vayan a recuperar. | El servidor flexible realizará la conmutación automática por error en el sitio en espera. Vea la [página de conceptos de alta disponibilidad](../concepts-high-availability.md) para obtener más detalles. |
| **Error de región** | La réplica entre regiones y las características de restauración geográfica todavía no se admiten en la versión preliminar. | |


> [!IMPORTANT]
> Los servidores eliminados **no** se pueden restaurar. Si elimina el servidor, todas las bases de datos que pertenecen al servidor también se eliminan y no se pueden recuperar. Use el [bloqueo de recursos de Azure](https://docs.microsoft.com/azure/azure-resource-manager/management/lock-resources) para evitar la eliminación accidental del servidor.


## <a name="next-steps"></a>Pasos siguientes

-   Obtenga información sobre [alta disponibilidad con redundancia de zona](./concepts-high-availability.md)
-   Más información sobre [copia de seguridad y recuperación](./concepts-backup-restore.md)
