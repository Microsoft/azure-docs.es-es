---
title: Solución de problemas de aplicaciones de funciones de Python en Azure Functions
description: Aprenda a solucionar problemas de funciones de Python.
author: Hazhzeng
ms.topic: article
ms.date: 07/29/2020
ms.author: hazeng
ms.custom: devx-track-python
ms.openlocfilehash: 9b9f5d389eda5d74e7e78cfcfa9a46fba7276cbd
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87846044"
---
# <a name="troubleshoot-python-errors-in-azure-functions"></a>Solución de errores de Python en Azure Functions

A continuación, se muestra una lista de las guías de solución de problemas comunes en funciones de Python:

* [ModuleNotFoundError e ImportError](#troubleshoot-modulenotfounderror)
* [No se puede importar "cygrpc"](#troubleshoot-cannot-import-cygrpc)

## <a name="troubleshoot-modulenotfounderror"></a>Solución de problemas de ModuleNotFoundError

Esta sección le ayuda a solucionar los errores relacionados con el módulo en su aplicación de funciones de Python. Estos errores suelen generar el siguiente mensaje de error de Azure Functions:

> `Exception: ModuleNotFoundError: No module named 'module_name'.`

Este error se produce cuando una aplicación de funciones de Python no puede cargar un módulo de Python. La causa principal de este error es alguno de los siguientes problemas:

- [No se puede encontrar el paquete](#the-package-cant-be-found)
- [El paquete no se ha resuelto con el archivo wheel de Linux adecuado](#the-package-isnt-resolved-with-proper-linux-wheel)
- [El paquete no es compatible con la versión del intérprete de Python](#the-package-is-incompatible-with-the-python-interpreter-version)
- [El paquete entra en conflicto con otros paquetes](#the-package-conflicts-with-other-packages)
- [El paquete solo es compatible con plataformas Windows o macOS](#the-package-only-supports-windows-or-macos-platforms)

### <a name="view-project-files"></a>Visualización de los archivos del proyecto

Para identificar la causa real del problema, debe obtener los archivos de proyecto de Python que se ejecutan en la aplicación de funciones. Si no tiene los archivos de proyecto en el equipo local, puede obtenerlos de una de las siguientes formas:

- Si la aplicación de funciones tiene la configuración de aplicación `WEBSITE_RUN_FROM_PACKAGE` y su valor es una URL, copie y pegue dicha URL en el explorador para descargar el archivo.
- Si la aplicación de funciones tiene la configuración `WEBSITE_RUN_FROM_PACKAGE` y está establecida en `1`, diríjase a `https://<app-name>.scm.azurewebsites.net/api/vfs/data/SitePackages` y descargue el archivo de la dirección URL de `href` más reciente.
- Si la aplicación de funciones no tiene la configuración de aplicación mencionada anteriormente, diríjase a `https://<app-name>.scm.azurewebsites.net/api/settings` y busque la dirección URL en `SCM_RUN_FROM_PACKAGE`. Copie y pegue la URL en el explorador para descargar el archivo.
- Si ninguna de estas opciones se ajusta a sus necesidades, diríjase a `https://<app-name>.scm.azurewebsites.net/DebugConsole` y revele el contenido en `/home/site/wwwroot`.

El resto de este artículo le ayudará a solucionar las posibles causas de este error al inspeccionar el contenido de la aplicación de funciones, identificar la causa principal y solucionar el problema específico.

### <a name="diagnose-modulenotfounderror"></a>Diagnóstico de ModuleNotFoundError

En esta sección se detallan las posibles causas principales de los errores relacionados con el módulo. Después de averiguar cuál es la causa principal más probable, puede dirigirse a la mitigación de riesgos correspondiente.

#### <a name="the-package-cant-be-found"></a>No se puede encontrar el paquete

Vaya a `.python_packages/lib/python3.6/site-packages/<package-name>` o `.python_packages/lib/site-packages/<package-name>`. Si la ruta de acceso del archivo no existe, es probable que esta ruta faltante sea la causa principal.

Usar herramientas de terceros u obsoletas durante la implementación puede generar este problema.

Consulte [Habilitación de la compilación remota](#enable-remote-build) o [Creación de dependencias nativas](#build-native-dependencies) para consultar la mitigación.

#### <a name="the-package-isnt-resolved-with-proper-linux-wheel"></a>El paquete no se ha resuelto con el archivo wheel de Linux adecuado

Vaya a `.python_packages/lib/python3.6/site-packages/<package-name>-<version>-dist-info` o `.python_packages/lib/site-packages/<package-name>-<version>-dist-info`. Use su editor de texto preferido para abrir el archivo **wheel** y compruebe la sección **Tag:** . Si el valor de la etiqueta no contiene la palabra **linux**, este podría ser el problema.

Las funciones de Python solo se ejecutan en Linux con Azure: Azure Functions Runtime v2.x se ejecuta en Debian Stretch y la versión v3.x en Debian Buster. Se espera que el artefacto contenga los archivos binarios de Linux correctos. El uso de la marca `--build local` en Core Tools o en herramientas de terceros u obsoletas puede causar el uso de archivos binarios anteriores.

Consulte [Habilitación de la compilación remota](#enable-remote-build) o [Creación de dependencias nativas](#build-native-dependencies) para consultar la mitigación.

#### <a name="the-package-is-incompatible-with-the-python-interpreter-version"></a>El paquete no es compatible con la versión del intérprete de Python

Vaya a `.python_packages/lib/python3.6/site-packages/<package-name>-<version>-dist-info` o `.python_packages/lib/site-packages/<package-name>-<version>-dist-info`. Con un editor de texto, abra el archivo METADATA y compruebe la sección **Classifiers:** . Si esta sección no contiene `Python :: 3`, `Python :: 3.6`, `Python :: 3.7` o `Python :: 3.8`, la versión del paquete es demasiado antigua o, probablemente, el paquete ya no recibe mantenimiento.

Puede comprobar la versión de Python de la aplicación de funciones en [Azure Portal](https://portal.azure.com). Vaya a la aplicación de funciones, elija **Resource Explorer**y seleccione **Ir**.

:::image type="content" source="media/recover-module-not-found/resource-explorer.png" alt-text="Abra Resource Explorer para la aplicación de funciones en el portal":::

Una vez que se haya cargado el explorador, busque **LinuxFxVersion**, que mostrará la versión de Python.

Consulte [Actualización del paquete a la versión más reciente](#update-your-package-to-the-latest-version) o [Reemplazo del paquete por otros equivalentes](#replace-the-package-with-equivalents) para consultar la mitigación.

#### <a name="the-package-conflicts-with-other-packages"></a>El paquete entra en conflicto con otros paquetes

Si ha comprobado que el paquete se resolvió correctamente con los archivos wheel de Linux adecuados, puede haber un conflicto con otros paquetes. En el caso de algunos paquetes, la documentación de PyPi puede señalar claramente a los módulos incompatibles. Por ejemplo, en [`azure 4.0.0`](https://pypi.org/project/azure/4.0.0/), hay una instrucción como la siguiente:

<pre>This package isn't compatible with azure-storage.
If you installed azure-storage, or if you installed azure 1.x/2.x and didn’t uninstall azure-storage,
you must uninstall azure-storage first.</pre>

Puede encontrar la documentación para su versión del paquete en `https://pypi.org/project/<package-name>/<package-version>`.

Consulte [Actualización del paquete a la versión más reciente](#update-your-package-to-the-latest-version) o [Reemplazo del paquete por otros equivalentes](#replace-the-package-with-equivalents) para consultar la mitigación.

#### <a name="the-package-only-supports-windows-or-macos-platforms"></a>El paquete solo es compatible con plataformas Windows o macOS

Abra el archivo `requirements.txt` con un editor de texto y consulte el paquete en `https://pypi.org/project/<package-name>`. Algunos paquetes solo se ejecutan en plataformas Windows o macOS. Por ejemplo, pywin32 solo se ejecuta en Windows.

El error `Module Not Found` puede no producirse cuando se usa Windows o macOS para el desarrollo local. Sin embargo, el paquete no se puede importar en Azure Functions, que usa Linux en el entorno de ejecución. La causa más probable es el uso de `pip freeze` para exportar el entorno virtual al archivo requirements.txt desde la máquina Windows o macOS durante la inicialización del proyecto.

Consulte [Reemplazo del paquete por otros equivalentes](#replace-the-package-with-equivalents) o [Escritura a mano de requirements.txt](#handcraft-requirementstxt) para consultar la mitigación.

### <a name="mitigate-modulenotfounderror"></a>Mitigación de ModuleNotFoundError

A continuación se indican posibles mitigaciones de riesgos para los problemas relacionados con el módulo. Use los [diagnósticos anteriores](#diagnose-modulenotfounderror) para determinar cuál de estas mitigaciones debe probar.

#### <a name="enable-remote-build"></a>Habilitación de la compilación remota

Asegúrese de que la compilación remota esté habilitada. La manera de hacer esto depende del método de implementación.

## <a name="visual-studio-code"></a>[Visual Studio Code](#tab/vscode)
Asegúrese de que está instalada la versión más reciente de la [extensión de Azure Functions para Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions). Compruebe que existe el archivo `.vscode/settings.json` y que contiene la configuración `"azureFunctions.scmDoBuildDuringDeployment": true`. De lo contrario, cree el archivo con la opción de configuración `azureFunctions.scmDoBuildDuringDeployment` habilitada y vuelva a implementar el proyecto.

## <a name="azure-functions-core-tools"></a>[Azure Functions Core Tools](#tab/coretools)

Asegúrese de que esté instalada la versión más reciente de [Azure Functions Core Tools](https://github.com/Azure/azure-functions-core-tools/releases). Vaya a la carpeta de proyecto de la función local y use `func azure functionapp publish <app-name>` para la implementación.

## <a name="manual-publishing"></a>[Publicación manual](#tab/manual)

Si va a publicar manualmente el paquete en el punto de conexión `https://<app-name>.scm.azurewebsites.net/api/zipdeploy`, asegúrese de que **SCM_DO_BUILD_DURING_DEPLOYMENT** y **ENABLE_ORYX_BUILD** se hayan establecido en **true**. Para más información, consulte [Trabajo con la configuración de la aplicación](functions-how-to-use-azure-function-app-settings.md#settings).

---

#### <a name="build-native-dependencies"></a>Creación de dependencias nativas

Asegúrese de que esté instalada la versión más reciente de **Docker** y [Azure Functions Core Tools](https://github.com/Azure/azure-functions-core-tools/releases). Vaya a la carpeta de proyecto de la función local y use `func azure functionapp publish <app-name> --build-native-deps` para la implementación.

#### <a name="update-your-package-to-the-latest-version"></a>Actualización del paquete a la versión más reciente

Examine la versión más reciente del paquete en `https://pypi.org/project/<package-name>` y compruebe la sección **Classifiers:** . El paquete debe ser del tipo `OS Independent` o debe ser compatible con `POSIX` o `POSIX :: Linux` en **Sistema operativo**. Además, el lenguaje de programación debe contener `Python :: 3`, `Python :: 3.6`, `Python :: 3.7` o `Python :: 3.8`.

Si los valores son correctos, puede actualizar el paquete a la versión más reciente. Para ello, cambie la línea `<package-name>~=<latest-version>` del archivo requirements.txt.

#### <a name="handcraft-requirementstxt"></a>Escritura a mano de requirements.txt

Algunos desarrolladores usan `pip freeze > requirements.txt` para generar la lista de paquetes de Python de los entornos de desarrollo. Aunque este elemento práctico debería funcionar en la mayoría de los casos, puede haber problemas en los escenarios de implementación multiplataforma; por ejemplo, desarrollar funciones localmente en Windows o macOS, pero publicarlas en una aplicación de funciones que se ejecuta en Linux. En este escenario, `pip freeze` puede incluir dependencias específicas para el sistema operativo inesperadas o dependencias para el entorno de desarrollo local. Estas dependencias pueden interrumpir el funcionamiento de la aplicación de funciones de Python cuando se ejecute en Linux.

El procedimiento recomendado consiste en comprobar la instrucción Import de cada archivo .py en el código fuente del proyecto e incluir solo esos módulos en el archivo requirements.txt. Esto garantiza que la resolución de paquetes se administrará correctamente en sistemas operativos diferentes.

#### <a name="replace-the-package-with-equivalents"></a>Reemplazo del paquete por otros equivalentes

En primer lugar, es necesario revisar la versión más reciente del paquete en `https://pypi.org/project/<package-name>`. Normalmente, este paquete tiene su propia página de GitHub; vaya a la sección **Issues** (Incidencias) en GitHub y busque si se ha corregido el problema. Si ese es el caso, actualice el paquete a la versión más reciente.

En ocasiones, es posible que el paquete se haya integrado en la [biblioteca estándar de Python](https://docs.python.org/3/library/) (por ejemplo, pathlib). Si ese es el caso, dado que se proporciona una distribución de Python concreta en Azure Functions (Python 3.6, Python 3.7 y Python 3.8), el paquete se debe quitar del archivo requirements.txt.

Sin embargo, si el problema no se ha corregido y tiene una fecha límite, se recomienda que investigue un poco y busque un paquete similar para su proyecto. Normalmente, la comunidad de Python le proporcionará una gran variedad de bibliotecas similares que puede usar.

---

## <a name="troubleshoot-cannot-import-cygrpc"></a>Solución de problemas cuando no se puede importar "cygrpc"

Esta sección le ayuda a solucionar los errores relacionados con "cygrpc" en su aplicación de funciones de Python. Estos errores suelen generar el siguiente mensaje de error de Azure Functions:

> `Cannot import name 'cygrpc' from 'grpc._cython'`

Este error se produce cuando una aplicación de funciones de Python no puede iniciarse con un intérprete de Python adecuado. La causa principal de este error es alguno de los siguientes problemas:

- [El intérprete de Python no coincide con la arquitectura del sistema operativo](#the-python-interpreter-mismatches-os-architecture).
- [El intérprete de Python no es compatible con el trabajo de Python de Azure Functions](#the-python-interpreter-is-not-supported-by-azure-functions-python-worker).

### <a name="diagnose-cygrpc-reference-error"></a>Diagnóstico del error de referencia de "cygrpc"

#### <a name="the-python-interpreter-mismatches-os-architecture"></a>El intérprete de Python no coincide con la arquitectura del sistema operativo

Lo más probable es que se deba a que un intérprete de Python de 32 bits está instalado en el sistema operativo de 64 bits.

Si está ejecutando en un sistema operativo x64, asegúrese de que el intérprete de Python 3.6, 3.7 o 3.8 también esté en la versión de 64 bits.

Puede comprobar el valor de bits del intérprete de Python mediante los siguientes comandos:

En Windows en PowerShell: `py -c 'import platform; print(platform.architecture()[0])'`

En un shell similar a Unix: `python3 -c 'import platform; print(platform.architecture()[0])'`

Si hay una discrepancia entre el valor de bits del intérprete de Python y la arquitectura del sistema operativo, descargue un intérprete de Python adecuado desde [Python Software Foundation](https://python.org/downloads/release).

#### <a name="the-python-interpreter-is-not-supported-by-azure-functions-python-worker"></a>El intérprete de Python no es compatible con el trabajo de Python de Azure Functions

El trabajo de Python de Azure Functions solo admite Python 3.6, 3.7 y 3.8.
Compruebe si el intérprete de Python coincide con la versión esperada mediante `py --version` en Windows o `python3 --version` en sistemas similares a Unix. Asegúrese de que el resultado devuelto es Python 3.6.x, Python 3.7.x o Python 3.8.x.

Si la versión del intérprete de Python no cumple con nuestra expectativa, descargue el intérprete Python 3.6, 3.7 o 3.8 desde [Python Software Foundation](https://python.org/downloads/release).

## <a name="next-steps"></a>Pasos siguientes

Si no puede resolver el problema, informe al equipo de Functions:

> [!div class="nextstepaction"]
> [Notificar un problema sin resolver](https://github.com/Azure/azure-functions-python-worker/issues)
