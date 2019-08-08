# Parte1: Construcción de una red de blockchain de Hyperledger Fabric usando Amazon Managed Blockchain

Esta sección construirá una red blockchain de Hyperledger Fabric usando Amazon Managed Blockchain. Se utilizará una combinación de la consola y la CLI de AWS. El proceso para crear la red es el siguiente:

* Creación de una instancia de AWS Cloud9.
* Crear una red Fabric, un miembro y un nodo de dicho miembro a traves de la consola Amazon Managed Blockchain
* Desde Cloud9, despliegue de un template de AWS CloudFormation para crear una VPC y un nodo cliente de la red.
* Desde el nodo del cliente Fabric, se creará un canal, se instalará el chaincode, y ejecutarán transacciones en la red.

## Requisitos previos - AWS Cloud9

1. Inicie un ambiente [Cloud9 IDE](https://us-east-1.console.aws.amazon.com/cloud9/home?region=us-east-1) desde la consola de AWS. Haga clic en 'Crear entorno'. Usar 'us-east-1'.
2. Proporcione un nombre para su entorno, p. Ej. fabric-c9 y haga clic en ** Siguiente paso **
3. Seleccione `Otro tipo de instancia`, luego seleccione` t2.medium` y haga clic en ** Siguiente paso **
4. Haga clic en ** Crear entorno **. Por lo general, tomaría entre 30 y 60 segundos crear su IDE Cloud9
5. En el terminal Cloud9, en el directorio de inicio, clone este repositorio:

```
cd ~
git clone https://github.com/aws-samples/non-profit-blockchain.git
```

Actualice su AWS CLI a la última versión.

```
sudo pip install awscli --upgrade
```

## Paso 1 - Crear la red de blockchain de Hyperledger Fabric
En la consola de Amazon Managed Blockchain: https://console.aws.amazon.com/managedblockchain

Asegúrese de estar en la región de AWS correcta (es decir, us-east-1, también conocida como N. Virginia) y siga los pasos a continuación:

1. Haga clic en `Crear una red`
2. Asegúrese de que esté seleccionado `Hyperleger Fabric 1.2`
3. Ingrese un nombre de red y una descripción opcional, y haga clic en `Siguiente`. No use caracteres especiales en el nombre de la red, ya que este nombre se usa como prefijo al crear recursos en el paso 3.
4. Ingrese un nombre de miembro (por ejemplo, este podría ser el nombre de la organización a la que pertenece) y una descripción opcional
5. Ingrese un nombre de usuario y contraseña de administrador y guardelos. Lo necesitarás más tarde. Haga clic en "Siguiente"
6. Verifique su configuración y haga clic en `Crear red y miembro`
7. Espere hasta que el estado de su red y su miembro estén disponibles.

Antes de continuar, verifique que su red Fabric se haya creado y esté disponible. Si no,
espera a que se complete. De lo contrario, los pasos a continuación pueden fallar.

## Paso 2 - Crea el nodo del miembro
En la consola de Amazon Managed Blockchain: https://console.aws.amazon.com/managedblockchain

1. En la nueva red que ha creado, haga clic en el miembro en la sección Miembros.
2. Haga clic en `Crear nodo`
3. Deje los valores predeterminados y haga clic en `Crear nodo`

Continuaremos con los siguientes pasos mientras esperamos que el nodo par esté disponible.

## Paso 3 - Crear el nodo del cliente Fabric
En su ventana de terminal Cloud9.

Cree el nodo de cliente de Fabric, que alojará la CLI de Fabric. Utilizará la CLI para administrar
La red Fabric. El nodo del cliente Fabric se creará en su propia VPC en su cuenta de AWS, con VPC endpoints
apuntando a la red Fabric que creó en [Parte 1](../ngo-fabric/README.md).

El template de AWS CloudFormation requiere una serie de valores de parámetros. Nos aseguraremos de que estos
están disponibles como variables de entorno antes de ejecutar el siguiente script.

En Cloud9:

```
export REGION=us-east-1
export NETWORKID=<id asignada a la red>
export NETWORKNAME=<nombre de la red>
```

Defina el VPC endpoint. Asegúrese de que se haya completado y exportado.

```
export VPCENDPOINTSERVICENAME=$(aws managedblockchain get-network --region $REGION --network-id $NETWORKID --query 'Network.VpcEndpointServiceName' --output text)
echo $VPCENDPOINTSERVICENAME
```

Si la salida del último echo está en blanco revise bien los valores de las variables de entorno creadas anteriormente.

Si el VPC endpoint se llena con un valor, continúe y ejecute este script. Esto creará el
Pila de CloudFormation. Verá un error que dice `par de claves no existe`. Esto se espera como el script
comprobará si el par de claves existe antes de crearlo. No quiero sobrescribir ninguna existente
pares de llaves que tiene, así que simplemente ignore este error y deje que el script continúe:

```
cd ~/non-profit-blockchain/ngo-fabric
./3-vpc-client-node.sh
```

Verifique el progreso en la consola de AWS CloudFormation y espere hasta que la pila esté CREAR COMPLETA.
Encontrará información útil en la pestaña Salidas de la pila de CloudFormation una vez que la pila
Esta completo. Usaremos esta información en el paso posterior.

## Paso 4 - Prepare el nodo del cliente Fabric e inscriba una identidad
En el nodo del cliente Fabric.

Antes de ejecutar cualquier comando en el nodo del cliente Fabric, deberá exportar variables ENV
que proporcionan un contexto a Hyperledger Fabric. Estas variables le dirán al nodo cliente qué par
nodo con el que interactuar, qué certificados TLS usar, etc.

Desde Cloud9, SSH en el nodo del cliente Fabric. La clave (es decir, el archivo .PEM) debe estar en su directorio de inicio.
El DNS de la instancia EC2 del nodo del cliente Fabric se puede encontrar en la salida de CloudFormation que creado en el paso 3 anterior.

Responda 'sí' si se le solicita: `¿Está seguro de que desea continuar conectándose (sí / no)`

```
cd ~
ssh ec2-user@<dns of EC2 instance> -i ~/<Fabric network name>-keypair.pem
```

Clone el repositorio:

```
cd ~
git clone https://github.com/aws-samples/non-profit-blockchain.git
```

Cree el archivo que incluye los valores de exportación ENV que definen su configuración de red de Fabric.

```
cd ~/non-profit-blockchain/ngo-fabric
cp templates/exports-template.sh fabric-exports.sh
vi fabric-exports.sh
```

Actualice las declaraciones de exportación en la parte superior del archivo. La información debe coincidir con la ingresada al crear la red en [Parte 1](../ngo-fabric/README.md), o se puede encontrar en la consola de Amazon Managed Blockchain, debajo de su red.

Ejecute el archivo y las exportaciones se aplicaran a su sesión actual. Si sales de la SSH
sesión y vuelva a conectar, deberá volver a ejecutar el archivo.

```
cd ~/non-profit-blockchain/ngo-fabric
source fabric-exports.sh
```

Obtener el archivo hará dos cosas:
* exportar las variables ENV necesarias
* cree otro archivo que contenga los valores de exportación que necesita usar cuando trabaje con un nodo similar de Fabric.
Esto se puede encontrar en el archivo: `~ / peer-exports.sh`. Verá cómo usar esto en un paso posterior.

Compruebe que el `source` funcionó:

```
$ echo $PEERSERVICEENDPOINT
nd-4MHB4EKFCRF7VBHXZE2ZU4F6GY.m-B7YYBFY4GREBZLPCO2SUS4GP3I.n-WDG36TTUD5HEJORZUPF4REKMBI.managedblockchain.us-east-1.amazonaws.com:30003
```

Compruebe que el archivo de exportación de pares existe y que contiene una serie de claves de exportación con valores:

```
cat ~/peer-exports.sh 
```

Si el archivo tiene valores para todas las claves, búsquelo:

```
source ~/peer-exports.sh 
```

Obtenga la última versión del archivo Managed Blockchain PEM. Esto sobrescribirá el archivo existente en el directorio de inicio con la última versión de este archivo:

```
aws s3 cp s3://us-east-1.managedblockchain/etc/managedblockchain-tls-chain.pem  /home/ec2-user/managedblockchain-tls-chain.pem
```

Registre una identidad de administrador con Fabric CA (autoridad de certificación). Usaremos esta
identidad para administrar la red y realizar tareas como crear canales e instalar chaincode.

```
export PATH=$PATH:/home/ec2-user/go/src/github.com/hyperledger/fabric-ca/bin
cd ~
fabric-ca-client enroll -u https://$ADMINUSER:$ADMINPWD@$CASERVICEENDPOINT --tls.certfiles /home/ec2-user/managedblockchain-tls-chain.pem -M /home/ec2-user/admin-msp 
```

Es necesaria una copia final de los certificados:

```
mkdir -p /home/ec2-user/admin-msp/admincerts
cp ~/admin-msp/signcerts/* ~/admin-msp/admincerts/
cd ~/non-profit-blockchain/ngo-fabric
```

## Paso 5 - Actualice la configuración del canal configtx
En el nodo del cliente Fabric.

Actualice la configuración del canal configtx. Los campos Nombre e ID deben actualizarse con la ID de miembro.
Puede obtener la identificación de miembro de la consola Amazon Managed Blockchain Console o de las variables ENV
exportado a su sesión actual.

```
echo $MEMBERID
```

Actualice el archivo configtx.yaml. Asegúrese de editar el archivo configtx.yaml que copia a su hogar
directorio a continuación, NO el del repositorio:

```
cp ~/non-profit-blockchain/ngo-fabric/configtx.yaml ~
vi ~/configtx.yaml
```

Genere la configuración del canal configtx ejecutando el siguiente script. Cuando se crea el canal, esta configuración de canal se convertirá en el bloque de génesis (es decir, el bloque 0) en el canal:

```
docker exec cli configtxgen -outputCreateChannelTx /opt/home/$CHANNEL.pb -profile OneOrgChannel -channelID $CHANNEL --configPath /opt/home/
```

Debería ver:

```
2018-11-26 21:41:22.885 UTC [common/tools/configtxgen] main -> INFO 001 Loading configuration
2018-11-26 21:41:22.887 UTC [common/tools/configtxgen] doOutputChannelCreateTx -> INFO 002 Generating new channel configtx
2018-11-26 21:41:22.887 UTC [common/tools/configtxgen/encoder] NewApplicationGroup -> WARN 003 Default policy emission is deprecated, please include policy specificiations for the application group in configtx.yaml
2018-11-26 21:41:22.887 UTC [common/tools/configtxgen/encoder] NewApplicationOrgGroup -> WARN 004 Default policy emission is deprecated, please include policy specificiations for the application org group m-BHX24CQGP5CUNFS3YZTO2MPSRI in configtx.yaml
2018-11-26 21:41:22.888 UTC [common/tools/configtxgen] doOutputChannelCreateTx -> INFO 005 Writing new channel tx
```

Verifique que se haya generado la configuración del canal:

```
ls -lt ~/$CHANNEL.pb 
```

## Paso 6 - Crea un canal en la red
En el nodo del cliente Fabric.

Crea un canal en la red.

Ejecute el siguiente script:

```
docker exec -e "CORE_PEER_TLS_ENABLED=true" -e "CORE_PEER_TLS_ROOTCERT_FILE=/opt/home/managedblockchain-tls-chain.pem" \
    -e "CORE_PEER_ADDRESS=$PEER" -e "CORE_PEER_LOCALMSPID=$MSP" -e "CORE_PEER_MSPCONFIGPATH=$MSP_PATH" \
    cli peer channel create -c $CHANNEL -f /opt/home/$CHANNEL.pb -o $ORDERER --cafile $CAFILE --tls --timeout 900s
```

Debería ver:

```
2018-11-26 21:41:29.684 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2018-11-26 21:41:29.752 UTC [cli/common] readBlock -> INFO 002 Got status: &{NOT_FOUND}
2018-11-26 21:41:29.761 UTC [channelCmd] InitCmdFactory -> INFO 003 Endorser and orderer connections initialized
2018-11-26 21:41:29.963 UTC [cli/common] readBlock -> INFO 004 Got status: &{NOT_FOUND}
2018-11-26 21:41:29.972 UTC [channelCmd] InitCmdFactory -> INFO 005 Endorser and orderer connections initialized
2018-11-26 21:41:30.174 UTC [cli/common] readBlock -> INFO 006 Got status: &{NOT_FOUND}
2018-11-26 21:41:34.370 UTC [cli/common] readBlock -> INFO 026 Received block: 0
```

Esto creará un archivo llamado `mychannel.block` en el contenedor en la ruta ` /opt/home/fabric-samples/chaincode/hyperledger/fabric/peer`. Dado que este directorio está montado desde el host, es posible ver el archivo de bloque alli:

```
ls -lt /home/ec2-user/fabric-samples/chaincode/hyperledger/fabric/peer
```

Si se agota el tiempo de creación del canal, es posible que el canal se haya creado y que pueda obtener
el bloque. Al ejecutar el siguiente comando, se leerá la configuración del canal y se guardará el
bloque de génesis en el mismo directorio mencionado anteriormente:

```
docker exec -e "CORE_PEER_TLS_ENABLED=true" -e "CORE_PEER_TLS_ROOTCERT_FILE=/opt/home/managedblockchain-tls-chain.pem"  \
    -e "CORE_PEER_ADDRESS=$PEER"  -e "CORE_PEER_LOCALMSPID=$MSP" -e "CORE_PEER_MSPCONFIGPATH=$MSP_PATH" \
    cli peer channel fetch oldest /opt/home/fabric-samples/chaincode/hyperledger/fabric/peer/$CHANNEL.block \
    -c $CHANNEL -o $ORDERER --cafile /opt/home/managedblockchain-tls-chain.pem --tls   
```

Compruebe que el archivo de bloque ahora existe:

```
ls -lt /home/ec2-user/fabric-samples/chaincode/hyperledger/fabric/peer
```

## Paso 7: une tu nodo al canal
En el nodo del cliente Fabric.

Únase al canal de la red.

Ejecute el siguiente script:

```
docker exec -e "CORE_PEER_TLS_ENABLED=true" -e "CORE_PEER_TLS_ROOTCERT_FILE=/opt/home/managedblockchain-tls-chain.pem" \
    -e "CORE_PEER_ADDRESS=$PEER" -e "CORE_PEER_LOCALMSPID=$MSP" -e "CORE_PEER_MSPCONFIGPATH=$MSP_PATH" \
    cli peer channel join -b $CHANNEL.block  -o $ORDERER --cafile $CAFILE --tls
```

Debería ver:

```
2018-11-26 21:41:40.983 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2018-11-26 21:41:41.022 UTC [channelCmd] executeJoin -> INFO 002 Successfully submitted proposal to join channel
```

## Paso 8 - Instale chaincode en su nodo par
En el nodo del cliente Fabric.

Instale chaincode en el nodo cliente.

Ejecute el siguiente script:

```
docker exec -e "CORE_PEER_TLS_ENABLED=true" -e "CORE_PEER_TLS_ROOTCERT_FILE=/opt/home/managedblockchain-tls-chain.pem" \
    -e "CORE_PEER_ADDRESS=$PEER" -e "CORE_PEER_LOCALMSPID=$MSP" -e "CORE_PEER_MSPCONFIGPATH=$MSP_PATH" \
    cli peer chaincode install -n $CHAINCODENAME -v $CHAINCODEVERSION -p $CHAINCODEDIR
```

Debería ver:

```
2018-11-26 21:41:46.585 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-11-26 21:41:46.585 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
2018-11-26 21:41:48.004 UTC [chaincodeCmd] install -> INFO 003 Installed remotely response:<status:200 payload:"OK" > 
```

## Paso 9 - Inicializar el chaincode en el canal
En el nodo del cliente Fabric.

Instale el chaincode en el canal Fabric. Esta declaración puede demorar alrededor de 30 segundos y usted
No verá una respuesta específica de éxito.

Ejecute el siguiente script:

```
docker exec -e "CORE_PEER_TLS_ENABLED=true" -e "CORE_PEER_TLS_ROOTCERT_FILE=/opt/home/managedblockchain-tls-chain.pem" \
    -e "CORE_PEER_ADDRESS=$PEER" -e "CORE_PEER_LOCALMSPID=$MSP" -e "CORE_PEER_MSPCONFIGPATH=$MSP_PATH" \
    cli peer chaincode instantiate -o $ORDERER -C $CHANNEL -n $CHAINCODENAME -v $CHAINCODEVERSION \
    -c '{"Args":["init","a","100","b","200"]}' --cafile $CAFILE --tls
```

Debería ver:

```
2018-11-26 21:41:53.738 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-11-26 21:41:53.738 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
```

## Paso 10 - Consulta el chaincode
En el nodo del cliente Fabric.

Consulta el chaincode en el nodo cliente.

Ejecute el siguiente script:

```
docker exec -e "CORE_PEER_TLS_ENABLED=true" -e "CORE_PEER_TLS_ROOTCERT_FILE=/opt/home/managedblockchain-tls-chain.pem" \
    -e "CORE_PEER_ADDRESS=$PEER" -e "CORE_PEER_LOCALMSPID=$MSP" -e "CORE_PEER_MSPCONFIGPATH=$MSP_PATH" \
    cli peer chaincode query -C $CHANNEL -n $CHAINCODENAME -c '{"Args":["query","a"]}' 
```

Debería ver:

```
100
```

## Paso 11 - Invocar una transacción
En el nodo del cliente.

Invocar una transacción en la red.

Ejecute el siguiente script:

```
docker exec -e "CORE_PEER_TLS_ENABLED=true" -e "CORE_PEER_TLS_ROOTCERT_FILE=/opt/home/managedblockchain-tls-chain.pem" \
    -e "CORE_PEER_ADDRESS=$PEER" -e "CORE_PEER_LOCALMSPID=$MSP" -e "CORE_PEER_MSPCONFIGPATH=$MSP_PATH" \
    cli peer chaincode invoke -o $ORDERER -C $CHANNEL -n $CHAINCODENAME \
    -c '{"Args":["invoke","a","b","10"]}' --cafile $CAFILE --tls
```

Debería ver:

```
2018-11-26 21:45:20.935 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 001 Chaincode invoke successful. result: status:200 
```

## Paso 12 - Consulta el chaincode nuevamente y verifica el cambio en el valor
En el nodo del cliente Fabric.

Consulte el chaincode en el nodo de la red y verifique el cambio de valor. Esto demuestra el éxito de la invocación. Si ejecuta la consulta inmediatamente después de la invocación, puede notar que los datos no han cambiado.
¿Alguna idea de por qué? Debe haber un espacio de (aproximadamente) 2 segundos entre la invocación y la consulta.

Invocar una transacción en Fabric implica una serie de pasos, que incluyen:

* Envío de la transacción a los pares que lo respaldan para simulación y respaldo
* Empaquetado de los avales de los pares
* Envío de avales empaquetados al servicio de orden para realizar el ordenamiento
* El servicio de orden agrupa las transacciones en bloques (que se crean cada 2 segundos, por defecto)
* El servicio de orden envía los bloques a todos los nodos pares para validarlos y confirmarlos en el libro mayor

Solo después de que las transacciones en el bloque se hayan comprometido con el libro mayor, puede leer el
nuevo valor del libro mayor (o más específicamente, del almacén de valores clave del estado mundial).

Ejecute el siguiente script:

```
docker exec -e "CORE_PEER_TLS_ENABLED=true" -e "CORE_PEER_TLS_ROOTCERT_FILE=/opt/home/managedblockchain-tls-chain.pem" \
    -e "CORE_PEER_ADDRESS=$PEER" -e "CORE_PEER_LOCALMSPID=$MSP" -e "CORE_PEER_MSPCONFIGPATH=$MSP_PATH" \
    cli peer chaincode query -C $CHANNEL -n $CHAINCODENAME -c '{"Args":["query","a"]}' 
```

Debería ver:

```
90
```

## Pase a la Parte 2
Las instrucciones del taller se pueden encontrar en los archivos README en las partes 1-4:

* [Parte 1:](../ngo-fabric/README.md) Comience el taller construyendo la red de blockchain Hyperledger Fabric usando Amazon Managed Blockchain.
* [Parte 2:](../ngo-chaincode/README.md) Implemente el chaincode sin fines de lucro.
* [Parte 3:](../ngo-rest-api/README.md) Ejecute el servidor RESTful API.
* [Parte 4:](../ngo-ui/README.md) Ejecute la aplicación.
* [Parte 5:](../new-member/README.md) Agrega un nuevo miembro a la red.