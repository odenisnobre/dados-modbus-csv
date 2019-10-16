# Objetivo

Atualmente na rede existe um arquivo `.csv` que contém estruturas de dados que são mostradas para o usuário.
Basicamente, o arquivo possui, obviamente, linhas e colunas. O objetivo é ler o índice da coluna e retonar os valores das colunas.


# Configuração

Para os testes foram utilizados:
1. Node-Red - versão 1.0.1
2. CPU M580 - Schneider

## Configuração PLC

Em aplicação convencional o valor de referência(índice da coluna) seria escrito via interface com o usuário - SCADA. No nosso caso o criamos um contador simples com reset para poder simular o envio dos dados via Modbus TCP.

A CPU M580 utilizada já vem embarcada com o protoco modbus TCP o que facilita a integração.


#### Configuração do Rack:
<img src="https://github.com/dedynobre/dados-modbus-csv/blob/master/imagens/config-rack"/></br>






