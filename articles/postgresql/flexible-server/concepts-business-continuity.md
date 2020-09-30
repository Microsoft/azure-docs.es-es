---
title: 'Introducción a la continuidad empresarial con Azure Database for PostgreSQL: servidor flexible'
description: 'Información sobre los conceptos de la continuidad empresarial con Azure Database for PostgreSQL: servidor flexible'
author: sr-msft
ms.author: srranga
ms.service: postgresql
ms.topic: conceptual
ms.date: 09/22/2020
ms.openlocfilehash: 264bb8c66510c90fecf12d2e4e68bd969b4fb474
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932143"
---
# <a name="overview-of-business-continuity-with-azure-database-for-postgresql---flexible-server"></a>Introducción a la continuidad empresarial con Azure Database for PostgreSQL: servidor flexible

> [!IMPORTANT]
> Azure Database for PostgreSQL: servidor flexible en versión preliminar

En Azure Database for PostgreSQL: servidor flexible, la **continuidad empresarial** hace referencia a los mecanismos, directivas y procedimientos que permiten a la empresa seguir funcionando en caso de interrupciones, especialmente en lo que respecta a su infraestructura informática. En la mayoría de los casos, el servidor flexible controlará los eventos de interrupción que puedan ocurrir en el entorno en la nube y permitirá que se sigan ejecutando las aplicaciones y los procesos empresariales. No obstante, hay algunos eventos que no se pueden controlar automáticamente, por ejemplo:

- El usuario elimina o actualiza accidentalmente una fila de una tabla.
- Los terremotos provocan un corte de energía y deshabilitan temporalmente un centro de datos o una zona de disponibilidad.
- Se necesita una revisión de bases de datos para corregir un error o un problema de seguridad.

El servidor flexible proporciona características que protegen los datos y mitigan el tiempo de inactividad de las bases de datos fundamentales en el caso de eventos de tiempo de inactividad planeados y no planeados. Según la infraestructura de Azure que ya ofrece una sólida resistencia y disponibilidad, un servidor flexible dispone de características de continuidad empresarial que proporcionan protección adicional ante errores, se ocupan de los requisitos de tiempo de recuperación y reducen el riesgo de pérdida de datos. Al diseñar las aplicaciones, debe tener en cuenta la tolerancia al tiempo de inactividad, que es el objetivo de tiempo de recuperación (RTO), y el riesgo de la pérdida de datos, que es el objetivo de punto de recuperación (RPO). Por ejemplo, su base de datos empresarial esencial tiene unos requisitos de tiempo de actividad mucho más estrictos en comparación con una base de datos de prueba.  

> [!IMPORTANT]
> Durante la versión preliminar no se ofrece un contrato de nivel de servicio (SLA) de % de tiempo de actividad. 

En la siguiente tabla se muestran las características que ofrece un servidor flexible.


| **Característica** | **Descripción** | **Consideraciones** |
| ---------- | ----------- | ------------ |
| **Copias de seguridad automáticas** | Un servidor flexible realiza automáticamente copias de seguridad diarias de los archivos de la base de datos y copias de seguridad de los registros de transacciones de forma continuada. Las copias de seguridad se pueden conservar desde 7 hasta 35 días. Es posible restaurar el servidor de bases de datos a un momento dado dentro del período de retención de la copia de seguridad. El RTO depende del tamaño de los datos que se van a restaurar más el tiempo necesario para realizar la recuperación del registro. Puede tardar desde unos minutos hasta 12 horas. Para obtener más información, vea [Conceptos: copia de seguridad y restauración](./concepts-backup-restore.md). |Los datos de copia de seguridad permanecen en la región. |
| **Alta disponibilidad con redundancia de zona** | Un servidor flexible se puede implementar con configuración de alta disponibilidad (HA) redundante de zona. De este modo, los servidores principal y en espera se implementarán en dos zonas de disponibilidad diferentes dentro de una región. Esta configuración de alta disponibilidad protege sus bases de datos frente a errores de nivel de zona y además ayuda a reducir el tiempo de inactividad de la aplicación durante eventos de tiempo de inactividad planeados y no planeados. Los datos del servidor principal se replican de modo sincrónico en la réplica en espera. En caso de que se produzca una interrupción en el servidor principal, el servidor se conmuta por error automáticamente a la réplica en espera. Se espera que el RTO en la mayoría de los casos dure entre 60 y 120 s. Se espera que el RPO sea cero (sin pérdida de datos). Para obtener más información, consulte [Conceptos: alta disponibilidad](./concepts-high-availability.md). | Compatible con los niveles de proceso de uso general y optimizado para memoria. Disponible únicamente en regiones con varias zonas disponibles. |
| **Discos administrados premium** | Los archivos de la base de datos se guardan en un almacenamiento premium administrado de alta disponibilidad y muy duradero. Esto proporciona redundancia de datos con tres copias de réplicas almacenadas en una zona de disponibilidad con capacidades automáticas de recuperación de datos. Para obtener más información, consulte la [documentación de Managed Disks](https://docs.microsoft.com/azure/virtual-machines/managed-disks-overview). | Datos almacenados en una zona de disponibilidad. |
| **Copia de seguridad con redundancia de zona** | Las copias de seguridad de un servidor flexible se almacenan de forma automática y segura en un almacenamiento con redundancia de zona dentro de una región. Durante un error de nivel de zona en el que se ha aprovisionado el servidor, y si el servidor no está configurado con redundancia de zona, puede restaurar sin problemas la base de datos con el punto de restauración más reciente en otra zona. Para obtener más información, consulte [Conceptos: copias de seguridad y restauración](./concepts-backup-restore.md).| Aplicable únicamente a regiones con varias zonas disponibles.|


## <a name="planned-downtime-events"></a>Eventos de tiempo de inactividad planeados
A continuación puede ver algunos escenarios de mantenimiento planeado. Normalmente, estos eventos incurren en pocos minutos de tiempo de inactividad y sin pérdida de datos.

| **Escenario** | **Process**|
| ------------------- | ----------- | 
| <b>Escalado de proceso (iniciado por el usuario)| Durante la operación de escalado de proceso, se permite que se completen los puntos de comprobación activos, se purgan las conexiones de cliente, se cancelan las transacciones no confirmadas, se retira el almacenamiento y, a continuación, se apaga el servidor. Se aprovisiona un nuevo servidor flexible con el mismo nombre de servidor de base de datos con la configuración de proceso escalado. Después, el almacenamiento se conecta al nuevo servidor y se inicia la base de datos, que, en caso necesario, realiza una recuperación antes de aceptar conexiones de cliente. |
| <b>Escalado vertical del almacenamiento (iniciado por el usuario) | Cuando se inicia una operación de escalado vertical del almacenamiento, se permite que se completen los puntos de comprobación activos, se purgan las conexiones de cliente, se cancelan las transacciones no confirmadas y, a continuación, se apaga el servidor. El almacenamiento se escala al tamaño deseado y, a continuación, se conecta al nuevo servidor. Si es necesario, se realiza una recuperación antes de aceptar conexiones de clientes. Tenga en cuenta que no se admite la reducción vertical del tamaño de almacenamiento. |
| <b>Nueva implementación de software (iniciada por Azure) | La implementación de nuevas características o las correcciones de errores se producen automáticamente como parte del mantenimiento planeado del servicio, y es posible programar cuándo se realizan estas actividades. Para obtener más información, consulte su [portal](https://aka.ms/servicehealthpm). | 
| <b>Actualizaciones de versiones secundarias (iniciadas por Azure) | Azure Database for PostgreSQL revisa automáticamente los servidores de bases de datos según la versión secundaria determinada por Azure. Se produce como parte del mantenimiento planeado del servicio. El servidor de base de datos se reinicia automáticamente con la nueva versión secundaria. Para obtener más información, vea la [documentación](https://docs.microsoft.com/azure/postgresql/concepts-monitoring#planned-maintenance-notification). También puede comprobar su [portal](https://aka.ms/servicehealthpm).| 

 Cuando el servidor flexible se configura con **alta disponibilidad con redundancia de zona**, este realiza las operaciones de escalado y mantenimiento en el servidor en espera en primer lugar. Para obtener más información, consulte [Conceptos: alta disponibilidad](./concepts-high-availability.md).

##  <a name="unplanned-downtime-mitigation"></a>Mitigación del tiempo de inactividad no planeado

Se pueden producir tiempos de inactividad no planeados como resultado de interrupciones imprevistas, incluidos errores subyacentes de hardware, problemas de red y errores de software. Si el servidor de bases de datos configurado con una alta disponibilidad deja de funcionar de forma inesperada, la réplica en espera se activa y los clientes pueden reanudar sus operaciones. Si no se ha configurado con alta disponibilidad (HA), si se produce un error en el intento de reinicio, se aprovisiona automáticamente un nuevo servidor de base de datos. Aunque no es posible evitar un tiempo de inactividad no planeado, el servidor flexible lo reduce gracias a las operaciones de recuperación automáticas sin necesidad de intervención humana. 
   
### <a name="unplanned-downtime-failure-scenarios-and-service-recovery"></a>Tiempo de inactividad no planeado: escenarios de error y recuperación de servicio
A continuación puede ver algunos escenarios de error no planeados y el proceso de recuperación. 

| **Escenario** | **Proceso de recuperación (sin alta disponibilidad)** | **Proceso de recuperación (con alta disponibilidad)** |
| ---------- | ---------- | ------- |
| <B>Error de servidor de bases de datos | Si el servidor de base de datos está inactivo, Azure intentará reiniciar el servidor de bases de datos. Si se produce un error, el servidor de bases de datos se reiniciará en otro nodo físico.  <br /> <br /> El tiempo de recuperación (RTO) depende de varios factores, por ejemplo, la actividad en el momento del error, como una transacción de gran tamaño, y el volumen de recuperación que se va a realizar durante el proceso de inicio del servidor de bases de datos. <br /> <br /> Las aplicaciones que usan bases de datos de PostgreSQL se deben crear de forma que detecten y reintenten conexiones eliminadas y transacciones erróneas. | Si se detecta un error del servidor de bases de datos, se activa el servidor en espera, lo que reduce el tiempo de inactividad. Para obtener más información, consulte la página de [Conceptos de alta disponibilidad](../concepts-high-availability.md). Se espera que el RTO dure entre 60 y 120 s, sin pérdida de datos. |
| <B>Error de almacenamiento | Las aplicaciones no verán ningún impacto por los problemas relacionados con el almacenamiento, como un error de disco o un daño de bloque físico. Puesto que los datos se almacenan en tres copias, el almacenamiento sobreviviente proporciona la copia de los datos. El bloque de datos dañado se repara automáticamente y se crea una copia de los datos de manera automática. | En el caso de los errores poco frecuentes y no recuperables, como aquellos en los que no se puede acceder a todo el almacenamiento, el servidor flexible se conmuta por error a la réplica en espera para reducir el tiempo de inactividad. Para obtener más información, consulte la página de [Conceptos de alta disponibilidad](../concepts-high-availability.md). |
| <b> Errores de usuario o lógicos | Para recuperarse de los errores de usuario, como las tablas eliminadas accidentalmente o los datos actualizados incorrectamente, debe realizar una [recuperación a un momento dado](https://docs.microsoft.com/azure/postgresql/concepts-backup) (PITR). Al realizar la operación de restauración, especifica el punto de restauración personalizado, que es el momento justo antes de que se produjera el error.<br> <br>  Si quiere restaurar únicamente un subconjunto de bases de datos o tablas específicas en lugar de todas las bases de datos del servidor de bases de datos, puede restaurar el servidor de base de datos en una nueva instancia, exportar las tablas a través de [pg_dump](https://www.postgresql.org/docs/11/app-pgdump.html) y, a continuación, usar [pg_restore](https://www.postgresql.org/docs/11/app-pgrestore.html) para restaurar esas tablas en la base de datos. | Estos errores de usuario no están protegidos con alta disponibilidad, ya que todos los cambios se replican también de forma sincrónica en la réplica en espera. Tiene que realizar una restauración a un momento dado para recuperarse de estos errores. |
| <b> Error de zona de disponibilidad | Para recuperarse de un error de nivel de zona, puede realizar una recuperación a un momento dado mediante la copia de seguridad y elegir un punto de restauración personalizado del último momento para restaurar los datos más recientes. Se implementa un nuevo servidor flexible en otra zona no afectada. El tiempo necesario para la restauración depende de la copia de seguridad anterior y del volumen de registros de transacciones que se van a recuperar. | El servidor flexible se conmuta por error automáticamente al servidor en espera en un plazo entre 60 y 120 s sin pérdida de datos. Para obtener más información, consulte la página de [Conceptos de alta disponibilidad](../concepts-high-availability.md). | 
| <b> Error de región | La réplica de lectura entre regiones y las características de restauración geográfica de las copias de seguridad todavía no se admiten en la versión preliminar. | |


> [!IMPORTANT]
> Los servidores eliminados **no se pueden** restaurar. Si elimina el servidor, todas las bases de datos que pertenecen al servidor también se eliminan y no se pueden recuperar. Use el [bloqueo de recursos de Azure](https://docs.microsoft.com/azure/azure-resource-manager/management/lock-resources) para evitar la eliminación accidental del servidor.


## <a name="next-steps"></a>Pasos siguientes

-   Obtenga información sobre [alta disponibilidad con redundancia de zona](./concepts-high-availability.md)
-   Más información sobre las [copias de seguridad y la recuperación](./concepts-backup-restore.md)
