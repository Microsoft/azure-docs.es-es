---
title: Creación de artefactos personalizados para la máquina virtual de DevTest Labs | Microsoft Docs
description: Obtenga información sobre cómo crear artefactos para usarlos con Azure DevTest Labs, a fin de implementar y configurar aplicaciones después de aprovisionar una máquina virtual.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 85acfcc3811e671e58fadab08a23951778e1323d
ms.sourcegitcommit: 2bab7c1cd1792ec389a488c6190e4d90f8ca503b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/17/2020
ms.locfileid: "88270689"
---
# <a name="create-custom-artifacts-for-your-devtest-labs-virtual-machine"></a>Creación de artefactos personalizados para la máquina virtual de DevTest Labs

Vea el siguiente vídeo para obtener información general de los pasos descritos en este artículo:

> [!VIDEO https://channel9.msdn.com/Blogs/Azure/how-to-author-custom-artifacts/player]
>
>

## <a name="overview"></a>Información general
Puede usar *artefactos* para implementar y configurar la aplicación después de aprovisionar una máquina virtual. Un artefacto consta de un archivo de definición de artefacto y de otros archivos de script que se almacenan en una carpeta de un repositorio de Git. Los archivos de definición de artefacto constan de JSON y expresiones que puede utilizar para especificar lo que desea instalar en una máquina virtual. Por ejemplo, puede definir el nombre de un artefacto, un comando que se va a ejecutar y los parámetros que están disponibles cuando se ejecuta el comando. Puede hacer referencia a otros archivos de script en el archivo de definición de artefacto por nombre.

## <a name="artifact-definition-file-format"></a>Formato del archivo de definición de artefacto
En el ejemplo siguiente se muestran las secciones que componen la estructura básica de un archivo de definición:

```json
  {
    "$schema": "https://raw.githubusercontent.com/Azure/azure-devtestlab/master/schemas/2016-11-28/dtlArtifacts.json",
    "title": "",
    "description": "",
    "iconUri": "",
    "targetOsType": "",
    "parameters": {
      "<parameterName>": {
        "type": "",
        "displayName": "",
        "description": ""
      }
    },
    "runCommand": {
      "commandToExecute": ""
    }
  }
```

| Nombre del elemento | ¿Necesario? | Descripción |
| --- | --- | --- |
| $schema |No |Ubicación del archivo de esquema JSON. El archivo de esquema JSON puede ayudarle a probar la validez del archivo de definición. |
| title |Sí |Nombre del artefacto que se muestra en el laboratorio. |
| description |Sí |Descripción del artefacto que se muestra en el laboratorio. |
| iconUri |No |Identificador URI del icono que se muestra en el laboratorio. |
| targetOsType |Sí |Sistema operativo de la máquina virtual en que se instala el artefacto. Las opciones admitidas son Windows y Linux. |
| parámetros |No |Los valores que se proporcionan cuando el comando de instalación del artefacto se ejecuta en un equipo. Esto le ayuda a personalizar el artefacto. |
| runCommand |Sí |Comando de instalación de artefacto que se ejecuta en una máquina virtual. |

### <a name="artifact-parameters"></a>Parámetros de artefacto
En la sección de parámetros del archivo de definición, especifique los valores que un usuario puede indicar al instalar un artefacto. Puede hacer referencia a estos valores en el comando de instalación del artefacto.

Para definir los parámetros, use la estructura siguiente:

```json
  "parameters": {
    "<parameterName>": {
      "type": "<type-of-parameter-value>",
      "displayName": "<display-name-of-parameter>",
      "description": "<description-of-parameter>"
    }
  }
```

| Nombre del elemento | ¿Necesario? | Descripción |
| --- | --- | --- |
| type |Sí |Tipo del valor de parámetro. Consulte la lista siguiente de los tipos permitidos. |
| DisplayName |Sí |Nombre del parámetro que se muestra a un usuario en el laboratorio. |
| description |Sí |Descripción del parámetro que se muestra en el laboratorio. |

Los tipos permitidos son:

* string (cualquier cadena JSON válida)
* int (cualquier entero JSON válido)
* bool (cualquier booleano JSON válido)
* array (cualquier matriz JSON válida)

## <a name="secrets-as-secure-strings"></a>Secretos como cadenas seguras
Declare los secretos como cadenas seguras. Esta es la sintaxis para declarar un parámetro de cadena segura dentro de la sección `parameters` del archivo **artifactfile.json**:

```json

    "securestringParam": {
      "type": "securestring",
      "displayName": "Secure String Parameter",
      "description": "Any text string is allowed, including spaces, and will be presented in UI as masked characters.",
      "allowEmpty": false
    },
```

Para el comando de instalación del artefacto, ejecute el script de PowerShell que toma la cadena segura creado mediante el comando ConvertTo-SecureString. 

```json
  "runCommand": {
    "commandToExecute": "[concat('powershell.exe -ExecutionPolicy bypass \"& ./artifact.ps1 -StringParam ''', parameters('stringParam'), ''' -SecureStringParam (ConvertTo-SecureString ''', parameters('securestringParam'), ''' -AsPlainText -Force) -IntParam ', parameters('intParam'), ' -BoolParam:$', parameters('boolParam'), ' -FileContentsParam ''', parameters('fileContentsParam'), ''' -ExtraLogLines ', parameters('extraLogLines'), ' -ForceFail:$', parameters('forceFail'), '\"')]"
  }
```

Para consultar los archivos artifactfile.json y artifact.ps1 del ejemplo completo (script de PowerShell), consulte [este ejemplo en GitHub](https://github.com/Azure/azure-devtestlab/tree/master/Artifacts/windows-test-paramtypes).

Otro punto importante que tener en cuenta es no registrar los secretos en la consola a medida que la salida se capture para la depuración de los usuarios. 

## <a name="artifact-expressions-and-functions"></a>Expresiones y funciones de artefacto
Puede utilizar expresiones y funciones para construir el comando de instalación del artefacto.
Las expresiones se incluyen entre corchetes ([ y ]) y se evalúan cuando se instala el artefacto. Las expresiones pueden aparecer en cualquier lugar en un valor de cadena JSON. Las expresiones siempre devuelven otro valor JSON. Si necesita usar una cadena literal que comienza por un corchete ([), debe usar dos corchetes ([[).
Normalmente, se utilizan expresiones con funciones para construir un valor. **Al igual que en JavaScript, las llamadas de función tienen el formato functionName(arg1, arg2, arg3)** .

En la lista siguiente se muestran las funciones comunes:

* **parameters(parameterName)** : devuelve un valor de parámetro que se proporciona cuando se ejecuta el comando de artefacto.
* **concat(arg1, arg2, arg3,…)** : combina varios valores de cadena. Esta función puede tomar diversos argumentos.

En el ejemplo siguiente se muestra cómo utilizar expresiones y funciones para construir un valor:

```json
  runCommand": {
      "commandToExecute": "[concat('powershell.exe -ExecutionPolicy bypass \"& ./startChocolatey.ps1'
  , ' -RawPackagesList ', parameters('packages')
  , ' -Username ', parameters('installUsername')
  , ' -Password ', parameters('installPassword'))]"
  }
```

## <a name="create-a-custom-artifact"></a>Creación de un artefacto personalizado

1. Instalación de un editor de JSON. Necesita un editor de JSON para trabajar con los archivos de definición del artefacto. Se recomienda usar [Visual Studio Code](https://code.visualstudio.com/), que está disponible para Windows, Linux y OS X.
2. Obtenga un archivo de definición artifactfile.json de ejemplo. Desproteja los artefactos creados por el equipo de DevTest Labs en nuestro [repositorio de GitHub](https://github.com/Azure/azure-devtestlab). Hemos creado una biblioteca enriquecida de artefactos que pueden ayudarle a crear sus propios artefactos. Descargue un archivo de definición de artefacto y haga cambios sobre él para crear sus propios artefactos.
3. Use IntelliSense. Use IntelliSense para ver los elementos válidos que se pueden utilizar para construir un archivo de definición de artefacto. También puede ver las distintas opciones para los valores de un elemento. Por ejemplo, al editar el elemento **targetOsType**, IntelliSense le muestra las dos opciones, para Windows o Linux.
4. Guarde el artefacto en el [repositorio público de Git para DevTest Labs](https://github.com/Azure/azure-devtestlab/tree/master/Artifacts) o en [su propio repositorio de Git](devtest-lab-add-artifact-repo.md). En el repositorio público, puede visualizar los artefactos compartidos por otros usuarios que puede utilizar directamente o personalizar para adaptarlos a sus necesidades.
   
   1. Cree un directorio independiente para cada artefacto. El nombre del directorio debe ser el mismo que el del artefacto.
   2. Almacene el archivo de definición de artefacto (artifactfile.json) en el directorio que ha creado.
   3. Almacene los scripts a los que hace referencia el comando de instalación del artefacto.
      
      Este es un ejemplo del aspecto que tendrá una carpeta de artefacto:
      
      ![Ejemplo de la carpeta de artefacto](./media/devtest-lab-artifact-author/git-repo.png)
5. Si está utilizando su propio repositorio para almacenar artefactos, agregue el repositorio al laboratorio siguiendo las instrucciones del artículo: [Adición de un repositorio de Git de artefactos y plantillas](devtest-lab-add-artifact-repo.md).

## <a name="related-articles"></a>Artículos relacionados
* [How to diagnose artifact failures in DevTest Labs](devtest-lab-troubleshoot-artifact-failure.md) (Diagnóstico de errores de artefactos en DevTest Labs)
* [Join a VM to existing AD Domain using a resource manager template in DevTest Labs](https://www.visualstudiogeeks.com/blog/DevOps/Join-a-VM-to-existing-AD-domain-using-ARM-template-AzureDevTestLabs) (Unión de una máquina virtual al dominio de AD existente mediante la plantilla de Resource Manager en DevTest Labs)

## <a name="next-steps"></a>Pasos siguientes
* Aprenda cómo [agregar un repositorio de artefactos Git a un laboratorio](devtest-lab-add-artifact-repo.md).
