---
title: 'Problemas conocidos: Azure Digital Twins'
description: Obtenga ayuda para reconocer y mitigar los problemas conocidos de Azure Digital Twins.
author: baanders
ms.author: baanders
ms.topic: troubleshooting
ms.service: digital-twins
ms.date: 07/14/2020
ms.openlocfilehash: bdde2076039a6f7687e06edef6dfd6f6f5148ce4
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87044146"
---
# <a name="known-issues-in-azure-digital-twins"></a>Problemas conocidos en Azure Digital Twins

En este artículo se proporciona información sobre los problemas conocidos asociados a Azure Digital Twins.

## <a name="400-client-error-bad-request-in-cloud-shell"></a>"400 Error de cliente: Solicitud incorrecta" en Cloud Shell

Los comandos de Cloud Shell pueden producir intermitentemente el error "400 Error de cliente: Solicitud incorrecta de la dirección URL: http://localhost:50342/oauth2/token"seguida del seguimiento de la pila completa.

### <a name="troubleshooting-steps"></a>Pasos para solucionar problemas

Esto puede resolverse si se vuelve a ejecutar el comando `az login` y se completan los pasos de inicio de sesión posteriores.

Después de esto, debería poder ejecutar el comando nuevamente.

### <a name="possible-causes"></a>Causas posibles

Este es el resultado de un problema conocido en Cloud Shell: [*Error intermitente al obtener el token de Cloud Shell con 400 Error de cliente: Solicitud incorrecta*](https://github.com/Azure/azure-cli/issues/11749).

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre la seguridad y los permisos de Azure Digital Twins:
* [*Conceptos: Seguridad para las soluciones de Azure Digital Twins*](concepts-security.md)