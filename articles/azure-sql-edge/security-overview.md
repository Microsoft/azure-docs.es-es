---
title: Protección de Azure SQL Edge
description: Información sobre la seguridad en Azure SQL Edge
keywords: SQL Edge, seguridad
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 09/22/2020
ms.openlocfilehash: 17e3e8dca1c03f9783c0ca94350bb8a4ba5aca64
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932134"
---
# <a name="securing-azure-sql-edge"></a>Protección de Azure SQL Edge

Con la mayor frecuencia de adopción de IoT y de proceso perimetral en los distintos sectores, existe un aumento en el número de dispositivos y de datos generados desde estos dispositivos. El mayor volumen de datos y número de puntos de conexión de dispositivo supone un reto importante en cuanto a la seguridad de los datos y los dispositivos. 

Azure SQL Edge ofrece varias características y funcionalidades que facilitan la protección de los datos de IoT en las bases de datos de SQL Server. Azure SQL Edge se ha compilado con el mismo motor de SQL que impulsa Microsoft SQL Server y Azure SQL, comparten las mismas funcionalidades de seguridad, lo que facilita la transmisión de las directivas y prácticas de seguridad de la nube al dispositivo perimetral.

Al igual que con Microsoft SQL Server y Azure SQL, proteger las implementaciones de Azure SQL Edge conlleva una serie de pasos que afectan a cuatro áreas: la plataforma, la autenticación, los objetos (incluidos los datos) y las aplicaciones que acceden al sistema. 

## <a name="platform-and-system-security"></a>Seguridad de la plataforma y del sistema

La plataforma de Azure SQL Edge incluye el host físico de Docker, el sistema operativo del host y los sistemas de red que conectan el dispositivo físico a las aplicaciones y los clientes. 

La implementación de la seguridad de la plataforma comienza por mantener a los usuarios no autorizados fuera de la red. Algunos de los procedimientos recomendados incluyen:
- La implementación de reglas de firewall para garantizar la directiva de seguridad de la organización. 
- Garantiza que el sistema operativo del dispositivo físico tiene aplicadas todas las actualizaciones de seguridad más recientes. 
- La especificación y la restricción de los puertos de host que se usan para Azure SQL Edge.
- Garantiza que el control de acceso adecuado se aplica a todos los volúmenes de datos que hospedan datos de Azure SQL Edge. 

Para más información sobre los protocolos de red de Azure SQL Edge y los puntos de conexión TDS, consulte [Protocolos de red y puntos de conexión TDS](https://docs.microsoft.com//previous-versions/sql/sql-server-2008-r2/ms191220(v=sql.105)).

## <a name="authentication-and-authorization"></a>Autenticación y autorización 

### <a name="authentication"></a>Authentication  
La autenticación es el proceso por el cual se demuestra que el usuario es quien dice ser. Azure SQL Edge actualmente solo admite el mecanismo `SQL Authentication`.

- *Autenticación de SQL*:

    hace referencia a la autenticación de un usuario al conectarse a Azure SQL Edge con el nombre de usuario y la contraseña. Debe especificarse la contraseña de inicio de sesión de SQL **sa** durante la implementación de SQL Edge. Después de eso, pueden crearse inicios de sesión SQL y usuarios adicionales mediante el administrador del servidor, lo que permite a los usuarios conectarse usando el nombre de usuario y contraseña.

    Para más información sobre cómo crear y administrar inicios de sesión y usuarios en SQL Edge, consulte [Creación de un inicio de sesión](https://docs.microsoft.com/sql/relational-databases/security/authentication-access/create-a-login) y [Creación de un usuario de la base de datos](https://docs.microsoft.com/sql/relational-databases/security/authentication-access/create-a-database-user).

### <a name="authorization"></a>Autorización   

La autorización hace referencia a los permisos asignados a un usuario dentro de una base de datos de Azure SQL Edge y determina lo que este puede hacer. Los permisos se controlan mediante la adición de cuentas de usuario a [roles de base de datos](https://docs.microsoft.com/sql/relational-databases/security/authentication-access/database-level-roles) y la asignación de permisos de nivel de base de datos a estos roles o concediendo al usuario determinados [permisos de nivel de objeto](https://docs.microsoft.com/sql/relational-databases/security/permissions-database-engine). Para más información, consulte [Inicios de sesión y usuarios](https://docs.microsoft.com/azure/azure-sql/database/logins-create-manage).

Como procedimiento recomendado, cree roles personalizados cuando sea necesario. Agregue usuarios al rol con los privilegios mínimos necesarios para realizar su función de trabajo. No asigne permisos directamente a los usuarios. La cuenta de administrador del servidor es un miembro del rol db_owner integrado, que tiene amplios permisos y se debe conceder solo a pocos usuarios con responsabilidades administrativas. En las aplicaciones, use [EXECUTE AS](https://docs.microsoft.com/sql/t-sql/statements/execute-as-clause-transact-sql) para especificar el contexto de ejecución del módulo llamado o use [Roles de la aplicación](https://docs.microsoft.com/sql/relational-databases/security/authentication-access/application-roles) con permisos limitados. Esta práctica garantiza que la aplicación que se conecta a la base de datos tenga los privilegios mínimos necesarios para la aplicación. Seguir estos procedimientos recomendados también fomenta la separación de responsabilidades.

## <a name="database-object-security"></a>Seguridad de los objetos de base de datos

Las entidades de seguridad son los individuos, grupos y procesos que tienen acceso a SQL Edge. Los "elementos protegibles" son el servidor, la base de datos y los objetos incluidos en la base de datos. Cada uno de estos elementos dispone de un conjunto de permisos que pueden configurarse para reducir el área expuesta. En la tabla siguiente se incluye información sobre las entidades de seguridad y los elementos protegibles.

|Para información acerca de|Vea|  
|---------------------------|---------|  
|Usuarios, roles y procesos de servidor y base de datos|[Motor de base de datos principal](https://docs.microsoft.com/sql/relational-databases/security/authentication-access/principals-database-engine)|  
|Seguridad de objetos de servidor y base de datos|[Elementos protegibles](https://docs.microsoft.com/sql/relational-databases/security/securables)|
| &nbsp; | &nbsp; |

### <a name="encryption-and-certificates"></a>Cifrado y certificados  
 
El cifrado no resuelve los problemas de control de acceso. Sin embargo, mejora la seguridad debido a que limita la pérdida de datos, incluso en el caso poco probable de que se superen los controles de acceso. Por ejemplo, si el equipo host de base de datos no está configurado correctamente y un usuario malintencionado obtiene datos confidenciales, como números de tarjetas de crédito, esa información robada podría resultar inservible si está cifrada. La tabla siguiente contiene más información sobre el cifrado Azure SQL Edge.  
  
|Para información acerca de|Vea|  
|---------------------------|---------|  
|Implementar conexiones seguras|[Cifrado de conexiones](https://docs.microsoft.com/sql/linux/sql-server-linux-encrypted-connections)|  
|Funciones de cifrado|[Funciones de cifrado &#40;Transact-SQL&#41;](https://docs.microsoft.com/sql/t-sql/functions/cryptographic-functions-transact-sql)|
|Cifrado de datos en reposo|[Cifrado de datos transparente](https://docs.microsoft.com/sql/relational-databases/security/encryption/transparent-data-encryption)|
|Always Encrypted|[Always Encrypted](https://docs.microsoft.com/sql/relational-databases/security/encryption/always-encrypted-database-engine)|
| &nbsp; | &nbsp; |

> [!NOTE]
> Las limitaciones de seguridad descritas para [SQL Server en Linux](https://docs.microsoft.com/sql/linux/sql-server-linux-security-overview) también se aplican a Azure SQL Edge. 


> [!NOTE]
> Azure SQL Edge no incluye la utilidad mssql-conf. Todas las configuraciones, incluida la configuración relacionada con el cifrado, deben realizarse mediante el [archivo mssql.conf](configure.md#configure-by-using-an-mssqlconf-file) o [variables de entorno](configure.md#configure-by-using-environment-variables). 


De forma similar a Azure SQL y Microsoft SQL Server, Azure SQL Edge proporciona el mismo mecanismo de creación y usar certificados con el fin de mejorar la seguridad de los objetos y las conexiones. Para más información, consulte [CREATE CERTIFICATE (TRANSACT-SQL)](https://docs.microsoft.com/sql/t-sql/statements/create-certificate-transact-sql).


## <a name="application-security"></a>Seguridad de la aplicación

### <a name="client-programs"></a>Programas de cliente

Los procedimientos recomendados de seguridad de Azure SQL Edge incluyen la escritura de aplicaciones cliente seguras. Para obtener más información sobre cómo proteger las aplicaciones cliente en el nivel de red, vea [Client Network Configuration](https://docs.microsoft.com/sql/database-engine/configure-windows/client-network-configuration).

### <a name="sql-server-security-catalog-views-and-functions"></a>Funciones y vistas de catálogo de seguridad de SQL Server  
 La información de seguridad se expone en varias vistas y funciones que se optimizan en cuanto a rendimiento y utilidad. En la tabla siguiente se incluye más información acerca de las funciones y vistas de seguridad.  
  
|Funciones y vistas|Vínculos|  
|---------------------------|---------|  
|Vistas de catálogo de seguridad que devuelven información sobre los permisos de nivel de base de datos y de servidor, entidades de seguridad, roles, etc. También hay vistas de catálogo que proporcionan información acerca de las claves de cifrado, los certificados y las credenciales.|[Vistas de catálogo de seguridad &#40;Transact-SQL&#41;](https://docs.microsoft.com/sql/relational-databases/system-catalog-views/security-catalog-views-transact-sql)|  
|Funciones de seguridad, que devuelven información sobre el usuario, los permisos y los esquemas actuales.|[Funciones de seguridad &#40;Transact-SQL&#41;](https://docs.microsoft.com/sql/t-sql/functions/security-functions-transact-sql)|  
|Vistas de administración dinámica de la seguridad.|[Funciones y vistas de administración dinámica relacionadas con la seguridad &#40;Transact-SQL&#41;](https://docs.microsoft.com/sql/relational-databases/system-dynamic-management-views/security-related-dynamic-management-views-and-functions-transact-sql)|  
| &nbsp; | &nbsp; |

### <a name="auditing"></a>Auditoría 

Azure SQL Edge proporciona los mismos mecanismos de auditoría que SQL Server. Para más información, consulte [SQL Server Audit (motor de base de datos)](https://docs.microsoft.com/sql/relational-databases/security/auditing/sql-server-audit-database-engine).


## <a name="next-steps"></a>Pasos siguientes

- [Introducción a las características de seguridad](https://docs.microsoft.com/sql/linux/sql-server-linux-security-get-started)
- [Ejecución de Azure SQL Edge como usuario no raíz](configure.md#run-azure-sql-edge-as-non-root-user)
- [Preguntas más frecuentes de Azure Security Center for IoT](https://docs.microsoft.com/azure/asc-for-iot/overview)

