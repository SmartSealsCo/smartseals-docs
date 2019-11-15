---
title: Montaje CD/CI integrando Bitbucket y Amazon CodeCommit, CodeDeploy y CodePipeline en Amazon Elastic Load Balancer (EC2)
keywords: aws cd-ci codecommit codedeploy codepipeline
summary: "Aprende a integrar CC, CD y CP con Bitbucket"
sidebar: mydoc_sidebar
permalink: montaje_cd-ci_aws.html
folder: mydoc
---

# Resumen

En este post se encontrara toda la informacion y los proceso respectivos para el montaje de un servicio CD/CI con el objetivo de montar un servicio capaz de administrar automaticamente la actualizacion de los codigos, la sobrecarga y la caida de los servicios asociados.


>**Nota**: Este post tomara en cuenta que ya se tiene creado un repositorio de Bitbucket y ya se tiene configurado el repositorio del EC2 con SSH.

---

## **1. Configuración inicial**

Como primera tarea se deben crear el repositorio y las credenciales para lograr una correcta y segura conexion de los servicios.
- ### Creacion de repositorio en CodeCommit
    Para la creacion del repositorio primero hay que ingresar con la cuenta de la consola de AWS e ingresar al servicio de CodeCommit, el cual se accede entrando a este link de [aquí.](https://us-west-2.console.aws.amazon.com/codesuite/codecommit)

    Una vez ingresado se dara click al boton _"Crear El Repositorio"_.
    ![Imagen_1-1-1](/images/aws/cd-ci/1-1-1.png){:class="center_image"}
    
    Cuando se inserte toda la informacion del repositorio se dara en el boton crear.
    ![Imagen_1-1-2](/images/aws/cd-ci/1-1-2.png){:class="center_image"}
    
    Y con eso ya se tendra un repositorio CodeCommit listo para trabajar.
    
- ### Generacion de claves SSH
    Para lograr una conexion exitosa entre BitBucket y CodeCommit se necesitan generar claves para asociarlas a los repositorios, para eso primero generaremos las claves SSH, esto lo logramos con el comando
    >`ssh-keygen -f ~/.ssh/codecommit_rsa`
    
    Este comando creara los archivos `codecommit_rsa` (clave privada) y `codecommit_rsa.pub`(clave publica) en la carpeta especificada, que en este caso es `~/.ssh`
    
    Ahora se asociara la clave SSH publica con el servicio de AWS, para eso ingresaremos a las credenciales IAM de CodeCommit con [esta url](https://console.aws.amazon.com/iam/home?#/security_credentials?credentials=codecommit) y accederemos a _"Cargar una llave publica de SSH"_
    ![Imagen_1-2-1](/images/aws/cd-ci/1-2-1.png){:class="center_image"}

    Una vez alli se procedera a copiar la llave publica de nuestro archivo `codecommit_rsa.pub`, esto puede ser por el medio que se desee, en este caso usamos el comando `cat ~/.ssh/codecommit_rsa.pub`. Esta clave se pegara en la ventana que abrio AWS y le presionara el boton _"Cargar una clave publica de SSH"_.
    ![Imagen_1-2-2](/images/aws/cd-ci/1-2-2.png){:class="center_image"}
    
    Esto nos devolvera una ID de clave unica que podemos usar para generar el archivo de configuracion.
    ![Imagen_1-2-3](/images/aws/cd-ci/1-2-3.png){:class="center_image"}
- ### Generacion de archivo de configuración
    
    Una vez generada la ID de clave de IAM para CodeCommit se procede a crear el archivo de configuracion, este lo guardaremos en la carpeta `~/.ssh`, lo llamaremos `config` y copiaremos el siguiente archivo: 

    _**~/.ssh/config**_ (Reemplazar la clave IAM por la generada en AWS)
    
      Host git-codecommit.*.amazonaws.com
          User <Your-IAM-SSH-Key-ID-Here>
          IdentityFile ~/.ssh/codecommit_rsa
    
## **2. Conectando Bitbucket con CodeCommit**
Ahora que tenemos las variables toca realizar la conexion entre BitBucket y CodeCommit para eso primero tenemos que definir las variables del repositorio.
- ### Configuracion de las variables necesarias en BitBucket
    Para asignar las variables primero hay que acceder a _configuraciones --> variables del repositorio_ en nuestro repositorio y se procedera a crear 5 variables:
    
    - **CodeCommitUser**: EL ID de clave SSH generado por IAM
    - **CodeCommitRepo**: La url de nuestro repositorio CodeCommit
    - **CodeCommitConfig**: Nuestro archivo de configuracion (`~/.ssh/config`) en base 64
        > Para convertir el archivo en base 64 ejecutar el comando `cat ~/.ssh/config | base64 -w 0`, eso nos mostrara en consola el archivo ya codificado. Copiamos ese dato y lo pegamos en la variable

    - **CodeCommitKey**: Nuestra archivo de clave privada (`~/.ssh/codecommit_rsa`) en base 64
        > Para convertir el archivo en base 64 ejecutar el comando `cat ~/.ssh/codecommit_rsa | base64 -w 0`, eso nos mostrara en consola el archivo ya codificado. Copiamos ese dato y lo pegamos en la variable
        
    - **CodeCommitHost**: La url de nuestro Host de CodeCommit
        > Si nuestro repositorio es `https://git-codecommit.us-west-2.amazonaws.com/v1/repos/test_repo` entonces nuestro host sera `https://git-codecommit.us-west-2.amazonaws.com`

    ![Imagen_2-1-1](/images/aws/cd-ci/2-1-1.png){:class="center_image"}
    
- ### Configuracion del archivo yaml para envio del repositorio a CodeCommit
    Una vez que tenemos las variables del repositorio en BitBucket, procederemos a ir a la ventana de Pipelines en el repositorio. Aqui dentro modificaremos el YAML e insertaremos los siguientes comandos y daremos en _"Commit file"_:

      pipelines:
        default:
          - step:
              script:
                - echo $CodeCommitKey > ~/.ssh/codecommit_rsa.tmp
                - base64 -d ~/.ssh/codecommit_rsa.tmp > ~/.ssh/codecommit_rsa
                - chmod 400 ~/.ssh/codecommit_rsa
                - echo $CodeCommitConfig > ~/.ssh/config.tmp
                - base64 -d  ~/.ssh/config.tmp > ~/.ssh/config
                - set +e
                - ssh -o StrictHostKeyChecking=no $CodeCommitHost
                - set -e
                - git remote add codecommit ssh://$CodeCommitRepo
                - git push codecommit master
    
    Basicamente lo que sucede aca es que estamos replicando los archivos de nuestra configuracion en la maquina del bitbucket, de esta forma podemos autenticarnos en el CodeCommit desde el BitBucket simulando que el usuario que esta ingresando es un usuario de Amazon.
    
    ![Imagen_2-2-1](/images/aws/cd-ci/2-2-1.png){:class="center_image"}
## **3. Configuracion CodeDeploy**
- ### Creación de la tarea CodeDeploy
- ### Insercion de scripts de la tarea en el repositorio
## **4. Configuracion CodePipeline**
- ### Seleccion de tarea CodeDeploy a Ejecutar
- ### Ejecucion del servicio
