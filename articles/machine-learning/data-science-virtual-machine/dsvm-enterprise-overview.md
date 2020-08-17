---
title: Análisis de equipos y entorno de inteligencia artificial
titleSuffix: Azure Data Science Virtual Machine
description: Patrones para implementar Data Science VM como entorno de equipos de empresa.
keywords: aprendizaje profundo, AI, herramientas de ciencia de datos, data science virtual machine, análisis geoespacial, proceso de ciencia de datos en equipo
services: machine-learning
ms.service: machine-learning
ms.subservice: data-science-vm
author: vijetajo
ms.author: vijetaj
ms.topic: overview
ms.date: 05/08/2018
ms.openlocfilehash: b1357c9bb125cb881ac4aa6dd31c9dcaf53954f0
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87919946"
---
# <a name="data-science-virtual-machine-based-team-analytics-and-ai-environment"></a>Entorno de análisis de equipo e IA basado en Data Science Virtual Machine 
[Data Science Virtual Machine](overview.md) (DSVM) ofrece un completo entorno en la plataforma Azure con software pregenerado de inteligencia artificial (IA) y análisis de datos.

Tradicionalmente, DSVM se ha usado como un escritorio de análisis individuales. Los científicos de datos individuales obtienen productividad con este entorno de análisis compartido y pregenerado. Dado que los equipos de análisis grandes planean los entornos para sus científicos de datos y desarrolladores de IA, uno de los temas recurrentes es una infraestructura de análisis compartida para desarrollo y experimentación. Esta infraestructura se administra en línea con las directivas de TI empresariales que también facilitan la colaboración y la coherencia entre los equipos de ciencia y análisis de datos.

Una infraestructura compartida permite un mejor uso de TI del entorno de análisis. Algunas organizaciones denominan a la infraestructura de ciencia o análisis de datos basada en el equipo *espacio aislado de análisis*. Este espacio permite a los científicos de datos acceder a varios recursos de datos para comprender rápidamente los datos. También ayuda a los científicos de datos a ejecutar experimentos, validar supuestos y crear modelos predictivos sin que ello afecte al entorno de producción.

Dado que DSVM funciona en el nivel de infraestructura de Azure, los administradores de TI pueden configurar con facilidad DSVM para que funcione conforme a las directivas de TI de la empresa. DSVM ofrece total flexibilidad en la implementación de distintas arquitecturas de uso compartido, al tiempo que también ofrece acceso a los recursos de datos corporativos de forma controlada.

En esta sección se describen algunos patrones e instrucciones que pueden utilizarse para implementar la máquina virtual de ciencia de datos como una infraestructura de ciencia de datos basada en equipos. Como los bloques de creación de estos patrones proceden de la infraestructura como servicio (IaaS) de Azure, se aplican a todas las máquinas virtuales de Azure. Esta serie de artículos se centra en la aplicación de estas funcionalidades estándar de la infraestructura de Azure a DSVM.

Algunos de los principales bloques de creación de un entorno de análisis de equipo empresarial son:

* [Un grupo de escalabilidad automática de instancias de DSVM](dsvm-pools.md)
* [Identidad común y acceso a un área de trabajo desde cualquier máquina virtual DSVM del grupo](dsvm-common-identity.md)
* [Acceso seguro a orígenes de datos](dsvm-secure-access-keys.md)


En esta serie se proporcionan instrucciones y referencias para cada uno de los elementos anteriores. No se tratan todas las consideraciones y requisitos de implementación de DSVM en configuraciones empresariales de gran tamaño. A continuación se incluyen algunos otros recursos de Azure que puede usar al implementar instancias de DSVM en su empresa:

* [Seguridad de las redes](https://docs.microsoft.com/azure/security/fundamentals/network-security)
* [Supervisión](https://docs.microsoft.com/azure/virtual-machines/windows/monitor) y [administración](https://docs.microsoft.com/azure/virtual-machines/windows/maintenance-and-updates)
* [Registro y auditoría](https://docs.microsoft.com/azure/security/fundamentals/log-audit)
* [Control de acceso basado en roles de Azure (Azure RBAC)](https://docs.microsoft.com/azure/role-based-access-control/overview)
* [Cumplimiento y configuración de directivas](../../governance/policy/overview.md)
* [Antimalware](https://docs.microsoft.com/azure/security/fundamentals/antimalware)
* [Cifrado](https://docs.microsoft.com/azure/virtual-machines/windows/disk-encryption-overview)
* [Gobernanza y detección de datos](https://docs.microsoft.com/azure/data-catalog/)

El [Centro de arquitectura de Azure](https://docs.microsoft.com/azure/architecture/) proporciona una arquitectura detallada de un extremo a otro y modelos para compilar y administrar la infraestructura de análisis basada en la nube.