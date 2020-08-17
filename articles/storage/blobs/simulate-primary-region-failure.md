---
title: 'Tutorial: Simulación de un error al leer datos de la región principal'
titleSuffix: Azure Storage
description: Simule un error de lectura de datos de la región primaria cuando la cuenta de almacenamiento se haya configurado para el almacenamiento con redundancia geográfica con acceso de lectura (RA-GZRS).
services: storage
author: tamram
ms.service: storage
ms.subservice: blobs
ms.topic: tutorial
ms.date: 04/16/2020
ms.author: tamram
ms.reviewer: artek
ms.custom: devx-track-javascript
ms.openlocfilehash: 59eb826d8d17ed2adb3027f0aab11a6933b47ae5
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87905744"
---
# <a name="tutorial-simulate-a-failure-in-reading-data-from-the-primary-region"></a>Tutorial: Simulación de un error al leer datos de la región principal

Este tutorial es la segunda parte de una serie. En él aprenderá sobre las ventajas del [almacenamiento con redundancia geográfica con acceso de lectura](../common/storage-redundancy.md) (RA-GZRS) mediante la simulación de un error.

Para simular un error, puede usar el [enrutamiento estático](#simulate-a-failure-with-an-invalid-static-route) o [Fiddler](#simulate-a-failure-with-fiddler). Ambos métodos le permiten simular un error para las solicitudes enviadas al punto de conexión principal de la cuenta de almacenamiento con [redundancia geográfica con acceso de lectura](../common/storage-redundancy.md) (RA-GZRS), lo que motivará que la aplicación efectúe la lectura desde el punto de conexión secundario.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

En la segunda parte de la serie, se aprende a:

> [!div class="checklist"]
> * Ejecutar y pausar la aplicación
> * Simulación de un error con [una ruta estática no válida](#simulate-a-failure-with-an-invalid-static-route) o [Fiddler](#simulate-a-failure-with-fiddler)
> * Simular la restauración del punto de conexión principal

## <a name="prerequisites"></a>Prerrequisitos

Antes de comenzar este tutorial, complete la parte primera, [Logre la alta disponibilidad de los datos de la aplicación con Azure Storage][previous-tutorial].

Para simular un error con enrutamiento estático, usará un símbolo del sistema con privilegios elevados.

Para simular un error mediante Fiddler, descargue e [instale Fiddler](https://www.telerik.com/download/fiddler).

## <a name="simulate-a-failure-with-an-invalid-static-route"></a>Simulación de un error con una ruta estática no válida

Puede crear una ruta estática no válida para todas las solicitudes enviadas al punto de conexión principal de la cuenta de almacenamiento con [redundancia geográfica con acceso de lectura](../common/storage-redundancy.md) (RA-GZRS). En este tutorial, se utiliza el host local como puerta de enlace para enrutar las solicitudes a la cuenta de almacenamiento. El uso del host local como puerta de enlace hace que todas las solicitudes al punto de conexión principal de la cuenta de almacenamiento creen un bucle dentro del host que provoca errores. Siga los pasos a continuación para simular un error y la restauración del punto de conexión principal con una ruta estática no válida.

### <a name="start-and-pause-the-application"></a>Inicio y pausa de la aplicación

Siga las instrucciones del [tutorial anterior][previous-tutorial] para iniciar el ejemplo y descargar el archivo de prueba, pero debe confirmar que procede del almacenamiento principal. En función de la plataforma de destino, puede pausar el ejemplo manualmente o esperar a un símbolo del sistema.

### <a name="simulate-failure"></a>Simulación de error

Mientras la aplicación está en pausa, abra un símbolo del sistema en Windows como administrador o ejecute el terminal como raíz en Linux.

Para obtener información sobre el dominio del punto de conexión principal de la cuenta de almacenamiento, escriba el siguiente comando en un símbolo del sistema o en el terminal y reemplace `STORAGEACCOUNTNAME` por el nombre de la cuenta de almacenamiento.

```
nslookup STORAGEACCOUNTNAME.blob.core.windows.net
```

Copie la dirección IP de la cuenta de almacenamiento en un editor de texto para usarla después.

Para obtener la dirección IP del host local, escriba `ipconfig` en el símbolo del sistema de Windows o `ifconfig` en el terminal de Linux.

Para agregar una ruta estática para un host de destino, escriba el siguiente comando en un símbolo del sistema de Windows o en un terminal de Linux, reemplazando `<destination_ip>` con la dirección IP de la cuenta de almacenamiento y `<gateway_ip>` con la dirección IP del host local.

#### <a name="linux"></a>Linux

```
route add <destination_ip> gw <gateway_ip>
```

#### <a name="windows"></a>Windows

```
route add <destination_ip> <gateway_ip>
```

En la ventana con el ejemplo en ejecución, reanude la aplicación o presione la tecla correspondiente para descargar el archivo de ejemplo y confirme que procede de un almacenamiento secundario. A continuación, puede pausar el ejemplo de nuevo o esperar al símbolo del sistema.

### <a name="simulate-primary-endpoint-restoration"></a>Simular la restauración del punto de conexión principal

Para simular que el punto de conexión principal vuelve a funcionar, elimine la ruta estática no válida de la tabla de enrutamiento. Esto permite que todas las solicitudes al punto de conexión principal se enruten mediante la puerta de enlace predeterminada. Escriba el siguiente comando en un símbolo del sistema de Windows o en un terminal de Linux.

#### <a name="linux"></a>Linux

```bash
route del <destination_ip> gw <gateway_ip>
```

#### <a name="windows"></a>Windows

```console
route delete <destination_ip>
```

A continuación, puede reanudar la aplicación o presionar la tecla correspondiente para descargar el archivo de ejemplo de nuevo; en esta ocasión, confirme que también procede de un almacenamiento principal.

## <a name="simulate-a-failure-with-fiddler"></a>Simule un error con Fiddler

Para simular un error con Fiddler, inserte una respuesta con error para las solicitudes enviadas al punto de conexión principal de la cuenta de almacenamiento con redundancia geográfica con acceso de lectura.

En las secciones siguientes se muestra cómo simular un error y la restauración del punto de conexión principal con Fiddler.

### <a name="launch-fiddler"></a>Inicio de Fiddler

Abra Fiddler, seleccione **Rules** (Reglas) y **Customize Rules** (Personalizar reglas).

![Personalización de reglas de Fiddler](media/simulate-primary-region-failure/figure1.png)

Se inicia Fiddler ScriptEditor y se muestra el archivo **SampleRules.js**. Este archivo se utiliza para personalizar Fiddler.

Pegue el siguiente ejemplo de código en la función `OnBeforeResponse`, reemplazando `STORAGEACCOUNTNAME` por el nombre de la cuenta de almacenamiento. Según el ejemplo, puede que también necesite reemplazar `HelloWorld` con el nombre del archivo de prueba que se va a descargar, o bien un prefijo como `sampleFile`. El nuevo código se convierte en comentario para garantizar que no se ejecuta inmediatamente.

Cuando haya terminado, seleccione **File** (Archivo) y **Save** (Guardar) para guardar los cambios. Deje abierta la ventana ScriptEditor para usarla en los pasos siguientes.

```javascript
    /*
        // Simulate data center failure
        // After it is successfully downloading the blob, pause the code in the sample,
        // uncomment these lines of script, and save the script.
        // It will intercept the (probably successful) responses and send back a 503 error.
        // When you're ready to stop sending back errors, comment these lines of script out again
        //     and save the changes.

        if ((oSession.hostname == "STORAGEACCOUNTNAME.blob.core.windows.net")
            && (oSession.PathAndQuery.Contains("HelloWorld"))) {
            oSession.responseCode = 503;
        }
    */
```

![Pegar la regla personalizada](media/simulate-primary-region-failure/figure2.png)

### <a name="start-and-pause-the-application"></a>Inicio y pausa de la aplicación

Siga las instrucciones del [tutorial anterior][previous-tutorial] para iniciar el ejemplo y descargar el archivo de prueba, pero debe confirmar que procede del almacenamiento principal. En función de la plataforma de destino, puede pausar el ejemplo manualmente o esperar a un símbolo del sistema.

### <a name="simulate-failure"></a>Simulación de error

Mientras la aplicación está en pausa, cambie a Fiddler y quite la marca de comentario de la regla personalizada guardada en la función `OnBeforeResponse`. No olvide seleccionar **Archivo** y **Guardar** para guardar los cambios, a fin de que la regla se aplique. Este código busca las solicitudes enviadas a la cuenta de almacenamiento con redundancia geográfica con acceso de lectura y, si la ruta de acceso contiene el nombre del archivo de ejemplo, devuelve un código de respuesta de `503 - Service Unavailable`.

En la ventana con el ejemplo en ejecución, reanude la aplicación o presione la tecla correspondiente para descargar el archivo de ejemplo y confirme que procede de un almacenamiento secundario. A continuación, puede pausar el ejemplo de nuevo o esperar al símbolo del sistema.

### <a name="simulate-primary-endpoint-restoration"></a>Simular la restauración del punto de conexión principal

En Fiddler, quite o convierta en comentario la regla personalizada de nuevo. Seleccione **Archivo** y **Guardar** para asegurarse de que la regla ya no estará en vigor.

En la ventana con el ejemplo en ejecución, reanude la aplicación o presione la tecla correspondiente para descargar el archivo de ejemplo y confirme de nuevo que procede de un almacenamiento principal. A continuación, puede salir del ejemplo.

## <a name="next-steps"></a>Pasos siguientes

En la segunda parte de la serie, aprendió a simular un error para probar el almacenamiento con redundancia geográfica con acceso de lectura.

Lea el artículo siguiente para obtener más información acerca del funcionamiento de RA-GZRS y sobre los riesgos asociados:

> [!div class="nextstepaction"]
> [Designación de aplicaciones de alta disponibilidad con RA-GZRS](../common/geo-redundant-design.md)

[previous-tutorial]: storage-create-geo-redundant-storage.md
