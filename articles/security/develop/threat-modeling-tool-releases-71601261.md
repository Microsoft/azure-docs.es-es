---
title: Microsoft Threat Modeling Tool, versión 29/01/2019
titleSuffix: Azure
description: Lea las notas de la versión de Microsoft Threat Modeling Tool publicada el 29/01/2019. Las notas incluyen cambios de características y problemas conocidos.
author: jegeib
ms.author: jegeib
ms.service: security
ms.subservice: security-develop
ms.topic: article
ms.date: 01/25/2019
ms.openlocfilehash: 17147d412bd888cdd3cd270829ad6d6103867b34
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87539057"
---
# <a name="threat-modeling-tool-update-release-71601261---1292019"></a>Threat Modeling Tool, versión actualizada 7.1.60126.1: 29 de enero de 2019

El 29 de enero de 2019 se lanzó la versión 7.1.60126.1 de Microsoft Threat Modeling Tool, que incluye los cambios siguientes:

- La versión mínima necesaria de .NET ha aumentado a [.NET 4.7.1](https://go.microsoft.com/fwlink/?LinkId=863262).
- La versión mínima necesaria de Windows ha aumentado a [Actualización de aniversario de Windows 10](https://blogs.windows.com/windowsexperience/2016/08/02/how-to-get-the-windows-10-anniversary-update/#HTkoK5Zdv0g2F2Zq.97) debido a la dependencia de .NET.
- Se ha agregado una característica de alternancia de validación de modelos al menú Opciones de la herramienta.
- Se han actualizado varios vínculos en las propiedades de las amenazas.
- Se han realizado algunos cambios menores en la experiencia del usuario en la página principal de la herramienta.
- Threat Modeling Tool hereda ahora la configuración de TLS del sistema operativo host y se admite en entornos que requieren TLS 1.2 o posterior.

## <a name="feature-changes"></a>Cambios de características

### <a name="model-validation-option"></a>Opción de validación del modelo

Según los comentarios de los clientes, se ha agregado una opción a la herramienta para habilitar o deshabilitar la validación del modelo. Anteriormente, si la plantilla usó un único flujo de datos unidireccional entre dos objetos, es posible que haya recibido un mensaje de error en el marco de mensajes en el que se indica: ObjectsName requiere al menos una instancia de "Any". Al deshabilitar la validación del modelo, se impedirá que estas advertencias se muestren en la vista.

La opción para alternar la validación del modelo se puede encontrar en el menú Archivo -> Configuración -> Opciones. El valor predeterminado para esta opción es Deshabilitado.

![Opción de validación del modelo](./media/threat-modeling-tool-releases-71601261/tmt_model_validation_option.png)

## <a name="system-requirements"></a>Requisitos del sistema

- Sistemas operativos compatibles
  - [Actualización de aniversario de Microsoft Windows 10](https://blogs.windows.com/windowsexperience/2016/08/02/how-to-get-the-windows-10-anniversary-update/#HTkoK5Zdv0g2F2Zq.97) o posterior
- Versión de .NET necesaria
  - [.NET 4.7.1](https://go.microsoft.com/fwlink/?LinkId=863262) o posterior
- Requisitos adicionales
  - Se necesita una conexión a Internet para recibir actualizaciones de la herramienta, así como plantillas.

## <a name="known-issues"></a>Problemas conocidos

### <a name="unsupported-systems"></a>Sistemas no admitidos

#### <a name="issue"></a>Problema

Los usuarios de los sistemas de Windows 10 que no pueden instalar .NET 4.7.1 o versiones posteriores, como Windows 10 Enterprise LTSB (versión 1507), no podrán abrir la herramienta tras la actualización. Estas versiones anteriores de Windows ya no son plataformas admitidas de Threat Modeling Tool y no deben instalar la actualización más reciente.

#### <a name="workaround"></a>Solución alternativa

Los usuarios de Windows 10 Enterprise LTSB (versión 1507) que han instalado la actualización más reciente pueden revertir a la versión anterior de Threat Modeling Tool mediante el cuadro de diálogo de desinstalación de Aplicaciones y características.

## <a name="documentation-and-feedback"></a>Documentación y comentarios

- La documentación de Threat Modeling Tool se encuentra en [docs.microsoft.com](threat-modeling-tool.md) e incluye información [sobre el uso de la herramienta](threat-modeling-tool-getting-started.md).

## <a name="next-steps"></a>Pasos siguientes

Descargue la versión más reciente de [Microsoft Threat Modeling Tool](https://aka.ms/threatmodelingtool).
