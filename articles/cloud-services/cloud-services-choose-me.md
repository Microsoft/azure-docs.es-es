---
title: Qué es Azure Cloud Services | Microsoft Docs
description: Obtenga información sobre qué es Azure Cloud Services, en concreto su diseño para admitir aplicaciones escalables, confiables y económicas para operar.
services: cloud-services
author: tgore03
ms.service: multiple
ms.topic: article
ms.date: 04/19/2017
ms.author: tagore
ms.openlocfilehash: 0013a3a29bae9d2dde7896b3ae23d0d358946f2b
ms.sourcegitcommit: 152c522bb5ad64e5c020b466b239cdac040b9377
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88224295"
---
# <a name="overview-of-azure-cloud-services"></a>Información general de Azure Cloud Services
Azure Cloud Services es un ejemplo de [plataforma como servicio](https://azure.microsoft.com/overview/what-is-paas/) (PaaS). Al igual que [Azure App Service](../app-service/overview.md), esta tecnología está diseñada para ser compatible con aplicaciones escalables, confiables y de funcionamiento asequible. En la misma manera que App Service se hospeda en máquinas virtuales (VM), así ocurre también con Azure Cloud Services. Sin embargo, tiene más control sobre las máquinas virtuales. Puede instalar su propio software en las máquinas virtuales que usan Azure Cloud Services y puede tener acceso a ellas remotamente.

![Diagrama de Azure Cloud Services](./media/cloud-services-choose-me/diagram.png)

Más control también significa menos facilidad de uso. A menos que necesite opciones de control adicionales, por lo general es más rápido y fácil poner en marcha una aplicación web en la característica Web Apps de App Service que en Azure Cloud Services.

Hay dos tipos de roles de Azure Cloud Services. La única diferencia entre ambos es cómo se hospeda el rol en las máquinas virtuales:

* **Rol Web**: implementa y hospeda automáticamente la aplicación a través de IIS.

* **Rol de trabajo**: no usa IIS y ejecuta la aplicación independiente.

Por ejemplo, una aplicación simple podría utilizar solo un rol web, para dar servicio a un sitio web. Una aplicación más compleja podría utilizar un rol web para tratar las solicitudes entrantes provenientes de los usuarios y luego transmitirlas a un rol de trabajo para su procesamiento. (Esta comunicación podría usar [Azure Service Bus](../service-bus-messaging/service-bus-messaging-overview.md) o [Azure Queue Storage](../storage/common/storage-introduction.md)).

Como sugiere la ilustración anterior, todas las VM de una sola aplicación se ejecutan en el mismo servicio en la nube. Los usuarios acceden a la aplicación a través de una sola dirección IP pública y el equilibrio de carga de las solicitudes se realiza automáticamente en todas las VM de la aplicación. La plataforma [escala e implementa](cloud-services-how-to-scale-portal.md) las máquinas virtuales en una aplicación de Azure Cloud Services para así evitar un único punto de errores de hardware.

A pesar de que las aplicaciones se ejecutan en máquinas virtuales, resulta importante comprender que Azure Cloud Services proporciona PaaS y no una infraestructura como servicio (IaaS). Esta es una manera de considerarlo. Con IaaS, como Azure Virtual Machines, primero crea y configura el entorno en el que la aplicación se ejecuta. A continuación, implementa la aplicación en este entorno. Es responsabilidad suya administrar gran parte de él, por ejemplo, debe implementar las nuevas versiones revisadas del sistema operativo en cada máquina virtual. Por el contrario, en PaaS ocurre como si el entorno ya existiese. Todo lo que tiene que hacer es implementar la aplicación. Usted maneja la administración de la plataforma en que se ejecuta, incluida la implementación de versiones nuevas del sistema operativo.

## <a name="scaling-and-management"></a>Escalado y administración
Con Azure Cloud Services no se crean máquinas virtuales. En cambio, se proporciona un archivo de configuración que le indica a Azure qué cantidad de cada una le gustaría tener, por ejemplo, tres instancias de rol web y dos instancias de rol de trabajo. La plataforma se crea entonces automáticamente. Sigue siendo necesario que elija [qué tamaño](cloud-services-sizes-specs.md) deben tener esas máquinas virtuales (las opciones son las mismas que con las máquinas virtuales de Azure), pero no tiene que crearlas por sí mismo explícitamente. Si la aplicación necesita manejar una carga mayor, puede pedir más VM y Azure creará esas instancias. Si la carga disminuye, puede apagar esas instancias y así dejar de pagar por ellas.

Una aplicación de Azure Cloud Services normalmente está a disposición de los usuarios a través de un proceso de dos pasos. Un desarrollador primero [carga la aplicación](cloud-services-how-to-create-deploy-portal.md) en el área de ensayo de la plataforma. Cuando el desarrollador está preparado para activar definitivamente la aplicación, utiliza Azure Portal para intercambiar el ensayo por la producción. Este [intercambio entre el área de ensayo y la producción](cloud-services-how-to-manage-portal.md#swap-deployments-to-promote-a-staged-deployment-to-production) se puede llevar a cabo sin tiempo de inactividad, lo que permite actualizar una aplicación en ejecución a una versión nueva sin interrumpir ni molestar a sus usuarios.

## <a name="monitoring"></a>Supervisión
Azure Cloud Services también proporciona supervisión. Al igual que con Azure Virtual Machines, detecta un servidor físico con errores y reinicia las máquinas virtuales que se ejecutaban en ese servidor en una máquina nueva. Pero Azure Cloud Services también detecta las máquinas virtuales y aplicaciones con errores, no solo los errores de hardware. A diferencia de Máquinas virtuales, Servicios en la nube tiene un agente dentro de cada rol web o rol de trabajo, por lo que puede iniciar nuevas instancias de aplicaciones y máquinas virtuales cuando se produce un error.

La naturaleza PaaS de Azure Cloud Services tiene también otras implicaciones. Una de las más importantes es que las aplicaciones creadas sobre la base de esta tecnología deben estar escritas para ejecutarse correctamente en caso de cualquier error en instancia de rol web o de trabajo. Para lograrlo, una aplicación de Azure Cloud Services no debería mantener el estado en el sistema de archivos de sus propias máquinas virtuales. A diferencia de lo que ocurre en las máquinas virtuales creadas con Azure Virtual Machines, las escrituras realizadas en máquinas virtuales de Azure Cloud Services no son persistentes. No es como un disco de datos de Azure Virtual Machines. En su lugar, una aplicación de Azure Cloud Services debe escribir explícitamente todo el estado en Azure SQL Database, blobs, tablas u algún otro tipo de almacenamiento externo. Cuando se crean aplicaciones de esta manera, estas resultan más fáciles de escalar y son más resistentes ante los errores, y ambos son objetivos importantes de Azure Cloud Services.

## <a name="next-steps"></a>Pasos siguientes
* [Crear una aplicación de servicio en la nube en .NET](cloud-services-dotnet-get-started.md) 
* [Crear una aplicación de servicio en la nube en Node.js](cloud-services-nodejs-develop-deploy-app.md) 
* [Crear una aplicación de servicio en la nube en PHP](../cloud-services-php-create-web-role.md) 
* [Creación de una aplicación de servicio en la nube en Python](cloud-services-python-ptvs.md)






