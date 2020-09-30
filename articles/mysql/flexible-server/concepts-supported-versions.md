---
title: Versiones admitidas en el Servidor flexible de Azure Database for MySQL
description: Obtenga información sobre las versiones del servidor de MySQL que se admiten en el Servidor flexible de Azure Database for MySQL
author: kummanish
ms.author: manishku
ms.service: mysql
ms.topic: conceptual
ms.date: 09/21/2020
ms.openlocfilehash: f29a34fd8916110355c0122fee9db29599a01231
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90931868"
---
# <a name="supported-versions-for-azure-database-for-mysql---flexible-server"></a>Versiones admitidas para Azure Database for MySQL: Servidor flexible


> [!IMPORTANT]
> Actualmente, Azure Database for MySQL: Servidor flexible se encuentra en versión preliminar pública.


El Servidor flexible de Azure Database for MySQL cuenta con tecnología de [MySQL Community Edition](https://www.mysql.com/products/community/), con el motor InnoDB.

MySQL usa el esquema de nomenclatura X.Y.Z. X es la versión principal, Y es la versión secundaria y Z es la versión de corrección de errores. Para más información sobre el esquema, consulte la [documentación de MySQL](https://dev.mysql.com/doc/refman/5.7/en/which-version.html).

> [!NOTE]
> Para determinar la versión de la instancia del servidor MySQL, use el comando `SELECT VERSION();` en el símbolo del sistema de MySQL.

Azure Database for MySQL actualmente admite las siguientes versiones:

## <a name="mysql-version-57"></a>MySQL versión 5.7

Versión de corrección de errores: 5.7.29

El servicio aplica automáticamente revisiones al hardware, sistema operativo y motor de base de datos subyacentes. La revisión incluye actualizaciones de seguridad y software. En el motor de MySQL, las actualizaciones de versión secundaria también se incluyen como parte de la versión de mantenimiento planeado. Los usuarios pueden configurar la programación de la revisión para que la administre el sistema o definir su programación personalizada. Durante la programación de mantenimiento, se aplica la revisión y es posible que el servidor requiera un reinicio como parte del proceso de aplicación de revisiones para completar la actualización. Con la programación personalizada, los usuarios pueden hacer que el ciclo de revisión sea predecible y elegir una ventana de mantenimiento con un impacto mínimo en el negocio. En general, el servicio sigue una programación mensual de versiones como parte de la integración y publicación continuas.

Consulte las [notas de la versión](https://dev.mysql.com/doc/relnotes/mysql/5.7/en/news-5-7-29.html) de MySQL para más información sobre mejoras y correcciones de esta versión.

## <a name="managing-updates-and-upgrades"></a>Administración de actualizaciones
El servicio administra automáticamente la aplicación de partes para las actualizaciones de las versiones de corrección de errores. Por ejemplo, de 5.7.29 a 5.7.30.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
>[Compilación de una aplicación PHP en Windows con MySQL](../../app-service/app-service-web-tutorial-php-mysql.md)<br/>
>[Compilación de una aplicación PHP en Linux con MySQL](../../app-service/containers/tutorial-php-mysql-app.md)<br/>
>[Compilación de una aplicación de Spring basada en Java con MySQL](https://docs.microsoft.com/azure/developer/java/spring-framework/spring-app-service-e2e?tabs=bash)<br/>
