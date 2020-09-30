---
title: Solución de problemas de Red Hat OpenShift en Azure
description: Solución de problemas y resolución de incidencias comunes con Red Hat OpenShift en Azure
author: jimzim
ms.author: jzim
ms.service: container-service
ms.topic: troubleshooting
ms.date: 05/08/2019
ms.openlocfilehash: 55360ef295ff80b700b059d053203458f9f384db
ms.sourcegitcommit: 4feb198becb7a6ff9e6b42be9185e07539022f17
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89469089"
---
# <a name="troubleshooting-for-azure-red-hat-openshift"></a>Solución de problemas de Red Hat OpenShift en Azure

En este artículo se detallan algunas incidencias comunes detectadas durante la creación o administración de clústeres de Red Hat OpenShift en Microsoft Azure.

## <a name="retrying-the-creation-of-a-failed-cluster"></a>Reintento de la creación de un clúster con errores

Si la creación de un clúster de Red Hat OpenShift en Azure con el comando de la CLI `az` produce un error, volver a intentar la creación seguirá generando un error.
Use `az openshift delete` para eliminar el clúster con errores y, a continuación, cree un clúster completamente nuevo.

## <a name="hidden-azure-red-hat-openshift-cluster-resource-group"></a>Grupo de recursos oculto del clúster de Red Hat OpenShift en Azure

Actualmente, el recurso `Microsoft.ContainerService/openShiftManagedClusters` que se creó automáticamente con la CLI de Azure (comando `az openshift create`) está oculto en Azure Portal. En la vista **Grupo de recursos**, puede activar **Mostrar tipos ocultos** para ver el grupo de recursos.

![Captura de pantalla de la casilla de verificación de tipos ocultos en el portal](./media/aro-portal-hidden-type.png)

## <a name="creating-a-cluster-results-in-error-that-no-registered-resource-provider-found"></a>La creación de un clúster produce un error que indica que no se encontró ningún proveedor de recursos registrado

Si la creación de un clúster genera el error `No registered resource provider found for location '<location>' and API version '2019-04-30' for type 'openShiftManagedClusters'. The supported api-versions are '2018-09-30-preview`, significa que formaba parte de la versión preliminar y ahora debe [adquirir instancias reservadas de máquina virtual de Azure](https://aka.ms/openshift/buy) para usar el producto disponible con carácter general. Una reserva reduce el gasto al pagar por adelantado por servicios de Azure totalmente administrados. Consulte [ *¿Qué son las reservas de Azure*](../cost-management-billing/reservations/save-compute-costs-reservations.md) para más información sobre las reservas y cómo pueden ahorrar dinero.

## <a name="next-steps"></a>Pasos siguientes

- Pruebe el [Centro de ayuda de Red Hat OpenShift](https://help.openshift.com/) para más soluciones de problemas de OpenShift.

- Obtenga respuestas a algunas [preguntas frecuentes sobre Red Hat OpenShift en Azure](openshift-faq.md).