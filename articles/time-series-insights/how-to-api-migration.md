---
title: Migración a las nuevas versiones de API de Azure Time Series Insights Gen2 | Microsoft Docs
description: Cómo actualizar los entornos de Azure Time Series Insights Gen2 para que usen las nuevas versiones disponibles con carácter general.
ms.service: time-series-insights
services: time-series-insights
author: shreyasharmamsft
ms.author: shresha
manager: dpalled
ms.workload: big-data
ms.topic: conceptual
ms.date: 08/12/2020
ms.custom: shresha
ms.openlocfilehash: 784c19844c658af6850c755244314145223c45ef
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88163958"
---
# <a name="migrating-to-new-azure-time-series-insights-gen2-api-versions"></a>Migración a las nuevas versiones de API de Azure Time Series Insights Gen2

## <a name="overview"></a>Información general

Si ha creado un entorno de Azure Time Series Insights Gen2 cuando su versión era la versión preliminar pública (antes del 16 de julio de 2020), actualice el entorno de TSI para usar las nuevas versiones de API disponibles con carácter general siguiendo los pasos descritos en este artículo.

La nueva versión de la API es la `2020-07-31` y utiliza una [sintaxis de expresiones de serie temporal](https://docs.microsoft.com/rest/api/time-series-insights/reference-time-series-expression-syntax) actualizada.

Los usuarios deben migrar las [variables del modelo de serie temporal](./concepts-variables.md) del entorno, las consultas guardadas, las consultas de Power BI y cualquier herramienta personalizada que realice llamadas a los puntos de conexión de la API. Si tiene alguna pregunta o duda sobre este proceso de migración, envíe una incidencia de soporte técnico a través de Azure Portal y mencione este documento.

> [!IMPORTANT]
> La versión preliminar de la API `2018-11-01-preview` seguirá siendo compatible hasta el 31 de octubre de 2020. Complete todos los pasos aplicables de esta migración antes de esa fecha para evitar cualquier interrupción del servicio.

## <a name="migrate-time-series-model-and-saved-queries"></a>Migración del modelo de serie temporal y las consultas guardadas

Para ayudar a los usuarios a migrar sus [variables del modelo de serie temporal](./concepts-variables.md) y las consultas guardadas, dispone de una herramienta integrada disponible a través del [Explorador de Time Series Insights](https://insights.timeseries.azure.com). Navegue hasta el entorno que desea migrar y siga los pasos que se indican a continuación. **Puede completar la migración parcialmente y volver para completarla más adelante; sin embargo, no se puede revertir ninguna de las actualizaciones.**

> [!NOTE]
> Debe ser un colaborador del entorno para realizar actualizaciones en el modelo de serie temporal y en las consultas guardadas. Si no es un colaborador, solo podrá migrar sus consultas guardadas personales. Revise las [directivas de acceso del entorno](./concepts-access-policies.md) y el nivel de acceso antes de continuar.

1. El explorador le pedirá que actualice la sintaxis que las variables del modelo de serie temporal y las consultas guardadas usan.

    [![Mensaje de solicitud](media/api-migration/ux-prompt.png)](media/v2-update-overview/overview-one.png#lightbox)

    Si cierra accidentalmente la notificación, la puede encontrar en el panel de notificaciones.

1. Haga clic en **Show Updates** (Mostrar actualizaciones) para abrir la herramienta de migración.

1. Haga clic en **Download types** (Descargar tipos). Dado que la migración sobrescribirá los tipos actuales para modificar la sintaxis de las variables, se le pedirá que guarde una copia de los tipos actuales. La herramienta le enviará una notificación cuando se hayan descargado los tipos.

    [![Descargar tipos](media/api-migration/ux-migration-tool.png)](media/v2-update-overview/overview-one.png#lightbox)

1. Haga clic en **Update variables** (Actualizar variables). La herramienta le enviará una notificación cuando se hayan actualizado las variables.

    > [!IMPORTANT]
    > Si actualiza los tipos, las aplicaciones personalizadas que usen una versión anterior de la API (`2018-11-01-preview`) necesitarán usar la nueva versión de la API (`2020-07-31`) para seguir funcionando. Si no está seguro de qué versión de API está usando, consulte al administrador antes de actualizar. Puede cerrar la herramienta de migración y volver a estos pasos más adelante. Obtenga más información sobre [cómo migrar aplicaciones personalizadas](#migrate-custom-applications).

    [![Actualizar variables](media/api-migration/ux-migration-tool-downloaded-types.png)](media/v2-update-overview/overview-one.png#lightbox)

1. Haga clic en **Update saved queries** (Actualizar consultas guardadas). La herramienta le enviará una notificación cuando se hayan actualizado las variables.

    [![Actualizar consultas guardadas](media/api-migration/ux-migration-tool-updated-variables.png)](media/v2-update-overview/overview-one.png#lightbox)

1. Haga clic en **Done**(Listo).

    [![Migración completada](media/api-migration/ux-migration-tool-updated-saved-queries.png)](media/v2-update-overview/overview-one.png#lightbox)

Revise el entorno actualizado mediante la representación gráfica de algunas de las variables creadas recientemente y las consultas guardadas. Si ve un comportamiento inesperado durante la representación gráfica, envíenos sus comentarios mediante la herramienta de comentarios del explorador.

## <a name="migrate-power-bi-queries"></a>Migración de consultas de Power BI

Si ha generado consultas mediante el conector de Power BI, estas realizan llamadas a Azure Time Series Insights mediante la versión preliminar de la API y la antigua sintaxis de expresiones de serie temporal. Estas consultas seguirán recuperando los datos correctamente hasta que la API de versión preliminar esté en desuso.

Para actualizar las consultas para que usen la nueva versión de la API y la nueva sintaxis de expresiones de serie temporal, las consultas deberán volver a generarse desde el explorador. Obtenga más información sobre cómo [crear consultas mediante el conector de Power BI](./how-to-connect-power-bi.md).

> [!NOTE]
> Debe usar la versión de julio de 2020 de Power BI Desktop. Si no la usa, puede aparecer un [error de versión de carga de consulta no válida](./how-to-diagnose-troubleshoot.md#problem-power-bi-connector-shows-unable-to-connect).

## <a name="migrate-custom-applications"></a>Migración de aplicaciones personalizadas

Si la aplicación personalizada realiza llamadas a los siguientes puntos de conexión de REST, basta con actualizar la versión de la API a `2020-07-31` en el URI:

- API de modelo de serie temporal
  - API de configuración del modelo
    - [Get](https://docs.microsoft.com/rest/api/time-series-insights/dataaccessgen2/modelsettings/get)
    - [Actualizar](https://docs.microsoft.com/rest/api/time-series-insights/dataaccessgen2/modelsettings/update)
  - API de instancia
    - [Todas las operaciones por lotes](https://docs.microsoft.com/rest/api/time-series-insights/dataaccessgen2/timeseriesinstances/executebatch)
    - [Lista](https://docs.microsoft.com/rest/api/time-series-insights/dataaccessgen2/timeseriesinstances/list)
    - [Búsqueda](https://docs.microsoft.com/rest/api/time-series-insights/dataaccessgen2/timeseriesinstances/search)
    - [Sugerir](https://docs.microsoft.com/rest/api/time-series-insights/dataaccessgen2/timeseriesinstances/suggest)
  - API de jerarquía
    - [Todas las operaciones por lotes](https://docs.microsoft.com/rest/api/time-series-insights/dataaccessgen2/timeserieshierarchies/executebatch)
    - [Lista](https://docs.microsoft.com/rest/api/time-series-insights/dataaccessgen2/timeserieshierarchies/list)
  - API de tipos
    - [Operaciones Delete y Get](https://docs.microsoft.com/rest/api/time-series-insights/dataaccessgen2/timeseriestypes/executebatch)
    - [Lista](https://docs.microsoft.com/rest/api/time-series-insights/dataaccessgen2/timeseriestypes/list)

En el caso de los siguientes puntos de conexión de REST, debe actualizar la versión de la API a `2020-07-31` en el URI y asegurarse de que todas las repeticiones de la propiedad `tsx` usan la [sintaxis de expresiones de serie temporal](https://docs.microsoft.com/rest/api/time-series-insights/reference-time-series-expression-syntax) actualizada.

- API de tipos
  - [Operación Put](https://docs.microsoft.com/rest/api/time-series-insights/dataaccessgen2/timeseriestypes/executebatch#typesbatchput)
- API de consulta
  - [GetEvents](https://docs.microsoft.com/rest/api/time-series-insights/dataaccessgen2/query/execute#getevents)
  - [GetSeries](https://docs.microsoft.com/rest/api/time-series-insights/dataaccessgen2/query/execute#getseries)
  - [GetAggregateSeries](https://docs.microsoft.com/rest/api/time-series-insights/dataaccessgen2/query/execute#aggregateseries)

### <a name="examples"></a>Ejemplos

#### <a name="typesbatchput"></a>TypesBatchPut

Cuerpo de solicitud anterior (usado por `2018-11-01-preview`):

```JSON
{
  "put": [
    {
      "id": "c1cb7a33-ed9b-4cf1-9958-f3162fed8ee8",
      "name": "OutdoorTemperatureSensor",
      "description": "This is an outdoor temperature sensor.",
      "variables": {
        "AverageTemperature": {
          "kind": "numeric",
          "value": {
            "tsx": "$event.[Temperature_Celsius].Double"
          },
          "filter": {
            "tsx": "$event.[Mode].String = 'outdoor'"
          },
          "aggregation": {
            "tsx": "avg($value)"
          }
        }
      }
    }
  ]
}
```

Cuerpo de solicitud actualizado (usado por `2020-07-31`):

```JSON
{
  "put": [
    {
      "id": "c1cb7a33-ed9b-4cf1-9958-f3162fed8ee8",
      "name": "OutdoorTemperatureSensor",
      "description": "This is an outdoor temperature sensor.",
      "variables": {
        "AverageTemperature": {
          "kind": "numeric",
          "value": {
            "tsx": "$event['Temperature_Celsius'].Double"
          },
          "filter": {
            "tsx": "$event['Mode'].String = 'outdoor'"
          },
          "aggregation": {
            "tsx": "avg($value)"
          }
        }
      }
    }
  ]
}
```

Como alternativa, `filter` también puede ser `$event.Mode.String = 'outdoor'`. `value` debe usar los corchetes para escapar el carácter especial (`_`).

#### <a name="getevents"></a>GetEvents

Cuerpo de solicitud anterior (usado por `2018-11-01-preview`):

```JSON
{
  "getEvents": {
    "timeSeriesId": [
      "006dfc2d-0324-4937-998c-d16f3b4f1952",
      "T1"
    ],
    "searchSpan": {
      "from": "2016-08-01T00:00:00Z",
      "to": "2016-08-01T00:16:50Z"
    },
    "filter": {
      "tsx": "($event.[Value].Double != null) OR ($event.[Status].String = 'Good')"
    },
    "projectedProperties": [
      {
        "name": "Temperature",
        "type": "Double"
      }
    ]
  }
}
```

Cuerpo de solicitud actualizado (usado por `2020-07-31`):

```JSON
{
  "getEvents": {
    "timeSeriesId": [
      "006dfc2d-0324-4937-998c-d16f3b4f1952",
      "T1"
    ],
    "searchSpan": {
      "from": "2016-08-01T00:00:00Z",
      "to": "2016-08-01T00:16:50Z"
    },
    "filter": {
      "tsx": "($event.Value.Double != null) OR ($event.Status.String = 'Good')"
    },
    "projectedProperties": [
      {
        "name": "Temperature",
        "type": "Double"
      }
    ]
  }
}
```

Como alternativa, `filter` también puede ser `($event['Value'].Double != null) OR ($event['Status'].String = 'Good')`.

#### <a name="getseries"></a>GetSeries

Cuerpo de solicitud anterior (usado por `2018-11-01-preview`):

```JSON
{
  "getSeries": {
    "timeSeriesId": [
      "006dfc2d-0324-4937-998c-d16f3b4f1952"
    ],
    "searchSpan": {
      "from": "2016-08-01T00:00:00Z",
      "to": "2016-08-01T00:16:50Z"
    },
    "inlineVariables": {
      "pressure": {
        "kind": "numeric",
        "value": {
          "tsx": "$event.[Bar-Pressure-Offset]"
        },
        "aggregation": {
          "tsx": "avg($value)"
        }
      }
    },
    "projectedVariables": [
      "pressure"
    ]
  }
}
```

Cuerpo de solicitud actualizado (usado por `2020-07-31`):

```JSON
{
  "getSeries": {
    "timeSeriesId": [
      "006dfc2d-0324-4937-998c-d16f3b4f1952"
    ],
    "searchSpan": {
      "from": "2016-08-01T00:00:00Z",
      "to": "2016-08-01T00:16:50Z"
    },
    "inlineVariables": {
      "pressure": {
        "kind": "numeric",
        "value": {
          "tsx": "$event['Bar-Pressure-Offset']"
        },
        "aggregation": {
          "tsx": "avg($value)"
        }
      }
    },
    "projectedVariables": [
      "pressure"
    ]
  }
}
```

Como alternativa, `value` también puede ser `$event['Bar-Pressure-Offset'].Double`. Si no se especifica ningún tipo de datos, siempre se supone que el tipo de datos es Double. La notación de corchetes debe usarse para escapar el carácter especial (`-`).

#### <a name="aggregateseries"></a>AggregateSeries

Cuerpo de solicitud anterior (usado por `2018-11-01-preview`):

```JSON
{
  "aggregateSeries": {
    "timeSeriesId": [
      "006dfc2d-0324-4937-998c-d16f3b4f1952"
    ],
    "searchSpan": {
      "from": "2016-08-01T00:00:00Z",
      "to": "2016-08-01T00:16:50Z"
    },
    "interval": "PT1M",
    "inlineVariables": {
      "MinTemperature": {
        "kind": "numeric",
        "value": {
          "tsx": "coalesce($event.[Temp].Double, toDouble($event.[Temp].Long))"
        },
        "aggregation": {
          "tsx": "min($value)"
        }
      },
    },
    "projectedVariables": [
      "MinTemperature"
    ]
  }
}
```

Cuerpo de solicitud actualizado (usado por `2020-07-31`):

```JSON
  "aggregateSeries": {
    "timeSeriesId": [
      "006dfc2d-0324-4937-998c-d16f3b4f1952"
    ],
    "searchSpan": {
      "from": "2016-08-01T00:00:00Z",
      "to": "2016-08-01T00:16:50Z"
    },
    "interval": "PT1M",
    "inlineVariables": {
      "MinTemperature": {
        "kind": "numeric",
        "value": {
          "tsx": "coalesce($event.Temp.Double, toDouble($event.Temp.Long))"
        },
        "aggregation": {
          "tsx": "min($value)"
        }
      },
    },
    "projectedVariables": [
      "MinTemperature"
    ]
  }
}
```

Como alternativa, `value` también puede ser `coalesce($event['Temp'].Double, toDouble($event['Temp'].Long))`.

### <a name="potential-errors"></a>Posibles errores

#### <a name="invalidinput"></a>InvalidInput

Si ve el siguiente error, está usando la nueva versión de API (`2020-07-31`), pero no se ha actualizado la sintaxis TSX. Revise la [sintaxis de expresiones de serie temporal](https://docs.microsoft.com/rest/api/time-series-insights/reference-time-series-expression-syntax) y los ejemplos de migración anteriores. Asegúrese de que todas las propiedades `tsx` estén actualizadas correctamente antes de volver a enviar la solicitud de API.

```JSON
{
    "error": {
        "code": "InvalidInput",
        "message": "Unable to parse 'value' time series expression (TSX) in variable 'Temperature'.",
        "target": "projectedVariables.Temperature.value",
        "innerError": {
            "parseErrorDetails": [
                {
                    "pos": [
                        0,
                        5
                    ],
                    "line": 1,
                    "msg": "Unsupported Time Series Expression version TSX01 used instead of TSX00.",
                    "code": "UnsupportedTSXVersionTSX01",
                    "target": "$event"
                }
            ],
            "code": "TsxParseError"
        }
    }
}
```

## <a name="next-steps"></a>Pasos siguientes

- Pruebe su entorno a través del [Explorador de Azure Time Series Insights](./concepts-ux-panels.md) o a través de su aplicación personalizada.
