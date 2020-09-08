---
title: Vínculos adicionales sobre Avere vFXT para Azure
description: Use estos recursos para obtener más información sobre Avere vFXT for Azure, incluida la documentación de clúster de Avere y la documentación de administración de vFXT.
author: ekpgh
ms.service: avere-vfxt
ms.topic: conceptual
ms.date: 01/13/2020
ms.author: rohogue
ms.openlocfilehash: 6f9a9f8e6d2a279871ded3dff614be99dd2901df
ms.sourcegitcommit: 2bab7c1cd1792ec389a488c6190e4d90f8ca503b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/17/2020
ms.locfileid: "88271097"
---
# <a name="additional-documentation"></a>Documentación adicional

En este artículo encontrará vínculos que le permitirán obtener documentación adicional sobre la interfaz de administración del panel de control de Avere y otros temas relacionados.

## <a name="avere-cluster-documentation"></a>Documentación del clúster de Avere

Puede encontrar documentación adicional sobre el clúster de Avere en el sitio web de <https://azure.github.io/Avere/>. Gracias a estos documentos podrá comprender las capacidades del clúster y sabrá cómo configurar sus ajustes.

* La [guía de creación de clústeres de FXT](<https://azure.github.io/Avere/#fxt_cluster>) está diseñada para crear clústeres compuestos por nodos de hardware físicos, pero también podrá encontrar cierta información que también es relevante para los clústeres de vFXT. En particular, los nuevos administradores de clústeres de vFXT pueden obtener información valiosa si leen estas secciones:
  * En [Customizing Support and Monitoring Settings](<https://azure.github.io/Avere/legacy/create_cluster/4_8/html/config_support.html#config-support>) (Personalización de la configuración de soporte y supervisión) se explica cómo personalizar la configuración de la carga de soporte técnico y habilitar la supervisión remota.
  * En [Configuring VServers and Global Namespace](<https://azure.github.io/Avere/legacy/create_cluster/4_8/html/config_vserver.html#config-vserver>) (Configuración de VServers y el espacio de nombres global) encontrará información sobre cómo crear un espacio de nombres orientado al cliente.
  * En [Configuring DNS for the Avere cluster](<https://azure.github.io/Avere/legacy/create_cluster/4_8/html/config_network.html#dns-overview>) (Configuración DNS para el clúster de Avere) se explica cómo configurar el DNS de round robin.
  * En [Adición de almacenamiento de back-end](<https://azure.github.io/Avere/legacy/create_cluster/4_8/html/config_core_filer.html#add-core-filer>) se documenta el modo de agregar archivadores centrales.

* En [Cluster Configuration Guide](<https://azure.github.io/Avere/#operations>) (Guía de configuración del clúster) encontrará una referencia completa acerca de la configuración y las opciones de un clúster de Avere. Un clúster de vFXT usa un subconjunto de estas opciones, pero se puede aplicar la mayoría de páginas de esta configuración.

* En [Dashboard Guide](<https://azure.github.io/Avere/#operations>) (Guía del panel) se explica cómo usar las características de supervisión de clústeres del panel de control de Avere.

## <a name="vfxt-creation-and-management-documentation"></a>Documentación de creación y administración de vFXT

En GitHub se ofrece una guía completa del uso de vfxt.py, la creación de clústeres en la nube basados en script y la utilidad de administración: [Cloud Cluster Management with vfxt.py](https://github.com/Azure/AvereSDK/blob/master/docs/README.md) (Administración de clústeres en la nube con vfxt.py).
