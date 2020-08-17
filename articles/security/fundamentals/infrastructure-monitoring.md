---
title: Supervisión de la infraestructura de Azure
description: Obtenga información sobre los aspectos de la supervisión de la infraestructura de la red de producción de Azure, como el examen de vulnerabilidades.
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
ms.date: 06/28/2018
ms.author: terrylan
ms.openlocfilehash: 7b75c9dc874a41d4221c55a8b00dd12d943e80fc
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87542949"
---
# <a name="azure-infrastructure-monitoring"></a>Supervisión de la infraestructura de Azure   

## <a name="configuration-and-change-management"></a>Configuración y administración de cambios
Azure revisa y actualiza anualmente los valores de configuración y las configuraciones de línea de base de los dispositivos de hardware, software y red. Los cambios se desarrollan, prueban y aprueban antes de pasar al entorno de producción desde un entorno de desarrollo o pruebas.

El equipo de seguridad y cumplimiento de Azure, y también los equipos de servicio, se encarga de revisar las configuraciones de línea de base necesarias para los servicios basados en Azure. Una revisión del equipo de servicio forma parte de las pruebas que tienen lugar antes de la implementación del servicio en producción.

## <a name="vulnerability-management"></a>Administración de vulnerabilidades
La administración de actualizaciones de seguridad ayuda a proteger los sistemas frente a las vulnerabilidades conocidas. Azure utiliza sistemas de implementación integrados para administrar la distribución e instalación de actualizaciones de seguridad de software de Microsoft. Azure también es capaz de aprovechar los recursos de Microsoft Security Response Center (MSRC). MSRC identifica, supervisa, responde y resuelve los incidentes de seguridad y las vulnerabilidades de la nube de manera ininterrumpida.

## <a name="vulnerability-scanning"></a>Examen de vulnerabilidades
El examen de vulnerabilidades se ejecuta en sistemas operativos de servidor, bases de datos y dispositivos de red. Los exámenes de vulnerabilidades se realizan de forma trimestral como mínimo. Azure contrata asesores independientes para realizar pruebas de penetración del límite de Azure. También se llevan a cabo ejercicios de simulacro de ataques regularmente y los resultados se usan para realizar mejoras de seguridad.

## <a name="protective-monitoring"></a>Supervisión de protección
El equipo de seguridad de Azure ha definido los requisitos para una supervisión activa. Los equipos de servicio configuran herramientas de supervisión activas según estos requisitos. Las herramientas de supervisión activas incluyen Microsoft Monitoring Agent (MMA) y System Center Operations Manager. Estas herramientas están configuradas para proporcionar alertas de tiempo al personal de seguridad de Azure en situaciones que requieren acción inmediata.

## <a name="incident-management"></a>Administración de incidentes
Microsoft implementa un proceso de administración de incidentes de seguridad para facilitar una respuesta coordinada frente a incidentes, en caso de que estos se produzcan.

Si Microsoft detecta cualquier acceso no autorizado a los datos de un cliente almacenados en su equipo o en sus instalaciones, o si detecta un acceso no autorizado a dichos equipos o instalaciones que provoca la pérdida, revelación o alteración de los datos del cliente, Microsoft lleva a cabo las acciones siguientes:

- Notifica inmediatamente al cliente el incidente de seguridad.
- Investiga rápidamente el incidente de seguridad y proporciona al cliente información detallada sobre dicho incidente.
- Toma medidas razonables e inmediatas para mitigar los efectos y minimizar el daño producido como consecuencia del incidente de seguridad.

Se ha establecido un marco de administración de incidentes con roles definidos y responsabilidades asignadas. El equipo de administración de incidentes de seguridad de Azure es responsable de administrar los incidentes de seguridad, solicitando la implicación de equipos especialistas en los casos que sea necesario. Los responsables de operaciones de Azure están encargados de supervisar la investigación y resolución de los incidentes de seguridad y privacidad.

## <a name="next-steps"></a>Pasos siguientes
Para más información sobre lo que hace Microsoft para proteger la infraestructura de Azure, consulte:

- [Instalaciones de Azure, entornos locales y seguridad física](physical-security.md)
- [Disponibilidad de la infraestructura de Azure](infrastructure-availability.md)
- [Componentes y límites del sistema de información de Azure](infrastructure-components.md)
- [Arquitectura de red de Azure](infrastructure-network.md)
- [Red de producción de Azure](production-network.md)
- [Características de seguridad de Azure SQL Database](infrastructure-sql.md)
- [Operaciones de producción y administración de Azure](infrastructure-operations.md)
- [Integridad de la infraestructura de Azure](infrastructure-integrity.md)
- [Protección de datos de cliente de Azure](protection-customer-data.md)
