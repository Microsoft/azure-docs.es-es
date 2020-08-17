---
title: Notas de la versión de Azure SQL Edge
description: Notas de la versión en las que se detallan las novedades y los cambios en las imágenes de Azure SQL Edge
keywords: notas de la versión SQL Server
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
ms.subservice: ''
author: VasiyaKrishnan
ms.author: vakrishn
ms.reviewer: sstein
ms.date: 07/27/2020
ms.openlocfilehash: 74e9772ada010d79e81ef36cae89ba586db73077
ms.sourcegitcommit: 1b2d1755b2bf85f97b27e8fbec2ffc2fcd345120
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87553417"
---
# <a name="azure-sql-edge-release-notes"></a>Notas de la versión de Azure SQL Edge 

En este artículo se describen las novedades y los cambios en cada nueva compilación de Azure SQL Edge.

## <a name="ctp-22"></a>CTP 2.2
### <a name="sql-engine-build-number---15020001546"></a>Número de compilación del motor de SQL: 15.0.2000.1546
### <a name="fixes"></a>Correcciones
1. Compatibilidad con contenedores que no son raíz 
2. Compatibilidad con la recopilación de datos de uso y diagnóstico 
3. Actualizaciones de streaming de T-SQL
   - Compatibilidad con los caracteres Unicode para nombres de objetos de secuencia
   - Mejoras en la limpieza del proceso
   - Mejoras en el registro y los diagnósticos
4. Mejora del rendimiento de la ingesta de datos

## <a name="ctp-21"></a>CTP 2.1 
### <a name="sql-engine-build-number---15020001545"></a>Número de compilación del motor de SQL: 15.0.2000.1545
### <a name="fixes"></a>Correcciones
1. Corrección de PREDICT con modelos de ONNX para administrar el problema de CPUID en ARM 
2. Corrección para mejorar el control de la ruta de acceso de error en el inicio de la transmisión de TSQL 
3. Corrección del valor incorrecto del retraso de la marca de agua en las métricas de trabajo cuando no hay datos 
4. Corrección del problema con el adaptador de salida cuando el adaptador tiene un esquema de variables entre lotes  

## <a name="ctp-20"></a>CTP 2.0 
### <a name="sql-engine-build-number---15020001401"></a>Número de compilación del motor de SQL: 15.0.2000.1401
### <a name="whats-new"></a>Novedades
1.  Actualización del nombre de producto a "Azure SQL Edge"
1.  Función Date_bucket

    i.  Compatibilidad con el tipo Date, Time, DateTime
3.  PREDICT con ONNX
    
    i.  Parámetro RUNTIME necesario para ONNX 
    
4.  Compatibilidad con streaming de TSQL (versión preliminar limitada) 
 
### <a name="known-issues"></a>Problemas conocidos

1. <b>Problema:</b> posibles errores al aplicar dacpac en el inicio debido a un problema de temporización.

    <b>Solución alternativa:</b> con el reinicio de SQL Server o el contenedor se intentará de nuevo la aplicación de dacpac, con lo que el problema debería solucionarse.
### <a name="request-support"></a>Solicitud de soporte técnico
1. Puede solicitar soporte técnico en la [página de soporte técnico](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

4. Asegúrese de que están seleccionados los campos siguientes: 
    * Tipo de problema: Técnico 
    * Servicio: IoT Edge
    * Tipo de problema: Mi problema está relacionado con un módulo de IoT Edge
    * Subtipo de problema: Azure SQL Edge

   ![Incidencia de soporte técnico de ejemplo](media/get-support/support-ticket.png)

## <a name="ctp-15"></a>CTP 1.5
### <a name="sql-engine-build-number---15020001331"></a>Número de compilación del motor de SQL: 15.0.2000.1331
### <a name="whats-new"></a>Novedades
1. Función Date_bucket
    
    i. Compatibilidad con el tipo DateTimeOffset
2. Modelos de PREDICT con ONNX

    i. Compatibilidad con nvarchar
 
## <a name="ctp-14"></a>CTP 1.4
### <a name="sql-engine-build-number---15020001247"></a>Número de compilación del motor de SQL: 15.0.2000.1247
### <a name="whats-new"></a>Novedades
1.  Modelos de PREDICT con ONNX
 
    i.  Compatibilidad con Varchar
    
    ii. Migración a la versión en tiempo de ejecución de ONNX 1.0 
2.  Compatibilidad con características: Están habilitadas las siguientes características:

    i.   Compatibilidad con CDC

    ii.  Tabla de historial con compresión

    iii. Mayor factor de escala para la lectura anticipada del registro

    iv.  Modo por lotes frente a aplicación de filtro

    v.   Optimizaciones de lectura anticipada
 
## <a name="ctp-13"></a>(CTP 1.3)
### <a name="sql-engine-build-number---15020001147"></a>Número de compilación del motor de SQL: 15.0.2000.1147
### <a name="whats-new"></a>Novedades
1. Implementación del portal de Azure IoT 

    i.   Compatibilidad con la implementación de imágenes de AMD64 y ARM

    ii.  Compatibilidad con la creación de trabajos de streaming

    iii. Implementación de dacpac
2. Modelos de PREDICT con ONNX

    i. Compatibilidad con tipos numéricos
3. Compatibilidad con características: están habilitadas las siguientes características:

    i.  Agregado de aplicación a examen de almacén de columnas

    ii. Recorridos cícliclos
4. Trabajo de reducción del consumo de memoria y superficie
