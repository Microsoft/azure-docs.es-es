---
title: Guía de precios y administración de los costos
description: Se proporcionan procedimientos recomendados para elegir el modelo de fijación de precios adecuado de máquinas virtuales de SQL Server.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
editor: ''
tags: azure-service-management
ms.assetid: ''
ms.service: virtual-machines-sql
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 08/09/2018
ms.author: mathoma
ms.reviewer: jroth
ms.custom: seo-lt-2019
ms.openlocfilehash: b1d07b3ff41c16ff2145311a0b42d8c57a8e7b82
ms.sourcegitcommit: a2a7746c858eec0f7e93b50a1758a6278504977e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/12/2020
ms.locfileid: "88141965"
---
# <a name="pricing-guidance-for-sql-server-on-azure-vms"></a>Orientación de precios de SQL Server en máquinas virtuales de Azure
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

En este artículo se proporcionan orientaciones sobre precios de [SQL Server en máquinas virtuales de Azure](sql-server-on-azure-vm-iaas-what-is-overview.md). Hay varias opciones que afectan al costo, y es importante elegir la imagen correcta que equilibra los costes con los requisitos empresariales.

> [!TIP]
> Si solo necesita averiguar el costo estimado de una combinación específica de edición de SQL Server y tamaño de máquina virtual (VM), vea la página de precios de [Windows](https://azure.microsoft.com/pricing/details/virtual-machines/windows) o [Linux](https://azure.microsoft.com/pricing/details/virtual-machines/linux). Seleccione la plataforma y la edición de SQL Server en la lista **OS/Software** (SO o software).
>
> ![Interfaz de usuario en la página de precios de VM](./media/pricing-guidance/virtual-machines-pricing-ui.png)
>
> O bien, use la [calculadora de precios](https://azure.microsoft.com/pricing/#explore-cost) para agregar y configurar una máquina virtual. 

## <a name="free-licensed-sql-server-editions"></a>Ediciones de SQL Server con licencia gratuita

Si quiere desarrollar, probar o compilar una prueba de concepto, use la versión **SQL Server Developer Edition** con licencia gratuita. Esta edición incluye todas las características de SQL Server Enterprise Edition, lo que le permite compilar y probar cualquier tipo de aplicación. Pero no puede ejecutar la edición Developer en producción. Una máquina virtual de SQL Server Developer Edition solo incurre en cargos por el costo de la máquina virtual, porque no hay ningún costo de licencia de SQL Server asociado.

Si quiere ejecutar una carga de trabajo ligera en producción (< 4 núcleos, < 1 GB de memoria, < 10 GB/base de datos), use la versión **SQL Server Express Edition** con licencia gratuita. Una máquina virtual de SQL Server Express Edition solo incurre en cargos por el costo de la máquina virtual.

Para estas cargas de trabajo de producción ligera y de desarrollo y pruebas, también puede ahorrar dinero si elige un tamaño de máquina virtual más pequeño adecuado a estas cargas de trabajo. DS1v2 puede ser una buena elección en algunos escenarios.

Para crear una máquina virtual de Azure que ejecute SQL Server 2017 con alguna de estas imágenes, vea los siguientes vínculos:

| Plataforma | Imágenes con licencia gratuita |
|---|---|
| Windows Server 2016 | [Máquina virtual de Azure en SQL Server 2017 Developer](https://portal.azure.com/#create/Microsoft.FreeSQLServerLicenseSQLServer2017DeveloperonWindowsServer2016)<br/>[Máquina virtual de Azure en SQL Server 2017 Express](https://portal.azure.com/#create/Microsoft.FreeSQLServerLicenseSQLServer2017ExpressonWindowsServer2016) |
| Red Hat Enterprise Linux | [Máquina virtual de Azure en SQL Server 2017 Developer](https://portal.azure.com/#create/Microsoft.FreeSQLServerLicenseSQLServer2017DeveloperonRedHatEnterpriseLinux74)<br/>[Máquina virtual de Azure en SQL Server 2017 Express](https://portal.azure.com/#create/Microsoft.FreeSQLServerLicenseSQLServer2017ExpressonRedHatEnterpriseLinux74) |
| SUSE Linux Enterprise Server | [Máquina virtual de Azure en SQL Server 2017 Developer](https://portal.azure.com/#create/Microsoft.FreeSQLServerLicenseSQLServer2017DeveloperonSLES12SP2)<br/>[Máquina virtual de Azure en SQL Server 2017 Express](https://portal.azure.com/#create/Microsoft.FreeSQLServerLicenseSQLServer2017ExpressonSLES12SP2) |
| Ubuntu | [Máquina virtual de Azure en SQL Server 2017 Developer](https://portal.azure.com/#create/Microsoft.FreeSQLServerLicenseSQLServer2017DeveloperonUbuntuServer1604LTS)<br/>[Máquina virtual de Azure en SQL Server 2017 Express](https://portal.azure.com/#create/Microsoft.FreeSQLServerLicenseSQLServer2017ExpressonUbuntuServer1604LTS) |

## <a name="paid-sql-server-editions"></a>Ediciones de pago de SQL Server

Si tiene una carga de trabajo de producción no ligera, use una de las siguientes ediciones de SQL Server:

| Edición de SQL Server | Carga de trabajo |
|-----|-----|
| Web | Sitios web pequeños |
| Estándar | Cargas de trabajo de pequeñas a medianas |
| Enterprise | Cargas de trabajo grandes o críticas|

Tiene dos opciones para pagar licencias de SQL Server para estas ediciones: *pago por uso* o *traiga su propia licencia (BYOL)* .

## <a name="pay-per-usage"></a>Pago por uso

**Pago en función del uso de la licencia de SQL Server** (también conocido como **pago por uso**): el costo por segundo de ejecución de la máquina virtual de Azure incluye el costo de la licencia de SQL Server. Puede ver el precio de las diferentes ediciones de SQL Server (Web, Standard y Enterprise) en la página de precios de máquinas virtuales de Azure para [Windows](https://azure.microsoft.com/pricing/details/virtual-machines/windows) o [Linux](https://azure.microsoft.com/pricing/details/virtual-machines/linux).

El costo es el mismo para todas las versiones de SQL Server (2012 SP3 a 2019). Los costos de licencia por segundo dependen del número de vCPU de máquina virtual.

Se recomienda pagar las licencias por uso de SQL Server en los siguientes casos:

- **Cargas de trabajo temporales o periódicas**. Por ejemplo, una aplicación que necesita admitir un evento durante un par de meses cada año, o bien análisis de negocio todos los lunes.

- **Cargas de trabajo con una duración o escala desconocidas**. Por ejemplo, una aplicación que puede no ser necesaria en unos meses, o que puede requerir más o menos capacidad de proceso, según la demanda.

Para crear una máquina virtual de Azure que ejecute SQL Server 2017 con alguna de estas imágenes de pago por uso, vea los siguientes vínculos:

| Plataforma | Imágenes con licencia |
|---|---|
| Windows Server 2016 | [Máquina virtual de Azure en SQL Server 2017 Web](https://portal.azure.com/#create/Microsoft.SQLServer2017WebonWindowsServer2016)<br/>[Máquina virtual de Azure en SQL Server 2017 Standard](https://portal.azure.com/#create/Microsoft.SQLServer2017StandardonWindowsServer2016)<br/>[Máquina virtual de Azure en SQL Server 2017 Enterprise](https://portal.azure.com/#create/Microsoft.SQLServer2017EnterpriseWindowsServer2016) |
| Red Hat Enterprise Linux | [Máquina virtual de Azure en SQL Server 2017 Web](https://portal.azure.com/#create/Microsoft.SQLServer2017WebonRedHatEnterpriseLinux74)<br/>[Máquina virtual de Azure en SQL Server 2017 Standard](https://portal.azure.com/#create/Microsoft.SQLServer2017StandardonRedHatEnterpriseLinux74)<br/>[Máquina virtual de Azure en SQL Server 2017 Enterprise](https://portal.azure.com/#create/Microsoft.SQLServer2017EnterpriseonRedHatEnterpriseLinux74) |
| SUSE Linux Enterprise Server | [Máquina virtual de Azure en SQL Server 2017 Web](https://portal.azure.com/#create/Microsoft.SQLServer2017WebonSLES12SP2)<br/>[Máquina virtual de Azure en SQL Server 2017 Standard](https://portal.azure.com/#create/Microsoft.SQLServer2017StandardonSLES12SP2)<br/>[Máquina virtual de Azure en SQL Server 2017 Enterprise](https://portal.azure.com/#create/Microsoft.SQLServer2017EnterpriseonSLES12SP2) |
| Ubuntu | [Máquina virtual de Azure en SQL Server 2017 Web](https://portal.azure.com/#create/Microsoft.SQLServer2017WebonUbuntuServer1604LTS)<br/>[Máquina virtual de Azure en SQL Server 2017 Standard](https://portal.azure.com/#create/Microsoft.SQLServer2017StandardonUbuntuServer1604LTS)<br/>[Máquina virtual de Azure en SQL Server 2017 Enterprise](https://portal.azure.com/#create/Microsoft.SQLServer2017EnterpriseonUbuntuServer1604LTS) |

> [!IMPORTANT]
> Cuando se crea una máquina virtual de SQL Server en Azure Portal, la ventana **Elegir un tamaño** muestra un costo estimado. Es importante tener en cuenta que esta estimación incluye solo los costos de proceso derivados de la ejecución de la máquina virtual junto con los costos de licencia del sistema operativo (Windows o sistemas operativos Linux de terceros).
>
> ![Hoja Elegir un tamaño](./media/pricing-guidance/sql-vm-choose-size-pricing-estimate.png)
>
>No incluye los costos de licencia adicionales de SQL Server para las ediciones Web, Standard y Enterprise. Para obtener la estimación más precisa de precios, seleccione el sistema operativo y la edición de SQL Server en la página de precios de [Windows](https://azure.microsoft.com/pricing/details/virtual-machines/windows/) o [Linux](https://azure.microsoft.com/pricing/details/virtual-machines/linux/).

> [!NOTE]
> Ahora es posible cambiar del modelo de licencias de pago por uso a traiga su propia licencia (BYOL) y viceversa. Para más información, consulte [Modificación del modelo de licencia para una máquina virtual de SQL Server en Azure](licensing-model-azure-hybrid-benefit-ahb-change.md). 

## <a name="bring-your-own-license-byol"></a><a id="byol"></a> Traiga su propia licencia (BYOL)

**Traiga su propia licencia de SQL Server a través de License Mobility**, que también se conoce como **BYOL**, consiste en usar una licencia por volumen existente de SQL Server con Software Assurance en una máquina virtual de Azure. Si se trata de una máquina virtual con SQL Server con licencia BYOL, solo se carga el costo de la ejecución de la máquina virtual, no el relativo a la licencia de SQL Server, puesto que ya ha adquirido licencias y Software Assurance a través de un programa de licencias por volumen.

> [!IMPORTANT]
> Las imágenes BYOL requieren un contrato Enterprise con Software Assurance. No están disponibles como parte de Azure Cloud Solution Partner (CSP) en este momento. Los clientes de CSP pueden aportar su propia licencia al implementar una imagen de pago por uso y, a continuación, habilitar la [Ventaja híbrida de Azure](licensing-model-azure-hybrid-benefit-ahb-change.md).

> [!NOTE]
> Actualmente, las imágenes BYOL solo están disponibles para máquinas virtuales de Windows. Sin embargo, puede instalar manualmente SQL Server en una máquina virtual con solo Linux. Consulte las instrucciones de [Preguntas más frecuentes para SQL Server en máquinas virtuales Linux](../linux/frequently-asked-questions-faq.md).

Se recomienda traer su propia licencia de SQL Server a través de la Movilidad de licencias:

- **Cargas de trabajo continuas**. Por ejemplo, una aplicación que necesita admitir operaciones empresariales de forma ininterrumpida.

- **Cargas de trabajo con una duración y escala desconocidas**. Por ejemplo, una aplicación que sea necesaria durante todo el año y cuya demanda ya se ha previsto.

Para usar BYOL con una máquina virtual con SQL Server, debe tener una licencia para SQL Server Standard o Enterprise y [Software Assurance](https://www.microsoft.com/licensing/licensing-programs/software-assurance-default.aspx#tab=1), que es una opción necesaria en algunos programas de licencias por volumen y una adquisición opcional en otros. Los niveles de precios proporcionados a través de programas de licencias por volumen varían según el tipo de contrato y la cantidad y el compromiso con SQL Server. No obstante, por norma general, el modelo traiga su propia licencia para cargas de trabajo de producción continuas tiene las siguientes ventajas:

| Ventaja de BYOL | Descripción |
|-----|-----|
| **Ahorros en costos** | El [Ventaja híbrida de Azure](https://azure.microsoft.com/pricing/hybrid-benefit/) ofrece un ahorro de hasta el 55 %. Para más información, consulte el artículo sobre cómo [cambiar el modelo de licencia](licensing-model-azure-hybrid-benefit-ahb-change.md). |
| **Réplica secundaria pasiva gratuita** | Otra ventaja que aporta traer su propia licencia es la [licencia gratuita para una réplica secundaria pasiva](https://azure.microsoft.com/pricing/licensing-faq/) por SQL Server para fines de alta disponibilidad. De esta forma, se reduce a la mitad el costo de una implementación de SQL Server de alta disponibilidad (por ejemplo, el uso de grupos de disponibilidad Always On). Los derechos para ejecutar la réplica secundaria pasiva se proporcionan con la ventaja de Software Assurance en servidores de conmutación por error. |

Para crear una máquina virtual de Azure que ejecute SQL Server 2017 con alguna de estas imágenes del modelo traiga su propia licencia, vea las máquinas virtuales con el prefijo "{BYOL}":

- [Máquina virtual de Azure en SQL Server 2017 Enterprise](https://portal.azure.com/#create/Microsoft.BYOLSQLServer2017EnterpriseWindowsServer2016)
- [Máquina virtual de Azure en SQL Server 2017 Standard](https://portal.azure.com/#create/Microsoft.BYOLSQLServer2017StandardonWindowsServer2016)

> [!IMPORTANT]
> Informe en un plazo de diez días de cuántas licencias de SQL Server va a usar en Azure. Los vínculos a las imágenes anteriores tienen instrucciones sobre cómo hacerlo.

> [!NOTE]
> Ahora es posible cambiar del modelo de licencias de pago por uso a traiga su propia licencia (BYOL) y viceversa. Para más información, consulte [Modificación del modelo de licencia para una máquina virtual de SQL Server en Azure](licensing-model-azure-hybrid-benefit-ahb-change.md). 



## <a name="reduce-costs"></a>Reducir los costos

Para evitar costos innecesarios, elija un tamaño de máquina virtual óptimo y considere la posibilidad de realizar apagados intermitentes para cargas de trabajo continuas.

### <a name="correctly-size-your-vm"></a><a id="machinesize"></a> Selección del tamaño adecuado de VM

El costo de las licencias de SQL Server está directamente relacionado con el número de vCPU. Elija un tamaño de memoria virtual que coincida con sus necesidades previstas de CPU, memoria, almacenamiento y ancho de banda de E/S. Para obtener una lista completa de opciones de tamaño de máquinas, vea [Tamaños de las máquinas virtuales Windows en Azure](https://docs.microsoft.com/azure/virtual-machines/windows/sizes) y [Tamaños de las máquinas virtuales Linux en Azure](https://docs.microsoft.com/azure/virtual-machines/linux/sizes?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).

Hay nuevos tamaños de máquina que funcionan bien con determinados tipos de cargas de trabajo de SQL Server. Estos tamaños de máquinas mantienen altos niveles de memoria, almacenamiento y ancho de banda de E/S, pero tienen una cantidad inferior de núcleos virtuales. Por ejemplo, considere el siguiente ejemplo:

| Tamaño de VM | vCPU | Memoria | Discos máx. | Rendimiento de E/S máx. | Costos de licencias de SQL Server | Costos totales (proceso + licencias) |
|---|---|---|---|---|---|---|
| **Standard_DS14v2** | 16 | 112 GB | 32 | 51 200 IOPS o 768 MB/s | | |
| **Standard_DS14 4v2** | 4 | 112 GB | 32 | 51 200 IOPS o 768 MB/s | 75 % menos | 57 % menos |

> [!IMPORTANT]
> Se trata de un ejemplo de un momento en el tiempo. Para conocer las especificaciones más recientes, vea los artículos de tamaños de máquina y la página de precios de Azure para [Windows](https://azure.microsoft.com/pricing/details/virtual-machines/windows/) y [Linux](https://azure.microsoft.com/pricing/details/virtual-machines/linux/).

En el ejemplo anterior, puede ver que las especificaciones de **Standard_DS14v2** y **Standard_DS14-4v2** son idénticas, a excepción de vCPU. El sufijo **-4v2** al final del tamaño de máquina **Standard_DS14-4v2** indica el número de vCPU activos. Dado que los costos de licencia de SQL Server están asociados al número de vCPU, esto reduce considerablemente el costo derivado de VM en escenarios donde no se necesitan vCPU adicionales. Esto es solo un ejemplo, y hay muchos tamaños de máquina con vCPU restringidas que se identifican con este patrón de sufijo. Para obtener más información, consulte la entrada de blog [Announcing new Azure VM sizes for more cost-effective database work](https://azure.microsoft.com/blog/announcing-new-azure-vm-sizes-for-more-cost-effective-database-workloads/) (Anuncio de nuevos tamaños de VM para un trabajo de base de datos rentable).

### <a name="shut-down-your-vm-when-possible"></a>Apagado de la máquina virtual siempre que sea posible

Si usa cargas de trabajo que no se ejecutan continuamente, considere la posibilidad de apagar la máquina virtual durante los períodos inactivos. Pague solo por lo que usa.

Por ejemplo, si solo tiene intención de probar SQL Server en una máquina virtual de Azure, no querría incurrir en cargos por dejarla en ejecución durante semanas de forma accidental. Una solución consiste en utilizar la [característica de parada automática](https://azure.microsoft.com/blog/announcing-auto-shutdown-for-vms-using-azure-resource-manager/).

![Apagado automático de máquina virtual de SQL](./media/pricing-guidance/sql-vm-auto-shutdown.png)

La parada automática forma parte de un conjunto mayor de características similares proporcionadas por [Azure DevTest Labs](https://azure.microsoft.com/services/devtest-lab).

Para otros flujos de trabajo, considere la posibilidad de parar y reiniciar automáticamente las máquinas virtuales con una solución de scripting, como [Azure Automation](https://azure.microsoft.com/services/automation/).

> [!IMPORTANT]
> Apagar la máquina virtual y desasignarla es la única manera de evitar cargos. Incurre en cargos por uso si solo detiene o usa opciones de energía para parar la máquina virtual.

## <a name="next-steps"></a>Pasos siguientes

Para consultar una guía de precios general de Azure, vea [Prevención de costos inesperados con la administración de costos y facturación de Azure](../../../cost-management-billing/manage/getting-started.md). Para conocer los precios más recientes de Azure Virtual Machines, incluido SQL Server, vea la página de precios de Azure Virtual Machines para [máquinas virtuales Windows](https://azure.microsoft.com/pricing/details/virtual-machines/windows/) y [máquinas virtuales Linux](https://azure.microsoft.com/pricing/details/virtual-machines/linux/).

Para obtener información general de SQL Server en Azure Virtual Machines, vea los siguientes artículos:

- [Introducción a SQL Server en máquinas virtuales Windows](sql-server-on-azure-vm-iaas-what-is-overview.md)
- [Introducción a SQL Server en máquinas virtuales Linux](../linux/sql-server-on-linux-vm-what-is-iaas-overview.md)
