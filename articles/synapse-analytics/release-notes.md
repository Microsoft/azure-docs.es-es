---
title: 'Notas de la versión: Azure Synapse Analytics (áreas de trabajo)'
description: Notas de la versión para Azure Synapse Analytics (áreas de trabajo)
services: synapse-analytics
author: julieMSFT
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: ''
ms.date: 04/15/2020
ms.author: jrasnick
ms.reviewer: jrasnick
ms.openlocfilehash: add5c89e83f33980803bf571239023859653c4f1
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87059606"
---
# <a name="azure-synapse-analytics-preview-release-notes"></a>Notas de la versión de Azure Synapse Analytics (versión preliminar)

En este artículo se describen las limitaciones y los problemas con Azure Synapse Analytics (áreas de trabajo). Para más información, consulte [¿Qué es Azure Synapse Analytics (versión preliminar para áreas de trabajo)?](overview-what-is.md)

[!INCLUDE [preview](includes/note-preview.md)]

## <a name="azure-synapse-workspaces"></a>Azure Synapse (áreas de trabajo) 

### <a name="azure-synapse-cli"></a>CLI de Azure Synapse

- Problema e impacto en el cliente: Las áreas de trabajo creadas por el SDK no pueden iniciar Synapse Studio.

- Solución alternativa: Complete los pasos siguientes: 
  1.    Cree el área de trabajo mediante la ejecución de `az synapse workspace create`.
  2.    Extraiga el identificador de identidad administrada mediante la ejecución de `$identity=$(az synapse workspace show --name {workspace name}  --resource-group {resource group name} --query "identity.principalId")`.
  3.    Agregue el área de trabajo como rol a la cuenta de almacenamiento mediante la ejecución de ` az role assignment create --role "Storage Blob Data Contributor" --assignee-object-id {identity } --scope {storage account resource id}`.
  4.    Agregue la regla de firewall mediante la ejecución de ` az synapse firewall-rule create --name allowAll --start-ip-address 0.0.0.0 --end-ip-address 255.255.255.255 `.

## <a name="next-steps"></a>Pasos siguientes

* [Creación de un área de trabajo](quickstart-create-workspace.md)
* [Inicio rápido: Uso de Synapse Studio (versión preliminar)](quickstart-synapse-studio.md)
* [Creación de un grupo de SQL](quickstart-create-sql-pool-portal.md)
* [Uso de SQL a petición](quickstart-sql-on-demand.md)
* [Creación de un grupo de Apache Spark](quickstart-create-apache-spark-pool-portal.md)