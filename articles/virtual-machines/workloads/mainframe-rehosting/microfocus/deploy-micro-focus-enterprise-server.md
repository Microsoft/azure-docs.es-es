---
title: Implementación de Micro Focus Enterprise Server 5.0 en AKS | Microsoft Docs
description: Rehospede las cargas de trabajo del sistema central IBM z/OS con el entorno de desarrollo y pruebas Micro Focus en Azure Virtual Machines (VM).
services: virtual-machines-linux
documentationcenter: ''
author: maggsl
ms.author: edprice
manager: edprice
editor: edprice
ms.topic: conceptual
ms.date: 06/29/2020
tags: ''
keywords: ''
ms.service: multiple
ms.openlocfilehash: 6780942d922f885c7afebd8e64f4f28654c3800e
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87042535"
---
# <a name="deploy-micro-focus-enterprise-server-50-to-aks"></a>Implementación de Micro Focus Enterprise Server 5.0 en AKS

En otro [artículo](./run-enterprise-server-container.md), se describen los pasos para ejecutar Micro Focus Enterprise Server 5.0 en un contenedor de Docker. A modo de seguimiento, quiero mostrar cómo ir algo más lejos e implementar la imagen de Docker que creó en Azure Kubernetes Service (AKS).

Azure Kubernetes Service es un servicio de orquestación administrado que se basa en Kubernetes. Permite implementar, escalar y administrar contenedores de Docker (y otras aplicaciones basadas en contenedores) en un clúster de hosts de contenedor.

Se trata de un proceso de tres pasos. Necesita:

1.  Crear una instancia de Azure Container Registry para almacenar las imágenes de Docker.

2.  Crear un clúster de Azure Kubernetes para ejecutar la imagen de Docker.

3.  Ejecute la aplicación.

Esto le permite escalar horizontalmente (y reducir verticalmente) las cargas de trabajo de modernización de grandes sistemas en Azure, lo que aprovecha la verdadera ventaja de la plataforma en la nube.

¿Listo? Comencemos.

## <a name="create-the-azure-container-registry"></a>Creación de la instancia de Azure Container Registry

En Azure Portal, seleccione **Crear un recurso** en la esquina superior izquierda. En el panel de Marketplace, seleccione **Containers** y luego **Container Registry**. Esto le llevará al panel **Crear Registro de contenedor**, donde debe rellenar los campos **Nombre del Registro**, **Suscripción de Azure**, **Grupo de recursos** y **Ubicación**. El **nombre del Registro** debe resolverse, por lo que ha de ser único. Seleccione el **grupo de recursos** que usó en la entrada de blog anterior y la misma **ubicación** correspondiente. Seleccione **Habilitar** para **Usuario administrador** y **Básico** para la **SKU**. Cuando lo haya rellenado todo, seleccione **Crear**.

![Creación de una interfaz Registro de contenedor](media/deploy-image-1.png)

Cuando haya implementado el registro, seleccione **Ir al recurso**. Esto le llevará a la hoja principal de la instancia de Azure Container Registry. Una buena característica aquí es la opción de menú **Inicio rápido**. Selecciónela y verá instrucciones sobre lo que hay que hacer para insertar y extraer imágenes en el registro. Vamos a seguir estos pasos:

1.  **Instalar Docker**: no tiene que preocuparse de ello porque ya está listo.

2.  **Ejecutar la imagen base "hello – world"** : de nuevo, no es necesario, pero no dude en probarlo.

3.  **Iniciar sesión en el registro de contenedor**: debe hacerlo desde la máquina virtual (VM). Copie el comando en el Portapapeles o el Bloc de notas.

    En el caso del registro que se creó, el comando es: **docker login acrmf50.azurecr.io**

4.  **Insertar en el Registro**: también debe hacerlo en la imagen de Micro Focus cuando haya iniciado sesión en la máquina virtual.

5.  **Extraer del Registro**: no es importante para este tutorial, pero conviene saberlo en caso de que tenga que ejecutar otra imagen de Docker.

Antes de salir del portal, ha de obtener las credenciales del registro para que pueda iniciar sesión. Salga de la hoja **Inicio rápido** y seleccione **Claves de acceso** en el menú Registro. Copie el **nombre de usuario** y una de las **contraseñas** (hay dos) en el Portapapeles y el Bloc de notas. Las necesitará más adelante para iniciar sesión.

Ahora que sabe lo que hay que hacer, inicie sesión en la máquina virtual.

## <a name="rdp-to-the-virtual-machine-you-used-to-create-the-docker-image"></a>RDP en la máquina virtual que usó para crear la imagen de Docker

Dado que ya ha creado la imagen de Docker en un servidor Windows Server 2016, debe iniciar sesión en esa máquina virtual. Desde esta máquina virtual, puede insertar la imagen en la instancia de Azure Container Registry que acaba de crear. Vaya a la máquina virtual en Azure Portal, seleccione **Información general** y luego **Conectar**. Así se conecta a la máquina virtual a través del Protocolo de escritorio remoto (RDP). Debe usar las credenciales con las que creó la máquina virtual.

## <a name="log-in-and-push-the-image-to-the-registry"></a>Inicio de sesión e inserción de la imagen en el registro

Cuando haya iniciado sesión, abra un símbolo del sistema e inicie los siguientes comandos de Docker:

-   **docker images**: muestra una lista de todas las imágenes actualmente instaladas en la máquina virtual. Tome nota de la imagen **microfocus/es-acctdemo** porque esa es con la que va a trabajar.

-   **docker login acrmf50.azurecr.io**: el formato correcto en este caso es *docker login \<registry name\>* . Sustituya el nombre que utilizó al crear el registro.

    -   Necesitará el **nombre de usuario** y la **contraseña** que copió de Azure Portal. Debe ver algo parecido a la imagen siguiente.

    ![Captura de pantalla de un símbolo del sistema de administrador](media/deploy-image-2.png)

-   **docker tag microfocus/es-acctdemo acrmf50.azurecr.io/es-acctdemo**: esto etiqueta la imagen adecuada con el nombre del repositorio. **NOTA**: Si el nombre \<microfocus/es-acctdemo\> no funciona, pruebe a usar el identificador de imagen completo. Después de ejecutar el comando, escriba **docker images –no-trunc**. Debe ver algo parecido a la imagen siguiente. Observe que la imagen está etiquetada correctamente.

    ![Selección de la pantalla Símbolo del sistema de administrador](media/deploy-image-3.png)

-   **docker push acrmf50.azurecr.io/es-acctdemo**: inicia la extracción real en el repositorio. Dado que el tamaño es de 15 GB, tarda un par de minutos en ejecutarse. Si todo va bien, debería ver algo parecido a la siguiente imagen.

    ![Pantalla Símbolo del sistema de administrador](media/deploy-image-4.png)

Ahora vuelva al Azure Portal, específicamente al **Repositorio**. En el menú del **Repositorio**, seleccione **Repositorios** y debería ver que se muestra **es-acctdemo**. Cree ahora el clúster de AKS.

## <a name="create-the-azure-kubernetes-aks-cluster"></a>Creación del clúster de Azure Kubernetes Service (AKS)

En Azure Portal, seleccione **Crear un recurso** y luego **Containers/Kubernetes Service** en el menú de **Marketplace**. A continuación, debe rellenar la hoja **Crear un clúster de Kubernetes**. Asegúrese de mantener el clúster en la misma región y grupo de recursos que ha estado usando. Puede aceptar el resto de los valores predeterminados excepto el correspondiente a **Número de nodos**, que solo debe ser 1. Cuando haya terminado, seleccione **Revisar y crear**.

![Pantalla Crear un clúster de Kubernetes](media/deploy-image-5.png)

Cuando haya finalizado, la implementación colocará los artefactos de **Kubernetes Service** en el **grupo de recursos** seleccionado en la hoja. Aunque el clúster real tendrá su propio grupo de recursos que se creó durante la implementación. Si selecciona **Grupos de recursos** en el menú de la izquierda, puede buscarlo en función de la convención de nomenclatura. Aquí aparece una imagen mía: es la última de la lista.

![Captura de pantalla de Grupos de recursos](media/deploy-image-6.png)

**Ejecución de la imagen**

Ahora es el momento de extraer la imagen y ejecutarla en AKS. La forma más fácil de hacerlo desde Azure Portal es usar el servicio Cloud Shell. Encontrará el icono en la parte superior derecha de Azure Portal. Tenga en cuenta que en este tutorial se usa el shell de Bash.

![Captura de pantalla del icono de Cloud Shell](media/deploy-image-7.png)

Una vez cargado el shell, escriba el siguiente comando:

**kubectl run es-acctdemo --image acrmf50.azurecr.io/es-acctdemo --port=9040**

Este comando extrae la imagen del repositorio **acrmf50.azurecr.io** y la carga en AKS. Entonces ejecuta la imagen con el puerto 9040 abierto. Quizá recuerde que este era el puerto que tenía abierto para la imagen de Docker. Lo necesita para acceder a Enterprise Server.

Kubernetes debe responder con un mensaje que indique que se ha creado la implementación.

![Captura de pantalla de un mensaje de implementación](media/deploy-image-8.jpg)

Para ver si el contenedor está realmente en ejecución, escriba: **kubectl get pods**.

Debería ver es-acctdemo como pod en ejecución, tal y como se ve en la imagen siguiente.

![Captura de pantalla de es-acctdemo como pod en ejecución](media/deploy-image-9.png)

Felicidades. Ahora está ejecutando Enterprise Server en Azure Kubernetes Service. En el siguiente artículo, le mostraré cómo acceder a la consola de administración de Enterprise Server y también cómo aprovechar Kubernetes para escalar horizontalmente la implementación.
