
# Manual de Instalação do Zentyal 8.0

## Introdução
Este manual tem como objetivo orientar o processo de instalação do Zentyal 8.0 sobre o Ubuntu 22.04 LTS, seja em sua versão Server ou Desktop. 
O guia apresenta os passos necessários para configurar e acessar a interface de administração do Zentyal, garantindo uma instalação bem-sucedida 
e com considerações importantes para o uso posterior.

**Autor:** Gladiston Santana  
**Data:** 02/01/2025  
**Contato:** gladiston.santana[at]gmail.com  

## 1. Baixar o script de instalação
Abra o terminal e execute:

```bash
wget https://raw.githubusercontent.com/zentyal/zentyal/master/extra/ubuntu_installers/zentyal_installer_8.0.sh
```

## 2. Conceder permissão de execução ao script
```bash
sudo chmod u+x zentyal_installer_8.0.sh
```

## 3. Executar o script de instalação
```bash
sudo ./zentyal_installer_8.0.sh
```

## 4. Escolher a instalação do ambiente gráfico
Durante a execução, o script perguntará se você deseja instalar o ambiente gráfico do Zentyal.  
Digite 'y' para instalar ou 'n' para não instalar:

```bash
Do you want to install the Zentyal Graphical environment? (n|y) y
```

## 5. Acessar a interface web de administração do Zentyal
Após a conclusão da instalação, será exibida a URL para acessar a interface web de administração:

```
Installation complete, you can access the Zentyal Web Interface at:
* https://<zentyal-ip-address>:8443/
```

Substitua `<zentyal-ip-address>` pelo endereço IP do seu servidor Zentyal.  
No navegador, acesse a URL fornecida e faça login com o usuário do sistema Ubuntu.  
Após o login, o assistente de configuração será iniciado.

## 6. Considerações adicionais
- **Edição do Zentyal:** A instalação via script configura a Edição de Desenvolvimento. Após a configuração inicial, você pode ativar uma Edição Trial ou Comercial acessando 'Sistema -> Edição do Servidor' e inserindo sua Chave de Licença.
- **Configuração de Rede:** É importante configurar o módulo de rede através do assistente de configuração antes de reiniciar o servidor para evitar perda de configuração de rede.
- **Ambiente Gráfico:** Se o ambiente gráfico não iniciar após o reinício, utilize os atalhos de teclado `CTRL + ALT + F7` ou `CTRL + ALT + F5` para acessá-lo.

Para mais detalhes, consulte a documentação oficial do Zentyal: [https://doc.zentyal.org/en/installation.html](https://doc.zentyal.org/en/installation.html)


## 7. Configuração do Squid como Proxy Transparente

### 7.1 Ativação do Squid
1. Instale o Squid usando o seguinte comando:
   ```bash
   sudo apt install squid
   ```

2. Configure o Squid para funcionar como proxy transparente. Edite o arquivo de configuração:
   ```bash
   sudo nano /etc/squid/squid.conf
   ```

3. Adicione ou altere as seguintes linhas no arquivo `squid.conf`:
   ```
   http_port 3128 intercept
   acl localnet src 192.168.0.0/24  # Substitua pelo IP da sua rede local
   http_access allow localnet
   http_access deny all
   ```

4. Salve e reinicie o Squid:
   ```bash
   sudo systemctl restart squid
   ```

### 7.2 Configuração de Proxy Transparente com Autenticação
1. Habilite a autenticação básica. Edite o arquivo `squid.conf` e adicione as seguintes linhas:
   ```
   auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
   auth_param basic realm Proxy Autenticado
   acl authenticated proxy_auth REQUIRED
   http_access allow authenticated
   ```

2. Crie um arquivo para armazenar as credenciais dos usuários:
   ```bash
   sudo touch /etc/squid/passwd
   sudo chmod 600 /etc/squid/passwd
   ```

3. Adicione um usuário ao arquivo de autenticação:
   ```bash
   sudo htpasswd -c /etc/squid/passwd usuario1
   ```

### 7.3 Configuração do Firewall para Proxy Transparente
1. Redirecione o tráfego HTTP e HTTPS para o proxy Squid. Adicione as seguintes regras de iptables:
   ```bash
   sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j REDIRECT --to-port 3128
   sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 443 -j REDIRECT --to-port 3128
   ```

3. Salve as regras do firewall para que persistam após a reinicialização:
   ```bash
   sudo apt install iptables-persistent
   sudo netfilter-persistent save
   ```

Agora o Squid está configurado como proxy transparente com autenticação nas portas 80 e 443. O firewall está configurado para permitir comunicação nas demais portas sem intervenção do proxy.



## 8. Liberação de Portas Específicas pelo Firewall

### 8.1 Porta FTP (21)
Para liberar o tráfego FTP, execute:
```bash
sudo iptables -A INPUT -p tcp --dport 21 -j ACCEPT
sudo iptables -A OUTPUT -p tcp --sport 21 -j ACCEPT
```

### 8.2 Porta SSH (22)
Para liberar o tráfego SSH, execute:
```bash
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
sudo iptables -A OUTPUT -p tcp --sport 22 -j ACCEPT
```

### 8.3 Porta SMTP (25)
Para liberar o tráfego SMTP, execute:
```bash
sudo iptables -A INPUT -p tcp --dport 25 -j ACCEPT
sudo iptables -A OUTPUT -p tcp --sport 25 -j ACCEPT
```

### 8.4 Porta POP3 (110)
Para liberar o tráfego POP3, execute:
```bash
sudo iptables -A INPUT -p tcp --dport 110 -j ACCEPT
sudo iptables -A OUTPUT -p tcp --sport 110 -j ACCEPT
```

### 8.5 Porta IMAP (143)
Para liberar o tráfego IMAP, execute:
```bash
sudo iptables -A INPUT -p tcp --dport 143 -j ACCEPT
sudo iptables -A OUTPUT -p tcp --sport 143 -j ACCEPT
```

### 8.6 Porta Remote Desktop Protocol (RDP) (3389)
Para liberar o tráfego RDP, execute:
```bash
sudo iptables -A INPUT -p tcp --dport 3389 -j ACCEPT
sudo iptables -A OUTPUT -p tcp --sport 3389 -j ACCEPT
```

### 8.7 Outros Serviços Populares
Para liberar outras portas de serviços populares, substitua `<PORTA>` pelo número da porta desejada:
```bash
sudo iptables -A INPUT -p tcp --dport <PORTA> -j ACCEPT
sudo iptables -A OUTPUT -p tcp --sport <PORTA> -j ACCEPT
```

Certifique-se de substituir `<PORTA>` pela porta apropriada ao serviço que deseja liberar.


### 8.8 Porta Ted - Sefaz (8017)
Para liberar o tráfego na porta 8017 para o Ted - Sefaz, execute:
```bash
sudo iptables -A INPUT -p tcp --dport 8017 -j ACCEPT
sudo iptables -A OUTPUT -p tcp --sport 8017 -j ACCEPT
```

### 8.9 Porta CagedNet (2500)
Para liberar o tráfego na porta 2500 para o CagedNet, execute:
```bash
sudo iptables -A INPUT -p tcp --dport 2500 -j ACCEPT
sudo iptables -A OUTPUT -p tcp --sport 2500 -j ACCEPT
```

### 8.10 Porta Sefip-CNS (2631)
Para liberar o tráfego na porta 2631 para o Sefip-CNS, execute:
```bash
sudo iptables -A INPUT -p tcp --dport 2631 -j ACCEPT
sudo iptables -A OUTPUT -p tcp --sport 2631 -j ACCEPT
sudo iptables -A INPUT -p udp --dport 2631 -j ACCEPT
sudo iptables -A OUTPUT -p udp --sport 2631 -j ACCEPT
```

### 8.11 Porta ReceitaNet (3456)
Para liberar o tráfego na porta 3456 para o ReceitaNet, execute:
```bash
sudo iptables -A INPUT -p tcp --dport 3456 -j ACCEPT
sudo iptables -A OUTPUT -p tcp --sport 3456 -j ACCEPT
sudo iptables -A INPUT -p udp --dport 3456 -j ACCEPT
sudo iptables -A OUTPUT -p udp --sport 3456 -j ACCEPT
```

### 8.12 Portas CAT (5017 e 5022)
Para liberar o tráfego nas portas 5017 e 5022 para o CAT, execute:
```bash
sudo iptables -A INPUT -p tcp --dport 5017 -j ACCEPT
sudo iptables -A OUTPUT -p tcp --sport 5017 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 5022 -j ACCEPT
sudo iptables -A OUTPUT -p tcp --sport 5022 -j ACCEPT
```

