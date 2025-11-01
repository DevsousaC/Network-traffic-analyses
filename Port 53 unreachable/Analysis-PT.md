# Analise de tráfego de rede: Porta 53 não encontrada

### Introdução ao incidente

Os registros do analisador de protocolo de rede indicam que a porta 53 UDP está inacessível ao tentar acessar o site da empresa. A porta 53 UDP é normalmente usada para consultar serviços de DNS. Isso pode indicar um problema com o serviço de DNS. É possível que o serviço esteja inativo ou mal configurado.

### Analise inicial e possiveis causas

Após uma falha de conexão com o domínio "yummyrecipesforme.com", o problema foi investigado através da captura do tráfego de rede.

Identificou-se que o problema teve início hoje, às 13h24, quando uma requisição UDP para o servidor DNS (203.0.113.2) a partir do IP 192.51.100.15 recebeu uma resposta de erro ICMP.

Durante a busca, foram identificados o IP de origem (192.51.100.15), o destino (203.0.113.2), o protocolo utilizado (UDP) e a porta de destino (53).

Confirmou-se que o erro ocorreu pelo menos três vezes, indicando que o erro é contínuo e não se trata de uma corrupção no tráfego enviado.

A resposta foi um protocolo ICMP com o seguinte conteúdo: "Porta UDP 53 inacessível".

A causa mais provável é que o serviço DNS no servidor de destino (203.0.113.2) esteja inativo. Isso significa que o servidor pode estar online, mas o software específico responsável por responder às consultas de DNS não estava em execução, estava mal configurado ou não estava escutando na porta UDP 53 no momento do incidente.

### Próximos passos na investigação

O próximo passo para resolver este incidente é determinar se o servidor DNS não está funcionando corretamente. Se o servidor DNS estiver funcionando corretamente, as configurações do firewall devem ser verificadas para garantir que nenhuma alteração de configuração esteja bloqueando o tráfego de rede na porta 53.