---
title: Información general del modelo de compra de núcleo virtual
titleSuffix: Azure SQL Database & Azure SQL Managed Instance
description: El modelo de compra de núcleo virtual permite escalar los recursos de proceso y de almacenamiento de manera independiente, igualar el rendimiento local y optimizar el precio para Azure SQL Database e Instancia administrada de Azure SQL.
services: sql-database
ms.service: sql-db-mi
ms.subservice: features
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: sashan, moslake, carlrab
ms.date: 08/14/2020
ms.openlocfilehash: 7131ddac840d2854969147da2eeb82a890ce3410
ms.sourcegitcommit: 02ca0f340a44b7e18acca1351c8e81f3cca4a370
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88586824"
---
# <a name="vcore-model-overview---azure-sql-database-and-azure-sql-managed-instance"></a>Información general del modelo de núcleo virtual: Azure SQL Database y Azure SQL Managed Instance 
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

El modelo de compra de núcleo virtual usado por Azure SQL Database e Instancia administrada de Azure SQL proporciona varias ventajas:

- Mayores límites de proceso, memoria, E/S y almacenamiento.
- Control sobre la generación de hardware para satisfacer mejor los requisitos de proceso y memoria de la carga de trabajo.
- Descuentos en los precios de [Ventaja híbrida de Azure (AHB)](../azure-hybrid-benefit.md) e [Instancia reservada (RI)](reserved-capacity-overview.md).
- Mayor transparencia en los detalles de hardware que potencian el proceso, lo que facilita la planeación de las migraciones desde implementaciones locales.

## <a name="service-tiers"></a>Niveles de servicio

Entre las opciones de nivel de servicio del modelo núcleo virtual se incluyen Uso general, Crítico para la empresa e Hiperescala. El nivel de servicio define generalmente la arquitectura de almacenamiento, los límites de espacio y de E/S y las opciones de continuidad empresarial relacionadas con la disponibilidad y la recuperación ante desastres.

|-|**Uso general**|**Crítico para la empresa**|**Hiperescala**|
|---|---|---|---|
|Más adecuado para|La mayoría de las cargas de trabajo empresariales. Ofrece opciones de proceso y almacenamiento equilibradas y escalables pensando en el presupuesto. |Ofrece a las aplicaciones empresariales la mayor resistencia a los errores mediante el uso de varias réplicas aisladas y proporciona el mayor rendimiento de E/S por réplica de base de datos.|La mayoría de las cargas de trabajo de una empresa que tengan requisitos altamente escalables de almacenamiento y escalado de lectura.  Ofrece mayor resistencia a los errores al permitir la configuración de más de una réplica de base de datos aislada. |
|Storage|Usa el almacenamiento remoto.<br/>**Proceso aprovisionado de SQL Database**:<br/>5 GB – 4 TB<br/>**Proceso sin servidor**:<br/>5 GB - 3 TB<br/>**Instancia administrada de SQL**: 32 GB - 8 TB |Usa almacenamiento local de SSD.<br/>**Proceso aprovisionado de SQL Database**:<br/>5 GB – 4 TB<br/>**Instancia administrada de SQL**:<br/>32 GB - 4 TB |Crecimiento automático flexible de almacenamiento según sea necesario. Admite hasta 100 TB de almacenamiento. Utiliza almacenamiento SSD local para la caché del grupo de búferes local y almacenamiento de datos local. Utiliza almacenamiento remoto de Azure como almacén de datos final a largo plazo. |
|IOPS y rendimiento (aproximado)|**SQL Database**: vea los límites de recursos para [bases de datos únicas](resource-limits-vcore-single-databases.md) y [grupos elásticos](resource-limits-vcore-elastic-pools.md).<br/>**Instancia administrada de SQL**: consulte [Introducción a los límites de recursos de Instancia administrada de Azure SQL](../managed-instance/resource-limits.md#service-tier-characteristics).|Vea los límites de recursos para [bases de datos únicas](resource-limits-vcore-single-databases.md) y [grupos elásticos](resource-limits-vcore-elastic-pools.md).|Hiperescala es una arquitectura de varios niveles con almacenamiento en caché en varios niveles. IOPS y el rendimiento efectivos dependen de la carga de trabajo.|
|Disponibilidad|1 réplica, sin réplicas de escalado de lectura|3 réplicas, 1 [réplica de escalado de lectura](read-scale-out.md),<br/>Alta disponibilidad (HA) con redundancia de zona|1 réplica de lectura y escritura, además de 0 a 4 [réplicas de escalado de lectura](read-scale-out.md)|
|Copias de seguridad|[Almacenamiento con redundancia geográfica con acceso de lectura (RA-GRS)](../../storage/common/geo-redundant-design.md), de 7 a 35 días (7 días de forma predeterminada)|[RA-GRS](../..//storage/common/geo-redundant-design.md), de 7 a 35 días (7 días de forma predeterminada)|Copias de seguridad basadas en instantáneas en el almacenamiento remoto de Azure. Los procesos de restauración usan estas instantáneas para conseguir una recuperación rápida. Las copias de seguridad son instantáneas y no afectan al rendimiento de E/S del proceso. Las restauraciones son rápidas y no son operaciones relacionadas con el tamaño de los datos (tardan minutos en lugar de horas o días).|
|En memoria|No compatible|Compatible|No compatible|
|||


### <a name="choosing-a-service-tier"></a>Selección de un nivel de servicio

Para obtener información sobre cómo seleccionar un nivel de servicio para una carga de trabajo determinada, consulte los siguientes artículos:

- [Cuándo elegir el nivel de servicio De uso general](service-tier-general-purpose.md#when-to-choose-this-service-tier)
- [Cuándo elegir el nivel de servicio Crítico para la empresa](service-tier-business-critical.md#when-to-choose-this-service-tier)
- [Cuándo elegir el nivel de servicio Hiperescala](service-tier-hyperscale.md#who-should-consider-the-hyperscale-service-tier)


## <a name="compute-tiers"></a>Niveles de proceso

Entre las opciones de nivel de proceso del modelo de núcleo virtual se incluyen los niveles de proceso aprovisionados y sin servidor.


### <a name="provisioned-compute"></a>Proceso aprovisionado

El nivel de proceso aprovisionado proporciona una cantidad específica de recursos de proceso que se aprovisionan continuamente con independencia de la actividad de carga de trabajo y factura la cantidad de proceso aprovisionado a un precio fijo por hora.


### <a name="serverless-compute"></a>Proceso sin servidor

El [nivel de proceso sin servidor](serverless-tier-overview.md) escala automáticamente los recursos de proceso en función de la actividad de la carga de trabajo y se factura según la cantidad de proceso usado por segundo.



## <a name="hardware-generations"></a>Generaciones de hardware

Entre las opciones de generación de hardware del modelo de núcleo virtual se incluyen Gen 4/5, la serie M y la serie Fsv2. En general, la generación de hardware define los límites de proceso y de memoria, así como otras características que afectan al rendimiento de la carga de trabajo.

### <a name="gen4gen5"></a>Gen4/Gen5

- El hardware Gen4/Gen5 proporciona recursos equilibrados de proceso y memoria, y es adecuado para la mayoría de las cargas de trabajo de base de datos que no tienen una memoria mayor, un núcleo virtual superior o unos requisitos de núcleo virtual único más rápidos como los proporcionados por la serie Fsv2 o la serie M.

Para ver las regiones en las que Gen4/Gen5 está disponible, vea la [disponibilidad de Gen4/Gen5](#gen4gen5-1).

### <a name="fsv2-series"></a>Serie Fsv2

- La serie Fsv2 es una opción de hardware optimizado para proceso que ofrece una latencia de CPU baja y una velocidad de reloj elevada para las cargas de trabajo más exigentes de CPU.
- En función de la carga de trabajo, la serie Fsv2 puede ofrecer más rendimiento de CPU por núcleo virtual que Gen5, y el tamaño de 72 núcleos virtuales puede proporcionar más rendimiento de CPU con menos costos que 80 núcleos virtuales en Gen5. 
- Fsv2 proporciona menos memoria y tempdb por núcleo virtual que otro hardware, por lo que se pueden tomar en consideración las series Gen5 y M para las cargas de trabajo sensibles a esos límites.  

La serie Fsv2 solo se admite en el nivel De uso general. Para las regiones en las que está disponible la serie Fsv2, consulte la [disponibilidad de la serie Fsv2](#fsv2-series-1).


### <a name="m-series"></a>Serie M

- La serie M es una opción de hardware optimizado para memoria para las cargas de trabajo que exigen más memoria y mayores límites de proceso que los proporcionados por Gen5.
- La serie M cuenta con 29 GB por cada núcleo virtual y hasta 128 núcleos virtuales, lo que multiplica por ocho el límite de memoria con respecto a Gen5 hasta alcanzar prácticamente los 4 TB.

La serie M solo se admite en el nivel Crítico para la empresa y no es compatible con la redundancia de zona.  La suscripción debe ser un tipo de oferta de pago, como Pago por uso o Contrato Enterprise (EA). Para ver las regiones en las que la serie M está disponible, consulte la [disponibilidad de la serie M](#m-series-1).

<!--
To enable M-series hardware for a subscription and region, a support request must be opened. The subscription must be a paid offer type including Pay-As-You-Go or Enterprise Agreement (EA).  If the support request is approved, then the selection and provisioning experience of M-series follows the same pattern as for other hardware generations. For regions where M-series is available, see [M-series availability](#m-series).
-->

### <a name="compute-and-memory-specifications"></a>Especificaciones de memoria y proceso


|Generación de hardware  |Proceso  |Memoria  |
|:---------|:---------|:---------|
|Gen4     |- Procesadores Intel® E5-2673 v3 (Haswell) de 2,4 GHz<br>- Aprovisionamiento de hasta 24 núcleos virtuales (1 núcleo virtual = 1 núcleo físico)  |- 7 GB por núcleo virtual<br>- Aprovisionamiento de hasta 168 GB|
|Gen5     |**Proceso aprovisionado**<br>- Procesadores Intel® E5-2673 v4 (Broadwell) de 2,3 GHz, Intel® SP-8160 (Skylake)\* e Intel® 8272CL (Cascade Lake) de 2,5 GHz\*<br>- Aprovisionamiento de hasta 80 núcleos virtuales (1 núcleo virtual = 1 hiperproceso)<br><br>**Proceso sin servidor**<br>- Procesadores Intel® E5-2673 v4 (Broadwell) de 2,3 GHz e Intel® SP-8160 (Skylake)*<br>- Escalado vertical automático de hasta 40 núcleos virtuales (1 núcleo virtual = 1 hiperproceso)|**Proceso aprovisionado**<br>- 5,1 GB por núcleo virtual<br>- Aprovisionamiento de hasta 408 GB<br><br>**Proceso sin servidor**<br>- Escalado vertical automático de hasta 24 GB por núcleo virtual<br>- Escalado vertical automático de hasta 120 GB máx.|
|Serie Fsv2     |- Procesadores Intel® 8168 (Skylake)<br>- Presentación de una velocidad de reloj turbo sostenida de todos los núcleos de hasta 3,4 GHz y una velocidad de reloj turbo de un solo núcleo máxima de 3,7 GHz.<br>- Aprovisionamiento de hasta 72 núcleos virtuales (1 núcleo virtual = 1 hiperproceso)|1,9 GB por núcleo virtual<br>- Aprovisionamiento de hasta 136 GB|
|Serie M     |- Procesadores Intel® E7-8890 v3 de 2,5 GHz e Intel® 8280M de 2,7 GHz (Cascade Lake)<br>- Aprovisionamiento de hasta 128 núcleos virtuales (1 núcleo virtual = 1 hiperproceso)|29 GB por núcleo virtual<br>- Aprovisionamiento de hasta 3,7 TB|

\* En la vista de administración dinámica [sys.dm_user_db_resource_governance](https://docs.microsoft.com/sql/relational-databases/system-dynamic-management-views/sys-dm-user-db-resource-governor-azure-sql-database), la generación de hardware para bases de datos que usan procesadores Intel® SP-8160 (Skylake) aparece como Gen6, mientras que la generación de hardware para bases de datos que usan procesadores Intel® 8272CL (Cascade Lake) aparece como Gen7. Los límites de recursos en todas las bases de datos Gen5 son los mismos, independientemente del tipo de procesador (Broadwell, Skylake o Cascade Lake).

Para obtener más información sobre los límites de recursos, vea [Límites de recursos para bases de datos únicas (núcleo virtual)](resource-limits-vcore-single-databases.md) o [Límites de recursos para grupos elásticos (núcleo virtual)](resource-limits-vcore-elastic-pools.md).

### <a name="selecting-a-hardware-generation"></a>Selección de una generación de hardware

En Azure Portal, puede seleccionar la generación de hardware para una base de datos o un grupo de SQL Database en el momento de la creación o puede cambiar la generación de hardware de una base de datos o grupo existente.

**Seleccionar una generación de hardware al crear una instancia de SQL Database o un grupo SQL**

Para obtener información detallada, consulte [Creación de una instancia de SQL Database](single-database-create-quickstart.md).

En la pestaña **Básico**, seleccione el vínculo **Configurar base de datos** en la sección **Compute + storage** (Proceso + almacenamiento) y, a continuación, seleccione el vínculo **Cambiar configuración**:

  ![configurar base de datos](./media/service-tiers-vcore/configure-sql-database.png)

Seleccione la generación de hardware deseada:

  ![seleccionar hardware](./media/service-tiers-vcore/select-hardware.png)


**Cambiar la generación de hardware de una instancia de SQL Database o un grupo SQL existente**

En el caso de una base de datos, en la página de información general, seleccione el vínculo **Plan de tarifa**:

  ![cambiar hardware](./media/service-tiers-vcore/change-hardware.png)

En la página de información general, seleccione **Configurar**.

Siga los pasos para cambiar la configuración y seleccione la generación de hardware como se describe en los pasos anteriores.

**Selección de una generación de hardware al crear una instancia administrada de SQL**

Para obtener información detallada, consulte [Creación de una instancia administrada de SQL](../managed-instance/instance-create-quickstart.md).

En la pestaña **Aspectos básicos**, seleccione el vínculo **Configurar base de datos** en la sección **Compute + storage** (Proceso y almacenamiento) y, a continuación, seleccione la generación de hardware deseada:

  ![configurar la instancia administrada de SQL](./media/service-tiers-vcore/configure-managed-instance.png)
  
**Cambio de la generación de hardware de una instancia administrada de SQL existente**

# <a name="the-azure-portal"></a>[Portal de Azure](#tab/azure-portal)

En la página de instancia administrada de SQL, seleccione el vínculo **Plan de tarifa** situado en la sección Configuración.

![cambiar el hardware de instancia administrada de SQL](./media/service-tiers-vcore/change-managed-instance-hardware.png)

En la página Plan de tarifa, podrá cambiar la generación de hardware tal y como se describe en los pasos anteriores.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Use el siguiente script de PowerShell:

```powershell-interactive
Set-AzSqlInstance -Name "managedinstance1" -ResourceGroupName "ResourceGroup01" -ComputeGeneration Gen5
```

Para más información, vea el comando [Set-AzSqlInstance](https://docs.microsoft.com/powershell/module/az.sql/set-azsqlinstance).

# <a name="the-azure-cli"></a>[La CLI de Azure](#tab/azure-cli)

Use el siguiente comando de la CLI:

```azurecli-interactive
az sql mi update -g mygroup -n myinstance --family Gen5
```

Para más información, vea el comando [az sql mi update](https://docs.microsoft.com/cli/azure/sql/mi#az-sql-mi-update).

---

### <a name="hardware-availability"></a>Disponibilidad de hardware

#### <a name="gen4gen5"></a><a name="gen4gen5-1"></a> Gen4/Gen5

El hardware de Gen4 está [en proceso de eliminación gradual](https://azure.microsoft.com/updates/gen-4-hardware-on-azure-sql-database-approaching-end-of-life-in-2020/) y ya no está disponible para las nuevas implementaciones. Todas las nuevas bases de datos deben implementarse en hardware de Gen5.

Gen5 está disponible en la mayoría de las regiones de todo el mundo.

#### <a name="fsv2-series"></a>Serie Fsv2

La serie Fsv2 está disponible en las siguientes regiones: Centro de Australia, Centro de Australia 2, Este de Australia, Sudeste de Australia, Sur de Brasil, Centro de Canadá, Este de Asia, Asia Pacífico, Centro de Francia, Centro de la India, Oeste de la India, Centro de Corea del Sur, Sur de Corea del Sur, Norte de Europa, Norte de Sudáfrica, Sudeste de Asia, Sur de Reino Unido, Oeste de Reino Unido, Oeste de Europa, Oeste de Europa 2.


#### <a name="m-series"></a>Serie M

La serie M está disponible en las siguientes regiones: Este de EE. UU., Norte de Europa, Oeste de Europa, Oeste de EE. UU. 2.
<!--
M-series may also have limited availability in additional regions. You can request a different region than listed here, but fulfillment in a different region may not be possible.

To enable M-series availability in a subscription, access must be requested by [filing a new support request](#create-a-support-request-to-enable-m-series).


##### Create a support request to enable M-series: 

1. Select **Help + support** in the portal.
2. Select **New support request**.

On the **Basics** page, provide the following:

1. For **Issue type**, select **Service and subscription limits (quotas)**.
2. For **Subscription** = select the subscription to enable M-series.
3. For **Quota type**, select **SQL database**.
4. Select **Next** to go to the **Details** page.

On the **Details** page, provide the following:

1. In the **PROBLEM DETAILS** section select the **Provide details** link. 
2. For **SQL Database quota type** select **M-series**.
3. For **Region**, select the region to enable M-series.
    For regions where M-series is available, see [M-series availability](#m-series).

Approved support requests are typically fulfilled within 5 business days.
-->

## <a name="next-steps"></a>Pasos siguientes

Para empezar, consulte: 
- [Creación de una instancia de SQL Database mediante Azure Portal](single-database-create-quickstart.md)
- [Creación de una instancia administrada de SQL mediante Azure Portal](../managed-instance/instance-create-quickstart.md)

Para obtener información detallada acerca de los precios, consulte la [página de precios de Azure SQL Database](https://azure.microsoft.com/pricing/details/sql-database/single/).

Para obtener más información sobre los tamaños específicos de proceso y almacenamiento disponibles en los niveles de servicio De uso general y Crítico para la empresa, consulte:

- [Límites de recursos basados en núcleos virtuales de Azure SQL Database](resource-limits-vcore-single-databases.md).
- [Límites de recursos basados en núcleos virtuales de Azure SQL Database agrupada](resource-limits-vcore-elastic-pools.md).
- [Límites de recursos basados en núcleos virtuales para Instancia administrada de Azure SQL](../managed-instance/resource-limits.md). 

