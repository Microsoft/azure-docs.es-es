---
title: Ejecución de la instancia de Azure Functions desde un paquete
description: Para que el sistema en tiempo de ejecución de Azure Functions ejecute sus funciones, monte un archivo del paquete de implementación que contenga los archivos de proyecto de la aplicación de función.
ms.topic: conceptual
ms.date: 07/15/2019
ms.openlocfilehash: b2d90cf78263b30b4315199cf1c543186a435f17
ms.sourcegitcommit: 628be49d29421a638c8a479452d78ba1c9f7c8e4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88639892"
---
# <a name="run-your-azure-functions-from-a-package-file"></a>Ejecución de la instancia de Azure Functions desde un archivo de paquete

En Azure, puede ejecutar las funciones directamente desde un archivo del paquete de implementación de la aplicación de función. La otra opción consiste en implementar los archivos en el directorio `d:\home\site\wwwroot` de la aplicación de función.

En este artículo se describen las ventajas de ejecutar las funciones desde un paquete. También se muestra cómo habilitar esta funcionalidad en la aplicación de función.

## <a name="benefits-of-running-from-a-package-file"></a>Ventajas de la ejecución desde un archivo de paquete
  
Son varias las ventajas de ejecutar las funciones desde un archivo de paquete:

+ Se reduce el riesgo de problemas de bloqueo de copia de archivos.
+ Se pueden implementar en una aplicación de producción (con reinicio).
+ Puede estar seguro de los archivos que se ejecutan en la aplicación.
+ Mejora el rendimiento de las [implementaciones de Azure Resource Manager](functions-infrastructure-as-code.md).
+ Puede reducir los tiempos de arranque en frío, especialmente para las funciones de JavaScript con árboles de paquete de npm grandes.

Para más información, consulte [este anuncio](https://github.com/Azure/app-service-announcements/issues/84).

## <a name="enabling-functions-to-run-from-a-package"></a>Ejecución de las funciones desde un paquete

Para permitir la ejecución de la aplicación de función desde un paquete, solo tiene que agregar un valor `WEBSITE_RUN_FROM_PACKAGE` a la configuración de la aplicación de función. El valor `WEBSITE_RUN_FROM_PACKAGE` puede tener uno de los siguientes valores:

| Value  | Descripción  |
|---------|---------|
| **`1`**  | Se recomienda para las aplicaciones de función que se ejecutan en Windows. Ejecución desde un archivo de paquete en la carpeta `d:\home\data\SitePackages` de la aplicación de función. Si no se va a [implementar con un archivo zip](#integration-with-zip-deployment), esta opción requiere que la carpeta tenga también un archivo denominado `packagename.txt`. Este archivo contiene solo el nombre del archivo de paquete en la carpeta, sin ningún espacio en blanco. |
|**`<URL>`**  | Ubicación del archivo de paquete específico que desea ejecutar. Cuando se usa Blob Storage, debe usar un contenedor privado con una [firma de acceso compartido (SAS)](../vs-azure-tools-storage-manage-with-storage-explorer.md#generate-a-sas-in-storage-explorer) para permitir que sistema en tiempo de ejecución de Functions acceda al paquete. Puede usar el [Explorador de Azure Storage](../vs-azure-tools-storage-manage-with-storage-explorer.md) para cargar archivos de paquete en la cuenta de Blob Storage. Al especificar una dirección URL, también debe [sincronizar desencadenadores](functions-deployment-technologies.md#trigger-syncing) tras publicar un paquete actualizado. |

> [!CAUTION]
> Al ejecutar una aplicación de función en Windows, la opción de dirección URL externa tiene peor rendimiento con el arranque en frío. Al implementar la aplicación de función en Windows, debe establecer `WEBSITE_RUN_FROM_PACKAGE` en `1` y publicar con implementación de un archivo zip.

A continuación se muestra una aplicación de función configurada para ejecutarse desde un archivo .zip que se hospeda en Azure Blob Storage:

![Valor de aplicación WEBSITE_RUN_FROM_ZIP](./media/run-functions-from-deployment-package/run-from-zip-app-setting-portal.png)

> [!NOTE]
> Actualmente, solo se admiten archivos de paquete .zip.

## <a name="integration-with-zip-deployment"></a>Integración con la implementación de archivos ZIP

La [implementación de archivos ZIP][Zip deployment for Azure Functions] es una característica de Azure App Service que le permite implementar su proyecto de aplicación de funciones en el directorio `wwwroot`. El proyecto se empaqueta como un archivo de implementación .zip. Las mismas API se pueden usar para implementar el paquete en la carpeta `d:\home\data\SitePackages`. Con el valor de configuración de la aplicación `WEBSITE_RUN_FROM_PACKAGE` de `1`, las API de implementación de archivos ZIP copian el paquete en la carpeta `d:\home\data\SitePackages` en lugar de extraer los archivos en `d:\home\site\wwwroot`. También se crea el archivo `packagename.txt`. Después de un reinicio, el paquete se monta en `wwwroot` como un sistema de archivos de solo lectura. Para más información sobre la implementación de archivos ZIP, consulte [Implementación para insertar archivos ZIP en Azure Functions](deployment-zip-push.md).

> [!NOTE]
> Cuando se produce una implementación, se desencadena un reinicio de la aplicación de funciones. Antes del reinicio, todas las ejecuciones de función existentes pueden finalizar o agotar el tiempo de espera. Para más información, consulte [Comportamientos de implementación](functions-deployment-technologies.md#deployment-behaviors).

## <a name="adding-the-website_run_from_package-setting"></a>Incorporación del valor WEBSITE_RUN_FROM_PACKAGE

[!INCLUDE [Function app settings](../../includes/functions-app-settings.md)]


## <a name="troubleshooting"></a>Solución de problemas

- La ejecución desde el paquete hace que `wwwroot` sea de solo lectura, por lo que recibirá un error al escribir archivos en este directorio.
- No se admiten los formatos de archivo tar y gzip.
- Esta característica no se crea con caché local.
- Para mejorar el rendimiento del arranque en frío, utilice la opción de archivo zip local (`WEBSITE_RUN_FROM_PACKAGE` = 1).
- La ejecución desde paquete es incompatible con la opción de personalización de implementación (`SCM_DO_BUILD_DURING_DEPLOYMENT=true`); el paso de compilación se omitirá durante la implementación.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Implementación continua para Azure Functions](functions-continuous-deployment.md)

[Zip deployment for Azure Functions]: deployment-zip-push.md
