---
title: Introducción a Red Hat OpenShift en Azure | Microsoft Docs
description: Aprenda las características y ventajas de Red Hat OpenShift en Microsoft Azure para implementar y administrar aplicaciones basadas en contenedor.
services: container-service
author: tylermsft
ms.author: twhitney
ms.service: container-service
manager: jeconnoc
ms.topic: overview
ms.date: 05/08/2019
ms.custom: mvc
ms.openlocfilehash: bfa3e63a433a6948f05ece7ec3c3a6180933510b
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/09/2019
ms.locfileid: "65466161"
---
# <a name="azure-red-hat-openshift"></a>Red Hat OpenShift en Azure

El servicio *Red Hat OpenShift en Microsoft* Azure permite implementar clústeres de [OpenShift](https://www.openshift.com/) totalmente administrados.

Red Hat OpenShift en Azure amplía [Kubernetes](https://kubernetes.io/). La ejecución de contenedores en producción con Kubernetes requiere herramientas y recursos adicionales como, por ejemplo, un registro de imágenes, administración de almacenamiento, soluciones de red y herramientas de registro y supervisión. Todo ello debe modificarse para crear versiones y probarse de forma conjunta. La creación de aplicaciones basadas en contenedor requiere un trabajo de integración aún mayor con middleware, marcos, bases de datos y herramientas de CI/CD. Red Hat OpenShift en Azure combina todo esto en una sola plataforma, ofreciendo facilidad de operaciones a los equipos de TI mientras proporciona a los equipos de aplicaciones lo que tienen que ejecutar.

Red Hat y Microsoft han diseñado, operado y admitido Red Hat OpenShift en Azure de forma conjunta para ofrecer una experiencia de soporte integrado. No hay máquinas virtuales que operar y no se requiere ninguna aplicación de revisiones. Red Hat y Microsoft revisan, actualizan y supervisan los nodos maestros, de infraestructura y aplicación en su nombre. Sus clústeres de Red Hat OpenShift en Azure se implementan en su suscripción a Azure y se incluyen en su factura de Azure.

Puede elegir sus propias soluciones de CI/CD, almacenamiento, red y registro, o bien usar las soluciones integradas para la administración de código fuente automatizada, la creación de contenedores y aplicaciones, implementaciones, el escalado, la administración del estado, etc. Red Hat OpenShift en Azure proporciona una experiencia de inicio de sesión integrada a través de Azure Active Directory.

Para comenzar, complete el tutorial [Create an Azure Red Hat OpenShift cluster](tutorial-create-cluster.md) (Creación de un clúster de Red Hat OpenShift en Azure).

## <a name="access-security-and-monitoring"></a>Acceso, seguridad y supervisión

Para una seguridad y administración mejoradas, Red Hat OpenShift en Azure le permite integrarse con Azure Active Directory (Azure AD) y usar el control de acceso basado en rol (RBAC) de Kubernetes. También puede supervisar el mantenimiento del clúster y recursos.

## <a name="cluster-and-node"></a>Clúster y nodo

Los nodos de Red Hat OpenShift en Azure se ejecutan en máquinas virtuales de Azure. Puede conectar el almacenamiento a nodos y pods, actualizar los componentes de clúster y usar GPU.

## <a name="virtual-networks-and-ingress"></a>Redes virtuales y entrada

Puede conectar un clúster de Red Hat OpenShift en Azure en una red virtual existente mediante emparejamiento. En esta configuración, los pods se pueden conectar con otros servicios de una red virtual emparejada, y con redes locales a través de [ExpressRoute](https://docs.microsoft.com/azure/expressroute/) o conexiones VPN de sitio a sitio (S2S).

See [Conexión de la red virtual del clúster a una red virtual existente](tutorial-create-cluster.md#optional-connect-the-clusters-virtual-network-to-an-existing-virtual-network) para obtener más información.

## <a name="kubernetes-certification"></a>Certificación de Kubernetes

El servicio Red Hat OpenShift en Azure tiene el certificado CNCF de cumplimiento con Kubernetes.

## <a name="next-steps"></a>Pasos siguientes

Aprenda los requisitos previos de Red Hat OpenShift en Azure:

> [!div class="nextstepaction"]
> [Configuración de un entorno de desarrollo](howto-setup-environment.md)