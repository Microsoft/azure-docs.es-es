---
title: Solución de problemas de Azure Red Hat OpenShift | Microsoft Docs
description: Solucionar problemas y resolver problemas comunes con Azure Red Hat OpenShift
services: container-service
author: tylermsft
ms.author: twhitney
manager: jeconnoc
ms.service: container-service
ms.topic: troubleshooting
ms.date: 05/08/2019
ms.openlocfilehash: 9ef32273c007e7319b7f6df99f3d904f805c31f2
ms.sourcegitcommit: f013c433b18de2788bf09b98926c7136b15d36f1
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2019
ms.locfileid: "65550920"
---
# <a name="troubleshooting-for-azure-red-hat-openshift"></a>Solución de problemas para Azure Red Hat OpenShift

En este artículo se detalla algunos problemas comunes detectados durante la creación o administración de clústeres de Microsoft Azure Red Hat OpenShift.

## <a name="retrying-the-creation-of-a-failed-cluster"></a>Reintentando la creación de un clúster con error

Si la creación de una Azure Red Hat OpenShift clúster mediante el `az` produce un error de comando de CLI, volver a intentar la creación seguirá generando un error.
Use `az openshift delete` para eliminar el clúster con errores, a continuación, cree un clúster completamente nuevo.

## <a name="untrusted-azure-red-hat-openshift-server-certificate"></a>Certificado de servidor de confianza Azure Red Hat OpenShift

La consola de OpenShift utiliza un certificado autofirmado. Cuando se le solicite en el explorador, omita la advertencia y acepte el certificado que "no es de confianza".

## <a name="hidden-azure-red-hat-openshift-cluster-resource-group"></a>Grupo de recursos de clúster de Azure Red Hat OpenShift oculto

Actualmente, el `Microsoft.ContainerService/openShiftManagedClusters` recursos que se crearon automáticamente mediante la CLI de Azure (`az openshift create` comandos) está oculta en el portal de Azure. En el **grupo de recursos** vista, verificación **mostrar tipos ocultos** para ver el grupo de recursos.

![Captura de pantalla de la casilla de verificación de tipo oculto en el portal](./media/aro-portal-hidden-type.png)

## <a name="creating-a-cluster-results-in-error-that-no-registered-resource-provider-found"></a>Crear un clúster de los resultados de error que se encontró ningún proveedor de recursos registrado

Si crear un clúster da lugar a un error que `No registered resource provider found for location '<location>' and API version '2019-04-30' for type 'openShiftManagedClusters'. The supported api-versions are '2018-09-30-preview`, entonces formaban parte de la vista previa y ahora necesito [instancias reservadas de máquina virtual de Azure de la compra](https://aka.ms/openshift/buy) usando el producto disponible con carácter general. Una reserva reduce el gasto previamente pagando por servicios de Azure totalmente administrados. Consulte [ *¿cuáles son las reservas de Azure* ](https://docs.microsoft.com/azure/billing/billing-save-compute-costs-reservations) para obtener más información sobre las reservas y cómo pueden ahorrar dinero.

## <a name="next-steps"></a>Pasos siguientes

- Pruebe el [centro de Ayuda de Red Hat OpenShift](https://help.openshift.com/) para la solución de OpenShift en más.

- Encuentre respuestas a [preguntas más frecuentes sobre Azure Red Hat OpenShift](openshift-faq.md).