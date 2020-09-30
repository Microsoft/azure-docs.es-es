---
title: 'Tutorial: Uso de R para crear un modelo de Machine Learning (versión preliminar)'
titleSuffix: Azure Machine Learning
description: En este tutorial usará el SDK de R de Azure Machine Learning para crear un modelo de regresión logística que prediga la probabilidad de que se produzca una muerte en un accidente de coche.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
ms.reviewer: sgilley
author: revodavid
ms.author: davidsmi
ms.date: 02/07/2020
ms.openlocfilehash: faf2257f944127748569466d314ff0a76faae4f0
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90906731"
---
# <a name="tutorial-use-r-to-create-a-machine-learning-model-preview"></a>Tutorial: Uso de R para crear un modelo de Machine Learning (versión preliminar)


> [!IMPORTANT]
> El SDK de Azure Machine Learning para R se encuentra actualmente en versión preliminar pública.
> Se ofrece la versión preliminar sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

En este tutorial usará el SDK para R de Azure Machine Learning (versión preliminar) para crear un modelo de regresión logística que prediga la probabilidad de que se produzca una muerte en un accidente de coche. Verá cómo funcionan los recursos en la nube de Azure Machine Learning con R para proporcionar un entorno escalable para entrenar e implementar un modelo.  

En este tutorial se realizan las siguientes tareas:
> [!div class="checklist"]
> * Creación de un área de trabajo de Azure Machine Learning
> * Apertura de RStudio desde el área de trabajo
> * Clonación desde https://github.com/Azure/azureml-sdk-for-r con los archivos necesarios para ejecutar este tutorial en el área de trabajo
> * Carga de datos y preparación para el entrenamiento
> * Carga de datos en un almacén de datos para que estén disponibles para el entrenamiento remoto
> * Creación de un recurso de proceso para entrenar el modelo de manera remota
> * Entrenamiento de un modelo `caret` para predecir la probabilidad de muerte
> * Implementación de un punto de conexión de predicción
> * Prueba del modelo desde R

Si no tiene una suscripción de Azure, cree una cuenta gratuita antes de empezar. Pruebe hoy mismo la [versión gratuita o de pago de Azure Machine Learning](https://aka.ms/AMLFree).


## <a name="create-a-workspace"></a>Crear un área de trabajo

Un área de trabajo de Azure Machine Learning es un recurso básico de la nube que se usa para experimentar, entrenar e implementar modelos de aprendizaje automático. Vincula la suscripción y el grupo de recursos de Azure con un objeto fácilmente consumido del servicio. 

Puede crear un área de trabajo mediante Azure Portal, una consola basada en web para administrar los recursos de Azure. 

[!INCLUDE [aml-create-portal](../../includes/aml-create-in-portal.md)]

>[!IMPORTANT] 
> Tome nota del **área de trabajo** y de la **suscripción**. Los necesitará para asegurarse de que crea el experimento en el lugar correcto. 


## <a name="open-rstudio"></a><a name="open"></a>Apertura de RStudio

En este ejemplo se usa una instancia de proceso en el área de trabajo para obtener una experiencia sin instalación y configurada previamente. Si prefiere tener control sobre su entorno, los paquetes y las dependencias de la máquina, use [su propio entorno](https://azure.github.io/azureml-sdk-for-r/articles/installation.html).

Use RStudio en una instancia de proceso de Azure ML para llevar a cabo este tutorial.  

1. Seleccione **Proceso** a la izquierda.

1. Agregue un recurso de proceso si aún no existe uno.

1. Cuando el proceso esté en ejecución, use el vínculo **RStudio** para abrir RStudio.


## <a name="clone-the-sample-vignettes"></a><a name="azure"></a>Clonación de las viñetas de ejemplo 

Clone el repositorio de GitHub https://github.com/azure/azureml-sdk-for-r para obtener una copia de los archivos de cada viñeta que se ejecutarán en este tutorial.

1. En RStudio, vaya a la pestaña "Terminal" y use el comando cd para colocarse en el directorio en el que quiere clonar el repositorio.

1. Ejecute "git clone https://github.com/Azure/azureml-sdk-for-r" en el terminal para clonar el repositorio.

1. En RStudio, vaya a la carpeta *vignettes* de la carpeta clonada *azureml-sdk-for-r*.  En *vignettes*, seleccione el archivo *train-and-deploy-first-model.rmd* para buscar la viñeta que se usa en este tutorial. Los archivos adicionales que se usan para la viñeta se encuentran en la subcarpeta *train-and-deploy-first-model*. Una vez que haya abierto la viñeta, establezca el directorio de trabajo en la ubicación del archivo mediante **Sesión > Establecer el directorio de trabajo en la ubicación del archivo de origen**. 

> [!Important]
> El resto de este artículo incluye el mismo contenido que el del archivo *train-and-deploy-first-model.rmd*. Si tiene experiencia con RMarkdown, no dude en usar el código de ese archivo.  También puede copiar y pegar los fragmentos de código desde allí o desde este artículo en un script de R o en la línea de comandos. 


## <a name="set-up-your-development-environment"></a>Configurado su entorno de desarrollo
La configuración del trabajo de desarrollo en este tutorial incluye las siguientes acciones:

* Instalación de los paquetes requeridos
* Conexión a un área de trabajo para que la instancia de proceso pueda comunicarse con los recursos remotos
* Creación de un experimento para realizar un seguimiento de las ejecuciones
* Creación de un destino de proceso remoto que se usará para el entrenamiento

### <a name="install-required-packages"></a>Instalación de los paquetes requeridos
La instancia de proceso ya tiene instalada la versión más reciente del SDK de R de CRAN. Si desea instalar la versión de desarrollo de GitHub en su lugar para elegir las correcciones de errores más recientes, ejecute lo siguiente:
    
```R
remotes::install_github('https://github.com/Azure/azureml-sdk-for-r')
azuremlsdk::install_azureml()
```

> [!WARNING]
> Durante el proceso de instalación, si recibe el mensaje "`Would you like to install Miniconda? [Y/n]:`", responda con "`n`" dado que la instancia de proceso tiene ya instalado Anaconda y no se necesita una instalación de Miniconda.

Continúe e importe el paquete **azuremlsdk**.

```R
library(azuremlsdk)
```

Los scripts de entrenamiento y puntuación (`accidents.R` y `accident_predict.R`) tienen algunas dependencias adicionales. Si piensa ejecutar esos scripts localmente, asegúrese de que también tiene los paquetes necesarios.

### <a name="load-your-workspace"></a>Carga del área de trabajo
Cree una instancia de objeto de área de trabajo desde el área de trabajo existente. El código siguiente cargará los detalles del área de trabajo desde el archivo **config.json**. También puede recuperar un área de trabajo mediante [`get_workspace()`](https://azure.github.io/azureml-sdk-for-r/reference/get_workspace.html).

```R
ws <- load_workspace_from_config()
```

### <a name="create-an-experiment"></a>Creación de un experimento
Un experimento de Azure Machine Learning realiza un seguimiento de una agrupación de ejecuciones, normalmente desde el mismo script de entrenamiento. Cree un experimento para realizar el seguimiento de las ejecuciones para entrenar el modelo de caret en los datos de accidentes.

```R
experiment_name <- "accident-logreg"
exp <- experiment(ws, experiment_name)
```

### <a name="create-a-compute-target"></a>Creación de un destino de proceso
Al usar Azure Machine Learning Compute (AmlCompute), un servicio administrado, los científicos de datos pueden entrenar modelos de aprendizaje automático en clústeres de máquinas virtuales de Azure, entre las que se incluyen las que tienen compatibilidad con GPU. En este tutorial se crea un clúster de AmlCompute de un solo nodo como entorno de entrenamiento. El código siguiente crea el clúster de proceso automáticamente si no existe aún en el área de trabajo.

Es posible que tenga que esperar unos minutos para que se aprovisione el clúster de proceso si aún no existe.

```R
cluster_name <- "rcluster"
compute_target <- get_compute(ws, cluster_name = cluster_name)
if (is.null(compute_target)) {
  vm_size <- "STANDARD_D2_V2" 
  compute_target <- create_aml_compute(workspace = ws,
                                       cluster_name = cluster_name,
                                       vm_size = vm_size,
                                       max_nodes = 1)
}

wait_for_provisioning_completion(compute_target)
```

## <a name="prepare-data-for-training"></a>Preparación de los datos para el entrenamiento
En este tutorial se usan datos de [National Highway Traffic Safety Administration](https://cdan.nhtsa.gov/tsftables/tsfar.htm) de EE. UU. (gracias a [Mary C. Meyer y Tremika Finney](https://www.stat.colostate.edu/~meyer/airbags.htm)).
Este conjunto de datos incluye datos de más de 25 000 accidentes de coche en EE. UU., con variables que puede usar para predecir la probabilidad de muerte. En primer lugar, importe los datos en R, transfórmelos en una nueva trama de datos `accidents` para su análisis y expórtelos a un archivo `Rdata`.

```R
nassCDS <- read.csv("nassCDS.csv", 
                     colClasses=c("factor","numeric","factor",
                                  "factor","factor","numeric",
                                  "factor","numeric","numeric",
                                  "numeric","character","character",
                                  "numeric","numeric","character"))
accidents <- na.omit(nassCDS[,c("dead","dvcat","seatbelt","frontal","sex","ageOFocc","yearVeh","airbag","occRole")])
accidents$frontal <- factor(accidents$frontal, labels=c("notfrontal","frontal"))
accidents$occRole <- factor(accidents$occRole)
accidents$dvcat <- ordered(accidents$dvcat, 
                          levels=c("1-9km/h","10-24","25-39","40-54","55+"))

saveRDS(accidents, file="accidents.Rd")
```

### <a name="upload-data-to-the-datastore"></a>Carga datos en el almacén de datos
Cargue los datos en la nube para que el entorno de entrenamiento remoto pueda acceder a ellos. Cada área de trabajo de Azure Machine Learning incluye un almacén de datos predeterminado que guarda la información de conexión al contenedor de blobs de Azure que está aprovisionado en la cuenta de almacenamiento asociada al área de trabajo. El código siguiente cargará los datos de los accidentes creados anteriormente en ese almacén de datos.

```R
ds <- get_default_datastore(ws)

target_path <- "accidentdata"
upload_files_to_datastore(ds,
                          list("./accidents.Rd"),
                          target_path = target_path,
                          overwrite = TRUE)
```


## <a name="train-a-model"></a>Entrenamiento de un modelo

En este tutorial se ajusta un modelo de regresión logística a los datos cargados mediante el clúster de proceso remoto. Para enviar un trabajo, deberá:

* Preparar el script de entrenamiento
* Crear un estimador
* Enviar el archivo

### <a name="prepare-the-training-script"></a>Preparar el script de entrenamiento
Se incluyó un script de entrenamiento llamado `accidents.R` en el directorio *train-and-deploy-first-model*. **Dentro del script de entrenamiento**, tenga en cuenta los siguientes detalles que se han creado para aprovechar las ventajas de Azure Machine Learning para el entrenamiento:

* El script de entrenamiento toma un argumento `-d` para buscar el directorio que contiene los datos de entrenamiento. Al definir y enviar el trabajo más adelante, apunte al almacén de datos de este argumento. Azure Machine Learning montará la carpeta de almacenamiento en el clúster remoto para el trabajo de entrenamiento.
* El script de entrenamiento registra la precisión final como métrica del registro de ejecución en Azure Machine Learning mediante `log_metric_to_run()`. El SDK de Azure Machine Learning proporciona un conjunto de API de registro para el registro de varias métricas durante las ejecuciones de entrenamiento. Estas métricas se registran y se conservan en el registro de ejecución del experimento. A continuación, se puede acceder a las métricas en cualquier momento o visualizarlas en la página de detalles de la ejecución de [Studio](https://ml.azure.com). Consulte la [referencia](https://azure.github.io/azureml-sdk-for-r/reference/index.html#section-training-experimentation) para ver el conjunto completo de métodos de registro `log_*()`.
* El script de entrenamiento guarda el modelo en un directorio denominado **outputs**. La carpeta `./outputs` recibe un tratamiento especial por parte de Azure Machine Learning. Durante el entrenamiento, los archivos escritos en `./outputs` se cargan automáticamente en el registro de ejecución mediante Azure Machine Learning y se conservan como artefactos. Al guardar el modelo entrenado en `./outputs`, podrá acceder al archivo del modelo y recuperarlo incluso después de que la ejecución haya finalizado y ya no tenga acceso a su entorno de entrenamiento remoto.

### <a name="create-an-estimator"></a>Crear un estimador

Un estimador de Azure Machine Learning encapsula la información de configuración de ejecución necesaria para ejecutar un script de entrenamiento en el destino de proceso. Las ejecuciones de Azure Machine Learning se ejecutan como trabajos en contenedores en el destino de proceso especificado. De forma predeterminada, la imagen de Docker compilada para el trabajo de entrenamiento incluirá R, el SDK de Azure Machine Learning y un conjunto de paquetes de R que se usan con frecuencia. Consulte la lista completa de los paquetes predeterminados que se incluye aquí.

Para crear el estimador, defina:

* El directorio que contiene los scripts necesarios para el entrenamiento (`source_directory`). Todos los archivos de este directorio se cargan en los nodos del clúster para su ejecución. El directorio debe contener el script de entrenamiento y los scripts adicionales necesarios.
* El script de entrenamiento que se ejecutará (`entry_script`).
* El destino de proceso (`compute_target`); en este caso, el clúster de AmlCompute que creó anteriormente.
* Los parámetros necesarios del script de entrenamiento (`script_params`). Azure Machine Learning ejecutará el script de entrenamiento como script de línea de comandos con `Rscript`. En este tutorial se especifica un argumento para el script, el punto de montaje del directorio de datos, al que se puede acceder con `ds$path(target_path)`.
* Todas las dependencias de entorno necesarias para el entrenamiento. La imagen de Docker predeterminada compilada para el entrenamiento ya contiene los tres paquetes (`caret`, `e1071` y `optparse`) necesarios en el script de entrenamiento.  Por lo tanto, no es necesario especificar ninguna información adicional. Si usa paquetes de R que no están incluidos de forma predeterminada, use el parámetro `cran_packages` del estimador para agregar paquetes de CRAN adicionales. Consulte la referencia de [`estimator()`](https://azure.github.io/azureml-sdk-for-r/reference/estimator.html) para ver el conjunto completo de opciones configurables.

```R
est <- estimator(source_directory = "train-and-deploy-first-model",
                 entry_script = "accidents.R",
                 script_params = list("--data_folder" = ds$path(target_path)),
                 compute_target = compute_target
                 )
```

### <a name="submit-the-job-on-the-remote-cluster"></a>Envío del trabajo al clúster remoto

Por último, envíe el trabajo para que se ejecute en el clúster. `submit_experiment()` devuelve un objeto Run que se utiliza después para interactuar con la ejecución. En total, la primera ejecución tarda **aproximadamente 10 minutos**. Pero para las ejecuciones posteriores, se reutiliza la misma imagen de Docker, siempre y cuando no cambien las dependencias del script.  En este caso, la imagen se almacena en caché y el tiempo de inicio del contenedor es mucho más rápido.

```R
run <- submit_experiment(exp, est)
```

Puede ver los detalles de la ejecución en el visor de RStudio. Al hacer clic en el vínculo "Vista web", se le llevará a Azure Machine Learning Studio, donde podrá supervisar la ejecución en la interfaz de usuario.

```R
view_run_details(run)
```

El entrenamiento del modelo se produce en segundo plano. Espere a que el modelo haya completado el entrenamiento para poder ejecutar más código.

```R
wait_for_run_completion(run, show_output = TRUE)
```

Todos los usuarios que tengan acceso al área de trabajo pueden enviar varios experimentos en paralelo y Azure Machine Learning tendrá que programar las tareas en el clúster de proceso. Puede incluso configurar el clúster para escalar verticalmente de forma automática a varios nodos y volver a reducir cuando no haya más tareas de proceso en la cola. Esta configuración resulta rentable para que los equipos compartan recursos de proceso.

## <a name="retrieve-training-results"></a>Recuperación de los resultados del entrenamiento
Una vez finalizado el entrenamiento del modelo, puede acceder a los artefactos del trabajo que se guardaron en el registro de ejecución, incluidas las métricas registradas y el modelo entrenado final.

### <a name="get-the-logged-metrics"></a>Obtención de las métricas registradas
En el script de entrenamiento `accidents.R`, ha registrado una métrica del modelo: la precisión de las predicciones en los datos de entrenamiento. Puede ver las métricas en [Studio](https://ml.azure.com) o extraerlas a la sesión local como lista de R tal y como se indica a continuación:

```R
metrics <- get_run_metrics(run)
metrics
```

Si ha ejecutado varios experimentos (por ejemplo, con diferentes variables, algoritmos o hiperparámetros), puede usar las métricas de cada ejecución para comparar y elegir el modelo que usará en producción.

### <a name="get-the-trained-model"></a>Obtención del modelo entrenado
Puede recuperar el modelo entrenado y ver los resultados en la sesión local de R. En el código siguiente se descargará el contenido del directorio `./outputs`, que incluye el archivo del modelo.

```R
download_files_from_run(run, prefix="outputs/")
accident_model <- readRDS("outputs/model.rds")
summary(accident_model)
```

Verá algunos factores que contribuyen a un aumento de la probabilidad estimada de muerte:

* mayor velocidad de impacto 
* hombre al volante
* ocupante de mayor edad
* pasajero

Verá las menores probabilidades de muerte con:

* presencia de airbags
* presencia de cinturones de seguridad
* colisión frontal 

El año de fabricación del vehículo no tiene un efecto significativo.

Puede usar este modelo para realizar nuevas predicciones:

```R
newdata <- data.frame( # valid values shown below
 dvcat="10-24",        # "1-9km/h" "10-24"   "25-39"   "40-54"   "55+"  
 seatbelt="none",      # "none"   "belted"  
 frontal="frontal",    # "notfrontal" "frontal"
 sex="f",              # "f" "m"
 ageOFocc=16,          # age in years, 16-97
 yearVeh=2002,         # year of vehicle, 1955-2003
 airbag="none",        # "none"   "airbag"   
 occRole="pass"        # "driver" "pass"
 )

## predicted probability of death for these variables, as a percentage
as.numeric(predict(accident_model,newdata, type="response")*100)
```

## <a name="deploy-as-a-web-service"></a>Implementación como servicio web

Con el modelo puede predecir el riesgo de muerte en una colisión. Use Azure Machine Learning para implementar el modelo como servicio de predicción. Ahora ya está listo para implementar el servicio web en [Azure Container Instances](https://docs.microsoft.com/azure/container-instances/) (ACI).

### <a name="register-the-model"></a>Registro del modelo

En primer lugar, registre el modelo que descargó en el área de trabajo con [`register_model()`](https://azure.github.io/azureml-sdk-for-r/reference/register_model.html). Un modelo registrado puede ser cualquier colección de archivos, pero para este caso el objeto del modelo de R es suficiente. Azure Machine Learning usará el modelo registrado para la implementación.

```R
model <- register_model(ws, 
                        model_path = "outputs/model.rds", 
                        model_name = "accidents_model",
                        description = "Predict probablity of auto accident")
```

### <a name="define-the-inference-dependencies"></a>Definición de las dependencias de la inferencia
Para crear un servicio web para el modelo, primero debe crear un script de puntuación (`entry_script`), un script de R que tomará valores como variables de entrada (en formato JSON) y generará una predicción del modelo. En este tutorial se usa el archivo de puntuación proporcionado `accident_predict.R`. El script de puntuación debe contener un método `init()` que cargue el modelo y devuelva una función que use el modelo para realizar una predicción basada en los datos de entrada. Consulte la [documentación](https://azure.github.io/azureml-sdk-for-r/reference/inference_config.html#details) para más detalles.

A continuación, defina un **entorno** de Azure Machine Learning para las dependencias de paquete del script. Con un entorno, se especifican los paquetes de R (desde CRAN o cualquier otro lugar) necesarios para que se ejecute el script. También puede proporcionar los valores de las variables de entorno a las que puede hacer referencia el script para modificar su comportamiento. De forma predeterminada, Azure Machine Learning compilará la misma imagen de Docker predeterminada que se usa con el estimador para el entrenamiento. Dado que el tutorial no tiene ningún requisito especial, cree un entorno sin atributos especiales.

```R
r_env <- r_environment(name = "basic_env")
```

De lo contrario, si desea usar su propia imagen de Docker para la implementación, especifique el parámetro `custom_docker_image`. Consulte la referencia de [`r_environment()`](https://azure.github.io/azureml-sdk-for-r/reference/r_environment.html) para ver el conjunto completo de opciones configurables para definir un entorno.

Ahora tiene todo lo que necesita para crear un documento de **configuración de inferencias** para encapsular el script de puntuación y las dependencias del entorno.

```R
inference_config <- inference_config(
  entry_script = "accident_predict.R",
  source_directory = "train-and-deploy-first-model",
  environment = r_env)
```

### <a name="deploy-to-aci"></a>Implementación en ACI
En este tutorial implementará el servicio en ACI. Este código aprovisiona un único contenedor para responder a las solicitudes entrantes, lo que es adecuado para pruebas y cargas ligeras. Consulte [`aci_webservice_deployment_config()`](https://azure.github.io/azureml-sdk-for-r/reference/aci_webservice_deployment_config.html) para ver más opciones configurables. (Para implementaciones en producción, también puede [implementar en Azure Kubernetes Service](https://azure.github.io/azureml-sdk-for-r/articles/deploy-to-aks.html)).

``` R
aci_config <- aci_webservice_deployment_config(cpu_cores = 1, memory_gb = 0.5)
```

Ahora, implemente el modelo como servicio web. La implementación **puede tardar varios minutos**. 

```R
aci_service <- deploy_model(ws, 
                            'accident-pred', 
                            list(model), 
                            inference_config, 
                            aci_config)

wait_for_deployment(aci_service, show_output = TRUE)
```

## <a name="test-the-deployed-service"></a>Prueba del modelo implementado

Ahora que el modelo se ha implementado como servicio, puede probarlo desde R mediante [`invoke_webservice()`](https://azure.github.io/azureml-sdk-for-r/reference/invoke_webservice.html).  Proporcione un nuevo conjunto de datos a partir del cual se harán las predicciones, conviértalo al formato JSON y envíelo al servicio.

```R
library(jsonlite)

newdata <- data.frame( # valid values shown below
 dvcat="10-24",        # "1-9km/h" "10-24"   "25-39"   "40-54"   "55+"  
 seatbelt="none",      # "none"   "belted"  
 frontal="frontal",    # "notfrontal" "frontal"
 sex="f",              # "f" "m"
 ageOFocc=22,          # age in years, 16-97
 yearVeh=2002,         # year of vehicle, 1955-2003
 airbag="none",        # "none"   "airbag"   
 occRole="pass"        # "driver" "pass"
 )

prob <- invoke_webservice(aci_service, toJSON(newdata))
prob
```

También puede obtener el punto de conexión HTTP del servicio web, que acepta llamadas de cliente REST. Puede compartir este punto de conexión con todo aquel que desee probar el servicio web, o bien lo puede integrar en una aplicación.

```R
aci_service$scoring_uri
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Elimine los recursos cuando ya no los necesite. No elimine ningún recurso que tenga previsto seguir usando. 

Elimine el servicio web:
```R
delete_webservice(aci_service)
```

Elimine el modelo registrado:
```R
delete_model(model)
```

Elimine el clúster de proceso:
```R
delete_compute(compute)
```

### <a name="delete-everything"></a>Eliminar todo el contenido

[!INCLUDE [aml-delete-resource-group](../../includes/aml-delete-resource-group.md)]

También puede mantener el grupo de recursos pero eliminar una sola área de trabajo. Muestre las propiedades del área de trabajo y seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

* Ahora que ha completado el primer experimento de Azure Machine Learning en R, aprenda más acerca del [SDK de Azure Machine Learning para R](https://azure.github.io/azureml-sdk-for-r/index.html).

* Más información sobre Azure Machine Learning con R en los ejemplos de las demás carpetas *vignettes*.
