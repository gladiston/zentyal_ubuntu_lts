
# Manual de Instalação e Configuração do Zentyal 8.0 no Ubuntu 24.04 LTS

**Autor**: Gladiston Santana<br>
**Data**: 02 de janeiro de 2025<br>
**Contato**: gladiston.santana [em] gmail [ponto] com

---

## Introdução

Este documento tem como objetivo fornecer um guia prático para instalação e configuração do Zentyal 8.0 em sistemas baseados no Ubuntu 24.04 LTS, tanto nas versões Server quanto Desktop. O Zentyal é uma ferramenta poderosa para gerenciar servidores, oferecendo funcionalidades como controlador de domínio, servidor de arquivos e impressão, gateway de rede e muito mais.

Seguindo este guia, você poderá instalar e configurar o Zentyal de forma rápida e eficiente, garantindo um ambiente funcional e seguro para suas necessidades.

---

## Requisitos Pré-Instalação

Antes de iniciar, certifique-se de que seu sistema atende aos seguintes requisitos:

- Ubuntu 24.04 LTS (Server ou Desktop) instalado e atualizado.
- Acesso à Internet.
- Privilegios de administrador no sistema.

---

## Parametros assumidos

Antes de iniciar, saiba que neste HowTo usaremos os seguintes parametros:

- Rede: `192.168.1.0/24`
- IP do servidor Ubuntu 24.04 LTS com Zentyal: `192.168.1.3`
- Mascara de rede: `255.255.255.0`
- Nome do host (fqdn): `acme.lan`
- Nome NETBIOS: `ACME`
- Gateway: `192.168.1.254`
- DNS: `192.168.1.3` (o mesmo servidor)
Ao longo do artigo faça a suas adaptações para sua rede e servidor.

---

## Etapas de Instalação

### 1. Baixe o script de instalação

```bash
wget https://raw.githubusercontent.com/zentyal/zentyal/master/extra/ubuntu_installers/zentyal_installer_8.0.sh
```

### 2. Conceda permissão de execução ao script

```bash
sudo chmod u+x zentyal_installer_8.0.sh
```

### 3. Execute o script

```bash
sudo ./zentyal_installer_8.0.sh
```

Durante a execução, o script perguntará se você deseja instalar o ambiente gráfico do Zentyal. Digite `y` para instalar ou `n` para não instalar:

```
Do you want to install the Zentyal Graphical environment? (n|y) y
```

### 4. Acesse a interface web de administração

Após a conclusão da instalação, será exibida a URL para acessar a interface web de administração do Zentyal:

```
Installation complete, you can access the Zentyal Web Interface at:
* https://192.168.1.3:8443/
```

Substitua `<zentyal-ip-address>` pelo endereço IP do seu servidor Zentyal.

### 5. Autentique-se na interface web

Na tela de login, utilize as credenciais do usuário do sistema Ubuntu para autenticar-se.

---

## Configuração Inicial

### Assistente de Configuração

Após o login, o assistente de configuração será iniciado. Ele permite:

- Selecionar pacotes a serem instalados (ex.: Servidor DNS, LDAP, Gateway).
- Configurar as interfaces de rede.
- Ajustar outras definições iniciais do sistema.

**Atenção**: Se o assistente não for concluído, a configuração de rede pode ser perdida após reinícios. Certifique-se de concluir todas as etapas.

### Configurando Rede e Hostname

1. **Configurar o endereço IP fixo**:

   - Acesse `Network -> Interfaces` na interface do Zentyal.
   - Localize a interface de rede conectada e clique em `Edit`.
   - Configure as opções:
     - **IP Address**: `192.168.1.3`
     - **Netmask**: `255.255.255.0`
     - **Gateway**: `192.168.1.254` (ou o gateway da sua rede).
   - Clique em `Save` para aplicar as alterações.

2. **Alterar o hostname**:

   - Acesse `System -> General Settings`.
   - No campo **Hostname**, insira `dc1`.
   - Clique em `Change` para salvar.

3. **Configurar os nameservers**:

   - Vá para `Network -> DNS`.
   - Insira os servidores DNS desejados. Por exemplo:
     - **Primary DNS**: `192.168.1.3` (o próprio servidor Zentyal).
     - **Secondary DNS**: `8.8.8.8` (ou o DNS de sua escolha).
   - No campo **Search Domain**, insira `acme.lan`.
   - Clique em `Save`.

4. **Teste as configurações**:

   - No terminal do Zentyal, execute:
     ```bash
     ping -c 4 192.168.1.254
     nslookup dc1.acme.lan
     ```

   - Certifique-se de que o ping retorna pacotes corretamente e o `nslookup` resolve o hostname do servidor.

---

## Ativando o Servidor de Domínio

### Introdução

Ativar o servidor de domínio é uma das etapas mais importantes ao configurar o Zentyal. Ele permite centralizar a administração de usuários, computadores e políticas de segurança dentro de uma rede, aumentando a produtividade e facilitando a gestão. Um controlador de domínio bem configurado também reduz problemas comuns de configuração manual e melhora a segurança geral da infraestrutura.

### Passo a Passo

1. **Acesse a interface de administração do Zentyal**:
   - Abra um navegador e acesse `https://192.168.1.3:8443/`.
   - Autentique-se com as credenciais do usuário administrador criado na instalação.

2. **Habilite os módulos necessários**:
   - Navegue até `Software > Zentyal Components`.
   - Marque as opções:
     - `Domain Controller and File Sharing`
     - `DNS Service`
   - Clique em **Save Changes** para aplicar.

3. **Configure o domínio**:
   - Acesse `Domain > General Settings`.
   - Preencha os seguintes campos:
     - **Domain Name**: `acme.lan`
     - **NetBIOS Name**: `ACME`
   - Clique em **Save** para confirmar as alterações.

4. **Crie um usuário administrador do domínio**:
   - Acesse `Users and Computers > Manage`.
   - Clique em **Add User**.
   - Preencha os detalhes do usuário e marque a opção `Administrator`.
   - Clique em **Save**.

5. **Teste a configuração**:
   - No terminal do servidor ou em um cliente na rede, execute:
     ```bash
     ping -c 4 192.168.1.3
     nslookup acme.lan 192.168.1.3
     ```
   - Em um cliente Linux, teste com:
     ```bash
     ping -c 4 192.168.1.3
     dig @192.168.1.3 acme.lan
     ```
   - Verifique se o `ping`, `nslookup` e `dig` retornam resultados corretos.

### Configurações adicionais

- **Gerencie políticas de grupo (GPOs)**:
  - Utilize o RSAT (Remote Server Administration Tools) do Windows para criar e aplicar GPOs.
  - Certifique-se de que os computadores no domínio utilizam o servidor Zentyal como DNS primário.

- **Adicione computadores ao domínio**:
  - No cliente Windows, abra `Configurações do Sistema` > `Nome do Computador` > `Alterar`.
  - Insira o nome do domínio (`acme.lan`) e as credenciais do administrador.
  - Reinicie o computador após a adesão.

Seguindo essas etapas, seu servidor de domínio estará configurado e pronto para uso.


---

## Integrando o Squid com o Samba

A integração do Squid com o Samba permite autenticar usuários do domínio diretamente no proxy, utilizando as credenciais configuradas no Zentyal. A maior parte da configuração pode ser feita pela interface gráfica do Zentyal, mas algumas etapas avançadas requerem o uso do terminal.

### Configurando Autenticação no Proxy via Zentyal

1. **Ative a autenticação no Squid**:
   - Acesse `HTTP Proxy -> General Settings`.
   - Marque a opção `Enable authentication`.

2. **Configure o domínio para autenticação**:
   - No mesmo menu, insira os seguintes valores no campo `Domain Controller`:
     - **Domain Name**: `acme.lan`.
     - **Controller IP**: `192.168.1.3` (ou o IP do servidor Zentyal).

3. **Configure perfis de filtro**:
   - Vá para `HTTP Proxy -> Filter Profiles`.
   - Crie regras específicas para grupos de usuários:
     - Escolha o grupo correspondente configurado no Samba.
     - Adicione palavras-chave ou categorias que devem ser bloqueadas.

---

## Adicionando Grupos e Usuários no Zentyal

### Introdução
Gerenciar grupos e usuários é essencial em qualquer ambiente corporativo. No Zentyal, essa tarefa é simplificada através da interface gráfica, permitindo que você organize o acesso a recursos, atribua permissões e mantenha a segurança da sua rede de forma centralizada. Isso também facilita a administração de equipes e departamentos, garantindo que apenas os usuários autorizados tenham acesso às informações necessárias.

A seguir, apresentamos um passo a passo para criar os usuários "administrador", "joão" e "silva" e os grupos "orcamentos", "projetos" e "financeiro" utilizando a interface gráfica do Zentyal.

---

### Passo a Passo para Adicionar Grupos

1. **Acesse a interface web do Zentyal:**
   - Abra um navegador e acesse `https://192.168.1.3:8443`.
   - Insira as credenciais de administrador para fazer login.

2. **Navegue para a seção de grupos:**
   - No menu lateral, clique em `Users and Computers` e, em seguida, em `Groups`.

3. **Crie os grupos desejados:**
   - Clique no botão `Add Group`.
   - Preencha os seguintes detalhes:
     - **Group Name:** `orcamentos`
     - Clique em `Add`.
   - Repita o processo para os grupos `projetos` e `financeiro`.

4. **Salvar alterações:**
   - Clique em `Save Changes` para aplicar as modificações.

---

### Passo a Passo para Adicionar Usuários

1. **Navegue para a seção de usuários:**
   - No menu lateral, clique em `Users and Computers` e, em seguida, em `Users`.

2. **Adicione o usuário "administrador":**
   - Clique no botão `Add User`.
   - Preencha os seguintes detalhes:
     - **Full Name:** Administrador
     - **Username:** administrador
     - **Password:** (insira uma senha segura)
     - **Primary Group:** (mantenha o padrão)
   - Clique em `Add`.

3. **Adicione os usuários "joão" e "silva":**
   - Para cada usuário, repita o processo acima com os seguintes detalhes:
     - **Full Name:** João / Silva
     - **Username:** joao / silva
     - **Password:** (insira uma senha segura para cada um)
     - **Primary Group:** (mantenha o padrão)
   - Clique em `Add` para cada usuário criado.

4. **Atribua os usuários aos grupos:**
   - Para cada usuário, clique no nome dele na lista.
   - Em `Group Membership`, selecione os grupos relevantes:
     - Para "joão", selecione `orcamentos` e `projetos`.
     - Para "silva", selecione `financeiro`.
   - Clique em `Save` para salvar as alterações.

---

### Testando as Configurações

1. **Verifique os grupos:**
   - Navegue até `Users and Computers -> Groups`.
   - Certifique-se de que os grupos `orcamentos`, `projetos` e `financeiro` contêm os usuários atribuídos corretamente.

2. **Teste as permissões:**
   - Autentique-se na rede usando os usuários criados.
   - Verifique se cada usuário tem acesso apenas aos recursos configurados para os grupos aos quais pertence.

Com essas etapas, você pode gerenciar eficientemente grupos e usuários no Zentyal, mantendo um ambiente organizado e seguro.

---

## Configurações Avançadas no Terminal

### Introdução

Apesar da interface gráfica do Zentyal simplificar grande parte da configuração, é fundamental dominar também o terminal para realizar ajustes específicos ou resolver problemas mais complexos. O uso combinado das duas abordagens garante uma gestão eficiente e permite personalizar detalhes que não estão acessíveis pela interface gráfica. Nesta seção, mostraremos como incluir um usuário para autenticação no proxy e configurar ACLs no Squid utilizando o terminal.

### Configurando um Usuário para o Proxy

1. **Criar o usuário `proxy_user`:**
   - Na interface gráfica do Zentyal, acesse `Users and Computers > Users`.
   - Clique em **Add User** e insira os seguintes dados:
     - **Full Name**: Proxy User
     - **Username**: proxy_user
     - **Password**: senha123
   - Clique em **Save** para salvar o novo usuário.

2. **Verificar o usuário no terminal:**
   - No terminal do servidor, execute o seguinte comando para listar os usuários do sistema:
     ```bash
     wbinfo -u
     ```
     Certifique-se de que `proxy_user` está listado.

3. **Configurar o Squid para usar `proxy_user`:**
   - Edite o arquivo de configuração do Squid:
     ```bash
     sudo nano /etc/squid/squid.conf
     ```
   - Altere ou adicione a seguinte configuração de autenticação:
     ```bash
     auth_param basic program /usr/lib/squid/basic_ldap_auth -R -b "dc=acme,dc=lan" -D "cn=proxy_user,dc=acme,dc=lan" -w senha123 -f sAMAccountName=%s -h 192.168.1.3
     acl autenticados proxy_auth REQUIRED
     http_access allow autenticados
     ```
     - Substitua `192.168.1.3` pelo IP do seu servidor Zentyal, se diferente.

4. **Recarregar o Squid:**
   - Aplique as alterações executando:
     ```bash
     sudo systemctl reload squid
     ```

5. **Testar a autenticação:**
   - No terminal de um cliente, teste o acesso ao proxy:
     ```bash
     curl --proxy-user proxy_user:senha123 -x http://192.168.1.3:3128 http://www.google.com
     ```
     Certifique-se de substituir o IP e porta conforme sua configuração.

6. **Monitorar os logs do Squid:**
   - Verifique os registros de acesso e autenticação:
     ```bash
     tail -f /var/log/squid/access.log
     ```

Após estes passos, o usuário `proxy_user` estará configurado corretamente para autenticação no proxy. Isso garante maior segurança ao utilizar uma conta dedicada, ao invés do usuário administrador, e permite um controle mais preciso sobre os acessos ao sistema.

---

## Ativando e Conferindo a Presença do DNS Local

### Por que o DNS Local é importante?

O DNS (Domain Name System) é essencial para o funcionamento de uma rede, pois traduz nomes de domínio amigáveis para endereços IP, facilitando a comunicação entre dispositivos. No contexto de servidores Zentyal, o DNS local garante que serviços, dispositivos e usuários na rede interna possam se comunicar de maneira eficiente, reduzindo atrasos e otimizando a resolução de nomes. Além disso, ele pode ser configurado para oferecer maior segurança e controle administrativo.

### Ativando o Serviço DNS no Zentyal

1. **Acesse a interface gráfica do Zentyal**:
   - Utilize um navegador e acesse `https://<endereço-ip-zentyal>:8443`.

2. **Ative o módulo de DNS**:
   - Navegue para `Software -> Zentyal Components`.
   - Marque a caixa `DNS Service`.
   - Clique em `Save Changes`.

3. **Configure o serviço DNS**:
   - Acesse `DNS -> General Settings`.
   - Certifique-se de que a opção `Enable DNS Service` está ativada.
   - No campo **Domain Name**, insira o nome de domínio interno, por exemplo, `acme.lan`.
   - Clique em `Save`.

4. **Adicione registros DNS**:
   - Vá para `DNS -> DNS Records`.
   - Clique em `Add Record`.
   - Insira os detalhes do registro, por exemplo:
     - **Type**: `A`.
     - **Name**: `dc1`.
     - **IP Address**: `<endereço-ip-zentyal>`.
   - Clique em `Save`.

### Testando o Serviço DNS

#### Em um computador com Windows:
1. Abra o Prompt de Comando (`cmd`).
2. Verifique a resolução de nomes com o comando:
   ```cmd
   nslookup dc1.acme.lan <endereço-ip-zentyal>
   ```
3. Certifique-se de que o retorno contém o endereço correto.

#### Em um computador com Linux:
1. Abra o terminal.
2. Execute o comando `dig` para verificar a resolução:
   ```bash
   dig @<endereço-ip-zentyal> dc1.acme.lan
   ```
3. Confirme se a resposta mostra o endereço IP correspondente.

Com essas configurações, o serviço DNS local estará funcional e pronto para uso na sua rede.
    
---

## Ativando Autoconfiguração de Proxy

### O que é WPAD?

O WPAD (Web Proxy Auto-Discovery Protocol) é um protocolo que permite que dispositivos em uma rede detectem automaticamente as configurações de proxy. Isso elimina a necessidade de configurar manualmente os navegadores ou sistemas operacionais, simplificando o gerenciamento e reduzindo erros em redes corporativas.

### Benefícios de ativar o WPAD

- **Redução de erros**: Configurações manuais podem ser propensas a erros.
- **Gerenciamento centralizado**: Alterações no proxy são aplicadas automaticamente a todos os dispositivos na rede.
- **Melhoria na segurança**: Permite maior controle sobre o tráfego de rede.

### Passos para habilitar o WPAD no Zentyal

1. **Habilitar WPAD no Proxy HTTP**
   - Acesse `HTTP Proxy -> General Settings` na interface do Zentyal.
   - Marque a opção `Enable WPAD`.
   - Salve as alterações clicando em `Save`.

2. **Criar um registro DNS para o WPAD**
   - Vá para `DNS -> DNS Records`.
   - Clique em `Add Record`.
   - Preencha os campos da seguinte forma:
     - **Type**: `CNAME`.
     - **Name**: `wpad`.
     - **Target**: O hostname do servidor Zentyal (ex.: `dc1.acme.lan`).
   - Clique em `Save`.

3. **Configurar o arquivo wpad.dat**
   - Crie o arquivo `wpad.dat` no servidor Zentyal contendo as configurações de proxy:
     ```plaintext
     function FindProxyForURL(url, host) {
         if (isInNet(host, "192.168.1.0", "255.255.255.0")) {
             return "PROXY 192.168.1.3:3128";
         } else {
             return "DIRECT";
         }
     }
     ```
   - Salve o arquivo no diretório `/var/www/html/`.

4. **Configurar o servidor web**
   - Certifique-se de que o servidor web no Zentyal está configurado para servir o arquivo `wpad.dat`:
     - No terminal, execute:
       ```bash
       sudo systemctl enable apache2
       sudo systemctl start apache2
       ```

5. **Testar o WPAD**
   - Em um navegador de um cliente na rede, habilite a configuração automática de proxy e reinicie o navegador.
   - Acesse um site para verificar se o proxy está sendo utilizado. Verifique os logs do Squid para confirmar.

6. **Depuração**
   - Para monitorar possíveis problemas, verifique os logs do servidor web:
     ```bash
     sudo tail -f /var/log/apache2/access.log
     sudo tail -f /var/log/apache2/error.log
     ```
Com essas etapas, o WPAD estará ativado e funcional em sua rede, facilitando a configuração do proxy para todos os dispositivos.


---

# Habilitando Suporte Remoto via SSH

## Por que ativar o SSH?
O SSH (Secure Shell) é essencial para a administração remota de servidores. Ele permite gerenciar o sistema com segurança, realizar manutenção, aplicar configurações e solucionar problemas sem a necessidade de acesso físico ao servidor. No caso do Zentyal, o suporte remoto é especialmente útil para administrar serviços e realizar diagnósticos.

---

## Etapas para Configuração

### 1. Ativando o SSH no Zentyal
1. Acesse a interface gráfica do Zentyal.
2. Navegue até `System -> Services`.
3. Localize o serviço **SSH** na lista.
4. Habilite o serviço marcando a caixa correspondente e clique em `Save Changes`.

---

### 2. Configurando o Firewall para Permitir Conexões SSH
1. Vá para `Firewall -> Packet Filter Rules`.
2. No perfil **External Networks to Zentyal**, clique em `Add Rule`.
3. Configure a regra:
   - **Decision**: Accept.
   - **Source**: Any.
   - **Destination Service**: SSH.
4. Clique em `Add` e, em seguida, `Save Changes` para aplicar as configurações.

---

### 3. Habilitando o Acesso do Usuário "root"
Por padrão, o acesso remoto via SSH como "root" está desabilitado por questões de segurança. Se necessário:

#### **Ativar a senha do root**:
1. No terminal do servidor Zentyal, execute:
   ```bash
   sudo passwd root
   ```
2. Insira e confirme a nova senha para o usuário "root".

#### **Permitir login root no SSH**:
1. Edite o arquivo de configuração do SSH:
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```
2. Localize a linha:
   ```
   PermitRootLogin prohibit-password
   ```
3. Substitua por:
   ```
   PermitRootLogin yes
   ```
4. Salve as alterações e reinicie o serviço SSH:
   ```bash
   sudo systemctl restart ssh
   ```

---

### 4. Testando o Acesso SSH
1. Em outro computador, use um cliente SSH para testar a conexão com o IP `192.168.1.3`:
   ```bash
   ssh root@192.168.1.3
   ```
2. Certifique-se de que a conexão é estabelecida com sucesso e que as credenciais do usuário "root" funcionam.

---
## Liberações Básicas do Firewall

A interface gráfica do Zentyal permite configurar o firewall de forma intuitiva, garantindo que serviços essenciais sejam liberados sem comprometer a segurança da rede. Abaixo estão as etapas para liberar os serviços mencionados neste guia:

### 1. Liberar Conexões SSH
1. Acesse `Firewall -> Packet Filter Rules`.
2. No perfil **External Networks to Zentyal**, clique em `Add Rule`.
3. Configure a regra:
   - **Decision**: Accept.
   - **Source**: Any.
   - **Destination Service**: SSH.
4. Clique em `Add` e, em seguida, em `Save Changes`.

### 2. Liberar Serviço HTTP/HTTPS (Interface Web)
1. Acesse `Firewall -> Packet Filter Rules`.
2. No perfil **External Networks to Zentyal**, clique em `Add Rule`.
3. Configure as regras:
   - Para HTTP:
     - **Decision**: Accept.
     - **Source**: Any.
     - **Destination Service**: HTTP.
   - Para HTTPS:
     - **Decision**: Accept.
     - **Source**: Any.
     - **Destination Service**: HTTPS.
4. Clique em `Add` para cada regra e, em seguida, em `Save Changes`.

### 3. Liberar Serviços de Domínio e DNS
1. Acesse `Firewall -> Packet Filter Rules`.
2. No perfil **External Networks to Zentyal**, clique em `Add Rule`.
3. Configure as regras:
   - Para DNS:
     - **Decision**: Accept.
     - **Source**: Any.
     - **Destination Service**: DNS.
   - Para Controlador de Domínio (LDAP/SMB):
     - **Decision**: Accept.
     - **Source**: Any.
     - **Destination Service**: LDAP.
     - **Destination Service**: SMB.
4. Clique em `Add` para cada regra e, em seguida, em `Save Changes`.

### 4. Liberar WPAD (Proxy Automático)
1. Acesse `Firewall -> Packet Filter Rules`.
2. No perfil **External Networks to Zentyal**, clique em `Add Rule`.
3. Configure a regra:
   - **Decision**: Accept.
   - **Source**: Any.
   - **Destination Service**: HTTP.
4. Clique em `Add` e, em seguida, em `Save Changes`.

### Testando as Configurações
Após configurar as regras, teste os serviços liberados para garantir que o firewall está permitindo as conexões necessárias. Utilize comandos como `ping`, `nslookup` e `curl` para verificar o acesso.

Com essas configurações, o firewall do Zentyal estará devidamente configurado para permitir os serviços básicos essenciais mencionados no guia.


## Usando o Zentyal como Gateway

Quando o Zentyal é utilizado como gateway de rede, é essencial configurar corretamente o firewall para garantir que os serviços necessários para os usuários da rede estejam liberados. A interface gráfica do Zentyal facilita essa configuração, permitindo adicionar regras para liberar portas específicas de forma segura e eficiente.

### Liberando Serviços Essenciais

Para liberar os serviços essenciais no firewall do Zentyal, siga os passos abaixo:

1. **Acesse a interface gráfica do Zentyal:**
   - Utilize um navegador para acessar `https://192.168.1.3:8443` e faça login com as credenciais administrativas.

2. **Navegue até as regras do firewall:**
   - Vá para `Firewall -> Packet Filter Rules`.
   - Escolha o perfil adequado, como **External Networks to Zentyal**.

3. **Adicione as regras para os serviços:**
   - Clique em `Add Rule` e configure as seguintes portas:

| Porta | Serviço                                           | Protocolo |
|-------|---------------------------------------------------|-----------|
| 20    | Transferência de dados FTP                       | TCP       |
| 21    | Controle de FTP                                  | TCP       |
| 22    | Secure shell (SSH)                               | TCP       |
| 25    | Simple mail transfer protocol (SMTP)            | TCP       |
| 53    | Domain name system (DNS)                        | TCP/UDP   |
| 80    | Hypertext transfer protocol (HTTP)              | TCP       |
| 110   | Post office protocol v3 (POP3)                  | TCP       |
| 113   | Serviço de autenticação/protocolo de identificação | TCP       |
| 123   | Network time protocol (NTP)                     | UDP       |
| 143   | Internet message access protocol (IMAP)         | TCP       |
| 443   | Hypertext transfer protocol com SSL/TLS (HTTPS) | TCP       |
| 465   | SMTP SSL                                        | TCP       |
| 587   | Envio de mensagem de e-mail (SMTP)              | TCP       |
| 993   | Internet message access protocol com SSL (IMAPS)| TCP       |
| 995   | Post office protocol 3 com TLS/SSL (POP3S)      | TCP       |

4. **Configure as regras:**
   - Para cada serviço, selecione:
     - **Decision**: Accept
     - **Source**: Any
     - **Destination Service**: Escolha o serviço na lista ou insira a porta manualmente.
   - Clique em `Add` após cada regra.

5. **Salve as alterações:**
   - Após adicionar todas as regras, clique em `Save Changes` para aplicar as configurações.

### Liberando Serviços do Governo

Alguns serviços específicos utilizados por órgãos governamentais também precisam ser liberados no firewall. Para isso, siga os mesmos passos acima e adicione as seguintes portas:

| Porta   | Serviço                   | Protocolo |
|---------|---------------------------|-----------|
| 8017    | Ted - Sefaz              | TCP       |
| 2500    | CagedNet                 | TCP       |
| 2631    | Sefip-CNS                | TCP/UDP   |
| 3456    | ReceitaNet               | TCP/UDP   |
| 5017    | CAT                      | TCP       |
| 5022    | CAT                      | TCP       |

Certifique-se de repetir os passos para cada porta e serviço acima, salvando as alterações ao final.

### Testando as Configurações

1. Utilize ferramentas como `ping`, `nslookup` ou `curl` em clientes da rede para testar o acesso aos serviços configurados.
2. Verifique os logs do firewall no Zentyal para garantir que as conexões estão sendo tratadas conforme esperado.

Com essas configurações, o Zentyal estará pronto para operar como um gateway eficiente e seguro para sua rede.

---

# Conclusão

Neste guia, percorremos o processo de instalação, configuração e uso do Zentyal 8.0 no Ubuntu 22.04 LTS. Desde os requisitos iniciais até as configurações avançadas, abordamos os seguintes pontos principais:

- Preparação do ambiente e instalação do Zentyal;
- Configuração da interface web e módulos essenciais, como DNS e controlador de domínio;
- Gerenciamento de usuários, grupos e permissões para um ambiente corporativo mais eficiente;
- Integração de serviços, como proxy autenticado e configuração de firewall;
- Utilização do terminal para ajustes e personalizações avançadas.

Com o Zentyal devidamente configurado, você conta com uma solução robusta para gerenciar sua infraestrutura de rede, aumentar a segurança e centralizar a administração. Esperamos que este guia tenha sido útil e que você aproveite ao máximo as funcionalidades oferecidas por esta ferramenta poderosa.

