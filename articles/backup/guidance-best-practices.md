---
title: Guía y procedimientos recomendados
description: Descubra los procedimientos recomendados y las instrucciones para realizar copias de seguridad en la nube de las cargas de trabajo locales y en la nube.
ms.topic: conceptual
ms.date: 07/22/2020
ms.openlocfilehash: db6eec5351a9015b136226610d2bb3deb8bdc651
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89000369"
---
# <a name="backup-cloud-and-on-premises-workloads-to-cloud"></a>Copia de seguridad en la nube de cargas de trabajo locales y en la nube

## <a name="introduction"></a>Introducción

Azure Backup protege todos sus recursos de datos de Azure por medio de una solución sencilla, segura y rentable que no precisa ninguna infraestructura. Se trata de una solución de protección de datos integrada de Azure válida para una amplia variedad de cargas de trabajo. Ayuda a proteger las cargas de trabajo críticas que se ejecutan en la nube y garantiza que las copias de seguridad estén siempre disponibles y se administren a gran escala en todos los recursos de copia de seguridad.

### <a name="intended-audience"></a>Destinatarios

Este artículo está principalmente destinado a los administradores de TI y aplicaciones, así como a los implementadores de organizaciones grandes y medianas, que quieren obtener información sobre las funcionalidades de la tecnología de protección de datos integrada de Azure, Azure Backup, y saber cómo poner en práctica de forma eficaz soluciones que protejan mejor sus implementaciones. En este artículo se da por supuesto que está familiarizado con las tecnologías básicas de Azure y con los conceptos de protección de datos, y que cuenta con experiencia trabajando con una solución de copia de seguridad. Las instrucciones que se describen en este artículo pueden facilitar el diseño de la solución de copia de seguridad en Azure mediante patrones establecidos y evitar problemas conocidos.

### <a name="how-this-article-is-organized"></a>Organización de este artículo

Aunque empezar a proteger la infraestructura y las aplicaciones en Azure resulta fácil, si se asegura de que los recursos subyacentes de Azure están configurados correctamente y se utilizan de forma óptima, también puede reducir el tiempo necesario para hacerlo. En este artículo se ofrece una breve introducción a las consideraciones sobre diseño y las instrucciones para configurar de forma óptima la implementación de Azure Backup. Se examinan los componentes (por ejemplo, el almacén de Recovery Services, la directiva de Backup) y los conceptos (por ejemplo, la gobernanza) esenciales, cómo concebirlos y cómo concebir sus funcionalidades, con vínculos a la documentación detallada del producto.

## <a name="architecture"></a>Architecture

![Arquitectura de Azure Backup](./media/guidance-best-practices/azure-backup-architecture.png)

### <a name="workloads"></a>Cargas de trabajo

Azure Backup permite proteger los datos de diversas cargas de trabajo (locales y en la nube). Es un mecanismo de protección de datos integrado, seguro y confiable de Azure. Puede escalar sin problemas su protección en múltiples cargas de trabajo sin que esto suponga ninguna sobrecarga de administración. También existen varios canales de automatización para conseguirlo (a través de PowerShell, la CLI, las plantillas de Azure Resource Manager y las API REST).

* **Almacenamiento escalable, duradero y seguro**: Azure Backup usa almacenamiento de blobs confiable con características de alta disponibilidad y seguridad integradas. Puede elegir almacenamiento LRS, GRS o RA-GRS para los datos de copia de seguridad.  

* **Integración nativa de cargas de trabajo**: Azure Backup proporciona integración nativa con las cargas de trabajo de Azure (máquinas virtuales, SAP HANA, SQL en máquinas virtuales de Azure e incluso Azure Files) sin necesidad de administrar la automatización o la infraestructura para implementar agentes, escribir nuevos scripts o aprovisionar almacenamiento.

### <a name="data-plane"></a>Plano de datos

* **Administración automatizada de almacenamiento**: Azure Backup automatiza el aprovisionamiento y la administración de cuentas de almacenamiento para los datos de copia de seguridad a fin de garantizar que se escale a medida que estos datos crecen.

* **Protección contra eliminación malintencionada**: la eliminación temporal de las copias de seguridad protege contra cualquier intento accidental o malintencionado de eliminar las copias de seguridad. Los datos de copia de seguridad eliminados se almacenan durante 14 días de forma gratuita y se pueden recuperar desde ese estado.

* **Copias de seguridad cifradas seguras**: Azure Backup garantiza que los datos de copia de seguridad se almacenan de forma segura, aprovechando las funcionalidades de seguridad integradas de la plataforma Azure, como el control de acceso basado en roles (RBAC) y el cifrado.

* **Administración del ciclo de vida de los datos de copia de seguridad**: Azure Backup limpia automáticamente los datos de copia de seguridad antiguos para cumplir las directivas de retención. También puede organizar los datos desde un almacenamiento operativo hasta la opción de almacén.

### <a name="management-plane"></a>Plano de administración

* **Control de acceso**: el almacén de Recovery Services proporciona las funcionalidades de administración y es accesible a través de Azure Portal, el SDK, la CLI e incluso las API REST. También supone un límite para RBAC, lo que le ofrece la opción de restringir el acceso a las copias de seguridad solo a los administradores de copias de seguridad autorizados.

* **Administración de directivas**: las directivas de Azure Backup dentro de cada almacén definen cuándo se deben desencadenar las copias de seguridad y cuánto tiempo deben conservarse. También puede administrar estas directivas y aplicarlas en varios elementos.

* **Supervisión e informes**: Azure Backup se integra con Log Analytics y proporciona la capacidad de ver informes también a través de libros.

* **Administración de instantáneas**: Azure Backup toma instantáneas de algunas cargas de trabajo nativas de Azure (máquinas virtuales y Azure Files), administra estas instantáneas y permite restaurarlas rápidamente. Esta opción reduce drásticamente el tiempo de recuperación de los datos en el almacenamiento original.

## <a name="vault-considerations"></a>Consideraciones sobre el almacén

Azure Backup usa almacenes de Recovery Services para organizar y administrar las copias de seguridad, y para almacenar los datos con copia de seguridad realizada. Un diseño de almacén eficaz ayuda a las organizaciones a establecer una estructura para organizar y administrar los recursos de copia de seguridad de Azure que se adapte a sus prioridades empresariales. A la hora de crear un almacén, tenga en cuenta las siguientes directrices:  

### <a name="align-to-subscription-design-strategy"></a>Alineación con la estrategia de diseño de suscripciones

Dado que el almacén tienen como ámbito una suscripción, ajuste el diseño del almacén para que cumpla con la estrategia de diseño de la suscripción como, por ejemplo, la *estrategia de categoría de aplicación*, en la que las suscripciones se separan en función de aplicaciones o servicios específicos o en arquetipos de aplicación. Para más información, consulte este [artículo](/azure/cloud-adoption-framework/decision-guides/subscriptions/).

### <a name="single-or-multiple-vault"></a>Uno o varios almacenes

Puede usar un solo almacén o varios almacenes para organizar y administrar la copia de seguridad. Tenga en cuenta las directrices siguientes:

* Si todas las cargas de trabajo se administran mediante una sola suscripción y un único recurso, puede usar un solo almacén para supervisar y administrar su copia de seguridad.

* Si las cargas de trabajo se distribuyen entre las suscripciones, puede crear varios almacenes, uno o varios por suscripción.
  * Para simplificar la supervisión de las actividades operativas en todos los almacenes, suscripciones e inquilinos, puede utilizar el Explorador de Backup e informes. [Consulte más información aquí](monitor-azure-backup-with-backup-explorer.md) para obtener una vista agregada.
  * Si necesita una directiva coherente entre almacenes, puede usar Azure Policy para propagar la directiva de copia de seguridad en varios almacenes. Puede escribir una [definición de Azure Policy](../governance/policy/concepts/definition-structure.md) personalizada que use el efecto ["deployIfNotExists"](../governance/policy/concepts/effects.md#deployifnotexists) para propagar una directiva de copia de seguridad en varios almacenes. Después puede [asignar](../governance/policy/assign-policy-portal.md) esta definición de Azure Policy a un ámbito determinado (suscripción o grupo de recursos), para que se implemente un recurso de "directiva de copia de seguridad" en todos los almacenes de Recovery Services incluidos en el ámbito de la asignación de Azure Policy. El usuario debe especificar la configuración de la directiva de copia de seguridad (como la frecuencia de copia de seguridad, la retención, etc.) como parámetros de la asignación de Azure Policy.

* A medida que crezca la superficie de la organización, es posible que desee trasladar las cargas de trabajo entre las suscripciones por las siguientes razones: alinear por directiva de copia de seguridad, consolidar almacenes, aceptar una menor redundancia para ahorrar costos (pasar de GRS a LRS).  Azure Backup admite el traslado de un almacén de Recovery Services entre suscripciones de Azure o a otro grupo de recursos dentro de la misma suscripción. [Obtenga más información aquí](backup-azure-move-recovery-services-vault.md).

### <a name="review-default-settings"></a>Revisión de la configuración predeterminada

Revise los valores predeterminados del tipo de replicación de almacenamiento y la configuración de seguridad de modo que cumplan sus requisitos antes de configurar copias de seguridad en el almacén.

* El *tipo de replicación de almacenamiento* se establece de forma predeterminada en Almacenamiento con redundancia geográfica (GRS). Una vez que configure la copia de seguridad, se deshabilitará la opción para modificar. [Siga estos pasos](backup-create-rs-vault.md#set-storage-redundancy) para revisar y modificar la configuración.

* La *eliminación temporal* está habilitada de forma predeterminada en los almacenes recién creados para proteger los datos de copia de seguridad contra eliminaciones accidentales o malintencionadas. [Siga estos pasos](backup-azure-security-feature-cloud.md#enabling-and-disabling-soft-delete) para revisar y modificar la configuración.

* La *restauración entre regiones* permite restaurar máquinas virtuales de Azure en una región secundaria, que es una región emparejada de Azure. Esta opción permite llevar a cabo maniobras para cumplir con los requisitos de auditoría o cumplimiento y para restaurar la máquina virtual o su disco si se produce un desastre en la región primaria. Se trata de una característica opcional en cualquier almacén con redundancia geográfica. [Obtenga más información aquí](backup-create-rs-vault.md#set-cross-region-restore).

* Antes de finalizar el diseño del almacén, revise las [matrices de compatibilidad de los almacenes](backup-support-matrix.md#vault-support) para conocer los factores que pueden influir en las opciones de diseño o limitarlas.

## <a name="backup-policy-considerations"></a>Consideraciones sobre Backup Policy

Una directiva de Azure Backup tiene dos componentes: *programación* (cuándo realizar la copia de seguridad) y *retención* (cuánto tiempo se conserva la copia de seguridad). Puede definir la directiva en función del tipo de datos del que se hará la copia de seguridad, los requisitos de RTO/RPO, las necesidades de cumplimiento operativo o normativo y el tipo de carga de trabajo (por ejemplo, máquina virtual, base de datos, archivos). [Obtenga más información aquí](backup-architecture.md#backup-policy-essentials).

Al crear una directiva de copia de seguridad, tenga en cuenta las siguientes directrices:

### <a name="schedule-considerations"></a>Consideraciones sobre la programación

* Considere la posibilidad de agrupar las máquinas virtuales que requieren la misma configuración de hora de inicio de programación, frecuencia y retención en una única directiva.

* Asegúrese de que la hora de inicio programada de la copia de seguridad se sitúa fuera de las horas punta de aplicación de producción.

* Para distribuir el tráfico de copia de seguridad, puede hacer la copia de seguridad de diferentes máquinas virtuales en distintos momentos del día, asegurándose de que los horarios no se superpongan.

### <a name="retention-considerations"></a>Consideraciones sobre la retención

* La retención a corto plazo puede ser de minutos o diaria. La retención de los puntos de copia de seguridad semanales, mensuales o anuales se conoce como retención a largo plazo.

* Retención a largo plazo:
  * Planeada (requisitos de cumplimiento): si sabe por adelantado que los datos serán necesarios durante años desde el momento actual, use la retención a largo plazo.
  * No planeada (requisito a petición): si no lo sabe de antemano, puede usar la opción a petición con una configuración de retención personalizada específica (esta configuración de retención personalizada no se ve afectada por la configuración de la directiva).

* Copia de seguridad a petición con retención personalizada: si necesita realizar una copia de seguridad no programada mediante una directiva de copia de seguridad, puede usar una copia de seguridad a petición. Esto puede ser útil para realizar copias de seguridad que no se ajustan a la copia de seguridad programada o para realizar copias de seguridad pormenorizadas (por ejemplo, varias copias de seguridad de máquinas virtuales de IaaS por día, ya que la copia de seguridad programada solo permite una ejecución al día). Es importante tener en cuenta que la directiva de retención definida en la directiva programada no se aplica a las copias de seguridad a petición.

### <a name="optimize-backup-policy"></a>Optimización de la directiva de copia de seguridad

* A medida que cambien los requisitos de su empresa, es posible que necesite ampliar o reducir la duración de la retención. Al hacerlo, puede contar con que sucederá lo siguiente:  
  * Si se amplía la retención, los puntos de recuperación existentes se marcarán para mantenerlos de acuerdo con la nueva directiva.
  * Si se reduce la retención, se marcarán para eliminarse y, posteriormente, se eliminarán en el siguiente trabajo de limpieza.
  * En todos los puntos de retención (excepto los puntos a petición) se aplicarán las reglas de retención más recientes. Por lo tanto, si se amplía el período de retención (por ejemplo, a cien días), cuando se realice la copia de seguridad seguida de una reducción de retención (por ejemplo, de cien a siete días), todos los datos de copia de seguridad se conservarán según el último período de retención especificado (es decir, siete días).

* Azure Backup proporciona la flexibilidad necesaria para *detener la protección y administrar las copias de seguridad*:
  * *Detener la protección y conservar los datos de copia de seguridad*. Si está retirando o dando de baja el origen de datos (máquina virtual, aplicación), pero necesita conservar los datos con fines de auditoría o cumplimiento, puede usar esta opción para detener todos los trabajos de copia de seguridad futuros de protección del origen de datos y conservar los puntos de recuperación de los que se ha realizado una copia de seguridad. Después, podrá restaurar o reanudar la protección de la máquina virtual.
  * *Detener la protección y eliminar los datos de copia de seguridad*. Esta opción hará que todos los trabajos de copia de seguridad futuros dejen de proteger la máquina virtual y eliminará todos los puntos de recuperación. No podrá restaurar la máquina virtual ni usar la opción Reanudar copia de seguridad.

  * Si reanuda la protección (de un origen de datos que se ha detenido con retención de datos), se aplicarán las reglas de retención. Los puntos de recuperación expirados se quitarán (a la hora programada).

* Antes de completar el diseño de la directiva, es importante tener en cuenta los siguientes factores que pueden influir en las opciones de diseño.
  * Una directiva de copia de seguridad tiene como ámbito un almacén.
  * Existe un límite en cuanto al número de elementos por directiva (por ejemplo, 100 máquinas virtuales). Para escalar, puede crear directivas duplicadas con la misma programación o con distintas programaciones.
  * No se pueden eliminar de forma selectiva puntos de recuperación específicos.
  * No se puede deshabilitar completamente la copia de seguridad programada y mantener el origen de datos en un estado protegido. La mínima frecuencia de copia de seguridad que puede configurar con la directiva es una copia de seguridad semanal programada. Como alternativa, podría detener la protección con la opción de retención de datos y habilitar la protección cada vez que desee realizar una copia de seguridad, realizar una copia de seguridad a petición y, a continuación, desactivar la protección, pero conservar los datos de copia de seguridad. [Obtenga más información aquí](backup-azure-manage-vms.md#stop-protecting-a-vm).

## <a name="security-considerations"></a>Consideraciones sobre la seguridad

Con el fin de ayudarle a proteger los datos de copia de seguridad y satisfacer las necesidades de seguridad de su negocio, Azure Backup ofrece garantías de confidencialidad, integridad y disponibilidad frente a ataques deliberados y abuso de sus valiosos datos y sistemas. Tenga en cuenta las siguientes directrices de seguridad para la solución Azure Backup:

### <a name="authentication-and-authorization"></a>Autenticación y autorización

* El control de acceso basado en rol de Azure (Azure RBAC) permite realizar una administración detallada del acceso, segregar tareas dentro del equipo y conceder solo la cantidad de acceso que los usuarios necesitan para realizar su trabajo. [Obtenga más información aquí](backup-rbac-rs-vault.md).

* Azure Backup proporciona tres roles integrados para controlar las operaciones de administración de copia de seguridad: colaboradores, operadores y lectores de copias de seguridad. [Obtenga más información aquí](backup-rbac-rs-vault.md#mapping-backup-built-in-roles-to-backup-management-actions).

* Azure Backup tiene varios controles de seguridad integrados en el servicio para evitar, detectar y responder a los puntos vulnerables de seguridad (obtenga más información).

* Las cuentas de almacenamiento empleadas por los almacenes de Recovery Services están aisladas, y los usuarios no pueden acceder a ellas con fines malintencionados. Solo se permite el acceso mediante operaciones de administración de Azure Backup, como la restauración.

### <a name="encryption-of-data-in-transit-and-at-rest"></a>Cifrado de datos en tránsito y en reposo

Mediante el cifrado, se protegen los datos y es más fácil cumplir los compromisos de cumplimiento y seguridad de la organización.

* Dentro de Azure, los datos en tránsito entre Azure Storage y el almacén se protegen mediante HTTPS. Estos datos permanecen dentro de la red de Azure.

* Los datos de copia de seguridad se cifran automáticamente mediante claves administradas por Microsoft. Como alternativa, puede usar sus propias claves, también conocidas como [claves administradas por el cliente](encryption-at-rest-with-cmk.md).

* Azure Backup admite la copia de seguridad y la restauración de máquinas virtuales de Azure que tengan cifrados sus discos del sistema operativo o de datos con Azure Disk Encryption (ADE). [Obtenga más información aquí](backup-azure-vms-encryption.md).

### <a name="protection-of-backup-data-from-unintentional-deletes"></a>Protección de los datos de copia de seguridad ante eliminaciones accidentales

Azure Backup proporciona características de seguridad que ayudan a proteger los datos de copia de seguridad incluso después de su eliminación. Con la eliminación temporal, si un usuario elimina la copia de seguridad (de una máquina virtual, base de datos SQL Server, recurso compartido de Azure, base de datos SAP HANA), sus datos se conservan durante 14 días más, lo que permite la recuperación de ese elemento de copia de seguridad sin pérdida de datos. Esta retención adicional de 14 días de los datos de copia de seguridad en el estado de "eliminación temporal" no le supone ningún costo. [Obtenga más información aquí](backup-azure-security-feature-cloud.md).

### <a name="monitoring-and-alerts-of-suspicious-activity"></a>Supervisión y alertas por actividad sospechosa

Azure Backup proporciona funcionalidades integradas de supervisión y creación de alertas para ver y configurar acciones en eventos relacionados con Azure Backup. [Obtenga más información aquí](security-overview.md#monitoring-and-alerts-of-suspicious-activity).

### <a name="security-features-to-help-protect-hybrid-backups"></a>Características de seguridad que ayudan a proteger las copias de seguridad híbridas

El servicio Azure Backup usa el agente de Microsoft Azure Recovery Services (MARS) para crear copias de seguridad y restaurar archivos y carpetas, así como el estado del volumen o el sistema desde un equipo local en Azure. Ahora MARS incluye características de seguridad: una frase de contraseña para cifrar antes de la carga y descifrar después de la descarga desde Azure Backup, los datos de copia de seguridad eliminados se conservan durante 14 días más a partir de la fecha de eliminación, y las operaciones críticas (por ejemplo, cambiar una frase de contraseña) solo las pueden realizar los usuarios que tengan credenciales de Azure válidas. [Obtenga más información aquí](backup-azure-security-feature.md).

## <a name="network-considerations"></a>Consideraciones sobre la red

Azure Backup requiere el traslado de los datos de la carga de trabajo al almacén de Recovery Services. Azure Backup proporciona varias funcionalidades para impedir que los datos de copia de seguridad se expongan accidentalmente (por ejemplo, un ataque de tipo "Man in the Middle" en la red). Tenga en cuenta las directrices siguientes:

### <a name="internet-connectivity"></a>Conectividad de Internet

* *Copia de seguridad de máquinas virtuales de Azure*: todas las comunicaciones y transferencias de datos necesarias entre el almacenamiento y Azure Backup se producen dentro de la red de Azure sin necesidad de acceder a la red virtual. Por lo tanto, la copia de seguridad de las máquinas virtuales de Azure colocadas dentro de redes protegidas no requiere que permita el acceso a ninguna dirección IP ni nombre de dominio completo.

* *Bases de datos SAP HANA y bases de datos SQL Server que se ejecutan en máquinas virtuales de Azure*: necesitan conectividad con el servicio Azure Backup, Azure Storage y Azure Active Directory. Esto puede lograrse mediante puntos de conexión privados o si se permite el acceso a las direcciones IP públicas o FQDN necesarios. No permitir la conectividad adecuada con los servicios de Azure necesarios puede provocar errores en operaciones como la detección de bases de datos, la configuración de copias de seguridad, la realización de copias de seguridad y la restauración de datos. Para obtener una guía de red completa cuando use etiquetas de NSG, Azure Firewall y un proxy HTTP, consulte estos artículos sobre [SQL](backup-sql-server-database-azure-vms.md#establish-network-connectivity) y [SAP HANA](./backup-azure-sap-hana-database.md#establish-network-connectivity).

* *Híbrida*: el agente MARS (Microsoft Azure Recovery Services) requiere acceso a la red para todas las operaciones críticas: instalación, configuración, copia de seguridad y restauración. El agente MARS puede conectarse al servicio Azure Backup a través de [Azure ExpressRoute](install-mars-agent.md#use-azure-expressroute) mediante el uso del emparejamiento público (disponible para los circuitos anteriores) y el emparejamiento de Microsoft, con [puntos de conexión privados](install-mars-agent.md#private-endpoints) o a través de [proxy o firewall con los controles de acceso adecuados](install-mars-agent.md#verify-internet-access).

### <a name="private-endpoints-for-azure-backup"></a>Puntos de conexión privados para Azure Backup

Un [punto de conexión privado](../private-link/private-endpoint-overview.md) de Azure es una interfaz de red que le conecta de forma privada y segura a un servicio con la tecnología de Azure Private Link. Azure Backup permite realizar copias de seguridad de los datos y restaurarlos de forma segura desde los almacenes de Recovery Services mediante puntos de conexión privados.

* Cuando habilita puntos de conexión privados para el almacén, solo se usan para la copia de seguridad y restauración de las cargas de trabajo de SQL y SAP HANA en una máquina virtual de Azure y en las copias de seguridad del agente MARS.  También puede usar el almacén para realizar copias de seguridad de otras cargas de trabajo (si bien no requerirán los puntos de conexión privados). Además de la copia de seguridad de las cargas de trabajo de SQL y SAP HANA y de las copias de seguridad mediante el agente MARS, los puntos de conexión privados también se usan para realizar la recuperación de archivos en el caso de la copia de seguridad de máquinas virtuales de Azure. [Obtenga más información aquí](private-endpoints.md#recommended-and-supported-scenarios).

* Actualmente, Azure Active Directory no admite puntos de conexión privados. Por tanto, las direcciones IP y los nombres de dominio completos que se necesiten en Azure Active Directory tendrán que disponer de acceso de salida desde la red protegida cuando se realicen copias de seguridad de bases de datos en máquinas virtuales de Azure y copias de seguridad mediante el agente MARS. También puede usar etiquetas NSG y etiquetas de Azure Firewall para permitir el acceso a Azure AD, según corresponda. [Obtenga más información sobre los requisitos previos aquí](./private-endpoints.md#before-you-start).

## <a name="governance-considerations"></a>Consideraciones de gobernanza

La gobernanza en Azure se implementa principalmente con [Azure Policy](../governance/policy/overview.md) y [Azure Cost Management](../cost-management-billing/cost-management-billing-overview.md). [Azure Policy](../governance/policy/overview.md) permite crear, asignar y administrar definiciones de directivas para aplicar las reglas en los recursos. Con esta característica, esos recursos se mantienen en cumplimiento con los estándares corporativos. [Azure Cost Management](../cost-management-billing/cost-management-billing-overview.md) permite realizar un seguimiento del uso y de los gastos de la nube de los recursos no solo de Azure, sino también de otros proveedores de servicios en la nube. Además, herramientas como la [Calculadora de precios de Azure](https://azure.microsoft.com/pricing/calculator/) y [Azure Advisor](../advisor/advisor-overview.md) desempeñan un papel importante en el proceso de administración de costos.

### <a name="azure-backup-support-two-key-scenarios-via-built-in-azure-policy"></a>Azure Backup admite dos escenarios clave mediante directivas integradas de Azure Policy

* Asegúrese de que se realiza automáticamente una copia de seguridad con la configuración de retención correcta de las máquinas críticas para la empresa recién creadas. Azure Backup proporciona una directiva integrada (mediante Azure Policy) que se puede asignar a todas las máquinas virtuales de Azure de una ubicación especificada dentro de una suscripción o un grupo de recursos. Cuando esta directiva se asigna a un ámbito determinado, todas las máquinas virtuales nuevas creadas en ese ámbito se configuran automáticamente para que se cree su copia de seguridad en un almacén existente en la misma ubicación y suscripción. El usuario puede especificar el almacén y la directiva de retención a los que se deben asociar las máquinas virtuales de las que se ha creado una copia de seguridad. [Obtenga más información aquí](backup-azure-auto-enable-backup.md).

* Asegúrese de que los almacenes recién creados tienen habilitado el diagnóstico para admitir informes. A menudo, agregar manualmente una configuración de diagnóstico por almacén puede ser una tarea complicada. Además, cualquier nuevo almacén creado debe tener habilitada la configuración de diagnóstico para poder ver los informes correspondientes. Para simplificar la creación de la configuración de diagnóstico a gran escala (con Log Analytics como destino), Azure Backup dispone de una directiva de Azure Policy integrada. Esta directiva agrega una configuración de diagnóstico de Log Analytics a todos los almacenes de cada suscripción o grupo de recursos. En las secciones siguientes se proporcionan instrucciones para usar esta directiva. [Obtenga más información aquí](azure-policy-configure-diagnostics.md).

### <a name="azure-backup-cost-considerations"></a>Consideraciones sobre los costos de Azure Backup

Las funcionalidades del servicio Azure Backup ofrecen la flexibilidad necesaria para administrar de forma eficaz los costos y seguir cumpliendo los requisitos empresariales de continuidad empresarial y recuperación ante desastres (BCDR). Tenga en cuenta las directrices siguientes:

* Use la calculadora de precios para evaluar y optimizar el costo mediante el ajuste de varios parámetros. [Más información aquí](azure-backup-pricing.md).

* Explorador de Backup: use Explorador de Backup o Informes de Backup para comprender y optimizar el crecimiento del almacenamiento de copia de seguridad y detener las copias de seguridad de cargas de trabajo no críticas o máquinas virtuales eliminadas. Puede obtener una vista agregada de todos sus recursos desde una perspectiva de copia de seguridad. Esto incluye no solo la información de los elementos de copia de seguridad, sino también de los recursos que no están configurados para la copia de seguridad. Así se asegura de que nunca deja de proteger datos críticos en su patrimonio en crecimiento y de que las copias de seguridad están optimizadas para cargas de trabajo no críticas o cargas de trabajo eliminadas.

* Optimización de la directiva de copia de seguridad
  * Optimice la configuración de programación y retención en función de arquetipos de carga de trabajo (por ejemplo, misión crítica y no crítica).
  * Optimice la configuración de retención para la restauración instantánea.
  * Elija el tipo de copia de seguridad adecuado para cumplir los requisitos, teniendo a la vez en cuenta los tipos de copia de seguridad admitidos (completa, incremental, de registro, diferencial) por la carga de trabajo en Azure Backup.

* Realice la copia de seguridad de los discos de forma selectiva: la característica en versión preliminar de exclusión de disco ofrece una manera eficaz y rentable de hacer copias de seguridad de los datos críticos de forma selectiva. Por ejemplo, puede hacer la copia de seguridad de un solo disco si no desea hacerla del resto de los discos conectados a una máquina virtual. Esto también resulta útil si tiene varias soluciones de copia de seguridad, como en el caso de que realice una copia de seguridad de las bases de datos o los datos con una solución de copia de seguridad de carga de trabajo (copia de seguridad de base de datos SQL Server en máquina virtual de Azure) y desee usar la copia de seguridad en el nivel de máquina virtual de Azure para discos seleccionados.

* Azure Backup toma instantáneas de las máquinas virtuales de Azure y las almacena junto con los discos para acelerar la creación de puntos de recuperación y agilizar las operaciones de restauración. Esto se conoce como restauración instantánea. De forma predeterminada, estas instantáneas se conservan durante dos días. Esta característica permite que la operación de restauración que se realiza a partir de ellas reduzca drásticamente los tiempos de restauración, ya que se recorta el tiempo necesario para transformar y copiar los datos de nuevo desde el almacén. Como resultado, verá los costos de almacenamiento que corresponden a las instantáneas tomadas durante este período. [Obtenga más información aquí](backup-instant-restore-capability.md#configure-snapshot-retention).

* El tipo de replicación de almacenamiento del almacén de Azure Backup se establece de forma predeterminada en la opción de almacenamiento con redundancia geográfica (GRS). Esta opción no se puede cambiar después de proteger los elementos. Este almacenamiento proporciona un mayor nivel de durabilidad de los datos que el almacenamiento con redundancia local (LRS), permite usar la restauración entre regiones y cuesta más. Revise las ventajas e inconvenientes entre menores costos y mayor durabilidad de los datos y decida qué es lo mejor para su escenario. [Más información aquí](backup-create-rs-vault.md#set-storage-redundancy).

* Si protege tanto la carga de trabajo que se ejecuta dentro de una máquina virtual como la propia máquina virtual, compruebe si es necesaria esta doble protección.

## <a name="monitoring-and-alerting-considerations"></a>Consideraciones sobre la supervisión y las alertas

Como administrador o usuario de copias de seguridad, debe ser capaz de supervisar todas las soluciones de copia de seguridad y recibir notificaciones sobre escenarios importantes. En esta sección se detallan las funcionalidades de supervisión y notificación que presta el servicio Azure Backup.

### <a name="monitoring"></a>Supervisión

* Azure Backup proporciona **supervisión integrada de trabajos** para operaciones como configurar copias de seguridad, hacerlas, restaurarlas, eliminarlas, etc. Esta supervisión tiene como ámbito el almacén y es ideal para controlar un único almacén. [Obtenga más información aquí](backup-azure-monitoring-built-in-monitor.md#backup-jobs-in-recovery-services-vault).

* Si necesita supervisar las actividades operativas a gran escala, dispone del **Explorador de Backup**, que proporciona una vista agregada de todos los recursos de copia de seguridad, lo que permite realizar análisis detallados y solucionar problemas. Se trata de un libro de Azure Monitor integrado que proporciona una única ubicación central para ayudarle a supervisar las actividades operativas en todos los recursos de copia de seguridad en Azure, lo que abarca inquilinos, ubicaciones, suscripciones, grupos de recursos y almacenes. [Obtenga más información aquí](monitor-azure-backup-with-backup-explorer.md).
  * Úselo para identificar los recursos que no están configurados para la copia de seguridad y asegúrese de que nunca se queda sin la protección de datos críticos de su patrimonio en crecimiento.
  * El panel muestra las actividades operativas de los últimos siete días (máximo). Si necesita conservar estos datos, puede exportarlos como un archivo de Excel.
  * Si es un usuario de Azure Lighthouse, puede ver la información de varios inquilinos, lo que supone una supervisión sin límites.

* Si necesita conservar y ver las actividades operativas a largo plazo, use **informes**. Un requisito común de los administradores de copias de seguridad es que necesitan información detallada sobre las copias de seguridad, en función de datos que abarcan un largo período de tiempo. Entre los casos de uso de esta solución se incluyen:
  * Asignación y previsión del almacenamiento en la nube consumido.
  * Auditoría de copias de seguridad y restauraciones.
  * Identificación de las tendencias clave con diferentes niveles de granularidad.

* Además:
  * Puede enviar datos (por ejemplo, trabajos, directivas, etc.) al **área de trabajo de Log Analytics**. Esto habilitará las características de los registros de Azure Monitor que permiten la correlación de estos datos con otros datos de supervisión recopilados por Azure Monitor, la consolidación de las entradas de registro de varias suscripciones e inquilinos de Azure en una ubicación para su análisis conjunto, y el uso de consultas de registro para realizar análisis complejos y obtener información detallada sobre las entradas del registro. [Obtenga más información aquí](../azure-monitor/platform/activity-log.md#send-to-log-analytics-workspace).
  * Puede enviar datos al centro de eventos para después enviar entradas fuera de Azure, por ejemplo, a un sistema de administración de eventos e información de seguridad (SIEM) de terceros o a otra solución de análisis de registros. [Obtenga más información aquí](../azure-monitor/platform/activity-log.md#send-to-azure-event-hubs).
  * Puede enviar los datos de registro a una cuenta de Azure Storage si quiere conservarlos durante más de 90 días para su auditoría, análisis estático o copia de seguridad. Si solo necesita conservar los eventos durante 90 días o menos, no es necesario configurar el archivado en una cuenta de almacenamiento, ya que los eventos del registro de actividad se conservan en la plataforma de Azure durante ese tiempo. [Más información](../azure-monitor/platform/activity-log.md#send-to--azure-storage).

### <a name="alerting"></a>Alertas

* Las alertas son principalmente una forma de recibir notificaciones para tomar las medidas pertinentes. La sección Alertas de Backup muestra las alertas generadas por el servicio Azure Backup.

* Azure Backup ofrece un mecanismo integrado de notificación por correo electrónico de **alertas** en caso de errores, advertencias y operaciones críticas. Puede especificar direcciones de correo electrónico individuales o listas de distribución para recibir una notificación cuando se genere una alerta. También puede optar por recibir una notificación con cada alerta individual, o bien agruparlas en un resumen cada hora y, luego, recibir la notificación correspondiente.
  * Estas alertas las define el servicio y ofrecen asistencia en escenarios limitados: errores de copia de seguridad y restauración, detención de la protección con retención de datos o detención de la protección con eliminación de datos, etc. [Obtenga más información aquí](backup-azure-monitoring-built-in-monitor.md#alert-scenarios).
  * Si se lleva a cabo una operación de destrucción, como la detención de la protección con eliminación de datos, se generará una alerta y se enviará un correo electrónico a los propietarios de la suscripción, los administradores y los coadministradores, aun cuando las notificaciones **no** estén configuradas en el almacén de Recovery Services.
  * Ciertas cargas de trabajo pueden generar una alta frecuencia de errores (por ejemplo, SQL Server cada 15 minutos). Para evitar la saturación que provocaría el envío excesivo de alertas generadas por cada aparición del error, las alertas se consolidan. [Obtenga más información aquí](backup-azure-monitoring-built-in-monitor.md#consolidated-alerts).
  * Las alertas integradas no se pueden personalizar y se restringen a los correos electrónicos definidos en Azure Portal.

* Si necesita **crear alertas personalizadas** (por ejemplo, alertas de trabajos correctos), use Log Analytics. En Azure Monitor, puede crear sus propias alertas en un área de trabajo de Log Analytics. Las cargas de trabajo híbridas (DPM/MABS) también pueden enviar datos a Log Analytics y usar este servicio para proporcionar alertas comunes entre cargas de trabajo compatibles con Azure Backup.

* También puede recibir notificaciones a través de los **registros de actividad** integrados del almacén de Recovery Services. Sin embargo, estos solo admiten escenarios limitados y no son adecuados para operaciones como la copia de seguridad programada, que se alinea mejor con los registros de recursos que con los registros de actividad. Para obtener más información acerca de estas limitaciones y cómo puede usar el área de trabajo de Log Analytics para la supervisión y la generación de alertas a gran escala para todas las cargas de trabajo protegidas por Azure Backup, consulte este [artículo](backup-azure-monitoring-use-azuremonitor.md#using-log-analytics-to-monitor-at-scale).

## <a name="next-steps"></a>Pasos siguientes

Se recomienda leer los siguientes artículos como puntos de partida para usar Azure Backup:

* [Introducción a Azure Backup](backup-overview.md)
* [Preguntas más frecuentes](backup-azure-backup-faq.md)
