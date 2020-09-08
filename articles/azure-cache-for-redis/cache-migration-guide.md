---
title: Migración a Azure Cache for Redis
description: Obtenga información sobre cómo migrar la caché existente a Azure Cache for Redis
author: yegu-ms
ms.service: cache
ms.topic: conceptual
ms.date: 07/22/2020
ms.author: yegu
ms.openlocfilehash: 2a95aa9e9fccdb7047c2c0901f4349fecfbab672
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88009586"
---
# <a name="migrate-to-azure-cache-for-redis"></a>Migración a Azure Cache for Redis
En este artículo se describen varios métodos para migrar una instancia de Redis Cache existente que se ejecuta de forma local o en otro servicio en la nube Azure Cache for Redis.

## <a name="migration-scenarios"></a>Escenarios de migración
Las instancias de Redis de código abierto se pueden ejecutar en muchos entornos de proceso. Algunos ejemplos frecuentes son:

- **En el entorno local**: cachés en Redis que se ejecutan en centros de datos privados.
- **Máquinas virtuales basadas en la nube**: cachés en Redis que se ejecutan en máquinas virtuales de Azure, AWS EC2, etc.
- **Servicios de hospedaje**: servicios de Redis administrados, como AWS ElastiCache.
- **Diferentes regiones**: cachés en Redis ubicadas en otra región de Azure.

Si tiene una caché de este tipo, podría moverla a Azure Cache for Redis con interrupciones o tiempo de inactividad mínimos.

## <a name="migration-options"></a>Opciones de migración

Hay diferentes maneras de cambiar de una memoria caché a otra. En función de la ubicación de la caché y del modo en que la aplicación interactúa con ella, un método resultará más útil que los demás. A continuación se detallan algunas de las estrategias de migración que se usan de forma frecuente.

   | Opción       | Ventajas | Desventajas |
   | ------------ | ---------- | ------------- |
   | Crear una nueva caché | La más sencilla de implementar. | Es necesario volver a rellenar los datos en la nueva caché, lo que puede no funcionar con varias aplicaciones. |
   | Exportar e importar datos mediante un archivo RDB | Es compatible con cualquier caché en Redis en general. | Algunos datos pueden perderse si se escriben en la caché existente después de generar el archivo RDB. | 
   | Doble escritura de datos en dos cachés | Sin pérdida de datos ni tiempo de inactividad. No se interrumpen las operaciones de la caché existente. Pruebas más sencillas de la nueva caché. | Necesita dos cachés durante un período de tiempo prolongado. | 
   | Migración de datos mediante programación | Control total sobre cómo se mueven los datos. | Requiere código personalizado. | 

### <a name="create-a-new-azure-cache-for-redis"></a>Creación de una nueva instancia de Azure Redis Cache

Técnicamente, esta estrategia no es una migración. Si la pérdida de datos no es un problema, la manera más sencilla de pasar a Azure Cache for Redis consiste en crear una instancia de caché y conectar la aplicación a ella. Por ejemplo, si usa Redis como una caché de búsqueda de registros de base de datos, puede volver a generar fácilmente la memoria caché desde cero.

Los pasos generales para implementar esta opción son los siguientes:

1. Cree una nueva instancia de Azure Cache for Redis.

2. Actualice la aplicación para que use la nueva instancia.

3. Elimine la instancia de Redis anterior.

### <a name="export-data-to-an-rdb-file-and-import-it-into-azure-cache-for-redis"></a>Exportación de datos a un archivo RDB e importación a Azure Cache for Redis

La instancia de Redis de código abierto define un mecanismo estándar para tomar una instantánea del conjunto de datos en memoria de la caché y guardarla en un archivo. Otra caché en Redis puede leer este archivo, denominado RDB. [El nivel Premium de Azure Cache for Redis](cache-overview.md#service-tiers) admite la importación de datos en una instancia de caché mediante archivos RDB. Puede usar un archivo RDB para transferir datos de una caché existente a Azure Cache for Redis.

> [!IMPORTANT]
> El formato de archivo RDB puede presentar cambios entre versiones de Redis y podría no mantener la compatibilidad con versiones anteriores. La versión de Redis de la caché desde la que realizará la exportación debe ser igual o menor que la versión proporcionada por Azure Cache for Redis.
>

Los pasos generales para implementar esta opción son los siguientes:

1. Cree una nueva instancia de Azure Cache for Redis en el nivel Premium con un tamaño igual (o mayor) que la caché existente.

2. Guarde una instantánea de la caché en Redis existente. Puede [configurar Redis para guardar instantáneas](https://redis.io/topics/persistence) periódicamente o ejecutar el proceso manualmente mediante los comandos [SAVE](https://redis.io/commands/save) o [BGSAVE](https://redis.io/commands/bgsave). De forma predeterminada, el archivo RDB se denomina "dump.rdb" y se ubica en la ruta de acceso especificada en el archivo de configuración *redis.conf*.

    > [!NOTE]
    > Si va a migrar datos dentro de Azure Cache for Redis, consulte [estas instrucciones sobre cómo exportar un archivo RDB](cache-how-to-import-export-data.md) o use el [cmdlet de exportación de PowerShell](https://docs.microsoft.com/powershell/module/azurerm.rediscache/export-azurermrediscache?view=azurermps-6.13.0&viewFallbackFrom=azurermps-6.4.0) en su lugar.
    >

3. Copie el archivo RDB a una cuenta de almacenamiento de Azure en la región en la que se encuentra la nueva caché. Puede usar AzCopy para esta tarea.

4. Importe el archivo RDB en la nueva caché según estas [instrucciones de importación](cache-how-to-import-export-data.md) o use el [cmdlet de importación de PowerShell](https://docs.microsoft.com/powershell/module/azurerm.rediscache/import-azurermrediscache?view=azurermps-6.13.0&viewFallbackFrom=azurermps-6.4.0).

5. Actualice la aplicación para que use la nueva instancia de caché.

### <a name="write-to-two-redis-caches-simultaneously-during-migration-period"></a>Escritura en dos cachés en Redis simultáneamente durante el período de migración

En lugar de mover los datos directamente entre cachés, puede usar su aplicación para escribir datos en una caché existente y en una nueva que esté configurando. Al principio, la aplicación seguirá leyendo los datos de la caché existente. Cuando la nueva caché tenga los datos necesarios, la aplicación pasará a esa caché y se retirará la anterior. Supongamos, por ejemplo, que usa Redis como almacén de sesión y las sesiones de la aplicación son válidas durante siete días. Después de escribir en las dos cachés durante una semana, estará seguro de que la nueva caché contiene toda la información de sesión que no ha expirado. Podrá confiar en ella con total seguridad a partir de ese momento sin preocuparse por la pérdida de datos.

Los pasos generales para implementar esta opción son los siguientes:

1. Cree una nueva instancia de Azure Cache for Redis en el nivel Premium con un tamaño igual (o mayor) que la caché existente.

2. Modifique el código de la aplicación para que escriba tanto en la instancia nueva como en la original.

3. Siga leyendo los datos de la instancia original hasta que la instancia nueva se haya rellenado con suficientes datos.

4. Actualice el código de la aplicación para que lea y escriba únicamente desde la nueva instancia.

5. Elimine la instancia original.

### <a name="migrate-programmatically"></a>Migración mediante programación

Puede crear un proceso de migración personalizado al leer datos de una caché existente y escribirlos en Azure Cache for Redis mediante programación. Puede usar esta [herramienta de código abierto](https://github.com/deepakverma/redis-copy) para copiar datos de una instancia de Azure Cache for Redis a otra. Esta herramienta resulta útil para mover datos entre instancias de caché en diferentes regiones de Azure Cache. También está disponible la [versión compilada](https://github.com/deepakverma/redis-copy/releases/download/alpha/Release.zip). También puede usar el código fuente como una guía útil para escribir su propia herramienta de migración.

> [!NOTE]
> Oficialmente, Microsoft no admite esta herramienta. 
>

Los pasos generales para implementar esta opción son los siguientes:

1. Cree una máquina virtual en la región en la que se encuentra la caché existente. Si el conjunto de datos es de gran tamaño, elija una máquina virtual relativamente eficaz para reducir el tiempo de copia.

2. Cree una nueva instancia de Azure Cache for Redis.

3. Vacíe los datos de la nueva caché para asegurarse de que está vacía. Eso es necesario porque la herramienta de copia no sobrescribe ninguna de las claves existentes en la caché de destino.

    > [!IMPORTANT]
    > No vacíe la memoria caché de origen.
    >

4. Use una aplicación como la herramienta de código abierto anterior para automatizar la copia de los datos desde la caché de origen a la de destino. No olvide que el proceso de copia puede tardar varios minutos en completarse, según el tamaño del conjunto de datos.

## <a name="next-steps"></a>Pasos siguientes
Más información sobre las características de Azure Cache for Redis.

* [Niveles de servicio de Azure Cache for Redis](cache-overview.md#service-tiers)
* [Importar datos](cache-how-to-import-export-data.md#import)
