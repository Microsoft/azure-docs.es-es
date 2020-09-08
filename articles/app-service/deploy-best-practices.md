---
title: Procedimientos recomendados de implementación
description: Obtenga información sobre los mecanismos clave de la implementación en Azure App Service. Busque recomendaciones específicas del lenguaje y otras advertencias.
keywords: azure app service, web app, deploy, deployment, pipelines, build
author: jasonfreeberg
ms.assetid: bb51e565-e462-4c60-929a-2ff90121f41d
ms.topic: article
ms.date: 07/31/2019
ms.author: jafreebe
ms.openlocfilehash: b4581b7e93cde9d6ba9a20d46ee263a879c05402
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88961880"
---
# <a name="deployment-best-practices"></a>Procedimientos recomendados de implementación

Cada equipo de desarrollo tiene requisitos únicos que pueden dificultar la implementación de una canalización de implementación eficaz en cualquier servicio en la nube. En este artículo se presentan los tres componentes principales de la implementación de App Service: orígenes de implementación, canalizaciones de compilación y mecanismos de implementación. En este artículo también se tratan algunos procedimientos recomendados y sugerencias para pilas de idiomas específicos.

## <a name="deployment-components"></a>Componentes de implementación

### <a name="deployment-source"></a>Origen de implementación

Un origen de implementación es la ubicación del código de la aplicación. En el caso de las aplicaciones de producción, el origen de implementación suele ser un repositorio hospedado por software de control de versiones como [GitHub, BitBucket o Azure Repos](deploy-continuous-deployment.md). En escenarios de desarrollo y prueba, el origen de implementación puede ser [un proyecto en la máquina local](deploy-local-git.md). App Service también admite las [ carpetas OneDrive y Dropbox](deploy-content-sync.md) como orígenes de implementación. A pesar de que las carpetas en la nube pueden facilitar la introducción a App Service, normalmente no se recomienda usar este origen para las aplicaciones de producción de nivel empresarial. 

### <a name="build-pipeline"></a>Canalización de compilación

Una vez que decida un origen de implementación, el paso siguiente consistirá en elegir una canalización de compilación. Una canalización de compilación lee el código fuente del origen de implementación y ejecuta una serie de pasos (por ejemplo, compilar el código, compactar el código HTML y JavaScript, ejecutar pruebas o empaquetar componentes) para hacer que la aplicación pueda ejecutarse. Los comandos específicos que la canalización de compilación ejecuta dependen de la pila de lenguajes. Estas operaciones se pueden ejecutar en un servidor de compilación como Azure Pipelines o de forma local.

### <a name="deployment-mechanism"></a>Mecanismo de implementación

El mecanismo de implementación es la acción que se usa para colocar la aplicación compilada en el directorio */home/site/wwwroot* de la aplicación Web. El directorio */wwwroot* es una ubicación de almacenamiento montada que todas las instancias de la aplicación Web comparten. Cuando el mecanismo de implementación coloca la aplicación en este directorio, las instancias reciben una notificación para sincronizar los nuevos archivos. App Service admite los siguientes mecanismos de implementación:

- Puntos de conexión de Kudu: [Kudu](https://github.com/projectkudu/kudu/wiki) es la herramienta de productividad para desarrolladores de código abierto que se ejecuta como un proceso independiente en Windows App Service y como un segundo contenedor en Linux App Service. Kudu controla las implementaciones continuas y proporciona puntos de conexión HTTP para la implementación, como zipdeploy.
- FTP y WebDeploy: Con las [credenciales de usuario o de sitio](deploy-configure-credentials.md), puede cargar archivos [a través de FTP ](deploy-ftp.md) o WebDeploy. Estos mecanismos no pasan por Kudu.  

Las herramientas de implementación como Azure Pipelines, Jenkins y los complementos de editor usan uno de estos mecanismos de implementación.

## <a name="use-deployment-slots"></a>Uso de ranuras de implementación

Siempre que sea posible, use [ranuras de implementación](deploy-staging-slots.md) al implementar una nueva compilación de producción. Al usar un nivel de Plan de App Service estándar o superior, puede implementar la aplicación en un entorno de ensayo, validar los cambios y realizar pruebas de humo. Cuando esté preparado, puede intercambiar las ranuras de ensayo y de producción. La operación de intercambio prepara las instancias de trabajo necesarias para que coincidan con la escala de producción, lo que elimina el tiempo de inactividad.

### <a name="continuously-deploy-code"></a>Código de implementación continua

Si el proyecto tiene ramas designadas para pruebas, control de calidad y ensayo, cada una de esas ramas debe implementarse continuamente en un espacio de ensayo. (Esto se conoce como [diseño Gitflow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow)). De esta manera, las partes interesadas pueden evaluar y probar fácilmente la rama implementada. 

La implementación continua nunca debe estar habilitada para el espacio de producción. Al contrario, la rama de producción (a menudo la maestra) debe implementarse en un espacio que no sea de producción. Cuando esté listo para liberar la rama base, cámbiela al espacio de producción. El cambio a producción, en lugar de la implementación en producción, evita tiempos de inactividad y permite revertir los cambios intercambiando de nuevo. 

![Objeto visual de uso de espacio](media/app-service-deploy-best-practices/slot_flow_code_diagam.png)

### <a name="continuously-deploy-containers"></a>Contenedores de implementación continua

Para contenedores personalizados de Docker u otros registros de contenedor, implemente la imagen en un espacio de ensayo y cámbielo a producción para evitar tiempos de inactividad. La automatización es más compleja que la implementación de código porque debe introducir la imagen en un registro de contenedor y actualizar la etiqueta de imagen en la aplicación web.

Para cada rama que quiera implementar en un espacio, configure la automatización para que haga lo siguiente en cada confirmación en la rama.

1. **Compilar y etiquetar la imagen**. Como parte de la canalización de compilación, etiquete la imagen con el identificador de confirmación de Git, la marca de tiempo u otra información de identificación. Es mejor no usar la etiqueta predeterminada "latest". De lo contrario, es difícil realizar un seguimiento del código que se implementa actualmente, lo que dificulta mucho más la depuración.
1. **Insertar la imagen etiquetada**. Una vez que la imagen se compila y etiqueta, la canalización la envía a nuestro registro de contenedor. En el paso siguiente, el espacio de implementación extraerá la imagen etiquetada del registro de contenedor.
1. **Actualizar el espacio de implementación con la nueva etiqueta de imagen**. Cuando se actualice esta propiedad, el sitio se reiniciará automáticamente y se extraerá la nueva imagen de contenedor.

![Objeto visual de uso de espacio](media/app-service-deploy-best-practices/slot_flow_container_diagram.png)

A continuación se muestran ejemplos de marcos de automatización comunes.

### <a name="use-azure-devops"></a>Uso de Azure DevOps

App Service dispone de [entrega continua integrada](deploy-continuous-deployment.md) para los contenedores mediante el Centro de implementación. Vaya a la aplicación en [Azure Portal](https://portal.azure.com/) y seleccione **Deployment Center** (Centro de implementación) en **Implementación**. Siga las instrucciones para seleccionar el repositorio y la rama. De esta manera se configurará una canalización de versión y compilación de DevOps para compilar, etiquetar e implementar automáticamente el contenedor cuando se inserten nuevas confirmaciones en la rama seleccionada.

### <a name="use-github-actions"></a>Uso de acciones de GitHub

También puede automatizar la implementación del contenedor [con acciones de GitHub](deploy-container-github-action.md).  El archivo de flujo de trabajo siguiente compilará y etiquetará el contenedor con el identificador de confirmación, lo insertará en un registro de contenedor y actualizará el espacio del sitio con la nueva etiqueta de imagen.

```yaml
name: Build and deploy a container image to Azure Web Apps

on:
  push:
    branches:
    - <your-branch-name>

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@master

    -name: Authenticate using a Service Principal
      uses: azure/actions/login@v1
      with:
        creds: ${{ secrets.AZURE_SP }}

    - uses: azure/container-actions/docker-login@v1
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}

    - name: Build and push the image tagged with the git commit hash
      run: |
        docker build . -t contoso/demo:${{ github.sha }}
        docker push contoso/demo:${{ github.sha }}

    - name: Update image tag on the Azure Web App
      uses: azure/webapps-container-deploy@v1
      with:
        app-name: '<your-webapp-name>'
        slot-name: '<your-slot-name>'
        images: 'contoso/demo:${{ github.sha }}'
```

### <a name="use-other-automation-providers"></a>Uso de otros proveedores de automatización

Los pasos enumerados anteriormente se aplican a otras utilidades de automatización como CircleCI o Travis CI. Sin embargo, debe usar la CLI de Azure para actualizar los espacios de implementación con nuevas etiquetas de imagen en el paso final. Para usar la CLI de Azure en el script de automatización, genere una entidad de servicio con el siguiente comando.

```shell
az ad sp create-for-rbac --name "myServicePrincipal" --role contributor \
   --scopes /subscriptions/{subscription}/resourceGroups/{resource-group} \
   --sdk-auth
```

En el script, inicie sesión con `az login --service-principal` y proporcione la información de la entidad de seguridad. Después, puede usar `az webapp config container set` para establecer el nombre del contenedor, la etiqueta, la dirección URL del registro y la contraseña del registro. A continuación se muestran algunos vínculos útiles para crear el proceso de CI del contenedor.

- [Inicio de sesión en la CLI de Azure en Circle CI](https://circleci.com/orbs/registry/orb/circleci/azure-cli) 

## <a name="language-specific-considerations"></a>Consideraciones específicas del idioma

### <a name="java"></a>Java

Use la API[ zipdeploy/](deploy-zip.md) de Kudu para implementar las aplicaciones JAR y [wardeploy/](deploy-zip.md#deploy-war-file) para las aplicaciones WAR. Si usa Jenkins, puede usar esas API directamente en la fase de implementación. Para obtener más información, consulte [este artículo](/azure/developer/jenkins/deploy-to-azure-app-service-using-azure-cli).

### <a name="node"></a>Nodo

De forma predeterminada, Kudu ejecuta los pasos de compilación para la aplicación Node (`npm install`). Si usa un servicio de compilación como Azure DevOps, la compilación de Kudu es innecesaria. Para deshabilitarla, cree una opción de aplicación, `SCM_DO_BUILD_DURING_DEPLOYMENT`, con el valor `false`.

### <a name="net"></a>.NET 

De forma predeterminada, Kudu ejecuta los pasos de compilación para la aplicación .NET (`dotnet build`). Si usa un servicio de compilación como Azure DevOps, la compilación de Kudu es innecesaria. Para deshabilitarla, cree una opción de aplicación, `SCM_DO_BUILD_DURING_DEPLOYMENT`, con el valor `false`.

## <a name="other-deployment-considerations"></a>Otras consideraciones de implementación

### <a name="local-cache"></a>caché local

El contenido de Azure App Service se almacena en Azure Storage y surge de manera duradera como un recurso compartido de contenido. Sin embargo, algunas aplicaciones solo necesitan un almacén de contenido de solo lectura y de gran rendimiento que puedan ejecutar con alta disponibilidad. Estas aplicaciones pueden beneficiarse del uso de la [memoria caché local](overview-local-cache.md). No se recomienda la memoria caché local para los sitios de administración de contenido como WordPress.

Use siempre la memoria caché local junto con [ranuras de implementación](deploy-staging-slots.md) para evitar tiempos de inactividad. Consulte [esta sección](overview-local-cache.md#best-practices-for-using-app-service-local-cache) para información sobre el uso conjunto de estas características.

### <a name="high-cpu-or-memory"></a>CPU o memoria elevadas

Si el Plan de App Service usa más del 90 % de la CPU o la memoria disponibles, es posible que la máquina virtual subyacente tenga problemas para procesar la implementación. Cuando esto suceda, escale de forma temporal el número de instancias para realizar la implementación. Una vez finalizada la implementación, puede devolver el número de instancias a su valor anterior.

Para obtener más información sobre los procedimientos recomendados, visite [Diagnósticos de App Service](./overview-diagnostics.md) para obtener información sobre los procedimientos recomendados viables específicos para el recurso.

- Vaya a la aplicación web en [Azure Portal](https://portal.azure.com).
- En el panel izquierdo, haga clic en **Diagnosticar y solucionar problemas** para abrir la página Diagnósticos de App Service.
- Elija el icono **Procedimientos recomendados** de la página principal.
- Haga clic en **Best Practices for Availability & Performance** (Prácticas recomendadas de disponibilidad y rendimiento) o en **Best Practices for Optimal Configuration** (Procedimientos recomendados para la configuración óptima) para ver el estado actual de la aplicación en lo que respecta a estas prácticas recomendadas.

También puede usar este vínculo para abrir directamente Diagnósticos de App Service para el recurso: `https://ms.portal.azure.com/?websitesextension_ext=asd.featurePath%3Ddetectors%2FParentAvailabilityAndPerformance#@microsoft.onmicrosoft.com/resource/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Web/sites/{siteName}/troubleshoot`.