---
title: Introducción a Azure Arc
description: Obtenga información sobre qué es Azure Arc y cómo ayuda a los clientes a habilitar la administración y el gobierno de sus recursos híbridos con otros servicios y características de Azure.
ms.date: 08/25/2020
ms.topic: overview
ms.openlocfilehash: 46bd2089dbc2794d0fb98ceb9a5e97011e36d6ea
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89052049"
---
# <a name="azure-arc-overview"></a>Introducción a Azure Arc

En la actualidad, las empresas luchan por controlar y gobernar un entorno que cada vez se vuelve más complejo. Estos entornos se extienden a través de los centros de datos, varias nubes y el borde. Cada entorno y nube tienen su propio conjunto de herramientas de administración separadas que tiene que aprender y operar.

En paralelo, los nuevos modelos operativos DevOps y ITOps son difíciles de implementar, ya que las herramientas existentes no proporcionan compatibilidad con los nuevos patrones nativos en la nube.

Azure Arc simplifica el gobierno y la administración al ofrecer una plataforma de administración local y multinube coherente. Azure Arc le permite administrar todo el entorno, con un solo panel de cristal, proyectando los recursos existentes en Azure Resource Manager. Ahora puede administrar las máquinas virtuales, los clústeres de Kubernetes y las bases de datos como si se ejecutaran en Azure. Independientemente de dónde se encuentren, puede usar los servicios y funcionalidades de administración de Azure que conozca. Azure Arc le permite seguir usando ITOps tradicionales, al tiempo que presenta prácticas de DevOps para admitir en el entorno nuevos patrones nativos en la nube.

:::image type="content" source="./media/overview/azure-arc-control-plane.png" alt-text="Diagrama del plano de control de la administración de Azure Arc" border="false":::

En la actualidad, Azure Arc le permite administrar los siguientes tipos de recursos hospedados fuera de Azure:

* Servidores: máquinas físicas y virtuales que ejecutan Windows o Linux.
* Clústeres de Kubernetes: admiten varias distribuciones de Kubernetes.
* Servicios de datos de Azure: servicios de hiperescala de Azure SQL Database y PostgreSQL.

## <a name="what-does-azure-arc-deliver"></a>¿Qué ofrece Azure Arc?

Entre las principales características de Azure Arc se incluyen:

* Implementación de un inventario, administración, gobierno y seguridad coherentes para los servidores en todo el entorno.

* Configuración de [extensiones de máquina virtual de Azure](./servers/manage-vm-extensions.md) para usar los servicios de administración de Azure y supervisar, proteger y actualizar los servidores.

* Administración y control de los clústeres de Kubernetes a escala. 

* Uso de la configuración basada en GitOps como administración de código para implementar las aplicaciones y las configuraciones en uno o varios clústeres directamente desde el control de origen, como GitHub.

* Cumplimiento y configuración automáticos para los clústeres de Kubernetes mediante Azure Policy.

* Ejecución de los servicios de datos de Azure en cualquier entorno de Kubernetes, en concreto en Azure SQL Managed Instance e Hiperescala para Azure Database for PostgreSQL, con ventajas como las actualizaciones, la seguridad y la supervisión como si se ejecutaran en Azure. Aprovechamiento del escalado elástico, aplicación de las actualizaciones, sin tiempo de inactividad de la aplicación, incluso si no existe una conexión continua con Azure.

* Una experiencia unificada para ver los recursos habilitados para Azure Arc, tanto si usa Azure Portal, la CLI de Azure, Azure PowerShell o la API REST de Azure.

## <a name="how-much-does-azure-arc-cost"></a>¿Cuánto cuesta Azure Arc?

A continuación se muestran los detalles de precios de las características disponibles hoy en día con Azure Arc.

### <a name="arc-enabled-servers"></a>Servidores habilitados para Arc

En la fase de versión preliminar actual, los servidores habilitados para Azure Arc se ofrecen sin costo adicional.

Cualquier servicio de Azure que se use en los servidores habilitados para Arc, como Azure Security Center o Azure Monitor, se cobrará según los precios de ese servicio. Para más información, consulte la [página de precios](https://azure.microsoft.com/pricing/).

### <a name="azure-arc-enabled-kubernetes"></a>Kubernetes habilitado para Azure Arc

En la fase de versión preliminar actual, Kubernetes habilitado para Azure Arc se ofrece sin costo adicional.

### <a name="azure-arc-enabled-data-services"></a>Servicios de datos habilitados para Azure Arc

En la fase de versión preliminar actual, los servicios de datos habilitados para Azure Arc se ofrecen sin costo adicional.

## <a name="next-steps"></a>Pasos siguientes

* Para más información acerca de los servidores habilitados para Arc, consulte la siguiente [información general](./servers/overview.md)

* Para más información acerca de Kubernetes habilitado para Arc, consulte la siguiente [información general](./kubernetes/overview.md)

* Para más información acerca de los servicios de datos habilitados para Arc, consulte la siguiente [información general](https://azure.microsoft.com/services/azure-arc/hybrid-data-services/)
