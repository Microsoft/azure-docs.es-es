---
title: Uso de Azure Portal para solucionar problemas de Azure Stack Edge | Microsoft Docs
description: Aprenda a solucionar problemas relacionados con Azure Stack Edge. Puede ejecutar diagnósticos, recopilar información de soporte técnico y usar registros para solucionar problemas.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: troubleshooting
ms.date: 08/11/2020
ms.author: alkohli
ms.openlocfilehash: f9dabdfd9df77822f06c41fa4355a1657bceb76c
ms.sourcegitcommit: 1aef4235aec3fd326ded18df7fdb750883809ae8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/12/2020
ms.locfileid: "88134571"
---
# <a name="troubleshoot-your-azure-stack-edge-issues"></a>Solución de problemas de Azure Stack Edge

En este artículo se describe cómo solucionar los problemas de Azure Stack Edge. 

En este artículo aprenderá a:

> [!div class="checklist"]
>
> * Ejecución de diagnósticos
> * Recopilación de un paquete de soporte
> * Uso de registros para solucionar problemas

## <a name="run-diagnostics"></a>Ejecución de diagnósticos

Para diagnosticar y solucionar los errores de cualquier dispositivo, puede ejecutar las pruebas de diagnóstico. Realice los pasos siguientes en la interfaz de usuario web local del dispositivo para ejecutar las pruebas de diagnóstico.

1. En la interfaz de usuario de web local, vaya a **Solución de problemas > Pruebas de diagnóstico en**. Seleccione la prueba que quiere ejecutar y elija **Ejecutar prueba**. Así se ejecutan las pruebas necesarias para diagnosticar posibles problemas en la red, el dispositivo, el proxy web, la hora o la configuración de la nube. Aparece una notificación que indica que hay algunas pruebas en ejecución en el dispositivo.

    ![Seleccionar pruebas](media/azure-stack-edge-troubleshoot/run-diag-1.png)

2. Al terminar las pruebas, se muestran los resultados.

    ![Revisar los resultados de la prueba](media/azure-stack-edge-troubleshoot/run-diag-2.png)

    Si se produce un error en una prueba, se especifica una dirección URL en la que se muestra la acción recomendada. Seleccione dicha dirección para ver la acción recomendada.

    ![Revisión de las advertencias de las pruebas con errores](media/azure-stack-edge-troubleshoot/run-diag-3.png)

## <a name="collect-support-package"></a>Recopilación de un paquete de soporte

Los paquetes de registros contienen todos los registros pertinentes que pueden ayudar al equipo de soporte técnico de Microsoft a solucionar los problemas de cualquier dispositivo. Los paquetes de registros se pueden generar a través de la interfaz de usuario web local.

Siga estos pasos para recopilar un paquete de soporte.

1. En la interfaz de usuario web local, vaya a **Solución de problemas > Soporte**. Seleccione **Crear un paquete de soporte técnico**. El sistema empieza a recopilar el paquete de soporte. Dicha recopilación puede tardar varios minutos.

    ![Hacer clic en agregar usuario](media/azure-stack-edge-troubleshoot/collect-logs-1.png)

2. Una vez que se cree el paquete, haga clic en **Descargar el paquete de soporte técnico**. Se descarga un paquete comprimido en la ruta de acceso que eligió. Puede descomprimir el paquete de registro y ver los archivos de registro del sistema.

    ![Hacer clic en agregar usuario](media/azure-stack-edge-troubleshoot/collect-logs-2.png)

## <a name="use-logs-to-troubleshoot"></a>Uso de registros para solucionar problemas

Los errores encontrados durante los procesos de carga y actualización se incluyen en los archivos de error respectivos.

1. Para ver los archivos de error, vaya al recurso compartido y haga clic en el recurso compartido para ver el contenido. 

      ![Conexión y visualización del contenido del recurso compartido](media/azure-stack-edge-troubleshoot/troubleshoot-logs-1.png)

2. Seleccione la _carpeta de Microsoft Azure Stack Edge_. Esta carpeta tiene dos subcarpetas:

    * Cargue la que contiene los archivos de registro de errores de carga.
    * Actualice la carpeta para ver los errores durante la actualización.

    Este es un archivo de registro de ejemplo de actualización.

    ```xml
    <root container="test1" machine="VM15BS020663" timestamp="03/18/2019 00:11:10" />
    <file item="test.txt" local="False" remote="True" error="16001" />
    <summary runtime="00:00:00.0945320" errors="1" creates="2" deletes="0" insync="3" replaces="0" pending="9" />
    ```

3. Cuando vea un error en este archivo (resaltado en el ejemplo), anote el código de error, en este caso el 16001. Busque la descripción del código de error en la siguiente referencia de errores.

    [!INCLUDE [data-box-edge-edge-upload-error-reference](../../includes/data-box-edge-gateway-upload-error-reference.md)]

## <a name="next-steps"></a>Pasos siguientes

* Más información acerca de los [problemas conocidos de esta versión](data-box-gateway-release-notes.md).
