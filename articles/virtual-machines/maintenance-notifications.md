---
title: Notificaciones de mantenimiento
description: Información general sobre las notificaciones de mantenimiento de las máquinas virtuales que se ejecutan en Azure.
author: shants123
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.topic: conceptual
ms.date: 11/19/2019
ms.author: shants
ms.openlocfilehash: f7d9912686be8ba6076278f148067575a40f32f8
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86510045"
---
# <a name="handling-planned-maintenance-notifications"></a>Control de las notificaciones de mantenimiento planeado

Azure realiza periódicamente actualizaciones para mejorar la confiabilidad, el rendimiento y la seguridad de la infraestructura host de las máquinas virtuales. Las actualizaciones son cambios como la aplicación de revisiones en el entorno de hospedaje o la actualización y la retirada de hardware. La mayoría de estas actualizaciones se completan sin que las máquinas virtuales hospedadas resulten afectadas. Sin embargo, hay casos en los que las actualizaciones tienen un impacto:

- Si el mantenimiento no requiere un reinicio, Azure usa la migración en contexto para pausar la máquina virtual mientras se actualiza el host. Estos tipos de operaciones de mantenimiento se aplican los dominios de error, de uno en uno. El progreso se detiene si no se reciben señales de estado de advertencia.

- Si el mantenimiento requiere un reinicio, el usuario recibe un aviso de cuándo está programado el mantenimiento. Se le asigna también un período de aproximadamente 35 días en el que puede iniciar el mantenimiento, en el momento en que le sea más conveniente.


El mantenimiento planeado que requiere un reinicio se programa en olas. Cada ola tiene un ámbito diferente (regiones).

- Una ola comienza con una notificación a los clientes. De forma predeterminada, la notificación se envía al administrador y los coadministradores del servicio. Puede agregar más destinatarios y opciones de mensajes, como correo electrónico, SMS y webhooks, mediante [alertas del registro de actividad](../service-health/alerts-activity-log-service-notifications-portal.md).  
- Cuando desaparece la notificación, aparece un *periodo de autoservicio*. Durante esta periodo, puede consultar cuáles de las máquinas virtuales han resultado afectadas e iniciar el mantenimiento en función de sus propias necesidades de programación. El periodo de autoservicio suele ser de unos 35 días.
- Después de la ventana de autoservicio, comienza una *ventana de mantenimiento programado*. Mientras está ventana está activa Azure programa el mantenimiento necesario y lo aplica a la máquina virtual. 

El objetivo de tener dos ventanas es proporcionar tiempo suficiente para que pueda iniciar el mantenimiento y reiniciar la máquina virtual sabiendo cuando iniciará Azure el mantenimiento automáticamente.


Puede usar Azure Portal, PowerShell, la API de REST y la CLI para consultar las ventanas de mantenimiento de las máquinas virtuales e iniciar el mantenimiento de autoservicio.

 
## <a name="should-you-start-maintenance-using-during-the-self-service-window"></a>¿Debe comenzar el mantenimiento durante la ventana de autoservicio?  

Las directrices siguientes están pensadas para ayudarle a decidir si debe utilizar esta funcionalidad e iniciar el mantenimiento a su ritmo. 

> [!NOTE] 
> Es posible que el mantenimiento de autoservicio no esté disponible para todas las máquinas virtuales. Para determinar si su máquina virtual se puede volver a implementar de forma proactiva, busque **Iniciar ahora** en el estado del mantenimiento. El mantenimiento de autoservicio no está disponible para Cloud Services (rol de trabajo o web) y Service Fabric.


El mantenimiento de autoservicio no es aconsejable para implementaciones que usan **conjuntos de disponibilidad**. Los conjuntos de disponibilidad solo se actualizan dominio a dominio. 

- Deje que Azure desencadene el mantenimiento. En caso de que el mantenimiento requiera un reinicio, se realizará por dominio de actualización. Los dominios de actualización no necesariamente reciben el mantenimiento de forma secuencial y hay una pausa de 30 minutos entre dominios de actualización. 
- Si la pérdida temporal de cierta capacidad (1 dominio de actualización) supone un problema, se pueden agregar instancias durante el periodo de mantenimiento. 
- Para el mantenimiento que no requiere el reinicio, las actualizaciones se aplican en el nivel de dominio de error. 

**No** utilice el mantenimiento de autoservicio en los escenarios siguientes: 
- Si apaga las máquinas virtuales con frecuencia, ya sea manualmente, mediante DevTest Labs, con el apagado automático o de forma programada, puede revertir el estado de mantenimiento y, por tanto, provocar mayor tiempo de inactividad.
- En las máquinas virtuales de corta duración que sepa que se van a eliminar antes del final del mantenimiento. 
- En el caso de las cargas de trabajo con un estado grande almacenadas en el disco local (efímero) en el que se desea realizar el mantenimiento tras la actualización. 
- En los casos en los que se cambia el tamaño de una máquina virtual con frecuencia, ya que pudo revertir el estado de mantenimiento. 
- Si ha realizado eventos programados que permiten una conmutación por error proactiva o un apagado ordenado de la carga de trabajo, 15 minutos antes de que comience el apagado de mantenimiento

**Use** el mantenimiento de autoservicio si tiene planeado ejecutar la máquina virtual de forma ininterrumpida durante la fase de mantenimiento programado y no se produce ninguna de las contraindicaciones anteriores. 

Se recomienda usar el mantenimiento de autoservicio en los siguientes casos:
- Es preciso comunicar una ventana de mantenimiento exacta a la dirección o al cliente final. 
- Es preciso completar el mantenimiento en una fecha determinada. 
- Es preciso controlar la secuencia de mantenimiento, por ejemplo, la aplicación de niveles múltiples para garantizar la recuperación segura.
- Se necesitan más de 30 minutos de recuperación de la máquina virtual entre dos dominios de actualización (UD). Para controlar el tiempo entre los dominios de actualización, no puede desencadenar el mantenimiento en las máquinas virtuales en más de un dominio de actualización (UD) a la vez.


## <a name="faq"></a>Preguntas más frecuentes


**P: ¿Por qué hay que reiniciar las máquinas virtuales ahora?**

**R:** Aunque la mayoría de las actualizaciones de la plataforma de Azure no afectan a la disponibilidad de la máquina virtual, hay casos en los que es imposible evitar reiniciar las máquinas virtuales hospedadas en Azure. Hemos acumulado varios cambios que requieren el reinicio de nuestros servidores, lo que conllevará el reinicio de las máquinas virtuales.

**P: Si sigo las recomendaciones para lograr una alta disponibilidad mediante un conjunto de disponibilidad, ¿estoy seguro?**

**R:** Las máquinas virtuales implementadas en un conjunto de disponibilidad o en conjuntos de escalado de máquinas virtuales tienen la noción de dominios de actualización (UD). Al realizar mantenimiento, Azure respeta la restricción de UD, por lo que no reiniciará máquinas virtuales de diferentes UD (del mismo conjunto de disponibilidad).  Azure también espera al menos 30 minutos antes de pasar al siguiente grupo de máquinas virtuales. 

Para más información sobre la alta disponibilidad, consulte [Availability for virtual machines in Azure](availability.md) (Disponibilidad para máquinas virtuales en Azure).

**P: ¿Cómo recibo notificaciones acerca del mantenimiento planeado?**

**R:** Una oleada de mantenimiento planeado se inicia mediante el establecimiento de una programación en una o varias regiones de Azure. Poco después, se envía una notificación por correo electrónico a los administradores de las suscripciones (un correo electrónico por suscripción). Mediante Alertas de registro de actividad se pueden configurar más canales y destinatarios de esta notificación. En caso de que implemente una máquina virtual en una región en la que ya se ha programado un mantenimiento planeado, no recibirá la notificación, sino que tendrá que comprobar el estado de mantenimiento de la máquina virtual.

**P: No veo ninguna indicación de mantenimiento planeado en el portal, PowerShell o la CLI. ¿Qué está pasando?**

**R:** La información relacionada con el mantenimiento planeado está disponible durante una tanda de mantenimiento planeado solo para las máquinas virtuales que van a verse afectadas por el mismo. En otras palabras, si no ve datos, es posible que la oleada de mantenimiento se haya completado (o que no haya empezado) o que la máquina virtual ya esté alojada en un servidor actualizado.

**P: ¿Existe alguna una manera de saber exactamente cuándo se verá afectada mi máquina virtual?**

**R:** Al establecer la programación, definimos una ventana de tiempo de varios días. Sin embargo, se desconoce la secuencia exacta de los servidores (y las máquinas virtuales) en esta ventana. Los clientes que deseen conocer el tiempo exacto para sus máquinas virtuales pueden usar [eventos programados](./linux/scheduled-events.md) y realizar consultas desde la propia máquina virtual y recibir una notificación 15 minutos antes de que una máquina virtual se reinicie.

**P: ¿Cuánto tiempo tardará en reiniciar mi máquina virtual?**

**R:**  En función del tamaño de la máquina virtual, el reinicio podría tardar varios minutos durante la ventana de mantenimiento de autoservicio. En los reinicios ejecutados por Azure en la ventana de mantenimiento programada, el reinicio suele tardar unos 25 minutos. Tenga en cuenta que, si usa Cloud Services (roles de trabajo o web), Virtual Machine Scale Sets o conjuntos de disponibilidad, se le proporcionarán treinta minutos entre cada grupo de máquinas virtuales (UD) durante la ventana de mantenimiento programado.

**P: ¿Cuál es la experiencia en el caso de los conjuntos de escalado de máquinas virtuales?**

**R:** El mantenimiento planeado está ahora disponible para los conjuntos de escalado de máquinas virtuales. Para instrucciones sobre cómo iniciar el mantenimiento de autoservicio, consulte el documento [Notificaciones de mantenimiento planeado de conjuntos de escalado de máquinas virtuales](../virtual-machine-scale-sets/virtual-machine-scale-sets-maintenance-notifications.md).

**P: ¿Cómo es la experiencia en el caso de Cloud Services (roles de trabajo o web) y Service Fabric?**

**R:** Aunque estas plataformas resultan afectadas por el mantenimiento planeado, se considera que los clientes que las usan están seguros, ya que solo resultarán afectadas en cualquier momento las máquinas virtuales de un dominio de actualización (UD) individual. El mantenimiento de autoservicio no está disponible para Cloud Services (rol de trabajo o web) y Service Fabric.

**P: No veo información de mantenimiento en mis máquinas virtuales. ¿A qué se debe?**

**R:** Hay varios motivos por los que no ve información de mantenimiento en sus máquinas virtuales:
1.  Utiliza una suscripción marcada como interna de Microsoft.
2.  Las máquinas virtuales no están programadas para su mantenimiento. Es posible que la tanda de mantenimiento haya finalizado, se haya cancelado o se haya modificado para que las máquinas virtuales dejen de verse afectadas por ella.
3.  La columna **Mantenimiento** no se ha agregado a la vista de la lista de máquinas virtuales. Aunque hemos agregado esta columna a la vista predeterminada, los clientes que se han configurado para ver las columnas no predeterminadas deben agregar manualmente la columna de **mantenimiento** a la vista de lista de máquinas virtuales.

**P: Está programado que se realice un segundo mantenimiento programado de mi máquina virtual. ¿Por qué?**

**R:** Hay varios casos en los que verá que el mantenimiento de su máquina virtual está programado después de que se haya completado el ciclo de mantenimiento y reimplementación:
1.  Hemos cancelado la oleada de mantenimiento y la hemos reiniciado con otra carga útil. Es posible que hayamos detectado una carga útil con errores y que necesitemos implementar una carga adicional.
2.  El *servicio de una máquina virtual se ha reparado* en otro nodo debido a un error de hardware.
3.  Ha seleccionado detener (desasignar) y reiniciar la máquina virtual.
4.  El **apagado automático** se ha activado en la máquina virtual.



## <a name="next-steps"></a>Pasos siguientes

El mantenimiento planeado se puede controlar mediante la [CLI de Azure](maintenance-notifications-cli.md), [Azure PowerShell](maintenance-notifications-powershell.md) o [Azure Portal](maintenance-notifications-portal.md).
