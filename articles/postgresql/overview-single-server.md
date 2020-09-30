---
title: Azure Database for PostgreSQL con la opción Servidor único
description: Proporciona información general sobre Azure Database for PostgreSQL con la opción Servidor único.
author: sunilagarwal
ms.author: sunila
ms.custom: mvc
ms.service: postgresql
ms.topic: overview
ms.date: 09/21/2020
ms.openlocfilehash: 469235957ebe26dd44cc6ce464a68167629099ab
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90945811"
---
# <a name="azure-database-for-postgresql-single-server"></a>Azure Database for PostgreSQL con la opción Servidor único

[Azure Database for PostgreSQL](./overview.md), con tecnología de PostgreSQL Community Edition, está disponible en tres modos de implementación:

- Servidor único
- Servidor flexible (versión preliminar)
- Hiperescala (Citus)

En este artículo, se proporcionará información general y una introducción a los conceptos básicos del modelo de implementación de servidor único. Para obtener más información sobre el modo de implementación de servidor flexible, consulte la [información general sobre el servidor flexible](./flexible-server/overview.md) y la información general sobre Hiperescala (Citus), respectivamente.

## <a name="overview"></a>Información general

Servidor único es un servicio de base de datos totalmente administrado con requisitos mínimos para las personalizaciones de la base de datos. La plataforma de servidor único está diseñada para administrar la mayoría de las funciones de administración de bases de datos, como las de revisión, copias de seguridad, alta disponibilidad y seguridad con una configuración y un control mínimos para el usuario. La arquitectura está optimizada para proporcionar una disponibilidad del 99,99 % en una zona de disponibilidad única. Es compatible con la versión Community de PostgreSQL 9.5, 9.6, 10 y 11. El servicio está disponible con carácter general hoy en muchas [regiones de Azure](https://azure.microsoft.com/global-infrastructure/services/).

Los servidores únicos son los más adecuados para las aplicaciones nativas en la nube diseñadas para controlar la aplicación automatizada de revisiones sin necesidad de un control granular sobre la programación de aplicación de revisión ni de una configuración personalizada de PostgreSQL.

## <a name="high-availability"></a>Alta disponibilidad

El modelo de implementación de servidor único está optimizado para la alta disponibilidad integrada y la elasticidad con un costo reducido. La arquitectura separa el proceso y el almacenamiento. El motor de base de datos se ejecuta en un contenedor de proceso propio, mientras que los archivos de datos residen en Azure Storage. El almacenamiento mantiene tres copias sincrónicas con redundancia local de los archivos de base de datos, lo que garantiza la durabilidad de los datos.

Durante los eventos de conmutación por error planeados o no planeados, si el servidor deja de funcionar, el servicio mantiene la alta disponibilidad de los servidores mediante el siguiente procedimiento automático:

1. Se aprovisiona un nuevo contenedor de proceso.
2. El almacenamiento con archivos de datos se asigna al nuevo contenedor.
3. El motor de base de datos PostgreSQL pasa a estar en línea en el nuevo contenedor de proceso.
4. El servicio de puerta de enlace garantiza una conmutación por error transparente, por lo que no se requiere ningún cambio en la aplicación.
   
 :::image type="content" source="./media/overview/overview-azure-postgres-single-server.png" alt-text="Azure Database for PostgreSQL con la opción Servidor único":::

Los intervalos de tiempo de conmutación por error normales oscilan entre los 60 y 120 segundos. El diseño nativo en la nube del servicio de servidor único permite ofrecer una disponibilidad del 99,99 %, lo que elimina el costo de la espera activa pasiva.

El Acuerdo de Nivel de Servicio (SLA) de Azure, con una disponibilidad del 99,99 % líder del sector y con la tecnología de una red global de centros de datos administrados por Microsoft, ayuda a mantener las aplicaciones en funcionamiento de forma ininterrumpida.

## <a name="automated-patching"></a>Aplicación de revisiones automatizada

El servicio aplica automáticamente revisiones al hardware, sistema operativo y motor de base de datos subyacentes. La revisión incluye actualizaciones de seguridad y software. En el motor de PostgreSQL, las actualizaciones de versiones secundarias son automáticas y se incluyen como parte del ciclo de aplicación de revisión. No se requiere ninguna acción del usuario ni opciones de configuración para la aplicación de revisión. La frecuencia de la aplicación de revisión es la administra el servicio en función de la importancia de la carga. En general, el servicio sigue una programación mensual de versiones como parte de la integración y publicación continuas. Los usuarios pueden suscribirse a la [notificación de mantenimiento planeado]() para recibir la notificación del próximo mantenimiento 72 horas antes del evento.

## <a name="automatic-backups"></a>Copias de seguridad automáticas

El servicio de servidor único crea automáticamente copias de seguridad del servidor y las almacena en el almacenamiento con redundancia local (LRS) o con redundancia geográfica configurado por el usuario. Las copias de seguridad se pueden usar para restaurar el servidor a un momento dado dentro del período de retención de la copia de seguridad. El período de retención predeterminado es siete días. La retención se puede configurar para un total de 35 días. Todas las copias de seguridad se cifran mediante cifrado AES de 256 bits. Consulte [Copias de seguridad](./concepts-backup.md) para obtener más información.

## <a name="adjust-performance-and-scale-within-seconds"></a>Ajustar el rendimiento y la escala en cuestión de segundos

El servicio de servidor único está disponible en tres niveles de SKU: Básico, De uso general y Optimizado para memoria. El nivel Básico es el más adecuado para las cargas de trabajo de desarrollo que son de bajo costo y baja simultaneidad. Los niveles De uso general y Optimizado para memoria son los más adecuados para las cargas de trabajo de producción que requieren un alto nivel de simultaneidad, escala y rendimiento predecible. Puede compilar su primera aplicación en una base de datos pequeña por poco dinero al mes y, después, ajustar la escala para satisfacer las necesidades de la solución. El escalado de almacenamiento se realiza en línea y admite el crecimiento automático del almacenamiento. La escalabilidad dinámica permite a la base de datos responder de manera transparente a los cambiantes requisitos de recursos. Solo paga por los recursos que consume. Vea los [Planes de tarifa]() para obtener más información.

## <a name="enterprise-grade-security-compliance-and-governance"></a>Seguridad, cumplimiento y gobernanza empresarial

El servicio de servidor único usa el módulo criptográfico con validación FIPS 140-2 para el cifrado del almacenamiento de los datos en reposo. Se cifran los datos, incluidas las copias de seguridad, y los archivos temporales creados mientras se ejecutan las consultas. El servicio usa el cifrado AES de 256 bits que se incluye en el cifrado de Azure Storage, y el sistema (valor predeterminado) o [el cliente]() pueden administrar las claves. El servicio cifra los datos en movimiento con la Seguridad de la capa de transporte (SSL/TLS) que se aplica de manera predeterminada. El servicio admite las versiones de TLS 1.2, 1.1 y 1.0 con capacidad para aplicar la [versión de TLS mínima]().

El servicio permite el acceso privado a los servidores mediante Private Link y ofrece la característica Microsoft Defender for Identity. Microsoft Defender for Identity detecta actividades anómalas que indican intentos poco habituales y posiblemente dañinos de acceder a las bases de datos o aprovecharse de ellas.

Además de la autenticación nativa, el servicio de servidor único admite la autenticación de Azure Active Directory. La autenticación de Azure AD es un mecanismo que permite establecer conexión con los servidores de PostgreSQL mediante identidades definidas y administradas en Azure AD. Con la autenticación de Azure AD, puede administrar las identidades de los usuarios de la base de datos y otros servicios de Azure en una ubicación central, lo que simplifica y centraliza el control de acceso.

Hay un [registro de auditoría]() (en versión preliminar) disponible para realizar un seguimiento de todas las actividades en el nivel de base de datos.

El servicio de servidor único cumple con todas las certificaciones líderes del sector, como FedRAMP, HIPAA y PCI DSS. Para obtener más información sobre la seguridad de plataforma de Azure, vaya al [Centro de confianza de Azure]().

Para obtener más información sobre las características de seguridad de Azure Database for PostgreSQL, consulte la [información general sobre seguridad]().

## <a name="monitoring-and-alerting"></a>Supervisión y alertas

El servicio de servidor único está equipado con características integradas de alertas y supervisión de rendimiento. Todas las métricas de Azure tienen una frecuencia de un minuto y cada métrica proporciona 30 días de historial. Puede configurar alertas en las métricas. El servicio permite configurar registros de consultas lentas e incluye la característica diferenciada [Almacén de consultas](./concepts-query-store.md). El Almacén de consultas simplifica la solución de problemas de rendimiento al ayudar a encontrar rápidamente las consultas que tardan más en ejecutarse y consumen más recursos. Con estas herramientas, puede optimizar rápidamente sus cargas de trabajo y configurar el servidor para lograr el máximo rendimiento. Consulte [Supervisión](./concepts-monitoring.md) para obtener más información.

## <a name="migration"></a>Migración

El servicio ejecuta la versión Community de PostgreSQL. Esto ofrece una compatibilidad total con las aplicaciones y requiere un costo mínimo de refactorización para migrar la aplicación existente desarrollada en el motor de PostgreSQL a un servicio de servidor único. La migración al servidor único se puede realizar mediante una de las siguientes opciones:

- **Volcado y restauración**: en el caso de las migraciones sin conexión en las que los usuarios pueden permitirse un tiempo de inactividad, el volcado y la restauración mediante herramientas de la comunidad, como Pg_dump y Pg_restore, pueden proporcionar una manera más rápida de realizar la migración. Consulte [Migración mediante volcado y restauración](https://docs.microsoft.com/azure/postgresql/howto-migrate-using-dump-and-restore) para obtener más información.
- **Azure Database Migration Service**: puede aprovechar Azure Database Migration Service para las migraciones sin problemas y simplificadas a un servidor único con un tiempo de inactividad mínimo. Consulte [DMS a través del portal](https://docs.microsoft.com/azure/dms/tutorial-postgresql-azure-postgresql-online-portal) y [DMS a través de la CLI](https://docs.microsoft.com/azure/dms/tutorial-postgresql-azure-postgresql-online).

## <a name="contacts"></a>Contactos

Para cualquier pregunta o sugerencia que pueda tener con respecto al uso de Azure Database for PostgreSQL, envíe un correo electrónico al equipo de Azure Database for PostgreSQL ([@Ask Azure DB for PostgreSQL](mailto:AskAzureDBforPostgreSQL@service.microsoft.com)). Esta dirección de correo electrónico no es un alias de soporte técnico.

Además, tenga en cuenta los siguientes puntos de contacto según corresponda:

- Para ponerse en contacto con el servicio de soporte técnico de Azure, [presente una incidencia de soporte técnico en Azure Portal](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade).
- Para corregir un problema con la cuenta, envíe una [solicitud de soporte técnico](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest) en Azure Portal.
- Para proporcionar comentarios o solicitar nuevas características, cree una entrada mediante [UserVoice](https://feedback.azure.com/forums/597976-azure-database-for-postgresql).

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha leído la introducción al modo de implementación Servidor único de Azure Database for PostgreSQL, ya está listo para lo siguiente:
- Creación del primer servidor.
  