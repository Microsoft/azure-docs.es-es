---
title: Ciclo de vida de soporte técnico del Explorador de Azure Storage | Microsoft Docs
description: Información general sobre la directiva y ciclo de vida del soporte técnico del Explorador de Azure Storage
services: storage
author: MRayermannMSFT
manager: jinglouMSFT
ms.service: storage
ms.topic: article
ms.date: 07/10/2020
ms.author: marayerm
ms.openlocfilehash: 30eaa9b9f8700877607af047e5e64eca65471545
ms.sourcegitcommit: 5a37753456bc2e152c3cb765b90dc7815c27a0a8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87759604"
---
# <a name="azure-storage-explorer-support-lifecycle-and-policy"></a>Ciclo de vida y directiva de soporte técnico del Explorador de Azure Storage

Este documento cubre el ciclo de vida y la directiva de soporte técnico del Explorador de Azure Storage.

## <a name="update-schedule-and-process"></a>Programación de actualizaciones y proceso

Se publican versiones de Explorador de Azure Storage entre cuatro y seis veces al año. Microsoft también puede omitir esta programación para publicar correcciones de problemas críticos.

El Explorador de Storage comprueba las actualizaciones cada hora y las descarga cuando están disponibles. Sin embargo, se requiere la aceptación del usuario para iniciar el proceso de instalación. Si los usuarios decide realizar la actualización en otro momento, pueden comprobar manualmente si hay actualizaciones. En Windows y Linux, los usuarios pueden comprobar si hay actualizaciones al seleccionar **Buscar actualizaciones** desde el menú **Ayuda**. En macOS, puede encontrar la opción**Buscar actualizaciones** en el **menú de aplicación**. Los usuarios también pueden ir directamente a la página de descarga del [Explorador de Storage](https://azure.microsoft.com/features/storage-explorer/) para descargar e instalar la versión más reciente.

Se recomienda usar siempre las versiones más recientes del Explorador de Storage. Si un problema impide que se actualice a la versión más reciente del Explorador de Storage, abra una incidencia en nuestro [GitHub](https://github.com/microsoft/AzureStorageExplorer).

## <a name="support-lifecycle"></a>Ciclo de vida de soporte técnico

El Explorador de Storage se rige por la [Directiva moderna de ciclo de vida](https://support.microsoft.com/help/30881/modern-lifecycle-policy). Se espera que los usuarios mantengan su instalación de Explorador de Storage actualizada. Mantenerse al día garantiza que los usuarios tienen las más recientes funcionalidades, mejoras de rendimiento, seguridad y confiabilidad del servicio.

A partir de la versión 1.14.1, todas las versiones del Explorador de Storage que tengan más de 12 meses de antigüedad se considerarán no compatibles. Todas las versiones anteriores a 1.14.1 se considerarán no compatibles a partir del 14 de julio de 2021. No se garantiza que las versiones que no son compatibles funcionarán en su totalidad de la manera esperada ni como se diseñaron. Para obtener una lista de todas las versiones, su fecha de lanzamiento y la fecha de finalización del soporte técnico, consulte [Versiones](#releases).

A partir de la versión 1.13.0, se mostrará una alerta en la aplicación cuando una versión esté aproximadamente a un mes de quedarse sin soporte técnico. La alerta anima a los usuarios a actualizar a la versión más reciente del Explorador de Storage. Una vez que una versión ya no es compatible, la alerta en aplicación se mostrará en cada inicio.

## <a name="releases"></a>Versiones

En esta tabla se describe la fecha de lanzamiento y la fecha de finalización del soporte técnico de todas las versiones del Explorador de Azure Storage.

| Versión del Explorador de Storage  | Fecha de la versión       | Fecha de finalización de soporte técnico |
|:-------------------------:|:------------------:|:-------------------:|
| v1.14.2                   | 16 de julio de 2020      | 16 de julio de 2021       |
| v1.14.1                   | 14 de julio de 2020      | 14 de julio de 2021       |
| v1.14.0                   | 24 de junio de 2020      | 14 de julio de 2021       |
| v1.13.1                   | 18 de mayo de 2020       | 14 de julio de 2021       |
| v1.13.0                   | 1 de mayo de 2020        | 14 de julio de 2021       |
| v1.12.0                   | 16 de enero de 2020   | 14 de julio de 2021       |
| v1.11.2                   | 17 de diciembre de 2019  | 14 de julio de 2021       |
| v1.11.1                   | 20 de noviembre de 2019  | 14 de julio de 2021       |
| v1.11.0                   | 4 de noviembre de 2019   | 14 de julio de 2021       |
| v1.10.1                   | 19 de septiembre de 2019 | 14 de julio de 2021       |
| v1.10.0                   | 12 de septiembre de 2019 | 14 de julio de 2021       |
| v1.9.0                    | 1 de julio de 2019       | 14 de julio de 2021       |
| v1.8.1                    | 10 de mayo de 2019       | 14 de julio de 2021       |
| v1.8.0                    | 2 de mayo de 2019        | 14 de julio de 2021       |
| v1.7.0                    | 5 de marzo de 2019      | 14 de julio de 2021       |
| v1.6.2                    | 8 de enero de 2019    | 14 de julio de 2021       |
| v1.6.1                    | 18 de diciembre de 2018  | 14 de julio de 2021       |
| v1.6.0                    | 4 de diciembre de 2018   | 14 de julio de 2021       |
| v1.5.0                    | 29 de octubre de 2018   | 14 de julio de 2021       |
| v1.4.4                    | 15 de octubre de 2018   | 14 de julio de 2021       |
| v1.4.2                    | 24 de septiembre de 2018 | 14 de julio de 2021       |
| v1.4.1                    | 28 de agosto de 2018    | 14 de julio de 2021       |
| v1.3.1                    | 11 de julio de 2018      | 14 de julio de 2021       |
| v1.2.0                    | 12 de junio de 2018      | 14 de julio de 2021       |
| v1.1.0                    | 9 de mayo de 2018        | 14 de julio de 2021       |
| v1.0.0 (y posteriores)        | 16 de abril de 2018     | 14 de julio de 2021       |
