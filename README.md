# Auditoria de Autenticação com Medusa

Projeto prático de cibersegurança desenvolvido em um homelab próprio, com o objetivo de estudar e testar técnicas de força bruta e password spraying em serviços de autenticação, utilizando Linux Mint, Medusa e outras ferramentas de análise de rede.

## Sobre o projeto

A ideia deste projeto é colocar em prática alguns conceitos de segurança da informação, principalmente aqueles relacionados à autenticação e à exposição de serviços de rede. Para isso, será utilizado um ambiente controlado, com serviços configurados especificamente para os testes, permitindo observar como funcionam as tentativas automatizadas de login, quais informações podem ser obtidas durante uma enumeração e quais riscos estão envolvidos no uso de credenciais fracas.

O laboratório será executado utilizando Linux Mint XFCE como sistema de auditoria, em substituição ao Kali Linux originalmente proposto no desafio, uma escolha feita para aproveitar os recursos disponíveis sem precisar manter outra máquina virtual em execução.

## Objetivos

Durante o desenvolvimento, serão realizados experimentos envolvendo autenticação FTP, enumeração de usuários e password spraying em SMB, além de testes em formulários web utilizando o DVWA (Damn Vulnerable Web Application).

Também fazem parte do projeto a criação de pequenas wordlists, a utilização de ferramentas de reconhecimento e auditoria, o registro dos resultados e a análise de medidas que poderiam dificultar ou impedir os ataques demonstrados.

## Ambiente e ferramentas

O ambiente será baseado em um homelab próprio, utilizando serviços isolados e contas criadas exclusivamente para os experimentos.

As principais ferramentas previstas são:

- **Linux Mint XFCE:** sistema operacional utilizado para executar as ferramentas de auditoria
- **Medusa:** ferramenta utilizada para testar autenticação em diferentes serviços
- **Nmap:** reconhecimento de rede e identificação de serviços e portas abertas
- **enum4linux:** enumeração de informações disponibilizadas por serviços SMB
- **DVWA:** aplicação web propositalmente vulnerável, utilizada para estudar falhas de autenticação
- **Docker:** gerenciamento do ambiente da aplicação web

## Experimentos

Os testes serão divididos em três cenários principais:

**FTP:** identificação do serviço e realização de tentativas controladas de autenticação utilizando a Medusa, com uma wordlist pequena e previamente preparada.

**SMB:** identificação e enumeração de usuários disponibilizados pelo serviço, seguida de um teste de password spraying para observar o comportamento da autenticação diante de uma senha candidata.

**Web / DVWA:** utilização de uma aplicação vulnerável para estudar a automação de tentativas de login, observando as respostas do formulário e os mecanismos de proteção disponíveis.

## Segurança e escopo

Todos os testes serão realizados em infraestrutura própria, com serviços e contas preparados exclusivamente para a atividade, sem utilização de credenciais pessoais ou acesso a sistemas de terceiros.

O objetivo não é comprometer sistemas, mas compreender o funcionamento dessas técnicas em um ambiente controlado e identificar formas de reduzir os riscos associados a elas.

## Resultados e documentação

Os resultados serão registrados ao longo dos experimentos, incluindo comandos utilizados, capturas de tela, comportamento dos serviços e observações sobre cada teste.

Também serão apresentadas recomendações de segurança relacionadas a políticas de senhas, limitação de tentativas, monitoramento de autenticação e restrição de acesso aos serviços expostos.

## Conclusão

Este projeto é uma oportunidade de trabalhar na prática com ferramentas de auditoria, administração Linux e segurança de redes, indo além da execução de comandos e buscando entender o que acontece por trás de cada tentativa de autenticação.

Mais do que demonstrar que uma ferramenta consegue testar credenciais, a proposta é compreender por que determinados serviços ficam vulneráveis e o que pode ser feito para protegê-los.

## Referências

- [Medusa — GitHub](https://github.com/jmk-foofus/medusa)
- [Nmap](https://nmap.org/)
- [DVWA — GitHub](https://github.com/digininja/DVWA)
- [enum4linux — GitHub](https://github.com/CiscoCXSecurity/enum4linux)
