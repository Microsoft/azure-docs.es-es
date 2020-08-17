---
title: 'Seguridad física para centros de datos de Azure: Microsoft Azure | Microsoft Docs'
description: En el artículo se describe lo que hace Microsoft para proteger los centros de datos de Azure, incluida la infraestructura física, la seguridad y las ofertas de cumplimiento.
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
ms.date: 07/10/2020
ms.author: terrylan
ms.openlocfilehash: 3eac3c9abda74f0414fc97512f011d235d3991f5
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87076326"
---
# <a name="azure-facilities-premises-and-physical-security"></a>Instalaciones de Azure, entornos locales y seguridad física
En este artículo se describe lo que hace Microsoft para proteger la infraestructura de Azure.

## <a name="datacenter-infrastructure"></a>Infraestructura de centros de datos
Azure está formada por una [infraestructura de centros de datos distribuida globalmente](https://azure.microsoft.com/global-infrastructure/) que da soporte a miles de servicios en línea y abarca más de 100 instalaciones de alta seguridad en todo el mundo.

La infraestructura está diseñada para acercar las aplicaciones a usuarios de todo el mundo. De este modo, mantiene la residencia de los datos y ofrece a los clientes opciones muy completas de cumplimiento normativo y resistencia. Azure tiene 58 regiones en todo el mundo y está disponible en 140 países o regiones.

Una región es un conjunto de centros de datos que están conectados entre sí mediante una red masiva y resiliente. La red incluye distribución de contenido, equilibrio de carga, redundancia y [cifrado de capa de vínculo de datos de forma predeterminada](encryption-overview.md#encryption-of-data-in-transit) para todo el tráfico de Azure dentro de una región o que viaje de una región a otra. Con más regiones globales que ningún otro proveedor de servicios en la nube, Azure la flexibilidad de implementar aplicaciones donde sea necesario.

Las regiones de Azure se organizan por zonas geográficas. Una zona geográfica de Azure garantiza que se cumplan los requisitos de residencia, soberanía, cumplimiento normativo y resistencia de los datos dentro de las fronteras geográficas.

Las zonas geográficas permiten a los clientes con necesidades específicas de residencia de datos y cumplimiento normativo mantener sus datos y aplicaciones cerca. Las zonas geográficas son tolerantes a errores hasta el punto de resistir una interrupción total del funcionamiento de una región gracias a su conexión con nuestra infraestructura de red dedicada de alta capacidad.

Las zonas de disponibilidad son ubicaciones separadas físicamente dentro de una región de Azure. Cada zona de disponibilidad consta de uno o varios centros de datos equipados con alimentación, refrigeración y redes independientes. Las zonas de disponibilidad permiten ejecutar aplicaciones críticas con alta disponibilidad y replicación con baja latencia.

La siguiente ilustración muestra cómo la infraestructura global de Azure combina una región con zonas de disponibilidad que estén dentro de los mismos límites de residencia de datos para obtener alta disponibilidad, recuperación ante desastres y copias de seguridad.

![Diagrama que muestra los límites de residencia de datos](./media/physical-security/data-residency-boundary.png)

Los centros de datos distribuidos geográficamente permiten a Microsoft estar más cerca de los clientes, para reducir la latencia de red y permitir conmutación por error y copia de seguridad con redundancia geográfica.

## <a name="physical-security"></a>Seguridad física
Microsoft diseña, crea y opera los centros de datos de forma que se controla estrictamente el acceso físico a las áreas donde se almacenan los datos. Microsoft reconoce la importancia de proteger los datos y se compromete a ayudar a proteger los centros de datos que contienen sus datos. Contamos con toda una división en Microsoft dedicada al diseño, creación y operación de las instalaciones físicas que dan soporte a Azure. Este equipo se dedica a mantener una seguridad física de última generación.

Microsoft adopta un enfoque por capas para la seguridad física con el fin de reducir el riesgo de que los usuarios no autorizados obtengan acceso físico a los datos y los recursos del centro de datos. Los centros de datos administrados por Microsoft tienen numerosas capas de protección: autorización del acceso en el perímetro de la instalación, en el perímetro del edificio, dentro del edificio y en la planta del centro de datos. Las capas de seguridad física son:

- **Solicitud de acceso y aprobación.** Debe solicitar acceso antes de llegar al centro de datos. Se le pedirá que proporcione una justificación comercial válida para su visita, por ejemplo, con fines de cumplimiento o auditoría. Los empleados de Microsoft aprueban las solicitudes según la necesidad de acceso. La autorización por necesidad de acceso ayuda a reducir al mínimo el número de personas necesarias para completar una tarea en los centros de datos. Una vez que Microsoft concede el permiso, solo una persona tiene acceso al área del centro de datos pertinente según la justificación empresarial aprobada. Los permisos se limitan a un determinado período de tiempo y expiran.

- **Perímetro de la instalación.** Cuando llega a un centro de datos, se le pide que vaya a un punto de acceso bien definido. Normalmente, una alta valla de acero y hormigón rodea cada centímetro del perímetro. Hay cámaras alrededor de los centros de datos, con un equipo de seguridad que supervisa los vídeos en todo momento.

- **Entrada al edificio.** La entrada al centro de datos está protegida por profesionales de seguridad que han recibido rigurosos cursos de aprendizaje y comprobaciones de antecedentes. Estos responsables de seguridad patrullan habitualmente el centro de datos y supervisan los vídeos de las cámaras situadas dentro continuamente.

- **Dentro del edificio.** Después de entrar al edificio, debe pasar por una autenticación en dos fases con datos biométricos para continuar avanzando por el centro de datos. Si se valida la identidad, solo puede entrar en la parte del centro de datos a la que se le ha concedido acceso. Puede permanecer allí solo durante el tiempo aprobado.

- **Planta del centro de datos.** Solo se le permite entrar en la planta para la que se le haya autorizado. Deberá pasar por un detector de metales de cuerpo completo. Para reducir el riesgo de que datos no autorizados entren o salgan del centro de datos sin nuestro conocimiento, solo podrán entrar en el centro de datos los dispositivos aprobados. Además, hay cámaras de vídeo que supervisan la parte frontal y posterior de cada bastidor de servidores. Al salir de la planta del centro de datos, debe pasar de nuevo por el arco de detección de metales para todo el cuerpo. Para salir del centro de datos, deberá pasar por un examen de seguridad adicional.

Microsoft exige que los visitantes entreguen las tarjetas al salir de cualquier instalación de Microsoft.

## <a name="physical-security-reviews"></a>Revisiones de la seguridad física
Periódicamente se realizan revisiones de la seguridad física de las instalaciones para garantizar que los centros de datos aborden correctamente los requisitos de seguridad de Azure. El personal del proveedor de hospedaje del centro de datos no proporciona administración de servicios de Azure. El personal no puede iniciar sesión en los sistemas de Azure y no tiene acceso físico a la sala de colocación ni a las jaulas de Azure.

## <a name="data-bearing-devices"></a>Dispositivos que contienen datos
Microsoft usa los procedimientos recomendados y una solución de borrado que [cumple la norma NIST 800-88](https://csrc.nist.gov/publications/detail/sp/800-88/archive/2006-09-01). En el caso de las unidades de disco duro que no se pueden borrar, se usa un proceso de destrucción que las destruye y hace que sea imposible recuperar la información. Este proceso de destrucción puede ser desintegración, triturado, pulverización o incineración. Los métodos de eliminación se determinan en función del tipo de recurso. Se conservan registros de la destrucción.  

## <a name="equipment-disposal"></a>Eliminación de equipos
Al final del ciclo de vida de un sistema, el personal operativo de Microsoft sigue unos rigurosos procedimientos de control de los datos y de eliminación del hardware que garantizan que ningún elemento del hardware que contenga sus datos estará disponible para terceros que no son de confianza. Se usa un enfoque de borrado seguro con las unidades de disco duro que lo admitan. En el caso de unidades de disco duro que no se pueden borrar, se usa un proceso de destrucción que las destruye y hace que sea imposible recuperar la información. Este proceso de destrucción puede ser desintegración, triturado, pulverización o incineración. Los métodos de eliminación se determinan en función del tipo de recurso. Se conservan registros de la destrucción. Todos los servicios de Azure usan servicios aprobados de almacenamiento de elementos multimedia y administración de desechos.

## <a name="compliance"></a>Cumplimiento normativo
La infraestructura de Azure está diseñada y se administra para satisfacer un amplio conjunto de normas internacionales y específicas del sector, tales como ISO 27001, HIPAA, FedRAMP, SOC 1 y SOC 2. También se cumplen los estándares específicos del país o la región, incluidos IRAP en Australia, G-Cloud en Reino Unido y MTCS en Singapur. Auditorías de terceros rigurosas, como las del Instituto Británico de Normalización, confirman la observancia de los estrictos controles de seguridad que estos estándares exigen.

Para ver una lista completa de normas de cumplimiento que observa Azure, consulte las [ofertas de cumplimiento](https://www.microsoft.com/trustcenter/compliance/complianceofferings).

## <a name="next-steps"></a>Pasos siguientes
Para más información sobre lo que Microsoft hace para ayudar a proteger la infraestructura de Azure, consulte:

- [Disponibilidad de la infraestructura de Azure](infrastructure-availability.md)
- [Componentes y límites del sistema de información de Azure](infrastructure-components.md)
- [Arquitectura de red de Azure](infrastructure-network.md)
- [Red de producción de Azure](production-network.md)
- [Características de seguridad de Azure SQL Database](infrastructure-sql.md)
- [Operaciones de producción y administración de Azure](infrastructure-operations.md)
- [Supervisión de la infraestructura de Azure](infrastructure-monitoring.md)
- [Integridad de la infraestructura de Azure](infrastructure-integrity.md)
- [Protección de datos de cliente de Azure](protection-customer-data.md)


