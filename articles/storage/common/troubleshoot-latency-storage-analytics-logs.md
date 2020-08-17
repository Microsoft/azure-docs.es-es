---
title: Solución de problemas de latencia mediante registros de Storage Analytics
description: Identifique y solucione problemas de latencia mediante registros de Azure Storage Analytics, y optimice la aplicación cliente.
author: v-miegge
ms.topic: troubleshooting
ms.author: kartup
manager: dcscontentpm
ms.date: 10/21/2019
ms.service: storage
ms.subservice: common
services: storage
tags: ''
ms.openlocfilehash: efae9cd2a73bf6df89007ac313ca6dfe6efe6ddd
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87075954"
---
# <a name="troubleshoot-latency-using-storage-analytics-logs"></a>Solución de problemas de latencia mediante registros de Storage Analytics

Poder diagnosticar y solucionar problemas es una habilidad clave a la hora de crear y mantener aplicaciones cliente con el servicio Azure Storage.

Las aplicaciones de Azure suelen ser de naturaleza dispersa, de modo que diagnosticar y solucionar errores y problemas de rendimiento puede resultar más complicado que hacerlo en entornos tradicionales.

Los siguientes pasos muestran cómo identificar y solucionar problemas de latencia mediante registros de Azure Storage Analytics, y optimizar la aplicación cliente.

## <a name="recommended-steps"></a>Pasos recomendados

1. Descargue los [registros de Storage Analytics](https://docs.microsoft.com/azure/storage/common/storage-analytics-logging#download-storage-logging-log-data).

2. Use el siguiente script de PowerShell para convertir los registros de formato sin procesar en formato tabular:

   ```Powershell
   $Columns = 
        (   "version-number",
            "request-start-time",
            "operation-type",
            "request-status",
            "http-status-code",
            "end-to-end-latency-in-ms",
            "server-latency-in-ms",
            "authentication-type",
            "requester-account-name",
            "owner-account-name",
            "service-type",
            "request-url",
            "requested-object-key",
            "request-id-header",
            "operation-count",
            "requester-ip-address",
            "request-version-header",
            "request-header-size",
            "request-packet-size",
            "response-header-size",
            "response-packet-size",
            "request-content-length",
            "request-md5",
            "server-md5",
            "etag-identifier",
            "last-modified-time",
            "conditions-used",
            "user-agent-header",
            "referrer-header",
            "client-request-id"
        )

   $logs = Import-Csv “REPLACE THIS WITH FILE PATH” -Delimiter ";" -Header $Columns

   $logs | Out-GridView -Title "Storage Analytic Log Parser"
   ```

3. El script iniciará una ventana de interfaz gráfica de usuario donde puede filtrar la información por columnas, como se muestra a continuación.

   ![Ventana del analizador de registros de Storage Analytics](media/troubleshoot-latency-storage-analytics-logs/storage-analytic-log-parser-window.png)
 
4. Restrinja las entradas de registro basadas en "operation-type" y busque la entrada de registro creada durante el período de tiempo del problema.

   ![Entradas de registro de tipo de operación](media/troubleshoot-latency-storage-analytics-logs/operation-type.png)

5. En el momento en que se produjo el problema, son importantes los siguientes valores:

   * Tipo de operación = GetBlob
   * estado de la solicitud = SASNetworkError
   * Latencia de un extremo a otro en milisegundos = 8453
   * Latencia del servidor en milisegundos = 391

   La latencia de un extremo a otro se calcula mediante la siguiente ecuación:

   * Latencia de un extremo a otro = latencia del servidor + latencia de cliente

   Calcule la latencia de cliente mediante la entrada de registro:

   * Latencia de cliente = latencia de un extremo a otro – latencia del servidor

        Ejemplo: 8453 – 391 = 8062 ms

   En la siguiente tabla se proporciona información sobre los resultados de OperationType y RequestStatus de alta latencia:

   | Tipo de blob |RequestStatus=<br>Correcto|RequestStatus=<br>(SAS)NetworkError|Recomendación|
   |---|---|---|---|
   |GetBlob|Sí|No|[**Operación GetBlob:** RequestStatus = Correcto](#getblob-operation-requeststatus--success)|
   |GetBlob|No|Sí|[**Operación GetBlob:** RequestStatus = (SAS)NetworkError](#getblob-operation-requeststatus--sasnetworkerror)|
   |PutBlob|Sí|No|[**Operación Put:** RequestStatus = Correcto](#put-operation-requeststatus--success)|
   |PutBlob|No|Sí|[**Operación Put:** RequestStatus = (SAS)NetworkError](#put-operation-requeststatus--sasnetworkerror)|

## <a name="status-results"></a>Resultados de estado

### <a name="getblob-operation-requeststatus--success"></a>Operación GetBlob: RequestStatus = Correcto

Compruebe los siguientes valores tal como se mencionó en el paso 5 de la sección "Pasos recomendados":

* Latencia de un extremo a otro
* Latencia del servidor
* Latencia de cliente

En una **operación GetBlob** con **RequestStatus = Correcto**, si el **tiempo máximo** se pasa en la **latencia de cliente**, esto indica que Azure Storage pasa un gran volumen de tiempo escribiendo datos en el cliente. Este retraso indica un problema del lado cliente.

**Recomendación:**

* Investigue el código en su cliente.
* Utilice Wireshark, el Analizador de mensajes de Microsoft o Tcping para investigar los problemas de conectividad de red desde el cliente. 

### <a name="getblob-operation-requeststatus--sasnetworkerror"></a>Operación GetBlob: RequestStatus = (SAS)NetworkError

Compruebe los siguientes valores tal como se mencionó en el paso 5 de la sección "Pasos recomendados":

* Latencia de un extremo a otro
* Latencia del servidor
* Latencia de cliente

En una **operación GetBlob** con **RequestStatus = (SAS)NetworkError**, si el **tiempo máximo** se pasa en la **latencia de cliente**, el problema más habitual es que el cliente se desconecta antes de expirar el tiempo de espera en el servicio de almacenamiento.

**Recomendación:**

* Investigue el código en el cliente para comprender por qué y cuándo se desconecta el cliente del servicio de almacenamiento.
* Utilice Wireshark, el Analizador de mensajes de Microsoft o Tcping para investigar los problemas de conectividad de red desde el cliente. 

### <a name="put-operation-requeststatus--success"></a>Operación Put: RequestStatus = Correcto

Compruebe los siguientes valores tal como se mencionó en el paso 5 de la sección "Pasos recomendados":

* Latencia de un extremo a otro
* Latencia del servidor
* Latencia de cliente

En una **operación Put** con **RequestStatus = Correcto**, si el **tiempo máximo** se pasa en la **latencia de cliente**, esto indica que el cliente tarda más tiempo en enviar datos a Azure Storage. Este retraso indica un problema del lado cliente.

**Recomendación:**

* Investigue el código en su cliente.
* Utilice Wireshark, el Analizador de mensajes de Microsoft o Tcping para investigar los problemas de conectividad de red desde el cliente. 

### <a name="put-operation-requeststatus--sasnetworkerror"></a>Operación Put: RequestStatus = (SAS)NetworkError

Compruebe los siguientes valores tal como se mencionó en el paso 5 de la sección "Pasos recomendados":

* Latencia de un extremo a otro
* Latencia del servidor
* Latencia de cliente

En una **operación PutBlob** con **RequestStatus = (SAS)NetworkError**, si el **tiempo máximo** se pasa en la **latencia de cliente**, el problema más habitual es que el cliente se desconecta antes de expirar el tiempo de espera en el servicio de almacenamiento.

**Recomendación:**

* Investigue el código en el cliente para comprender por qué y cuándo se desconecta el cliente del servicio de almacenamiento.
* Utilice Wireshark, el Analizador de mensajes de Microsoft o Tcping para investigar los problemas de conectividad de red desde el cliente.

