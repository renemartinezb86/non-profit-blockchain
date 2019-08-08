# Amazon Managed Blockchain Workshop

! [Amazon Managed Blockchain] (images / AmazonManagedBlockchain.png "Amazon Managed Blockchain")

## Construyendo e implementando una aplicación para Hyperledger Fabric en Amazon Managed Blockchain

Se necesita un usuario con permisos para lanzar instancias de EC2, ambientes de Cloud9, creación de redes de Blockchain y lectura desde S3.

Este taller construirá una red de blockchain Hyperledger Fabric usando Amazon Managed Blockchain, implementará el chaincode, iniciará el servidor de API y finalmente se usará una aplicación de interfaz de usuario que consume los servicios de la API RESTful para interactuar con red. 

El caso de uso es un sistema de seguimiento de donaciones y uso de los fondos de una ONG.

La aplicación de 3 capas consta de los siguientes componentes:
1. UI en Node.js / Aplicación de interfaz de usuario angular, acceso a servicios proporcionados via API.
2. API RESTful, que se ejecuta como una aplicación Node.js Express, utiliza el SDK del cliente Hyperledger Fabric para realizar consultas e invocar chaincode.
3. Fabric Chaincode, escrito en Node.js, implementado en una red Hyperledger Fabric.


El taller se divide en cuatro partes:

1. Construir una red de blockchain Hyperledger Fabric usando Amazon Managed Blockchain. Las instrucciones se pueden encontrar en la carpeta: [ngo-fabric] (ngo-fabric)
2. Implementar el chaincode, o contrato inteligente, que proporciona la funcionalidad de donación y seguimiento de gastos. Las instrucciones se pueden encontrar en la carpeta: [ngo-chaincode] (ngo-chaincode)
3. Inicio del servidor RESTful API que expone las funciones de chaincode a las aplicaciones cliente. Las instrucciones se pueden encontrar en la carpeta: [ngo-rest-api] (ngo-rest-api)
4. Ejecutando la aplicación de interfaz de usuario. Las instrucciones se pueden encontrar en la carpeta: [ngo-ui] (ngo-ui)

## Empezando

Para construir la red, implemente el chaincode, inicie el servidor RESTful API y ejecute la aplicación, siga las instrucciones en este orden:

* [Parte 1:] (ngo-fabric / README.md) Comience el taller construyendo la red de blockchain Hyperledger Fabric usando Amazon Managed Blockchain.
* [Parte 2:] (ngo-chaincode / README.md) Implemente el chaincode sin fines de lucro.
* [Parte 3:] (ngo-rest-api / README.md) Ejecute el servidor API RESTful.
* [Parte 4:] (ngo-ui / README.md) Ejecute la aplicación.
* [Parte 5:] (nuevo miembro / README.md) Agregue un nuevo miembro a la red.

## Limpiar

Para limpiar sus recursos, elimine la red Hyperledger Fabric administrada por Amazon Managed Blockchain y la plantilla AWS CloudFormation de la siguiente manera:

* En la consola de AWS CloudFormation, elimine la pila con el nombre de pila `<su red> -fabric-client-node`
* En la consola Amazon Managed Blockchain, elimine el miembro de su red. Esto eliminará el nodo par, el miembro y, finalmente, la red Fabric (suponiendo que haya creado un solo miembro)
* En la consola de AWS Cloud9, elimine su instancia de AWS Cloud9

## Licencia

Esta biblioteca tiene licencia bajo la licencia Apache 2.0.