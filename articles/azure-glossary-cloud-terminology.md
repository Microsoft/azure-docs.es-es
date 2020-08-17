---
title: Glosario de Azure - Diccionario de Azure | Microsoft Docs
description: Utilice el glosario de Azure para comprender la terminología de la nube sobre la plataforma Azure. Este breve diccionario de Azure define algunos términos comunes de la nube para Azure.
keywords: Diccionario de Azure, terminología de la nube, glosario de Azure, definiciones de terminología, términos de la nube
services: na
documentationcenter: na
author: MonicaRush
manager: jhubbard
editor: ''
ms.assetid: d7ac12f7-24b5-4bcd-9e4d-3d76fbd8d297
ms.service: multiple
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/16/2017
ms.author: monicar
ms.openlocfilehash: 8a06435d4b5a4c11f74073e67e3f48fd2ed0779d
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87079577"
---
# <a name="microsoft-azure-glossary-a-dictionary-of-cloud-terminology-on-the-azure-platform"></a>Glosario de Microsoft Azure: un diccionario de terminología de la nube sobre la plataforma de Azure

El glosario de Microsoft Azure es un breve diccionario de terminología de la nube para la plataforma Azure. Consulte también:

* [Microsoft Azure y Amazon Web Services](https://azure.microsoft.com/campaigns/azure-vs-aws/mapping/): definiciones de los servicios de Azure y sus equivalentes de AWS.<!-- I propose to link to https://azure.microsoft.com/services/ instead of this -->
* [Términos de informática en la nube](https://azure.microsoft.com/overview/cloud-computing-dictionary/): términos generales sobre informática en la nube.

## <a name="account"></a>account
Una cuenta que se usa para acceder y administrar una suscripción de Azure. A menudo se hace referencia a ella como una cuenta de Azure, aunque una cuenta puede ser cualquiera de estas: una cuenta de Microsoft profesional, educativa o personal existente, o un nombre de usuario y una contraseña de Office 365. También puede crear una cuenta para administrar una suscripción de Azure al registrarse para la [evaluación gratuita](https://azure.microsoft.com).  
Consulte [Regístrese para obtener una suscripción a Azure con su cuenta de Microsoft 365](cost-management-billing/manage/microsoft-365-account-for-azure-subscription.md) y [Asociación o incorporación de una suscripción de Azure al inquilino de Azure Active Directory](active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md).

## <a name="api-app"></a>Aplicación de API
Otro nombre para [aplicación de App Service](#app-service-app).

## <a name="app-service-app"></a>Aplicación de App Service
Los recursos de proceso que [Azure App Service](app-service/overview.md) proporciona para hospedar un sitio o aplicación web, API web o [back-end de aplicación móvil](app-service-mobile/app-service-mobile-value-prop.md). Las aplicaciones de App Service también se conocen como *servicios de aplicaciones*, *aplicaciones web*, *aplicaciones de API* y *aplicaciones móviles*.

## <a name="availability-set"></a>conjunto de disponibilidad
Una colección de máquinas virtuales que se administran de forma conjunta para proporcionar confiabilidad y redundancia a las aplicaciones. El uso de un conjunto de disponibilidad garantiza que durante un evento de mantenimiento planeado o no planeado, al menos una máquina virtual estará disponible.  
Consulte [Administración de la disponibilidad de las máquinas virtuales con Windows](virtual-machines/windows/manage-availability.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) y [Administración de la disponibilidad de las máquinas virtuales con Linux](virtual-machines/linux/manage-availability.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)

## <a name="azure-classic-deployment-model"></a><a name="classic-model"></a>modelo de implementación clásica de Azure
Uno de los dos [modelos de implementación](resource-manager-deployment-model.md) utilizados para implementar recursos en Azure (el nuevo modelo es Azure Resource Manager). Algunos servicios de Azure admiten solo el modelo de implementación Resource Manager, otros son compatibles con el modelo de implementación clásica y otros admiten ambos. La documentación para cada servicio de Azure especifica qué modelos admite.

## <a name="azure-command-line-interface-cli"></a><a name="cli"></a>Interfaz de la línea de comandos (CLI) de Azure
Una interfaz de la línea de comandos que se puede utilizar para administrar los servicios de Azure desde Windows, OSX y Linux.  Algunos servicios o características de un servicio pueden administrarse solo a través de PowerShell o la CLI. Consulte [CLI de Azure](/cli/azure)

## <a name="azure-powershell"></a><a name="powershell"></a>Azure PowerShell.
Una interfaz de la línea de comandos para administrar los servicios de Azure a través de una línea de comandos desde equipos con Windows. Algunos servicios o características de un servicio pueden administrarse solo a través de PowerShell o la CLI.
Consulte [Instalación y configuración de Azure PowerShell](/powershell/azure/)

## <a name="azure-resource-manager-deployment-model"></a><a name="arm-model"></a>Modelo de implementación de Azure Resource Manager
Uno de los dos [modelos de implementación](resource-manager-deployment-model.md) utilizados para implementar recursos en Microsoft Azure (el otro es el modelo de implementación clásica). Algunos servicios de Azure admiten solo el modelo de implementación Resource Manager, otros son compatibles con el modelo de implementación clásica y otros admiten ambos. La documentación para cada servicio de Azure especifica qué modelos admite.

## <a name="fault-domain"></a>dominio de error
La colección de máquinas virtuales de un conjunto de disponibilidad que posiblemente den error al mismo tiempo. Un ejemplo es un grupo de máquinas en bastidor que comparten una fuente de alimentación y un conmutador de red. En Azure, las máquinas virtuales de un conjunto de disponibilidad se separan automáticamente en varios dominios de error.  
Consulte [Administración de la disponibilidad de las máquinas virtuales con Windows](virtual-machines/windows/manage-availability.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) o [Administración de la disponibilidad de las máquinas virtuales con Linux](virtual-machines/linux/manage-availability.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)  

## <a name="geo"></a>geoárea
Un límite definido para la residencia de datos que normalmente contiene dos o más regiones. Los límites pueden situarse dentro o fuera de las fronteras nacionales y se ven afectados por la normativa fiscal. Cada geoárea tiene al menos una región. Ejemplos de geoáreas son Asia Pacífico y Japón. Este concepto está relacionado con la *geografía*.  
Vea [Regiones de Azure](best-practices-availability-paired-regions.md)

## <a name="geo-replication"></a>replicación geográfica
El proceso de replicación automática de contenido como blobs, tablas y colas entre zonas regionales emparejadas.  
Vea [Replicación geográfica activa para Azure SQL Database](sql-database/sql-database-geo-replication-overview.md)
<!-- The meaning of "geo" in this term seems to be different than the meaning provided in the "geo" entry -->

## <a name="image"></a>imagen
Un archivo que contiene la configuración del sistema operativo y la aplicación que puede utilizarse para crear cualquier cantidad de máquinas virtuales. En Azure existen dos tipos de imágenes: Imagen de máquina virtual e imagen del sistema operativo. Una imagen de máquina virtual incluye un sistema operativo y todos los discos conectados a una máquina virtual cuando se crea la imagen. Una imagen de sistema operativo contiene solo un sistema operativo generalizado sin configuraciones de disco de datos.  
Vea [Navegación y selección de las imágenes de máquina virtual Windows en Azure con Powershell o la CLI](virtual-machines/windows/cli-ps-findimage.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)

## <a name="limits"></a>límites
El número de recursos que se pueden crear o las pruebas comparativas de rendimiento que se pueden lograr. Los límites se suelen asociar a las suscripciones, los servicios y las ofertas.  
Vea [Límites, cuotas y restricciones de suscripción y servicios de Microsoft Azure](azure-resource-manager/management/azure-subscription-service-limits.md)

## <a name="load-balancer"></a>load balancer
Un recurso que distribuye el tráfico entrante entre equipos en una red. En Azure, un equilibrador de carga distribuye el tráfico a las máquinas virtuales definidas en un conjunto de equilibrador de carga. Un [equilibrador de carga](load-balancer/load-balancer-overview.md) puede ser accesible desde Internet o de uso interno.  

## <a name="mobile-app"></a>aplicación móvil
Otro nombre para [aplicación de App Service](#app-service-app).

## <a name="offer"></a>offer
Los precios, créditos y términos relacionados aplicables a una suscripción de Azure.  
Consulte la página [Detalles de las ofertas de Microsoft Azure](https://azure.microsoft.com/support/legal/offer-details/)

## <a name="portal"></a>portal
El portal web seguro que se usa para implementar y administrar servicios de Azure.

## <a name="region"></a>region
Un área dentro de una geoárea que no traspasa las fronteras nacionales y contiene uno o varios centros de datos. Los precios, los servicios regionales y los tipos de ofertas se exponen a nivel de región. Una región se empareja normalmente con otra, que puede estar a cientos de kilómetros de distancia. Las parejas regionales pueden utilizarse como un mecanismo para escenarios de alta disponibilidad y recuperación ante desastres. Además, este concepto está relacionado con la *ubicación*.  
Vea [Regiones de Azure](best-practices-availability-paired-regions.md)

## <a name="resource"></a>resource
Un elemento que forma parte de la solución de Azure. Cada servicio de Azure permite implementar diferentes tipos de recursos, como bases de datos o máquinas virtuales.   
Vea [Información general del Azure Resource Manager](azure-resource-manager/management/overview.md)

## <a name="resource-group"></a>resource group
Un contenedor en Resource Manager que incluye los recursos relacionados de una aplicación. El grupo de recursos puede incluir todos los recursos de una aplicación o solo aquellos que se agrupan juntos lógicamente. Puede decidir cómo desea asignar los recursos a los grupos de recursos en función de lo que más convenga a su organización.  
Vea [Información general del Azure Resource Manager](azure-resource-manager/management/overview.md)

## <a name="resource-manager-template"></a><a name="arm-template"></a>plantilla de Resource Manager
Un archivo JSON que define de forma declarativa uno o varios recursos de Azure y que define las dependencias entre los recursos implementados. La plantilla se puede usar para implementar los recursos de manera repetida y uniforme.  
Vea [Creación de plantillas de Azure Resource Manager](resource-group-authoring-templates.md)

## <a name="resource-provider"></a>proveedor de recursos
Un servicio que proporciona los recursos que puede implementar y administrar mediante Resource Manager. Cada proveedor de recursos ofrece operaciones para trabajar con los recursos que se implementan. Es posible acceder a los proveedores de recursos mediante el Portal de Azure, Azure PowerShell y varios SDK de programación.  
Vea [Información general del Azure Resource Manager](azure-resource-manager/management/overview.md)

## <a name="role"></a>rol
Un medio para controlar el acceso que se puede asignar a usuarios, grupos y servicios. Los roles pueden realizar acciones como crear, administrar y leer en recursos de Azure.  
Consulte [RBAC: roles integrados](role-based-access-control/built-in-roles.md)

## <a name="service-level-agreement-sla"></a><a name="sla"></a>contrato de nivel de servicio (SLA)
El contrato que describe los compromisos de Microsoft en cuanto a tiempo de actividad y conectividad. Cada servicio de Azure tiene un acuerdo de nivel de servicio específico.  
Vea [Contratos de nivel de servicio](https://azure.microsoft.com/support/legal/sla/)

## <a name="shared-access-signature-sas"></a><a name="sas"></a>firma de acceso compartido (SAS)
Firma que le permite otorgar acceso limitado a un recurso, sin exponer su clave de cuenta. Por ejemplo, [Azure Storage usa SAS](storage/common/storage-dotnet-shared-access-signature-part-1.md) para conceder acceso de cliente a objetos como blobs. [IoT Hub usa SAS](iot-hub/iot-hub-devguide-security.md#security-tokens) para conceder a los dispositivos permiso para enviar telemetría.

## <a name="storage-account"></a>Cuenta de almacenamiento
Cuenta que proporciona acceso a los servicios de Azure Blob, Queue, Table y File en Azure Storage. El nombre de la cuenta de almacenamiento define el espacio de nombres exclusivo para los objetos de datos de Azure Storage.  
Consulte [Acerca de las cuentas de Azure Storage](storage/common/storage-create-storage-account.md)

## <a name="subscription"></a>subscription
Contrato de un cliente con Microsoft que le permite obtener servicios de Azure. Los precios de la suscripción y los términos relacionados se rigen por la oferta elegida para la suscripción.
Consulte [Contrato Microsoft Online Subscription](https://azure.microsoft.com/support/legal/subscription-agreement/) y [Asociación de las suscripciones de Azure con Azure Active Directory](active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md)

## <a name="tag"></a>etiqueta
Un término de indexación que permite clasificar los recursos según sus requisitos de administración o facturación. Cuando se tiene un conjunto complejo de grupos de recursos, puede usar etiquetas para visualizar estos recursos de la manera más conveniente. Por ejemplo, puede etiquetar recursos que cumplen una función similar en la organización o que pertenecen al mismo departamento.  
Consulte [Uso de etiquetas para organizar los recursos de Azure](resource-group-using-tags.md)

## <a name="update-domain"></a>actualizar dominio
La colección de máquinas virtuales en un conjunto de disponibilidad que se actualizan al mismo tiempo. Las máquinas virtuales que se encuentran en el mismo dominio de actualización se reinician en conjunto durante el mantenimiento planeado. Azure no reinicia nunca más de un dominio de actualización a la vez. Así es como funcionan los dominios de actualización.  
Consulte [Administración de la disponibilidad de las máquinas virtuales con Windows](virtual-machines/windows/manage-availability.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) y [Administración de la disponibilidad de las máquinas virtuales con Linux](virtual-machines/linux/manage-availability.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)

## <a name="virtual-machine"></a><a name="vm"></a>máquina virtual
La implementación de software de un equipo físico que ejecuta un sistema operativo. Se pueden ejecutar varias máquinas virtuales a la vez en un mismo hardware. En Azure, hay máquinas virtuales disponibles en diferentes tamaños.  
Consulte [Documentación sobre máquinas virtuales](https://azure.microsoft.com/documentation/services/virtual-machines/)

## <a name="virtual-machine-extension"></a><a name="vm-extension"></a>extensión de máquina virtual
Un recurso que implementa comportamientos o características que cooperan en el funcionamiento de otros programas o le permiten interactuar con un equipo en ejecución. Por ejemplo, podría utilizar la extensión de acceso a máquinas virtuales para restablecer o modificar los valores de acceso remoto en una máquina virtual de Azure.
<!-- This definition seems obscure to me; maybe a list of examples would work better than a conceptual definition? -->
Consulte [Acerca de las características y extensiones de las máquinas virtuales (Windows)](virtual-machines/windows/extensions-features.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) o [Acerca de las características y extensiones de las máquinas virtuales (Linux)](virtual-machines/linux/extensions-features.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)

## <a name="virtual-network"></a><a name="vnet"></a>red virtual
Una red que proporciona conectividad entre los recursos de Azure que se encuentra aislada del resto de inquilinos de Azure. Una instancia de [Azure VPN Gateway](vpn-gateway/vpn-gateway-about-vpngateways.md) le permite establecer conexiones entre redes virtuales y entre una red virtual y una red local. Puede controlar por completo los bloques de direcciones IP, la configuración DNS, las directivas de seguridad y las tablas de rutas dentro de una red virtual.  
Consulte [Información general sobre redes virtuales](virtual-network/virtual-networks-overview.md)  

## <a name="web-app"></a>Aplicación web
Otro nombre para [aplicación de App Service](#app-service-app).

## <a name="see-also"></a>Consulte también

* [Comience a usar Azure](https://azure.microsoft.com/get-started/)
* [Centro de recursos en la nube](https://azure.microsoft.com/resources/)  
* [Azure para aplicaciones empresariales](https://azure.microsoft.com/overview/business-apps-on-azure/)
* [Azure en su centro de datos](https://azure.microsoft.com/overview/business-apps-on-azure/)

