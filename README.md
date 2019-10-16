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
```json
	[{"id":"7d851b28.c19994","type":"modbus-read","z":"7aaf1b1a.016644","name":"","topic":"","showStatusActivities":false,"logIOActivities":false,"showErrors":false,"unitid":"","dataType":"HoldingRegister","adr":"102","quantity":"1","rate":"500","rateUnit":"ms","delayOnStart":false,"startDelayTime":"","server":"f7d7534b.020d3","useIOFile":false,"ioFile":"","useIOForPayload":false,"x":250,"y":460,"wires":[["24d157b1.c80458","8f4ff681.9e7498"],[]]},{"id":"f7d7534b.020d3","type":"modbus-client","z":"","name":"M580","clienttype":"tcp","bufferCommands":true,"stateLogEnabled":false,"tcpHost":"192.168.238.50","tcpPort":"502","tcpType":"DEFAULT","serialPort":"/dev/ttyUSB","serialType":"RTU-BUFFERD","serialBaudrate":"9600","serialDatabits":"8","serialStopbits":"1","serialParity":"none","serialConnectionDelay":"100","unit_id":"1","commandDelay":"1","clientTimeout":"1000","reconnectTimeout":"2000"}]
```

Logo após a saida do node *Modbus Read* foi adicionado um node *Change* para poder setar uma objeto para ser considerado index da busca. Que posteriormente será usado no node *function*:
> <img src="https://github.com/dedynobre/dados-modbus-csv/blob/master/imagens/node-config-04.png"/>

Código: </br>
```json
	[[{"id":"24d157b1.c80458","type":"change","z":"7aaf1b1a.016644","name":"","rules":[{"t":"set","p":"linhaDesejada","pt":"msg","to":"payload[0]","tot":"msg"}],"action":"","property":"","from":"","to":"","reg":false,"x":540,"y":440,"wires":[["9e0afb0b.db1598"]]}]
```

#### Lendo arquivo

Configura o caminho do arquivo desejado para leitura.
> <img src="https://github.com/dedynobre/dados-modbus-csv/blob/master/imagens/node-config-05.png"/>
> <img src="https://github.com/dedynobre/dados-modbus-csv/blob/master/imagens/node-config-06.png"/>

Código: </br>
```json
	[{"id":"9e0afb0b.db1598","type":"file in","z":"7aaf1b1a.016644","name":"Altere para o local do seu arquivo","filename":"C:\\Users\\denis.nobre\\Downloads\\csv_curvas.csv","format":"utf8","chunk":false,"sendError":false,"encoding":"none","x":860,"y":440,"wires":[["66fa5cf.b6673a4"]]}]
```

#### Lendo arquivo csv
> <img src="https://github.com/dedynobre/dados-modbus-csv/blob/master/imagens/node-config-07.png"/>
> <img src="https://github.com/dedynobre/dados-modbus-csv/blob/master/imagens/node-config-08.png"/>
Código: </br>
```json
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

#### Mostrando valores para o usuário

Para facilitar foi a visualização foi criado um dashboard com os dados.
> <img src="https://github.com/dedynobre/dados-modbus-csv/blob/master/imagens/node-config-11.png"/>

Para extrair os dados de obejto de saída da função utitlizamos o node *change* para extrair o item desejado, tipo, temp, tempo, ect e enviar para o *payload*:
> <img src="https://github.com/dedynobre/dados-modbus-csv/blob/master/imagens/node-config-10.png"/>

```json
    [{"id":"35072d3c.7fe2f2","type":"ui_text","z":"7aaf1b1a.016644","group":"88b611f0.30adc","order":7,"width":0,"height":0,"name":"","label":"Tipo","format":"{{msg.payload}}","layout":"row-spread","x":1670,"y":480,"wires":[]},{"id":"7fbbb785.819398","type":"ui_text","z":"7aaf1b1a.016644","group":"88b611f0.30adc","order":2,"width":0,"height":0,"name":"","label":"Curva","format":"{{msg.payload}}","layout":"row-spread","x":1670,"y":520,"wires":[]},{"id":"7392a472.e8a46c","type":"ui_text","z":"7aaf1b1a.016644","group":"88b611f0.30adc","order":3,"width":0,"height":0,"name":"","label":"Segmento","format":"{{msg.payload}}","layout":"row-spread","x":1690,"y":560,"wires":[]},{"id":"40288298.c3063c","type":"ui_text","z":"7aaf1b1a.016644","group":"88b611f0.30adc","order":4,"width":0,"height":0,"name":"","label":"Taxa","format":"{{msg.payload}}","layout":"row-spread","x":1670,"y":600,"wires":[]},{"id":"b88d83da.fdf64","type":"ui_text","z":"7aaf1b1a.016644","group":"88b611f0.30adc","order":5,"width":0,"height":0,"name":"","label":"Temperatura","format":"{{msg.payload}}","layout":"row-spread","x":1690,"y":640,"wires":[]},{"id":"6d905b82.5a38a4","type":"ui_text","z":"7aaf1b1a.016644","group":"88b611f0.30adc","order":6,"width":0,"height":0,"name":"","label":"Tempo","format":"{{msg.payload}}","layout":"row-spread","x":1670,"y":680,"wires":[]},{"id":"b7609000.fcda3","type":"ui_text","z":"7aaf1b1a.016644","group":"88b611f0.30adc","order":1,"width":0,"height":0,"name":"","label":"Codigo","format":"{{msg.topic}}","layout":"row-spread","x":1680,"y":440,"wires":[]},{"id":"6803745c.21f5ac","type":"change","z":"7aaf1b1a.016644","name":"","rules":[{"t":"set","p":"payload","pt":"msg","to":"topic","tot":"msg"}],"action":"","property":"","from":"","to":"","reg":false,"x":1480,"y":440,"wires":[["b7609000.fcda3"]]},{"id":"acafb318.6410a","type":"change","z":"7aaf1b1a.016644","name":"","rules":[{"t":"set","p":"payload","pt":"msg","to":"payload.tipo","tot":"msg"}],"action":"","property":"","from":"","to":"","reg":false,"x":1480,"y":480,"wires":[["35072d3c.7fe2f2"]]},{"id":"ab4ae719.a4df68","type":"change","z":"7aaf1b1a.016644","name":"","rules":[{"t":"set","p":"payload","pt":"msg","to":"payload.curva","tot":"msg"}],"action":"","property":"","from":"","to":"","reg":false,"x":1480,"y":520,"wires":[["7fbbb785.819398"]]},{"id":"3c654877.c3da08","type":"change","z":"7aaf1b1a.016644","name":"","rules":[{"t":"set","p":"payload","pt":"msg","to":"payload.segmento","tot":"msg"}],"action":"","property":"","from":"","to":"","reg":false,"x":1480,"y":560,"wires":[["7392a472.e8a46c"]]},{"id":"ffa53ad1.161708","type":"change","z":"7aaf1b1a.016644","name":"","rules":[{"t":"set","p":"payload","pt":"msg","to":"payload.taxa","tot":"msg"}],"action":"","property":"","from":"","to":"","reg":false,"x":1480,"y":600,"wires":[["40288298.c3063c"]]},{"id":"620fef6c.679b3","type":"change","z":"7aaf1b1a.016644","name":"","rules":[{"t":"set","p":"payload","pt":"msg","to":"payload.temp","tot":"msg"}],"action":"","property":"","from":"","to":"","reg":false,"x":1480,"y":640,"wires":[["b88d83da.fdf64"]]},{"id":"70219075.50425","type":"change","z":"7aaf1b1a.016644","name":"","rules":[{"t":"set","p":"payload","pt":"msg","to":"payload.tempo","tot":"msg"}],"action":"","property":"","from":"","to":"","reg":false,"x":1480,"y":680,"wires":[["6d905b82.5a38a4"]]},{"id":"88b611f0.30adc","type":"ui_group","z":"","name":"Teste Modbus - CSV","tab":"d910612a.ca81e","disp":true,"width":"6","collapse":false},{"id":"d910612a.ca81e","type":"ui_tab","z":"","name":"Home","icon":"dashboard","disabled":false,"hidden":false}]
```

# Conclusão

Este é um exemplo claro de aplicação de IIOT, onde podemos integrar um controlador conectado na rede de automação e tratando dados em sistemas corporativos.

> Nota: Este é um exemplo típico para testes de funcionalidade, onde ambos componentes estão na mesma rede. Geralmente a conexão entre as redes(de automação e corporativa) estão segregadas e a conexão é feita via **firewall** garantindo a segurança entre as duas redes.

## Help

Caso precisem te ajuda ou tenham alguma sugestão, deixe seu comentário [Aqui](https://github.com/dedynobre/dados-modbus-csv/issues).

