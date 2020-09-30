---
title: Información general sobre el servidor flexible de Azure Database for MySQL
description: Obtenga más información acerca del servidor flexible de Azure Database for MySQL, que es un servicio de base de datos relacional de la nube de Microsoft basado en MySQL Community Edition.
author: savjani
ms.service: mysql
ms.author: pariks
ms.custom: mvc
ms.topic: overview
ms.date: 8/21/2020
ms.openlocfilehash: 4b79ee0999db0a19794cc167ae79ed6e58193b30
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90946218"
---
# <a name="azure-database-for-mysql---flexible-server-preview"></a>Servidor flexible (versión preliminar) de Azure Database for MySQL

Azure Database for MySQL, con tecnología de MySQL Community Edition, está disponible en dos modos de implementación:
- Servidor único 
- Servidor flexible (versión preliminar)

En este artículo, se proporcionará información general y una introducción a los conceptos básicos del modelo de implementación de servidor flexible. 

## <a name="overview"></a>Información general

Servidor flexible de Azure Database for MySQL es un servicio de base de datos totalmente administrado diseñado para proporcionar un control más granular y una mayor flexibilidad sobre las funciones de administración de bases de datos y las opciones de configuración. En general, el servicio proporciona mayor flexibilidad y personalizaciones de la configuración del servidor en función de los requisitos del usuario. La arquitectura de servidor flexible permite a los usuarios optar por la alta disponibilidad dentro de una única zona de disponibilidad o entre varias. Los servidores flexibles también proporcionan mejores controles de optimización de costos con la capacidad de iniciar o detener el servidor, y SKU flexibles, idóneas para cargas de trabajo que no necesitan una capacidad de proceso completa de forma continuada. El servicio admite actualmente la versión de comunidad de MySQL 5.7. El servicio está disponible actualmente en versión preliminar en muchas [regiones de Azure](https://azure.microsoft.com/global-infrastructure/services/).

Los servidores flexibles son idóneos para 
- Desarrollo de aplicaciones que requieren un control mejorado y personalizaciones.
- Alta disponibilidad con redundancia de zona
- Ventanas de mantenimiento administradas

![Diagrama conceptual del servidor flexible](media/overview/1-flexible-server-conceptual-diagram.png) 

## <a name="high-availability-within-and-across-availability-zones"></a>Alta disponibilidad dentro y entre zonas de disponibilidad

El modelo de implementación del servidor flexible está diseñado para admitir la alta disponibilidad dentro de una única zona de disponibilidad o entre varias. La arquitectura separa el proceso y el almacenamiento. El motor de base de datos se ejecuta en una máquina virtual, mientras que los archivos de datos residen en Azure Storage. El almacenamiento mantiene tres copias sincrónicas con redundancia local de los archivos de base de datos, lo que garantiza la durabilidad de los datos en todo momento. 

En una zona de disponibilidad única, si el servidor deja de funcionar debido a eventos planeados o no planeados, el servicio mantiene la alta disponibilidad de los servidores mediante el siguiente procedimiento automático:

1. Se aprovisiona una nueva máquina virtual de proceso.
2. El almacenamiento con archivos de datos se asigna a la nueva máquina virtual.
3. El motor de base de datos MySQL pasa a estar en línea en la nueva máquina virtual.
4. Las aplicaciones cliente pueden volver a conectarse una vez que el servidor esté listo para aceptar conexiones.
   
:::image type="content" source="media/overview/2-flexible-server-architecture.png" alt-text="Diagrama conceptual de la alta disponibilidad de zona única"::: 

Si está configurada la alta disponibilidad con redundancia de zona, el servicio aprovisiona y mantiene un servidor en espera activa entre zonas de disponibilidad dentro de la misma región de Azure. Los cambios en los datos del servidor de origen se replican sincrónicamente en el servidor en espera para garantizar que no se pierden datos. Con la alta disponibilidad con redundancia de zona, una vez que se desencadena el evento de conmutación por error planeado o no planeado, el servidor en espera se conecta inmediatamente y está disponible para procesar las transacciones entrantes. Los tiempos de conmutación por error normales oscilan entre los 60 y 120 segundos. Esto permite que el servicio admita la alta disponibilidad y proporcione una mayor resistencia con tolerancia para los errores de zona de disponibilidad única en una región de Azure determinada. 

Consulte los [conceptos de alta disponibilidad](concepts-high-availability.md) para obtener más detalles.

:::image type="content" source="media/overview/3-flexible-server-overview-zone-redundant-ha.png" alt-text="Diagrama conceptual de la alta disponibilidad con redundancia de zona"::: 

## <a name="automated-patching-with-managed-maintenance-window"></a>Aplicación de revisiones automatizada con la ventana de mantenimiento administrado

El servicio aplica automáticamente revisiones al hardware, sistema operativo y motor de base de datos subyacentes. La revisión incluye actualizaciones de seguridad y software. En el motor de MySQL, las actualizaciones de versión secundaria también se incluyen como parte de la versión de mantenimiento planeado. Los usuarios pueden configurar la programación de la aplicación de revisión para que la administre el sistema o definir su programación personalizada. Durante la programación de mantenimiento, se aplica la revisión y es posible que el servidor requiera un reinicio como parte del proceso de aplicación de revisiones para completar la actualización. Con la programación personalizada, los usuarios pueden hacer que el ciclo de aplicación de revisiones sea predecible y elegir una ventana de mantenimiento con un impacto mínimo en el negocio. En general, el servicio sigue una programación mensual de versiones como parte de la integración y publicación continuas. 

## <a name="automatic-backups"></a>Copias de seguridad automáticas

El servicio de servidor flexible crea automáticamente copias de seguridad del servidor y las almacena en el almacenamiento con redundancia local o con redundancia geográfica configurado por el usuario. Las copias de seguridad se pueden usar para restaurar el servidor a un momento dado dentro del período de retención de la copia de seguridad. El período de retención predeterminado es siete días. La retención se puede configurar para un total de 35 días. Todas las copias de seguridad se cifran mediante cifrado AES de 256 bits. 

Consulte los [conceptos de copia de seguridad](concepts-backup-restore.md) para obtener más información.

## <a name="network-isolation"></a>Aislamiento de red

Tiene dos opciones de red para conectar su instancia de Servidor flexible de Azure Database for MySQL. Las opciones son **acceso privado (integración con red virtual)** y **acceso público (direcciones IP permitidas)** . 

* **Acceso privado (Integración con red virtual)** : puede implementar el servidor flexible en la instancia de [Azure Virtual Network](../../virtual-network/virtual-networks-overview.md). Las redes virtuales de Azure proporcionan una comunicación de red privada y segura. Los recursos de una red virtual se pueden comunicar mediante direcciones IP privadas.

   Elija la opción Integración con red virtual si quiere las funcionalidades siguientes:
   * Conexión desde recursos de Azure en la misma red virtual a un servidor flexible mediante direcciones IP privadas
   * Uso de VPN o ExpressRoute para conectarse desde recursos que no son de Azure al servidor flexible
   * Ningún punto de conexión público

* **Acceso público (direcciones IP permitidas)** : puede implementar el servidor flexible a con un punto de conexión público. El punto de conexión público es una dirección DNS que se puede resolver públicamente. La frase "direcciones IP permitidas" hace referencia a un intervalo de direcciones IP a las que decida conceder permiso para acceder al servidor. Estos permisos se denominan **reglas de firewall**.

Consulte los [conceptos de redes](concepts-networking.md) para obtener más información.

## <a name="adjust-performance-and-scale-within-seconds"></a>Ajustar el rendimiento y la escala en cuestión de segundos

El servicio de servidor flexible está disponible en tres niveles de SKU: flexible, de uso general y optimizado para memoria. El nivel flexible es el más adecuado para las cargas de trabajo de desarrollo de bajo costo y simultaneidad que no necesitan una capacidad de proceso completa continuamente. Los niveles De uso general y Optimizado para memoria son los más adecuados para las cargas de trabajo de producción que requieren un alto nivel de simultaneidad, escala y rendimiento predecible. Puede compilar su primera aplicación en una base de datos pequeña por poco dinero al mes y, después, ajustar fácilmente la escala para satisfacer las necesidades de la solución. El escalado de almacenamiento se realiza en línea y admite el crecimiento automático del almacenamiento. La escalabilidad dinámica permite a la base de datos responder de manera transparente a los cambiantes requisitos de recursos. Solo paga por los recursos que consume. 

Consulte [Conceptos de proceso y almacenamiento](concepts-compute-storage.md) para obtener más información.

## <a name="stopstart-server-to-optimize-cost"></a>Detención o inicio del servidor para optimizar el costo

El servicio de servidor flexible le permite detener e iniciar el servidor a petición para optimizar los costos. La facturación del nivel de proceso se detiene inmediatamente al detener el servidor. De este modo puede lograr un importante ahorro en el costo durante el desarrollo, las pruebas y las cargas de trabajo de producción predecibles enlazadas con límites de tiempo. El servidor permanece en estado detenido durante siete días, a menos que se reinicie antes. 

## <a name="enterprise-grade-security-and-privacy"></a>Privacidad y seguridad de calidad empresarial

El servicio de servidor flexible usa el módulo criptográfico con validación FIPS 140-2 para el cifrado del almacenamiento de los datos en reposo. Se cifran los datos, incluidas las copias de seguridad, y los archivos temporales creados mientras se ejecutan las consultas. El servicio usa el cifrado AES de 256 bits que se incluye en el cifrado de Azure Storage y las claves las puede administrar el sistema (valor predeterminado). 

El servicio cifra los datos en movimiento con la seguridad de la capa de transporte que se aplica de manera predeterminada. Los servidores flexibles solo admiten conexiones cifradas mediante la Seguridad de la capa de transporte (TLS 1.2) y se denegarán todas las conexiones entrantes con TLS 1.0 y TLS 1.1. 

Consulte el [Uso de conexiones cifradas para servidores flexibles](/articles/mysql/flexible-server/how-to-connect-tls-ssl.md) para obtener más información.

Los servidores flexibles permiten el acceso privado completo a los servidores mediante la integración con [Azure Virtual Network](https://docs.microsoft.com/azure/virtual-network/virtual-networks-overview) (VNet). Solo puede acceder a los servidores de la red virtual de Azure y conectarse con ellos mediante las direcciones IP privadas. Con la integración con red virtual, se deniega el acceso público y no se puede acceder a los servidores mediante puntos de conexión públicos. 

Consulte los [conceptos de redes](concepts-networking.md) para obtener más información.


## <a name="monitoring-and-alerting"></a>Supervisión y alertas

El servicio de servidor flexible está equipado con características integradas de alertas y supervisión de rendimiento. Todas las métricas de Azure tienen una frecuencia de un minuto y cada métrica proporciona 30 días de historial. Puede configurar alertas en las métricas. El servicio expone las métricas del servidor host para supervisar el uso de los recursos y permite configurar registros de consultas lentas. Con estas herramientas, puede optimizar rápidamente sus cargas de trabajo y configurar el servidor para lograr el mejor rendimiento. 

Consulte los [conceptos de supervisión](concepts-monitoring.md) para obtener más información.

## <a name="migration"></a>Migración

El servicio ejecuta la versión Community de MySQL. Esto ofrece una compatibilidad total con las aplicaciones y requiere un costo mínimo de refactorización para migrar la aplicación existente desarrollada en el motor de MySQL a un servicio de servidor único. La migración al servidor único se puede realizar mediante una de las siguientes opciones:

- **Volcado y restauración**: en el caso de las migraciones sin conexión en las que los usuarios pueden permitirse un tiempo de inactividad, el volcado y la restauración mediante herramientas de la comunidad como mysqldump/mydumper pueden proporcionar una manera más rápida de migrar. Consulte Migración mediante volcado y restauración para más información. 
- **Azure Database Migration Service**: puede aprovechar [Azure Database Migration Service](https://docs.microsoft.com/azure/dms/tutorial-mysql-azure-mysql-online) para las migraciones sin problemas y simplificadas a un servidor único con un tiempo de inactividad mínimo. 

## <a name="next-steps"></a>Pasos siguientes
Ahora que ha leído la introducción al modo de implementación del servidor único de Azure Database for MySQL, ya está listo para lo siguiente:

- Creación del primer servidor. 
  - [Creación de un servidor flexible de Azure Database for MySQL mediante Azure Portal](quickstart-create-server-portal.md).
  - [Creación de un servidor flexible de Azure Database for MySQL mediante la CLI de Azure](quickstart-create-server-cli.md)
  - [Administración de un servidor flexible de Azure Database for MySQL mediante la CLI de Azure](how-to-manage-server-portal.md)

- Cree su primera aplicación con su lenguaje preferido:
  - [Python](connect-python.md)
  - [PHP](connect-php.md)
