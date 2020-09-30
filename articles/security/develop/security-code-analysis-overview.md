---
title: Introducción a la documentación de Análisis de código de seguridad de Microsoft
description: Más información sobre la extensión Microsoft Security Code Analysis. Con esta extensión, puede agregar análisis de código de seguridad a las canalizaciones de integración y entrega continuas (CI/CD) de Azure DevOps.
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
ms.openlocfilehash: af4797d7d7d7cb866674747ea8871bebb059207d
ms.sourcegitcommit: 3fc3457b5a6d5773323237f6a06ccfb6955bfb2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/11/2020
ms.locfileid: "90031252"
---
# <a name="about-microsoft-security-code-analysis"></a>Acerca de Análisis de código de seguridad de Microsoft

Con la extensión Análisis de código de seguridad de Microsoft, los equipos pueden agregar análisis de código de seguridad a sus canalizaciones de integración y entrega continuas (CI/CD) de Azure DevOps. Este análisis lo recomiendan los expertos del [Ciclo de vida de desarrollo de seguridad de Microsoft (SDL)](https://www.microsoft.com/securityengineering/sdl/practices).

Una experiencia de usuario coherente simplifica la seguridad al ocultar la complejidad de ejecutar herramientas. Con una entrega de las herramientas basada en NuGet, los equipos ya no tienen que administrar la instalación ni actualización de las herramientas. Con las interfaces de línea de comandos y básicas para las tareas de compilación, todos los usuarios todos los usuarios pueden tener tanto control sobre las herramientas como deseen.

Los equipos también pueden utilizar eficaces funcionalidades de posprocesamiento, como:

- Publicación de registros para la retención
- Generación de informes accionables y centrados en el desarrollador
- Configuración de interrupciones de compilación en pruebas de regresión

## <a name="why-should-i-use-microsoft-security-code-analysis"></a>¿Por qué debo usar Análisis de código de seguridad de Microsoft?

### <a name="security-simplified"></a>Seguridad simplificada

Agregar herramientas de Análisis de código de seguridad de Microsoft a la canalización de Azure DevOps es tan sencillo como agregar nuevas tareas. Personalice las tareas o utilice su comportamiento predeterminado. Las tareas se ejecutan como parte de la canalización de Azure DevOps y generan registros que detallan muchos tipos de resultados.

### <a name="clean-builds"></a>Compilaciones limpias

Después de solucionar los problemas iniciales informados por las herramientas, puede configurar la extensión para interrumpir las compilaciones en los nuevos problemas. Es fácil configurar la compilación de integración continua en cada solicitud de incorporación de cambios.

### <a name="set-it-and-forget-it"></a>Establézcalas y olvídese

De forma predeterminada, las herramientas y las tareas de compilación se mantienen actualizadas. Si hay una versión actualizada de una herramienta, no es necesario descargarla e instalarla. La extensión se encarga de todo.

### <a name="under-the-hood"></a>En segundo plano

Las tareas de compilación de la extensión ocultan las complejidades de:
  - Ejecutar las herramientas de análisis estáticos de seguridad.
  - Procesar los resultados de los archivos de registro para crear un informe de resumen o interrumpir la compilación.

## <a name="microsoft-security-code-analysis-tool-set"></a>Conjunto de herramientas de Análisis de código de seguridad de Microsoft

La extensión Análisis de código de seguridad de Microsoft hace que estén disponibles las versiones más recientes de las herramientas de análisis importantes. La extensión incluye tanto herramientas administradas de Microsoft como de código abierto.

Las herramientas se descargan automáticamente en el agente hospedado en la nube una vez que se usa la tarea de compilación correspondiente para configurar y ejecutar la canalización.

En esta sección se enumera el conjunto de herramientas que están disponibles en la extensión actualmente. Esté atento a la adición de más herramientas. Además, envíenos sus sugerencias sobre las herramientas que desea agregar.

### <a name="anti-malware-scanner"></a>Anti-Malware Scanner

La tarea de compilación de Anti-Malware Scanner se incluye ahora en la extensión Análisis de código de seguridad de Microsoft. Esta tarea debe ejecutarse en un agente de compilación que ya tenga instalado Windows Defender. Para más información, consulte el [sitio web de Windows Defender](https://aka.ms/defender).

### <a name="binskim"></a>BinSkim

BinSkim es un detector portátil ligero ejecutable (PE) que valida la configuración del compilador, la del vinculador y otras características de seguridad pertinentes de archivos binarios. Esta tarea de compilación proporciona un contenedor de línea de comandos alrededor de la aplicación de consola binskim.exe. BinSkim es una herramienta de código abierto. Para más información, consulte [BinSkim en GitHub](https://github.com/Microsoft/binskim).

### <a name="credential-scanner"></a>Credential Scanner

Las contraseñas y otros secretos almacenados en el código fuente actualmente son un problema importante. Credential Scanner es una herramienta de análisis estático patentada que ayuda a resolver este problema. La herramienta detecta credenciales, secretos, certificados y otros contenidos confidenciales en el código fuente y en la salida de la compilación.

### <a name="roslyn-analyzers"></a>Analizadores basados en Roslyn

Los analizadores basados en Roslyn son herramientas integradas por el compilador de Microsoft para analizar de forma estática el código administrado de C# y Visual Basic. Para más información, consulte [Analizadores basados en Roslyn](https://docs.microsoft.com/dotnet/standard/analyzers/api-analyzer).

### <a name="tslint"></a>TSLint

TSLint es una herramienta de análisis estático extensible que comprueba el código TypeScript para mejorar la legibilidad, el mantenimiento y los errores de funcionalidad. Es ampliamente compatible con los editores modernos y sistemas de compilación. Puede personalizarlo con sus propias reglas de Lint, configuraciones y formateadores. TSLint es una herramienta de código abierto. Para más información, consulte [TSLint en GitHub](https://github.com/palantir/tslint).

## <a name="analysis-and-post-processing-of-results"></a>Análisis y procesamiento posterior de los resultados

La extensión Análisis de código de seguridad de Microsoft también tiene tres tareas de posprocesamiento. Estas tareas le ayudan a analizar los resultados encontrados por las tareas de las herramientas de seguridad. Cuando se agregan a una canalización, estas tareas normalmente siguen todas las demás tareas de herramientas.

### <a name="publish-security-analysis-logs"></a>Publish Security Analysis Logs

La tarea de compilación Publish Security Analysis Logs conserva los archivos de registro de las herramientas de seguridad que se ejecutan durante la compilación. Puede leer estos registros para investigación y seguimiento.

Puede publicar los archivos de registro en Azure Artifacts como archivo .zip. También puede copiarlos en un recurso compartido de archivos accesible desde el agente de compilación privado.

### <a name="security-report"></a>Security Report

La tarea de compilación Security Report analiza los archivos de registro. Estos archivos se crean mediante las herramientas de seguridad que se ejecutan durante la compilación. La tarea de compilación crea después un único archivo de informe de resumen. Este archivo muestra todos los problemas que detectan las herramientas de análisis.

Puede configurar esta tarea para que informe de los resultados de herramientas específicas o de todas las herramientas. También puede elegir el nivel de problema que se va a notificar, como solo errores o errores y advertencias.

### <a name="post-analysis-build-break"></a>Post-Analysis (interrupción de la compilación)

Con la tarea de compilación Post Analysis, puede insertar una interrupción de la compilación que provoque un error en la compilación. Se inserta una interrupción de la compilación si una o varias herramientas de análisis notifican problemas en el código.

Puede configurar esta tarea para interrumpir la compilación de los problemas encontrados por herramientas específicas o por todas las herramientas. También puede configurarla en función de la gravedad de los problemas encontrados, como errores o advertencias.

>[!NOTE]
>Cada tarea de compilación se realiza correctamente si la tarea finaliza correctamente. Esto es cierto tanto si una herramienta encuentra problemas como si no, de modo que la compilación puede ejecutarse hasta el final al permitir que se ejecuten todas las herramientas.

## <a name="next-steps"></a>Pasos siguientes

Para más instrucciones sobre cómo incorporar e instalar Análisis de código de seguridad de Microsoft, consulte [nuestra guía de incorporación e instalación](security-code-analysis-onboard.md).

Para más información sobre la configuración de las tareas de compilación, consulte nuestra [guía de configuración](security-code-analysis-customize.md) o la [guía de configuración de YAML](yaml-configuration.md).

Si tiene más preguntas sobre la extensión y las herramientas que se ofrecen, consulte nuestra [página de preguntas más frecuentes](security-code-analysis-faq.md).
