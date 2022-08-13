# Projeto de Infraestrutura e Serviços de Redes

Nesse tutorial, mostraremos os passos para instalar e configurar 8 VMs interconectadas, distribuídas em 4 computadores.

## Topologia física

Antes de tudo, é necessário fazer com que os PCs físicos estejam, de fato, conectados. Para isso, conecte os 4 computadores a um switch e, caso queira, conecte um cabo de acesso à internet no switch para prover o acesso aos PCs.

No nosso caso, a organicação ficou da seguinte forma (as setas vermelhas indicam as conexões da rede interna, e a linha verde, o cabo para acesso à internet, todos conectados ao switch):

![figura1](/images/figura1.jpg)

## Definições de endereços IPs da rede e nomes de hosts

Os dados dessa tabela servirão como base para a configuração da interface de rede de cada VM.

    -------------------------------------------------------------------------------------
    | DESCRICAO   | IP            | hostname    | FQDN                       | alias   |
    -------------------------------------------------------------------------------------
    | VM1-PC1     | 192.168.13.33 | vm1-pc1     | vm1.grupo3-913.ifalara.net | vm1      |
    | VM2-PC1     | 192.168.13.34 | vm2-pc1     | vm2.grupo3-913.ifalara.net | vm2      |
    | VM1-PC2     | 192.168.13.35 | vm1-pc2     | vm3.grupo3-913.ifalara.net | vm3      |
    | VM2-PC2     | 192.168.13.36 | vm2-pc2     | vm4.grupo3-913.ifalara.net | vm4      |
    | VM1-PC3     | 192.168.13.37 | vm1-pc3     | vm5.grupo3-913.ifalara.net | vm5      |
    | VM2-PC3     | 192.168.13.38 | vm2-pc3     | vm6.grupo3-913.ifalara.net | vm6      |
    | VM1-PC4     | 192.168.13.39 | vm1-pc4     | vm7.grupo3-913.ifalara.net | vm7      |
    | VM2-PC4     | 192.168.13.40 | vm2-pc4     | vm8.grupo3-913.ifalara.net | vm8      |
    -------------------------------------------------------------------------------------

## Instalação das VMs

> _Os passos a seguir, devem ser utilizados com usuário com permissão de root._
>
> _O procedimento mostrado será o utilizado para cada computador. Então repita o mesmo processo para todos os 4 computadores físicos, instalando 2 VMs em cada._

- Crie uma pasta com o nome de sua preferência, no diretório de sua preferência (**certifique-se de que o usuário padrão tenha acesso ao diretório**). Escolhemos o diretório `/labredes/` como diretório base para esse guia e criamos a pasta `/labredes/projeto-913-grupo3/`.

  ```bash
  sudo mkdir /labredes/<nome-da-pasta>/
  ```

- Crie as subpastas `images/` e `vm/` em sua pasta principal.

  ```bash
  sudo mkdir /labredes/<nome-da-pasta>/images/
  sudo mkdir /labredes/<nome-da-pasta>/vm/
  ```

---

Caso tenha escolhido criar a sua pasta dentro do diretório `/labredes` , prossiga. Caso não, modifique as permissões dos arquivos e pastas:

```bash
sudo usermod -aG <grupo-de-proprietários> <usuário-a-ser-adicionado-ao-grupo>
sudo chown -R nobody:nogroup <diretório>
sudo chgrp -R <grupo-de-proprietários> <diretório>
sudo chmod -R 771 <diretório>
```

---

- Obtenha o arquivo de instalação das máquinas virtuais `ubuntu-server-mini.ova` das seguintes
  maneiras:

  - **Default:** Baixe o arquivo do servidor da máquina do professor, rodando:

    ```bash
    scp aluno@192.168.101.10:~/Public/iso-images/ubuntu-server-mini.ova <nome-da-pasta>/images/
    ```

  - No nosso caso, a máquina do professor não estava ligada e, portanto, tivemos que obter o arquivo copiando de uma pasta que já o possuía. Usamos o seguinte comando para copiar para a devida pasta:

  ```bash
  cp /labredes/images/original/ubuntu-server-mini.ova <nome-da-pasta>/images/
  ```

- Instale o Virtualbox Extension Pack:

  ```bash
  sudo apt install virtualbox-ext-pack
  ```

### Criando a VM a partir do arquivo `.ova` :

![figura2](/images/figura2.png)

Clique em Arquivo > Importar Appliance

![figura3](/images/figura3.png)

- Inicie as VMs e em cada uma faça login com o usuário `administrador` e a senha `adminifal`.

---

> Caso queira configurar o mapeamento de teclas da VM, rode o seguinte comando e selecione as opções para Português do Brasil:
>
> ```bash
> sudo dpkg-reconfigure keyboard-configuration
> ```
>
> E por fim, reinicie a VM com:
>
> ```bash
> sudo reboot
> ```
>
> Pronto. Seu teclado está mapeado corretamente!

---

- Utilize os comandos para atualizar os pacotes:

  ```bash
  sudo apt update
  sudo apt upgrade -y
  ```

- Instale o openssh-server:

  ```bash
  sudo apt-get install openssh-server
  ```

- Instale o net-tools:

  ```bash
  sudo apt install net-tools -y
  ```

- Verifique se a porta 22 está como LISTEN para conexões TCP:

  ```bash
  netstat -an | grep LISTEN
  ```

  ![figura4](/images/figura4.png)

- Permita o acesso SSH no firewall:

  ```bash
  sudo ufw allow ssh
  ```

- Ative o firewall:

  ```bash
  sudo ufw enable
  ```

- Verifique se a conexão 22/TCP está, de fato, como ALLOW

  ```bash
  sudo ufw status
  ```

  ![figura5](/images/figura5.png)

## Configuração das interfaces de redes das VMs

- Agora edite o arquivo `01-netcfg.yaml`

  ```bash
  sudo nano /etc/netplan/01-netcfg.yaml
  ```

  - Edite para deixar o arquivo de cada máquina no seguinte formato, apenas alterando o endereço no campo addresses - deixe em cada máquina o endereço atribuído à ela na tabela inicial, seguido de `/28`.

    ```bash
    network:
        ethernets:
            enp0s3:
                addresses: [192.168.13.33/28]
                gateway4: 192.168.13.33
                dhcp4: false
        version: 2
    ```

  - Para aplicar as configurações, rode:
    ```bash
    sudo netplan apply
    ```

- Com as máquinas virtuais desligadas, abra o VM Box, e em cada VM, vá nas configurações de rede e mude a rede para “Placa em modo Bridge”:

  ![figura6](/images/figura6.png)

- Vá em avançado, e clique no botão de recarregar, em Endereço MAC para adquirir um novo endereço e clique em "OK":

  ![figura7](/images/figura7.png)

- Em cada PC físico edite o arquivo `01-network-manager-all.yaml`:

  Rode:

  ```bash
  sudo nano /etc/netplan/01-network-manager-all.yaml
  ```

  Deixe a propriedade `renderer` como `NetworkManager`:

  ```bash
  # Let NetworkManager manage all devices on this system
  network:
      version: 2
          renderer: NetworkManager

  ```

- Inicie cada VM e defina o hostname de acordo com a tabela inicial:

  ```bash
  sudo hostnamectl set-hostname <hostname-da-tabela>
  ```

- Por fim, edite em cada VM, o arquivo de configuração de hosts:

  ```bash
  sudo nano /etc/hosts
  ```

  Adicione as 8 linhas de configuração seguindo a tabela inicial. Siga o seguinte padrão:

  `<endereço-ip> <hostname> <FQDN> <alias>`

  No nosso exemplo, ficou da seguinte forma:

  ![figura8](/images/figura8.png)

  **_Certifique-se fazer todo o processo em todos os computadores._**

- Adicione os usuários em cada VM:

  Aqui, adicionaremos 4 usuários em cada máquina virtual (cada um correspondendo a um integrante do grupo). Porém, você pode adicionar quais e quantos quiser. Basta rodar:

  ```bash
  sudo adduser <nome-do-usuario>
  ```

---

## Testando conectividade e funcionalidades

Agora testaremos se a conectividade e as configurações das VMs estão funcionando corretamente. Para isso, usaremos os comandos `ping` e `ssh`. Como definimos os endereços IP, nomes de host, FQDN e aliases, você pode usar qualquer uma dessas opções para se referenciar a uma máquina virtual. Portanto, nos comandos que se seguirão, você pode usar como `<alvo>` qualquer uma dessas opções.

### Ping

Execute o seguinte comando para testar se as máquinas estão conseguindo se comunicar:

```bash
ping <alvo>
```

Segue alguns dos variados testes:

![figura9](/images/figura9.png)

![figura10](/images/figura10.png)

![figura11](/images/figura11.png)

![figura12](/images/figura12.png)

### SSH

Execute o seguinte comando para tstar se o protocolo e conexão SSH estão funcionando corretamente:

```bash
ssh <usuário-na-vm-alvo>@<alvo>
```

> _Verifique se há acesso ao terminal da VM alvo. Caso haja, tudo ocorreu como esperado._

Segue alguns dos variados testes:

![figura13](/images/figura13.png)

![figura14](/images/figura14.png)

![figura15](/images/figura15.png)

![figura16](/images/figura16.png)

![figura17](/images/figura17.png)

---

## Conclusão

Após os testes serem concluídos com sucesso, você já pode utilizar a sua rede recém criada e configurada para os mais diversos usos. E ainda mais! Agora você poderá montar outras redes com especificidades diferentes, conforme a sua necessidade!

**Obrigado por ler até aqui! :D**
