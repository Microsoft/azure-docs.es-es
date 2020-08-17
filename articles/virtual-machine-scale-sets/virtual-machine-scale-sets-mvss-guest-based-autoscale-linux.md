---
title: Uso del escalado automático de Azure con métricas de invitado en una plantilla de conjunto de escalado de Linux
description: Obtenga información acerca de cómo usar el escalado automático con métricas de invitado en una plantilla de conjunto de escalado de máquinas virtuales de Linux
author: ju-shim
ms.author: jushiman
ms.topic: how-to
ms.service: virtual-machine-scale-sets
ms.subservice: autoscale
ms.date: 04/26/2019
ms.reviewer: avverma
ms.custom: avverma
ms.openlocfilehash: 549f8fbc1e3acf435011f223faeb5b8240f0c55d
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87080427"
---
# <a name="autoscale-using-guest-metrics-in-a-linux-scale-set-template"></a>Escalado automático con métricas de invitado en una plantilla de conjunto de escalado de Linux

Hay dos tipos generales de métricas en Azure que se recopilan desde máquinas virtuales y conjuntos de escalado: las métricas de host y las métricas de invitado. A nivel general, si quiere usar métricas estándar de CPU, disco y red, las métricas de host resultan adecuadas. Sin embargo, si necesita una selección de métricas más grande, debería considerar las métricas de invitado.

Las métricas de host no requieren configuración adicional porque las recopila la máquina virtual host, mientras que las métricas de invitado requieren la instalación de la [extensión Azure Diagnostics para Windows](../virtual-machines/extensions/diagnostics-template.md) o [Azure Diagnostics para Linux](../virtual-machines/extensions/diagnostics-linux.md) en la máquina virtual invitada. Un motivo habitual para usar las métricas de invitado en lugar de las métricas de host es que las métricas de invitado proporcionan una mayor variedad de métricas que las de host. Un ejemplo es las métricas de consumo de memoria, que solo están disponibles mediante las métricas de invitado. La lista de las métricas de host admitidas se encuentran [aquí](../azure-monitor/platform/metrics-supported.md), y las métricas de invitado de uso frecuente se encuentran [aquí](../azure-monitor/platform/autoscale-common-metrics.md). En este artículo se muestra cómo modificar la [plantilla de conjunto de escalado viable básico](virtual-machine-scale-sets-mvss-start.md) para usar reglas de escalado automático en función de las métricas de invitado para conjuntos de escalado de Linux.

## <a name="change-the-template-definition"></a>Cambio de la definición de la plantilla

En un [artículo anterior](virtual-machine-scale-sets-mvss-start.md) creamos una plantilla de conjunto de escalado básico. Ahora usaremos esa plantilla anterior y la modificaremos para crear una plantilla que implementa un conjunto de escalado de Linux con escalabilidad automática basada en métricas de invitado.

Primero, se van a agregar parámetros para `storageAccountName` y `storageAccountSasToken`. El agente de Diagnostics almacena los datos de las métricas en una [tabla](../cosmos-db/tutorial-develop-table-dotnet.md), en esta cuenta de almacenamiento. A partir de la versión 3.0 del agente de Diagnostics para Linux ya no admite el uso de una clave de acceso de almacenamiento. En su lugar, use un [token de SAS](../storage/common/storage-sas-overview.md).

```diff
     },
     "adminPassword": {
       "type": "securestring"
+    },
+    "storageAccountName": {
+      "type": "string"
+    },
+    "storageAccountSasToken": {
+      "type": "securestring"
     }
   },
```

A continuación, modifique el conjunto de escalado `extensionProfile` para incluir la extensión Diagnostics. En esta configuración, se especifica el identificador de recurso del conjunto de escalado del que se recopilarán las métricas, así como la cuenta de almacenamiento y el token de SAS que se usará para almacenarlas. Especifique con qué frecuencia se agregan las métricas (en este caso, cada minuto) y las métricas para las que desea realizar un seguimiento (en este caso, porcentaje de memoria usado). Para más información sobre esta configuración y las métricas, aparte del porcentaje de memoria usado, consulte [esta documentación](../virtual-machines/extensions/diagnostics-linux.md).

```diff
                 }
               }
             ]
+          },
+          "extensionProfile": {
+            "extensions": [
+              {
+                "name": "LinuxDiagnosticExtension",
+                "properties": {
+                  "publisher": "Microsoft.Azure.Diagnostics",
+                  "type": "LinuxDiagnostic",
+                  "typeHandlerVersion": "3.0",
+                  "settings": {
+                    "StorageAccount": "[parameters('storageAccountName')]",
+                    "ladCfg": {
+                      "diagnosticMonitorConfiguration": {
+                        "performanceCounters": {
+                          "sinks": "WADMetricJsonBlob",
+                          "performanceCounterConfiguration": [
+                            {
+                              "unit": "percent",
+                              "type": "builtin",
+                              "class": "memory",
+                              "counter": "percentUsedMemory",
+                              "counterSpecifier": "/builtin/memory/percentUsedMemory",
+                              "condition": "IsAggregate=TRUE"
+                            }
+                          ]
+                        },
+                        "metrics": {
+                          "metricAggregation": [
+                            {
+                              "scheduledTransferPeriod": "PT1M"
+                            }
+                          ],
+                          "resourceId": "[resourceId('Microsoft.Compute/virtualMachineScaleSets', 'myScaleSet')]"
+                        }
+                      }
+                    }
+                  },
+                  "protectedSettings": {
+                    "storageAccountName": "[parameters('storageAccountName')]",
+                    "storageAccountSasToken": "[parameters('storageAccountSasToken')]",
+                    "sinksConfig": {
+                      "sink": [
+                        {
+                          "name": "WADMetricJsonBlob",
+                          "type": "JsonBlob"
+                        }
+                      ]
+                    }
+                  }
+                }
+              }
+            ]
           }
         }
       }
```

Por último, agregue un recurso `autoscaleSettings` para configurar el escalado automático en función de estas métricas. Este recurso tiene una cláusula `dependsOn` que hace referencia al conjunto de escalado para asegurarse de que el conjunto de escalado existe antes de intentar realizar el escalado automático. Si se elige una métrica diferente para el escalado automático, se utilizaría el valor de `counterSpecifier` de la configuración de la extensión Diagnostics como valor de `metricName` en la configuración del escalado automático. Para más información sobre la configuración de escalado automático, consulte los [procedimientos recomendados de escalado automático](../azure-monitor/platform/autoscale-best-practices.md) y la [documentación de referencia de la API de REST de Azure Monitor](/rest/api/monitor/autoscalesettings).

```diff
+    },
+    {
+      "type": "Microsoft.Insights/autoscaleSettings",
+      "apiVersion": "2015-04-01",
+      "name": "guestMetricsAutoscale",
+      "location": "[resourceGroup().location]",
+      "dependsOn": [
+        "Microsoft.Compute/virtualMachineScaleSets/myScaleSet"
+      ],
+      "properties": {
+        "name": "guestMetricsAutoscale",
+        "targetResourceUri": "[resourceId('Microsoft.Compute/virtualMachineScaleSets', 'myScaleSet')]",
+        "enabled": true,
+        "profiles": [
+          {
+            "name": "Profile1",
+            "capacity": {
+              "minimum": "1",
+              "maximum": "10",
+              "default": "3"
+            },
+            "rules": [
+              {
+                "metricTrigger": {
+                  "metricName": "/builtin/memory/percentUsedMemory",
+                  "metricNamespace": "",
+                  "metricResourceUri": "[resourceId('Microsoft.Compute/virtualMachineScaleSets', 'myScaleSet')]",
+                  "timeGrain": "PT1M",
+                  "statistic": "Average",
+                  "timeWindow": "PT5M",
+                  "timeAggregation": "Average",
+                  "operator": "GreaterThan",
+                  "threshold": 60
+                },
+                "scaleAction": {
+                  "direction": "Increase",
+                  "type": "ChangeCount",
+                  "value": "1",
+                  "cooldown": "PT1M"
+                }
+              },
+              {
+                "metricTrigger": {
+                  "metricName": "/builtin/memory/percentUsedMemory",
+                  "metricNamespace": "",
+                  "metricResourceUri": "[resourceId('Microsoft.Compute/virtualMachineScaleSets', 'myScaleSet')]",
+                  "timeGrain": "PT1M",
+                  "statistic": "Average",
+                  "timeWindow": "PT5M",
+                  "timeAggregation": "Average",
+                  "operator": "LessThan",
+                  "threshold": 30
+                },
+                "scaleAction": {
+                  "direction": "Decrease",
+                  "type": "ChangeCount",
+                  "value": "1",
+                  "cooldown": "PT1M"
+                }
+              }
+            ]
+          }
+        ]
+      }
     }
   ]
 }
```





## <a name="next-steps"></a>Pasos siguientes

[!INCLUDE [mvss-next-steps-include](../../includes/mvss-next-steps.md)]
