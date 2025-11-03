# Relatório de Análise de ataque DoS

## Sumário
Uma análise do tráfego de rede capturado (referência: [TCP-HTTP_log](TCP-HTTP_log.md)) do servidor web 192.0.2.1 revelou duas descobertas de segurança significativas que exigem atenção imediata:

1. Vulnerabilidade Crítica de Configuração: O servidor aceita comunicação HTTP em texto puro na porta 443, que é designada para tráfego seguro (HTTPS). Isso anula completamente a proteção de criptografia esperada e expõe os usuários a ataques de interceptação de dados (Man-in-the-Middle).

2. Ataque Ativo de Negação de Serviço: O servidor está sofrendo um ataque de inundação SYN (SYN Flood) originado do endereço IP 203.0.113.0. Este ataque visa esgotar os recursos do servidor e impedir que usuários legítimos acessem o serviço.

---

## Transmissão de Dados em Texto Puro na Porta Segura (443)
Um cliente legitimo (198.51.100.23) realizou com sucesso um three-way handshake TCP com o servidor na porta 192.0.2.1:443, comprovado pela transação dos pacotes Nº 47, 48 e 49 que utilizam o protocolo TCP com as flags [SYN] e [ACK].

Em seguida, no pacote 50 é realizado uma requisição GET /sales.html HTTP/1.1 do cliente legitimo para o servidor, que responde com um status 200 OK, confirmando que aceitou e processou a requisição HTTP em texto puro.

A porta 443 é o padrão universal para HTTPS, implicando que toda a comunicação deve ser criptografada via TLS/SSL. Ao aceitar HTTP simples nesta porta, o servidor quebra essa premissa de segurança.

Isso expõe qualquer cliente que se conecte (mesmo que espere segurança) a um ataque Man-in-the-Middle (MitM). Um atacante na mesma rede poderia facilmente interceptar, ler e modificar todo o tráfego, incluindo credenciais de login, cookies de sessão ou qualquer dado sensível trocado.

### Recomendações
Corrigir a configuração do servidor web (ex: Nginx, Apache) para forçar estritamente o TLS/SSL na porta 443.

Desabilitar qualquer listener HTTP nesta porta.

Implementar HSTS (HTTP Strict Transport Security) para garantir que os navegadores sempre usem HTTPS.

## Ataque de Negação de Serviço (DoS) por Inundação SYN

Os pacotes Nº 52-54 compoem o three-way handshake, método de garantia de recebimento utilizado pelo protocolo TCP, que neste caso indica que o cliente 203.0.113.0 deseja realizar uma transação de pacotes TCP. 

O mesmo atacante envia repetidamente novos pacotes [SYN] para o servidor, sem nunca completar os handshakes anteriores, como visto nos pacotes Nº 57, 59, 61 (e seguintes). 

O log mostra um volume desproporcional de tráfego TCP [SYN] vindo de uma única fonte, um padrão clássico de SYN Flood. Um ataque de esgotamento de recursos. 

Para cada pacote [SYN] recebido, o servidor aloca memória e recursos, abre um socket e entra em um estado SYN-RECEIVED (conexão semiaberta), aguardando o [ACK] final do cliente.

Como o atacante nunca envia o [ACK], a fila de conexões pendentes (backlog) do servidor enche rapidamente. Uma vez que a fila está cheia, o servidor rejeitará novas conexões legítimas, resultando em uma Negação de Serviço (DoS).


### Recomendações

A recomendação mais eficaz em nível de kernel (Linux) é habilitar ```net.ipv4.tcp_syncookies``` Isso permite que o servidor responda a SYNs sem alocar recursos, validando o cliente apenas quando o [ACK] final chegar.

Configurar o firewall ou Load Balance para limitar o número de novas conexões [SYN] por segundo de um único IP é uma solução, mas apenas para ataques DoS, uma vez que ataques DDoS utilizam-se de diversas maquinas, e portanto pode conter diversos IPs de origem.


---

### Outras informações sobre a classificação do ataque

A inundação de requisições é um padrão de ataque cibernético conhecido como DoS (Denial of Service), que traduzido para o português seria um ataque de negação de serviço que, como o próprio nome sugere, visa alcançar a indisponibilidade de um serviço através da sobrecarga de trafego.

Este ataque em especifico se trata de um ataque de negação de serviço (DoS) por inundação SYN, que consiste em um atacante enviar um grande volume de pacotes SYN (sincronizar) para um servidor alvo, o sobrecarregando e tornando-o incapaz de aceitar conexões legítimas. Trata-se de um tipo de ataque distribuído de negação de serviço (DDoS) em que o servidor utiliza seus recursos para monitorar as conexões, que nunca são concluídas.

É importante salientar que, um ataque DoS é realizado de uma mesma origem, entretanto, atacantes podem utilizar de diversos computadores conectados a uma botnet para enviar o ataque misturando seu IP com o de diversos outros que estarão realizando as mesmas requisições simultaneamente formando um ataque de negação de serviço distribuido (DDoS)
