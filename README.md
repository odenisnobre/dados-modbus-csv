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





