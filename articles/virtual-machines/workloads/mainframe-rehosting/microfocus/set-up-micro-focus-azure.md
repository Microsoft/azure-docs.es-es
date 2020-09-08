---
title: Instalación de Micro Focus Enterprise Server 5.0 y Enterprise Developer 5.0 en Azure | Microsoft Docs
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
ms.openlocfilehash: 59566fc051b1e84d8e271b3c9d061f2481dcc58f
ms.sourcegitcommit: c293217e2d829b752771dab52b96529a5442a190
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/15/2020
ms.locfileid: "88245322"
---
# <a name="install-micro-focus-enterprise-server-50-and-enterprise-developer-50-on-azure"></a>Instalación de Micro Focus Enterprise Server 5.0 y Enterprise Developer 5.0 en Azure

Este artículo muestra cómo configurar [Micro Focus Enterprise Server 5.0](https://www.microfocus.com/documentation/enterprise-developer/ed50pu5/ES-WIN/GUID-F7D8FD6E-BDE0-4169-8D8C-96DDFFF6B495.html) y [Micro Focus Enterprise Developer 5.0](https://www.microfocus.com/documentation/enterprise-developer/ed50/) en Microsoft Azure.

Un entorno de desarrollo y pruebas es una carga de trabajo común en Azure. Este escenario es común porque es rentable y fácil de implementar y anular. Con Enterprise Server, Micro Focus ha creado una de las mayores plataformas de rehospedaje de sistemas centrales disponibles. Puede ejecutar cargas de trabajo de z/OS en una plataforma x86 más económica en Azure con máquinas virtuales Windows o Linux (VM).

Esta configuración usa máquinas virtuales de Azure que ejecutan la imagen de Windows Server 2016 de Azure Marketplace con Microsoft SQL Server 2017 ya instalado. Esta configuración también se aplica a Azure Stack.

El entorno de desarrollo correspondiente de Enterprise Server es Enterprise Developer, que se ejecuta en Microsoft Visual Studio 2017 o versiones posteriores, Visual Studio Community (descarga gratuita) o Eclipse. En este artículo se muestra cómo se implementa con una máquina virtual Windows Server 2016 que viene con Visual Studio 2017 o posterior instalado.

## <a name="prerequisites"></a>Requisitos previos

Antes de comenzar, revise estos requisitos previos:

-   Suscripción a Azure. Si no tiene una, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

-   El software de Micro Focus y una licencia válida (o licencia de prueba). Si es un cliente existente de Micro Focus, póngase en contacto con su representante de Micro Focus. En caso contrario, [solicite una versión de evaluación](https://www.microfocus.com/products/enterprise-suite/enterprise-server/trial/).

-   Obtenga la documentación de [Enterprise Server y Enterprise Developer](https://www.microfocus.com/documentation/enterprise-developer/ed50/).

    > [!Note]
    > Hay algunas opciones para controlar el acceso a las máquinas virtuales:
    > -   Un procedimiento recomendado consiste en configurar [Azure Bastion](https://azure.microsoft.com/services/azure-bastion/).
    > -   Un túnel de [red privada virtual (VPN) de sitio a sitio](../../../../vpn-gateway/vpn-gateway-tutorial-vpnconnection-powershell.md).
    > -   Una máquina virtual jumpbox.

## <a name="install-enterprise-server"></a>Instalación de Enterprise Server

1.  Para una mejor seguridad y facilidad de uso, considere la posibilidad de crear un nuevo grupo de recursos solo para este proyecto; por ejemplo, **RGMicroFocusEntServer**. Use la primera parte del nombre en Azure para elegir el tipo de recurso para que resulte más fácil de detectar en una lista.

2.  Cree una máquina virtual. En Azure Marketplace, seleccione la máquina virtual y el sistema operativo que desee. Esta es una configuración recomendada:

    -   **Enterprise Server:** Seleccione una **máquina virtual ES2 v3** (con 2 vCPU y 16 GB de memoria) con Windows Server 2016 y SQL Server 2017 instalado. Esta imagen está disponible en Azure Marketplace. Enterprise Server puede usar también Azure SQL Database.

    -   **Enterprise Developer:** Seleccione una **máquina virtual B2ms** (con 2 vCPU y 8 GB de memoria) con Windows 10 y Visual Studio instalado. Esta imagen está disponible en Azure Marketplace.

3.  En la sección **Básico**, escriba el nombre de usuario y la contraseña. Seleccione la **Suscripción** y la **Ubicación/Región** que le gustaría usar para las máquinas virtuales. Seleccione **RGMicroFocusEntServer** para el grupo de recursos.

4.  Coloque ambas máquinas virtuales en la misma red virtual para que puedan comunicarse entre sí.

5.  En el resto de la configuración, acepte los valores predeterminados. Recuerde el nombre de usuario y la contraseña que ha creado para el administrador de estas máquinas virtuales.

6.  Cuando se hayan creado las máquinas virtuales, abra los puertos de entrada **9003, 86** y **80** para HTTP y **3389** para el Protocolo de escritorio remoto (RDP) en la máquina de Enterprise Server y el puerto **3389** en el equipo de Enterprise Developer.

7.  Para iniciar sesión en la máquina virtual de Enterprise Server, en Azure Portal, seleccione la máquina virtual ES2 v3. Vaya a la sección **Información general** y seleccione **Conectar** para iniciar una sesión de RDP. Inicie sesión con las credenciales que creó para la máquina virtual.

8.  Desde la sesión de RDP, cargue los dos archivos siguientes. Dado que usa Windows, puede arrastrar y soltar los archivos en la sesión de RDP:

    -   `es\_50.exe`, el archivo de instalación de Enterprise Developer.

    -   `mflic`, el archivo de licencia correspondiente; no se podrá cargar Enterprise Server sin él.

9.  Haga doble clic en el archivo para iniciar la instalación. En la primera ventana, seleccione la ubicación de instalación y acepte el contrato de licencia de usuario final.

    ![Pantalla de la instalación de Micro Focus Enterprise Server](media/install-image-1.png)

    Cuando se completa la instalación, aparece el mensaje siguiente:

    ![Pantalla de la instalación de Micro Focus Enterprise Server](media/install-image-2.png)

 ### <a name="check-for-updates"></a>Buscar actualizaciones

Después de la instalación, asegúrese de comprobar si hay actualizaciones adicionales, ya que varios requisitos previos, como Microsoft C++ Redistributable y .NET Framework, se instalan junto con Enterprise Server.

### <a name="upload-the-license"></a>Carga de la licencia

1.  Inicie la administración de licencias de Micro Focus.

2.  Seleccione **Iniciar** \> **Micro Focus License Manager** \> **Administración de licencias** y, a continuación, haga clic en la pestaña **Instalar**. Elija el tipo de formato de licencia que se va a cargar: un archivo de licencia o un código de licencia de 16 caracteres. Por ejemplo, si elige un archivo, en **Archivo de licencia**, vaya al archivo *`mflic` cargado previamente en la máquina virtual y seleccione **Instalar licencias**.

    ![Cuadro de diálogo de administración de licencias de Micro Focus](media/install-image-3.png)

3.  Compruebe que se carga Enterprise Server. Intente iniciar el sitio de administración de Enterprise Server desde un explorador con esta dirección URL: `http://localhost:86/`. Se muestra la página de administración de Enterprise Server.

    ![Página de administración de Enterprise Server](media/install-image-4.png)

## <a name="install-enterprise-developer-on-the-developer-machine"></a>Instalación de Enterprise Developer en el equipo del desarrollador

1.  Seleccione el grupo de recursos que creó anteriormente (por ejemplo, **RGMicroFocusEntServer**) y, a continuación, seleccione la imagen de Enterprise Developer.

2.  Para iniciar sesión en la máquina virtual, vaya a la sección **Información general** y seleccione **Conectar**. Este inicio de sesión inicia una sesión de RDP. Inicie sesión con las credenciales que creó para la máquina virtual.

3.  Desde la sesión de RDP, cargue los dos archivos siguientes (puede arrastrar y soltar si lo desea):

    -   `edvs2017.exe`, el archivo de instalación de Enterprise Developer.

    -   `mflic`, el archivo de licencia correspondiente (no se podrá cargar Enterprise Developer sin él).

4.  Haga doble clic en el archivo **edvs2017.exe** para iniciar la instalación. En la primera ventana, seleccione la ubicación de instalación y acepte el contrato de licencia de usuario final. Si lo desea, elija **Instalar Rumba 9.5** para instalar este emulador de terminal, probablemente lo necesitará.

    ![Cuadro de diálogo de la instalación de Micro Focus Enterprise Developer para Visual Studio 2017](media/install-image-5.png)

5.  Cuando se completa la instalación, aparece el mensaje siguiente:

    ![Mensaje de instalación correcta](media/install-image-6.png)

6.  Inicie el administrador de licencias de Micro Focus tal como hizo para Enterprise Server. Seleccione **Iniciar** \> **Micro Focus License Manager** \> **Administración de licencias** y haga clic en la pestaña **Instalar**.

7.  Elija el tipo de formato de licencia que se va a cargar: un archivo de licencia o un código de licencia de 16 caracteres. Por ejemplo, para un archivo, en **Archivo de licencia**, vaya al archivo `mflic` cargado previamente en la máquina virtual y seleccione **Instalar licencias**.

    ![Cuadro de diálogo de administración de licencias de Micro Focus](media/install-image-7.png)

Cuando se carga Enterprise Developer, la implementación de un entorno de desarrollo y pruebas de Micro Focus en Azure ha finalizado.

**Pasos siguientes**

-   [Configuración de la aplicación BankDemo](./demo.md)

-   [Ejecución de Enterprise Server en contenedores de Docker](./run-enterprise-server-container.md)

-   [Migrar la aplicación del sistema central](/azure/architecture/cloud-adoption/infrastructure/mainframe-migration/application-strategies)
