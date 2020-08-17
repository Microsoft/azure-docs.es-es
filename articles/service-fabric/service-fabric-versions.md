---
title: Versiones de clúster compatibles en Azure Service Fabric
description: Obtenga información sobre las versiones de clúster en Azure Service Fabric, incluido un vínculo a las versiones más recientes del blog del equipo de Service Fabric.
ms.topic: troubleshooting
ms.date: 06/15/2020
ms.openlocfilehash: 728d7403ee8e0ac9ccbd69260a718c65389adc43
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87433184"
---
# <a name="supported-service-fabric-versions"></a>Versiones admitidas de Service Fabric

Asegúrese de que el clúster siempre ejecute una versión admitida de Azure Service Fabric. El soporte técnico para la versión anterior de Service Fabric finaliza al menos 60 días después del lanzamiento de la nueva versión. Las versiones nuevas se anuncian en el [blog del equipo de Service Fabric](https://azure.microsoft.com/updates/?product=service-fabric).

En el caso de una versión determinada del entorno de ejecución de Service Fabric, puede usar las versiones especificadas o anteriores de los paquetes del SDK/NuGet. No se admiten las versiones más recientes de los paquetes y pueden presentar problemas con los clústeres más antiguos al establecerlos como destinos, ya que pueden tener cambios de características o protocolos no admitidos por estos entornos.

Consulte los documentos siguientes para ver detalles sobre cómo hacer que el clúster ejecute una versión compatible de Service Fabric:

- [Actualización de un clúster de Azure Service Fabric](service-fabric-cluster-upgrade.md)
- [Actualización de la versión de Service Fabric que se ejecuta en el clúster](service-fabric-cluster-upgrade-windows-server.md)

## <a name="supported-versions"></a>Versiones compatibles

En la tabla siguiente se indican las versiones de Service Fabric y sus fechas de finalización de soporte.

| Entorno de tiempo de ejecución de Service Fabric en el clúster | Se puede actualizar directamente desde la versión del clúster |Versiones del SDK o paquete de NuGet compatibles | Finalización del soporte técnico |
| --- | --- |--- | --- |
| Todas las versiones de clústeres anteriores a 5.3.121 | 5.1.158.* |Versión 2.3 o anterior |20 de enero de 2017 |
| 5.3* | 5.1.158.* |Versión 2.3 o anterior |24 de febrero de 2017 |
| 5.4.* | 5.1.158.* |Versión 2.4 o anterior |10 de mayo de 2017       |
| 5.5.* | 5.4.164.* |Versión 2.5 o anterior |10 de agosto de 2017    |
| 5.6.* | 5.4.164.* |Versión 2.6 o anterior |13 de octubre de 2017   |
| 5.7.* | 5.4.164.* |Versión 2.7 o anterior |15 de diciembre de 2017  |
| 6.0.* | 5.6.205.* |Versión menor o igual que la 2.8 |30 de marzo de 2018     |
| 6.1.* | 5.7.221.* |Versión 3.0 o anterior |15 de julio de 2018      |
| 6.2.* | 6.0.232.* |Versión 3.1 o anterior |26 de octubre de 2018   |
| 6.3.* | 6.1.480.* |Versión 3.2 o anterior |31 de marzo de 2019  |
| 6.4.* | 6.2.301.* |Versión 3.3 o anterior |15 de septiembre de 2019 |
| 6.5.* | 6.4.617.* |Versión 3.4 o anterior |1 de agosto de 2020 |
| 7.0.466.* | 6.4.664.* |Versión 4.0 o anterior|1 de agosto de 2020  |
| 7.0.466.* | 6.5.* |Versión 4.0 o anterior|1 de agosto de 2020 |
| 7.0.470.* | 7.0.466.* |Versión 4.0 o anterior |1 de agosto de 2020  |
| 7.0.472.* | 7.0.466.* |Versión 4.0 o anterior |1 de agosto de 2020  |
| 7.1.409.* | 7.0.466.* |Versión 4.1 o anterior |Versión actual; por lo tanto, sin fecha de finalización |
| 7.1.417.* | 7.0.466.* |Versión 4.1 o anterior |Versión actual; por lo tanto, sin fecha de finalización |
| 7.1.428.* | 7.0.466.* |Versión 4.1 o anterior |Versión actual; por lo tanto, sin fecha de finalización |

## <a name="supported-operating-systems"></a>Sistemas operativos admitidos

En la tabla siguiente se indican los sistemas operativos compatibles para las versiones de Service Fabric admitidas.

| Sistema operativo | Versión más antigua admitida de Service Fabric |
| --- | --- |
| Windows Server 2012 R2 | Todas las versiones |
| Windows Server 2016 | Todas las versiones |
| Windows Server 1709 | 6.0 |
| Windows Server 1803 | 6.4 |
| Windows Server 1809 | 6.4.654.9590 |
| Windows Server 2019 | 6.4.654.9590 |
| Linux Ubuntu 16.04 | 6.0 |
| Linux Ubuntu 18.04 | 7.1 |

## <a name="supported-version-names"></a>Nombres de versiones admitidas

En la tabla siguiente se indican los nombres de versión de Service Fabric y sus números de versión correspondientes.

| Nombre de versión | Número de versión de Windows | Número de versión de Linux |
| --- | --- | --- |
| 5.3 RTO | 5.3.121.9494 | N/D |
| 5.3 CU1 | 5.3.204.9494 | N/D |
| 5.3 CU2 | 5.3.301.9590 | N/D |
| 5.3 CU3 | 5.3.311.9590 | N/D |
| 5.4 CU2 | 5.4.164.9494 | N/D |
| 5.5 CU1 | 5.5.216.0    | N/D |
| 5.5 CU2 | 5.5.219.0    | N/D |
| 5.5 CU3 | 5.5.227.0    | N/D |
| 5.5 CU4 | 5.5.232.0    | N/D |
| 5.6 RTO | 5.6.204.9494 | N/D |
| 5.6 CU2 | 5.6.210.9494 | N/D |
| 5.6 CU3 | 5.6.220.9494 | N/D |
| 5.7 RTO | 5.7.198.9494 | N/D |
| 5.7 CU4 | 5.7.221.9494 | N/D |
| 6.0 RTO | 6.0.211.9494 | 6.0.120.1 |
| 6.0 CU1 | 6.0.219.9494 | 6.0.127.1 |
| 6.0 CU2 | 6.0.232.9494 | 6.0.133.1 |
| 6.1 CU1 | 6.1.456.9494 | 6.1.183.1 |
| 6.1 CU2 | 6.1.467.9494 | 6.1.185.1 |
| 6.1 CU3 | 6.1.472.9494 | N/D |
| 6.1 CU4 | 6.1.480.9494 | 6.1.187.1 |
| 6.2 RTO | 6.2.269.9494 | 6.2.184.1 | 
| 6.2 CU1 | 6.2.274.9494 | 6.2.191.1 |
| 6.2 CU2 | 6.2.283.9494 | 6.2.194.1 |
| 6.2 CU3 | 6.2.301.9494 | 6.2.199.1 |
| 6.3 RTO | 6.3.162.9494 | 6.3.119.1 |
| 6.3 CU1 | 6.3.176.9494 | 6.3.124.1 |
| 6.3 CU1 | 6.3.187.9494 | 6.3.129.1 |
| 6.4 RTO | 6.4.617.9590 | 6.4.625.1 |
| 6.4 CU2 | 6.4.622.9590 | N/D |
| 6.4 CU3 | 6.4.637.9590 | 6.4.634.1 |
| 6.4 CU4 | 6.4.644.9590 | 6.4.639.1 |
| 6.4 CU5 | 6.4.654.9590 | 6.4.649.1 |
| 6.4 CU6 | 6.4.658.9590 | N/D |
| 6.4 CU7 | 6.4.664.9590 | 6.4.661.1 |
| 6.4 CU8 | 6.4.670.9590 | N/D |
| 6.5 RTO | 6.5.639.9590 | 6.5.435.1 |
| 6.5 CU1 | 6.5.641.9590 | 6.5.454.1 |
| 6.5 CU2 | 6.5.658.9590 | 6.5.460.1 |
| 6.5 CU3 | 6.5.664.9590 | 6.5.466.1 |
| 6.5 CU5 | 6.5.676.9590 | 6.5.467.1 |
| 7.0 RTO | 7.0.457.9590 | 7.0.457.1 |
| 7.0 CU2 | 7.0.464.9590 | 7.0.464.1 |
| 7.0 CU3 | 7.0.466.9590 | 7.0.465.1 |
| 7.0 CU4 | 7.0.470.9590 | 7.0.469.1 |
| 7.0 CU6 | 7.0.472.9590 | 7.0.471.1 |
| 7.1 RTO | 7.1.409.9590 | 7.1.410.1 |
| 7.1 CU1 | 7.1.417.9590 | 7.1.418.1 |
| 7.1 CU2 | 7.1.428.9590 | 7.1.428.1 |

