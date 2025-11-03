# Análise de Tráfego de Rede: Identificação de ataque DoS

> Este projeto contém a análise de uma captura de tráfego de rede de um servidor web. O objetivo é dissecar os pacotes para identificar configurações de segurança inadequadas e um ataque ativo de Negação de Serviço (DoS).

Este documento é o ponto de partida para entender as descobertas e a metodologia utilizada.

---

##  Estrutura da Pasta

Aqui está uma visão geral dos arquivos contidos neste projeto:

* **`README.md`**: Este arquivo, com o resumo do projeto e das descobertas.
* **`TCP-HTTP_log.md`**: O arquivo de log utilizado para a analise transformado em .md para melhor visualização.
* **`Analysis-PT.md`**: O relatório de segurança escrito, detalhando as vulnerabilidades, riscos e recomendações de mitigação em português.

## Contexto do Projeto

A captura de tráfego foi fornecida como parte de um laboratório do **Connect and Protect: Networks and Network Security** da Google. O cenário simula um servidor web (`192.0.2.1`) recebendo tráfego de clientes legítimos (ex: `198.51.100.23`) e de uma fonte maliciosa (`203.0.113.0`).

## Ferramentas Utilizadas

* **Wireshark**: Ferramenta principal para a inspeção, filtragem e análise dos pacotes de rede.

---

## Resumo das Descobertas Principais

A análise revelou duas descobertas de segurança significativas:

### 1. Descoberta Crítica: Transmissão de HTTP em Texto Puro na Porta 443

* **Descrição:** O servidor (`192.0.2.1`) foi observado aceitando e respondendo a requisições **HTTP em texto puro** na porta `443`.
* **Evidência:** Pacotes 47-51 mostram um *three-way handshake* TCP completo seguido por uma requisição `GET /sales.html HTTP/1.1` (não criptografada).
* **Risco:** Esta configuração anula o propósito da porta 443 (HTTPS), expondo clientes a ataques de interceptação de dados (Man-in-the-Middle) e roubo de sessão/credenciais.

### 2. Descoberta de Alta Gravidade: Ataque de Negação de Serviço (DoS) - SYN Flood

* **Descrição:** O servidor está sob um ataque ativo de inundação SYN (SYN Flood) vindo do IP `203.0.113.0`.
* **Evidência:** Os pacotes 52, 53, 54, 57, 59, 61 (e seguintes) mostram um volume massivo e repetitivo de pacotes `[SYN]` do atacante, sem nunca completar o *handshake* TCP.
* **Risco:** Este ataque visa esgotar a tabela de conexões do servidor, consumindo seus recursos e tornando-o incapaz de aceitar conexões legítimas, resultando em indisponibilidade do serviço.

---

## Relatório de Análise Detalhada

Para uma análise técnica completa, incluindo a gravidade de cada descoberta, o impacto nos negócios e as recomendações de mitigação específicas (como a ativação de *SYN cookies*), consulte o relatório completo:

➡️ **[Relatório completo aqui](Analysis-PT.md)**

## Como ler o log

O log apresentado no arquivo [TCP-HTTP_log](TCP-HTTP_log.md) se trata de uma tabela contendo o trafego de rede de um servidor web. Nele você poderá identificar a origem,destino, protocolo, e outras informações dos pacotes transacionados.

| Coluna | Descrição |
|--------|-----------|
| ID |Identificador único, utilizado para referenciar um registro especifico. |
| Tempo | Quando em segundos aconteceu o trafego daquele pacote contando a partir do momento que o WireShark começou a interceptar o trafego da rede |
| Origem | O IP que enviou aquele pacote (De onde veio) |
|Destino | O IP de destino do pacote (Para onde vai) |
| Info | Outras informações pertinentes ao pacote, que variam conforme flag, protocolo ou tipo de requisição |