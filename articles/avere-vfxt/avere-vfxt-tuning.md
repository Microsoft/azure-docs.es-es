---
title: Ajuste del clúster de Avere vFXT - Azure
description: Obtenga información sobre la optimización personalizada de los clústeres de vFXT en Avere vFXT for Azure que se puede hacer, trabajando con un representante de soporte técnico.
author: ekpgh
ms.service: avere-vfxt
ms.topic: how-to
ms.date: 12/19/2019
ms.author: rohogue
ms.openlocfilehash: 5d9f81c9438cb992f81bd3e6319532d67db75552
ms.sourcegitcommit: 2bab7c1cd1792ec389a488c6190e4d90f8ca503b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/17/2020
ms.locfileid: "88272389"
---
# <a name="cluster-tuning"></a>Ajuste del clúster

La mayoría de los clústeres de vFXT pueden beneficiarse de la configuración de rendimiento personalizada. Estas opciones de configuración permiten que el clúster funcione mejor con un flujo de trabajo, un conjunto de datos y unas herramientas determinados.

Esta opción de personalización debe realizarse con la ayuda de representante de soporte técnico, ya que pude ser necesario configurar características que no estén disponibles en el panel de control de Avere.

En esta sección se describen algunas de las opciones de ajuste personalizada que pueden realizarse.

## <a name="general-optimizations"></a>Optimizaciones generales

Estos cambios son recomendables según las cualidades del conjunto de datos o el estilo del flujo de trabajo.

* Si la carga de trabajo es pesada en cuanto a la escritura, aumente el tamaño de la memoria caché de escritura a partir de su valor predeterminado del 20 %.
* Si el conjunto de datos tiene muchos archivos pequeños, aumente el límite de recuento de archivos de la memoria caché del clúster.
* Si el trabajo implica copiar o mover datos entre dos repositorios, ajuste el número de subprocesos que se usan para mover datos:
  * Para aumentar la velocidad, puede aumentar el número de subprocesos paralelos que se usan.
  * Si se está sobrecargando el volumen de almacenamiento de back-end, debe reducir el número de subprocesos paralelos que use.
* Si el clúster se almacena en la caché de datos de un archivador principal que usa las ACL de NFSv4, habilite el almacenamiento en caché del modo de acceso para agilizar la autorización de archivos para determinados clientes.

## <a name="cloud-nas-or-cloud-gateway-optimizations"></a>Optimizaciones en la nube de NAS o en la puerta de enlace en la nube

En un escenario de puerta de enlace o NAS de nube, el clúster de vFXT proporciona acceso al estilo NAS a un contenedor en la nube. Para aprovechar las velocidades de datos más altas entre el clúster de vFXT y el almacenamiento en la nube, su representante puede recomendarle cambiar opciones de configuración como éstas para insertar datos de forma más agresiva en volumen de almacenamiento desde la memoria caché. Por ejemplo:

* Aumentar el número de conexiones TCP entre el clúster y el contenedor de almacenamiento.

## <a name="cloud-bursting-or-hybrid-wan-optimizations"></a>Seguridad en la nube u optimizaciones WAN híbridas

En un escenario de seguridad en la nube o en uno dedicado a la optimización WAN del almacenamiento híbrido, el clúster de vFXT proporciona integración entre la nube y el almacenamiento de hardware local. Estos cambios pueden ser útiles:

* Aumentar el número de conexiones TCP entre el clúster y el archivador principal.
* Habilitar la configuración de optimización WAN para el archivador principal remoto (esta configuración puede usarse para un archivador remoto local o para un archivador principal en la nube que se encuentre en otra región de Azure).
* Aumentar el tamaño del búfer de sockets TCP<sup>*</sup>
* Habilitar la configuración "Reenviar siempre" para reducir los archivos que están en la memoria caché de forma redundante<sup>*</sup>

<sup>*</sup>Es posible que estos ajustes no se apliquen a todos los sistemas, en función de las necesidades de rendimiento y carga de trabajo.

## <a name="help-optimizing-your-avere-vfxt-for-azure"></a>Ayuda para optimizar Avere vFXT para Azure

Para ponerse en contacto con el personal de soporte técnico acerca de estas optimizaciones, siga el procedimiento descrito en [Get help with your system](avere-vfxt-open-ticket.md) (Obtener ayuda con el sistema).
