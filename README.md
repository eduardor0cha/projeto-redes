# Projeto de Infraestrutura e Serviços de Redes

Nesse tutorial, mostraremos os passos para instalar e configurar 8 VMs interconectadas, distribuídas em 4 máquinas.

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

> *Os passos a seguir, devem ser utilizados com usuário com permissão de root.*
>
> *O procedimento mostrado será o utilizado para cada computador. Então repita o mesmo processo para todos os 4 computadores físicos, instalando 2 VMs em cada.*

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

![figura1](https://lh6.googleusercontent.com/ZUL_7gZ4GCyExmWgv5A2AnYPWHFnZVVAAotdwWZ3OMoqNlQsrMCTf5JxLzqxEqSGQwqt-UlzOs3WNkICOzZNtiECQbVOhkaMU1QtIpbgpZYXtB43eG1DUvisS8th-FoHxQdVTj3UPbx_AVeJiXTSmgY)

Clique em Arquivo > Importar Appliance

![figura2](https://lh6.googleusercontent.com/9JXI99nVTxxpM2xFGhB02CcvMpkuP24XREjZGe6OCRC-LgwR3SUQW6MFWEwXJybke7Di7o1LG3Ds3vuNn0EZ-X5biWsyKDQjNGpNHKIi0JgKes-4z6DcKa7Ij57qbqEu_PnqiP5AQWDQrktocgRPiGg)

- Inicie as VMs e em cada uma faça login com o usuário `administrador` e a senha `adminifal`.

---

> Caso queira configurar o mapeamento de teclas da VM, rode o seguinte comando e selecione as opções para Português do Brasil:
>
> ```bash
> sudo dpkg-reconfigure keyboard-configuration
> ```
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

- Verifique se a porta 22 está como LISTEN para conexões TCP:

    ```bash
    netstat -an | grep LISTEN
    ```

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


## Configuração das interfaces de redes das VMs

- Instale o net-tools:

    ```bash
        sudo apt install net-tools -y
    ```

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

    ![figura3](https://lh3.googleusercontent.com/8qtLSmpWj59PcpPkMaJ20euEnJtrjJFKIPUCXLhHNTpCWmPAWr35icb_pZTbi86Zz-b3-O4r96-1zVCRSWiSgallUBD5Z7V1M8i3yb_-aOz3OHkcCVL5r2_3YE89LhA7pcXqLpiGtkSkC0yewYFgpsw)

- Vá em avançado, e clique no botão de recarregar, em Endereço MAC para adquirir um novo endereço e clique em "OK":

    ![figura4](https://lh4.googleusercontent.com/aky52a4zVpTtevEZEojYgsE5CAt64rmX4rPKk4P4NeFzawvB6VMWf7c9dG-xFB99muL90XaI6h-yRZet5yLdT57cgPo_793D5t_Aft-lIj2t4D52W7yRSNjnVuKz69idicXSaRHOprYXBHXmt26i0Go)


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

    ![figura5](https://lh3.googleusercontent.com/FjtXnxx_cRL4FJCB9CteUfKs4dwvjkAlx9-m75Pw3GZ_3EuCavMwMCeb7Iw5TOU14v1265sg90th5-RdSW3ARfkGdBzSF8cr2TLFfVcpy6q0EUqyzLufUcHAglcQ59rH65c3AjuLRlKbvdJd0bhFkSo)

    ***Certifique-se fazer todo o processo em todos os computadores.***

---

## Testando conectividade e funcionalidades

Conecte todos os 4 computadores a um switch para fazer propriamente os seguintes testes.

### Ping

Execute o seguinte comando para testar se as máquinas estão conseguindo se comunicar:

```bash
ping <ip-da-vm-alvo>
```

![figura6](https://lh3.googleusercontent.com/St-RMKPMIyRTH4HQ-O6Vai_6VxiRD47-jpNmN_C2Z4ZI1I30NgHElq_mm-5HOOI71lNbwEqu9kgOkI6QwXLASTK6zFIdTaFFGbNxRcy38K-1ldo2EhldeNId6nTMGcwpq5Pfff6RebcZ-3Z2sB8CBjo)


Para isso, utilize o comando Ping <endereço-da-rede-desejada>.


### SSH

Execute o seguinte comando para tstar se o protocolo SSH está funcionando corretamente:

```bash
ssh <usuário-na-vm-alvo>@<ip-da-vm-alvo>
```

![figura7](https://lh6.googleusercontent.com/PqQI0K0SNk-WnB1e2bH8vzCIA-ETDG26zZIOYGRknFTotWsCd-lVeCdDauJfQfVNn12vJbNpdgL6QQTClQkEx7CuCpv9lRNfXtZUj87KNR9ClxNhNvtajatvnqMo6_t4XnrBpj3GAoe8f9mvfWvNJnk)

Verifique se há acesso ao terminal da VM alvo. Caso haja, tudo ocorreu como esperado.

### Definição de nomes de usuário, FQDN e alias

Para verificar se a definição ocorreu com sucesso, execute os comandos `ping` e `ssh` como mostrado anteriormente, porém com uma alteração: troque o endereço IP da máquina virtual alvo por um dos atributos definidos (hostname, FQDN ou alias).