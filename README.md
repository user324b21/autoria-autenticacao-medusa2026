# Relatório Técnico — Simulação de Brute Force em FTP com Medusa e Fail2ban

## 1. Objetivo

Realizar uma simulação controlada de ataque de força bruta contra um serviço FTP em ambiente de laboratório, utilizando o Medusa para testar combinações de usuário e senha.

O objetivo foi demonstrar:

- Como funciona uma tentativa de força bruta contra autenticação FTP.
- Como identificar credenciais válidas por meio do Medusa.
- Como registrar tentativas de autenticação no servidor.
- Como configurar e validar a proteção contra tentativas repetidas utilizando Fail2ban.

Todos os testes foram realizados em equipamentos próprios, dentro de uma rede privada e controlada.

## 2. Ambiente utilizado

O laboratório foi composto por duas máquinas físicas com Linux Mint 22.3.

| Equipamento | Função | Endereço IP |
|---|---|---|
| Acer Aspire AG15-71P | Máquina de auditoria | 192.168.0.100 |
| Dell Inspiron 5558 | Servidor de laboratório | 192.168.0.101 |

### Ferramentas utilizadas

- Nmap: reconhecimento de serviços e portas.
- vsftpd: servidor FTP.
- Medusa 2.2: ferramenta de auditoria de autenticação.
- Fail2ban 1.0.2: proteção contra tentativas repetidas de autenticação.
- iptables: gerenciamento das regras de bloqueio.
- FTP: cliente utilizado para validar manualmente a autenticação.

### Justificativa da escolha do Linux Mint

A escolha do Linux Mint 22.3 foi intencional. O objetivo era construir um ambiente funcional, leve e estável, capaz de executar tanto as ferramentas de auditoria quanto os serviços necessários para a simulação.

Entre os motivos da escolha estão:

- **Baixo consumo de recursos:** permite executar serviços e ferramentas sem exigir hardware de alto desempenho.
- **Compatibilidade com hardware variado:** possibilita utilizar equipamentos de gerações diferentes no mesmo ambiente.
- **Facilidade de administração:** oferece gerenciamento tanto por interface gráfica quanto pelo terminal.
- **Padronização:** utilizar a mesma distribuição nas duas máquinas simplifica a manutenção e reduz diferenças de configuração.
- **Ambiente familiar:** permite concentrar o aprendizado em redes, serviços, autenticação e segurança, sem introduzir complexidade desnecessária na administração do sistema operacional.

A utilização de duas máquinas físicas também permitiu simular uma situação mais próxima de uma rede real, separando o equipamento responsável pela auditoria daquele que hospedava os serviços testados.

### Identificação dos equipamentos

**Máquina de auditoria — Acer Aspire AG15-71P**

- Sistema operacional: Linux Mint 22.3.
- Endereço IP: `192.168.0.100`.
- Função: executar o reconhecimento de rede e os testes de autenticação utilizando Nmap e Medusa.

**Máquina de destino — Dell Inspiron 5558**

- Sistema operacional: Linux Mint 22.3.
- Endereço IP: `192.168.0.101`.
- Função: hospedar o serviço FTP e executar o Fail2ban para monitoramento e proteção contra tentativas repetidas de autenticação.

## 3. Reconhecimento do servidor

Inicialmente, foi realizado o reconhecimento do equipamento Dell a partir do Acer:

```bash
nmap -sV 192.168.0.101
```

Foram identificadas as seguintes portas abertas:

| Porta | Serviço |
|---|---|
| 22 | SSH |
| 111 | rpcbind |
| 139 | Samba |
| 445 | Samba |

O reconhecimento permitiu identificar os serviços disponíveis no equipamento que seria utilizado como alvo do laboratório.

## 4. Preparação do serviço FTP

Foi instalado o vsftpd no Dell:

```bash
sudo apt install vsftpd
```

Após a instalação, foi verificado o funcionamento do serviço:

```bash
sudo systemctl status vsftpd --no-pager
```

Foi criado um usuário exclusivo para o laboratório:

```bash
sudo adduser ftp_lab
```

Também foi criado um diretório com um arquivo de teste:

```bash
sudo -u ftp_lab mkdir -p /home/ftp_lab/arquivos
```

O acesso FTP foi configurado para utilizar autenticação local, sem permitir acesso anônimo.

O usuário foi incluído na lista de usuários autorizados do vsftpd.

A autenticação foi validada manualmente a partir do Acer:

```bash
ftp 192.168.0.101
```

O servidor respondeu com:

```text
230 Login successful.
```

Isso confirmou o funcionamento do serviço e das credenciais de laboratório.

## 5. Preparação da wordlist

No Acer, foi criada uma lista com quatro entradas:

```bash
printf '%s\n' 'senhaerrada123' 'laboratorio2026' 'teste12345' > ~/ftp-lab-wordlist.txt
```

A lista continha três senhas incorretas e uma senha válida, definida exclusivamente para o usuário de laboratório `ftp_lab`.

A senha válida foi acrescentada à quarta linha utilizando entrada oculta no terminal:

```bash
read -rsp 'Digite a senha do ftp_lab (entrada oculta): ' senha; echo; printf '%s\n' "$senha" >> ~/ftp-lab-wordlist.txt; unset senha; chmod 600 ~/ftp-lab-wordlist.txt
```

O arquivo foi protegido com permissão 600, restringindo seu acesso ao proprietário.

**Observação importante:** durante a primeira execução, a senha válida havia sido digitada incorretamente na wordlist. O erro foi identificado ao comparar a entrada exibida pelo Medusa com a senha utilizada no login manual. A quarta linha foi corrigida por meio de entrada oculta, sem necessidade de expor a senha novamente.

Esse problema demonstrou a importância de validar a integridade da wordlist antes de interpretar os resultados de uma auditoria.

## 6. Simulação de força bruta com Medusa

A partir do Acer, foi executado:

```bash
medusa -h 192.168.0.101 -u ftp_lab -P ~/ftp-lab-wordlist.txt -M ftp -t 1 -f -v 6
```

Parâmetros utilizados:

- `-h`: endereço IP do servidor-alvo.
- `-u`: usuário utilizado na autenticação.
- `-P`: arquivo contendo a lista de senhas.
- `-M ftp`: módulo responsável pelo teste de autenticação FTP.
- `-t 1`: uma tentativa concorrente por vez.
- `-f`: interrompe a execução após encontrar uma credencial válida.
- `-v 6`: nível de detalhamento da saída.

### Resultado

Após a correção da quarta entrada da wordlist, o Medusa percorreu as quatro senhas.

As três primeiras tentativas falharam e a quarta foi aceita pelo servidor.

O resultado apresentou:

```text
Total Passwords: 4

ACCOUNT FOUND: [ftp] Host: 192.168.0.101
User: ftp_lab [SUCCESS]

GENERAL: Medusa has finished.
```

A linha `ACCOUNT FOUND` confirmou que o Medusa encontrou uma combinação válida de usuário e senha.

A credencial utilizada era exclusiva do laboratório e não foi incluída na documentação pública.

## 7. Configuração e validação do Fail2ban

O Fail2ban foi instalado e iniciado no Dell.

Foi verificado inicialmente que a proteção SSH estava ativa, mas ainda não havia uma jail específica para FTP.

Para permitir o monitoramento das tentativas de autenticação, foi habilitado o registro detalhado do vsftpd:

```ini
log_ftp_protocol=YES
```

O arquivo de log utilizado foi:

```text
/var/log/vsftpd.log
```

Foi criada a configuração:

```ini
[vsftpd]
enabled = true
port = 20,21
filter = vsftpd
logpath = /var/log/vsftpd.log
backend = auto
maxretry = 3
findtime = 10m
bantime = 10m
banaction = iptables-multiport
```

A configuração estabelece:

- Limite de três falhas de autenticação.
- Janela de observação de dez minutos.
- Bloqueio do endereço IP por dez minutos.
- Aplicação do bloqueio por meio de regras do iptables.

O filtro padrão do Fail2ban para vsftpd foi validado utilizando:

```bash
sudo fail2ban-regex /var/log/vsftpd.log /etc/fail2ban/filter.d/vsftpd.conf
```

Foram identificadas seis ocorrências compatíveis com falhas de autenticação no log analisado.

A configuração foi validada:

```bash
sudo fail2ban-client -t
```

E a jail FTP foi confirmada:

```bash
sudo fail2ban-client status
```

## 8. Teste de bloqueio automático

Foi realizada uma nova execução do Medusa enquanto a proteção FTP estava ativa.

O Fail2ban identificou três tentativas incorretas e bloqueou o endereço IP do Acer:

```text
Currently banned: 1
Banned IP list: 192.168.0.100
```

Na tentativa seguinte, o Medusa apresentou:

```text
NOTICE: ftp.mod: failed to connect, port 21 was not open on 192.168.0.101
```

A mensagem não significava que o serviço FTP havia sido encerrado. A investigação confirmou que o bloqueio estava sendo aplicado pelo firewall.

A regra correspondente foi identificada com:

```bash
sudo iptables -S | grep -i f2b
```

O endereço IP do Acer aparecia em uma regra de rejeição na cadeia `f2b-vsftpd`.

O bloqueio foi confirmado pelo estado da jail:

```bash
sudo fail2ban-client status vsftpd
```

## 9. Validação da senha e correção da wordlist

Durante as novas tentativas, foi identificado que o Medusa não encontrava a senha correta.

A investigação mostrou que havia um erro de digitação na quarta entrada da wordlist.

O login manual foi utilizado para confirmar que a senha cadastrada no servidor estava correta:

```bash
ftp 192.168.0.101
```

O servidor respondeu:

```text
230 Login successful.
```

Após a correção da wordlist, foi necessário controlar temporariamente a proteção FTP para permitir que o Medusa percorresse as quatro entradas sem que o bloqueio interrompesse a sequência.

A jail FTP foi pausada no Dell:

```bash
sudo fail2ban-client stop vsftpd
```

Com a proteção FTP temporariamente pausada, foi executada novamente a simulação no Acer.

O resultado apresentou:

```text
Total Passwords: 4
ACCOUNT FOUND: [ftp] Host: 192.168.0.101
User: ftp_lab [SUCCESS]
```

Assim, foi comprovado que a senha nova estava correta e que o erro anterior estava na digitação da wordlist.

A proteção FTP foi reativada após o teste:

```bash
sudo fail2ban-client start vsftpd
```

## 10. Resultados obtidos

Os testes demonstraram dois comportamentos distintos:

**Sem bloqueio ativo:**

- O Medusa conseguiu testar as quatro entradas.
- As três primeiras tentativas falharam.
- A quarta tentativa encontrou a credencial válida.

**Com Fail2ban ativo:**

- O servidor registrou as falhas de autenticação.
- Ao atingir três falhas dentro da janela configurada, o Fail2ban bloqueou o IP de origem.
- O Acer perdeu temporariamente o acesso à porta FTP.

Também foi possível observar a diferença entre uma falha de autenticação e uma conexão rejeitada pelo firewall.

## 11. Conclusão

A atividade demonstrou, na prática, como uma ferramenta de auditoria pode identificar credenciais válidas em um serviço FTP quando uma senha fraca ou conhecida está presente em uma wordlist.

Também demonstrou a importância de mecanismos defensivos como o Fail2ban, capazes de monitorar logs, identificar tentativas repetidas de autenticação e bloquear temporariamente os endereços IP responsáveis.

Um dos principais aprendizados foi que a preparação do ambiente, a validação das ferramentas, a análise dos logs e a interpretação dos resultados são tão importantes quanto a execução do teste de força bruta.

O laboratório também mostrou que um erro simples na wordlist pode alterar completamente o resultado de uma auditoria, reforçando a necessidade de conferir os dados utilizados antes de tirar conclusões.

**Recomendação de segurança:** em ambientes reais, deve-se evitar FTP tradicional sem criptografia. Recomenda-se utilizar SFTP ou FTPS, senhas fortes e exclusivas, limitação de tentativas, monitoramento de autenticação e regras de firewall adequadas.

---

**Observação sobre as evidências:** as capturas de tela devem ocultar senhas e quaisquer credenciais. A wordlist utilizada no laboratório não deve ser publicada com a senha válida.
