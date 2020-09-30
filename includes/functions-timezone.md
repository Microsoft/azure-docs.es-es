---
ms.openlocfilehash: dba7a3cc7a68d360fd6e56511b71ae364f624646
ms.sourcegitcommit: d0541eccc35549db6381fa762cd17bc8e72b3423
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/09/2020
ms.locfileid: "89569292"
---
La zona horaria predeterminada que se usa con las expresiones CRON es la Hora universal coordinada (UTC). Para que la expresión CRON se base en otra zona horaria, cree una configuración de aplicación para la aplicación de función denominada `WEBSITE_TIME_ZONE`. 

El valor de esta opción de configuración depende del sistema operativo y del plan en el que se ejecuta la aplicación de funciones.

|Sistema operativo |Plan |Value |
|-|-|-|
| **Windows** |All | Establezca el valor en el nombre de la zona horaria deseada como se muestra en [Microsoft Time Zone Index](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-vista/cc749073(v=ws.10)) (Índice de zona horaria de Microsoft). |
| **Linux** |Premium<br/>Dedicado |Establezca el valor en el nombre de la zona horaria deseada como se muestra en [tz database](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones). |

> [!NOTE]
> El plan de Consumo para Linux actualmente no admite `WEBSITE_TIME_ZONE`.

Por ejemplo, *Hora estándar del Este* (Windows) o *America/New_York* (Linux) es UTC-05:00. Para que el desencadenador de temporizador se dispare a las 10:00 a.m. (Hora estándar), use la siguiente expresión NCRONTAB que representa la zona horaria UTC:

```
"0 0 15 * * *"
``` 

O bien, cree una configuración de aplicación para la aplicación de funciones denominada `WEBSITE_TIME_ZONE`, establezca el valor en `Eastern Standard Time` (Windows) o `America/New_York` (Linux) y, a continuación, use la siguiente expresión NCRONTAB: 

```
"0 0 10 * * *"
``` 

Si usa `WEBSITE_TIME_ZONE`, la hora se ajusta a los cambios de hora de la zona horaria en cuestión como, por ejemplo, al horario de verano. 
