# OSDCloud

[OSDCloud](https://osdcloud.osdeploy.com/) es un concepto creado por [David Segura](https://twitter.com/SeguraOSD) para describir el **despliegue de un sistema operativo Windows a través de Internet** utilizando el entorno [WinPE](https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/winpe-intro) y su módulo PowerShell [OSD](https://osd.osdeploy.com/).

## Instalación de OSD

> **Nota**:  Si se utiliza [OSDBuilder](https://github.com/manelrodero/osdbuilder), el módulo OSD ya estará instalado en el equipo.

La instalación de OSD se realiza de la siguiente manera:

* Ejecutar PowerShell desde una cuenta con permisos de Administrador
* Ejecutar el _cmdlet_ `Install-Module` para instalar desde la [PowerShell Gallery](https://www.powershellgallery.com/):

```PowerShell
Install-Module -Name OSD -Force
```

> **Nota**:  Si hubiese algún problema para descargar el módulo es recomendable revisar la configuración del **cifrado TLS 1.2 en PowerShell** (ver Anexo).

* Cerrar y abrir PowerShell para que cargue los nuevos módulos instalados

## Desinstalar OSD

Si por algún motivo es necesario desinstalar OSD, se puede ejecutar el siguiente comando:

```PowerShell
Uninstall-Module -Name OSD -AllVersions -Force
```

## Windows ADK

Es necesario tener instalado [Windows ADK](https://docs.microsoft.com/en-us/windows-hardware/get-started/adk-install) para disponer de las herramientas necesarias para construir el entorno WinPE.

## Crear un entorno OSDCloud

### Plantilla OSDCloud.template

El primer paso para crear un entorno OSDCloud es instalar la [última versión de OSD](https://www.powershellgallery.com/packages/OSD/21.4.9.2) usando los comandos explicados anteriormente.

A continuación se **copia el entorno WinPE incluido en Windows ADK** al directorio `$env:ProgramData\OSDCloud` y se añaden una serie de utilidades (`curl.exe`, soporte para PowerShell, Microsoft DaRT, etc.) que serán de utilidad a posteriori.

Este directorio, conocido como [**Universal WinPE**](https://osdcloud.osdeploy.com/concepts/universal-winpey), se utiliza como plantilla para crear el entorno OSDCloud final.

Para crear esta plantilla se ejecuta el comando `New-OSDCloud.template` al que se le pueden añadir opciones para incluir [lenguajes adicionales](https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/dism-languages-and-international-servicing-command-line-options) y tener el [teclado correcto](https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/default-input-locales-for-windows-language-packs):


```PowerShell
# Creación de la plantilla para OSDCloud

# English (por defecto)
New-OSDCloud.template -Verbose

# Se añade soporte para es-ES
New-OSDCloud.template -Language es-ES -Verbose

# Se añade soporte para es-ES y se hace el idioma por defecto
New-OSDCloud.template -Language es-ES -SetAllIntl es-es -Verbose
```

### Autopilot

Si se usa [Autopilot](https://docs.microsoft.com/en-us/mem/autopilot/windows-autopilot) se pueden añadir los ficheros `*.json` con los [Configuration Profiles](https://docs.microsoft.com/en-us/mem/autopilot/existing-devices) en el directorio `$env:ProgramData\OSDCloud\Autopilot\Profiles`.

### Espacio de trabajo OSDCloud.workspace

A continuación hay que crear un **espacio de trabajo** a partir de la plantilla anterior:

```PowerShell
New-OSDCloud.workspace -WorkspacePath V:\OSDCloud\Workspace -Verbose
```

### Añadir Office 365

Esta [opción](https://osdcloud.osdeploy.com/get-started/enable-osdcloudodt) aún está en fase experimental.

### Configurar el entorno WinPE

Las modificaciones del entorno WinPE permiten añadir **controladores** (de fabricantes como HP, Dell, Lenovo o de entornos _cloud_ como VMware, Nutanix y Dell).

```PowerShell
# Añadir drivers de fabricante
Edit-OSDCloud.winpe -DriverPath V:\OSDCloud\Drivers -Verbose

# Añadir drivers de cloud
Edit-OSDCloud.winpe -CloudDriver Dell,Nutanix,VMware -Verbose
Edit-OSDCloud.winpe -CloudDriver VMware -Verbose
```

### Crear ISO o USB

Finalmente, según se vaya a usar un entorno virtual o un entorno físico, habrá que crear un **fichero ISO** o escribir en una **llave USB**.

```PowerShell
# Crear fichero ISO
New-OSDCloud.iso

# Crear llave USB
New-OSDCloud.usb
```

> **Nota**:  Realmente se crean dos ficheros ISO, `OSDCloud.iso` y `OSDCloud_NoPrompt.iso`, siendo este último una versión en la que no se necesita pulsar una tecla para iniciar desde él.

El comando `New-OSDCloud.usb` listará los dispositivos USB conectados al equipo para seleccionar en cual se quiere copiar OSDCloud y se pedirá confirmación por ser un proceso destructivo.

A continuación se creará una partición NTFS y una partición FAT32 de 2GB al final del USB para [poder recrearla fácilmente si es necesario](https://osdcloud.osdeploy.com/get-started/new-osdcloud.usb) y se copiarán los ficheros necesarios.

### Añadir OS y DriverPack

Se puede utilizar el comando `Save-OSDCloud.usb` para descargar el **Sistema Operativo** y el **driver pack** correspondiente al equipo que se quiere instalar.

```PowerShell
# Windows 10 Education 
Save-OSDCloud.usb -OSEdition Education -OSLanguage es-es -Verbose

# Manufacturer/Product
# -Manufacturer HP, Dell, Lenovo, Microsoft
# -Product
```

Este comando lanzará un "asistente" mediante el cual se selecciona:

* La letra que contiene la unidad USB OSDCloud creada anteriormente
* La versión de Windows 10 (21H1, 20H2, etc.)
* La licencia que se aplicará (Retail, Volume)

### Driver Packs adicionales

Esta [opción](https://osdcloud.osdeploy.com/get-started/save-osdcloud.usb/saving-alternate-driverpacks) queda pendiente por explorar.

## Ejecutar OSDCloud

Una vez creado el fichero ISO o la llave USB se puede iniciar la máquina virtual o física donde queramos realizar la instalación.

Una vez haya iniciado el entorno WinPE habrá que ejecutar los siguientes comandos para cargar el módulo **OSD**:

```PowerShell
# Instalar el módulo OSD en Windows PE
Install-Module OSD -Force

# (Opcional) Cambiar la resolución
Set-DisRes 1600
```

A continuación se puede ejecutar el comando `Start-OSDCloud`:

```PowerShell
Start-OSDCloud -Screenshot
```

Si no se especifica ningún parámetro se lanzará un "asistente" mediante el cual se seleccionará:

* La versión de Windows 10 (21H1, 20H2, etc.)
* La edición (Education, Enterprise, etc.)
* La licencia (Retail, Volume)
* (Opcional) El perfil de Autopilot
* (Opcional) Office ODT
* Disco en el que se instalará el SO (es un proceso destructivo)

A partir de aquí comienza el proceso de "instalación":

* Se descarga el SO indicado (o uso del ya descargado en el USB)
* Se descargan los controladores
* Se descarga el fichero `Unattend.xml` para usar la fase **Specialize**
* (Opcional) Se copian los instaladores de Office/ODT al disco
* Se añaden los scripts en PowerShell necesarios

Finalmente, después de rato (8 minutos en un antiguo HP 840 G1), aparece el OOBE de Windows 10.

<iframe width="560" height="315" src="https://www.youtube.com/embed/xD06ewDKn5w" title="Test de OSDCloud en VMware Workstation Pro" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Anexo

### <a name="TLS12"></a>Usar TLS1.2 en PowerShell

Desde abril de 2020, [PowerShell Gallery únicamente soporta TLS 1.2](https://devblogs.microsoft.com/powershell/powershell-gallery-tls-support/) (TLS 1.0 y TLS 1.1 son obsoletos).

Por ese motivo, al intentar descargar módulos puede haber problemas (_Install-Package : No match was found..._) si no se especifica el protocolo correcto:

```PowerShell
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
```

Si se quiere que [TLS 1.2 sea el cifrado por defecto en el SO,](https://twitter.com/manelrodero/status/1305409972759560198) se pueden cambiar estas claves del registro:

```PowerShell
# set strong cryptography on 64 bit .Net Framework (version 4 and above)
Set-ItemProperty -Path 'HKLM:\SOFTWARE\Wow6432Node\Microsoft\.NetFramework\v4.0.30319' -Name 'SchUseStrongCrypto' -Value '1' -Type DWord

# set strong cryptography on 32 bit .Net Framework (version 4 and above)
Set-ItemProperty -Path 'HKLM:\SOFTWARE\Microsoft\.NetFramework\v4.0.30319' -Name 'SchUseStrongCrypto' -Value '1' -Type DWord 
```

