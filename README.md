
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

## Habilitando Suporte Remoto via SSH

### Instalando e Ativando o SSH no Zentyal

1. **Verifique se o SSH está instalado**:

   No terminal do Zentyal, execute:

   ```bash
   dpkg -l | grep openssh-server
   ```

   - Se não estiver instalado, instale o SSH com o comando:

     ```bash
     sudo apt install openssh-server
     ```

2. **Inicie e habilite o serviço SSH**:

   Execute os seguintes comandos para iniciar o serviço e habilitá-lo na inicialização:

   ```bash
   sudo systemctl start ssh
   sudo systemctl enable ssh
   ```

3. **Permita conexões SSH no Firewall**:

   - Acesse `Firewall -> Packet Filter` na interface do Zentyal.
   - Em `Filtering rules for internal networks`, adicione uma nova regra para permitir conexões na porta 22 (SSH).
   - Clique em `Save` para aplicar as configurações.

### Habilitando o Usuário "root" para Acesso SSH

1. **Desbloqueie o usuário root**:

   No terminal do Zentyal, defina uma senha para o usuário root com o comando:

   ```bash
   sudo passwd root
   ```

   Escolha uma senha forte e confirme.

2. **Permita login como root no SSH**:

   Edite o arquivo de configuração do SSH:

   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

   Localize a linha:

   ```
   PermitRootLogin prohibit-password
   ```

   Substitua por:

   ```
   PermitRootLogin yes
   ```

   Salve o arquivo e reinicie o serviço SSH:

   ```bash
   sudo systemctl restart ssh
   ```

### Testando o Acesso Remoto

1. **De uma estação cliente Linux**:

   Use o comando abaixo para conectar ao servidor Zentyal:

   ```bash
   ssh administrador@192.168.1.3
   ```

   Substitua `administrador` pelo usuário administrador configurado anteriormente.

2. **De uma estação cliente Windows**:

   - Use um cliente SSH como o [PuTTY](https://www.putty.org/).
   - Configure o endereço IP como `192.168.1.3` e a porta como `22`.
   - Conecte-se usando as credenciais do administrador.

3. **Teste com o usuário root (opcional)**:

   Caso precise acessar como root, utilize o mesmo procedimento acima, substituindo `administrador` por `root` e utilizando a senha definida anteriormente.

---

## Dicas e Solução de Problemas

- **Ambiente gráfico não inicia**: Utilize os atalhos `CTRL + ALT + F7` ou `CTRL + ALT + F5` para alternar entre as telas virtuais.

- **Configuração de rede ausente**: Reconfigure manualmente usando o comando `ip` para acessar novamente o assistente.

- **Edição do Zentyal**: Após a configuração inicial, ative uma edição Comercial ou Trial acessando `Sistema -> Edição do Servidor` na interface web.

---

## Conclusão

Com este guia, você aprendeu a instalar e configurar o Zentyal 8.0 em um servidor Ubuntu 22.04 LTS. Caso necessite de suporte adicional, consulte a [documentação oficial do Zentyal](https://doc.zentyal.org/en/installation.html).

Boa sorte com sua nova instalação!
