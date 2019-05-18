---
title: 'Microsoft Genomics: Preguntas frecuentes: preguntas más frecuentes | Microsoft Docs'
titleSuffix: Azure
description: Aquí encontrará respuestas a las preguntas que hicieron los clientes sobre Microsoft Genomics.
services: genomics
author: grhuynh
manager: cgronlun
ms.author: grhuynh
ms.service: genomics
ms.topic: article
ms.date: 12/07/2017
ms.openlocfilehash: 2bcbf9d145d9b8b5a3c42893235906d24516405c
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65792633"
---
# <a name="microsoft-genomics-common-questions"></a>Microsoft Genomics: Preguntas frecuentes

En este artículo se enumeran las principales consultas que pueden surgir en relación con Microsoft Genomics. Para más información acerca del servicio de Microsoft Genomics, consulte [¿Qué es Microsoft Genomics?](overview-what-is-genomics.md). Para más información acerca de cómo solucionar problemas, consulte el documento [Troubleshooting guide](troubleshooting-guide-genomics.md) (Guía para la solución de problemas). 

## <a name="what-is-the-microsoft-genomics-service-gatk-4-promotion"></a>¿Qué es la promoción de GATK 4 del servicio de Microsoft Genomics?
Hasta el 30 de junio de 2019, el servicio Microsoft Genomics ofrece 20 ejecuciones WGS con GATK4 sin costo alguno. Para participar en esta oferta, regístrese [aquí](https://aka.ms/msgatk4). 

### <a name="what-are-the-common-issues-i-might-encounter-while-running-the-microsoft-genomics-service-gatk4-promotion"></a>¿Cuáles son los problemas comunes que podrían producirse durante la ejecución de la promoción de GATK4 del servicio de Microsoft Genomics?
A continuación, se incluye una lista de errores comunes que pueden surgir y su resolución recomendada:

| **Mensaje**                                                                                                                                                                                    | **Causa**                                                                                                    | **Resolución**                                                                                                                                                                                                       |
|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `gatk4-promo` no está habilitado para la cuenta. Para obtener más información, vea https://docs.microsoft.com/azure/genomics/frequently-asked-questions-genomics.                               | Está intentando ejecutar flujos de trabajo de GATK4 sin que el servicio de Microsoft Genomics esté activado.       | Vaya [aquí](https://aka.ms/msgatk4) para activar la cuenta. Tenga en cuenta que la versión de prueba caduca el 30 de junio de 2019. Tras esta fecha, no podrá activar la cuenta para las ejecuciones promocionales. |
| Gracias por probar `gatk4-promo`. Ha finalizado el período de prueba. Para obtener más información, https://docs.microsoft.com/azure/genomics/frequently-asked-questions-genomics.                  | El período de prueba de GATK4 ha expirado a finales del año natural y esta intentando invocar el process_name `gatk4-promo`.  | Cambie el parámetro process_name por `gatk4`, en lugar de `gatk4-promo`. Esta es la versión oficial de GATK4 y, si usa este parámetro, se facturará el flujo de trabajo.                                         |
| Gracias por probar `gatk4-promo`. Ha utilizado todas las ejecuciones asignadas. Para obtener más información, vea https://docs.microsoft.com/azure/genomics/frequently-asked-questions-genomics. | Ha enviado correctamente las 20 ejecuciones promocionales para GATK4.                               | Envíe cualquier ejecución nueva de GATK4 con el argumento process_name establecido en `gatk4`, en lugar de `gatk4-promo`. El flujo de trabajo se facturará cuando use este parámetro.                                                          |        


## <a name="can-i-run-gatk4-workflows-on-microsoft-genomics-without-signing-up-for-the-gatk4-promotion"></a>¿Puedo ejecutar los flujos de trabajo de GATK4 en Microsoft Genomics sin registrarme en la promoción de GATK4?
Sí, en el archivo config.txt del servicio de Microsoft Genomics, especifique el argumento process_name en `gatk4`. Tenga en cuenta que se le cobrará según las tarifas normales de facturación, así como que las 20 ejecuciones gratuitas no se aplicarán a la cuenta de Microsoft Genomics.



## <a name="what-is-the-sla-for-microsoft-genomics"></a>¿Qué es el SLA de Microsoft Genomics?
Le garantizamos que el 99,9 % de las veces el servicio de Microsoft Genomics estará disponible para recibir las solicitudes de API del flujo de trabajo. Para más información, consulte [Acuerdo de Nivel de Servicio](https://azure.microsoft.com/support/legal/sla/genomics/v1_0/).

## <a name="how-does-the-usage-of-microsoft-genomics-show-up-on-my-bill"></a>¿Cómo se refleja el uso de Microsoft Genomics en la factura?
Microsoft Genomics factura según el número de gigabases procesadas por flujo de trabajo. Para obtener más información, consulte el apartado [Precios](https://azure.microsoft.com/pricing/details/genomics/).


## <a name="where-can-i-find-a-list-of-all-possible-commands-and-arguments-for-the-msgen-client"></a>¿Dónde puedo encontrar una lista de todos los comandos y argumentos posibles para el cliente `msgen`?
Puede obtener una lista completa de comandos y argumentos disponibles ejecutando `msgen help`. Si no se proporciona ningún argumento más, se muestra una lista con las secciones de ayuda disponibles; una para cada uno de los comandos `submit`, `list`, `cancel` y `status`. Para obtener ayuda sobre un comando específico, escriba `msgen help command`; por ejemplo, `msgen help submit` enumera todas las opciones de envío.

## <a name="what-are-the-most-commonly-used-commands-for-the-msgen-client"></a>¿Cuáles son los comandos qué más usa el cliente `msgen`?
Los comandos más usados son argumentos del cliente `msgen` e incluyen: 

 |**Comando**          |  **Descripción del campo** |
 |:--------------------|:-------------         |
 |`list`               |Devuelve una lista con los trabajos que se han enviado. Para ver los argumentos, consulte `msgen help list`.  |
 |`submit`             |Envía una solicitud de flujo de trabajo al servicio. Para ver los argumentos, consulte `msgen help submit`.|
 |`status`             |Devuelve el estado del flujo de trabajo que especificó `--workflow-id`. Consulte también `msgen help status`. |
 |`cancel`             |Envía una solicitud para cancelar el procesamiento del flujo de trabajo que especificó `--workflow-id`. Consulte también `msgen help cancel`. |

## <a name="where-do-i-get-the-value-for---api-url-base"></a>¿Dónde se puede obtener el valor de `--api-url-base`?
Vaya a Azure Portal y abra la página de la cuenta de Genomics. En el encabezado **Administración**, elija **Claves de acceso**. Allí encontrará la URL de la API y las claves de acceso.

## <a name="where-do-i-get-the-value-for---access-key"></a>¿Dónde se puede obtener el valor de `--access-key`?
Vaya a Azure Portal y abra la página de la cuenta de Genomics. En el encabezado **Administración**, elija **Claves de acceso**. Allí encontrará la URL de la API y las claves de acceso.

## <a name="why-do-i-need-two-access-keys"></a>¿Por qué necesito dos claves de acceso?
Necesita dos claves de acceso en caso de que quiera actualizarlas (regenerarlas) sin interrumpir el uso del servicio. Por ejemplo, si quiere actualizar la primera clave, debe hacer que todos los flujos de trabajo nuevos usen la segunda clave. A continuación, espere a que todos los flujos de trabajo que usan la primera clave finalicen para poder actualizar dicha clave.

## <a name="do-you-save-my-storage-account-keys"></a>¿Se guardan mis claves de cuenta de almacenamiento?
La clave de cuenta de almacenamiento se usa para crear tokens de acceso a corto plazo para el servicio de Microsoft Genomics y así poder leer los archivos de entrada y escribir en los archivos de salida. La duración predeterminada del token es de 48 horas. La duración del token se puede cambiar con la opción `-sas/--sas-duration` del comando Enviar; el valor se indica en horas.

## <a name="what-genome-references-can-i-use"></a>¿Qué referencias genómicas puedo usar?

Se admiten estas referencias:

 |Referencia              | Valor de `-pa/--process-args` |
 |:-------------         |:-------------                 |
 |b37                    | `R=b37m1`                     |
 |hg38                   | `R=hg38m1`                    |      
 |hg38 (ningún análisis alternativo) | `R=hg38m1x`                   |  
 |hg19                   | `R=hg19m1`                    |    

## <a name="how-do-i-format-my-command-line-arguments-as-a-config-file"></a>¿Cómo puedo dar el formato de un archivo de configuración a los argumentos de la línea de comandos? 

msgen comprende los archivos de configuración en el formato siguiente:
* Todas las opciones se proporcionan como pares clave-valor con valores separados de las claves mediante dos puntos.
  Se ignora el espacio en blanco.
* Las líneas que empiezan con `#` se ignoran.
* Cualquier argumento de la línea de comandos que tenga un formato largo se puede convertir en una clave; para ello, hay que quitar los guiones principales y reemplazar los que están entre las palabras con guiones bajos. Estos son algunos ejemplos de conversión:

  |Argumento de línea de comandos            | Línea del archivo de configuración |
  |:-------------                   |:-------------                 |
  |`-u/--api-url-base https://url`  | *api_url_base:https://url*    |
  |`-k/--access-key KEY`            | *access_key:KEY*              |      
  |`-pa/--process-args R=B37m1`     | *process_args:R-b37m1*        |  

## <a name="next-steps"></a>Pasos siguientes

Use los siguientes recursos para empezar a trabajar con Microsoft Genomics:
- Ejecutar el primer flujo de trabajo mediante el servicio de Microsoft Genomics. [Ejecutar un flujo de trabajo a través del servicio Microsoft Genomics](quickstart-run-genomics-workflow-portal.md)
- Enviar sus propios datos para que el servicio de Microsoft Genomics los procese: [FASTQ emparejados](quickstart-input-pair-FASTQ.md) | [BAM](quickstart-input-BAM.md) | [varios FASTQ o BAM](quickstart-input-multiple.md) 

