---
title: Implementación de OpenShift Container Platform 4.x en Azure
description: Implementación de OpenShift Container Platform 4.x en Azure.
author: haroldwongms
manager: mdotson
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: how-to
ms.workload: infrastructure
ms.date: 10/14/2019
ms.author: haroldw
ms.openlocfilehash: f2fb1657c26fce3e1fdd67f36b0c6511b78dd216
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87373545"
---
# <a name="deploy-openshift-container-platform-4x-in-azure"></a>Implementación de OpenShift Container Platform 4.x en Azure

Ahora se admite la implementación de OpenShift Container Platform (OCP) 4.2 en Azure a través del modelo de infraestructura aprovisionada por el instalador (IPI).  La página de aterrizaje para probar OpenShift 4 es [try.openshift.com](https://try.openshift.com/). Para instalar OCP 4.2 en Azure, visite la página de [Red Hat OpenShift Cluster Manager](https://cloud.redhat.com/openshift/install/azure/installer-provisioned).  Se necesitan credenciales de Red Hat para acceder a este sitio.


## <a name="notes"></a>Notas 

 - Se necesita una entidad de servicio (SP) de Azure Active Directory (AAD) para instalar y ejecutar OCP 4.x en Azure.
     - Se debe conceder a la entidad de servicio el permiso de API **Application.ReadWrite.OwnedBy** para Azure Active Directory Graph.
     - Un Administrador de inquilinos de AAD debe conceder consentimiento de administrador para que este permiso de API surta efecto.
     - Se debe conceder a la entidad de servicio los roles **Colaborador** y **Administrador de acceso de usuario** para la suscripción.
 - El modelo de instalación de OCP 4.x es diferente de 3.x y no hay ninguna plantilla Azure Resource Manager disponible para implementar OCP 4.x en Azure
 - Si se producen problemas durante el proceso de instalación, póngase en contacto con la compañía adecuada (Microsoft o Red Hat).

| Descripción del problema | Punto de contacto |
|-------------------|---------------|
| Problemas específicos de Azure (AAD, SP, suscripción de Azure, etc.)                              | Microsoft |
| Problemas específicos de OpenShift (problemas de instalación/errores, suscripción de Red Hat, etc.) |  Red Hat  |




## <a name="next-steps"></a>Pasos siguientes

- [Introducción a OpenShift Container Platform](https://docs.openshift.com)
