---
title: Planeamiento de una implementación de Azure Files | Microsoft Docs
description: Conozca cómo se planea una implementación de Azure Files. Puede montar directamente un recurso compartido de archivos de Azure file share o almacenar en la caché un recurso compartido de archivos de Azure de forma local con Azure File Sync.
author: roygara
ms.service: storage
ms.topic: conceptual
ms.date: 1/3/2020
ms.author: rogarana
ms.subservice: files
ms.custom: references_regions
ms.openlocfilehash: db7ae0bd33bc52f80788db4994dcf2a3ca4d909a
ms.sourcegitcommit: e0785ea4f2926f944ff4d65a96cee05b6dcdb792
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88705918"
---
# <a name="planning-for-an-azure-files-deployment"></a>Planeamiento de una implementación de Azure Files
[Azure Files](storage-files-introduction.md) se puede implementar de dos formas principales: montando directamente los recursos compartidos de archivos de Azure sin servidor o almacenando en caché recursos compartidos de archivos de Azure localmente mediante Azure File Sync. La opción de implementación que elija cambiará todo aquello que debe tener en cuenta a la hora de planear la implementación. 

- **Montaje directo de un recurso compartido de archivos de Azure**: dado que Azure Files proporciona acceso SMB, puede montar recursos compartidos de archivos de Azure locales o en la nube mediante el cliente SMB estándar disponible en Windows, macOS y Linux. Dado que los recursos compartidos de archivos de Azure no tienen servidor, la implementación en escenarios de producción no requiere la administración de un servidor de archivos o un dispositivo NAS, lo que significa que no tiene que aplicar revisiones de software ni intercambiar discos físicos. 

- **Almacenamiento en caché de recursos compartidos de archivos de Azure localmente con Azure File Sync**: Azure File Sync le permite centralizar los recursos compartidos de archivos de su organización en Azure Files sin renunciar a la flexibilidad, el rendimiento y la compatibilidad de un servidor de archivos local. Azure File Sync transforma una instancia de Windows Server local (o en la nube) en una caché rápida de su recurso compartido de archivos de Azure. 

En este artículo se abordan principalmente las consideraciones de implementación de un recurso compartido de archivos de Azure que se va a montar directamente mediante un cliente local o en la nube. Para planear una implementación de Azure File Sync, consulte [Planeamiento de una implementación de Azure File Sync](storage-sync-files-planning.md).

## <a name="management-concepts"></a>Conceptos de administración
[!INCLUDE [storage-files-file-share-management-concepts](../../../includes/storage-files-file-share-management-concepts.md)]

Al implementar recursos compartidos de archivos de Azure en cuentas de almacenamiento, se recomienda:

- Implementar solo recursos compartidos de archivos de Azure en cuentas de almacenamiento con otros recursos compartidos de archivos de Azure. Aunque las cuentas de almacenamiento de GPv2 permiten tener cuentas de almacenamiento de propósito combinado, dado que los recursos de almacenamiento como los recursos compartidos de archivos de Azure y los contenedores de blobs comparten los límites de la cuenta de almacenamiento, la combinación de recursos puede dificultar la solución de problemas de rendimiento más adelante. 

- Prestar atención a las limitaciones de IOPS de la cuenta de almacenamiento al implementar recursos compartidos de archivos de Azure. Lo ideal sería asignar recursos compartidos de archivos 1:1 a cuentas de almacenamiento; sin embargo, quizás no sea posible debido a diversos límites y restricciones, tanto de su organización como de Azure. Cuando no sea posible tener un solo recurso compartido de archivos implementado en una cuenta de almacenamiento, tenga en cuenta qué recursos compartidos estarán muy activos y cuales estarán menos activos, con el fin de asegurarse de que los recursos compartidos de archivos más activos no se colocan en la misma cuenta de almacenamiento.

- Implementar solamente cuentas de GPv2 y FileStorage, y actualizar las cuentas de almacenamiento clásicas y de GPv1 cuando las encuentre en su entorno. 

## <a name="identity"></a>Identidad
Para acceder a un recurso compartido de archivos de Azure, el usuario debe estar autenticado y tener la debida autorización. Esto se hace en función de la identidad del usuario que accede al recurso compartido de archivos. Azure Files se integra con tres proveedores de identidades principales:
- **Active Directory Domain Services (AD DS) local (AD DS local)** : Las cuentas de almacenamiento de Azure pueden estar unidas a un dominio de Active Directory Domain Services propiedad del cliente, al igual que un servidor de archivos de Windows Server o un dispositivo NAS. Puede implementar un controlador de dominio en el entorno local, en una VM de Azure o, incluso, como VM en otro proveedor de nube. Azure Files es independiente de la ubicación donde se hospeda el controlador de dominio. Una vez que una cuenta de almacenamiento está unida a un dominio, el usuario final puede montar un recurso compartido de archivos con la cuenta de usuario con la que inició sesión en su equipo. La autenticación basada en AD usa el protocolo de autenticación Kerberos.
- **Azure Active Directory Domain Services (Azure AD DS)** : Azure AD DS proporciona un controlador de dominio administrado por Microsoft que se puede usar para los recursos de Azure. La unión a un dominio de la cuenta de almacenamiento a Azure AD DS proporciona ventajas similares a la unión a un dominio de dicha cuenta a una instancia de Active Directory propiedad del cliente. Esta opción de implementación es especialmente útil para escenarios de migración mediante lift-and-shift de aplicaciones que requieren permisos basados en AD. Dado que Azure AD DS proporciona autenticación basada en AD, esta opción también usa el protocolo de autenticación Kerberos.
- **Clave de la cuenta de Azure Storage**: los recursos compartidos de archivos de Azure también se pueden montar con una clave de cuenta de almacenamiento de Azure. Para montar un recurso compartido de archivos de esta forma, el nombre de la cuenta de almacenamiento se usa como nombre de usuario y la clave de la cuenta de almacenamiento se usa como contraseña. El uso de la clave de la cuenta de almacenamiento para montar el recurso compartido de archivos de Azure es realmente una operación de administrador, ya que el recurso compartido de archivos montado tendrá permisos completos para todos los archivos y todas las carpetas del recurso compartido, aunque tengan ACL. Cuando se usa la clave de la cuenta de almacenamiento para el montaje a través de SMB, se usa el protocolo de autenticación NTLMv2.

En el caso de los clientes que realizan la migración desde servidores de archivos locales o que crean nuevos recursos compartidos de archivos en Azure Files destinados a comportarse como servidores de archivos de Windows o dispositivos NAS, la unión a un dominio de la cuenta de almacenamiento a **Active Directory propiedad del cliente** es la opción recomendada. Para más información acerca de la unión a un dominio de la cuenta de almacenamiento a una instancia de Active Directory propiedad del cliente, consulte la [introducción a Active Directory de Azure Files](storage-files-active-directory-overview.md).

Si tiene previsto usar la clave de la cuenta de almacenamiento para acceder a los recursos compartidos de archivos de Azure, se recomienda usar puntos de conexión de servicio como se describe en la sección [Redes](#networking).

## <a name="networking"></a>Redes
Los recursos compartidos de archivos de Azure son accesibles desde cualquier lugar a través del punto de conexión público de la cuenta de almacenamiento. Esto significa que las solicitudes autenticadas, como las solicitudes autorizadas por la identidad de inicio de sesión de un usuario, pueden originarse de forma segura dentro o fuera de Azure. En muchos entornos de cliente, se producirá un error en el montaje inicial del recurso compartido de archivos de Azure en la estación de trabajo local, aunque los montajes desde las máquinas virtuales de Azure se realicen correctamente. El motivo es que muchas organizaciones y proveedores de servicios de Internet (ISP) bloquean el puerto que usa SMB para comunicarse, el puerto 445. Para ver el resumen de los ISP que permiten o deniegan el acceso desde el puerto 445, visite [TechNet](https://social.technet.microsoft.com/wiki/contents/articles/32346.azure-summary-of-isps-that-allow-disallow-access-from-port-445.aspx).

Para desbloquear el acceso al recurso compartido de archivos de Azure, tiene dos opciones principales:

- Desbloquear el puerto 445 para la red local de su organización. Solo se puede acceder externamente a recursos compartidos de archivos de Azure a través del punto de conexión público mediante protocolos seguros para Internet, como SMB 3.0 y la API de FileREST. Esta es la manera más fácil de acceder al recurso compartido de archivos de Azure desde el entorno local, ya que no requiere una configuración de red avanzada más allá de cambiar las reglas de puerto de salida de la organización. Sin embargo, se recomienda quitar las versiones heredadas y en desuso del protocolo SMB, concretamente SMB 1.0. Para obtener información sobre cómo hacerlo, consulte [Protección de Windows y Windows Server](storage-how-to-use-files-windows.md#securing-windowswindows-server) y [Protección de Linux](storage-how-to-use-files-linux.md#securing-linux).

- Acceder a recursos compartidos de archivos de Azure a través de una conexión de ExpressRoute o VPN. Al acceder al recurso compartido de archivos de Azure a través de un túnel de red, puede montar el recurso compartido de archivos de Azure como un recurso compartido de archivos local, ya que el tráfico SMB no atraviesa el límite de la organización.   

Aunque desde una perspectiva técnica es considerablemente más fácil montar los recursos compartidos de archivos de Azure a través del punto de conexión público, esperamos que la mayoría de los clientes opten por montar sus recursos compartidos de archivos de Azure a través de una conexión de ExpressRoute o VPN. Para ello, deberá configurar las opciones siguientes para su entorno:  

- **Tunelización de red mediante ExpressRoute o una VPN de sitio a sitio o de punto a sitio**: la tunelización en una red virtual permite acceder a recursos compartidos de archivos de Azure desde el entorno local, aunque el puerto 445 esté bloqueado.
- **Puntos de conexión privados**: los puntos de conexión privados proporcionan a su cuenta de almacenamiento una dirección IP dedicada desde el espacio de direcciones de la red virtual. Esto permite la tunelización de red sin necesidad de abrir redes locales en todos los intervalos de direcciones IP que son propiedad de los clústeres de almacenamiento de Azure. 
- **Reenvío DNS**: configure su DNS local para resolver el nombre de la cuenta de almacenamiento (es decir, `storageaccount.file.core.windows.net` para las regiones de la nube pública) para resolver la dirección IP de sus puntos de conexión privados.

Para planear las redes asociadas a la implementación de un recurso compartido de archivos de Azure, consulte [Consideraciones de redes para Azure Files](storage-files-networking-overview.md).

## <a name="encryption"></a>Cifrado
Azure Files admite dos tipos de cifrado diferentes: el cifrado en tránsito, que se relaciona con el cifrado que se usa al montar el recurso compartido de archivos de Azure y acceder a este, y el cifrado en reposo, relacionado con la forma en que se cifran los datos cuando se almacenan en el disco. 

### <a name="encryption-in-transit"></a>Cifrado en tránsito
De forma predeterminada, todas las cuentas de Azure Storage tienen habilitado el cifrado en tránsito. Esto significa que al montar un recurso compartido de archivos a través de SMB o acceder a él a través del protocolo de FileREST (por ejemplo, a través de Azure Portal, la CLI o PowerShell, o los SDK de Azure), Azure Files solo permitirá la conexión si se realiza con una versión posterior a SMB 3.0 con cifrado o HTTPS. Los clientes que no admiten SMB 3.0, o los clientes que admiten SMB 3.0, pero no al cifrado SMB no podrán montar el recurso compartido de archivos de Azure si está habilitado el cifrado en tránsito. Para obtener más información sobre qué sistemas operativos admiten SMB 3.0 con cifrado, consulte nuestra documentación detallada para [Windows](storage-how-to-use-files-windows.md), [macOS](storage-how-to-use-files-mac.md) y [Linux](storage-how-to-use-files-linux.md). Todas las versiones actuales de PowerShell, la CLI y los SDK admiten HTTPS.  

Puede deshabilitar el cifrado en tránsito para una cuenta de almacenamiento de Azure. Cuando el cifrado está deshabilitado, Azure Files también permite el uso de SMB 2.1, SMB 3.0 sin cifrado y las llamadas a la API de FileREST sin cifrar a través de HTTP. La razón principal para deshabilitar el cifrado en tránsito es admitir una aplicación heredada que debe ejecutarse en un sistema operativo anterior, como Windows Server 2008 R2 o una distribución de Linux anterior. Azure Files solo permite conexiones SMB 2.1 dentro de la misma región de Azure del recurso compartido de archivos de Azure. Un cliente SMB 2.1 fuera de la región de Azure del recurso compartido de archivos de Azure (por ejemplo, en un entorno local o en una región de Azure diferente) no podrá acceder al recurso compartido de archivos.

Se recomienda encarecidamente asegurarse de que está habilitado el cifrado de los datos en tránsito.

Para obtener más información sobre el cifrado en tránsito, consulte [Requerir transferencia segura en Azure Storage](../common/storage-require-secure-transfer.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json).

### <a name="encryption-at-rest"></a>Cifrado en reposo
[!INCLUDE [storage-files-encryption-at-rest](../../../includes/storage-files-encryption-at-rest.md)]

## <a name="data-protection"></a>Protección de los datos
Azure Files tiene un enfoque de varias capas para garantizar la copia de seguridad de los datos, su recuperación y su protección contra amenazas de seguridad.

### <a name="soft-delete"></a>Eliminación temporal
La eliminación temporal de recursos compartidos de archivos (versión preliminar) es una configuración de nivel de cuenta de almacenamiento que le permite recuperar el recurso compartido de archivos cuando se elimina accidentalmente. Cuando se elimina un recurso compartido de archivos, pasa a un estado de eliminación temporal, en lugar de borrarse de forma permanente. Se puede configurar el tiempo durante el que los datos eliminados de forma temporal se pueden recuperar antes de que se eliminen permanentemente y durante este período de retención el recurso compartido se puede recuperar en cualquier momento. 

Se recomienda activar la eliminación temporal para la mayoría de los recursos compartidos de archivos. Si tiene un flujo de trabajo en el que la eliminación de recursos compartidos es común y se espera, puede que decida tener un período de retención muy corto o no tener habilitada la eliminación temporal.

Para obtener más información acerca de la eliminación temporal, consulte [Evitar la eliminación accidental de datos](https://docs.microsoft.com/azure/storage/files/storage-files-prevent-file-share-deletion).

### <a name="backup"></a>Copia de seguridad
Puede realizar una copia de seguridad del recurso compartido de archivos de Azure a través de [instantáneas de recurso compartido](https://docs.microsoft.com/azure/storage/files/storage-snapshots-files), que son copias de solo lectura de un momento dado del recurso compartido. Las instantáneas son incrementales, lo que significa que solo contienen los datos que han cambiado desde la instantánea anterior. Puede tener hasta 200 instantáneas por recurso compartido de archivos y conservarlas durante un máximo de diez años. Puede realizar estas instantáneas manualmente en Azure Portal, a través de PowerShell o en la interfaz de la línea de comandos (CLI), o bien puede usar [Azure Backup](https://docs.microsoft.com/azure/backup/azure-file-share-backup-overview?toc=/azure/storage/files/toc.json). Las instantáneas se almacenan en el recurso compartido de archivos, lo que significa que si lo elimina, también se eliminarán las instantáneas. Para proteger las copias de seguridad de instantáneas contra eliminaciones accidentales, asegúrese de que la eliminación temporal está habilitada para el recurso compartido.

[Azure Backup para recursos compartidos de archivos de Azure](https://docs.microsoft.com/azure/backup/azure-file-share-backup-overview?toc=/azure/storage/files/toc.json) controla la programación y retención de instantáneas. Sus capacidades de abuelo-padre-hijo (GFS) significan que puede tomar instantáneas diarias, semanales, mensuales y anuales, cada una con su propio período de retención distinto. Azure Backup también organiza la habilitación de la eliminación temporal y toma un bloqueo de eliminación en una cuenta de almacenamiento en cuanto se configura un recurso compartido de archivos en ella para la copia de seguridad. Por último, Azure Backup proporciona ciertas capacidades clave de supervisión y alertas que permiten a los clientes tener una vista consolidada de su copia de seguridad.

Puede realizar restauraciones de nivel de elemento y de nivel de recurso compartido en Azure Portal mediante Azure Backup. Lo único que debe hacer es elegir el punto de restauración (una instantánea concreta), el archivo o directorio en cuestión si es pertinente y, a continuación, la ubicación (original o alternativa) en la que quiere realizar la restauración. El servicio de copia de seguridad controla la copia de los datos de instantáneas y muestra el progreso de la restauración en el portal.

Para obtener más información sobre Azure Backup, vea [Acerca de la copia de seguridad de recursos compartidos de archivos de Azure](https://docs.microsoft.com/azure/backup/azure-file-share-backup-overview?toc=/azure/storage/files/toc.json).

### <a name="advanced-threat-protection-for-azure-files-preview"></a>Advanced Threat Protection para Azure Files (versión preliminar)
Advanced Threat Protection (ATP) para Azure Storage ofrece una capa adicional de inteligencia de seguridad que proporciona alertas cuando detecta actividades anómalas en la cuenta de almacenamiento, por ejemplo, intentos no habituales de acceso a la cuenta de almacenamiento. ATP también ejecuta análisis de reputación de hash de malware y generará una alerta sobre malware conocido. Puede configurar ATP en un nivel de suscripción o de cuenta de almacenamiento a través de Azure Security Center. 

Para más información, consulte [Advanced Threat Protection para Azure Storage](https://docs.microsoft.com/azure/storage/common/storage-advanced-threat-protection).

## <a name="storage-tiers"></a>Niveles de almacenamiento
[!INCLUDE [storage-files-tiers-overview](../../../includes/storage-files-tiers-overview.md)]

En general, las características de Azure Files y la interoperabilidad con otros servicios coinciden en los recursos compartidos de archivos prémium y los recursos compartidos de archivos estándar (lo que incluye los recursos compartidos de archivos frecuentes, esporádicos y optimizados de transacciones). Sin embargo, hay algunas diferencias importantes:
- **Modelo de facturación**
    - Los recursos compartidos de archivos prémium se facturan con un modelo de facturación aprovisionado, lo que significa que se paga por la cantidad de almacenamiento que se aprovisiona en lugar de la cantidad de almacenamiento que se solicita realmente. 
    - Los recursos compartidos de archivos estándar se facturan con un modelo de pago por uso, que incluye un costo básico de almacenamiento para la cantidad de almacenamiento que realmente se consume y, a continuación, un costo de transacción adicional basado en el uso del recurso compartido. Con los recursos compartidos de archivos estándar, su factura aumentará si usa (lee, escribe o monta) más el recurso compartido de archivos de Azure.
- **Opciones de redundancia**
    - Los recursos compartidos de archivos prémium solo están disponibles para el almacenamiento con redundancia local (LRS) y redundancia de zona (ZRS).
    - Los recursos compartidos de archivos estándar están disponibles para el almacenamiento con redundancia local, redundancia de zona, redundancia geográfica (GRS) y redundancia de zona geográfica (GZRS).
- **Tamaño máximo de un recurso compartido de archivos**
    - Los recursos compartidos de archivos prémium se pueden aprovisionar hasta 100 TiB sin ningún trabajo adicional.
    - De forma predeterminada, los recursos compartidos de archivos estándar solo pueden abarcar hasta 5 TiB, aunque se puede aumentar el límite de recursos compartidos hasta 100 TiB al optar por recibir la marca de la característica de cuenta de almacenamiento de *recurso compartido de archivos de gran tamaño*. Los recursos compartidos de archivos estándar solo pueden abarcar hasta 100 TiB para las cuentas de almacenamiento con redundancia local o redundancia de zona. Para más información sobre cómo aumentar los tamaños de los recursos compartidos de archivos, vea [Habilitación y creación de recursos compartidos de archivos grandes](https://docs.microsoft.com/azure/storage/files/storage-files-how-to-create-large-file-share).
- **Disponibilidad regional**
    - Los recursos compartidos de archivos prémium no están disponibles en todas las regiones y la compatibilidad con la redundancia de zona está disponible en un subconjunto de regiones más pequeño. Para averiguar si los recursos compartidos de archivos prémium están disponibles actualmente en su región, consulte la página [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/?products=storage) para Azure. Para averiguar qué regiones admiten ZRS, consulte la tabla de [soporte técnico para las zonas de disponibilidad de Azure por región](../../availability-zones/az-region.md). Para ayudarnos a clasificar por orden de prioridad las nuevas regiones y características del nivel Premium, rellene esta [encuesta](https://aka.ms/pfsfeedback).
    - Los recursos compartidos de archivos estándar están disponibles en todas las regiones de Azure.
- Azure Kubernetes Service (AKS) admite recursos compartidos de archivos prémium en la versión 1.13 y posteriores.

Una vez que se crea un recurso compartido de archivos prémium o estándar, este no se puede convertir automáticamente al otro nivel. Para cambiar al otro nivel, debe crear un nuevo recurso compartido de archivos en ese nivel y copiar manualmente los datos del recurso compartido original en el nuevo que acaba de crear. Para realizar esta copia, se recomienda usar `robocopy` para Windows o `rsync` para macOS y Linux.

### <a name="understanding-provisioning-for-premium-file-shares"></a>Descripción del aprovisionamiento de recursos compartidos de archivos prémium
Los recursos compartidos de archivos Premium se aprovisionan en función de una relación fija de GiB/IOPS/rendimiento. Por cada GiB aprovisionado, se generará un IOPS y un rendimiento de 0,1 MiB por segundo en el recurso compartido hasta los límites máximos por recurso compartido. El aprovisionamiento mínimo que se permite es 100 GiB con un IOPS/rendimiento mínimos.

En su máximo esfuerzo, todos los recursos compartidos pueden aumentar hasta tres IOPS por GiB de almacenamiento aprovisionado durante 60 minutos, o más, según el tamaño del recurso compartido. Los nuevos recursos compartidos comienzan con todos los créditos de aumento según la capacidad aprovisionada.

Los recursos compartidos se deben aprovisionar en incrementos de 1 GiB. El tamaño mínimo es de 100 GB; el siguiente, de 101 GiB y así sucesivamente.

> [!TIP]
> IOPS de línea de base = 1 * GiB aprovisionados. (Hasta un máximo de 100 000 IOPS).
>
> Límite de ráfaga = 3 * IOPS de línea de base. (Hasta un máximo de 100 000 IOPS).
>
> Velocidad de salida = 60 MiB/s + 0,06 * GiB aprovisionados
>
> Velocidad de entrada = 40 MiB/s + 0,04 * GiB aprovisionados

El tamaño de recurso compartido aprovisionado se especifica mediante la cuota de recurso compartidos. El recurso compartido puede aumentar en cualquier momento, pero solo se puede reducir una vez transcurridas 24 horas desde el último aumento. Después de esperar 24 horas sin un aumento de cuota, puede reducir el recurso compartido tantas veces como quiera hasta que lo vuelva a aumentar. Los cambios de escala de IOPS/rendimiento se aplicarán minutos después del cambio de tamaño.

Es posible reducir el tamaño del recurso compartido aprovisionados por debajo de sus GiB usados. Si lo hace, no se perderán datos, pero se le seguirá facturando el tamaño usado y seguirá recibiendo el rendimiento (IOPS de línea de base, rendimiento e IOPS de ráfaga) del recurso compartido aprovisionado, no del tamaño utilizado.

En la tabla siguiente se ilustran algunos ejemplos de estas fórmulas para los tamaños de recursos compartidos aprovisionados:

|Capacidad (GiB) | IOPS base | IOPS de ráfaga | Salida (MiB/s) | Entrada (MiB/s) |
|---------|---------|---------|---------|---------|
|100         | 100     | Hasta 300     | 66   | 44   |
|500         | 500     | Hasta 1500   | 90   | 60   |
|1024       | 1024   | Hasta 3072   | 122   | 81   |
|5120       | 5120   | Hasta 15 360  | 368   | 245   |
|10 240      | 10 240  | Hasta 30 720  | 675 | 450   |
|33 792      | 33 792  | Hasta 100 000 | 2088 | 1392   |
|51 200      | 51 200  | Hasta 100 000 | 3132 | 2088   |
|102 400     | 100 000 | Hasta 100 000 | 6204 | 4136   |

> [!NOTE]
> El rendimiento de los recursos compartidos de archivos está sujeto a los límites de red de la máquina, el ancho de banda de red disponible, los tamaños de E/S y el paralelismo, entre muchos otros factores. Por ejemplo, en función de las pruebas internas con tamaños de e/s de lectura/escritura de 8 KiB, una sola máquina virtual de Windows (*Standard F16s_v2*) conectada al recurso compartido de archivos Premium a través de SMB podría alcanzar un valor de hasta 20 000 IOPS de lectura y 15 000 IOPS de escritura. Con tamaños de e/s de lectura/escritura de 512 MiB, la misma máquina virtual puede alcanzar un rendimiento de salida de 1,1 GiB/s y 370 MiB/s de entrada. Para lograr una escala de rendimiento máxima, distribuya la carga entre varias VM. Consulte en la [guía de solución de problemas](storage-troubleshooting-files-performance.md) algunos problemas de rendimiento comunes y soluciones alternativas.

#### <a name="bursting"></a>Creación de ráfagas
Los recursos compartidos de archivos prémium pueden crear ráfagas de su IOPS hasta un factor de tres. La creación de ráfagas está automatizada y funciona de acuerdo con un sistema de crédito. La creación de ráfagas funciona en la medida de lo posible y el límite de ráfaga no es una garantía: los recursos compartidos de archivos pueden crear ráfagas *hasta* el límite.

Cada vez que el tráfico para el recurso compartido de archivos se encuentra por debajo del valor de IOPS de la línea de base, se acumulan créditos en un cubo de ráfagas. Por ejemplo, un recurso compartido de 100 GiB tiene un valor de IOPS de línea de base de 100. Si el tráfico real del recurso compartido era de 40 IOPS para un intervalo específico de 1 segundo, el valor de 60 IOPS sin usar se agrega a un cubo de ráfagas. A continuación, estos créditos se usan más tarde si las operaciones superan el valor de IOPS de línea de base.

> [!TIP]
> Tamaño del cubo de ráfagas = IOPS de línea de base * 2 * 3600.

Cada vez que un recurso compartido supera el valor de IOPS de línea de base y tiene créditos en un cubo de ráfaga, crea ráfagas. Los recursos compartidos pueden seguir creando ráfagas mientras queden créditos, aunque los recursos compartidos de menos de 50 TiB solo permanecerán en el límite de ráfagas un máximo de una hora. Técnicamente, los recursos compartidos de más de 50 TiB pueden superar el límite de una hora, hasta dos horas, pero esto depende de la cantidad de créditos de ráfaga acumulados. Cada E/S más allá del valor de IOPS de línea de base consume un crédito y, una vez que se consumen todos los créditos, el recurso compartido volvería al valor de IOPS de la línea base.

Los créditos de recursos compartidos tienen tres estados:

- Acumulado: cuando el recurso compartido de archivos usa un valor inferior al de IOPS de línea de base.
- Declive: cuando se están creado ráfagas del recurso compartido de archivos.
- Remaining constant (Restante constante): cuando no hay créditos ni IOPS de línea de base en uso.

Los nuevos recursos compartidos de archivo empiezan con la cantidad total de créditos del cubo de ráfagas. Los créditos de ráfaga no se acumularán si el valor de IOPS del recurso compartido cae por debajo del valor de IOPS de la línea de base debido a una limitación del servidor.

### <a name="enable-standard-file-shares-to-span-up-to-100-tib"></a>Habilitación de recursos compartidos de archivos estándar para incluir hasta 100 TiB
[!INCLUDE [storage-files-tiers-enable-large-shares](../../../includes/storage-files-tiers-enable-large-shares.md)]

#### <a name="limitations"></a>Limitaciones
[!INCLUDE [storage-files-tiers-large-file-share-availability](../../../includes/storage-files-tiers-large-file-share-availability.md)]

## <a name="redundancy"></a>Redundancia
[!INCLUDE [storage-files-redundancy-overview](../../../includes/storage-files-redundancy-overview.md)]

## <a name="migration"></a>Migración
En muchos casos, no se establecerá un nuevo recurso compartido de archivos para su organización, sino que se migrará uno existente de un servidor de archivos local o un dispositivo NAS a Azure Files. La elección de la herramienta y estrategia de migración adecuadas para el escenario es importante para el éxito de la migración. 

En el [artículo de introducción a la migración](storage-files-migration-overview.md) se describen brevemente los aspectos básicos y se incluye una tabla que le conduce a guías de migración que probablemente cubran su escenario.

## <a name="next-steps"></a>Pasos siguientes
* [Planeamiento de una implementación de Azure File Sync](storage-sync-files-planning.md)
* [Implementación de Azure Files](storage-files-deployment-guide.md)
* [Implementación de Azure File Sync](storage-sync-files-deployment-guide.md)
* [Consulte el artículo de introducción a la migración para encontrar la guía de migración para su escenario.](storage-files-migration-overview.md)
