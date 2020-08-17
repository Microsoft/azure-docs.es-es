---
title: 'Preparación para la producción y procedimientos recomendados: Azure'
description: En este artículo se ofrecen instrucciones sobre cómo configurar e implementar el módulo Live Video Analytics on IoT Edge en entornos de producción.
ms.topic: conceptual
ms.date: 04/27/2020
ms.openlocfilehash: c34e05e184cfa6f0933701a76177fae3eed70c0a
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87071931"
---
# <a name="production-readiness-and-best-practices"></a>Preparación para la producción y procedimientos recomendados

En este artículo se ofrecen instrucciones sobre cómo configurar e implementar el módulo Live Video Analytics on IoT Edge en entornos de producción. También recomendamos que lea el artículo [Preparación para implementar la solución IoT Edge en producción](../../iot-edge/production-checklist.md) sobre la preparación de la solución IoT Edge. 

> [!NOTE]
> Debe consultar a los departamentos de TI de las organizaciones sobre los aspectos relacionados con la seguridad.

## <a name="running-the-module-as-a-local-user"></a>Ejecución del módulo como un usuario local

Al implementar el módulo Live Video Analytics on IoT Edge en un dispositivo perimetral, se ejecuta de forma predeterminada con privilegios elevados. Al hacerlo, si comprueba los registros en el módulo (`sudo iotedge logs {name-of-module}`), verá lo siguiente:

```
!! production readiness: user accounts – Warning
       LOCAL_USER_ID and LOCAL_GROUP_ID environment variables are not set. The program will run as root!
       For optimum security, make sure to set LOCAL_USER_ID and LOCAL_GROUP_ID environment variables to a non-root user and group.
       See https://aka.ms/lva-iot-edge-prod-checklists-user-accounts for more information.
```

En las secciones siguientes se explica cómo puede resolver la advertencia anterior.

### <a name="creating-and-using-a-local-user-account"></a>Creación y uso de una cuenta de usuario local

Puede y debe ejecutar el módulo Live Video Analytics on IoT Edge en producción con una cuenta que tenga el menor número de privilegios posible. Por ejemplo, los siguientes comandos muestran cómo se puede crear una cuenta de usuario local en una máquina virtual de Linux:

```
sudo groupadd -g 1010 localuser
sudo adduser --home /home/edgeuser --uid 1010 -gid 1010 edgeuser
```

Después, en el manifiesto de implementación, puede establecer las variables de entorno LOCAL_USER_ID y LOCAL_GROUP_ID en ese grupo y usuario no raíz:

```
"lvaEdge": {
"version": "1.0",
…
"env": {
    "LOCAL_USER_ID": 
    {
        "value": "1010"
    },
    "LOCAL_GROUP_ID": {
        "value": "1010"
    }
}
},
…
```

### <a name="granting-permissions-to-device-storage"></a>Concesión de permisos para el almacenamiento de dispositivos

Para el módulo Live Video Analytics on IoT Edge se necesita la capacidad de escribir archivos en el sistema de archivos local al:

* Usar una propiedad del módulo gemelo [[applicationDataDirectory](module-twin-configuration-schema.md#module-twin-properties)], donde debe especificar un directorio en el sistema de archivos local para almacenar los datos de configuración.
* Usar un gráfico multimedia para grabar vídeo en la nube; en este caso, el módulo necesita el uso de un directorio en el dispositivo perimetral como memoria caché. Vea el artículo [Grabación de vídeo continua](continuous-video-recording-concept.md) para más información.
* [Grabar en un archivo local](event-based-video-recording-concept.md#video-recording-based-on-events-from-other-sources), donde debe especificar una ruta de acceso de archivo para el vídeo grabado.

Si tiene previsto hacer uso de cualquiera de los anteriores, debe asegurarse de que la cuenta de usuario anterior tiene acceso al directorio pertinente. Por ejemplo, tenga en cuenta applicationDataDirectory. Puede crear un directorio en el dispositivo perimetral y vincular el almacenamiento del dispositivo al almacenamiento de módulos. 

```
sudo mkdir /var/local/mediaservices
sudo chown -R edgeuser /var/local/mediaservices
```

Después, en las opciones de creación para el módulo perimetral del manifiesto de implementación, puede agregar una configuración de enlace que asigne el directorio (“var/local/mediaservices/”) anterior a un directorio del módulo (por ejemplo, “/var/lib/azuremediaservices/”). Y usaría el último directorio como el valor de applicationDataDirectory.

```
"lvaEdge": {
    "version": "1.0",
    "type": "docker",
    "status": "running",
    "restartPolicy": "always",
    "settings": {
        "image": "mcr.microsoft.com/media/live-video-analytics:1.0",
        "createOptions": "{\"HostConfig\":{\"LogConfig\":{\"Type\":\"\",\"Config\":{\"max-size\":\"10m\",\"max-file\":\"10\"}},\"Binds\":[\"/var/local/mediaservices/:/var/lib/azuremediaservices/\"]}}"
    },
    "env": {
        "LOCAL_USER_ID": 
        {
            "value": "1010"
        },
        "LOCAL_GROUP_ID": {
            "value": "1010"
        }
    }
    },
    …
    
    "lvaEdge": {
    "properties.desired": {
    "applicationDataDirectory": "/var/lib/azuremediaservices",
    …
    }
}
```

Si observa los gráficos multimedia de ejemplo de la guía de inicio rápido y los tutoriales, como [Grabación de vídeo continua](continuous-video-recording-tutorial.md), observará que el directorio de caché de medios (localMediaCachePath) usa un subdirectorio en applicationDataDirectory. Este es el enfoque recomendado, ya que la memoria caché contiene datos transitorios.

### <a name="naming-video-assets-or-files"></a>Nomenclatura de archivos o recursos de vídeo

Los gráficos multimedia permiten crear recursos en la nube o archivos mp4 en el perímetro. Los recursos multimedia pueden generarse mediante la [grabación de vídeo continua](continuous-video-recording-tutorial.md) o la [grabación de vídeo basada en eventos](event-based-video-recording-tutorial.md). Aunque estos recursos y archivos pueden tener el nombre que quiera, la estructura de nomenclatura recomendada para el recurso multimedia basado en la grabación de vídeo continua es "&lt;anytext&gt;-${System.GraphTopologyName}-${System.GraphInstanceName}". Por ejemplo, puede establecer assetNamePattern en el receptor de recursos como se indica aquí:

```
"assetNamePattern": "sampleAsset-${System.GraphTopologyName}-${System.GraphInstanceName}
```

En el caso de los recursos generados mediante grabación de vídeo basada en eventos, el patrón de nomenclatura recomendado es "&lt;anytext&gt;-${System.DateTime}". La variable del sistema garantiza que no se sobrescriban los recursos si se producen eventos al mismo tiempo. Por ejemplo, puede establecer assetNamePattern en el receptor de recursos como se indica aquí:

```
"assetNamePattern": "sampleAssetFromEVR-LVAEdge-${System.DateTime}"
```

Si está ejecutando varias instancias del mismo gráfico, puede usar el nombre de la topología de gráfico y el nombre de instancia para diferenciarlos. Por ejemplo, puede establecer assetNamePattern en el receptor de recursos como se indica aquí:

```
"assetNamePattern": "sampleAssetFromEVR-${System.GraphTopologyName}-${System.GraphInstanceName} -${System.DateTime}"
```

En el caso de los clips de vídeo mp4 generados por grabación de vídeo basada en eventos, el patrón de nomenclatura recomendado debe incluir DateTime y, para varias instancias del mismo gráfico, se recomienda usar las variables del sistema GraphTopologyName y GraphInstanceName. Por ejemplo, puede establecer filePathPattern en el receptor de archivos como se indica aquí: 

```
"filePathPattern": "/var/media/sampleFilesFromEVR-${fileSinkOutputName}-${System.DateTime}"
```

Or 

```
"filePathPattern": "/var/media/sampleFilesFromEVR-${fileSinkOutputName}--${System.GraphTopologyName}-${System.GraphInstanceName} ${System.DateTime}"
```

### <a name="keeping-your-vm-clean"></a>Mantenimiento de la limpieza de la máquina virtual

La máquina virtual de Linux que se usa como dispositivo perimetral puede dejar de responder si no se administra de forma periódica. Es fundamental mantener limpias las memorias caché, eliminar los paquetes innecesarios y quitar también los contenedores no usados de la máquina virtual. Para hacerlo, recomendamos este conjunto de comandos que puede usar en la máquina virtual perimetral.

1. `sudo apt-get clean`

    El comando apt-get clean borra el repositorio local de los archivos de paquete recuperados que se dejan en /var/cache. Los directorios que limpia son /var/cache/apt/archives/ y /var/cache/apt/archives/partial/. Los únicos archivos que se conservan en /var/cache/apt/archives son el archivo de bloqueo y el subdirectorio parcial. El comando apt-get clean se usa generalmente para borrar el espacio en disco cuando sea necesario, normalmente como parte del mantenimiento programado regularmente. Para más información, vea [Cleaning up with apt-get](https://www.networkworld.com/article/3453032/cleaning-up-with-apt-get.html) (Limpieza con apt-get).
1. `sudo apt-get autoclean`

    La opción apt-get autoclean, como apt-get clean, borra el repositorio local de archivos de paquete recuperados, pero solo quita los archivos que ya no se pueden descargar y que no son útiles. Esto ayuda a evitar que la memoria caché crezca demasiado.
1. `sudo apt-get autoremove1`

    La opción de quitar automáticamente quita los paquetes que se instalaron automáticamente porque algún otro paquete los necesitaba pero, con esos otros paquetes quitados, ya no son necesarios.
1. `sudo docker image ls`: proporciona una lista de imágenes de Docker en el sistema perimetral
1. `sudo docker system prune `

    Docker toma un enfoque conservador para limpiar los objetos no usados (a menudo denominados "recolección de elementos no utilizados"), como imágenes, contenedores, volúmenes y redes: normalmente, estos objetos no se quitan a menos que se pida explícitamente a Docker que lo haga. Esto puede hacer que Docker use espacio en disco adicional. Para cada tipo de objeto, Docker proporciona un comando de eliminación. Además, puede usar la eliminación del sistema de Docker para limpiar varios tipos de objetos a la vez. Para más información, vea [Prune unused Docker objects](https://docs.docker.com/config/pruning/) (Eliminar los objetos de Docker no usados).
1. `sudo docker rmi REPOSITORY:TAG`

    A medida que las actualizaciones se producen en el módulo Edge, el Docker puede tener versiones anteriores del módulo Edge todavía presentes. En tal caso, se recomienda usar el comando docker rmi para quitar imágenes específicas identificadas por la etiqueta de versión de la imagen.

## <a name="next-steps"></a>Pasos siguientes

[Inicio rápido: Introducción: Live Video Analytics on IoT Edge](get-started-detect-motion-emit-events-quickstart.md)
