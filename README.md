
# Manual de Instalação e Configuração do Zentyal 8.0 no Ubuntu 22.04 LTS

**Autor**: Gladiston Santana<br>
**Data**: 02 de janeiro de 2025<br>
**Contato**: gladiston.santana [em] gmail [ponto] com

---

## Introdução

Este documento tem como objetivo fornecer um guia prático para instalação e configuração do Zentyal 8.0 em sistemas baseados no Ubuntu 22.04 LTS, tanto nas versões Server quanto Desktop. O Zentyal é uma ferramenta poderosa para gerenciar servidores, oferecendo funcionalidades como controlador de domínio, servidor de arquivos e impressão, gateway de rede e muito mais.

Seguindo este guia, você poderá instalar e configurar o Zentyal de forma rápida e eficiente, garantindo um ambiente funcional e seguro para suas necessidades.

---

## Requisitos Pré-Instalação

Antes de iniciar, certifique-se de que seu sistema atende aos seguintes requisitos:

- Ubuntu 22.04 LTS (Server ou Desktop) instalado e atualizado.
- Acesso à Internet.
- Privilegios de administrador no sistema.

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
* https://<zentyal-ip-address>:8443/
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
     - **Gateway**: `192.168.1.1` (ou o gateway da sua rede).
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
     ping -c 4 192.168.1.1
     nslookup dc1.acme.lan
     ```

   - Certifique-se de que o ping retorna pacotes corretamente e o `nslookup` resolve o hostname do servidor.

---

## Ativando o Servidor de Domínio

1. **Acesse a interface de administração do Zentyal**:
   - Navegue para `https://192.168.1.3:8443/` e autentique-se.

2. **Ative os módulos necessários**:
   - Acesse `Software -> Zentyal Components` e selecione:
     - `Domain Controller and File Sharing`.
     - `DNS Service`.
   - Clique em `Save Changes` para aplicar.

3. **Configure o domínio**:
   - Vá para `Domain -> General Settings` e configure:
     - **Domain Name**: `acme.lan`.
     - **NetBIOS Name**: `ACME`.
   - Clique em `Save`.

4. **Crie um usuário administrador**:
   - Em `Users and Computers -> Manage`, clique em `Add User`.
   - Insira os detalhes do usuário, marcando `Administrator`.

---

## Integrando o Squid com o Samba

### Configurando Autenticação no Proxy

1. Acesse `HTTP Proxy -> General Settings` e ative `Enable authentication`.

2. Configure o campo `Domain Controller` com:
   - **Domain Name**: `acme.lan`.
   - **Controller IP**: `192.168.1.3`.

3. Adicione regras em `HTTP Proxy -> Filter Profiles`.

---

## Ativando e Conferindo a Presença do DNS Local

1. Verifique se o DNS está ativo:
   - Vá para `DNS -> General Settings` e ative o `Enable DNS Service`, se necessário.

2. Teste o DNS local:
   ```bash
   nslookup dc1.acme.lan 192.168.1.3
   ```

---

## Ativando Autoconfiguração de Proxy

1. Habilite o WPAD em `HTTP Proxy -> General Settings`.
2. Adicione um registro `CNAME` para `wpad` em `DNS -> DNS Records`.

---

## Habilitando Suporte Remoto via SSH

1. **Instale e ative o SSH**:
   ```bash
   sudo apt install openssh-server
   sudo systemctl start ssh
   sudo systemctl enable ssh
   ```

2. **Habilite o acesso root**, se necessário:
   ```bash
   sudo passwd root
   ```

3. **Permita conexões SSH no Firewall**.

---

## Conclusão

Com este guia, você configurou o Zentyal 8.0 no Ubuntu 22.04 LTS. Aproveite!
