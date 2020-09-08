---
title: 'Visualización de registros de flujo de grupo de seguridad de red de Azure: Power BI'
titleSuffix: Azure Network Watcher
description: Obtenga información sobre cómo usar Power BI para visualizar los registros de flujo de grupos de seguridad de red para que pueda ver información sobre el tráfico IP en Azure Network Watcher.
services: network-watcher
documentationcenter: na
author: damendo
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2017
ms.author: damendo
ms.openlocfilehash: a4c97b8baaf5265f984622e2dc47e6c8c714172a
ms.sourcegitcommit: 2bab7c1cd1792ec389a488c6190e4d90f8ca503b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/17/2020
ms.locfileid: "88272644"
---
# <a name="visualizing-network-security-group-flow-logs-with-power-bi"></a>Visualización de registros de flujo del grupo de seguridad de red con Power BI

Los registros de flujo del grupo de seguridad de red le permiten visualizar información sobre el tráfico IP de entrada y salida en los grupos de seguridad de red. Estos registros de flujo muestran los flujos de entrada y salida en función de cada regla, la NIC a la que se aplica el flujo, información de 5-tupla sobre el flujo (IP de origen y de destino, puerto de origen y de destino, protocolo), y si se permitió o denegó el tráfico.

Puede ser difícil obtener información sobre los datos de registro de flujo mediante la búsqueda manual de los archivos de registro. En este artículo se proporcionan una solución para visualizar los registros de flujo más recientes y obtener información sobre el tráfico en la red.

> [!Warning]  
> En los pasos siguientes se trabajará con la versión 1 de los registros de flujo. Para obtener más detalles, vea [Introducción al registro de flujo de grupos de seguridad de red](network-watcher-nsg-flow-logging-overview.md). Las siguientes instrucciones no funcionarán con la versión 2 de los archivos de registro, sin ninguna modificación.

## <a name="scenario"></a>Escenario

En el siguiente escenario, conectamos Power BI Desktop a la cuenta de almacenamiento que hemos configurado como el receptor de los datos del registro de flujo de NSG. Una vez que se conecta a la cuenta de almacenamiento, Power BI descarga y analiza los registros para proporcionar una representación visual del tráfico que registran los grupos de seguridad de red.

Utilizando los objetos visuales proporcionados en la plantilla puede examinar:

* Top Talkers
* Datos de flujo de serie temporal por decisión de regla y dirección
* Flujos por dirección MAC de la interfaz de red
* Flujos por NSG y regla
* Flujos por puerto de destino

La plantilla que se proporciona es editable, por lo que puede modificarla agregando nuevos datos u objetos visuales, o modificando consultas para adaptarla a sus necesidades.

## <a name="setup"></a>Configurar

Antes de empezar, tiene que tener el registro de flujo de grupo de seguridad de red habilitado en uno o más grupos de seguridad de red de su cuenta. Para ver instrucciones para habilitar los registros de flujo de grupo de seguridad de red, consulte el artículo siguiente: [Introduction to flow logging for Network Security Groups](network-watcher-nsg-flow-logging-overview.md) (Introducción al registro de flujo para grupos de seguridad de red).

También tiene que tener el cliente de Power BI Desktop instalado en su equipo y suficiente espacio disponible en el mismo para descargar y cargar los datos del registro que existen en la cuenta de almacenamiento.

![Diagrama de Visio][1]

### <a name="steps"></a>Pasos

1. Descargue y abra la siguiente plantilla de Power BI en la aplicación de Power BI Desktop [plantilla de registros de flujo de Power BI de Network Watcher](https://aka.ms/networkwatcherpowerbiflowlogstemplate)
1. Escriba los parámetros de consulta necesarios
   1. **StorageAccountName**: especifica el nombre de la cuenta de almacenamiento que contiene los registros de flujo de NSG que desea cargar y visualizar.
   1. **NumberOfLogFiles**: especifica el número de archivos de registro que desear descargar y visualizar en Power BI. Por ejemplo, si se especifica 50, los 50 archivos de registro más recientes. Si tenemos 2 NSG habilitados y configurados para enviar registros de flujo de NSG a esta cuenta, se pueden ver las últimas 25 horas de registros.

      ![Ventana principal de Power BI][2]

1. Escriba la clave de acceso para la cuenta de almacenamiento. Puede encontrar claves de acceso válidas navegando a la cuenta de almacenamiento en Azure Portal y seleccionando **Claves de acceso** en el menú de configuración. Haga clic en **Conectar** y aplique los cambios.

    ![Claves de acceso][3]

    ![Clave de acceso 2][4]

4. Los registros se descargan y analizan, y ahora ya puede usar los objetos visuales creados previamente.

## <a name="understanding-the-visuals"></a>Descripción de los objetos visuales

En la plantilla se proporcionan un conjunto de objetos visuales que lo ayudan a comprender los datos del registro de flujo de NSG. Las siguientes imágenes muestran un ejemplo del aspecto del panel cuando se rellena con datos. A continuación se examina con más detalle cada objeto visual 

![Power BI][5]
 
El objeto visual Top Talkers muestra las direcciones IP que han iniciado la mayoría de las conexiones durante el período especificado. El tamaño de los cuadros corresponde al número relativo de conexiones. 

![Toptalkers][6]

Los siguientes gráficos de series temporales muestran el número de flujos en el período. El gráfico superior está segmentado por la dirección del flujo y el inferior está segmentado por la decisión MADD (permitir o denegar). Con este objeto visual, puede examinar las tendencias del tráfico a través del tiempo y detectar cualquier pico o disminución anómala la segmentación del tráfico.

![flowsoverperiod][7]

Los gráficos siguientes muestran los flujos por interfaz de red, con el superior segmentado por dirección del flujo y el inferior segmentado por decisión realizada. Con esta información, puede obtener detalles sobre cuáles máquinas virtuales se comunican más en relación con otras, y si el tráfico a una máquina virtual se está permitiendo o denegando.

![flowspernic][8]

El siguiente gráfico de anillo muestra un desglose de los flujos por puerto de destino. Con esta información, puede ver los puertos de destino más utilizados dentro del período especificado.

![anillo][9]

El siguiente gráfico de barras muestra el flujo por NSG y regla. Con esta información, puede ver los NSG responsables de la mayor parte del tráfico y el desglose del tráfico en un NSG por regla.

![gráfico de barras][10]
 
Los siguientes gráficos informativos muestran información sobre los NSG presentes en los registros, el número de flujos capturados durante el período y la fecha del registro más antiguo capturado. Esta información le ofrece una idea de cuáles NSG se están registrando y el intervalo de fechas de los flujos.

![infochart1][11]

![infochart2][12]

Esta plantilla incluye las segmentaciones siguientes para que pueda ver solo los datos en los que está más interesado. Puede filtrar por grupos de recursos, NSG y reglas. También puede filtrar por información de 5-tupla, decisión y la hora cuando se escribió el registro.

![segmentaciones][13]

## <a name="conclusion"></a>Conclusión

Hemos mostrado en este escenario que utilizando los registros de flujo de grupo de seguridad de red proporcionados por Network Watcher y Power BI, se puede visualizar y comprender el tráfico. Utilizando la plantilla proporcionada, Power BI descarga los registros directamente desde el almacenamiento y los procesa localmente. El tiempo necesario para cargar la plantilla varía dependiendo del número de archivos solicitados y del tamaño total de los archivos descargados.

Puede personalizar esta plantilla para que se adapte a sus necesidades. Hay muchas formas en las que puede utilizar Power BI con los registros de flujo de grupo de seguridad de red. 

## <a name="notes"></a>Notas

* De forma predeterminada los registros se almacenan en `https://{storageAccountName}.blob.core.windows.net/insights-logs-networksecuritygroupflowevent/`

    * Si hay otros datos en otro directorio, las consultas para la extracción y procesamiento de los datos tienen que modificarse.

* La plantilla proporcionada no se recomienda para su uso con más de 1 GB de registros.

* Si tiene una gran cantidad de registros, recomendamos que investigue una solución utilizando otro almacén de datos como Data Lake o SQL Server.

## <a name="next-steps"></a>Pasos siguientes

Aprenda a visualizar los registros de flujo del grupo de seguridad de red con la pila Elastic visitando [Visualización de registros de flujo de grupo de seguridad de red de Azure Network Watcher con herramientas de código abierto](network-watcher-visualize-nsg-flow-logs-open-source-tools.md).

[1]: ./media/network-watcher-visualize-nsg-flow-logs-power-bi/figure1.png
[2]: ./media/network-watcher-visualize-nsg-flow-logs-power-bi/figure2.png
[3]: ./media/network-watcher-visualize-nsg-flow-logs-power-bi/figure3.png
[4]: ./media/network-watcher-visualize-nsg-flow-logs-power-bi/figure4.png
[5]: ./media/network-watcher-visualize-nsg-flow-logs-power-bi/figure5.png
[6]: ./media/network-watcher-visualize-nsg-flow-logs-power-bi/figure6.png
[7]: ./media/network-watcher-visualize-nsg-flow-logs-power-bi/figure7.png
[8]: ./media/network-watcher-visualize-nsg-flow-logs-power-bi/figure8.png
[9]: ./media/network-watcher-visualize-nsg-flow-logs-power-bi/figure9.png
[10]: ./media/network-watcher-visualize-nsg-flow-logs-power-bi/figure10.png
[11]: ./media/network-watcher-visualize-nsg-flow-logs-power-bi/figure11.png
[12]: ./media/network-watcher-visualize-nsg-flow-logs-power-bi/figure12.png
[13]: ./media/network-watcher-visualize-nsg-flow-logs-power-bi/figure13.png
