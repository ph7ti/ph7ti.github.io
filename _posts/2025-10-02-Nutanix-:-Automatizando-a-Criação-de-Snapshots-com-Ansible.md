## Nutanix: Automatizando a Criação de Snapshots com Ansible

A automação de tarefas rotineiras é essencial para garantir agilidade e consistência na administração de ambientes de infraestrutura. Em um cenário de manutenção planejada, é importante garantir que os snapshots estarão concluídos antes de iniciar as atividades.

Neste artigo, vamos explorar um **playbook Ansible** que realiza a **criação de snapshots de VMs** em um cluster **Nutanix**, utilizando a CLI `acli`.

### Objetivo

Criar snapshots de máquinas virtuais (VMs) em um ambiente Nutanix, utilizando Ansible para identificar a VM (por nome ou IP) e executar o comando de snapshot.

### Estrutura do Playbook

> Obs.: Remover o "\\" do YAML entre os cochetes {\\{ e }\\} das variáveis.

```yaml
- name: Create Snapshots
  hosts: all
  vars:
    target: ''          # Alvo pode ser o IP ou Nome da VM (não confundir com hostname do sistema)
    string_vm: ''       # Nome da VM resolvido
    string_snapshot: '' # Nome do snapshot a ser criado
    
  tasks:

  - name: Get VM Name
    shell: |
      host_ip="{\{ target }\}"
      if [ $(echo $host_ip | grep -Eo '^(([1-9]?[0-9]|1[0-9][0-9]|2([0-4][0-9]|5[0-5]))\.){3}([1-9]?[0-9]|1[0-9][0-9]|2([0-4][0-9]|5[0-5]))$') != 1 ] ; then
        get_vms=$(/usr/local/nutanix/bin/acli vm.list | tail -n +2 | awk -F ' ' '{print $1"\n"}' | grep -v 'NTNX\|ntnx' | sed 's/$/\n/g')
        for vm in $get_vms ; do
            vm_ip=$(/usr/local/nutanix/bin/acli vm.nic_list "$vm" | tail -n +2 | awk -F ' ' '{print $3}')
            if [[ $vm_ip =~ $host_ip ]] ; then
                echo $vm
                break
            fi
        done
      fi
    ignore_errors: false
    register: output_vm_name
    when: target|regex_search('^\\d{1,3}(\\.\\d{1,3}){3}$')

  - name: Register variable - Host - If initial param is "IP"
    set_fact:
        string_vm: "{\{ output_vm_name.stdout | from_yaml }\}"
    when: target|regex_search('^\\d{1,3}(\\.\\d{1,3}){3}$')

  - name: Register variable - VM - If initial param is not "IP"
    set_fact:
        string_vm: "{\{ target }\}"
    when: target|regex_search('^[a-zA-Z]+.*$')

  - debug:
      msg: '{\{ string_vm }\}'
    when: target|regex_search('^[a-zA-Z]+.*$')

  - name: Run Snapshot
    shell: |
      snapshot_name='{\{ string_snapshot }\}'
      echo "/usr/local/nutanix/bin/acli vm.snapshot_create '{\{ string_vm }\}' snapshot_name_list='$snapshot_name'"
      /usr/local/nutanix/bin/acli vm.snapshot_create '{\{ string_vm }\}' snapshot_name_list='"'$snapshot_name'"'
      /usr/local/nutanix/bin/acli vm.snapshot_list '{\{ string_vm }\}'
    register: output_script

  - name: Output Snapshot run
    debug:
      msg: "{\{ output_script.stdout_lines }\}"
```

***

### Explicação Passo a Passo

#### 1. **Definição de Variáveis**

```yaml
vars:
  target: ''           # Pode ser o nome da VM ou o IP
  string_vm: ''        # Nome da VM resolvido
  string_snapshot: ''  # Nome do snapshot a ser criado
```

***

#### 2. **Task "Get VM Name": Identificação da VM por IP**

Se o valor de `target` for um IP, o script executa uma busca entre as VMs do cluster para encontrar aquela que possui esse IP.

*   Filtra VMs que não sejam de gerenciamento (`NTNX`).
*   Verifica o IP de cada VM usando `vm.nic_list`.
*   Quando encontra uma correspondência, registra o nome da VM.

***

#### 3. **Task "Register variable": Registro da VM**

Dependendo do tipo de entrada (`IP` ou `nome`), o playbook registra a variável `string_vm` com o nome correto da VM.

***

#### 4. **Task "Run Snapshot":Criação do Snapshot e Validação**

```yaml
/usr/local/nutanix/bin/acli vm.snapshot_create '{\{ string_vm }\}' snapshot_name_list='"'$snapshot_name'"'
```

Este comando cria o snapshot da VM identificada, com o nome definido em `string_snapshot`.

```yaml
/usr/local/nutanix/bin/acli vm.snapshot_list '{\{ string_vm }\}'
```
Após a criação, o playbook lista os snapshots da VM para confirmar a execução.

***

### Conclusão

Esse playbook é uma excelente base para automatizar a criação de snapshots em ambientes Nutanix. Ele pode ser expandido para incluir agendamento, retenção, ou integração com sistemas de backup e monitoramento.

***