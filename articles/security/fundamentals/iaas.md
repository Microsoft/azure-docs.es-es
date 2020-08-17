---
title: Procedimientos de seguridad recomendados para cargas de trabajo de IaaS en Azure | Documentos de Microsoft
description: " La migración de cargas de trabajo a IaaS de Azure nos brinda la oportunidad de volver a evaluar nuestros diseños "
services: security
documentationcenter: na
author: terrylanfear
manager: rkarlin
editor: TomSh
ms.assetid: 02c5b7d2-a77f-4e7f-9a1e-40247c57e7e2
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/28/2019
ms.author: terrylan
ms.openlocfilehash: 03258bf204491afce4635828b3a33a06886aca2d
ms.sourcegitcommit: cee72954f4467096b01ba287d30074751bcb7ff4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87448412"
---
# <a name="security-best-practices-for-iaas-workloads-in-azure"></a>Procedimientos de seguridad recomendados para cargas de trabajo de IaaS de Azure
En este artículo se describen los procedimientos recomendados de seguridad para máquinas virtuales y sistemas operativos.

Los procedimientos recomendados se basan en un consenso de opinión y son válidos para las funcionalidades y conjuntos de características actuales de la plataforma Azure. Puesto que las opiniones y las tecnologías pueden cambiar con el tiempo, este artículo se actualizará para reflejar dichos cambios.

En la mayoría de los escenarios de IaaS (infraestructura como servicio), las [máquinas virtuales de Azure](/azure/virtual-machines/) son la principal carga de trabajo de las organizaciones que usan la informática en la nube. Esto es evidente en los [escenarios híbridos](https://social.technet.microsoft.com/wiki/contents/articles/18120.hybrid-cloud-infrastructure-design-considerations.aspx), en los que las organizaciones quieran migrar lentamente las cargas de trabajo a la nube. En estos escenarios, siga las [consideraciones generales de seguridad de IaaS](https://social.technet.microsoft.com/wiki/contents/articles/3808.security-considerations-for-infrastructure-as-a-service-iaas.aspx) y aplique los procedimientos recomendados de seguridad a todas las máquinas virtuales.

## <a name="protect-vms-by-using-authentication-and-access-control"></a>Protección de máquinas virtuales mediante la autenticación y el control de acceso
El primer paso para proteger la máquina virtual es garantizar que solo los usuarios autorizados puedan configurar nuevas máquinas virtuales y obtener acceso a ellas.

> [!NOTE]
> Para mejorar la seguridad de las máquinas virtuales Linux en Azure, puede integrarlas con la autenticación de Azure Active Directory (AD). Cuando se usa la [autenticación de Azure AD para máquinas virtuales Linux](/azure/virtual-machines/linux/login-using-aad), se administran y se aplican de manera centralizada las directivas que permiten o deniegan el acceso a las máquinas virtuales.
>
>

**Procedimiento recomendado**: Control de acceso a las máquinas virtuales.   
**Detalles**: Use [directivas de Azure](/azure/azure-policy/azure-policy-introduction) para establecer convenciones para los recursos de la organización y crear directivas personalizadas. Aplique estas directivas a los recursos, como los [grupos de recursos](/azure/azure-resource-manager/resource-group-overview). Las máquinas virtuales que pertenecen a un grupo de recursos heredan sus directivas.

Si su organización tiene varias suscripciones, podría necesitar una manera de administrar el acceso, las directivas y el cumplimiento de esas suscripciones de forma eficaz. Los [grupos de administración de Azure](/azure/azure-resource-manager/management-groups-overview) proporcionan un nivel de ámbito por encima de las suscripciones. Las suscripciones se organizan en grupos de administración (contenedores) y aplican sus condiciones de gobernanza a dichos grupos. Todas las suscripciones dentro de un grupo de administración heredan automáticamente las condiciones que se aplican al grupo. Los grupos de administración proporcionan capacidad de administración de nivel empresarial a gran escala con independencia del tipo de suscripciones que tenga.

**Procedimiento recomendado**: Reducción de la variabilidad en la configuración e implementación de máquinas virtuales.   
**Detalles**: Use plantillas de [Azure Resource Manager](/azure/azure-resource-manager/resource-group-authoring-templates) para reforzar las opciones de implementación y facilitar la comprensión y la realización de inventario de las máquinas virtuales de su entorno.

**Procedimiento recomendado**: Protección del acceso con privilegios.   
**Detalles**: Use un [enfoque de privilegios mínimos](https://technet.microsoft.com/windows-server-docs/identity/ad-ds/plan/security-best-practices/implementing-least-privilege-administrative-models) y roles integrados de Azure para permitir que los usuarios configuren las máquinas virtuales y accedan a ellas.

- [Colaborador de la máquina virtual](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor): Puede administrar máquinas virtuales, pero no la cuenta de almacenamiento o la red virtual a la que están conectadas.
- [Colaborador de la máquina virtual clásica](../../role-based-access-control/built-in-roles.md#classic-virtual-machine-contributor): Puede administrar máquinas virtuales creadas con el modelo de implementación clásica, pero no la cuenta de almacenamiento ni la red virtual a la que están conectadas.
- [Administrador de seguridad](../../role-based-access-control/built-in-roles.md#security-admin): Solo en Security Center: puede ver las directivas de seguridad, los estados de seguridad, editar las directivas de seguridad, ver alertas y recomendaciones, y descartar alertas y recomendaciones.
- [Usuario de DevTest Labs](../../role-based-access-control/built-in-roles.md#devtest-labs-user): puede ver todo el contenido, así como conectar, iniciar, reiniciar y apagar las máquinas virtuales.

Los administradores y coadministradores de la suscripción pueden cambiar esta configuración, convirtiéndose en administradores de todas las máquinas virtuales de una suscripción. Asegúrese de que confía en todos los administradores y coadministradores de la suscripción para iniciar sesión en cualquiera de las máquinas.

> [!NOTE]
> Se recomienda consolidar las máquinas virtuales con el mismo ciclo de vida en el mismo grupo de recursos. Mediante los grupos de recursos, puede implementar, supervisar y acumular los costos para sus recursos.
>
>

Las organizaciones que controlan el acceso a la máquina virtual y la configuración mejoran la seguridad general de la máquina virtual.

## <a name="use-multiple-vms-for-better-availability"></a>Uso de varias máquinas virtuales para mejorar la disponibilidad
Si la máquina virtual ejecuta aplicaciones esenciales que necesitan tener una alta disponibilidad, recomendamos encarecidamente usar varias máquinas virtuales. Para mejorar la disponibilidad, use un [conjunto de disponibilidad](../../virtual-machines/windows/manage-availability.md#configure-multiple-virtual-machines-in-an-availability-set-for-redundancy) o [zonas](../../availability-zones/az-overview.md) de disponibilidad.

Un conjunto de disponibilidad es una agrupación lógica que puede usar en Azure para asegurarse de que los recursos de máquina virtual que coloque en dicho conjunto de disponibilidad estén aislados entre sí cuando se implementen en un centro de datos de Azure. Azure garantiza que las máquinas virtuales colocadas en un conjunto de disponibilidad se ejecuten en varios servidores físicos, grupos de proceso, unidades de almacenamiento y conmutadores de red. Si se produce un error de hardware o software de Azure, solo un subconjunto de las máquinas virtuales se ve afectado y la aplicación sigue estando disponible para los clientes. Los conjuntos de disponibilidad son una funcionalidad fundamental para compilar soluciones en la nube confiables.

## <a name="protect-against-malware"></a>Protección frente a malware
Debe instalar la protección antimalware para ayudar a identificar y eliminar virus, spyware y otro software malintencionado. Puede instalar [Microsoft Antimalware](antimalware.md) o una solución de protección de puntos de conexión de un asociado de Microsoft ([Trend Micro](https://help.deepsecurity.trendmicro.com/Welcome.html), [Broadcom](https://www.broadcom.com/products), [McAfee](https://www.mcafee.com/us/products.aspx), [Windows Defender](https://www.microsoft.com/windows/comprehensive-security) y [System Center Endpoint Protection](/configmgr/protect/deploy-use/endpoint-protection)).

Microsoft Antimalware incluye características como la protección en tiempo real, los análisis programados, la corrección de malware, las actualizaciones de firmas, las actualizaciones del motor, los ejemplos de informes y la colección de eventos de exclusión. En entornos hospedados por separado del entorno de producción, puede usar una extensión de antimalware para ayudar a proteger las máquinas virtuales y los servicios en la nube.

Puede integrar soluciones de asociados y Microsoft Antimalware con [Azure Security Center](../../security-center/index.yml) para facilitar la implementación y la integración de detecciones (alertas e incidentes).

**Procedimiento recomendado**: Instalación de una solución antimalware para protegerse frente a malware.   
**Detalles**: [Instale una solución de asociado de Microsoft o Microsoft Antimalware](../../security-center/security-center-install-endpoint-protection.md)

**Procedimiento recomendado**: Integración de la solución antimalware con Security Center para supervisar el estado de la protección.   
**Detalles**: [Administre problemas de Endpoint Protection con Security Center](../../security-center/security-center-partner-integration.md)

## <a name="manage-your-vm-updates"></a>Administrar las actualizaciones de la máquina virtual
Las máquinas virtuales de Azure, al igual que todas las máquinas virtuales locales, están diseñadas para que las administre el usuario. Azure no inserta las actualizaciones de Windows en ellas. Debe administrar las actualizaciones de la máquina virtual.

**Procedimiento recomendado**: Mantener actualizadas las máquinas virtuales.   
**Detalles**: Use la solución [Update Management](../../automation/update-management/update-mgmt-overview.md) de Azure Automation para administrar las actualizaciones del sistema operativo de los equipos Windows y Linux implementados en Azure, en entornos locales o en otros proveedores de nube. Puede evaluar rápidamente el estado de las actualizaciones disponibles en todos los equipos agente y administrar el proceso de instalación de las actualizaciones necesarias para los servidores.

Los equipos administrados por Update Management usan las siguientes configuraciones para evaluar e implementar actualizaciones:

- Microsoft Monitoring Agent (MMA) para Windows o Linux
- Extensión DSC (configuración de estado deseado) de PowerShell para Linux
- Hybrid Runbook Worker de Automation
- Microsoft Update o Windows Server Update Services (WSUS) para equipos Windows

Si usa Windows Update, deje habilitada la configuración automática de Windows Update.

**Procedimiento recomendado**: Comprobación de que en la implementación las imágenes creadas incluyen las actualizaciones de Windows más recientes.   
**Detalles**: Busque e instale las actualizaciones de Windows como primer paso de cada implementación. Es especialmente importante aplicar esta medida al implementar imágenes que proceden de usted o de su propia biblioteca. Aunque las imágenes de Azure Marketplace se actualizan automáticamente de forma predeterminada, puede producirse un intervalo de tiempo (hasta unas cuantas semanas) después de una versión pública.

**Procedimiento recomendado**: Volver a implementar periódicamente las máquinas virtuales para forzar una nueva versión del sistema operativo.   
**Detalles**: Defina la máquina virtual con una [plantilla de Azure Resource Manager](../../azure-resource-manager/templates/template-syntax.md) para poder implementarla fácilmente. El uso de una plantilla le ofrece una máquina virtual segura y revisada cuando la necesite.

**Procedimiento recomendado**: Aplique rápidamente las actualizaciones de seguridad a las máquinas virtuales.   
**Detalles**: Habilite Azure Security Center (nivel Gratis o Estándar) para [detectar si faltan actualizaciones de seguridad y aplicarlas](../../security-center/security-center-apply-system-updates.md).

**Procedimiento recomendado**: Instalación de las últimas actualizaciones de seguridad.   
**Detalles**: Algunas de las primeras cargas de trabajo que los clientes mueven a Azure son laboratorios y sistemas orientados externamente. Si las máquinas virtuales de Azure hospedan aplicaciones o servicios que deben estar accesibles desde Internet, esté atento a la aplicación de revisiones. Aplique revisiones no solo del sistema operativo. Las vulnerabilidades de aplicaciones de asociados a las que no se han aplicado revisiones también pueden provocar problemas que podrían haberse evitado si hubiera una buena administración de revisiones vigente.

**Procedimiento recomendado**: Implementación y comprobación de una solución de copia de seguridad.   
**Detalles**: Una copia de seguridad se debe realizar de la misma manera que cualquier otra operación. al menos en los sistemas que formen parte del entorno de producción que se extiende a la nube.

Los sistemas de prueba y desarrollo deben seguir estrategias de copia de seguridad que proporcionen funcionalidades de restauración que sean similares a las que los usuarios están acostumbrado en función de su experiencia en entornos locales. Las cargas de trabajo de producción movidas a Azure deben integrarse con las soluciones de copia de seguridad existentes cuando sea posible. O bien, puede usar [Azure Backup](../../backup/backup-azure-vms-first-look-arm.md) para ayudarle a enfrentar los requisitos de copia de seguridad.

Las organizaciones que no aplican directivas de actualización de software están más expuestas a amenazas que aprovechan las vulnerabilidades conocidas, previamente fijas. Para cumplir con las normativas del sector, las empresas tienen que demostrar que son diligentes y que usan los controles adecuados para mejorar la seguridad de sus cargas de trabajo ubicadas en la nube.

Los procedimientos recomendados de actualización de software para los centros de datos tradicionales e IaaS de Azure tienen muchas similitudes. Recomendamos evaluar las directivas de actualización de software actuales que se incluirán en las máquinas virtuales ubicadas en Azure.

## <a name="manage-your-vm-security-posture"></a>Administrar la posición de seguridad de la máquina virtual
Las ciberamenazas están en evolución. Para proteger las máquinas virtuales hace falta una funcionalidad de supervisión que pueda detectar rápidamente las amenazas, evitar el acceso no autorizado a los recursos, desencadenar alertas y reducir los falsos positivos.

Para supervisar la posición de seguridad de sus máquinas virtuales [Windows](../../security-center/security-center-virtual-machine.md) y [Linux VMs](../../security-center/security-center-linux-virtual-machine.md), utilice [Azure Security Center](../../security-center/security-center-intro.md). En Security Center, proteja las máquinas virtuales aprovechando las ventajas de las funcionalidades siguientes:

- Aplicar la configuración de seguridad del sistema operativo con las reglas de configuración recomendadas.
- Identificar y descargar las actualizaciones críticas y de seguridad del sistema que puedan faltar.
- Implementar recomendaciones de protección antimalware del punto de conexión.
- Validar el cifrado del disco.
- Evaluar y corregir las vulnerabilidades.
- Detectar amenazas.

Security Center puede supervisar activamente si hay posibles amenazas, que se mostrarán en alertas de seguridad. Las amenazas correlacionadas se agregan en una única vista denominada incidente de seguridad.

Security Center almacena datos en loa [registros de Azure Monitor](/azure/log-analytics/log-analytics-overview). Los registros de Azure Monitor proporcionan un lenguaje de consulta y un motor de análisis que ofrece información sobre el funcionamiento de las aplicaciones y los recursos. Los datos también se recopilan de [Azure Monitor](../../batch/monitoring-overview.md), de las soluciones de administración y de los agentes instalados en máquinas virtuales locales en la nube o en el entorno local. Esta funcionalidad compartida le ayuda a formarse una imagen completa de su entorno.

Las organizaciones que no aplican una seguridad sólida a sus máquinas virtuales no están informadas de posibles intentos de eludir los controles de seguridad llevados a cabo por usuarios no autorizados.

## <a name="monitor-vm-performance"></a>Supervisar el rendimiento de la máquina virtual
El abuso de los recursos puede ser un problema cuando los procesos de las máquinas virtuales consumen más recursos de los que deberían. Los problemas de rendimiento con una máquina virtual pueden provocar la interrupción del servicio, lo que infringe el principio de seguridad de disponibilidad. Esto es especialmente importante en el caso de las máquinas virtuales que hospedan IIS u otros servidores web, ya que el uso elevado de la CPU o la memoria podría indicar un ataque de denegación de servicio (DoS). Por esta razón, resulta imprescindible supervisar el acceso a las máquinas virtuales no solo de forma reactiva (aunque haya un problema), sino también de forma preventiva, usando como base de referencia un rendimiento medido durante el funcionamiento normal.

Se recomienda el uso de [Azure Monitor](/azure/monitoring-and-diagnostics/monitoring-overview-metrics) para obtener una mayor visibilidad del estado de los recursos. Características de Azure Monitor:

- [Archivos de registro de diagnóstico del recurso](../../azure-monitor/platform/platform-logs-overview.md): supervisa los recursos de la máquina virtual e identifica posibles problemas que podrían poner en peligro la disponibilidad y rendimiento.
- [Extensión de Azure Diagnostics](/azure/azure-monitor/platform/diagnostics-extension-overview): proporciona funcionalidades de supervisión y diagnóstico en máquinas virtuales Windows. Para habilitar estas funcionalidades, incluya la extensión como parte de la [plantilla de Azure Resource Manager](/azure/virtual-machines/windows/extensions-diagnostics-template).

Las organizaciones que no supervisan el rendimiento de la máquina virtual no pueden determinar si ciertos cambios en los patrones de rendimiento son normales o anómalos. Una máquina virtual que consume más recursos de lo habitual podría indicar un ataque procedente de un recurso externo o un proceso en peligro que se está ejecutando en la máquina virtual.

## <a name="encrypt-your-virtual-hard-disk-files"></a>Cifrado de los archivos de disco duro virtual
Se recomienda cifrar los discos duros virtuales (VHD) para ayudar a proteger el volumen de arranque y los volúmenes de datos en reposo en el almacenamiento, junto con las claves de cifrado y los secretos.

[Azure Disk Encryption](../azure-security-disk-encryption-overview.md) le ayuda a cifrar discos de las máquinas virtuales IaaS con Windows y Linux. Azure Disk Encryption usa la característica [BitLocker](https://technet.microsoft.com/library/cc732774.aspx) estándar del sector de Windows y la característica [DM-Crypt](https://en.wikipedia.org/wiki/Dm-crypt) de Linux para ofrecer cifrado de volumen para el sistema operativo y los discos de datos. La solución se integra con [Azure Key Vault](https://azure.microsoft.com/documentation/services/key-vault/) para ayudarle a controlar y administrar los secretos y las claves de cifrado de discos en su suscripción de Key Vault. La solución también garantiza que todos los datos de los discos de máquinas virtuales se cifran en reposo en Azure Storage.

Estos son algunos procedimientos recomendados para usar Azure Disk Encryption:

**Procedimiento recomendado**: Habilitar cifrado en las máquinas virtuales.   
**Detalles**: Azure Disk Encryption genera y escribe las claves de cifrado en el almacén de claves. La administración de claves de cifrado en el almacén de claves necesita la autenticación de Azure AD. Para ello, cree una aplicación de Azure AD. Para realizar la autenticación, se pueden usar la autenticación basada en secreto de cliente o la [autenticación de Azure AD basada en certificado del cliente](../../active-directory/authentication/active-directory-certificate-based-authentication-get-started.md).

**Procedimiento recomendado**: Uso de una clave de cifrado de claves (KEK) para una brindar una capa adicional de seguridad para las claves de cifrado. Agregue una KEK al almacén de claves.   
**Detalles**: Use el cmdlet [Add-AzKeyVaultKey](/powershell/module/az.keyvault/add-azkeyvaultkey) para crear una clave de cifrado de claves en el almacén de claves. También puede importar una KEK en el módulo de seguridad de hardware (HSM) de administración de claves local. Para más información, consulte la [documentación de Key Vault](../../key-vault/keys/hsm-protected-keys.md). Cuando se especifica una clave de cifrado de claves, Azure Disk Encryption usa esa clave para encapsular los secretos de cifrado antes de escribirlos en Key Vault. El mantenimiento de una copia de custodia de esta clave en un HSM de administración de claves local ofrece una protección adicional contra la eliminación accidental de claves.

**Procedimiento recomendado**: Tomar una [instantánea](../../virtual-machines/windows/snapshot-copy-managed-disk.md) o realizar una copia de seguridad antes de cifrar los discos. Las copias de seguridad proporcionan una opción de recuperación si se produce un error inesperado durante el cifrado.   
**Detalles**: Las máquinas virtuales con discos administrados requieren una copia de seguridad antes del cifrado. Después de hacer la copia de seguridad, puede usar el cmdlet **Set-AzVMDiskEncryptionExtension** para cifrar los discos administrados mediante la especificación del parámetro *-skipVmBackup*. Para más información sobre cómo realizar la copia de seguridad y restauración de máquinas virtuales cifradas, consulte el artículo sobre [Azure Backup](../../backup/backup-azure-vms-encryption.md).

**Procedimiento recomendado**: Para garantizar que los secretos de cifrado no traspasen los límites regionales, Azure Disk Encryption necesita que Key Vault y las máquinas virtuales estén ubicadas en la misma región.   
**Detalles**: Cree y use una instancia de Key Vault que se encuentre en la misma región que la máquina virtual que se va a cifrar.

Cuando se aplica Azure Disk Encryption, puede atender las siguientes necesidades empresariales:

- Las máquinas virtuales IaaS se protegen en reposo a través de la tecnología de cifrado estándar del sector para cumplir los requisitos de seguridad y cumplimiento de la organización.
- Las máquinas virtuales IaaS se inician bajo directivas y claves controladas por el cliente, y puede auditar su uso en el almacén de claves.

## <a name="restrict-direct-internet-connectivity"></a>Restringir la conectividad directa a Internet
Supervise y restrinja la conectividad directa a Internet de las máquinas virtuales. Los atacantes analizan constantemente los intervalos de IP de la nube pública en busca de puertos de administración abiertos e intentan realizar ataques "sencillos", como contraseñas comunes y vulnerabilidades conocidas sin revisiones. En esta tabla se enumeran los procedimientos recomendados para que sea más fácil protegerse frente a estos ataques:

**Procedimiento recomendado**: Evitar la exposición accidental a la seguridad y el enrutamiento de redes.   
**Detalles**: Use RBAC para asegurarse de que solo el grupo de redes central tiene permiso para los recursos de red.

**Procedimiento recomendado**: Identificar y corregir las máquinas virtuales expuestas que permiten el acceso desde "cualquier" dirección IP de origen.   
**Detalles**: Use Azure Security Center. Security Center le recomendará que restrinja el acceso a través de puntos de conexión accesibles desde Internet si alguno de los grupos de seguridad de red tiene una o varias reglas de entrada que permiten el acceso desde “cualquier” dirección IP de origen. Security Center recomienda editar estas reglas de entrada para [restringir el acceso](../../security-center/security-center-network-recommendations.md) a las direcciones IP de origen que realmente necesiten el acceso.

**Procedimiento recomendado**: Restringir los puertos de administración (RDP, SSH).   
**Detalles**: Puede usar el [acceso a VM Just-In-Time](../../security-center/security-center-just-in-time.md) para bloquear el tráfico entrante a las máquinas virtuales de Azure. Para ello, se reduce la exposición a ataques y se proporciona un acceso sencillo para conectarse a las máquinas virtuales cuando sea necesario. Cuando el acceso a Just-In-Time está habilitado, Security Center bloquea el tráfico entrante a las máquinas virtuales de Azure mediante la creación de una regla de grupo de seguridad de red. Se deben seleccionar los puertos de la máquina virtual para la que se bloqueará el tráfico entrante. Estos puertos los controla la solución Just-In-Time.

## <a name="next-steps"></a>Pasos siguientes
Vea [Patrones y procedimientos recomendados de seguridad en Azure](best-practices-and-patterns.md) para obtener más procedimientos recomendados de seguridad que pueda aplicar cuando diseñe, implemente y administre las soluciones en la nube mediante Azure.

En los siguientes recursos se ofrece más información general sobre la seguridad de Azure y los servicios de Microsoft relacionados:
* [Blog del equipo de seguridad de Azure](https://blogs.msdn.microsoft.com/azuresecurity/): ofrece información actualizada sobre lo último en seguridad de Azure
* [Microsoft Security Response Center](https://technet.microsoft.com/library/dn440717.aspx): aquí podrá notificar vulnerabilidades de seguridad de Microsoft, incluidos problemas con Azure, o también mediante correo electrónico a secure@microsoft.com.
