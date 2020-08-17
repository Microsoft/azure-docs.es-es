---
title: Implementación de la oferta de Marketplace autoadministrada de OpenShift Container Platform 3.11 en Azure
description: Implemente la oferta de Marketplace de OpenShift Container Platform 3.11 en Azure.
author: haroldwongms
manager: mdotson
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: how-to
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 10/14/2019
ms.author: haroldw
ms.openlocfilehash: 36adf35c5fbfc3e88b7d9af425ebabc852707e69
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87374106"
---
# <a name="configure-prerequisites"></a>Configuración de los requisitos previos

Antes de usar la oferta de Marketplace para implementar un clúster autoadministrado de OpenShift Container Platform 3.11 en Azure, se deben configurar algunos requisitos previos.  Lea el artículo de [requisitos previos de OpenShift](./openshift-container-platform-3x-prerequisites.md) para obtener instrucciones para crear una clave ssh (sin una frase de contraseña), una instancia de Azure Key Vault, el secreto de Key Vault y una entidad de servicio.

 
## <a name="deploy-using-the-marketplace-offer"></a>Implementar mediante la oferta de Marketplace

La manera más sencilla de implementar un clúster autoadministrado de OpenShift Container Platform 3.11 en Azure es usar la [oferta de Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/osatesting.open-shift-azure-proxy).

Esta opción es la más sencilla, pero también tiene funcionalidades de personalización limitadas. La oferta de Marketplace implementa OpenShift Container Platform 3.11.82 e incluye las siguientes opciones de configuración:

- **Nodos maestros**: tres (3) nodos maestros con tipo de instancia configurable.
- **Nodos de infraestructura**: tres (3) nodos de infraestructura con tipo de instancia configurable.
- **Nodos**: el número de nodos (entre 1 y 9) y el tipo de instancia son configurables.
- **Tipo de disco**: se usan discos administrados.
- **Redes**: compatibilidad con una red nueva o existente, y un intervalo CIDR personalizado.
- **CNS**: se puede habilitar CNS.
- **Métricas**: se pueden habilitar las métricas Hawkular.
- **Registro**: se puede habilitar el registro EFK.
- **Proveedor de soluciones en la nube de Azure**: habilitado de forma predeterminada, se puede deshabilitar.

En la parte superior izquierda de Azure Portal, haga clic en **Crear un recurso**, escriba "openshift container platform" en el cuadro de búsqueda y presione ENTRAR.

   ![Nueva búsqueda de recursos](media/openshift-marketplace-self-managed/ocp-search.png)  
<br>

Se abrirá la página de resultados con **Red Hat OpenShift Container Platform Self-Managed 3.11** en la lista. 

   ![Resultado de la nueva búsqueda de recursos](media/openshift-marketplace-self-managed/ocp-searchresult.png)  
<br>

Haga clic en la oferta para ver los detalles de la misma. Para implementar esta oferta, haga clic en **Crear**. Aparecerá la interfaz de usuario para que escriba los parámetros necesarios. La primera pantalla es la hoja **Aspectos básicos**.

   ![Página del título de la oferta](media/openshift-marketplace-self-managed/ocp-titlepage.png)  
<br>

**Conceptos básicos**

Para obtener ayuda sobre cualquiera de los parámetros de entrada, mantenga el mouse sobre la ***i*** junto al nombre del parámetro.

Escriba los valores para los parámetros de entrada y haga clic en **Aceptar**.

| Parámetro de entrada | Descripción del parámetro |
|-----------------------|-----------------|
| Nombre de usuario administrador de la máquina virtual | El usuario administrador que se creará en todas las instancias de máquina virtual |
| Clave pública SSH para el usuario administrador | Clave pública SSH usada para iniciar sesión en una máquina virtual: no debe tener una frase de contraseña |
| Subscription | Suscripción de Azure en la que se implementa el clúster. |
| Grupo de recursos | Crear un nuevo grupo de recursos o seleccionar uno vacío existente para los recursos de clúster |
| Location | Región de Azure en la que se implementa el clúster |

   ![Hoja de aspectos básicos de la oferta](media/openshift-marketplace-self-managed/ocp-basics.png)  
<br>

**Configuración de infraestructura**

Escriba los valores para los parámetros de entrada y haga clic en **Aceptar**.

| Parámetro de entrada | Descripción del parámetro |
|-----------------------|-----------------|
| Prefijo del nombre del clúster OCP | Prefijo del clúster usado para configurar los nombres de host para todos los nodos. Entre 1 y 20 caracteres. |
| Tamaño del nodo maestro | Acepte el tamaño de máquina virtual predeterminado o haga clic en **Cambiar tamaño** para seleccionar un tamaño de máquina virtual diferente.  Seleccione el tamaño de máquina virtual adecuado para su carga de trabajo. |
| Tamaño del nodo de infraestructura | Acepte el tamaño de máquina virtual predeterminado o haga clic en **Cambiar tamaño** para seleccionar un tamaño de máquina virtual diferente.  Seleccione el tamaño de máquina virtual adecuado para su carga de trabajo. |
| Número de nodos de aplicación | Acepte el tamaño de máquina virtual predeterminado o haga clic en **Cambiar tamaño** para seleccionar un tamaño de máquina virtual diferente.  Seleccione el tamaño de máquina virtual adecuado para su carga de trabajo. |
| Tamaño del nodo de aplicación | Acepte el tamaño de máquina virtual predeterminado o haga clic en **Cambiar tamaño** para seleccionar un tamaño de máquina virtual diferente.  Seleccione el tamaño de máquina virtual adecuado para su carga de trabajo. |
| Tamaño del host de tipo bastión | Acepte el tamaño de máquina virtual predeterminado o haga clic en **Cambiar tamaño** para seleccionar un tamaño de máquina virtual diferente.  Seleccione el tamaño de máquina virtual adecuado para su carga de trabajo. |
| Red virtual nueva o existente | Crear una nueva red virtual (valor predeterminado) o usar una red virtual existente. |
| Elegir la configuración predeterminada de CIDR o personalizar el intervalo de IP (CIDR) | Acepte los intervalos de CIDR predeterminados o seleccione **Intervalo IP personalizado** y escriba la información de CIDR personalizada.  La configuración predeterminada creará una red virtual con CIDR de 10.0.0.0/14, una subred maestra con 10.1.0.0/16, una subred de infraestructura con 10.2.0.0/16, y un proceso y subred de cns con 10.3.0.0/16. |
| Nombre del grupo de recursos de Key Vault | Nombre del grupo de recursos que contiene la instancia de Key Vault. |
| Nombre de Key Vault | El nombre de Key Vault que contiene el secreto con la clave privada ssh.  Puede contener solo caracteres alfanuméricos y guiones, y debe tener entre 3 y 24 caracteres. |
| Nombre del secreto | El nombre del secreto que contiene la clave privada ssh.  Puede contener solo caracteres alfanuméricos o guiones. |

   ![Hoja de la infraestructura de la oferta](media/openshift-marketplace-self-managed/ocp-inframain.png)  
<br>

**Cambiar el tamaño**

Para seleccionar otro tamaño de máquina virtual, haga clic en ***Cambiar tamaño***.  Se abrirá la ventana de selección de máquina virtual.  Seleccione el tamaño de máquina virtual que quiera y haga clic en **Seleccionar**.

   ![Selección de un tamaño de máquina virtual](media/openshift-marketplace-self-managed/ocp-selectvmsize.png)  
<br>

**Red virtual existente**

| Parámetro de entrada | Descripción del parámetro |
|-----------------------|-----------------|
| Nombre de la red virtual existente | Nombre de la red virtual existente |
| Nombre de subred para los nodos maestros | El nombre de la subred existente para los nodos maestros.  Debe contener al menos 16 direcciones IP y seguir el RFC 1918. |
| Nombre de subred para nodos de infraestructura | El nombre de la subred existente para los nodos de infraestructura.  Debe contener al menos 32 direcciones IP y seguir el RFC 1918. |
| Nombre de subred para los nodos de proceso y cns | El nombre de subred existente para los nodos de proceso y cns.  Debe contener al menos 32 direcciones IP y seguir el RFC 1918. |
| Grupo de recursos para la red virtual existente. | El nombre del grupo de recursos que contiene la red virtual existente. |

   ![Red virtual existente de la infraestructura de la oferta](media/openshift-marketplace-self-managed/ocp-existingvnet.png)  
<br>

**Intervalo IP personalizado**

| Parámetro de entrada | Descripción del parámetro |
|-----------------------|-----------------|
| Rango de direcciones de la red virtual | CIDR personalizada para la red virtual |
| Intervalo de direcciones para la subred que contiene los nodos maestros. | CIDR personalizada para la subred maestra |
| Intervalo de direcciones para la subred que contiene los nodos de infraestructura. | CIDR personalizada para la subred de infraestructura |
| Intervalo de direcciones para la subred que contiene los nodos de proceso y cns. | CIDR personalizada para los nodos de proceso y cns |

   ![Intervalo IP personalizado de la infraestructura de la oferta](media/openshift-marketplace-self-managed/ocp-customiprange.png)  
<br>

**OpenShift Container Platform 3.11**

Escriba los valores para los parámetros de entrada y haga clic en **Aceptar**.

| Parámetro de entrada | Descripción del parámetro |
|-----------------------|-----------------|
| Contraseña del usuario administrador de OpenShift | La contraseña del usuario inicial de OpenShift.  Este usuario será también el administrador del clúster. |
| Confirmar la contraseña del usuario administrador de OpenShift | Vuelva a escribir la contraseña del usuario administrador de OpenShift. |
| Nombre de usuario de Red Hat Subscription Manager | El nombre de usuario para acceder a su suscripción de Red Hat o identificador de la organización.  Esta credencial se usa para registrar la instancia RHEL en su suscripción y no será almacenada por Microsoft ni Red Hat. |
| Contraseña de usuario de Red Hat Subscription Manager | La contraseña para acceder a su suscripción de Red Hat o clave de activación.  Esta credencial se usa para registrar la instancia RHEL en su suscripción y no será almacenada por Microsoft ni Red Hat. |
| Identificador de grupo de Red Hat Subscription Manager OpenShift | El identificador de grupo que contiene los derechos de OpenShift Container Platform. Asegúrese de que tiene suficientes derechos de OpenShift Container Platform para la instalación del clúster. |
| Identificador de grupo de Red Hat Subscription Manager OpenShift para agentes o nodos maestros | El identificador de grupo que contiene los derechos de OpenShift Container Platform para agentes o nodos maestros. Asegúrese de que tiene suficientes derechos de OpenShift Container Platform para la instalación del clúster. Si no usa un identificador de grupo de agente o maestro, escriba el identificador de grupo para los nodos de aplicación. |
| Configurar el proveedor de soluciones en la nube de Azure | Configure OpenShift para que use el proveedor de soluciones en la nube de Azure. Es necesario si se asocia un disco de Azure para los volúmenes persistentes.  El valor predeterminado es Sí. |
| GUID de Id. de cliente de la entidad de servicio de Azure AD | El GUID de Id. de cliente de la entidad de servicio de Azure AD, también conocido como AppID. Solo es necesario si Configurar el proveedor de soluciones en la nube de Azure se establece en **Sí**. |
| Secreto de Id. de cliente de la entidad de servicio de Azure AD | El secreto de Id. de cliente de la entidad de servicio de Azure AD. Solo es necesario si Configurar el proveedor de soluciones en la nube de Azure se establece en **Sí**. |
 
   ![Hoja de OpenShift de la oferta](media/openshift-marketplace-self-managed/ocp-ocpmain.png)  
<br>

**Configuración adicional**

La hoja Configuración adicional permite configurar CNS para el almacenamiento de GlusterFS, registro, métricas y subdominio del enrutador.  El valor predeterminado no instala ninguna de estas opciones y usa nip.io como el subdominio de enrutador con fines de prueba. Al habilitar CNS se instalarán tres nodos de proceso adicionales con tres discos asociados adicionales que hospedarán pods de GlusterFS.  

Escriba los valores para los parámetros de entrada y haga clic en **Aceptar**.

| Parámetro de entrada | Descripción del parámetro |
|-----------------------|-----------------|
| Configurar Container Native Storage (CNS) | Instala CNS en el clúster de OpenShift y lo habilita como almacenamiento. Será el valor predeterminado si el proveedor de Azure está deshabilitado. |
| Configurar el registro del clúster | Instala la funcionalidad de registro EFK en el clúster.  Cambiar el tamaño de los nodos de infraestructura adecuadamente para el hospedaje de pods EFK. |
| Configurar las métricas para el clúster | Instala las métricas Hawkular en el clúster de OpenShift.  Cambie el tamaño de los nodos de infraestructura adecuadamente para el hospedaje de métricas Hawkular. |
| Subdominio del enrutador predeterminado | Seleccione nipio para pruebas o personalizado para escribir su propio subdominio para producción. |
 
   ![Hoja adicional de la oferta](media/openshift-marketplace-self-managed/ocp-additionalmain.png)  
<br>

**Configuración adicional: parámetros adicionales**

| Parámetro de entrada | Descripción del parámetro |
|-----------------------|-----------------|
| Tamaño del nodo (CNS) | Acepte el tamaño del nodo predeterminado o seleccione **Cambiar tamaño** para seleccionar un nuevo tamaño de máquina virtual. |
| Escribir el subdominio personalizado | El dominio de enrutamiento personalizado que se usará para exponer las aplicaciones a través del enrutador en el clúster de OpenShift.  Asegúrese de crear la entrada DNS comodín adecuada. |
 
   ![Instalación del cns adicional de la oferta](media/openshift-marketplace-self-managed/ocp-additionalcnsall.png)  
<br>

**Resumen**

En esta fase se produce la validación para comprobar que la cuota de núcleos es suficiente para implementar el número total de máquinas virtuales seleccionadas para el clúster.  Revise todos los parámetros que se especificaron.  Si las entradas son válidas, haga clic en **Aceptar** para continuar.

   ![Hoja de resumen de la oferta](media/openshift-marketplace-self-managed/ocp-summary.png)  
<br>

**Comprar**

Confirme la información de contacto en la página de compra y haga clic en **Comprar** para aceptar los términos de uso y empezar la implementación del clúster de OpenShift Container Platform.

   ![Hoja de compra de la oferta](media/openshift-marketplace-self-managed/ocp-purchase.png)  
<br>


## <a name="connect-to-the-openshift-cluster"></a>Conexión con el clúster de OpenShift

Cuando finalice la implementación, puede recuperar la conexión de la sección de salida de la implementación. Conéctese a la consola de OpenShift con el explorador mediante la **dirección URL de la consola de OpenShift**. También puede acceder mediante SSH al host de tipo bastión. El siguiente es un ejemplo en el que el nombre de usuario administrador es clusteradmin y el nombre de dominio completo de DNS de la dirección IP pública del host de tipo bastión es bastiondns4hawllzaavu6g.eastus.cloudapp.azure.com:

```bash
$ ssh clusteradmin@bastiondns4hawllzaavu6g.eastus.cloudapp.azure.com
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no se necesiten, puede usar el comando [az group delete](/cli/azure/group) para quitar el grupo de recursos, el clúster de OpenShift y todos los recursos relacionados.

```azurecli 
az group delete --name openshiftrg
```

## <a name="next-steps"></a>Pasos siguientes

- [Tareas posteriores a la implementación](./openshift-container-platform-3x-post-deployment.md)
- [Solución de problemas de implementación de OpenShift en Azure](./openshift-container-platform-3x-troubleshooting.md)
- [Introducción a OpenShift Container Platform](https://docs.openshift.com)
- 
