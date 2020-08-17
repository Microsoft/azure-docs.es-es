---
title: Administración de bases de datos con Azure Automation
description: Obtenga información sobre cómo puede usarse el servicio Azure Automation para administrar Azure SQL Database a escala.
services: sql-database
ms.service: sql-db-mi
ms.subservice: service
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: juliemsft
ms.author: jrasnick
ms.reviewer: carlrab
ms.date: 03/12/2019
ms.openlocfilehash: b90acafb9146ec6cd796a0916ad46bf880398b22
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87013987"
---
# <a name="manage-databases-in-azure-sql-database-by-using-azure-automation"></a>Administración de bases de datos en Azure SQL Database mediante Azure Automation

[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Esta guía le ofrece el servicio Azure Automation y cómo se puede usar para simplificar la administración de las bases de datos en Azure SQL Database.

## <a name="about-azure-automation"></a>Información sobre Azure Automation

[Azure Automation](https://azure.microsoft.com/services/automation/) es un servicio de Azure para simplificar la administración en la nube mediante la automatización de procesos. Mediante Azure Automation, se pueden automatizar las tareas de ejecución prolongada, manuales, propensas a errores y que se repiten con frecuencia para aumentar la confiabilidad, la eficiencia y el valioso tiempo para su organización. Para obtener información sobre cómo empezar, consulte [Introducción a Azure Automation](../../automation/automation-intro.md).

Azure Automation proporciona un motor de ejecución de flujo de trabajo altamente confiable y de alta disponibilidad con escalado para satisfacer sus necesidades a medida que crece la organización. En Azure Automation, los sistemas de terceros pueden interrumpir los procesos manualmente o en intervalos programados para que las tareas se realicen justo cuando sea necesario.

Reduzca la sobrecarga operativa y libere al personal de TI/DevOps para concentrarse en el trabajo que proporciona valor al negocio trasladando las tareas de administración en la nube para que se ejecuten automáticamente mediante Azure Automation.

## <a name="how-azure-automation-can-help-manage-your-databases"></a>Cómo Azure Automation puede ayudar a administrar las bases de datos

Con Azure Automation puede administrar bases de datos en Azure SQL Database mediante los [cmdlets de PowerShell](/powershell/module/servicemanagement/azure.service/#sql) que están disponibles en las [herramientas de Azure PowerShell](/powershell/azure/). Azure Automation tiene estos cmdlets de PowerShell para Azure SQL Database disponibles directamente para que pueda realizar todas las tareas de administración de su base de datos SQL dentro del servicio. También puede emparejar estos cmdlets en Azure Automation con los cmdlets para otros servicios de Azure, para automatizar tareas complejas entre los servicios de Azure y sistemas de terceros.

Azure Automation también tiene la capacidad de comunicarse con servidores SQL Server directamente, mediante la emisión de comandos SQL con PowerShell.

Las galerías de runbooks y módulos de [Azure Automation](../../automation/automation-runbook-gallery.md) ofrecen diversos runbooks de Microsoft y la comunidad que se pueden importar a Azure Automation. Puede usar uno al descargarlo de la galería, o puede importar runbooks directamente desde la galería o de la cuenta de Automation en Azure Portal.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido los aspectos básicos de Azure Automation y cómo se puede usar para administrar Azure SQL Database, siga estos vínculos para obtener más información acerca de Azure Automation.

- [Información general sobre Azure Automation](../../automation/automation-intro.md)
- [Mi primer runbook](../../automation/learn/automation-tutorial-runbook-graphical.md)
