---
title: Movimiento de un almacén de claves a una región diferente en Azure Key Vault | Microsoft Docs
description: En este artículo se ofrecen instrucciones para mover un almacén de claves a una región diferente.
services: key-vault
author: ShaneBala-keyvault
manager: ravijan
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: how-to
ms.date: 04/24/2020
ms.author: sudbalas
Customer intent: As a key vault administrator, I want to move my vault to another region.
ms.openlocfilehash: 7d58cd26b6e4ca77da98f8c2f82dbdb481ccbb50
ms.sourcegitcommit: 02ca0f340a44b7e18acca1351c8e81f3cca4a370
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88585759"
---
# <a name="move-an-azure-key-vault-across-regions"></a>Movimiento de un almacén de claves de Azure entre regiones

Azure Key Vault no admite una operación de movimiento de recursos que permita mover un almacén de claves a otra región. En este artículo se tratan las soluciones alternativas para las organizaciones que tienen una necesidad empresarial de trasladar un almacén de claves a otra región. Cada opción de solución tiene sus limitaciones. Es fundamental comprender las implicaciones de estas soluciones alternativas antes de intentarlas en un entorno de producción.

Para trasladar un almacén de claves a otra región, cree un almacén de claves en otra región y copie manualmente cada secreto individual del almacén de claves existente al nuevo. Para ello, puede usar cualquiera de las dos opciones siguientes.

## <a name="design-considerations"></a>Consideraciones de diseño

Antes de comenzar, tenga en cuenta los siguientes conceptos:

* Los nombres de los almacenes de claves son únicos globalmente. No se puede reutilizar un nombre de almacén.
* Tiene que volver a configurar las directivas de acceso y la configuración de red en el nuevo almacén de claves.
* Tiene que volver a configurar la protección de eliminación temporal y de purga en el nuevo almacén de claves.
* La operación de copia de seguridad y restauración no conservará la configuración de autogiro. Es posible que tenga que volver a configurar las opciones.

## <a name="option-1-use-the-key-vault-backup-and-restore-commands"></a>Opción 1: uso de los comandos para copia de seguridad y restauración de almacenes de claves

Puede realizar una copia de seguridad de cada secreto, clave y certificado individual del almacén mediante el comando de copia de seguridad. Los secretos se descargan como un blob cifrado. Después, puede restaurar el blob en el nuevo almacén de claves. Para obtener una lista de comandos, vea los [comandos de Azure Key Vault](https://docs.microsoft.com/powershell/module/azurerm.keyvault/?view=azurermps-6.13.0#key_vault).

El uso de los comandos backup y restore tiene dos limitaciones:

* no se puede realizar una copia de seguridad de un almacén de claves en una ubicación geográfica y restaurarlo en otra. Para obtener más información, consulte [Zonas geográficas de Azure](https://azure.microsoft.com/global-infrastructure/geographies/).

* El comando backup realiza una copia de seguridad de todas las versiones de cada secreto. Si tiene un secreto con un gran número de versiones anteriores (más de 10), existe la posibilidad de que la solicitud supere el tamaño máximo permitido de la solicitud y que se produzca un error en la operación.

## <a name="option-2-manually-download-and-upload-the-key-vault-secrets"></a>Opción 2: descarga y carga manual de los secretos del almacén de claves

Puede descargar determinados tipos de secreto manualmente. Por ejemplo, puede descargar certificados como un archivo .pfx. Esta opción elimina las restricciones geográficas de algunos tipos de secretos, como los certificados. Puede cargar los archivos .pfx en cualquier almacén de claves de cualquier región. El secreto se descarga en un formato sin protección con contraseña. Usted es responsable de proteger los secretos durante el traslado.
