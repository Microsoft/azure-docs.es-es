---
title: 'Procedimiento: Ejecución de una aplicación con Fortanix enclave Manager'
description: Obtenga información sobre cómo usar Fortanix Enclave Manager para convertir sus imágenes de contenedor
services: virtual-machines
author: JBCook
ms.service: virtual-machines
ms.subservice: workloads
ms.workload: infrastructure
ms.topic: how-to
ms.date: 8/12/2020
ms.author: JenCook
ms.openlocfilehash: b5b0f9acc45dba81bb7653c844bb8c78a8bd29ba
ms.sourcegitcommit: ac7ae29773faaa6b1f7836868565517cd48561b2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88826300"
---
# <a name="how-to-run-an-application-with-fortanix-enclave-manager"></a>Instrucciones: Ejecución de una aplicación con Fortanix Enclave Manager 

Comience a ejecutar la aplicación en la computación confidencial de Azure con [Fortanix Enclave Manager](https://azuremarketplace.microsoft.com/marketplace/apps/fortanix.enclave_manager?tab=Overview) y [Fortanix Node Agent](https://azuremarketplace.microsoft.com/marketplace/apps/fortanix.rte_node_agent) de [Fortanix](https://www.fortanix.com/).


Fortanix es un proveedor de software de terceros con productos y servicios basados en la infraestructura de Azure. Hay otros proveedores de terceros que ofrecen servicios de computación confidencial similares en Azure.

> [!Note] 
 > LOS PRODUCTOS A LOS QUE SE HACE REFERENCIA EN ESTE DOCUMENTO NO ESTÁN BAJO EL CONTROL DE MICROSOFT. MICROSOFT LE PROPORCIONA ESTA INFORMACIÓN SOLO PARA SU COMODIDAD Y LAS REFERENCIAS A ESTOS PRODUCTOS QUE NO SON DE MICROSOFT NO IMPLICA QUE TENGAN LA APROBACIÓN DE MICROSOFT.



En este tutorial se muestra cómo convertir la imagen de la aplicación en una imagen protegida para computación confidencial. Este entorno usa software de [Fortanix](https://www.fortanix.com/) basado en las máquinas virtuales habilitadas para Intel SGX de la serie DCsv2 de Azure. Esta solución organiza directivas de seguridad críticas, como la comprobación de identidades y el control de acceso a datos.

 Para obtener el soporte técnico específico de Fortanix, únase a la [comunidad Fortanix en Slack](https://fortanix.com/community/) y use el canal #enclavemanager.


## <a name="prerequisites"></a>Requisitos previos

1. Si no tiene una cuenta de Fortanix Enclave Manager, [regístrese](https://em.fortanix.com/auth/sign-up) antes de empezar.
1. Un registro de [Docker](https://docs.docker.com/) privado al que se enviarán las imágenes de aplicaciones convertidas.
1. Si no tiene una suscripción a Azure, [cree una cuenta](https://azure.microsoft.com/pricing/purchase-options/pay-as-you-go/) antes de empezar.

> [!NOTE]
> Las cuentas de evaluación gratuita no tienen acceso a las máquinas virtuales que se usan en este tutorial. Actualice a una suscripción de pago por uso.

## <a name="add-an-application-to-fortanix-enclave-manager"></a>Adición de una aplicación a Fortanix enclave Manager
1. Inicie sesión en [Fortanix EM](https://fortanix.com)
1. Vaya a la página **Accounts** (Cuentas) y seleccione **ADD ACCOUNT** (AGREGAR CUENTA) para crear una cuenta nueva. 
    
![Crear una cuenta](media/how-to-fortanix-enclave-manager/create-account.png)

1. Una vez creada la cuenta, pulse **SELECT** (SELECCIONAR) para seleccionar la cuenta recién creada. Ahora podemos empezar a inscribir nodos de ejecución y crear aplicaciones. 
1. Seleccione el botón **+ Nueva aplicación** para agregar una aplicación. En este ejemplo, vamos a agregar una aplicación de Flask Server Enclave OS. 

1. Seleccione el botón **ADD** (AGREGAR) para la aplicación Enclave OS. 

    ![Agregar aplicación](media/how-to-fortanix-enclave-manager/add-enclave-application.png)

    > [!NOTE]
    > En este tutorial solo se explica la adición de aplicaciones Enclave OS. [Obtenga más información](https://support.fortanix.com/hc/en-us/articles/360044746932-Bringing-EDP-Rust-Apps-to-Enclave-Manager) acerca de cómo migrar las aplicaciones EDP Rust a Fortanix enclave Manager. 

6. En este tutorial, usaremos el registro de Docker de Fortanix para la aplicación de ejemplo. Rellene los datos con la información siguiente. Use su registro de Docker privado para mantener la imagen de salida. 
 
    - **Nombre de la aplicación**: Python Application Server (servidor de aplicación de Python)
    - **Descripción**: Python Flask Server (servidor de Flask en Python)
    - **Nombre de la imagen de entrada**: fortanix/python-flask
    - **Nombre de la imagen de salida**: fortanix-private/python-flask-sgx
    - **ISVPRODID**: 1
    - **ISVSVM**: 1
    - **Tamaño de memoria**: 1 GB
    - **Número de subprocesos**: 128

    *Opcional*: Ejecute la aplicación.
    - **Docker Hub**: [https://hub.docker.com/u/fortanix](https://hub.docker.com/u/fortanix)
    - **Aplicación**: fortanix/python-flask

        Ejecute el siguiente comando:
         ```bash
            sudo docker run fortanix/python-flask
         ```

1. Agregue un certificado. Rellene la información con los datos siguientes y, a continuación, seleccione **NEXT** (SIGUIENTE):
    - **Dominio**: miapp.dominio.dom
    - **Tipo**: Certificate Issued by Enclave Manager (Certificado emitido por Enclave Manager) 
    - **Ruta de acceso de la clave**: /clavedeapp.pem
    - **Tipo de clave**: RSA
    - **Ruta de acceso del certificado**: /certificadodeapp.pem
    - **Tamaño de la clave RSA**: 2048 bits
    

## <a name="create-an-image"></a>Creación de una imagen
Una imagen de Fortanix EM es una versión de software o de una aplicación. Cada imagen está asociada con un hash de Enclave (MRENCLAVE). 
1. En la página **Add Image** (Agregar imagen), escriba las **REGISTRY CREDENTIALS** (CREDENCIALES DEL REGISTRO) para **Output image name** (Nombre de la imagen de salida). Estas credenciales se usan para acceder al registro privado de Docker al que se enviará la imagen. 

    ![creación de una imagen](media/how-to-fortanix-enclave-manager/create-image.png)
1. Proporcione la etiqueta de imagen y seleccione **Create** (Crear).

    ![adición de una etiqueta](media/how-to-fortanix-enclave-manager/add-tag.png)

## <a name="domain-and-image-allow-listing"></a>Lista de permitidos de dominio e imagen 
Una aplicación cuyo dominio se agrega a la lista de permitidos obtendrá un certificado TLS de Fortanix Enclave Manager. Del mismo modo, cuando una aplicación se ejecute desde la imagen convertida, intentará ponerse en contacto con Fortanix Enclave Manager. Después, la aplicación solicitará un certificado TLS. 

Cambie a la pestaña **Tasks** (Tareas) de la izquierda y apruebe las solicitudes pendientes para permitir el dominio y la imagen. 

## <a name="enroll-compute-node-agent-in-azure"></a>Inscripción del agente de nodo de ejecución en Azure

### <a name="generate-and-copy-join-token"></a>Generación y copia del token de unión
En Fortanix Enclave Manager, creará un token. Este permite que un nodo de ejecución de Azure se autentique a sí mismo. Deberá proporcionar este token a la máquina virtual de Azure.
1. En la consola de administración, seleccione el botón **+ ENROLL NODE** (INSCRIBIR NODO). 
1. Seleccione **GENERATE TOKEN** (GENERAR TOKEN) para generar el token de unión. Copie el token.

### <a name="enroll-nodes-into-fortanix-node-agent-in-azure-marketplace"></a>Inscripción de nodos en Fortanix Node Agent de Azure Marketplace

Al crear una instancia de Fortanix Node Agent se implementará una máquina virtual, una interfaz de red, una red virtual, un grupo de seguridad de red y una dirección IP pública en el grupo de recursos de Azure. La suscripción a Azure se facturará por hora para la máquina virtual. Antes de crear una instancia de Fortanix Node Agent, revise la [página de precios de máquinas virtuales](https://azure.microsoft.com/pricing/details/virtual-machines/linux/) de Azure de la serie DCsv2. Elimine los recursos de Azure cuando no estén en uso. 

1. Vaya a [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/) e inicie sesión con sus credenciales de Azure.
1. En la barra de búsqueda, escriba **Fortanix Confidential Computing Node Agent**. Seleccione la aplicación que aparece en el cuadro de búsqueda denominada **Fortanix Confidential Computing Node Agent** para ir a la página principal de la oferta. 
     ![buscar en Marketplace](media/how-to-fortanix-enclave-manager/search-fortanix-marketplace.png)
1. Seleccione **Obtenerla ahora**, rellene la información si es necesario y seleccione **Continuar**. Se le redirigirá a Azure Portal. 
1. Seleccione **Crear** para entrar en la página de implementación de Fortanix Confidential Computing Node Agent.
1. En esta página, especificará información para implementar una máquina virtual. En concreto, esta máquina virtual es una máquina virtual habilitada para Intel SGX de la serie DCsv2 de Azure con el software de Fortanix Node Agent instalado. Node Agent permitirá que la imagen convertida se ejecute de forma segura en los nodos de Intel SGX en Azure.  Seleccione la **suscripción** y el **grupo de recursos** en los que quiere implementar la máquina virtual y los recursos asociados. 
 
    > [!NOTE]
    > Existen restricciones al implementar máquinas virtuales de la serie DCsv2 en Azure. Quizá deba solicitar la cuota para núcleos adicionales. Para más información, consulte [Soluciones de computación confidencial en máquinas virtuales de Azure](https://docs.microsoft.com/azure/confidential-computing/virtual-machine-solution). 

1. Seleccione una región disponible.
1. Escriba un nombre para la máquina virtual en el campo **Nombre de nodo**. 
1. Escriba un nombre de usuario y contraseña (o clave SSH) para la autenticación en la máquina virtual.
1. Deje el tamaño del disco del SO predeterminado en 200 y seleccione un tamaño de máquina virtual (Standard_DC4s_v2 será suficiente para este tutorial).
1. Pegue el token generado anteriormente en el campo **Token de unión**.

     ![implementar recursos](media/how-to-fortanix-enclave-manager/deploy-fortanix-node-agent.png)

1. Seleccione **Revisar + crear**. Asegúrese de superar la validación y, después, seleccione **Crear**. Una vez que se hayan implementado todos los recursos, el nodo de ejecución ya estará inscrito en Enclave Manager. 

## <a name="run-the-application-image-on-the-compute-node"></a>Ejecución de la imagen de aplicación en el nodo de ejecución
Ejecute la aplicación mediante la ejecución del comando siguiente. Asegúrese de cambiar la dirección IP del nodo, el puerto y el nombre de la imagen convertida como entradas para su aplicación específica. 
 
En este tutorial, el comando que se ejecutará es:     

```bash
    sudo docker run `
        --device /dev/isgx:/dev/isgx `
        --device /dev/gsgx:/dev/gsgx `
        -v /var/run/aesmd/aesm.socket:/var/run/aesmd/aesm.socket `
        -e NODE_AGENT_BASE_URL=http://52.152.206.164:9092/v1/ fortanix-private/python-flask-sgx
```

donde, 
- *52.152.206.164* es la IP del host de Node Agent;
- *9092* es el puerto que Node Agent escucha;
- *fortanix-private/python-flask-sgx* es la aplicación convertida que se puede encontrar en la pestaña Imágenes, en la columna **Nombre de imagen** de la tabla **Imágenes** en el portal web de Fortanix Enclave Manage. 
    
## <a name="verify-and-monitor-the-running-application"></a>Comprobación y supervisión de la aplicación en ejecución
1. Vuelva a [Fortanix Enclave Manager](https://em.fortanix.com/console).
1. Asegúrese de que está trabajando dentro de la **Cuenta** en la que inscribió el nodo.
1. Diríjase a **Management Console** (Consola de administración) al seleccionar el icono superior del panel de navegación izquierdo. 
1. Seleccione la pestaña **Application** (Aplicación).
1. Compruebe que hay una aplicación en ejecución con un nodo de ejecución asociado.


## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, puede eliminar el grupo de recursos, la máquina virtual y todos los recursos asociados. Al eliminar el grupo de recursos, se anulará la inscripción de los nodos asociados con la imagen convertida. 

Seleccione el grupo de recursos de la máquina virtual y haga clic en **Eliminar**. Confirme el nombre del grupo de recursos para terminar de eliminar los recursos.

Para eliminar la cuenta de Fortanix Enclave Manager que ha creado, vaya a la [página de cuentas](https://em.fortanix.com/accounts) en Enclave Manager. Mantenga el puntero sobre la cuenta que quiere eliminar. Seleccione los puntos negros verticales en la esquina superior derecha y seleccione **Delete Account** (Eliminar cuenta).

  ![delete](media/how-to-fortanix-enclave-manager/delete-account.png)

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha usado las herramientas de Fortanix para convertir la imagen de la aplicación con el fin de que se ejecute en una máquina virtual de computación confidencial. Para más información sobre las máquinas virtuales de computación confidencial en Azure, consulte [Soluciones en Virtual Machines](virtual-machine-solutions.md). 

Para más información sobre las ofertas de computación confidencial de Azure, consulte [Introducción a la computación confidencial de Azure](overview.md).

 Más información sobre cómo completar tareas similares con otras ofertas de terceros en Azure, como [Anjuna](https://azuremarketplace.microsoft.com/marketplace/apps/anjuna-5229812.aee-az-v1) y [Scone](https://sconedocs.github.io).  