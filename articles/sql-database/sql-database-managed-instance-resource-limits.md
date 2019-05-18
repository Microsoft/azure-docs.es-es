---
title: 'Límites de recursos de Azure SQL Database: instancia administrada | Microsoft Docs'
description: En este artículo se proporciona información general de los límites de recursos de Azure SQL Database para las instancias administradas.
services: sql-database
ms.service: sql-database
ms.subservice: managed-instance
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: bonova
ms.author: bonova
ms.reviewer: carlrab, jovanpop, sachinp
manager: craigg
ms.date: 02/27/2019
ms.openlocfilehash: 9726c589472c9ff6d456c922f88d12071e5ebcaf
ms.sourcegitcommit: 179918af242d52664d3274370c6fdaec6c783eb6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2019
ms.locfileid: "65560580"
---
# <a name="overview-azure-sql-database-managed-instance-resource-limits"></a>Introducción a los límites de recursos de Instancia administrada de Azure SQL Database

En este artículo se proporciona información general acerca de los límites de recursos de Instancia administrada de Azure SQL Database y se proporciona información acerca de cómo crear una solicitud para aumentar los límites predeterminados de suscripciones regionales.

> [!NOTE]
> Para conocer otras limitaciones de Instancia administrada, consulte [modelo de compra basado en núcleo virtual](sql-database-managed-instance.md#vcore-based-purchasing-model) y [Niveles de servicio de Instancia administrada](sql-database-managed-instance.md#managed-instance-service-tiers). Para conocer las diferencias en las características e instrucciones T-SQL admitidas, consulte las instrucciones [Diferencias entre las características](sql-database-features.md) y [Compatibilidad con instrucciones T-SQL](sql-database-managed-instance-transact-sql-information.md).

## <a name="instance-level-resource-limits"></a>Límites de recursos a nivel de instancia

Instancia administrada tiene características y límites de recursos que dependen de la infraestructura y la arquitectura subyacentes. Los límites dependen de la generación de hardware y del nivel de servicio.

### <a name="hardware-generation-characteristics"></a>Características de la generación de hardware

Instancia administrada de Azure SQL Database puede implementarse en dos generaciones de hardware (Gen4 y Gen5). Las generaciones de hardware tienen diferentes características que se describen en la tabla siguiente:

|   | **Gen4** | **Gen5** |
| --- | --- | --- |
| Hardware | Procesadores Intel E5-2673 v3 (Haswell) de 2,4 GHz; núcleo virtual SSD conectado equivalente a 1 PP (núcleo físico) | Procesadores Intel E5-2673 v4 (Broadwell) de 2,3 GHz; SSD NVMe rápido, núcleo virtual equivalente a 1 LP (Hyper-Threading) |
| Núcleos virtuales | 8, 16, 24 núcleos virtuales | 8, 16, 24, 32, 40, 64, 80 núcleos virtuales |
| Memoria | 7 GB por núcleo virtual | 5,1 GB por núcleo virtual |
| Memoria de máximo de OLTP en memoria | 3 GB por núcleo virtual | 2,6 GB por núcleo virtual |
| Almacenamiento de instancia máximo (uso General) |  8 TB | 8 TB |
| Almacenamiento de instancia máximo (crítico para la empresa) | 1 TB | 1 TB, 2 TB o 4 TB, en función del número de núcleos |

### <a name="service-tier-characteristics"></a>Características del nivel de servicios

Instancia administrada tiene dos niveles de servicio: Uso general y Crítico para la empresa. Estos niveles proporcionan funcionalidades diferentes, como se describe en la tabla siguiente:

| **Característica** | **Uso general** | **Crítico para la empresa** |
| --- | --- | --- |
| Número de núcleos virtuales\* | Gen4: 8, 16, 24<br/>Gen5: 8, 16, 24, 32, 40, 64, 80 | Gen4: 8, 16, 24, 32 <br/> Gen5: 8, 16, 24, 32, 40, 64, 80 |
| Memoria | Gen4: 56 GB - 168 GB (7GB/núcleo virtual)<br/>Gen5: 40,8 GB - 408 GB (5.1 GB/núcleo virtual) | Gen4: 56 GB - 168 GB (7GB/núcleo virtual)<br/>Gen5: 40,8 GB - 408 GB (5.1 GB/núcleo virtual) |
| Tamaño máximo de almacenamiento de instancia | 8 TB | Gen4: 1 TB <br/> Gen5: <br/>- 1 TB para 8 y 16 núcleos virtuales<br/>- 2 TB para 24 núcleos virtuales<br/>- 4 TB para 32, 40, 64 y 80 núcleos virtuales |
| Almacenamiento máximo por base de datos | Determinado por el tamaño de almacenamiento máximo por instancia | Determinado por el tamaño de almacenamiento máximo por instancia |
| Número máximo de bases de datos por instancia | 100 | 100 |
| Número máximo de archivos de base de datos por instancia | Hasta 280 | Hasta 32 767 archivos por base de datos |
| Datos/IOPS de registro (aproximado) | 500 - 7500 por archivo<br/>\*[Depende del tamaño del archivo](https://docs.microsoft.com/azure/virtual-machines/windows/premium-storage-performance#premium-storage-disk-sizes)| 11 K - 110 K (1375/vCore) |
| Rendimiento de registro | 22 MB/s por instancia | 4 MB/s por núcleo virtual<br/>Max 48 MB/s por instancia|
| Rendimiento de datos (aproximado) | 100 - 250 MB/s por archivo<br/>\*[Depende del tamaño del archivo](https://docs.microsoft.com/azure/virtual-machines/windows/premium-storage-performance#premium-storage-disk-sizes) | |
| Latencia de E/S (aproximada) | 5-10 ms | 1-2 ms |
| Tamaño máximo de tempDB | 192 GB - 1920 GB (24 GB por núcleo virtual) | No hay restricciones: limitadas por el tamaño máximo de almacenamiento de la instancia |

**Notas**:

- Tanto los datos como el tamaño de archivo de registro en las bases de datos del usuario y las del sistema se incluyen en el tamaño de almacenamiento de la instancia que se compara con el límite de tamaño de almacenamiento máximo. Utilice la vista del sistema <a href="https://docs.microsoft.com/sql/relational-databases/system-catalog-views/sys-master-files-transact-sql">sys.master_files</a> para determinar el espacio total utilizado por las bases de datos. Los registros de errores no se mantienen y no se incluyen en el tamaño. Las copias de seguridad no se incluyen en el tamaño del almacenamiento.
- El rendimiento e IOPS también dependen del tamaño de página que no está limitado explícitamente por instancia administrada.

## <a name="supported-regions"></a>Regiones admitidas

Las instancias administradas solo se pueden crear en las [regiones admitidas](https://azure.microsoft.com/global-infrastructure/services/?products=sql-database&regions=all). Si desea crear una instancia administrada en una región que no se admita actualmente, puede [enviar la solicitud de soporte técnico a través de Azure Portal](#obtaining-a-larger-quota-for-sql-managed-instance).

## <a name="supported-subscription-types"></a>Tipos de suscripciones admitidos

Actualmente, Instancia administrada admite la implementación solo en los siguientes tipos de suscripciones:

- [Contrato Enterprise (EA)](https://azure.microsoft.com/pricing/enterprise-agreement/)
- [Pago por uso](https://azure.microsoft.com/offers/ms-azr-0003p/)
- [Proveedor de nube (CSP)](https://docs.microsoft.com/partner-center/csp-documents-and-learning-resources)
- [Desarrollo/pruebas - Enterprise](https://azure.microsoft.com/offers/ms-azr-0148p/)
- [Desarrollo/pruebas - Pago por uso](https://azure.microsoft.com/offers/ms-azr-0023p/)

> [!NOTE]
> Esta limitación es temporal. En el futuro se habilitarán nuevos tipos de suscripción.

## <a name="regional-resource-limitations"></a>Limitaciones de recursos regionales

Los tipos de suscripción compatibles pueden contener un número limitado de recursos por región. Instancia administrada tiene dos límites predeterminados por región de Azure en función del tipo de suscripción:

- **Límite de subred**: el número máximo de subredes en que se implementan instancias administradas en una sola región.
- **Límite del número de instancias**: el número máximo de instancias que se pueden implementar en una sola región.

> [!Note]
> Estos límites son la configuración predeterminada y las limitaciones técnicas no. Los límites pueden ser mayor y a petición mediante la creación de especial [solicitud de soporte técnico en Azure portal](#obtaining-a-larger-quota-for-sql-managed-instance) si necesita más instancias administradas en la región actual. Como alternativa, puede crear nuevas instancias administradas en otra región de Azure sin necesidad de enviar solicitudes de soporte técnico.

En la tabla siguiente se muestran los límites regionales predeterminados para las suscripciones admitidas:

|Tipo de suscripción| Número máximo de subredes de Instancia administrada | Número máximo de instancias |Número máximo de subredes de instancia administradas de GP*|Número máximo de subredes de instancia administradas de BC*|
| :---| :--- | :--- |:--- |:--- |
|Pago por uso|1*|4*|4*|1*|
|CSP |1*|4*|4*|1*|
|Desarrollo/pruebas - Pago por uso|1*|4*|4*|1*|
|Desarrollo/pruebas - Enterprise|1*|4*|4*|1*|
|EA|3**|12**|12**|3**|

\* Puede implementar 1 instancia de BC o 4 de GP en una subred, con el fin de que el número total de “unidades de instancia” de la subred nunca es superior a 4.

** Se aplica el número máximo de instancias en un nivel de servicio si no hay instancias en otro nivel de servicio. En caso de que planee mezclar instancias de GP y BC dentro de la misma subred, utilice la siguiente sección como referencia para las combinaciones permitidas. Como regla sencilla, el número total de subredes no puede superar as 3 y el número total de unidades de instancia no puede superar las 12.



> [!IMPORTANT]
> Al planear las implementaciones, considere la posibilidad de que una instancia de Crítico para la empresa (BC) (debido a la redundancia agregada) por lo general consume 4 veces más capacidad que una instancia De uso general (GP). Por tanto, para los cálculos, 1 instancia de GP = 1 unidad de instancia y 1 instancia de BC = 4 unidades de instancia. Para simplificar el análisis de consumo frente a los límites predeterminados, resuma las unidades de instancia de todas las subredes de la región en la que se implementan instancias administradas y comparar los resultados con los límites de la unidad de instancia del tipo de suscripción.

## <a name="strategies-for-deploying-mixed-general-purpose-and-business-critical-instances"></a>Estrategias para implementar instancias De uso general y de Crítico para la empresa mixtas

Las suscripciones [Contrato Enterprise (EA)](https://azure.microsoft.com/pricing/enterprise-agreement/) pueden tener combinaciones de las instancias de GP y BC. Sin embargo, existen algunas limitaciones con respecto a la colocación de las instancias en las subredes.

> [!Note]
> Los tipos de suscripción [Pago por uso](https://azure.microsoft.com/offers/ms-azr-0003p/) y [Proveedor de nube (CSP)](https://docs.microsoft.com/partner-center/csp-documents-and-learning-resources) pueden tener una instancia de Crítico para la empresa o hasta 4 De uso general.

Los siguientes ejemplos cubren los casos de implementación con subredes no vacías y niveles de servicio GP y BC mixtos.

|Cantidad de subredes|Subred 1|Subred 2|Subred 3|
|:---|:---|:---|:---|
|1|1 BC y un máximo de 8 GP<br>2 BC y un máximo de 4 GP|N/D| N/D|
|2|0 BC, un máximo de 4 GP|1 BC, un máximo de 4 GP<br>2 BC, 0 GP|N/D|
|2|1 BC, 0 GP|0 BC, un máximo de 8 GP<br>1 BC, un máximo de 4 GP|N/D|
|2|2 BC, 0 GP|0 BC, un máximo de 4 GP|N/D|
|3|1 BC, 0 GP|1 BC, 0 GP|0 BC, un máximo de 4 GP|
|3|1 BC, 0 GP|0 BC, un máximo de 4 GP|0 BC, un máximo de 4 GP|

## <a name="obtaining-a-larger-quota-for-sql-managed-instance"></a>Obtención de una cuota mayor Instancia administrada de SQL

Si necesita más instancias administradas en sus regiones actuales, puede enviar la solicitud de soporte técnico para ampliar la cuota mediante Azure Portal.
Para iniciar el proceso de obtención de una cuota mayor:

1. Abra **Ayuda y soporte técnico** y haga clic en **Nueva solicitud de soporte técnico**.

   ![Ayuda y soporte técnico](media/sql-database-managed-instance-resource-limits/help-and-support.png)
2. En la pestaña Conceptos básicos de la nueva solicitud de soporte técnico:
   - En **Tipo de problema**, seleccione **Límites de servicio y suscripción (cuotas)**.
   - En **Suscripción**, seleccione la suscripción.
   - En **Tipo de cuota**, seleccione **Instancia administrada de SQL Database**.
   - En **Plan de soporte técnico**, seleccione un plan de soporte técnico.

     ![Cuota de tipo de problema](media/sql-database-managed-instance-resource-limits/issue-type-quota.png)

3. Haga clic en **Next**.
4. En la pestaña Problema de la nueva solicitud de soporte técnico:
   - En **Gravedad**, seleccione el nivel de gravedad del problema.
   - En **Detalles**, especifique información adicional acerca del problema, lo que incluye los mensajes de error.
   - En **Carga de archivos**, adjunte un archivo con más información (hasta 4 MB).

     ![Detalles del problema](media/sql-database-managed-instance-resource-limits/problem-details.png)

     > [!IMPORTANT]
     > Una solicitud válida debe incluir:
     > - Región en la que hay que aumentar el límite de la suscripción
     > - Número requerido de instancias, por nivel de servicio en las subredes existentes después del aumento de la cuota (si cualquiera de las subredes existentes debe expandirse)
     > - Número requerido de nuevas subredes y número total de instancias por nivel de servicio dentro de las nuevas subredes (si tiene que implementar instancias administradas en nuevas subredes).

5. Haga clic en **Next**.
6. En la pestaña Información de contacto de la nueva solicitud de soporte técnico, especifique el método de contacto preferido (teléfono o correo electrónico) y los detalles de contacto.
7. Haga clic en **Create**(Crear).

## <a name="next-steps"></a>Pasos siguientes

- Para más información acerca de Instancia administrada, consulte [¿Qué es Instancia administrada de SQL Database (versión preliminar)?](sql-database-managed-instance.md).
- Para obtener información de precios, vea [Precios de Instancia administrada de SQL Database](https://azure.microsoft.com/pricing/details/sql-database/managed/).
- Para obtener información sobre cómo crear su primera instancia administrada, consulte la [guía de inicio rápido](sql-database-managed-instance-get-started.md).
