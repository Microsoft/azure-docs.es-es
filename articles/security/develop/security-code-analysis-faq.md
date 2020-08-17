---
title: Preguntas más frecuentes sobre la documentación de Análisis de código de seguridad de Microsoft
description: Obtenga información sobre la extensión Microsoft Security Code Analysis mediante la revisión de las preguntas más frecuentes.
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
ms.openlocfilehash: f872159e538dc85121a7a6d4d6503fd18a263628
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87543051"
---
# <a name="frequently-asked-questions"></a>Preguntas más frecuentes
¿Tiene alguna pregunta? Consulte estas Preguntas más frecuentes para más información.

## <a name="general-faq"></a>Preguntas más frecuentes generales

### <a name="can-i-install-the-extension-on-my-visual-studio-team-foundation-server-instance-instead-of-on-an-azure-devops-instance"></a>¿La extensión se puede instalar en la instancia de Visual Studio Team Foundation Server en lugar de hacerlo en una instancia de Azure DevOps?

No. La extensión no está disponible para descargarla e instalarla en Visual Studio Team Foundation Server.

### <a name="do-i-have-to-run-microsoft-security-code-analysis-with-my-build"></a>¿Tengo que ejecutar Análisis de código de seguridad de Microsoft con mi compilación? 

Es posible. Depende del tipo de herramienta de análisis. Es posible que el código fuente sea lo único necesario o puede que se necesite la salida de la compilación.

Por ejemplo, Credential Scanner (CredScan) analiza los archivos dentro de la estructura de carpetas del repositorio de código. Debido a este análisis, puede ejecutar las tareas de compilación de CredScan y Publish Security Analysis Logs en una compilación independiente para obtener los resultados.

En el caso de otras herramientas que analizan los artefactos posteriores a la compilación, como BinSkim, primero necesita la compilación.

### <a name="can-i-break-my-build-when-results-are-found"></a>¿Puedo interrumpir mi compilación cuando se encuentren los resultados?

Sí. Puede introducir una interrupción de la compilación cuando cualquier herramienta informe de un problema en el archivo de registro. Simplemente agregue la tarea de compilación Post-Analysis y active la casilla de la herramienta para la que quiere interrumpir la compilación.

En la UI de la tarea Post-Analysis, puede optar por interrumpir la compilación cuando alguna herramienta notifique solo errores o tanto errores como advertencias.

### <a name="how-do-the-command-line-arguments-in-azure-devops-differ-from-those-arguments-in-the-standalone-desktop-tools"></a>¿En qué se diferencian los argumentos de la línea de comandos de Azure DevOps de los argumentos de las herramientas de escritorio independientes? 

Por lo general, las tareas de compilación de Azure DevOps son contenedores directos para los argumentos de la línea de comandos de las herramientas de seguridad. Puede pasar como argumentos a una tarea de compilación todo lo que normalmente pasa a una herramienta de línea de comandos.

Diferencias perceptibles:

- Las herramientas se ejecutan desde la carpeta de origen del agente $(Build.SourcesDirectory) o desde %BUILD_SOURCESDIRECTORY%. Un ejemplo es C:\agent\_work\1\s.
- Las rutas de acceso de los argumentos pueden ser relativas a la raíz del directorio de origen que se mostró anteriormente. Las rutas de acceso también pueden ser absolutas. Las rutas de acceso absolutas se obtienen mediante el uso de las variables de compilación de Azure DevOps o a través de la ejecución de un agente local con ubicaciones de implementación conocidas de recursos locales.
- Las herramientas proporcionan automáticamente una carpeta o una ruta de acceso de archivo de salida. Si proporciona una ubicación de salida para una tarea de compilación, esa ubicación se reemplaza por una ruta de acceso a nuestra ubicación conocida de registros en el agente de compilación.
- Algunos argumentos de la línea de comandos adicionales se modifican para algunas herramientas. Un ejemplo es agregar o quitar opciones que garantizan que no se inicie ninguna GUI.

### <a name="can-i-run-a-build-task-like-credential-scanner-across-multiple-repositories-in-an-azure-devops-build"></a>¿Puedo ejecutar una tarea de compilación como Credential Scanner en varios repositorios en una compilación de Azure DevOps?

No. No se admite la ejecución de las herramientas de desarrollo seguro en varios repositorios en una sola canalización.

### <a name="the-output-file-i-specified-isnt-being-created-or-i-cant-find-the-output-file-i-specified"></a>El archivo de salida que especifiqué no se está creando o no encuentro el archivo de salida especificado

Las tareas de compilación filtran algunas de las entradas del usuario. Para esta pregunta en concreto, actualizan la ubicación del archivo de salida generado para que sea una ubicación común en el agente de compilación. Para obtener más información acerca de esta ubicación, consulte las siguientes preguntas.

### <a name="where-are-the-output-files-generated-by-the-tools-saved"></a>¿Dónde se guardan los archivos de salida generados por las herramientas? 

Las tareas de compilación agregan automáticamente las rutas de acceso de los resultados a esta ubicación conocida en el agente de compilación $(Agent.BuildDirectory)\_sdt\logs. Como estandarizamos esta ubicación, todos los equipos que generan o consumen registros de análisis de código tienen acceso a la salida.

### <a name="can-i-queue-a-build-to-run-these-tasks-on-a-hosted-build-agent"></a>¿Se puede poner en cola una compilación para ejecutar estas tareas en un agente de compilación hospedado? 

Sí. Todas las tareas y herramientas de la extensión se pueden ejecutar en un agente de compilación hospedado.

>[!NOTE]
> La tarea de compilación Anti-Malware Scanner requiere un agente de compilación con Windows Defender habilitado. Tanto Visual Studio 2017 hospedado como las versiones posteriores proporcionan ese agente. La tarea de compilación no se ejecutará en el agente hospedado de Visual Studio 2015.
>
> Aunque las firmas no se pueden actualizar en estos agentes, siempre deben tener una antigüedad inferior a tres horas.

### <a name="can-i-run-these-build-tasks-as-part-of-a-release-pipeline-as-opposed-to-a-build-pipeline"></a>¿Puedo ejecutar estas tareas de compilación como parte de una canalización de versión (en lugar de una canalización de compilación)?

En la mayoría de los casos, sí.

Sin embargo, Azure DevOps no admite la ejecución de tareas dentro de canalizaciones de versión cuando esas tareas publican artefactos. Esta falta de compatibilidad impide que la tarea Publish Security Analysis Logs se ejecute correctamente en una canalización de versiones. En su lugar, la tarea genera un mensaje de error descriptivo.

### <a name="from-where-do-the-build-tasks-download-the-tools"></a>¿Desde dónde descargan las herramientas las tareas de compilación?

Las tareas de compilación pueden descargar los paquetes NuGet de las herramientas de la [fuente de administración de paquetes de Azure DevOps](https://securitytools.pkgs.visualstudio.com/_packaging/SecureDevelopmentTools/nuget/v3/index.json). Las tareas de compilación también pueden usar el administrador de paquetes de Node, que se debe preinstalar en el agente de compilación. Un ejemplo de este tipo de instalación es el comando **npm install tslint**.

### <a name="what-effect-does-installing-the-extension-have-on-my-azure-devops-organization"></a>¿Qué efecto tiene la instalación de la extensión en mi organización de Azure DevOps? 

Tras su instalación, las tareas de compilación de seguridad que proporciona la extensión estarán disponibles para todos los usuarios de la organización. Al crear o editar una canalización de Azure, estas tareas están disponibles en la lista de colecciones build-task. Sin embargo, la instalación de la extensión en la organización de Azure DevOps no tiene ningún impacto. La instalación no modifica ninguna configuración de cuenta, configuración de proyecto ni canalizaciones.

### <a name="does-installing-the-extension-modify-my-existing-azure-pipelines"></a>¿La instalación de la extensión modifica mi instancia existente de Azure Pipelines? 

No. La instalación de la extensión hace que las tareas de compilación de seguridad estén disponibles para agregarlas a las canalizaciones. De todos modos es necesario agregar o actualizar las definiciones de compilación para que las herramientas puedan trabajar con el proceso de compilación.

## <a name="task-specific-faq"></a>Preguntas más frecuentes específicas sobre las tareas

En esta sección se enumeran las preguntas específicas sobre las tareas de compilación.

### <a name="credential-scanner"></a>Credential Scanner

#### <a name="what-are-common-suppression-scenarios-and-examples"></a>¿Cuáles son escenarios y ejemplos comunes de eliminación?

A continuación se muestran detalles de dos de los escenarios de eliminación más comunes.

##### <a name="to-suppress-all-occurrences-of-a-given-secret-within-the-specified-path"></a>Eliminar todas las repeticiones de un secreto determinado dentro de la ruta de acceso especificada

Como se muestra en el ejemplo siguiente, se necesita la clave hash del secreto ubicada en el archivo de salida de CredScan.

```js
{
    "tool": "Credential Scanner",
    "suppressions": [
    {
        "hash": "CLgYxl2FcQE8XZgha9/UbKLTkJkUh3Vakkxh2CAdhtY=",
        "_justification": "Secret used by MSDN sample, it is fake."
    }
  ]
}
```

>[!WARNING]
> La clave hash se genera mediante una parte del valor coincidente o del contenido del archivo. Cualquier revisión de código fuente puede cambiar la clave hash y deshabilitar la regla de eliminación.

##### <a name="to-suppress-all-secrets-in-a-specified-file-or-to-suppress-the-secrets-file-itself"></a>Para eliminar todos los secretos de un archivo especificado o para eliminar el archivo de secretos

La expresión de archivo puede ser un nombre de archivo. También puede ser la parte del nombre base de una ruta de acceso completa o un nombre de archivo. No se admiten caracteres comodín.

En los ejemplos siguientes se muestra cómo eliminar el archivo \<InputPath>\src\JS\lib\angular.js

Ejemplos de reglas de eliminación válidas:

- \<InputPath>\src\JS\lib\angular.js: elimina el archivo en la ruta de acceso especificada
- \src\JS\lib\angular.js
- \JS\lib\angular.js
- \lib\angular.js
- angular.js: elimina cualquier archivo con el mismo nombre

```js
{
    "tool": "Credential Scanner",
    "suppressions": [
    {
        "file": "\\files\\AdditonalSearcher.xml", 
        "_justification": "Additional CredScan searcher specific to my team"
    },
    {
        "file": "\\files\\unittest.pfx", 
        "_justification": "Legitimate UT certificate file with private key"
    }
  ]
}
```

>[!WARNING] 
> Todos los secretos futuros que se agreguen al archivo también se eliminarán automáticamente.

#### <a name="what-are-recommended-guidelines-for-managing-secrets"></a>¿Cuáles son las directrices recomendadas para la administración de secretos?

Los recursos siguientes lo ayudarán a administrar de manera segura los secretos y a acceder a información confidencial desde dentro de sus aplicaciones:

 - [Azure Key Vault](../../key-vault/index.yml)
 - [Azure Active Directory (Azure AD)](../../azure-sql/database/authentication-aad-overview.md)
 - [Managed Service Identity (MSI) de Azure AD](https://azure.microsoft.com/blog/keep-credentials-out-of-code-introducing-azure-ad-managed-service-identity/)
 - [Identidades administradas para recursos de Azure](../../active-directory/managed-identities-azure-resources/overview.md)
 - [Uso de identidades administradas para App Service y Azure Functions](../../app-service/overview-managed-identity.md)
 - [Biblioteca de AppAuthentication](../../key-vault/general/service-to-service-authentication.md)


Para más información, consulte la entrada de blog sobre la [administración de secretos de manera segura en la nube](https://devblogs.microsoft.com/visualstudio/managing-secrets-securely-in-the-cloud/).

#### <a name="can-i-write-my-own-custom-searchers"></a>¿Puedo escribir mis propios buscadores personalizados?

Credential Scanner se basa en un conjunto de buscadores de contenido que se suelen definir en el archivo buildsearchers.xml. El archivo contiene una matriz de objetos serializados en XML que representan un objeto **ContentSearcher**. El programa se distribuye con un conjunto de buscadores totalmente probados. Pero también puede implementar sus propios buscadores personalizados.

Un buscador de contenido se define de la siguiente manera:

- **Name**: el nombre descriptivo del buscador que se usará en el archivo de salida de Credential Scanner. Se recomienda usar la convención de nomenclatura en mayúsculas y minúsculas para los nombres de los buscadores.
- **RuleId**: el identificador opaco estable del buscador:
    - A un buscador predeterminado de Credential Scanner se asigna le asigna un valor **RuleId** como CSCAN0010, CSCAN0020 o CSCAN0030. El último dígito se reserva para la combinación o la división posible de grupos de buscadores a través de expresiones regulares (regex).
    - El valor **RuleId** de un buscador personalizado debe tener su propio espacio de nombres. Entre los ejemplos se incluyen CSCAN-\<Namespace\>0010, CSCAN-\<Namespace\>0020 y CSCAN-\<Namespace\>0030.
    - Un nombre completo de buscador es la combinación de un valor **RuleId** y un nombre de buscador. Algunos ejemplos son CSCAN0010.KeyStoreFiles y CSCAN0020.Base64EncodedCertificate.
- **ResourceMatchPattern**: regex de las extensiones de archivo que se deben comprobar respecto del buscador.
- **ContentSearchPatterns**: matriz de cadenas que contienen las instrucciones de expresión regular que deben coincidir. Si no se han definido patrones de búsqueda, se devolverán todos los archivos que coincidan con el valor **ResourceMatchPattern**.
- **ContentSearchFilters**: matriz de cadenas que contienen instrucciones de expresión regular para filtrar falsos positivos específicos del buscador.
- **MatchDetails**: mensaje descriptivo, instrucciones de mitigación (o ambos) que se van a agregar a cada coincidencia del buscador.
- **Recomendación:** proporciona el contenido del campo de sugerencias para las coincidencias con el formato de informe PREfast.
- **Gravedad**: un entero que refleja el nivel de gravedad de un problema. El nivel de gravedad más alto tiene el valor 1.

  ![Código XML que muestra la configuración de Credential Scanner](./media/security-tools/6-credscan-customsearchers.png)

### <a name="roslyn-analyzers"></a>Analizadores basados en Roslyn

#### <a name="what-are-common-errors-when-using-the-roslyn-analyzers-task"></a>¿Cuáles son errores comunes cuando se usa la tarea de analizadores basados en Roslyn?

##### <a name="the-project-was-restored-using-a-wrong-microsoftnetcoreapp-version"></a>El proyecto se restauró con una versión incorrecta de Microsoft.NETCore.App

El mensaje de error completo:

"Error: El proyecto se restauró con la versión de Microsoft.NETCore.App *x.x.x*, pero con la configuración actual se usaría la versión *y.y.y*. Para solucionar este problema, asegúrese de que se usa la misma configuración para la restauración y para las operaciones posteriores como compilar o publicar. Normalmente, este problema puede producirse si la propiedad RuntimeIdentifier se establece durante la compilación o publicación, en lugar de durante la restauración".

Como las tareas de analizadores basados en Roslyn se ejecutan como parte de la compilación, el árbol de origen de la máquina de compilación debe estar en un estado que pueda compilarse.

Es posible que un paso entre los pasos de la compilación principal y los analizadores basados en Roslyn haya puesto el árbol de origen en un estado que impide la compilación. Este paso adicional probablemente sea **dotnet.exe publish**. Intente duplicar el paso que realiza una restauración de NuGet justo antes del paso de los analizadores basados en Roslyn. Este paso duplicado podría volver a poner el árbol de origen en un estado que permita la compilación.

##### <a name="cscexe-cant-create-an-analyzer-instance"></a>csc.exe no puede crear una instancia del analizador

El mensaje de error completo:

""csc.exe" se cerró con el código de error 1: no se puede crear una instancia del analizador *AAAA* desde C:\\*BBBB*.dll : No se puede cargar el archivo o ensamblado "Microsoft.CodeAnalysis, Version=*X.X.X.X*, Culture=neutral, PublicKeyToken=31bf3856ad364e35" o alguna de sus dependencias. El sistema no encuentra el archivo especificado."

Asegúrese de que el compilador admite los analizadores basados en Roslyn. La ejecución del comando **csc.exe /version** debe notificar un valor de versión de 2.6 o posterior.

En algunas ocasiones, un archivo .csproj individual puede invalidar la instalación de Visual Studio de la máquina de compilación al hacer referencia a un paquete de Microsoft.Net.Compilers. Si no pretende usar una versión específica del compilador, quite las referencias a Microsoft.Net.Compilers. De lo contrario, asegúrese de que la versión del paquete al que se hace referencia también sea 2.6 o posterior.

Intente obtener la ruta de acceso del registro de errores, que se especifica en la opción **csc.exe /errorlog**. La opción y la ruta de acceso aparecen en el registro de la tarea de compilación de los analizadores basados en Roslyn. Puede tener un aspecto similar a **/errorlog:F:\ts-services-123\_work\456\s\Some\Project\Code\Code.csproj.sarif**

##### <a name="the-c-compiler-version-isnt-recent-enough"></a>La versión del compilador de C# no es lo suficientemente reciente

Para obtener las versiones más recientes del compilador de C#, vaya a [Microsoft.Net.Compilers](https://www.nuget.org/packages/Microsoft.Net.Compilers). Para saber cuál es la versión instalada, ejecuta **csc.exe /version** en un símbolo del sistema. Asegúrese de hacer referencia a un paquete NuGet de Microsoft.Net.Compilers de la versión 2.6 o posterior.

##### <a name="msbuild-and-vsbuild-logs-arent-found"></a>No se encuentran los registros de MSBuild ni VSBuild

La tarea de compilación de los analizadores basados en Roslyn necesita consultar a Azure DevOps para obtener el registro de MSBuild de la tarea de compilación de MSBuild. Si la tarea del analizador se ejecuta inmediatamente después de la tarea MSBuild, el registro todavía no estará disponible. Coloque otras tareas entre la tarea MSBuild y la tarea de los analizadores basados en Roslyn. Entre los ejemplos de otras tareas se incluyen BinSkim y Anti-Malware Scanner.

## <a name="next-steps"></a>Pasos siguientes

Si necesita ayuda adicional, el soporte técnico para el Análisis de código de seguridad de Microsoft está disponible de lunes a viernes de las 9:00 a. m. a las 5:00 p. m. hora estándar del Pacífico.

- Incorporación: Consulte la [documentación de incorporación](security-code-analysis-onboard.md)
  
- Soporte técnico: envíe un correo electrónico a nuestro equipo de [soporte técnico para el Análisis de código de seguridad de Microsoft](mailto:mscahelp@microsoft.com?Subject=Microsoft%20Security%20Code%20Analysis%20Support%20Request).