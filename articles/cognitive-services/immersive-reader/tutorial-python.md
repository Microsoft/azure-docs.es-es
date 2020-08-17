---
title: 'Tutorial: Inicio del Lector inmersivo mediante Python'
titleSuffix: Azure Cognitive Services
description: En este tutorial creará una aplicación de Python que inicie el Lector inmersivo.
services: cognitive-services
author: dylankil
manager: guillasi
ms.service: cognitive-services
ms.subservice: immersive-reader
ms.topic: tutorial
ms.date: 01/14/2020
ms.author: dylankil
ms.custom: devx-track-python
ms.openlocfilehash: 967e290af390ff4310de3e7d0fd844e340ac289d
ms.sourcegitcommit: dea88d5e28bd4bbd55f5303d7d58785fad5a341d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87876570"
---
# <a name="tutorial-start-the-immersive-reader-using-the-python-sample-project"></a>Tutorial: Inicio del Lector inmersivo con el proyecto de ejemplo de Python

En la [introducción](./overview.md), se indica lo que es el Lector inmersivo y cómo implementa técnicas demostradas para mejorar la comprensión lectora de nuevos lectores, alumnos de idiomas y alumnos con diferencias de aprendizaje. En este tutorial se explica cómo crear una aplicación web de Python que inicie el Lector inmersivo. En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear una aplicación web de Python con pip, Flask, Jinja y virtualenv mediante un proyecto de ejemplo.
> * Obtener un token de acceso.
> * Iniciar el Lector inmersivo con contenido de ejemplo.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

* Un recurso del Lector inmersivo configurado para la autenticación de Azure Active Directory. Siga [estas instrucciones](./how-to-create-immersive-reader.md) para realizar la configuración. Necesitará algunos de los valores que se crean aquí cuando configure las propiedades del entorno. Guarde la salida de la sesión en un archivo de texto para futuras referencias.
* [Git](https://git-scm.com/).
* [SDK del Lector inmersivo](https://github.com/microsoft/immersive-reader-sdk).
* [Python](https://www.python.org/downloads/) y [pip](https://docs.python.org/3/installing/index.html). A partir de Python 3.4, pip se incluye de forma predeterminada con los instaladores binarios de Python.
* [Flask](https://flask.palletsprojects.com/en/1.0.x/).
* [Jinja](http://jinja.pocoo.org/docs/2.10/).
* [virtualenv](https://virtualenv.pypa.io/en/latest/) y [virtualenvwrapper-win para Windows](https://pypi.org/project/virtualenvwrapper-win/) o [virtualenvwrapper para OSX](https://virtualenvwrapper.readthedocs.io/en/latest/).
* [Módulo requests](https://pypi.org/project/requests/2.7.0/).
* Un IDE como [Visual Studio Code](https://code.visualstudio.com/).

## <a name="configure-authentication-credentials"></a>Configuración de las credenciales de autenticación

Cree un archivo denominado **.env** y pegue en él los siguientes nombres y valores. Proporcione los valores especificados al crear el recurso del Lector inmersivo.

```text
TENANT_ID={YOUR_TENANT_ID}
CLIENT_ID={YOUR_CLIENT_ID}
CLIENT_SECRET={YOUR_CLIENT_SECRET}
SUBDOMAIN={YOUR_SUBDOMAIN}
```

No confirme este archivo en el control de código fuente, ya que contiene secretos que no deben hacerse públicos.

Proteja el punto de conexión de la API **getimmersivereadertoken** mediante algún tipo de autenticación, por ejemplo, [OAuth](https://oauth.net/2/). La autenticación impide que usuarios no autorizados obtengan tokens para usarlos en el servicio de Lector inmersivo y su facturación. Este escenario queda fuera del alcance de este tutorial.

## <a name="create-a-python-web-app-on-windows"></a>Creación de una aplicación web de Python en Windows

Cree una aplicación web de Python con `flask` en Windows.

Instale [Git](https://git-scm.com/).

Una vez instalado Git, abra un símbolo del sistema y clone el repositorio Git del SDK del Lector inmersivo en una carpeta del equipo.

```cmd
git clone https://github.com/microsoft/immersive-reader-sdk.git
```

Instale [Python](https://www.python.org/downloads/).

Seleccione la casilla **Add Python to PATH** (Agregar Python a PATH).

![Cuadro de diálogo de instalación de Python en Windows, paso 1](./media/pythoninstallone.jpg)

Seleccione las casillas correspondientes para agregar **Características opcionales** y seleccione **Siguiente**.

![Cuadro de diálogo de instalación de Python en Windows, paso 2](./media/pythoninstalltwo.jpg)

Seleccione **Custom installation** (Instalación personalizada) y establezca la ruta de acceso de instalación como carpeta raíz (por ejemplo, `C:\Python37-32\`). A continuación, seleccione **Instalar**.

![Cuadro de diálogo de instalación de Python en Windows, paso 3](./media/pythoninstallthree.jpg)

Una vez finalizada la instalación de Python, abra un símbolo del sistema y use `cd` para ir a la carpeta Scripts de Python.

```cmd
cd C:\Python37-32\Scripts
```

Instale Flask.

```cmd
pip install flask
```

Instale Jinja2. Se trata de un motor de plantillas completo para Python.

```cmd
pip install jinja2
```

Instale virtualenv. Se trata de una herramienta para crear entornos de Python aislados.

```cmd
pip install virtualenv
```

Instale virtualenvwrapper-win. Con virtualenvwrapper se quiere facilitar el uso de virtualenv.

```cmd
pip install virtualenvwrapper-win
```

Instale el módulo requests. Requests es una biblioteca HTTP con licencia de Apache2, escrita en Python.

```cmd
pip install requests
```

Instale el módulo python-dotenv. Este módulo lee el par clave-valor del archivo .env y lo agrega a la variable de entorno.

```cmd
pip install python-dotenv
```

Cree un entorno virtual.

```cmd
mkvirtualenv advanced-python
```

Use `cd` para ir a la carpeta raíz del proyecto de ejemplo.

```cmd
cd C:\immersive-reader-sdk\js\samples\advanced-python
```

Conecte el proyecto de ejemplo con el entorno. De este modo se asigna el entorno virtual recién creado a la carpeta raíz del proyecto de ejemplo.

```cmd
setprojectdir .
```

Active el entorno virtual.

```cmd
activate
```

Ahora el proyecto debe estar activo; verá algo parecido a `(advanced-python) C:\immersive-reader-sdk\js\samples\advanced-python>` en el símbolo del sistema.

Desactive el entorno.

```cmd
deactivate
```

El prefijo `(advanced-python)` habrá desaparecido, ya que el entorno se ha desactivado.

Para volver a activar el entorno, ejecute `workon advanced-python` desde la carpeta raíz del proyecto de ejemplo.

```cmd
workon advanced-python
```

### <a name="start-the-immersive-reader-with-sample-content"></a>Inicio del Lector inmersivo con contenido de ejemplo

Cuando el entorno esté activo, ejecute el proyecto de ejemplo escribiendo `flask run` en la carpeta raíz del proyecto de ejemplo.

```cmd
flask run
```

Abra el explorador web y vaya a http://localhost:5000.

## <a name="create-a-python-web-app-on-osx"></a>Creación de una aplicación web de Python en OSX

Cree una aplicación web de Python con `flask` en OSX.

Instale [Git](https://git-scm.com/).

Una vez instalado Git, abra el terminal y clone el repositorio Git del SDK del Lector inmersivo en una carpeta del equipo.

```bash
git clone https://github.com/microsoft/immersive-reader-sdk.git
```

Instale [Python](https://www.python.org/downloads/).

La carpeta raíz de Python, `Python37-32`, por ejemplo, debe estar ahora en la carpeta de aplicaciones.

Una vez finalizada la instalación de Python, abra el terminal y use `cd` para ir a la carpeta Scripts de Python.

```bash
cd immersive-reader-sdk/js/samples/advanced-python
```

Instale PIP.

```bash
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
```

Ejecute el código siguiente para instalar pip para el usuario que tiene actualmente iniciada la sesión para evitar problemas de permisos.

```bash
python get-pip.py --user
```

```bash
sudo nano /etc/paths
```

- Escriba la contraseña cuando se le solicite.
- Agregue la ruta de acceso de la instalación de pip a la variable PATH.
- Vaya al final del archivo y escriba la ruta de acceso que desea agregar como último elemento de la lista, por ejemplo, `PATH=$PATH:/usr/local/bin`.
- Seleccione **CTRL + X** para salir.
- Escriba **Y** para guardar el búfer modificado.
- Eso es todo. Para probarlo, escriba `echo $PATH` en una nueva ventana del terminal.

Instale Flask.

```bash
pip install flask --user
```

Instale Jinja2. Se trata de un motor de plantillas completo para Python.

```bash
pip install Jinja2 --user
```

Instale virtualenv. Se trata de una herramienta para crear entornos de Python aislados.

```bash
pip install virtualenv --user
```

Instale virtualenvwrapper. Con virtualenvwrapper se quiere facilitar el uso de virtualenv.

```bash
pip install virtualenvwrapper --user
```

Instale el módulo requests. Requests es una biblioteca HTTP con licencia de Apache2, escrita en Python.

```bash
pip install requests --user
```

Instale el módulo python-dotenv. Este módulo lee el par clave-valor del archivo .env y lo agrega a la variable de entorno.

```bash
pip install python-dotenv --user
```

Elija la carpeta donde desea conservar los entornos virtuales y ejecute este comando:

```bash
mkdir ~/.virtualenvs
```

Use `cd` para ir a la carpeta de aplicación de ejemplo de Python del SDK del Lector inmersivo.

```bash
cd immersive-reader-sdk/js/samples/advanced-python
```

Cree un entorno virtual.

```bash
mkvirtualenv -p /usr/local/bin/python3 advanced-python
```

Conecte el proyecto de ejemplo con el entorno. De este modo se asigna el entorno virtual recién creado a la carpeta raíz del proyecto de ejemplo.

```bash
setprojectdir .
```

Active el entorno virtual.

```bash
activate
```

Ahora el proyecto debe estar activo; verá algo parecido a `(advanced-python) /immersive-reader-sdk/js/samples/advanced-python>` en el símbolo del sistema.

Desactive el entorno.

```bash
deactivate
```

El prefijo `(advanced-python)` habrá desaparecido, ya que el entorno se ha desactivado.

Para volver a activar el entorno, ejecute `workon advanced-python` desde la carpeta raíz del proyecto de ejemplo.

```bash
workon advanced-python
```

## <a name="start-the-immersive-reader-with-sample-content"></a>Inicio del Lector inmersivo con contenido de ejemplo

Cuando el entorno esté activo, ejecute el proyecto de ejemplo escribiendo `flask run` en la carpeta raíz del proyecto de ejemplo.

```bash
flask run
```

Abra el explorador web y vaya a http://localhost:5000.

## <a name="next-steps"></a>Pasos siguientes

* Consulte [SDK del Lector inmersivo](https://github.com/microsoft/immersive-reader-sdk) y [Referencia del SDK del Lector inmersivo](./reference.md).
* Puede ver ejemplos de código en [GitHub](https://github.com/microsoft/immersive-reader-sdk/tree/master/js/samples/).
