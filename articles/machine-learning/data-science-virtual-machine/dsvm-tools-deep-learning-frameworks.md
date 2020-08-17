---
title: Marcos de aprendizaje profundo y AI
titleSuffix: Azure Data Science Virtual Machine
description: Disponibilidad de los marcos de aprendizaje profundo y las herramientas en Azure Data Science Virtual Machine.
keywords: herramientas de ciencia de datos, máquina virtual de ciencia de datos, herramientas para la ciencia de datos, ciencia de datos de linux
services: machine-learning
ms.service: machine-learning
ms.subservice: data-science-vm
ms.custom: devx-track-python
author: lobrien
ms.author: laobri
ms.topic: conceptual
ms.date: 12/12/2019
ms.openlocfilehash: 360899f7efc8850b336f9716b4fd8c89fefe277a
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87846673"
---
# <a name="deep-learning-and-ai-frameworks-for-the-azure-data-science-vm"></a>Marcos de aprendizaje profundo e inteligencia artificial para Azure Data Science VM
La lista a continuación enumera los marcos de aprendizaje profundo en DSVM.

## <a name="caffe"></a>[Caffe](https://github.com/BVLC/caffe)

| Category | Value |
| ------------- | ------------- |
| Versiones compatibles | |
| Ediciones de DSVM admitidas      | Linux (Ubuntu)     |
| ¿Cómo se configura/instala en DSVM?  | Caffe se instala en `/opt/caffe`.   Se pueden encontrar ejemplos en `/opt/caffe/examples`.|
| Cómo ejecutarlo      | use X2Go para iniciar sesión en la máquina virtual y, a continuación, inicie un nuevo terminal y escriba lo siguiente:<br/>`cd /opt/caffe/examples`<br/>`source activate root`<br/>`jupyter notebook`<br/><br/>Se abre una nueva ventana del explorador con blocs de notas de ejemplo. Los binarios se instalan en /opt/caffe/build/install/bin.<br/><br/>La versión instalada de Caffe requiere Python 2.7 y no funcionará con Python 3.5, que está activado de forma predeterminada. Para cambiar a Python 2.7, ejecute `source activate root` para cambiar al entorno de Anaconda.|    

## <a name="caffe2"></a>[Caffe2](https://github.com/caffe2/caffe2)

| Category | Value |
| ------------- | ------------- |
| Versiones compatibles | |
| Ediciones de DSVM admitidas      | Linux (Ubuntu)     |
| ¿Cómo se configura/instala en DSVM?  | Caffe2 se instala en el entorno de Conda de Python 2.7 (root). |
| Cómo ejecutarlo      | Terminal: Inicie Python e importe Caffe2. <br/> * JupyterHub: [conéctese a JupyterHub](dsvm-ubuntu-intro.md#how-to-access-the-ubuntu-data-science-virtual-machine) y desplácese al directorio Caffe2 para buscar blocs de notas de ejemplo. Algunos blocs de notas requieren que se establezca la raíz de Caffe2 en el código de Python. Escriba /opt/caffe2. |

## <a name="chainer"></a>[Chainer](https://chainer.org/)

| Category | Value |
| ------------- | ------------- |
| Versiones compatibles | 5.2 |
| Ediciones de DSVM admitidas      | Linux (Ubuntu)     |
| ¿Cómo se configura/instala en DSVM?  | Chainer se instala en Python 3.5. |
| Cómo ejecutarlo      | Terminal: active el entorno de Python 3.5, ejecute `python` y luego `import chainer`. <br/> * JupyterHub: [conéctese a JupyterHub](dsvm-ubuntu-intro.md#how-to-access-the-ubuntu-data-science-virtual-machine) y desplácese al directorio Chainer para buscar cuadernos de ejemplo.| 

## <a name="cuda-cudnn-nvidia-driver"></a>[CUDA, cuDNN, NVIDIA Driver](https://developer.nvidia.com/cuda-toolkit)

| Category | Value |
| ------------- | ------------- |
| Versiones compatibles | 10.0.130|
| Ediciones de DSVM admitidas      | Windows y Linux   |
| ¿Cómo se configura/instala en DSVM?  |_nvidia-smi_ está disponible en la ruta de acceso del sistema.  |
| Cómo ejecutarlo      | Abra un símbolo del sistema (en Windows) o un terminal (en Linux) y ejecute _nvidia-smi_. |


## <a name="horovod"></a>[Horovod](https://github.com/uber/horovod)

| Category | Value |
| ------------- | ------------- |
| Versiones compatibles | 0.16.1|
| Ediciones de DSVM admitidas      | Linux (Ubuntu)   |
| ¿Cómo se configura/instala en DSVM?  | Horovod se instala en Python 3.5 |
| Cómo ejecutarlo      | Active el entorno correcto en el terminal y, luego, ejecute Python. |

## <a name="keras"></a>[Keras](https://keras.io/)

| Category | Value |
| ------------- | ------------- |
| Versiones compatibles | 2.2.4 |
| Ediciones de DSVM admitidas      | Windows y Linux   |
| ¿Cómo se configura/instala en DSVM?  | Keras se instala en Python 3.6 en Windows, y en Python 3.5 en Linux |
| Cómo ejecutarlo      | Active el entorno correcto en el terminal y, luego, ejecute Python. |

## <a name="microsoft-cognitive-toolkit-cntk"></a>[Microsoft Cognitive Toolkit (CNTK)](https://docs.microsoft.com/cognitive-toolkit/)

| Category | Value |
| ------------- | ------------- |
| Versiones compatibles | 2.5.1 |
| Ediciones de DSVM admitidas      | Windows y Linux   |
| ¿Cómo se configura/instala en DSVM?  | CNTK se instala en Python 3.6 en [Windows 2016](dsvm-tools-languages.md#python-windows-server-2016-edition), y en Python 3.5 en [Linux](./dsvm-tools-languages.md#python-linux-edition) |
| Cómo ejecutarlo      | Terminal: active el entorno correcto y ejecute Python. <br/>Jupyter: conéctese a [Jupyter](provision-vm.md) o [JupyterHub](dsvm-ubuntu-intro.md#how-to-access-the-ubuntu-data-science-virtual-machine) y abra el directorio CNTK para acceder a los ejemplos. |

## <a name="mxnet"></a>[MXNet](https://mxnet.apache.org/)
| Category | Value |
| ------------- | ------------- |
| Versiones compatibles | 1.3.0 |
| Ediciones de DSVM admitidas      | Windows y Linux   |
| ¿Cómo se configura/instala en DSVM?  | MXNet se instala en `C:\dsvm\tools\mxnet` en Windows y en `/dsvm/tools/mxnet` en Ubuntu. Los enlaces de Python se instalan en Python 3.6 en [Windows 2016](dsvm-tools-languages.md#python-windows-server-2016-edition) y en Python 3.5 en [Linux](./dsvm-tools-languages.md#python-linux-edition)). Los enlaces de R también se incluyen en Ubuntu DSVM. |
| Cómo ejecutarlo      | Terminal: active el entorno conda correcto y ejecute `import mxnet`. <br/>Jupyter: conéctese a [Jupyter](provision-vm.md#access-the-dsvm) o [JupyterHub](dsvm-ubuntu-intro.md#how-to-access-the-ubuntu-data-science-virtual-machine) y abra el directorio `mxnet` para acceder a los ejemplos. |

## <a name="mxnet-model-server"></a>[Servidor de modelo de MXNet](https://github.com/awslabs/mxnet-model-server#quick-start)

| Category | Value |
| ------------- | ------------- |
| Versiones compatibles | 1.0.1 |
| Ediciones de DSVM admitidas      | Windows y Linux   |
| ¿Cómo se configura/instala en DSVM?  | MXNet Model Server se instala en Python 3.6 en [Windows 2016](dsvm-tools-languages.md#python-windows-server-2016-edition), y en Python 3.5 en [Linux](./dsvm-tools-languages.md#python-linux-edition)) |
| Cómo ejecutarlo      | Terminal: Ejecute `sudo systemctl stop jupyterhub` para detener el servicio JupyterHub en primer lugar, ya que ambos escuchan en el mismo puerto. Después, active el entorno conda correcto y ejecute `mxnet-model-server --start --models squeezenet=https://s3.amazonaws.com/model-server/model_archive_1.0/squeezenet_v1.1.mar`. |

## <a name="nvidia-system-management-interface-nvidia-smi"></a>[NVidia System Management Interface (nvidia-smi)](https://developer.nvidia.com/nvidia-system-management-interface)

| Category | Value |
| ------------- | ------------- |
| Versiones compatibles |  |
| Ediciones de DSVM admitidas      | Windows y Linux   |
| ¿Para qué sirven? | Herramienta NVIDIA para consultar la actividad GPU |
| ¿Cómo se configura/instala en DSVM?  | `nvidia-smi` está en la ruta de acceso del sistema. |
| Cómo ejecutarlo      | En una máquina virtual **con GPU**, abra un símbolo del sistema (en Windows) o un terminal (en Linux), y ejecute `nvidia-smi`. |

## <a name="pytorch"></a>[PyTorch](https://pytorch.org/)

| Category | Value |
| ------------- | ------------- |
| Versiones compatibles | 1.2.0 (Ubuntu 16.04, Windows 2016), 1.4.0 (Ubuntu 18.04, (Windows 2019). |
| Ediciones de DSVM admitidas      | Linux |
| ¿Cómo se configura/instala en DSVM?  | Se instala en [Python 3.5](dsvm-tools-languages.md#python-linux-edition). Se incluyen cuadernos de Jupyter de ejemplo, así como ejemplos en /dsvm/samples/pytorch. |
| Cómo ejecutarlo      | Terminal: active el entorno correcto y ejecute Python.<br/>* [JupyterHub](dsvm-ubuntu-intro.md#how-to-access-the-ubuntu-data-science-virtual-machine): conéctese y abra el directorio de PyTorch para obtener ejemplos.  |

## <a name="tensorflow"></a>[TensorFlow](https://www.tensorflow.org/)

| Category | Value |
| ------------- | ------------- |
| Versiones compatibles | 1.13 |
| Ediciones de DSVM admitidas      | Windows, Linux |
| ¿Cómo se configura/instala en DSVM?  | Se instala en Python 3.5 en [Linux](dsvm-tools-languages.md#python-linux-edition) y en Python 3.6 en [Windows 2016](dsvm-tools-languages.md#python-windows-server-2016-edition) |
| Cómo ejecutarlo      | Terminal: active el entorno correcto y ejecute Python. <br/> *Jupyter: conéctese a [Jupyter](provision-vm.md) o [JupyterHub](dsvm-ubuntu-intro.md#how-to-access-the-ubuntu-data-science-virtual-machine) y abra el directorio TensorFlow para acceder a los ejemplos.   |

## <a name="tensorflow-serving"></a>[TensorFlow Serving](https://www.tensorflow.org/serving/)

| Category | Value |
| ------------- | ------------- |
| Versiones compatibles | 1.12 |
| Ediciones de DSVM admitidas      | Linux |
| ¿Cómo se configura/instala en DSVM?  | tensorflow_model_server está disponible en el terminal. |
| Cómo ejecutarlo      |  Otros ejemplos están disponibles [en línea](https://www.tensorflow.org/serving/).   |


## <a name="theano"></a>[Theano](https://github.com/Theano/Theano)

| Category | Value |
| ------------- | ------------- |
| Versiones compatibles | 1.0.3 |
| Ediciones de DSVM admitidas      | Linux |
| ¿Cómo se configura/instala en DSVM?  |Theano se instala en el entorno de Python 2.7 (_root_), así como en el entorno de Python 3.5 (_py35_). |
| Cómo ejecutarlo      |  Terminal: active la versión de Python que quiera (root o py35), ejecute Python e importe Theano.<br/>*Jupyter: seleccione el kernel de Python 2.7 o 3.5 e importe Theano.  <br/>Para solucionar un error reciente de Math Kernel Library (MKL), primero debe establecer la capa de subprocesos de MKL de la siguiente manera:<br/><br/>`export MKL_THREADING_LAYER=GNU`  |
