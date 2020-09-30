---
title: Preguntas comunes sobre Azure Migrate Server Migration
description: Obtenga respuestas a preguntas habituales sobre el uso de Azure Migrate Server Migration para migrar máquinas.
ms.topic: conceptual
ms.date: 08/28/2020
ms.openlocfilehash: b0ae28fc387125b198bed202d857c3b9ecdd44bb
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89050665"
---
# <a name="azure-migrate-server-migration-common-questions"></a>Azure Migrate Server Migration: Preguntas frecuentes

En este artículo se responde a preguntas habituales sobre Azure Migrate: Herramienta de migración del servidor. Si tiene otras preguntas, repase estos recursos:

- [Preguntas generales](resources-faq.md) sobre Azure Migrate
- Preguntas sobre el dispositivo de [Azure Migrate](common-questions-appliance.md)
- Preguntas sobre la [detección, valoración y visualización de dependencias](common-questions-discovery-assessment.md)
- Obtenga respuestas a sus preguntas en el [foro de Azure Migrate](https://aka.ms/AzureMigrateForum)

## <a name="where-should-i-install-the-replication-appliance-for-agent-based-migrations"></a>¿Dónde se debe instalar el dispositivo de replicación para las migraciones basadas en agentes?

El dispositivo de replicación debe instalarse en una máquina dedicada. El dispositivo de replicación no debe instalarse en una máquina de origen que desee replicar ni en el dispositivo de detección y evaluación de Azure Migrate que pueda haber instalado antes. Siga el [tutorial](https://docs.microsoft.com/azure/migrate/tutorial-migrate-physical-virtual-machines) para obtener más detalles.

## <a name="how-can-i-migrate-my-aws-ec2-instances-to-azure"></a>¿Cómo puedo migrar mis instancias de AWS EC2 a Azure?

Revise este [artículo](https://docs.microsoft.com/azure/migrate/tutorial-migrate-aws-virtual-machines) para detectar, evaluar y migrar las instancias de AWS EC2 a Azure.

## <a name="can-i-migrate-aws-vms-running-amazon-linux-operating-system"></a>¿Puedo migrar máquinas virtuales de AWS que ejecuten el sistema operativo de Amazon Linux?

Las máquinas virtuales que ejecutan Amazon Linux no se pueden migrar tal cual, ya que el sistema operativo de Amazon Linux solo es compatible con AWS.
Para migrar las cargas de trabajo que se ejecutan en Amazon Linux, puede poner en marcha una máquina virtual CentOS/RHEL en Azure y migrar la carga de trabajo que se ejecuta en la máquina Linux de AWS mediante una estrategia de migración de carga de trabajo adecuada. Por ejemplo, en función de la carga de trabajo, puede haber herramientas específicas de carga de trabajo que ayuden a la migración, como las bases de datos o las herramientas de implementación, en el caso de los servidores web.

## <a name="what-geographies-are-supported-for-migration-with-azure-migrate"></a>¿Qué zonas geográficas se admiten para la migración con Azure Migrate?

Revise las zonas geográficas admitidas para [nubes públicas](migrate-support-matrix.md#supported-geographies-public-cloud) y [gubernamentales](migrate-support-matrix.md#supported-geographies-azure-government).

## <a name="can-we-use-the-same-azure-migrate-project-to-migrate-to-multiple-regions"></a>¿Se puede usar el mismo proyecto de Azure Migrate para migrar a varias regiones?

Aunque puede crear evaluaciones para varias regiones en un proyecto de Azure Migrate, cada uno se puede usar para migrar servidores a una sola región de Azure. Puede crear proyectos de Azure Migrate adicionales para cada región a la que tenga que migrar.

- En el caso de las migraciones de VMware sin agente, la región de destino se bloquea una vez que ha habilitado la primera replicación.
- En el caso de las migraciones basadas en agentes (VMware, servidores físicos y servidores de otras nubes), la región de destino se bloquea una vez que se hace clic en el botón "Crear recursos" del portal mientras se configura el dispositivo de replicación.
- En el caso de las migraciones de Hyper-V sin agente, la región de destino se bloquea una vez que se hace clic en el botón "Crear recursos" del portal durante la configuración del proveedor de replicación de Hyper-V.

## <a name="can-we-use-the-same-azure-migrate-project-to-migrate-to-multiple-subscriptions"></a>¿Se puede usar el mismo proyecto de Azure Migrate para migrar a varias suscripciones? 

Sí, puede migrar a varias suscripciones de la misma región de destino con un proyecto de Azure Migrate. Puede seleccionar la suscripción de destino al habilitar la replicación de una máquina o de un conjunto de máquinas. La región de destino se bloquea después de la primera replicación para las migraciones de VMware sin agente y durante la instalación del dispositivo de replicación y del proveedor de Hyper-V para las migraciones basadas en agentes y para las migraciones de Hyper-V sin agente, respectivamente.

## <a name="what-are-the-migration-options-in-azure-migrate-server-migration"></a>¿Cuáles son las opciones de migración en Azure Migrate: Server Migration?

Azure Migrate: Server Migration ofrece dos opciones para realizar la migración de los servidores de origen o máquinas virtuales a Azure: migración sin agente y basada en agente.

Independientemente de la opción de migración que elija, el primer paso para migrar un servidor mediante Azure Migrate: Server Migration consiste en habilitar la replicación para el servidor. Se este modo, se realiza una replicación inicial de los datos de la máquina virtual o el servidor en Azure. Una vez completada la replicación inicial, se establece una replicación en curso (sincronización diferencial en curso) para migrar los datos incrementales a Azure. Una vez que la operación alcance la fase de sincronización diferencial, podrá optar por migrar a Azure en cualquier momento.  

Estas son algunas consideraciones que debe tener en cuenta a la hora de elegir una opción de migración.

Las **migraciones sin agente** no requieren que se implemente software (agentes) en las máquinas virtuales o servidores de origen que se van a migrar. La opción sin agente orquesta la replicación al integrarse con la funcionalidad proporcionada por el proveedor de virtualización.
Las opciones de replicación sin agente están disponibles para las [máquinas virtuales de VMware](https://docs.microsoft.com/azure/migrate/tutorial-migrate-vmware) y [las de Hyper-V](https://docs.microsoft.com/azure/migrate/tutorial-migrate-hyper-v).

Las **migraciones basadas en agentes** requieren de la instalación de software (agentes) de Azure Migrate en las VM o máquinas de origen que se van a migrar. La opción basada en agente no depende de la plataforma de virtualización para la funcionalidad de replicación y, por lo tanto, se puede usar con cualquier servidor que ejecute una arquitectura x86 o x64 y una versión de un sistema operativo compatible con el método de replicación basado en agente.

La opción de migración basada en agente se puede usar para las [máquinas virtuales de VMware](https://docs.microsoft.com/azure/migrate/tutorial-migrate-vmware-agent), las [máquinas virtuales de Hyper-V](https://docs.microsoft.com/azure/migrate/tutorial-migrate-physical-virtual-machines), los [servidores físicos](https://docs.microsoft.com/azure/migrate/tutorial-migrate-physical-virtual-machines), las [máquinas virtuales que se ejecutan en AWS](https://docs.microsoft.com/azure/migrate/tutorial-migrate-aws-virtual-machines), las máquinas virtuales que se ejecutan en GCP o las máquinas virtuales que se ejecutan en un proveedor de virtualización diferente. La migración basada en agentes trata a las máquinas como servidores físicos para la migración.

Aunque la migración sin agente ofrece mayor comodidad y simplicidad en comparación con las opciones de replicación basadas en agentes para los escenarios admitidos (VMWare e Hyper-V), debería considerar la posibilidad de usar el escenario basado en agente para los siguientes casos de uso:

- Entorno con restricción de IOPS: la replicación sin agente usa instantáneas y consume IOPS/ancho de banda de almacenamiento. Se recomienda el método de migración basado en agente si hay restricciones de almacenamiento/IOPS en el entorno.
- Si no tiene una instancia de vCenter Server, puede tratar las máquinas virtuales de VMware como servidores físicos y usar el flujo de trabajo de migración basado en agente.

Para obtener más información, consulte este [artículo](https://docs.microsoft.com/azure/migrate/server-migrate-overview) para comparar las opciones de migración para VMware.

## <a name="how-does-agentless-migration-work"></a>¿Cómo funciona la migración sin agente?

Azure Migrate: Server Migration ofrece opciones de replicación sin agente para la migración de máquinas virtuales de VMware y Hyper-V que ejecutan Windows o Linux. La herramienta también ofrece una opción de replicación adicional basada en agente para los servidores Windows y Linux que se puede usar para migrar servidores físicos, así como máquinas virtuales x86/x64 en VMware, Hyper-V, AWS, GCP, etc. La opción de replicación basada en agente requiere de la instalación del software del agente en el servidor o máquina virtual que se está migrando, mientras que en la opción sin agente no es necesario instalar ningún software en las máquinas virtuales, por lo que ofrece mayor comodidad y simplicidad en comparación con la opción de replicación basada en agentes.

La opción de replicación sin agente funciona mediante los mecanismos que proporciona el proveedor de virtualización (VMware, Hyper-V). En el caso de las máquinas virtuales de VMware, el mecanismo de replicación sin agente usa las instantáneas de VMware y la tecnología de seguimiento de bloques con cambios de VMware para replicar datos de discos de máquinas virtuales. Este mecanismo es similar al que usan muchos productos de copia de seguridad. En el caso de las máquinas virtuales de Hyper-V, el mecanismo de replicación sin agente usa las instantáneas de VM y la funcionalidad de seguimiento de cambios de la réplica de Hyper-V para replicar datos de discos de máquinas virtuales.

Cuando la replicación está configurada para una máquina virtual, primero pasa por una fase de replicación inicial. Durante la replicación inicial, se realiza una instantánea de la máquina virtual y se replica una copia completa de los datos de los discos de instantánea en los discos administrados de la suscripción. Una vez completada la replicación inicial de la máquina virtual, el proceso de replicación cambia a una fase de replicación incremental (replicación diferencial). En la fase de replicación incremental, los cambios en los datos que se han producido desde el último ciclo de replicación completado se replican periódicamente y se aplican a los discos administrados de la réplica, lo que mantiene la replicación sincronizada con los cambios que se producen en la máquina virtual. En el caso de las máquinas virtuales de VMware, la tecnología de seguimiento de bloques con cambios de VMware se usa para realizar un seguimiento de los cambios entre los ciclos de replicación. Al principio del ciclo de replicación, se toma una instantánea de la máquina virtual y se usa el seguimiento de bloques con cambios para obtener los cambios entre la instantánea actual y la última instantánea que se replicó correctamente. De este modo, solo es necesario replicar los datos que han cambiado desde el último ciclo de replicación completado para mantener sincronizada la replicación de la máquina virtual. Al final de cada ciclo de replicación, se publica la instantánea y se realiza la consolidación de instantáneas para la máquina virtual. Del mismo modo, en el caso de las máquinas virtuales de Hyper-V, el motor de seguimiento de cambios de la réplica de Hyper-V se usa para realizar un seguimiento de los cambios entre los ciclos de replicación consecutivos.
Al realizar la operación de migración en una máquina virtual de replicación, tiene la opción de apagar la máquina virtual local y realizar una última replicación incremental para garantizar que no haya pérdida de datos. Al realizar la opción de migración, se usan los discos administrados de la réplica correspondientes a la máquina virtual para crear la máquina virtual en Azure.

Para empezar, consulte los tutoriales [Migración sin agente de VMware](https://docs.microsoft.com/azure/migrate/tutorial-migrate-vmware) y [Migración sin agente de Hyper-V](https://docs.microsoft.com/azure/migrate/tutorial-migrate-hyper-v).

## <a name="how-does-agent-based-migration-work"></a>¿Cómo funciona la migración basada en agente?

Además de las opciones de migración sin agente para las máquinas virtuales de Hyper-V y VMware, la herramienta Server Migration proporciona una opción de migración basada en agente para migrar servidores de Windows y Linux que se ejecutan en servidores físicos o como máquinas virtuales de x86/x64 en VMware, Hyper-V, AWS, Google Cloud Platform, etc.

El método de migración basado en agente usa el software de agente instalado en el servidor que se va a migrar para replicar los datos en Azure. El proceso de replicación utiliza una arquitectura de descarga en la que el agente retransmite los datos de replicación a un servidor de replicación dedicado denominado "dispositivo de replicación" o servidor de configuración (o servidor de procesos en el caso de escalabilidad horizontal). [Más información](https://docs.microsoft.com/azure/migrate/agent-based-migration-architecture) sobre cómo funciona la opción de migración basada en agente. 

Nota: El dispositivo de replicación es diferente del dispositivo de detección de Azure Migrate y debe instalarse en una máquina independiente o dedicada.

## <a name="how-do-i-gauge-the-bandwidth-requirement-for-my-migrations"></a>¿Cómo se mide el requisito de ancho de banda para las migraciones?

El ancho de banda para la replicación de datos en Azure depende de varios factores y es una función de la rapidez con la que el dispositivo Azure Migrate local puede leer y replicar los datos en Azure. La replicación tiene dos fases: la replicación inicial y la replicación diferencial.

Cuando se inicia la replicación de una máquina virtual, se produce un ciclo de replicación inicial en el que se replican las copias completas de los discos. Una vez completada la replicación inicial, se programan periódicamente ciclos de replicación incremental (ciclos diferenciales) para transferir los cambios que se hayan producido desde el ciclo de replicación anterior.

### <a name="agentless-vmware-vm-migration"></a>Migración de VM de VMware sin agentes

Puede calcular el requisito de ancho de banda en función del volumen de datos que necesita mover durante la fase y el tiempo en los que quiere que se complete la replicación inicial (lo ideal es que la replicación inicial se haya completado al menos 3 o 4 días antes del período de migración real, para que tenga el tiempo suficiente para realizar una migración de prueba antes del período real y para mantener un tiempo de inactividad mínimo durante este).

Puede calcular el ancho de banda o el tiempo necesario para la migración de máquinas virtuales de VMware sin agente con la siguiente fórmula:

Tiempo necesario para completar la replicación inicial = {tamaño de los discos (o tamaño usado si está disponible) x 0,7 (suponiendo un 30 % de promedio de compresión, que es una estimación conservadora)}/ancho de banda disponible para la replicación.

### <a name="agent-based-vmware-vm-migration"></a>Migración de máquina virtual de VMware basada en agente

Para un método de replicación basado en agente, Deployment Planner puede ayudar a generar perfiles del entorno para la renovación de datos y ayudarle a predecir el requisito de ancho de banda necesario. Para más información, consulte este [artículo](https://docs.microsoft.com/azure/migrate/agent-based-migration-architecture#plan-vmware-deployment). 

## <a name="how-do-i-throttle-replication-in-using-azure-migrate-appliance-for-agentless-vmware-replication"></a>¿Cómo puedo limitar la replicación en el uso del dispositivo de Azure Migrate para la replicación de VMware sin agente?  

Se puede limitar mediante NetQosPolicy. Por ejemplo:

El elemento AppNamePrefix que se va a usar en NetQosPolicy es "GatewayWindowsService.exe". Se puede crear una directiva en el dispositivo de Azure Migrate para limitar el tráfico de replicación desde el dispositivo mediante la creación de una directiva como la siguiente:

New-NetQosPolicy -Name "ThrottleReplication" -AppPathNameMatchCondition "GatewayWindowsService.exe" -ThrottleRateActionBitsPerSecond 1 MB

## <a name="how-is-the-data-transmitted-from-on-prem-environment-to-azure-is-it-encrypted-before-transmission"></a>¿Cómo se transmiten los datos desde el entorno local a Azure? ¿Se cifran antes de la transmisión?

El dispositivo Azure Migrate del caso de replicación sin agente comprime los datos y los cifra antes de cargarlos. Los datos se transmiten a través de un canal de comunicación seguro a través de HTTPS y usa TLS 1.2 o posterior. Además, Azure Storage cifra automáticamente los datos al guardarlos en la nube (cifrado en reposo).  

## <a name="how-does-churn-rate-affect-agentless-replication"></a>¿Cómo afecta la tasa de modificación a la replicación sin agente?

Dado que la replicación sin agente incorpora datos, el *patrón de modificación* es más importante que la *tasa de modificación*. Cuando un archivo se escribe una y otra vez, la tasa no tiene un gran efecto. Sin embargo, un patrón en el que se escriben todos los demás sectores produce una gran renovación en el ciclo siguiente. Dado que se reduce la cantidad de datos que se transfieren, se permite que los datos se plieguen lo más posible antes de programar el siguiente ciclo.

## <a name="how-frequently-is-a-replication-cycle-scheduled"></a>¿Con qué frecuencia se programa un ciclo de replicación?

La fórmula para programar el siguiente ciclo de replicación es igual al tiempo de ciclo anterior/2 o una hora, lo que sea mayor.

Por ejemplo, si una VM tarda cuatro horas en un ciclo diferencial, su siguiente ciclo se programa dos horas después y no en la próxima hora. El proceso es distinto inmediatamente después de la replicación inicial, en la que el primer ciclo diferencial se programa inmediatamente.

## <a name="how-do-i-migrate-windows-server-2003-running-on-vmwarehyper-v-to-azure"></a>¿Cómo se migra de Windows Server 2003 en ejecución en VMware o Hyper-V a Azure?

El [soporte extendido de Windows Server 2003](https://go.microsoft.com/fwlink/?linkid=2140400) finalizó el 14 de julio de 2015.  El equipo de soporte técnico de Azure sigue ayudando a solucionar problemas relacionados con la ejecución de Windows Server 2003 en Azure. Sin embargo, este soporte se limita a los problemas que no requieren revisiones o ni soluciones en el nivel de sistema operativo.
La migración de las aplicaciones a instancias de Azure que ejecutan una versión más reciente de Windows Server es el enfoque recomendado para asegurarse de que se aprovecha de forma eficaz la flexibilidad y confiabilidad de la nube de Azure.

Sin embargo, si todavía opta por migrar Windows Server 2003 a Azure, puede usar la herramienta Azure Migrate: Server Migration si el servidor de Windows es una máquina virtual que se ejecuta en VMware o Hyper-V; consulte este artículo para [preparar las máquinas de Windows Server 2003 para la migración](https://go.microsoft.com/fwlink/?linkid=2140302).

## <a name="what-is-the-difference-between-the-test-migration-and-migrate-operations"></a>¿Cuál es la diferencia entre las operaciones de migración de prueba y de migración?

La migración de prueba proporciona una manera de probar y validar las migraciones antes de una migración real. La migración de prueba funciona al permitirle crear copias de prueba de máquinas virtuales con réplicas en un entorno de espacio aislado de Azure. El entorno de espacio aislado está delimitado por la red virtual de prueba que especifique. La operación de migración de prueba no genera interrupciones, ya que las aplicaciones continúan ejecutándose en el origen mientras permiten realizar pruebas en una copia clonada dentro de un entorno de espacio aislado. Puede realizar varias pruebas según sea necesario para validar la migración, realizar pruebas de aplicaciones y resolver todos los problemas antes de la migración real.

## <a name="will-windows-server-2008-and-2008-r2-be-supported-in-azure-after-migration"></a>¿Windows Server 2008 y 2008 R2 serán compatibles en Azure después de la migración?

Puede migrar los servidores Windows Server 2008 y 2008 R2 a máquinas virtuales de Azure y recibir las Actualizaciones de seguridad extendidas durante tres años a contar desde la finalización del soporte técnico, sin cargos adicionales por encima del costo que implica ejecutar la máquina virtual. Puede usar la herramienta Azure Migrate: Server Migration para migrar las cargas de trabajo de Windows Server 2008 y 2008 R2.

## <a name="is-there-a-rollback-option-for-azure-migrate"></a>¿Hay una opción de reversión para Azure Migrate?

Puede usar la opción de migración de prueba para validar la funcionalidad y el rendimiento de la aplicación en Azure. Puede realizar el número de migraciones de prueba que quiera y ejecutar la migración final después de establecer un valor de confianza a través de la operación de migración de prueba. Una migración de prueba no afecta a la máquina local, que sigue funcionando y continúa las replicaciones hasta que se realiza la migración real. Si se produjeron errores durante la pruebas de aceptación de usuario de la migración de prueba, puede optar por posponer la migración final y mantener la máquina virtual o servidor de origen en ejecución y realizando replicaciones en Azure. Puede volver a intentar la migración final una vez que haya resuelto los errores.  
Nota: Una vez que haya realizado una migración final a Azure y que la máquina de origen local se haya apagado, no podrá realizar una reversión de Azure al entorno local.

## <a name="can-i-select-the-virtual-network-and-subnet-to-use-for-test-migrations"></a>¿Puedo seleccionar la red virtual y la subred que se usarán para las migraciones de prueba?

Puede seleccionar una red virtual para las migraciones de prueba. La subred se selecciona automáticamente en función de la siguiente lógica:

- Si se especificó una subred de destino (distinta de la predeterminada) como entrada al habilitar la replicación, Azure Migrate prioriza el uso de una subred con el mismo nombre en la red virtual seleccionada para la migración de prueba.
- Si no se encuentra una subred con el mismo nombre, Azure Migrate selecciona la primera subred disponible en orden alfabético que no sea una subred de puerta de enlace, Application Gateway, Firewall o Bastion.

## <a name="why-is-the-test-migration-button-disabled-for-my-server"></a>¿Por qué está deshabilitado el botón de migración de prueba para mi servidor?

El botón Migración de prueba puede estar en el estado deshabilitado en los escenarios siguientes:

- No se puede iniciar una migración de prueba hasta que se haya completado una replicación inicial para la máquina virtual. El botón migración de prueba estará deshabilitado hasta que se complete el proceso de replicación inicial. Puede realizar una migración de prueba una vez que la máquina virtual se encuentre en una fase de sincronización diferencial.
- El botón puede estar deshabilitar si ya se ha completado una migración de prueba, pero no se ha realizado una limpieza de esta para la máquina virtual. Realice una limpieza de la migración de prueba y vuelva a intentar la operación.

## <a name="what-happens-if-i-dont-clean-up-my-test-migration"></a>¿Qué sucede si no se limpia la migración de prueba?

La migración de prueba simula una migración real mediante la creación de una máquina virtual de Azure de prueba con datos replicados. El servidor se implementará con una copia en un momento dado de los datos replicados en el grupo de recursos de destino (que seleccionó al habilitar la replicación) con el sufijo "-test". Las migraciones de prueba están pensadas para validar la funcionalidad del servidor con el fin de minimizar los problemas posteriores a la migración. Si la migración de prueba no se limpia después de la prueba, la máquina virtual de prueba seguirá ejecutándose en Azure y se aplicarán cargos. Para realizar la limpieza de una migración de prueba, diríjase a la vista Replicación de máquinas en la herramienta Server Migration y use la acción "Migración de prueba de limpieza" en la máquina.

## <a name="can-i-migrate-active-directory-domain-controllers-using-azure-migrate"></a>¿Puedo migrar los controladores de dominio de Active Directory mediante Azure Migrate?

La herramienta Server Migration es independiente de la aplicación y funciona para la mayoría de las aplicaciones. Al migrar un servidor mediante dicha herramienta, todas las aplicaciones instaladas en el servidor se migran junto con ella. No obstante, en el caso de algunas aplicaciones, es posible que sean más adecuados otros métodos de migración alternativos distintos de la migración de servidor.  Para Active Directory, en el caso de los entornos híbridos en los que el sitio local está conectado a su entorno de Azure, puede extender el directorio a Azure al agregar controladores de dominio adicionales en Azure y configurar la replicación de Active Directory. Si va a migrar a un entorno aislado en Azure que requiere sus propios controladores de dominio (o va a realizar pruebas de aplicaciones en un entorno de espacio aislado), puede migrar servidores mediante la herramienta de migración de servidor.

## <a name="what-happens-if-i-dont-stop-replication-after-migration"></a>¿Qué ocurre si no se detiene la replicación después de una migración?

Al detener la replicación, la herramienta Azure Migrate: Server Migration limpia los discos administrados de la suscripción que se crearon para la replicación. Si no detiene la replicación después de una migración, se seguirán aplicando cargos por estos discos. Detener la replicación no afectará a los discos conectados a las máquinas que ya se han migrado.

## <a name="do-i-need-vmware-vcenter-to-migrate-vmware-vms"></a>¿Necesito VMware vCenter para migrar máquinas virtuales de VMware?

Para [migrar máquinas virtuales de VMware](server-migrate-overview.md) mediante la migración sin agente o basada en agentes de VMware. vCenter Server debe administrar los hosts de ESXi en los que se encuentran las máquinas virtuales. Si no tiene vCenter Server, puede migrar máquinas virtuales de VMware mediante su migración como servidores físicos. [Más información](migrate-support-matrix-physical-migration.md).

## <a name="can-i-upgrade-my-os-while-migrating"></a>¿Puedo actualizar mi sistema operativo durante la migración?

Azure Migrate: Server Migration solo es compatible actualmente con migraciones directas. La herramienta no admite actualizar la versión del sistema operativo durante la migración. La máquina que se haya migrado tendrá el mismo sistema operativo que la máquina de origen.

## <a name="i-deployed-two-or-more-appliances-to-discover-vms-in-my-vcenter-server-however-when-i-try-to-migrate-the-vms-i-only-see-vms-corresponding-to-one-of-the-appliances"></a>He implementado dos (o más) dispositivos para detectar máquinas virtuales en mi instancia de vCenter Server. Sin embargo, cuando intento migrar las máquinas virtuales, solo veo las máquinas virtuales que corresponden a uno de los dispositivos.

Si hay varios dispositivos configurados, es necesario que no exista ninguna superposición entre las máquinas virtuales en las cuentas de vCenter proporcionadas. No se admiten los escenarios con detecciones con este tipo de superposición.

## <a name="how-do-i-know-if-my-vm-was-successfully-migrated"></a>¿Cómo puedo saber si la máquina virtual se migró correctamente?

Una vez que haya migrado la máquina virtual o el servidor correctamente, podrá ver y administrar la VM desde la página Máquinas virtuales. Conéctese a la máquina virtual migrada para realizar la validación.
También puede revisar el "Estado del trabajo" de la operación con el fin de comprobar si la migración se completó correctamente. Si ve algún error, resuélvalo y vuelva a intentar la operación de migración.

## <a name="can-i-consolidate-multiple-source-vms-into-one-vm-while-migrating"></a>¿Puedo consolidar varias máquinas virtuales de origen en una sola máquina virtual durante la migración?

Actualmente, las funciones de migración de servidores de Azure Migrate admiten las migraciones directas. No se admite la consolidación de los servidores ni la actualización del sistema operativo como parte de la migración. 

## <a name="how-does-agentless-replication-affect-vmware-servers"></a>¿Cómo afecta la replicación sin agente a los servidores de VMware?

La replicación sin agente produce un impacto en el rendimiento de los hosts de VMware vCenter Server y VMware ESXi. Dado que la replicación sin agente utiliza instantáneas, consume IOPS en el almacenamiento, por lo que se necesita algo de ancho de banda de almacenamiento de IOPS. No se recomienda usar la replicación sin agente si hay restricciones en el almacenamiento o en el número de IOPS en el entorno.

## <a name="can-i-do-agentless-migration-of-uefi-vms-to-azure-gen-2"></a>¿Puedo realizar la migración sin agente de VM UEFI a Azure Gen 2?

No. Puede usar la [migración basada en agente de VMware](https://docs.microsoft.com/azure/migrate/tutorial-migrate-vmware-agent), la [migración de Hyper-V](https://docs.microsoft.com/azure/migrate/tutorial-migrate-physical-virtual-machines) o las opciones de [migración de servidores físicos](https://docs.microsoft.com/azure/migrate/tutorial-migrate-physical-virtual-machines) para migrar estas máquinas virtuales a las máquinas virtuales de Azure Gen 2.

***Nota:*** Asegúrese de seleccionar el tamaño de máquina virtual adecuado que sea compatible con UEFI Generación 2 en Azure.

## <a name="next-steps"></a>Pasos siguientes

Consulte la sección [Información general de Azure Migrate](migrate-services-overview.md).
