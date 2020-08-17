---
title: Integridad de la infraestructura de Azure
description: Obtenga información acerca de la integridad de la infraestructura de Azure y los pasos que sigue Microsoft para protegerla, como exámenes de virus en compilaciones de componentes de software.
services: security
documentationcenter: na
author: TerryLanfear
manager: barbkess
editor: TomSh
ms.assetid: 61e95a87-39c5-48f5-aee6-6f90ddcd336e
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/06/2018
ms.author: terrylan
ms.openlocfilehash: b0df14dfcf66f53e01989a27a95afa5e19fcb2fe
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87543646"
---
# <a name="azure-infrastructure-integrity"></a>Integridad de la infraestructura de Azure

## <a name="software-installation"></a>Instalación de software
Todos los componentes de la pila de software que se instalan en el entorno de Azure se crean de forma personalizada siguiendo el proceso Ciclo de vida de desarrollo de seguridad (SDL) de Microsoft. Todos los componentes de software, como las imágenes del sistema operativo (SO) y SQL Database, se implementan como parte del proceso de administración de versiones y cambios. El sistema operativo que se ejecuta en todos los nodos es una versión personalizada de Windows Server 2008 o Windows Server 2012. La versión exacta la elige el controlador de tejido (FC) de acuerdo con el rol que pretende que desempeñe el sistema operativo. Además, el sistema operativo host no permite la instalación de ningún componente de software no autorizado.

Algunos componentes de Azure se implementan como clientes de Azure en una máquina virtual invitada que se ejecuta en un sistema operativo invitado.

## <a name="virus-scans-on-builds"></a>Detección de virus en las compilaciones
Las compilaciones de software de Azure, lo que incluye el sistema operativo, deben someterse a una detección de virus con la herramienta antivirus Endpoint Protection. Cada detección de virus crea un registro dentro del directorio de compilación asociado, donde se detalla lo que se examinó y los resultados del examen. El análisis de virus forma parte del código fuente de la compilación de todos los componentes de Azure. El código no pasa a producción hasta que los resultados indican que está limpio y solo si el proceso de detección se ha llevado a cabo correctamente. Si se detecta algún problema, la compilación se bloquea y, después, se envía a los equipos de seguridad de Microsoft Security para identificar dónde entró el código "falso" en la compilación.

## <a name="closed-and-locked-environment"></a>Entorno bloqueado y cerrado
De forma predeterminada, los nodos de infraestructura Azure y las máquinas virtuales invitadas no tienen ninguna cuenta de usuario creada en ellas. Además, las cuentas de administrador predeterminadas de Windows también están deshabilitadas. Los administradores de Azure Live Support pueden, con la autenticación adecuada, iniciar sesión en estas máquinas y administrar la red de producción de Azure para reparaciones de emergencia.

## <a name="azure-sql-database-authentication"></a>Autenticación de Azure SQL Database
Al igual que con cualquier implementación de SQL Server, la administración de cuentas de usuario debe estar estrictamente controlada. Azure SQL Database admite solo la autenticación de SQL Server. Para complementar el modelo de seguridad de datos del cliente, también deben usarse cuentas de usuario con contraseñas seguras y configuradas con derechos específicos.

## <a name="acls-and-firewalls-between-the-microsoft-corporate-network-and-an-azure-cluster"></a>Listas ACL y firewalls entre la red corporativa de Microsoft y un clúster de Azure
Las listas de control de acceso (ACL) y los firewalls entre la plataforma de servicios y la red corporativa de Microsoft protegen las instancias de SQL Database del acceso no autorizado. Además, solo los usuarios de los intervalos de direcciones IP de la red corporativa de Microsoft pueden acceder al punto de conexión de administración de la plataforma Windows Fabric.

## <a name="acls-and-firewalls-between-nodes-in-a-sql-database-cluster"></a>Listas ACL y firewalls entre nodos de un clúster de SQL Database
Como protección adicional, y como parte de la fuerte estrategia de defensa, se han implementado listas ACL y un firewall entre los nodos de un clúster de SQL Database. Toda la comunicación dentro del clúster de la plataforma Windows Fabric, así como todo el código en ejecución, es de confianza.

## <a name="custom-monitoring-agents"></a>Agentes de supervisión personalizados
SQL Database emplea agentes de supervisión personalizados (MA), también conocidos como guardianes, para supervisar el mantenimiento del clúster de SQL Database.

## <a name="web-protocols"></a>Protocolos web

### <a name="role-instance-monitoring-and-restart"></a>Supervisión y reinicio de instancias de rol
Azure garantiza que todos los roles en ejecución (roles web accesibles desde Internet, o roles de trabajo de procesamiento de back-end) implementados se sometan a una supervisión sostenida de su mantenimiento para garantizar que están prestando de manera efectiva y eficiente los servicios para los que han sido aprovisionados. En el caso de que un rol no sea correcto, ya sea por un error crítico en la aplicación hospedada o por un problema de configuración subyacente dentro de la propia instancia del rol, el FC detecta el problema dentro de la instancia de rol e inicia un estado de corrección.

### <a name="compute-connectivity"></a>Conectividad de proceso
Azure garantiza que la aplicación o el servicio implementados sean accesibles a través de protocolos estándar basados en web. Las instancias virtuales de los roles web accesibles desde Internet tendrán conectividad externa a Internet y los usuarios de la web podrán acceder a ellas directamente. Con el fin de proteger la confidencialidad e integridad de las operaciones que los roles de trabajo realizan en nombre de las instancias virtuales del rol web de acceso público, las instancias virtuales de rol de trabajo de procesamiento de back-end tienen conectividad externa a Internet, pero el usuario web externo no puede acceder directamente a ellas.

## <a name="next-steps"></a>Pasos siguientes
Para más información sobre lo que hace Microsoft para proteger la infraestructura de Azure, consulte:

- [Instalaciones de Azure, entornos locales y seguridad física](physical-security.md)
- [Disponibilidad de la infraestructura de Azure](infrastructure-availability.md)
- [Componentes y límites del sistema de información de Azure](infrastructure-components.md)
- [Arquitectura de red de Azure](infrastructure-network.md)
- [Red de producción de Azure](production-network.md)
- [Características de seguridad de Azure SQL Database](infrastructure-sql.md)
- [Operaciones de producción y administración de Azure](infrastructure-operations.md)
- [Supervisión de la infraestructura de Azure](infrastructure-monitoring.md)
- [Protección de datos de cliente de Azure](protection-customer-data.md)
