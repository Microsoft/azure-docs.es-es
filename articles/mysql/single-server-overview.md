---
title: Introducción a un servidor único de Azure Database for MySQL
description: Obtenga más información acerca del servidor único de Azure Database for MySQL, que es un servicio de base de datos relacional de la nube de Microsoft basado en MySQL Community Edition.
author: savjani
ms.service: mysql
ms.author: pariks
ms.custom: mvc
ms.topic: overview
ms.date: 8/20/2020
ms.openlocfilehash: 38cf9f5d5102fa4cc322b1a9c7134db85de87814
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90945579"
---
# <a name="azure-database-for-mysql-single-server"></a>Servidor único de Azure Database for MySQL

[Azure Database for MySQL](overview.md), con tecnología de MySQL Community Edition, está disponible en dos modos de implementación:
- Servidor único 
- Servidor flexible (versión preliminar)

En este artículo, se proporcionará información general y una introducción a los conceptos básicos del modelo de implementación de servidor único. Para obtener información sobre el modo de implementación de servidor flexible, consulte [Información general de servidor flexible](flexible-server/index.yml).

## <a name="overview"></a>Información general

Servidor único es un servicio de base de datos totalmente administrado con requisitos mínimos para las personalizaciones de la base de datos. La plataforma de servidor único está diseñada para administrar la mayoría de las funciones de administración de bases de datos, como las de aplicación de revisión, copias de seguridad, alta disponibilidad y seguridad con una configuración y un control mínimos para el usuario. La arquitectura está optimizada para proporcionar una disponibilidad del 99,99 % en una zona de disponibilidad única. Es compatible con la versión Community de MySQL 5.6, 5.7 y 8.0. El servicio está disponible con carácter general hoy en muchas [regiones de Azure](https://azure.microsoft.com/global-infrastructure/services/). 

Los servidores únicos son los más adecuados para las aplicaciones nativas en la nube diseñadas para controlar la aplicación de revisiones automatizada sin necesidad de un control granular sobre la programación de aplicación de revisiones ni de una configuración personalizada de MySQL. 

## <a name="high-availability"></a>Alta disponibilidad

El modelo de implementación de servidor único está optimizado para la alta disponibilidad integrada y la elasticidad con un costo reducido. La arquitectura separa el proceso y el almacenamiento. El motor de base de datos se ejecuta en un contenedor de proceso propio, mientras que los archivos de datos residen en Azure Storage. El almacenamiento mantiene tres copias sincrónicas con redundancia local de los archivos de base de datos, lo que garantiza la durabilidad de los datos. 

Durante los eventos de conmutación por error planeados o no planeados, si el servidor deja de funcionar, el servicio mantiene la alta disponibilidad de los servidores mediante el siguiente procedimiento automático:

1. Se aprovisiona un nuevo contenedor de proceso.
2. El almacenamiento con archivos de datos se asigna al nuevo contenedor. 
3. El motor de base de datos MySQL pasa a estar en línea en el nuevo contenedor de proceso.
4. El servicio de puerta de enlace garantiza una conmutación por error transparente, por lo que no se requiere ningún cambio en la aplicación. 
  
Los tiempos de conmutación por error normales oscilan entre los 60 y 120 segundos. El diseño nativo en la nube del servicio de servidor único permite ofrecer una disponibilidad del 99,99 %, lo que elimina el costo de la espera activa pasiva.

El Acuerdo de Nivel de Servicio (SLA) de Azure, con una disponibilidad del 99,99 % líder del sector y con la tecnología de una red global de centros de datos administrados por Microsoft, ayuda a mantener las aplicaciones en funcionamiento de forma ininterrumpida.

:::image type="content" source="media/single-server-overview/1-singleserverarchitecture.png" alt-text="Diagrama conceptual de la arquitectura del servidor único de Azure Database for MySQL"::: 

## <a name="automated-patching"></a>Aplicación de revisiones automatizada 

El servicio aplica automáticamente revisiones al hardware, sistema operativo y motor de base de datos subyacentes. La aplicación de revisión incluye actualizaciones de seguridad y software. En el motor de MySQL, las actualizaciones de versiones secundarias son automáticas y se incluyen como parte del ciclo de aplicación de revisión. No se requiere ninguna acción del usuario ni opciones de configuración para la aplicación de revisión. La frecuencia de la aplicación de revisión es la administra el servicio en función de la importancia de la carga. En general, el servicio sigue una programación mensual de versiones como parte de la integración y publicación continuas. Los usuarios pueden suscribirse a la [notificación de mantenimiento planeado](concepts-monitoring.md) para recibir la notificación del próximo mantenimiento 72 horas antes del evento.

## <a name="automatic-backups"></a>Copias de seguridad automáticas

El servicio de servidor único crea automáticamente copias de seguridad del servidor y las almacena en el almacenamiento con redundancia local o con redundancia geográfica configurado por el usuario. Las copias de seguridad se pueden usar para restaurar el servidor a un momento dado dentro del período de retención de la copia de seguridad. El período de retención predeterminado es siete días. La retención se puede configurar opcionalmente los 35 días. Todas las copias de seguridad se cifran mediante cifrado AES de 256 bits. Consulte [Copias de seguridad](concepts-backup.md) para obtener más información.

## <a name="adjust-performance-and-scale-within-seconds"></a>Ajustar el rendimiento y la escala en cuestión de segundos

El servicio de servidor único está disponible en tres niveles de SKU: Básico, De uso general y Optimizado para memoria. El nivel Básico es el más adecuado para las cargas de trabajo de desarrollo que son de bajo costo y baja simultaneidad. Los niveles De uso general y Optimizado para memoria son los más adecuados para las cargas de trabajo de producción que requieren un alto nivel de simultaneidad, escala y rendimiento predecible. Puede compilar su primera aplicación en una base de datos pequeña por poco dinero al mes y, después, ajustar la escala para satisfacer las necesidades de la solución. El escalado de almacenamiento se realiza en línea y admite el crecimiento automático del almacenamiento. La escalabilidad dinámica permite a la base de datos responder de manera transparente a los cambiantes requisitos de recursos. Solo paga por los recursos que consume. Para obtener más información, consulte los  [planes de tarifa](concepts-service-tiers.md).

## <a name="enterprise-grade-security-compliance-and-governance"></a>Seguridad, cumplimiento y gobernanza empresarial

El servicio de servidor único usa el módulo criptográfico con validación FIPS 140-2 para el cifrado del almacenamiento de los datos en reposo. Se cifran los datos, incluidas las copias de seguridad, y los archivos temporales creados mientras se ejecutan las consultas. El servicio usa el cifrado AES de 256 bits que se incluye en el cifrado de Azure Storage y las claves puede administrarlas el sistema (valor predeterminado) o [el cliente](concepts-data-encryption-mysql.md). El servicio cifra los datos en movimiento con la seguridad de la capa de transporte que se aplica de manera predeterminada. El servicio admite las versiones de TLS 1.2, 1.1 y 1.0 con capacidad para aplicar la [versión de TLS mínima](concepts-ssl-connection-security.md). 

El servicio permite el acceso privado a los servidores mediante el [vínculo privado](concepts-data-access-security-private-link.md) y ofrece una característica de [protección contra amenazas avanzada](concepts-data-access-and-security-threat-protection.md). La protección contra amenazas avanzada detecta las actividades anómalas que indican intentos poco habituales y posiblemente dañinos de acceder a las bases de datos o aprovecharse de ellas.

Además de la autenticación nativa, el servicio de servidor único admite la autenticación de [Azure Active Directory](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-whatis). La autenticación de Azure AD es un mecanismo que permite conectar con los servidores de MySQL mediante identidades definidas y administradas en Azure AD. Con la autenticación de Azure AD, puede administrar las identidades de los usuarios de la base de datos y otros servicios de Azure en una ubicación central, lo que simplifica y centraliza el control de acceso.

Existe un [registro de auditoría](concepts-audit-logs.md) disponible para realizar un seguimiento de todas las actividades en el nivel de base de datos. 

El servicio de servidor único cumple con todas las certificaciones líderes del sector, como FedRAMP, HIPAA y PCI DSS. Para obtener más información sobre la seguridad de plataforma de Azure, vaya al [Centro de confianza de Azure](https://www.microsoft.com/trustcenter/security). 

Para más información acerca de las características de seguridad de Azure Database for MySQL, consulte la [información general sobre seguridad](concepts-security.md).

## <a name="monitoring-and-alerting"></a>Supervisión y alertas

El servicio de servidor único está equipado con características integradas de alertas y supervisión de rendimiento. Todas las métricas de Azure tienen una frecuencia de un minuto y cada métrica proporciona 30 días de historial. Puede configurar alertas en las métricas. El servicio permite configurar registros de consultas lentas e incluye una característica diferenciada de [almacén de consultas](concepts-query-store.md). El Almacén de consultas simplifica la solución de problemas de rendimiento al ayudar a encontrar rápidamente las consultas que tardan más en ejecutarse y consumen más recursos. Con estas herramientas, puede optimizar rápidamente sus cargas de trabajo y configurar el servidor para lograr el mejor rendimiento. Consulte [Supervisión](concepts-monitoring.md) para obtener más información.

## <a name="migration"></a>Migración

El servicio ejecuta la versión Community de MySQL. Esto ofrece una compatibilidad total con las aplicaciones y requiere un costo mínimo de refactorización para migrar la aplicación existente desarrollada en el motor de MySQL a un servicio de servidor único. La migración al servidor único se puede realizar mediante una de las siguientes opciones:

- **Volcado y restauración**: en el caso de las migraciones sin conexión en las que los usuarios pueden permitirse un tiempo de inactividad, el volcado y la restauración mediante herramientas de la comunidad como mysqldump/mydumper pueden proporcionar una manera más rápida de migrar. Consulte [Migración mediante volcado y restauración](concepts-migrate-dump-restore.md) para más información. 
- **Azure Database Migration Service**: puede aprovechar [Azure Database Migration Service](https://docs.microsoft.com/azure/dms/tutorial-mysql-azure-mysql-online) para las migraciones sin problemas y simplificadas a un servidor único con un tiempo de inactividad mínimo. 
- **Replicación de datos de entrada**: este servicio, que depende de la replicación basada en registros binarios, también se puede aprovechar para las migraciones con un tiempo de inactividad mínimo. Los expertos prácticos que buscan más control sobre la migración prefieren la replicación de datos de entrada para conseguir migraciones con un tiempo de inactividad mínimo. Consulte [Replicación de datos de entrada](concepts-data-in-replication.md) para obtener más información.

## <a name="contacts"></a>Contactos
Para cualquier pregunta o sugerencia que pueda tener con respecto al uso de Azure Database for MySQL, envíe un correo electrónico al equipo de Azure Database for MySQL ([@Ask Azure DB for MySQL](mailto:AskAzureDBforMySQL@service.microsoft.com)). Esta dirección de correo electrónico no es un alias de soporte técnico.

Además, tenga en cuenta los siguientes puntos de contacto según corresponda:

- Para ponerse en contacto con el servicio de soporte técnico de Azure, [presente una incidencia de soporte técnico en Azure Portal](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade).
- Para corregir un problema con la cuenta, envíe una [solicitud de soporte técnico](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest) en Azure Portal.
- Para proporcionar comentarios o solicitar nuevas características, cree una entrada mediante [UserVoice](https://feedback.azure.com/forums/597982-azure-database-for-mysql).

## <a name="next-steps"></a>Pasos siguientes
Ahora que ha leído la introducción al modo de implementación del servidor único de Azure Database for MySQL, ya está listo para lo siguiente:

- Creación del primer servidor. 
  - [Create an Azure Database for MySQL server using Azure Portal](quickstart-create-mysql-server-database-using-azure-portal.md) (Creación de un servidor de Azure Database for MySQL mediante Azure Portal)
  - [Create an Azure Database for MySQL server using Azure CLI](quickstart-create-mysql-server-database-using-azure-cli.md) (Creación de un servidor de Azure Database for MySQL mediante la CLI de Azure)
  - [Ejemplos de la CLI de Azure para Azure Database for MySQL](sample-scripts-azure-cli.md)

- Cree su primera aplicación con su lenguaje preferido:
  - [Python](./connect-python.md)
  - [Node.JS](./connect-nodejs.md)
  - [Java](./connect-java.md)
  - [Ruby](./connect-ruby.md)
  - [PHP](./connect-php.md)
  - [.NET (C#)](./connect-csharp.md)
  - [Go](./connect-go.md)
  
