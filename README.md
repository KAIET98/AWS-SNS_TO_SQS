# AWS-SNS_TO_SQS
El objetivo de este repositorio es construir una vía donde los mensajes de SNS lleguen a un queue de SQS 


# 1. Creacion de un SQS y un SNS paralelos

## Primero de todo creamos un SQS de la siguiente forma

Es muy importante que a parte del nombre que le demos, que el tipo de SQS sea de tipo FIFO, sino no podremos culminar el tutorial. Nombre: 

```
sns_to_sqs.fifo
```
Todas las demás configuraciones las dejamos en default por ahora. 

## Creación de un SNS

Vamos a crear un topic y para tener un poco de control vamos a crearlo en otra ventana del navegador.

En el topic name también tiene que ser de tipo FIFO, sino no va a funcionar.

```
SNS_FOR_SQS.fifo
```
y dejando las demás configruaciones sin más, creamos el topic.

## Copiamos las ARNS. 

Vamos a tener que editar las policies por lo que: 

### Entramos a nuestro SQS y copiamos la arn tal que así:

```
arn:aws:sqs:us-east-1:070307590085:sns_to_sqs.fifo
```

### SNS copiamos la ARN

EL mecanismo va a ser lo mismo:

```
arn:aws:sns:us-east-1:070307590085:SNS_FOR_SQS.fifo
```

# 2. Delegación de permisos. 

Ahora, tenemos que dar la posibilidad a SNS de enviar un mensaje es decir, de incluir la acción de 

```
sqs:SendMessage
```
Para ello vamos a SQS , entramos en nuestro "Access Policy" y le damos a "Edit".
Actualmente tendremos algo como esto:

```
{
  "Version": "2008-10-17",
  "Id": "__default_policy_ID",
  "Statement": [
    {
      "Sid": "__owner_statement",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::070307590085:root"
      },
      "Action": "SQS:*",
      "Resource": "arn:aws:sqs:us-east-1:070307590085:sns_to_sqs.fifo"
    }
  ]
}
```
Pues, vamos a tener que cambiar el Statement para que, el SNS escriba en SQS algo parecido a esto, bueno poniendolo fácil tendriamos que hacer un fill-in the gaps:

```
"Statement": [{
    "Effect":"Allow",
    "Principal": {
      "Service": "sns.amazonaws.com"
    },
    "Action":"sqs:SendMessage",
    "Resource":"<ARN DEL SQS>",
    "Condition":{
      "ArnEquals":{
        "aws:SourceArn":"< ARN DEL SNS >"
      }
    }
  }]
```
Nos quedaría algo como esto:
```
{
  "Version": "2008-10-17",
  "Id": "__default_policy_ID",
  "Statement": [{
    "Effect":"Allow",
    "Principal": {
      "Service": "sns.amazonaws.com"
    },
    "Action":"sqs:SendMessage",
    "Resource":"arn:aws:sqs:us-east-1:070307590085:sns_to_sqs.fifo",
    "Condition":{
      "ArnEquals":{
        "aws:SourceArn":"arn:aws:sns:us-east-1:070307590085:SNS_FOR_SQS.fifo"
      }
    }
  }]
}
```
Y así le damos a "Save Changes".

# 3. Creación de link entre servicios:

1. Vamos a SNS a nuestro topic y creamos una suscripción
2. En protocolo seleccionamos: SQS
3. En endpoint vamos a tener que poner la Arn de nuestro SQS: 

```
arn:aws:sqs:us-east-1:070307590085:sns_to_sqs.fifo

```
4. Crear subscripción.


# Prueba de concepto

Vamos a SNS y le damos a "Publish Message"

En subject ponemos algo fácil

```
trial
```
Message Group ID

```
1
```
Message deduplication ID
```
1
```
En el Message Body ponemos un 
```
Esto es un mensaje de prueba
```
Y publicamos el mensaje

Si todo va bien, accedemos a SQS, a nuestro QUEUE y le damos a "Send and Receive Messages". 
A continuación clickamos en "Poll for messages"; y si todo va bien tenmos que tener un mensaje. 

Clickamos en el "box" y al darle el View Details, en el body, tendriamos que ver el mensaje que acabamos de publicar en el JSON.

:)
