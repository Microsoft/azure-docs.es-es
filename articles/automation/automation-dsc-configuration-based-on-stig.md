---
title: Configurar datos basados en STIG para Azure Automation State Configuration
description: En este artículo se indica cómo configurar datos basados en STIG del DoD para Azure Automation State Configuration.
keywords: dsc,powershell,configuration,setup
services: automation
ms.service: automation
ms.subservice: dsc
author: mgreenegit
ms.author: migreene
ms.date: 08/08/2019
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: dab0cd7f7d660808b4ed7a91318baad55f80928c
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87015143"
---
# <a name="configure-data-based-on-security-technical-information-guide-stig"></a>Configuración de según la guía de implementación técnica de seguridad (STIG)

> Se aplica a: Windows PowerShell 5.1

Crear contenido de configuración por primera vez puede ser un reto.
En muchos casos, el objetivo es automatizar la configuración de los servidores después de una "línea de base" que, con suerte, se alinea con una recomendación del sector.

> [!NOTE]
> En este artículo se hace referencia a una solución que se mantiene en la comunidad de código abierto.
> La compatibilidad solo está disponible en forma de colaboración en GitHub, no con Microsoft.

## <a name="community-project-powerstig"></a>Proyecto de la comunidad: PowerSTIG

Un proyecto de la comunidad denominado [PowerSTIG](https://github.com/microsoft/powerstig) pretende resolver este problema mediante la generación de contenido DSC basado en la [información pública](https://public.cyber.mil/stigs/) proporcionada sobre la guía de implementación técnica de seguridad (STIG, por sus siglas en inglés).

Tratar con las líneas de base es más complicado de lo que parece.
Muchas organizaciones necesitan [documentar las excepciones](https://github.com/microsoft/powerstig#powerstigdata) a las reglas y administrar los datos a escala.
Para hacer frente a este problema, PowerSTIG proporciona [recursos compuestos](https://github.com/microsoft/powerstig#powerstigdsc) para abordar cada área de la configuración, en lugar de intentar abordar toda la configuración de un archivo grande.

Una vez que se han generado las configuraciones, puede usar los [scripts de configuración de DSC](/powershell/scripting/dsc/configurations/configurations) para generar archivos MOF y [cargarlos en Azure Automation](./tutorial-configure-servers-desired-state.md#create-and-upload-a-configuration-to-azure-automation).
A continuación, registre los servidores desde una [ubicación local](./automation-dsc-onboarding.md#enable-physicalvirtual-linux-machines) o [en Azure](./automation-dsc-onboarding.md#enable-azure-vms) para extraer las configuraciones.

Para probar PowerSTIG, visite la [Galería de PowerShell](https://www.powershellgallery.com) y descargue la solución o haga clic en "Project Site" (Sitio del proyecto) para ver la [documentación](https://github.com/microsoft/powerstig).

## <a name="next-steps"></a>Pasos siguientes

- Para comprender DSC de PowerShell, vea [Información general sobre Desired State Configuration de PowerShell](/powershell/scripting/dsc/overview/overview).
- Obtenga información sobre los recursos de DSC de PowerShell en [Recursos de DSC](/powershell/scripting/dsc/resources/resources).
- Para obtener información detallada sobre la configuración de Configuration Manager, vea [Configuración de Configuration Manager local](/powershell/scripting/dsc/managing-nodes/metaconfig).
