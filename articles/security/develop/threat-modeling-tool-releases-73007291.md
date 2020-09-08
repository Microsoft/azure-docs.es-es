---
title: 'Microsoft Threat Modeling Tool, versión 07/29/2020: Azure'
description: Documentación de las notas de la versión de la herramienta de modelado de amenazas
author: jegeib
ms.author: jegeib
ms.service: security
ms.topic: article
ms.date: 07/29/2020
ms.openlocfilehash: 47ffe753d661bb8c9a181148ea235b2a0eb54400
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87839833"
---
# <a name="threat-modeling-tool-update-release-73007291---07292020"></a>Threat Modeling Tool, versión actualizada 7.3.00729.1: 29 de julio de 2020

El 29 de julio de 2020 se lanzó la versión 7.3.00729.1 de Microsoft Threat Modeling Tool (TMT), que incluye los cambios siguientes:

- Corrección de errores
 
## <a name="known-issues"></a>Problemas conocidos

### <a name="errors-related-to-tmt7application-file-deserialization"></a>Errores relacionados con la deserialización del archivo TMT7.application

#### <a name="issue"></a>Problema

Algunos clientes han informado que reciben el siguiente mensaje de error al abrir descargar Threat Modeling Tool:

```
The threat model file '$PATH\TMT7.application' could not be deserialized. File is not an actual threat model or the threat model may be corrupted.
```

Este error se produce porque algunos exploradores no admiten de forma nativa la instalación de ClickOnce. En esos casos, el archivo de aplicación ClickOnce se descarga en el disco duro del usuario.

#### <a name="workaround"></a>Solución alternativa

Este error seguirá apareciendo si Threat Modeling Tool se inicia haciendo doble clic en el archivo TMT7.application. Sin embargo, después de omitir el error, la herramienta funcionará con normalidad. En lugar de iniciar Threat Modeling Tool haciendo doble clic en el archivo TMT7.application, los usuarios deben emplear accesos directos creados en el menú de Windows durante la instalación para iniciar Threat Modeling Tool.

## <a name="system-requirements"></a>Requisitos del sistema

- Sistemas operativos compatibles
  - [Actualización de aniversario de Microsoft Windows 10](https://blogs.windows.com/windowsexperience/2016/08/02/how-to-get-the-windows-10-anniversary-update/#HTkoK5Zdv0g2F2Zq.97) o posterior
- Versión de .NET necesaria
  - [.NET 4.7.1](https://go.microsoft.com/fwlink/?LinkId=863262) o posterior
- Requisitos adicionales
  - Se necesita una conexión a Internet para recibir actualizaciones de la herramienta, así como plantillas.

## <a name="documentation-and-feedback"></a>Documentación y comentarios

- La documentación de Threat Modeling Tool se encuentra en [docs.microsoft.com](https://docs.microsoft.com/azure/security/azure-security-threat-modeling-tool) e incluye información [sobre el uso de la herramienta](https://docs.microsoft.com/azure/security/azure-security-threat-modeling-tool-getting-started).

## <a name="next-steps"></a>Pasos siguientes

Descargue la versión más reciente de [Microsoft Threat Modeling Tool](https://aka.ms/threatmodelingtool).
