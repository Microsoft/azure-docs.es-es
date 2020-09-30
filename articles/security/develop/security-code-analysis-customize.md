---
title: Personalización de las tareas de análisis de código de seguridad de Microsoft
titleSuffix: Azure
description: En este artículo se describe la personalización de las tareas en la extensión Análisis de código de seguridad de Microsoft
author: sukhans
manager: sukhans
ms.author: terrylan
ms.date: 07/31/2019
ms.topic: article
ms.service: security
services: azure
ms.assetid: 521180dc-2cc9-43f1-ae87-2701de7ca6b8
ms.devlang: na
ms.tgt_pltfrm: na
ms.workload: na
ms.openlocfilehash: e36485b54f209b10f9a4e6b4e6a26248955b2275
ms.sourcegitcommit: 3fc3457b5a6d5773323237f6a06ccfb6955bfb2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/11/2020
ms.locfileid: "90031405"
---
# <a name="configure-and-customize-the-build-tasks"></a>Configuración y personalización de las tareas de compilación

En este artículo se describen de forma detallada las opciones de configuración disponibles en cada una de las tareas de compilación. El artículo se inicia con las tareas para las herramientas de análisis de código de seguridad. Finaliza con las tareas posteriores al procesamiento.

## <a name="anti-malware-scanner-task"></a>Tarea de Anti-Malware Scanner

>[!NOTE]
> La tarea de compilación Anti-Malware Scanner requiere un agente de compilación con Windows Defender habilitado. Tanto Visual Studio 2017 hospedado como las versiones posteriores proporcionan ese agente. La tarea de compilación no se ejecutará en el agente hospedado de Visual Studio 2015.
>
> Aunque las firmas no se pueden actualizar en estos agentes, siempre deben tener una antigüedad inferior a tres horas.

En la captura de pantalla y el texto siguientes se muestran detalles de la configuración de tareas.

![Configuración de la tarea de compilación de Anti-Malware Scanner](./media/security-tools/5-add-anti-malware-task600.png)

En el cuadro de lista **Tipo** de la captura de pantalla, se selecciona **Básico**. Seleccione **Personalizado** para proporcionar argumentos de la línea de comandos que personalicen el examen.

Windows Defender usa el cliente de Windows Update para descargar e instalar firmas. Si se produce un error en la actualización de firmas en el agente de compilación, es probable que el código de error **HRESULT** proceda de Windows Update.

Para obtener más información sobre los errores de Windows Update y su mitigación, consulte [Códigos de error de Windows Update por componente](https://docs.microsoft.com/windows/deployment/update/windows-update-error-reference) y el artículo de TechNet sobre los [códigos de error del agente de Windows Update](https://social.technet.microsoft.com/wiki/contents/articles/15260.windows-update-agent-error-codes.aspx).

Para obtener información sobre la configuración de YAML para esta tarea, consulte nuestras [opciones de antimalware de YAML](yaml-configuration.md#anti-malware-scanner-task).

## <a name="binskim-task"></a>Tarea de BinSkim

> [!NOTE]
> Para poder ejecutar la tarea de BinSkim, su compilación debe cumplir una de estas condiciones:
>  - La compilación genera artefactos binarios a partir de código administrado.
>  - Se han confirmado artefactos binarios que desea analizar con BinSkim.

En la captura de pantalla y la lista siguientes se muestran detalles de la configuración de tareas.

![Configuración de la tarea de compilación de BinSkim](./media/security-tools/7-binskim-task-details.png)

- Establezca la configuración de compilación en Depurar para generar archivos de depuración .pdb. BinSkim usa estos archivos para volver a asignar problemas de los archivos binarios de salida al código fuente.
- Para evitar la búsqueda y creación de su propia línea de comandos:
     - En la lista **Tipo**, seleccione **Básico**.
     - En la lista **Función**, seleccione **Analizar**.
- En **Destino**, escriba uno o varios especificadores para un archivo, directorio o patrón del filtro. Estos especificadores se resuelven en uno o varios archivos binarios que se van a analizar:
    - Cuando se agregan varios destinos especificados, deben separarse con un punto y coma (;).
    - Un especificador puede ser un único archivo o contener caracteres comodín.
    - Las especificaciones del directorio siempre deben terminar con \\*.
    - Ejemplos:

```binskim-targets
           *.dll;*.exe
           $(BUILD_STAGINGDIRECTORY)\*
           $(BUILD_STAGINGDIRECTORY)\*.dll;$(BUILD_STAGINGDIRECTORY)\*.exe;
```

- Si selecciona **Línea de comandos** en la lista **Tipo**, debe ejecutar binskim.exe:
     - Asegúrese de que los primeros argumentos de binskim.exe son el verbo **analizar** seguido de una o varias especificaciones de ruta de acceso. Cada ruta de acceso puede ser una ruta de acceso completa o una ruta de acceso relativa al directorio de origen.
     - Varias rutas de acceso de destino deben separarse con un espacio.
     - Puede omitir la opción **/o** o **/output**. El valor de salida se agrega automáticamente o se reemplaza.
     - Las configuraciones de línea de comandos se muestran de la siguiente manera.

```binskim-line-args
           analyze $(Build.StagingDirectory)\* --recurse --verbose
           analyze *.dll *.exe --recurse --verbose
```

> [!NOTE]
> El signo \\* final es importante si especifica los directorios para el destino.

Para obtener más información sobre los argumentos de la línea de comandos de BinSkim, las reglas por identificador o los códigos de salida, consulte la [guía de usuario de BinSkim](https://github.com/Microsoft/binskim/blob/master/docs/UserGuide.md).

Para obtener información sobre la configuración de YAML para esta tarea, consulte nuestras [opciones de BinSkim de YAML](yaml-configuration.md#binskim-task).

## <a name="credential-scanner-task"></a>Tarea de Credential Scanner

En la captura de pantalla y la lista siguientes se muestran detalles de la configuración de tareas.

![Configuración de la tarea de compilación de Credential Scanner](./media/security-tools/3-taskdetails.png)

Las opciones disponibles incluyen:
  - **Nombre para mostrar**: Nombre de la tarea de Azure DevOps. El valor predeterminado es Run Credential Scanner (Ejecutar el escáner de credenciales).
  - **Versión principal de la herramienta**: Los valores disponibles incluyen **CredScan V2**, **CredScan V1**. Se recomienda a los clientes que usen la versión **CredScan V2**.
  - **Formato de salida**: entre los valores disponibles se incluyen **TSV**, **CSV**, **SARIF** y **PREfast**.
  - **Versión de la herramienta**: recomendamos que seleccione **Más reciente**.
  - **Examinar carpeta**: carpeta del repositorio que se va a examinar.
  - **Tipo de archivo del buscador**: opciones para encontrar el archivo de buscador utilizado para el examen.
  - **Archivo de eliminaciones**: un archivo [JSON](https://json.org/) puede eliminar los problemas en el registro de salida. Para obtener más información sobre los escenarios de eliminación, consulte la sección P+F de este artículo.
  - **Resultado detallado**: se explica por sí solo.
  - **Tamaño de lote**: el número de subprocesos simultáneos que se usan para ejecutar Credential Scanner. El valor predeterminado es 20. Los valores posibles oscilan entre 1 y 2 147 483 647.
  - **Tiempo de espera de coincidencia**: la cantidad de tiempo en segundos que se invertirá en encontrar un buscador antes de detener la comprobación.
  - **Tamaño de búfer de lectura para el examen de archivos**: tamaño en bytes del búfer que se usa durante la lectura de contenido. El valor predeterminado es 524 288.  
  - **Bytes de lectura máximos para el examen de archivos**: número máximo de bytes que van a leerse en un archivo durante un análisis de contenido. El valor predeterminado es 104 857 600.
  - **Opciones de control** > **Ejecutar esta tarea**: especifica cuándo se ejecutará la tarea. Seleccione **Condiciones personalizadas** para especificar condiciones más complejas.
  - **Versión**: versión de la tarea de compilación en Azure DevOps. Esta opción no se usa con frecuencia.

Para obtener información sobre la configuración de YAML para esta tarea, consulte nuestras [opciones del detector de credenciales de YAML](yaml-configuration.md#credential-scanner-task).

## <a name="roslyn-analyzers-task"></a>Tarea de analizadores basados en Roslyn

> [!NOTE]
> Antes de poder ejecutar la tarea de analizadores basados en Roslyn, su compilación debe cumplir estas condiciones:
>
> - La definición de compilación incluye la tarea de compilación integrada de MSBuild o VSBuild para compilar código de C# o Visual Basic. La tarea de analizadores se basa en la entrada y la salida de la tarea integrada para ejecutar la compilación de MSBuild con los analizadores basados en Roslyn habilitados.
> - El agente de compilación que ejecuta esta tarea de compilación tiene instalado Visual Studio 2017 versión 15.5 o posterior, por lo que usa la versión 2.6 o posterior del compilador.

En la lista y nota siguientes se muestran detalles de la configuración de tareas.

Las opciones disponibles incluyen:

- **Conjunto de reglas**: los valores son **SDL Required** (SDL requerido), **SDL Recommended** (SDL recomendado) o su propio conjunto de reglas personalizado.
- **Versión del analizador**: recomendamos que seleccione **Más reciente**.
- **Archivo de eliminaciones de advertencias del compilador**: un archivo de texto con una lista de identificadores de advertencias que se eliminan.
- **Opciones de control** > **Ejecutar esta tarea**: especifica cuándo se ejecutará la tarea. Elija **Condiciones personalizadas** para especificar condiciones más complejas.

> [!NOTE]
>
> - Los analizadores basados en Roslyn están integrados con el compilador y solo se pueden ejecutar como parte de la compilación de csc.exe. Por lo tanto, esta tarea requiere que se vuelva a reproducir o ejecutar el comando del compilador que se ejecutó anteriormente en la compilación. Esta reproducción o ejecución se realiza consultando Visual Studio Team Services (VSTS) para los registros de tareas de compilación de MSBuild.
>
>   No hay ninguna otra manera de que la tarea obtenga de forma confiable la línea de comandos de compilación de MSBuild de la definición de compilación. Consideramos la posibilidad de agregar un cuadro de texto de forma libre para permitir a los usuarios escribir sus líneas de comandos. Sin embargo, sería difícil en ese caso mantener esas líneas de comandos actualizadas y sincronizadas con la compilación principal.
>
>   Las compilaciones personalizadas requieren que se vuelva a reproducir todo el conjunto de comandos, no solo los comandos del compilador. En estos casos, habilitar los analizadores basados en Roslyn no es trivial ni confiable.
>
> - Los analizadores basados en Roslyn están integrados con el compilador. Para su invocación, los analizadores basados en Roslyn requieren compilación.
>
>   Esta nueva tarea de compilación se implementa recompilando proyectos de C# que ya se han compilado. La nueva tarea solo usa las tareas de compilación de MSBuild y VSBuild en la misma compilación o definición de compilación que la tarea original. Sin embargo, en este caso, la nueva tarea las usa con los analizadores basados en Roslyn habilitados.
>
>   Si la nueva tarea se ejecuta en el mismo agente que la tarea original, el resultado de la nueva tarea sobrescribe el resultado de la tarea original en la carpeta de orígenes *s*. Aunque el resultado de la compilación es el mismo, se recomienda ejecutar MSBuild, copiar los resultados en el directorio de almacenamiento provisional de los artefactos y, después, ejecutar los analizadores basados en Roslyn.

Para obtener recursos adicionales para la tarea del analizador basado en Roslyn, consulte los [analizadores basados en Roslyn](https://docs.microsoft.com/dotnet/standard/analyzers/api-analyzer) en Microsoft Docs.

Encontrará el paquete de analizadores instalado y usado por esta tarea de compilación en la página de NuGet [Microsoft.CodeAnalysis.FxCopAnalyzers](https://www.nuget.org/packages/Microsoft.CodeAnalysis.FxCopAnalyzers).

Para obtener información sobre la configuración de YAML para esta tarea, consulte nuestras [opciones de analizadores de Roslyn de YAML](yaml-configuration.md#roslyn-analyzers-task).

## <a name="tslint-task"></a>Tarea de TSLint

Para obtener más información sobre TSLint, vaya al [repositorio de GitHub de TSLint](https://github.com/palantir/tslint).

>[!NOTE] 
>Como tal vez sepa, en la página principal del [repositorio de GitHub de TSLint](https://github.com/palantir/tslint) se indica que TSLint quedará obsoleto en algún momento de 2019. Microsoft investiga [ESLint](https://github.com/eslint/eslint) como tarea alternativa.

Para obtener información sobre la configuración de YAML para esta tarea, consulte nuestras [opciones de TSLint de YAML](yaml-configuration.md#tslint-task).

## <a name="publish-security-analysis-logs-task"></a>Tarea de Publish Security Analysis Logs

En la captura de pantalla y la lista siguientes se muestran detalles de la configuración de tareas.

![Configuración de la tarea de compilación de Publish Security Analysis Logs](./media/security-tools/9-publish-security-analsis-logs600.png)  

- **Nombre del artefacto**: cualquier identificador de cadena.
- **Tipo de artefacto**: Dependiendo de su selección, puede publicar los registros en Azure DevOps Server o en un recurso compartido al que pueda acceder el agente de compilación.
- **Herramientas**: puede optar por conservar los registros de herramientas específicas, o bien puede seleccionar **Todas las herramientas** para conservar todos los registros.

Para obtener información sobre la configuración de YAML para esta tarea, consulte nuestras [opciones de publicación de registros de seguridad de YAML](yaml-configuration.md#publish-security-analysis-logs-task).

## <a name="security-report-task"></a>Tarea de Security Report

En la captura de pantalla y la lista siguientes se muestran detalles de la configuración de Security Report.

![Configuración de la tarea de compilación de Security Report](./media/security-tools/4-createsecurityanalysisreport600.png)

- **Informes**: seleccione cualquiera de los siguientes formatos: **consola de la canalización**, **archivo TSV** y **archivo HTML**. Se crea un archivo de informe para cada formato seleccionado.
- **Herramientas**: seleccione las herramientas de la definición de compilación para las que quiere obtener un resumen de los problemas detectados. Para cada herramienta seleccionada, puede haber una opción para seleccionar si en el informe de resumen ve solo los errores o los errores y las advertencias.
- **Opciones avanzadas**: si no hay registros para alguna de las herramientas seleccionadas, puede optar por registrar una advertencia o un error. Si registra un error, se produce un error en la tarea.
- **Carpeta de registros base**: puede personalizar la carpeta de registros base donde se encuentran los registros. Sin embargo, esta opción no se utiliza normalmente.

Para obtener información sobre la configuración de YAML para esta tarea, consulte nuestras [opciones de informes de seguridad de YAML](yaml-configuration.md#security-report-task).

## <a name="post-analysis-task"></a>Tarea de Post-Analysis

En la captura de pantalla y la lista siguientes se muestran detalles de la configuración de tareas.

![Configuración de la tarea de compilación de Post-Analysis](./media/security-tools/a-post-analysis600.png)

- **Herramientas**: seleccione las herramientas de la definición de compilación para las que quiere insertar una interrupción de la compilación de forma condicional. Para cada herramienta seleccionada, puede haber una opción para seleccionar si quiere realizar una interrupción solo cuando haya errores o cuando haya errores y advertencias.
- **Informe**: puede escribir de forma opcional los resultados que provocan la interrupción de la compilación. Los resultados se escriben en el archivo de registro y la ventana de consola de Azure DevOps.
- **Opciones avanzadas**: si no hay registros para alguna de las herramientas seleccionadas, puede optar por registrar una advertencia o un error. Si registra un error, se produce un error en la tarea.

Para obtener información sobre la configuración de YAML para esta tarea, consulte nuestras [opciones de análisis posteriores de YAML](yaml-configuration.md#post-analysis-task).

## <a name="next-steps"></a>Pasos siguientes

Para obtener información sobre la configuración basada en YAML, consulte la [guía de configuración de YAML](yaml-configuration.md).

Si tiene más preguntas sobre la extensión de Análisis de código de seguridad y las herramientas que se ofrecen, consulte [nuestra página de preguntas más frecuentes](security-code-analysis-faq.md).