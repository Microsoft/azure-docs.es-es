---
title: Matriz de compatibilidad de Azure Backup para la copia de seguridad de SQL Server en VM de Azure
description: Proporciona un resumen de opciones de compatibilidad y limitaciones para realizar copias de seguridad de SQL Server en VM de Azure con el servicio Azure Backup.
ms.topic: conceptual
ms.date: 03/05/2020
ms.custom: references_regions
ms.openlocfilehash: 41511abaa071bd0f64ee699c52486b71ec036a68
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87926457"
---
# <a name="support-matrix-for-sql-server-backup-in-azure-vms"></a>Matriz de compatibilidad para la copia de seguridad de SQL Server en VM de Azure

Puede usar Azure Backup para realizar copias de seguridad de bases de datos de SQL Server en VM de Azure hospedadas en la plataforma de nube Microsoft Azure. En este artículo se resumen las configuraciones y limitaciones de compatibilidad generales para los escenarios e implementaciones de copia de seguridad de SQL Server en VM de Azure.

## <a name="scenario-support"></a>Compatibilidad con los escenarios

**Soporte técnico** | **Detalles**
--- | ---
**Implementaciones admitidas** | Se admiten máquinas virtuales de Azure de SQL Marketplace y que no son de Marketplace (SQL Server instalado manualmente).
**Regiones admitidas** | Sudeste de Australia (ASE), Este de Australia (AE), Centro de Australia (AC) y Centro de Australia 2 (AC) <br> Sur de Brasil (BRS)<br> Centro de Canadá (CNC) y Este de Canadá (CE)<br> Sudeste de Asia (SEA) y Este de Asia (EA) <br> Este de EE. UU. (EUS), Este de EE. UU. 2 (EUS2), Centro-oeste de EE. UU. (WCUS), Oeste de EE. UU. (WUS); Oeste de EE. UU. 2 (WUS 2) Centro-norte de EE. UU. (NCUS) Centro de EE. UU. (CUS) Centro-sur de EE. UU. (SCUS) <br> Centro de la India (INC), India del Sur (INS), India occidental <br> Japón Oriental (JPE) y Japón Occidental (JPW) <br> Centro de Corea del Sur (KRC), Sur de Corea del Sur (KRS) <br> Norte de Europa (NE) y Oeste de Europa <br> Sur de Reino Unido (UKS) y Oeste de Reino Unido (UKW) <br> US Gov Arizona, US Gov Virginia, US Gov Texas, US DoD (centro), US DoD (este) <br> Norte de Alemania, Centro-oeste de Alemania <br> Norte de Suiza, Oeste de Suiza <br> Centro de Francia <br> Este de China, Norte de China, Norte de China 2 y Este de China 2
**Sistemas operativos compatibles** | Windows Server 2019, Windows Server 2016, Windows Server 2012, Windows Server 2008 R2 SP1 <br/><br/> Linux no se admite actualmente.
**Versiones admitidas de SQL Server** | SQL Server 2019, SQL Server 2017 tal como se detalla en la [página de búsqueda del ciclo de vida del producto](https://support.microsoft.com/lifecycle/search?alpha=SQL%20server%202017), SQL Server 2016 y los SP tal como se detalla en la [página de búsqueda del ciclo de vida del producto](https://support.microsoft.com/lifecycle/search?alpha=SQL%20server%202016%20service%20pack), SQL Server 2014, SQL Server 2012, SQL Server 2008 R2, SQL Server 2008 <br/><br/> Enterprise, Standard, Web, Developer, Express.
**Versiones de .NET compatibles** | .NET Framework 4.5.2 o posterior instalado en la máquina virtual

## <a name="feature-considerations-and-limitations"></a>Consideraciones y limitaciones de las características

|Configuración  |Límite máximo |
|---------|---------|
|Número de bases de datos que se pueden proteger en un servidor (y en un almacén)    |   2000      |
|Tamaño de la base de datos compatible (más allá de esto, pueden aparecer problemas de rendimiento)   |   2 TB      |
|Número de archivos admitidos en una base de datos    |   1000      |

>[!NOTE]
> [Descargue el planeador de recursos detallado](https://download.microsoft.com/download/A/B/5/AB5D86F0-DCB7-4DC3-9872-6155C96DE500/SQL%20Server%20in%20Azure%20VM%20Backup%20Scale%20Calculator.xlsx) para calcular el número aproximado de bases de datos protegidas que se recomiendan por servidor en función de los recursos de la máquina virtual, el ancho de banda y la directiva de copia de seguridad.

* La copia de seguridad de SQL Server se puede configurar en Azure Portal o **PowerShell**. No se admite la CLI.
* La solución es compatible con ambos tipos de [implementaciones](../azure-resource-manager/management/deployment-models.md): las máquinas virtuales de Azure Resource Manager y las máquinas virtuales clásicas.
* Se admiten todos los tipos de copia de seguridad (completas, diferenciales y de registro) y los modelos de recuperación (simple, completo o registros de operaciones masivas).
* Los tipos de copia de seguridad Completa y Solo copia completa se admiten en las bases de datos **de solo lectura**.
* La compresión nativa de SQL es compatible si el usuario la habilita explícitamente en la directiva de copia de seguridad. Azure Backup invalida los valores predeterminados de nivel de instancia con la cláusula COMPRESSION/NO_COMPRESSION según el valor de este control establecido por el usuario.
* Se admite la copia de seguridad de base de datos habilitada para TDE. Para restaurar una base de datos cifrada TDE en otra de SQL Server, primero debe [restaurar el certificado en el servidor de destino](https://docs.microsoft.com/sql/relational-databases/security/encryption/move-a-tde-protected-database-to-another-sql-server). Está disponible la compresión de copia de seguridad para las bases de datos habilitadas para TDE para SQL Server 2016 y las versiones más recientes, pero con un tamaño de transferencia inferior, tal y como se explica [aquí](https://techcommunity.microsoft.com/t5/sql-server/backup-compression-for-tde-enabled-databases-important-fixes-in/ba-p/385593).
* No se admiten operaciones de copia de seguridad y restauración de bases de datos reflejadas ni de instantáneas de bases de datos.
* La **instancia del clúster de conmutación por error (FCI)** de SQL Server no se admite.
* Si se usa más de una solución para realizar copias de seguridad de una instancia de SQL Server independiente o de un grupo de disponibilidad Always On de SQL, se pueden producir errores en la copia de seguridad. Se aconseja no hacerlo. La realización de una copia de seguridad de dos nodos de un grupo de disponibilidad individualmente con las mismas soluciones o soluciones diferentes, también puede dar lugar a errores en la copia de seguridad.
* Si los grupos de disponibilidad están configurados, las copias de seguridad se realizan de los distintos nodos en función de una serie de factores. A continuación, se resume el comportamiento del proceso de copia de seguridad en un grupo de disponibilidad.

### <a name="back-up-behavior-with-always-on-availability-groups"></a>Comportamiento del proceso de copia de seguridad con los grupos de disponibilidad Always On

Se recomienda que la copia de seguridad se configure en un único nodo de un grupo de disponibilidad. Configure siempre la copia de seguridad en la misma región que el nodo principal. En otras palabras, siempre es necesario que el nodo principal esté presente en la región en la que se configura la copia de seguridad. Si todos los nodos del grupo de disponibilidad están en la misma región donde se configura la copia de seguridad, no hay ningún problema.

#### <a name="for-cross-region-ag"></a>Para los grupos de disponibilidad con varias regiones

* Independientemente de la preferencia de copia de seguridad, las copias de seguridad solo se ejecutarán desde los nodos que estén en la misma región donde se configure la copia de seguridad. Esto se debe a que no se admiten las copias de seguridad entre regiones. Si tiene solo dos nodos y el secundario está en la otra región, las copias de seguridad se seguirán ejecutando desde el nodo principal (a menos que su preferencia de copia de seguridad sea "solo secundaria").
* Si se produce una conmutación por error de un nodo en una región diferente de aquella en la que se configura la copia de seguridad, las copias de seguridad no se realizarán correctamente en los nodos de la región conmutada por error.

En función de las preferencias relativas a la copia de seguridad y de los tipos de copia de seguridad (completa, diferencial, de registros, y solo copia completa), se toman las copias de seguridad de un nodo concreto (principal o secundario).

#### <a name="backup-preference-primary"></a>Preferencia de copia de seguridad: Principal

**Tipo de copia de seguridad** | **Node**
--- | ---
Completo | Principal
Diferencial | Principal
Log |  Principal
Solo copia completa |  Principal

#### <a name="backup-preference-secondary-only"></a>Preferencia de copia de seguridad: solo secundaria

**Tipo de copia de seguridad** | **Node**
--- | ---
Completo | Principal
Diferencial | Principal
Log |  Secundario
Solo copia completa |  Secundario

#### <a name="backup-preference-secondary"></a>Preferencia de copia de seguridad: Secundario

**Tipo de copia de seguridad** | **Node**
--- | ---
Completo | Principal
Diferencial | Principal
Log |  Secundario
Solo copia completa |  Secundario

#### <a name="no-backup-preference"></a>Sin preferencia de copia de seguridad

**Tipo de copia de seguridad** | **Node**
--- | ---
Completo | Principal
Diferencial | Principal
Log |  Secundario
Solo copia completa |  Secundario

## <a name="next-steps"></a>Pasos siguientes

Aprenda a [hacer una copia de seguridad de una base de datos de SQL Server](backup-azure-sql-database.md) que se ejecuta en una máquina virtual de Azure.
