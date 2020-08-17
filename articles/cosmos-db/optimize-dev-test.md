---
title: Optimización para el desarrollo y pruebas en Azure Cosmos DB
description: En este artículo se explican las opciones gratuitas que ofrece Azure Cosmos DB para el desarrollo y pruebas del servicio.
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 04/27/2020
ms.openlocfilehash: ea5d975b73afcf03ad97bafd1c6f68f2b55263c2
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87084728"
---
# <a name="optimize-development-and-testing-cost-in-azure-cosmos-db"></a>Optimización del coste de desarrollo y pruebas en Azure Cosmos DB

En este artículo se describen las diferentes opciones para usar Azure Cosmos DB para desarrollo y pruebas de forma gratuita, así como técnicas para optimizar el costo en las cuentas de desarrollo o prueba.

## <a name="azure-cosmos-db-emulator-locally-downloadable-version"></a>Emulador de Azure Cosmos DB (versión descargable de forma local)

El [emulador de Azure Cosmos DB](local-emulator.md) es una versión que se puede descargar de forma local y que imita el servicio en la nube de Azure Cosmos DB. Puede escribir y probar el código que usan las API de Azure Cosmos DB incluso si no tiene conexión de red y sin que ello conlleve ningún costo. El emulador de Azure Cosmos DB proporciona un entorno local para propósitos de desarrollo con alta fidelidad al servicio en la nube. Gracias a él, puede desarrollar y probar su aplicación localmente y sin necesidad de crear una suscripción a Azure. Cuando esté listo para implementar su aplicación en la nube, actualice la cadena de conexión para conectarse al punto de conexión de Azure Cosmos DB en la nube; no es necesario que realice otras modificaciones. También puede [configurar una canalización de CI/CD con la tarea de compilación del emulador de Azure Cosmos DB](tutorial-setup-ci-cd.md) en Azure DevOps para ejecutar pruebas. Para comenzar, consulte el artículo referente al [emulador de Azure Cosmos DB](local-emulator.md).

## <a name="azure-cosmos-db-free-tier"></a>Nivel Gratis de Azure Cosmos DB

El nivel Gratis de Azure Cosmos DB facilita la introducción, el desarrollo y la prueba de las aplicaciones, o incluso la ejecución de pequeñas cargas de trabajo de producción de forma gratuita. Cuando el nivel Gratis esté habilitado en una cuenta, obtendrá las primeras 400 RU/s y 5 GB de almacenamiento de la cuenta gratis. También puede crear una base de datos de rendimiento compartida con 25 contenedores que compartan 400 RU/s en el nivel de base de datos, todo ello incluido en el nivel Gratis (limitar cinco bases de datos de rendimiento compartido en una cuenta de nivel Gratis). Al usar el nivel gratis, si aprovisiona una base de datos compartida con un rendimiento mínimo de 400 RU/s, todos los contenedores de esa base de datos pueden compartir el rendimiento. Las bases de datos nuevas con rendimiento compartido o los contenedores con un rendimiento dedicado se facturan según los precios normales.

El nivel Gratis se mantiene indefinidamente durante la vigencia de la cuenta e incluye todas las [ventajas y características](introduction.md#key-benefits) de una cuenta de Azure Cosmos DB normal, incluidos el almacenamiento ilimitado y el rendimiento (RU/s), los Acuerdos de Nivel de Servicio, la alta disponibilidad, la distribución global llave en mano en todas las regiones de Azure y mucho más. Puede tener una cuenta de nivel Gratis por cada suscripción de Azure y debe optar por tenerla al crear la cuenta. Para empezar, [cree una cuenta en Azure Portal con el nivel gratis habilitado](create-cosmosdb-resources-portal.md) o use una [plantilla de ARM](manage-sql-with-resource-manager.md#free-tier). Consulte la [página de precios](https://azure.microsoft.com/pricing/details/cosmos-db/) para más información.

## <a name="try-azure-cosmos-db-for-free"></a>Pruebe gratis Azure Cosmos DB

La opción para [probar Azure Cosmos DB gratis](https://azure.microsoft.com/try/cosmosdb/) es una experiencia sin costo alguno que le permite experimentar con Azure Cosmos DB en la nube sin registrarse para obtener una cuenta de Azure ni utilizar una tarjeta de crédito. Las cuentas de prueba de Azure Cosmos DB están disponibles por tiempo limitado (30 días actualmente). Aún así, se pueden renovar en cualquier momento. Las cuentas de prueba de Azure Cosmos DB facilitan la evaluación de Azure Cosmos DB, crear y probar una aplicación o utilizar los artículos de inicio rápido o tutoriales. También puede crear una demostración, realizar pruebas unitarias o incluso crear una cuenta de varias regiones y ejecutar una aplicación en ella sin incurrir en ningún costo. En una cuenta de prueba de Azure Cosmos DB, puede tener una base de datos de rendimiento compartida con un máximo de 25 contenedores y 20 000 RU/s de rendimiento, o un contenedor con hasta 5000 RU/s. Para comenzar, consulte la página [Probar Azure Cosmos DB de forma gratuita](https://azure.microsoft.com/try/cosmosdb/).

## <a name="azure-free-account"></a>Cuenta gratuita de Azure

Azure Cosmos DB se incluye en la [cuenta gratuita de Azure](https://azure.microsoft.com/free), que ofrece créditos y recursos de Azure de forma gratuita durante un período de tiempo determinado. Específicamente para Azure Cosmos DB, esta cuenta gratuita ofrece 5 GB de almacenamiento y 400 RU de rendimiento aprovisionado para todo el año. Esta experiencia permite a cualquier desarrollador probar fácilmente las características de Azure Cosmos DB o integrarlo con otros servicios de Azure sin ningún costo. Junto con la cuenta gratuita de Azure obtendrá un saldo de 200 USD para gastar durante los primeros 30 días. No se le cobrará nada hasta que decida actualizar la suscripción; incluso si comienza a usar los servicios. Para comenzar, visite la página de la [cuenta gratuita de Azure](https://azure.microsoft.com/free).

## <a name="use-shared-throughput-databases"></a>Uso de bases de datos de rendimiento compartido

En una [base de datos de rendimiento compartida](set-throughput.md#set-throughput-on-a-database), todos los contenedores dentro de la base de datos comparten el rendimiento aprovisionado (RU/s) de la base de datos. Por ejemplo, si aprovisiona una base de datos con 400 RU/s y tiene cuatro contenedores, los cuatro contenedores compartirán los 400 RU/s. En un entorno de desarrollo o pruebas, donde se puede tener acceso a cada contenedor con menos frecuencia y, por tanto, requerir un valor inferior al mínimo de 400 RU/s, la colocación de los contenedores en una base de datos de rendimiento compartida puede ayudar a optimizar el costo.

Por ejemplo, supongamos que la cuenta de desarrollo o de prueba tiene cuatro contenedores. Si crea cuatro contenedores con un rendimiento dedicado (mínimo de 400 RU/s), el total de RU/s será de 1600 RU/s. Por el contrario, si crea una base de datos de rendimiento compartida (mínimo 400 RU/s) y coloca los contenedores allí, el total de RU/s será de solo 400 RU/s. En general, las bases de datos de rendimiento compartido son excelentes para escenarios en los que no es necesario un rendimiento garantizado en ningún contenedor individual.  Aprenda más sobre las [bases de datos de rendimiento compartido](set-throughput.md#set-throughput-on-a-database).

## <a name="next-steps"></a>Pasos siguientes

Puede comenzar a usar el emulador o las cuentas gratuitas de Azure Cosmos DB con los siguientes artículos:

* Obtenga más información sobre la [optimización para desarrollo y pruebas](optimize-dev-test.md).
* Obtenga más información sobre [la factura de Azure Cosmos DB](understand-your-bill.md).
* Obtenga más información sobre la [optimización del costo de la capacidad del rendimiento](optimize-cost-throughput.md).
* Obtenga más información sobre la [optimización del costo del almacenamiento](optimize-cost-storage.md).
* Obtenga más información sobre la [optimización del costo de la lectura y la escritura](optimize-cost-reads-writes.md).
* Obtenga más información sobre la [optimización del costo de las consultas](optimize-cost-queries.md).
* Obtenga más información sobre la [optimización del costo de las cuentas de Azure Cosmos de varias regiones](optimize-cost-regions.md).
