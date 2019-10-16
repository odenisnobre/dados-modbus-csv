# Objetivo

Atualmente na rede existe um arquivo `.csv` que contém estruturas de dados que são mostradas para o usuário.
Basicamente, o arquivo possui, obviamente, linhas e colunas. O objetivo é ler o índice da coluna e retonar os valores das colunas.


# Configuração

Para os testes foram utilizados:
1. **Node-Red - versão 1.0.1**
2. **CPU M580 - Schneider**

## Configuração PLC

Em aplicação convencional o valor de referência(índice da coluna) seria escrito via interface com o usuário - SCADA. No nosso caso o criamos um contador simples com reset para poder simular o envio dos dados via Modbus TCP.

A CPU M580 utilizada já vem embarcada com o protoco modbus TCP o que facilita a integração.


#### Configuração do Rack
<img src="https://github.com/dedynobre/dados-modbus-csv/blob/master/imagens/config-rack.png"/></br>

#### Lógica
<img src="https://github.com/dedynobre/dados-modbus-csv/blob/master/imagens/logica.png"/></br>

> O pino CV do contador `CTU_1` é o pino onde o Node-Red lê o índice para ser procurado no arquivo `.csv`.
> Esta variável no programa tem o endereço de %MW102, parametro este que é configurado nos nodes Modbus.</br>
> <img src="https://github.com/dedynobre/dados-modbus-csv/blob/master/imagens/Var.png"/>


A programção foi feito no Unity Pro v11.0.
O proejeto de exemplo se encontra na pasta `Projeto NodeRed`

## Configuração Node-Red

No Node-Red foi utilizado os nodes:
1. **Modbus**
2. **File**
3. **CSV**
4. **Function**
5. **Dashboard**

#### Criando conexão Modbus TCP

Foi adicionado um node Modbus para lê dados do controlador - Modbus Read:
> <img src="https://github.com/dedynobre/dados-modbus-csv/blob/master/imagens/config-modbus-01.png"/>

Primeiramente é preciso criar uma conexão com o servidor Modbus, que basicamente precisamos configurar o IP do controlador e porta, no nosso caso as condigurações são:
* IP: 192.168.238.50
* Porta: 502

> A porta padrão do Modbus TCP é *502*. Existem softwares que aceitam fazer alteração outros não.

A configuração completa deve ficar da seguinte forma:
> <img src="https://github.com/dedynobre/dados-modbus-csv/blob/master/imagens/config-modbus-02.png"/>


> Sempre ficar atento a comunicação entre o controlador e o Node-Red. O status do node já informa como está a conexão. Se for possível mantenha sempre *o ping ativo para checar a comunicação*.


#### Configurando a leitura de dados

Após feita a conexão com servidor temos que configurar a escrita, ficando da seguinte forma:
> <img src="https://github.com/dedynobre/dados-modbus-csv/blob/master/imagens/config-modbus-02.png"/>

Onde:
* **FC**: é a função. No nosso caso será FC3: Read Holding Register
* **Address**: Endereço para leitura
* **Quantity**: quantidade de registros para leitura
* **Poll Rate**: taxa de atualização de dados

O resultado de saida será sempre um array:
> <img src="https://github.com/dedynobre/dados-modbus-csv/blob/master/imagens/node-config-02.png"/>
> <img src="https://github.com/dedynobre/dados-modbus-csv/blob/master/imagens/node-config-01.png"/>

Código:</br>
	```
	[{"id":"7d851b28.c19994","type":"modbus-read","z":"7aaf1b1a.016644","name":"","topic":"","showStatusActivities":false,"logIOActivities":false,"showErrors":false,"unitid":"","dataType":"HoldingRegister","adr":"102","quantity":"1","rate":"500","rateUnit":"ms","delayOnStart":false,"startDelayTime":"","server":"f7d7534b.020d3","useIOFile":false,"ioFile":"","useIOForPayload":false,"x":250,"y":460,"wires":[["24d157b1.c80458","8f4ff681.9e7498"],[]]},{"id":"f7d7534b.020d3","type":"modbus-client","z":"","name":"M580","clienttype":"tcp","bufferCommands":true,"stateLogEnabled":false,"tcpHost":"192.168.238.50","tcpPort":"502","tcpType":"DEFAULT","serialPort":"/dev/ttyUSB","serialType":"RTU-BUFFERD","serialBaudrate":"9600","serialDatabits":"8","serialStopbits":"1","serialParity":"none","serialConnectionDelay":"100","unit_id":"1","commandDelay":"1","clientTimeout":"1000","reconnectTimeout":"2000"}]
	```

Logo após a saida do node *Modbus Read* foi adicionado um node *Change* para poder setar uma objeto para ser considerado index da busca. Que posteriormente será usado no node *function*:
> <img src="https://github.com/dedynobre/dados-modbus-csv/blob/master/imagens/node-config-04.png"/>

Código: </br>
	```
	[[{"id":"24d157b1.c80458","type":"change","z":"7aaf1b1a.016644","name":"","rules":[{"t":"set","p":"linhaDesejada","pt":"msg","to":"payload[0]","tot":"msg"}],"action":"","property":"","from":"","to":"","reg":false,"x":540,"y":440,"wires":[["9e0afb0b.db1598"]]}]
	```

#### Lendo arquivo

Configura o caminho do arquivo desejado para leitura.
> <img src="https://github.com/dedynobre/dados-modbus-csv/blob/master/imagens/node-config-05.png"/>
> <img src="https://github.com/dedynobre/dados-modbus-csv/blob/master/imagens/node-config-06.png"/>

Código: </br>
	```
	[{"id":"9e0afb0b.db1598","type":"file in","z":"7aaf1b1a.016644","name":"Altere para o local do seu arquivo","filename":"C:\\Users\\denis.nobre\\Downloads\\csv_curvas.csv","format":"utf8","chunk":false,"sendError":false,"encoding":"none","x":860,"y":440,"wires":[["66fa5cf.b6673a4"]]}]
	```

#### Lendo arquivo csv
> <img src="https://github.com/dedynobre/dados-modbus-csv/blob/master/imagens/node-config-07.png"/>
> <img src="https://github.com/dedynobre/dados-modbus-csv/blob/master/imagens/node-config-08.png"/>
Código: </br>
	```
	[{"id":"66fa5cf.b6673a4","type":"csv","z":"7aaf1b1a.016644","name":"","sep":";","hdrin":"","hdrout":"","multi":"mult","ret":"\\r\\n","temp":"","skip":"0","strings":true,"x":1110,"y":440,"wires":[["fb9505f1.6780a8"]]}]
	```


#### Extraindo valores desejados

Neste node está a função que extrai o valor desejado referenciado no node de leitura do Modbus, setado no node *change*:

```javascript
    var linhaDesejada = msg.linhaDesejada;
    var a = msg.payload.length;
    var res = {};
    for(x = 1; x < a; x++){
    if(x == linhaDesejada){
    res = {
    tipo: msg.payload[x].col1,
    curva : msg.payload[x].col2,
    segmento : msg.payload[x].col3,
    taxa : msg.payload[x].col4,
    temp : msg.payload[x].col5,
    tempo : msg.payload[x].col6
    }
    }
    }
    msg.payload = res;
    msg.topic = linhaDesejada;
    return msg;
```


A função retorna um objeto no formato abaixo:
> <img src="https://github.com/dedynobre/dados-modbus-csv/blob/master/imagens/node-config-09.png"/>