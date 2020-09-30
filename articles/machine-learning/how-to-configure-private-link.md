---
title: Configuración de un punto de conexión privado (versión preliminar)
titleSuffix: Azure Machine Learning
description: Use Azure Private Link para acceder de forma segura al área de trabajo de Azure Machine Learning desde una red virtual.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to
ms.author: aashishb
author: aashishb
ms.reviewer: larryfr
ms.date: 09/03/2020
ms.openlocfilehash: 83927c9df9a4f1a6ab32c15c481898ec68f53c4c
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90898149"
---
# <a name="configure-azure-private-link-for-an-azure-machine-learning-workspace-preview"></a>Configuración de Azure Private Link para un área de trabajo de Azure Machine Learning (versión preliminar)

En este documento, aprenderá a usar Azure Private Link con el área de trabajo de Azure Machine Learning. Para más información sobre la configuración de una red virtual para Azure Machine Learning, consulte [Información general sobre la privacidad y el aislamiento de la red virtual](how-to-network-security-overview.md).

> [!IMPORTANT]
> El uso de Azure Private Link con áreas de trabajo de Azure Machine Learning se encuentra actualmente en versión preliminar pública. Esta funcionalidad solo está disponible en las regiones **Este de EE. UU.** , **Centro-sur de EE. UU.** y **Oeste de EE. UU. 2**. Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no es aconsejable usarla para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Azure Private Link le permite conectarse a su área de trabajo mediante un punto de conexión privado. El punto de conexión privado es un conjunto de direcciones IP privadas dentro de la red virtual. Después, puede limitar el acceso al área de trabajo para que solo se produzca en las direcciones IP privadas. Private Link ayuda a reducir el riesgo de una filtración de datos. Para más información sobre los puntos de conexión privados, consulte el artículo [Azure Private Link](/azure/private-link/private-link-overview).

> [!IMPORTANT]
> Azure Private Link no afecta al plano de control de Azure (operaciones de administración), como la eliminación del área de trabajo o la administración de los recursos de proceso. Por ejemplo, crear, actualizar o eliminar un destino de proceso. Estas operaciones se realizan sobre la red pública de Internet como de costumbre. Las operaciones del plano de datos, como el uso de Azure Machine Learning Studio, las API (incluidas las canalizaciones publicadas) o el SDK usan el punto de conexión privado.
>
> Si usa Mozilla Firefox, puede encontrar problemas al intentar acceder al punto de conexión privado del área de trabajo. Este problema puede estar relacionado con DNS a través de HTTPS en Mozilla. Como solución alternativa, se recomienda usar Microsoft Edge de Google Chrome.

> [!TIP]
> La instancia de proceso de Azure Machine Learning se puede usar con un área de trabajo y un punto de conexión privado. Esta funcionalidad está actualmente en versión preliminar pública en las regiones **Este de EE. UU.** , **Centro-sur de EE. UU.** y **Oeste de EE. UU. 2**.

## <a name="create-a-workspace-that-uses-a-private-endpoint"></a>Creación de un área de trabajo que usa un punto de conexión privado

> [!IMPORTANT]
> Actualmente, solo se admite la habilitación de un punto de conexión privado al crear una nueva área de trabajo de Azure Machine Learning.

Puede usar la plantilla que se encuentra en [https://github.com/Azure/azure-quickstart-templates/tree/master/201-machine-learning-advanced](https://github.com/Azure/azure-quickstart-templates/tree/master/201-machine-learning-advanced) para crear un área de trabajo con un punto de conexión privado.

Para información sobre el uso de esta plantilla, que incluye los puntos de conexión privados, consulte [Uso de una plantilla de Azure Resource Manager para crear un área de trabajo para Azure Machine Learning](how-to-create-workspace-template.md).

## <a name="using-a-workspace-over-a-private-endpoint"></a>Uso de un área de trabajo mediante un punto de conexión privado

Dado que la comunicación con el área de trabajo solo se permite desde la red virtual, cualquier entorno de desarrollo que use el área de trabajo debe ser miembro de la red virtual. Por ejemplo, una máquina virtual en la red virtual.

> [!IMPORTANT]
> Para evitar la interrupción temporal de la conectividad, Microsoft recomienda vaciar la memoria caché de DNS en las máquinas que se conectan al área de trabajo después de habilitar Private Link. 

Para más información sobre Azure Virtual Machines, consulte [Documentación sobre Virtual Machines](/azure/virtual-machines/).


## <a name="next-steps"></a>Pasos siguientes

Para más información sobre cómo proteger el área de trabajo de Azure Machine Learning, consulte el artículo [Información general sobre la privacidad y el aislamiento de la red virtual](how-to-network-security-overview.md).
