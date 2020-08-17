---
title: Introducción a OpenShift en Azure
description: Introducción a OpenShift en Azure.
author: haroldwongms
manager: mdotson
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: how-to
ms.workload: infrastructure
ms.date: 05/7/2019
ms.author: haroldw
ms.openlocfilehash: 989c67d0d1b1cd8257da089ec9c31f247228018e
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87367952"
---
# <a name="openshift-in-azure"></a>OpenShift en Azure

OpenShift es una plataforma de aplicación de contenedor abierta y extensible que aporta Docker y Kubernetes a la empresa.  

OpenShift incluye Kubernetes para la administración y orquestación de contenedores. Incluye herramientas centradas en los desarrolladores y en las operaciones que permiten:

- Desarrollo rápido de aplicaciones.
- Implementación y escalado sencillos.
- Mantenimiento del ciclo de vida a largo plazo de los equipos y las aplicaciones.

Hay varias versiones de OpenShift disponibles,  pero solo dos de ellas están disponibles en la actualidad para que los clientes las implementen en Azure: OpenShift Container Platform y OKD (anteriormente OpenShift Origin).

## <a name="azure-red-hat-openshift"></a>Red Hat OpenShift en Azure

Red Hat OpenShift en Microsoft Azure es una oferta totalmente administrada de OpenShift que se ejecuta en Azure. Microsoft y Red Hat realizan la administración y el soporte técnico de este servicio conjuntamente. Para saber más, vea [Servicio Red Hat OpenShift en Azure](../../openshift/index.yml).

## <a name="openshift-container-platform"></a>OpenShift Container Platform

Container Platform es una [versión comercial](https://www.openshift.com) preparada para la empresa, que procede de Red Hat y que cuenta con su respaldo. Con esta versión, los clientes adquieren los derechos necesarios para OpenShift Container Platform y son responsables de la instalación y la administración de toda la infraestructura.

Puesto que el cliente "posee" toda la plataforma, puede instalarla en su centro de datos local, o bien en una nube pública (como Azure).

## <a name="okd"></a>OKD

OKD es un proyecto ascendente de [código abierto](https://www.okd.io/) de OpenShift al que da soporte la comunidad. OKD puede instalarse en CentOS o Red Hat Enterprise Linux (RHEL).

## <a name="next-steps"></a>Pasos siguientes

- [Configuración de los requisitos previos comunes para OpenShift en Azure](./openshift-container-platform-3x-prerequisites.md)
- [Implementación de OpenShift Container Platform en Azure](./openshift-container-platform-3x.md)
- [Implementación con la oferta de Marketplace de OpenShift Container Platform autoadministrada](./openshift-container-platform-3x-marketplace-self-managed.md)
- [Implementación de OpenShift en Azure Stack](./openshift-azure-stack.md)
- [Tareas posteriores a la implementación](./openshift-container-platform-3x-post-deployment.md)
- [Solución de problemas de implementación de OpenShift](./openshift-container-platform-3x-troubleshooting.md)
