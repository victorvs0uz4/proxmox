## Convertendo uma máquina física para uma VM utilizando Prormox

1. Primeiro precisamos criar a VM no Proxmox com os recursos necessários: quantidade de núcleos, memória RAM e um disco temporário de 1GB com a controladora do tipo VirtIO SCSI, garantindo que após instalarmos os drivers Virtio será possível alterar o disco recém convertido para SCSI, pois por padrão ele vem em IDE. 
   	Outro ponto importante é que PCs com Windows 11 não devemos esquecer o TPM.

2. Baixar o Disk2vhd que será responsável por gerar o arquivo VHDX (https://learn.microsoft.com/pt-br/sysinternals/downloads/disk2vhd)

3. No PC físico (origem) devemos redimensionar o disco para o tamanho que realmente iremos utilizar. (Por ex: um disco de 1TB com utilização de 150Gb, redimensionar para pelo menos 250Gb)

4. Executar o Disk2vhd conforme o print:

   <img width="512" height="406" alt="image-20260825133649199" src="https://github.com/user-attachments/assets/b38d86b8-5ee8-420b-92b3-6eddc5a93a05" />


5. Após o arquivo ter sido gerado, vamos envia-lo ao Proxmox, esse processo pode ser feito via WinSCP, ou diretamente pelo scp no PowerShell.
      Por exemplo: `scp file.txt root@192.168.0.1:/directory/file.txt`
6. Vamos converter o arquivo VHDX para .qcow2 com o comando: `qemu-img convert -f vhdx -O qcow2 ARQUIVO.VHDX ARQUIVO.qcow2`
7. Carregar o módulo **nbd** para redimensionar o disco qcow2 com exatamente o tamanho que definimos no paço 3.
      Para isso, vamos utilizar alguns comandos, seguindo essa ordem:
      **`modprobe nbd max_part=16`**
      **`qemu-nbd --connect=/dev/nbd0 ARQUIVO.qcow2`**
      **`fdisk -l /dev/nbd0`**
      **`qemu-nbd --disconnect /dev/nbd0`**
      **`qemu-img resize --shrink ARQUIVO.qcow2 270G`**
8. Importar o disco qcow2 para a VM: `qm importdisk 117 ARQUIVO.qcow2 VMS`
   Uma explicação rápida: O ID da VM é uma numeração única no sistema e, a parte final do comando se refere em qual local de armazenamento será importado.
9. Vamos acessar o console Web do Proxmox e procurar a VM que importamos o qcow2, ao clicar em Hardware o disco está sendo mostrado como Unused, dar dois cliques nele e clicar em Add. Após isso, tentar realizar o Start da VM.
10. Com a VM online, instalar os drivers Virtio e reinicia-la. Após, podemos realizar o shutdown e alterar o tipo da controladora para SCSI
