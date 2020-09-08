---
title: Introducción a las características de seguridad
description: Obtenga información sobre las funcionalidades de seguridad de Azure Backup que le ayudan a proteger los datos de copia de seguridad y a satisfacer las necesidades de seguridad de su negocio.
ms.topic: conceptual
ms.date: 03/12/2020
ms.openlocfilehash: f9a505723b40df61665a99e898d59ecfb8c03fd5
ms.sourcegitcommit: c6b9a46404120ae44c9f3468df14403bcd6686c1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88890474"
---
# <a name="overview-of-security-features-in-azure-backup"></a>Introducción a las características de seguridad de Azure Backup

Una de las medidas más importantes que puede adoptar para proteger los datos es contar con una infraestructura de copia de seguridad confiable. Igual importancia tiene asegurarse de que la copia de seguridad de los datos se realiza de manera segura y de que las copias de seguridad están protegidas en todo momento. Azure Backup proporciona seguridad al entorno de copia de seguridad, tanto si los datos están en tránsito como en reposo. En este artículo se indican las funcionalidades de seguridad de Azure Backup que le ayudan a proteger los datos de copia de seguridad y a satisfacer las necesidades de seguridad de su negocio.

## <a name="management-and-control-of-identity-and-user-access"></a>Administración y control de identidades y del acceso de usuarios

Las cuentas de almacenamiento empleadas por los almacenes de Recovery Services están aisladas, y los usuarios no pueden acceder a ellas con fines malintencionados. Solo se permite el acceso mediante operaciones de administración de Azure Backup, como la restauración. Azure Backup le permite controlar las operaciones administradas mediante acceso específico con [control de acceso basado en rol (RBAC) de Azure](./backup-rbac-rs-vault.md). RBAC permite repartir las tareas entre el equipo y conceder a los usuarios únicamente el nivel de acceso que necesitan para realizar su trabajo.

Azure Backup proporciona tres [roles integrados](../role-based-access-control/built-in-roles.md) para controlar las operaciones de administración de copia de seguridad:

* Colaborador de copia de seguridad: crea y administra las copias de seguridad, excepto para eliminar el almacén de Recovery Services y facilitar acceso a otros usuarios.
* Operador de copia de seguridad: hace todo lo que puede hacer un colaborador, excepto quitar copias de seguridad y administrar directivas de copia de seguridad.
* Lector de copia de seguridad: tiene permisos para ver todas las operaciones de administración de copia de seguridad.

Obtenga más información sobre el [uso del control de acceso basado en roles para administrar puntos de recuperación de Azure Backup](./backup-rbac-rs-vault.md).

Azure Backup tiene varios controles de seguridad integrados en el servicio para evitar, detectar y responder a los puntos vulnerables de seguridad. Obtenga más información sobre los [controles de seguridad para Azure Backup](./backup-security-controls.md).

## <a name="separation-between-guest-and-azure-storage"></a>Separación entre el invitado y Azure Storage

Con Azure Backup, que incluye copias de seguridad de máquina virtual y copias de seguridad de SQL y SAP HANA en máquinas virtuales, los datos de copia de seguridad se almacenan en Azure Storage y el invitado no tiene acceso directo al almacenamiento de copia de seguridad ni su contenido.  En las copias de seguridad de máquina virtual, la creación y el almacenamiento de la instantánea de copia de seguridad se realizan mediante el tejido de Azure, donde el invitado no tiene más implicación que poner en modo inactivo la carga de trabajo para conseguir copias de seguridad coherentes.  En el caso de SQL y SAP HANA, la extensión de copia de seguridad obtiene acceso temporal para escribir en blobs específicos.  De esta manera, incluso en un entorno en peligro, el invitado no podrá alterar ni eliminar las copias de seguridad existentes.

## <a name="internet-connectivity-not-required-for-azure-vm-backup"></a>No se requiere conectividad a Internet para la copia de seguridad de máquinas virtuales de Azure

La copia de seguridad de las máquinas virtuales de Azure precisa que se muevan los datos del disco de la máquina virtual al almacén de Recovery Services. Sin embargo, todas las comunicaciones y transferencias de datos necesarias solo se producen en la red troncal de Azure, sin necesidad de acceder a la red virtual. Por lo tanto, la copia de seguridad de las máquinas virtuales de Azure colocadas dentro de redes protegidas no requiere que permita el acceso a direcciones IP ni nombres de dominio completo.

## <a name="private-endpoints-for-azure-backup"></a>Puntos de conexión privados en Azure Backup

Ahora puede usar [puntos de conexión privados](../private-link/private-endpoint-overview.md) para realizar copias de seguridad de los datos de forma segura desde los servidores de una red virtual al almacén de Recovery Services. El punto de conexión privado usa una dirección IP del espacio de direcciones de la red virtual para el almacén, por lo que no es necesario exponer las redes virtuales a ninguna dirección IP pública. Los puntos de conexión privados se pueden usar para realizar copias de seguridad y restaurar las bases de datos de SQL y SAP HANA que se ejecutan dentro de las máquinas virtuales de Azure. También se puede usar para los servidores locales mediante el agente de MARS.

Obtenga más información sobre los puntos de conexión privados para Azure Backup [aquí](./private-endpoints.md).

## <a name="encryption-of-data"></a>Cifrado de datos

Mediante el cifrado, se protegen los datos y es más fácil cumplir los compromisos de cumplimiento y seguridad de la organización. El cifrado de datos se produce en muchas fases en Azure Backup:

* Dentro de Azure, los datos en tránsito entre Azure Storage y el almacén [se protegen mediante HTTPS](backup-support-matrix.md#network-traffic-to-azure). Estos datos permanecen en la red troncal de Azure.

* Los datos de copia de seguridad se cifran automáticamente mediante [claves administradas por la plataforma](backup-encryption.md) y no es necesario realizar ninguna acción explícita para habilitarlos. También puede cifrar los datos de copia de seguridad con [claves administradas por el cliente](encryption-at-rest-with-cmk.md) almacenadas en Azure Key Vault. Se aplica a todas las cargas de trabajo de las que se realiza una copia de seguridad en el almacén de Recovery Services.

* Azure Backup admite la copia de seguridad y la restauración de máquinas virtuales de Azure que tengan cifrados sus discos del sistema operativo o de datos con [Azure Disk Encryption (ADE)](backup-azure-vms-encryption.md#encryption-support-using-ade) y [máquinas virtuales con discos cifrados con CMK](backup-azure-vms-encryption.md#encryption-using-customer-managed-keys). Obtenga más información sobre las [máquinas virtuales de Azure cifradas y Azure Backup](./backup-azure-vms-encryption.md).

* Cuando se realiza una copia de seguridad de los datos desde servidores locales con el agente de MARS, los datos se cifran con una frase de contraseña antes de cargarse en Azure Backup y se descifran solo después de descargarse de Azure Backup. Obtenga más información sobre las [características de seguridad que ayudan a proteger las copias de seguridad híbridas](#security-features-to-help-protect-hybrid-backups).

## <a name="protection-of-backup-data-from-unintentional-deletes"></a>Protección de los datos de copia de seguridad ante eliminaciones accidentales

Azure Backup proporciona características de seguridad que ayudan a proteger los datos de copia de seguridad incluso después de su eliminación. Con la eliminación temporal, si un usuario elimina la copia de seguridad de una máquina virtual, los datos de copia de seguridad se conservan durante 14 días más, lo que permite la recuperación de ese elemento de copia de seguridad sin pérdida de datos. Esta retención adicional de 14 días de los datos de copia de seguridad en el estado de "eliminación temporal" no le supone ningún costo. [Más información sobre la eliminación temporal](backup-azure-security-feature-cloud.md).

## <a name="monitoring-and-alerts-of-suspicious-activity"></a>Supervisión y alertas por actividad sospechosa

Azure Backup proporciona [funcionalidades de supervisión y alertas integradas](./backup-azure-monitoring-built-in-monitor.md) para ver y configurar acciones en eventos relacionados con Azure Backup. La solución [Informes de Backup](./configure-reports.md) sirve como destino único para realizar el seguimiento del uso, para llevar a cabo la auditoría de copias de seguridad y restauraciones, y para identificar las tendencias clave en diferentes niveles de granularidad. El uso de las herramientas de supervisión e informes de Azure Backup puede avisarle de cualquier actividad no autorizada, sospechosa o malintencionada en cuanto se produzca.

## <a name="security-features-to-help-protect-hybrid-backups"></a>Características de seguridad que ayudan a proteger las copias de seguridad híbridas

El servicio Azure Backup usa el agente de Microsoft Azure Recovery Services (MARS) para crear copias de seguridad y restaurar archivos y carpetas, así como el estado del volumen o el sistema desde un equipo local en Azure. Ahora MARS ofrece características de seguridad para ayudar a proteger las copias de seguridad híbridas. Estas características son:

* Se agrega una capa adicional de autenticación cada vez que se realiza una operación crítica, como cambiar la frase de contraseña. Esta validación se realiza para asegurarse de que dichas operaciones solo pueden realizarlas usuarios que tengan credenciales de Azure válidas. [Más información sobre las características que impiden ataques](./backup-azure-security-feature.md#prevent-attacks).

* Los datos de copia de seguridad eliminados se conservan durante 14 días a partir de la fecha de la eliminación. Esto garantiza la capacidad de recuperación de los datos en un período dado, con el fin de que no haya pérdida de datos aunque se produzca un ataque. Además, se mantiene un mayor número de puntos de recuperación mínimos para protegerse contra datos dañados. [Más información sobre la recuperación de datos de copia de seguridad eliminados](./backup-azure-security-feature.md#recover-deleted-backup-data).

* En el caso de los datos cuya copia de seguridad se ha realizado mediante el agente de Microsoft Azure Recovery Services (MARS), se usa una frase de contraseña para asegurarse de que los datos se cifran antes de la carga en Azure Backup y se descifran solo después de la descarga desde Azure Backup. Los detalles de la frase de contraseña solo están disponibles para el usuario que creó dicha frase y para el agente que se configura con ella. No se transmite ni se comparte nada con el servicio. Esto garantiza la seguridad completa de los datos, ya que cualquier dato que se exponga involuntariamente (por ejemplo, en un ataque de tipo "Man in the Middle" en la red) no se podrá usar sin la frase de contraseña y esta no se envía a la red.

## <a name="compliance-with-standardized-security-requirements"></a>Cumplimiento de los requisitos de seguridad estandarizados

Para ayudar a las organizaciones a cumplir los requisitos nacionales, regionales y específicos del sector que rigen la recopilación y el uso de los datos de las personas, Microsoft Azure y Azure Backup ofrecen un completo conjunto de certificaciones y testimonios. [Consulte la lista de certificaciones de cumplimiento](compliance-offerings.md).

## <a name="next-steps"></a>Pasos siguientes

* [Características de seguridad para proteger cargas de trabajo en la nube mediante Azure Backup](backup-azure-security-feature-cloud.md)
* [Características de seguridad para proteger copias de seguridad híbridas mediante Azure Backup](backup-azure-security-feature.md)
