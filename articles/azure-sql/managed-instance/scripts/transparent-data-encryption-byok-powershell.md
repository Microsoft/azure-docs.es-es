---
title: 'PowerShell: Habilitación del cifrado de datos transparente de Bring Your Own Key (BYOK)'
titleSuffix: Azure SQL Managed Instance
description: Aprenda a configurar una Instancia administrada de Azure SQL Database para comenzar a usar el cifrado de datos transparente (TDE) con Bring Your Own Key (BYOK) para cifrado en reposo mediante PowerShell.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: security
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: MladjoA
ms.author: mlandzic
ms.reviewer: vanto, carlrab
ms.date: 08/25/2020
ms.openlocfilehash: c812dfa0be3e5ced40b7f347639aeb2540b75351
ms.sourcegitcommit: 927dd0e3d44d48b413b446384214f4661f33db04
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88871537"
---
# <a name="transparent-data-encryption-in-sql-managed-instance-using-your-own-key-from-azure-key-vault"></a>Cifrado de datos transparente en una Instancia administrada de SQL con su propia clave desde Azure Key Vault

[!INCLUDE[appliesto-sqldb](../../includes/appliesto-sqlmi.md)]

Este ejemplo de script de PowerShell configura el cifrado de datos transparente (TDE) con una clave administrada por el cliente para Instancia administrada de Azure SQL, mediante una clave de Azure Key Vault. Con frecuencia esto se denomina escenario de Bring Your Own Key para TDE. Para más información, consulte [Cifrado de datos transparente de Azure SQL con una clave administrada por el cliente](../../database/transparent-data-encryption-byok-overview.md).

## <a name="prerequisites"></a>Requisitos previos

- Una instancia administrada ya existente. Consulte [Uso de PowerShell para crear una instancia administrada](create-configure-managed-instance-powershell.md).

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]
[!INCLUDE [updated-for-az](../../../../includes/updated-for-az.md)]
[!INCLUDE [cloud-shell-try-it.md](../../../../includes/cloud-shell-try-it.md)]

Tanto el uso local de PowerShell como el de Azure Cloud Shell requieren Azure PowerShell 2.3.2 o una versión posterior. Si necesita realizar la actualización, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps) o ejecute el script de ejemplo siguiente para instalar el módulo en el usuario actual:

`Install-Module -Name Az -AllowClobber -Scope CurrentUser`

Si PowerShell se ejecuta localmente, también debe ejecutar `Connect-AzAccount` para crear una conexión con Azure.

## <a name="sample-scripts"></a>Muestras de scripts 

[!code-powershell-interactive[main](../../../../powershell_scripts/sql-database/transparent-data-encryption/setup-tde-byok-sqlmi.ps1 "Set up BYOK TDE for SQL Managed Instance")]

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/).

Puede encontrar más ejemplos de scripts de PowerShell para Instancia administrada de SQL en [Scripts de PowerShell de Instancia administrada de Azure SQL](../../database/powershell-script-content-guide.md).
