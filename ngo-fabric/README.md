# Parte1: Construcción de una red de blockchain de Hyperledger Fabric usando Amazon Managed Blockchain

Esta sección construirá una red blockchain de Hyperledger Fabric usando Amazon Managed Blockchain. Se utilizará una combinación de la consola de AWS y la CLI de AWS. El proceso para crear la red es el siguiente:

* Provisión de una instancia de AWS Cloud9. Utilizaremos el terminal de Linux que proporciona Cloud9
* Crear una red Fabric y aprovisionar un nodo par a traves de la consola Amazon Managed Blockchain
* Desde Cloud9, despliegue de una plantilla de AWS CloudFormation para aprovisionar una VPC y un nodo de cliente Fabric. El nodo del cliente Fabric es para administrar la red Fabric
* Desde el nodo del cliente Fabric, se creará un canal Fabric, se instalará el chaincode, y
ejecutarán transacciones en la red.

## Requisitos previos - AWS Cloud9
Usaremos AWS Cloud9 para proporcionar un terminal de Linux que ya tenga la AWS CLI instalada.

1. Inicie un ambiente [Cloud9 IDE](https://us-east-1.console.aws.amazon.com/cloud9/home?region=us-east-1) desde la consola de AWS.
En la consola de Cloud9, haga clic en 'Crear entorno'. Usar 'us-east-1'.
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
5. Ingrese un nombre de usuario y contraseña de administrador y anótelo. Lo necesitarás más tarde. Haga clic en "Siguiente"
6. Verifique su configuración y haga clic en `Crear red y miembro`
7. Espere hasta que el estado de su red y su miembro estén disponibles.

Antes de continuar, verifique que su red Fabric se haya creado y esté disponible. Si no,
espera a que se complete. De lo contrario, los pasos a continuación pueden fallar.

## Paso 2 - Crea el par de tela
En la consola de Amazon Managed Blockchain: https://console.aws.amazon.com/managedblockchain

1. En la nueva red que ha creado, haga clic en el miembro en la sección Miembros.
2. Haga clic en `Crear nodo igual`
3. Deje los valores predeterminados y haga clic en `Crear nodo igual`

Continuaremos con los siguientes pasos mientras esperamos que el nodo par esté disponible.

## Paso 3 - Crear el nodo del cliente Fabric
En su ventana de terminal Cloud9.

Cree el nodo de cliente de Fabric, que alojará la CLI de Fabric. Utilizará la CLI para administrar
La red Fabric. El nodo del cliente Fabric se creará en su propia VPC en su cuenta de AWS, con puntos finales VPC
apuntando a la red Fabric que creó en [Parte 1](../ngo-fabric/README.md). AWS CloudFormation
se utilizará para crear el nodo del cliente Fabric, la VPC y los puntos finales VPC.

La plantilla de AWS CloudFormation requiere una serie de valores de parámetros. Nos aseguraremos de que estos
están disponibles como variables de exportación antes de ejecutar el siguiente script.

En Cloud9:

```
REGIÓN de exportación = us-east-1
export NETWORKID = <la ID de red que creó en el Paso 1, desde la Consola de cadena de bloques administrada de Amazon>
export NETWORKNAME = <el nombre que le dio a la red>
```

Establecer el punto final VPC. Asegúrese de que se haya completado y exportado. Si la siguiente declaración `echo` muestra
que está en blanco, verifique los detalles debajo de su red en la Consola Amazon Blockchain administrada:

```
export VPCENDPOINTSERVICENAME = $ (aws managedblockchain get-network --region $ REGION --network-id $ NETWORKID - consulta 'Network.VpcEndpointServiceName' - texto de salida)
echo $ VPCENDPOINTSERVICENAME
```

Si el punto final de VPC se llena con un valor, continúe y ejecute este script. Esto creará el
Pila de CloudFormation. Verá un error que dice `par de claves no existe`. Esto se espera como el script
comprobará si el par de claves existe antes de crearlo. No quiero sobrescribir ninguna existente
pares de llaves que tiene, así que simplemente ignore este error y deje que el script continúe:

```
cd ~ / sin fines de lucro blockchain / ngo-fabric
./3-vpc-client-node.sh
```

Verifique el progreso en la consola de AWS CloudFormation y espere hasta que la pila esté CREAR COMPLETA.
Encontrará información útil en la pestaña Salidas de la pila de CloudFormation una vez que la pila
Esta completo. Usaremos esta información en el paso posterior