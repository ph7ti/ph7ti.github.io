## Nutanix: Automatizando o Upgrade de Memória em VMs com Ansible (v1)

A gestão de recursos em ambientes virtualizados pode ser desafiadora, especialmente quando há necessidade de escalar rapidamente a capacidade de uma VM. Em ambientes críticos, se um (ou vários) sistema(s) para(m) por falta de recursos em horário de pico podemos lembrar da expressão popular: "barata-voa". Logo a possibilidade de automação de expansão vertical de recursos pode salvar sua pele.

Neste post, vamos explorar um playbook Ansible que automatiza o processo de aumento de memória em uma máquina virtual (VM) no ambiente Nutanix. Essa automação é especialmente útil para times de infraestrutura que buscam agilidade e padronização nas operações do dia a dia.

### Objetivo

O playbook tem como objetivo identificar uma VM (por nome ou IP) e aumentar sua memória em 10%, utilizando comandos da ferramenta `acli` do Nutanix.

***

### Estrutura do Playbook

> Obs.: Remover o "\\" do YAML entre os cochetes {\\{ e }\\} das variáveis.

```yaml
- name: Upgrade Memory
  hosts: all
  vars:
    string_vm: ''   # Nome da VM resolvido
    target: ''  # Alvo pode ser o IP ou Nome da VM (não confundir com hostname do sistema)

  tasks:

#Coletar Hostname da VM
  - name: Get VM Name
    shell: |
      host_ip="{\{ target }\}"
      if [ $(echo $host_ip | grep -Eo '^(([1-9]?[0-9]|1[0-9][0-9]|2([0-4][0-9]|5[0-5]))\.){3}([1-9]?[0-9]|1[0-9][0-9]|2([0-4][0-9]|5[0-5]))$') != 1 ] ; then
        get_vms=$(/usr/local/nutanix/bin/acli vm.list | tail -n +2 | awk -F ' ' '{print $1"\n"}' | grep -v 'NTNX\|ntnx' | sed 's/$/\n/g')   # Obs.: Filtro do "grep" para desconsiderar as VMs de gerenciamento do cluster, customize se precisar acrescentar outras VMs
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

#- Registrar variáveis de IP e Hostname

  - name: Register variable - Host - If initial param is "IP"
    set_fact:
        string_vm: "{\{ output_vm_name.stdout | from_yaml }\}"
    when: target|regex_search('^\\d{1,3}(\\.\\d{1,3}){3}$')

  - name: Register variable - VM
    set_fact:
        string_vm: "{\{ target }\}"
    when: target|regex_search('^[a-zA-Z]+.*')

  - debug:
      msg: '{\{ string_vm }\}'
    when: target|regex_search('^[a-zA-Z]+.*')

#- expandir memory do host
  - name: Run Scrip - UP MEMORY 10% Truncate
    shell: |
      #!/bin/bash
      vm_to_up='{\{ string_vm }\}'
      memory=$(/usr/local/nutanix/bin/acli vm.get $vm_to_up | grep 'memory_mb' -m 1 | awk '{print $2}')
      memory_GB=$(awk "BEGIN {printf \"%.0f \", $memory/1024}")
      echo "Total de Memória (GB) atual:" $memory_GB
      if (( $memory_GB > 4 )) ; then
        memory_up_to=$(awk "BEGIN {printf \"%.0f \", $memory_GB*1.1}" | sed 's/ /G/')
      else
        memory_up_to=$( expr $memory_GB + 1 | awk '{print $1"G"}')
      fi
      /usr/local/nutanix/bin/acli vm.update $vm_to_up memory=$memory_up_to
      echo "Status do processo de upgrade na VM:"
      memory=$(/usr/local/nutanix/bin/acli vm.get $vm_to_up | grep 'memory_mb' -m 1 | awk '{print $2}')
      memory_GB=$(awk "BEGIN {printf \"%.0f \", $memory/1024}")
      echo "Total de Memória (GB) alterado para:" $memory_GB
    register: output_script

#- Output da execução do script
  - name: Output run
    debug:
      msg: "{\{ output_script.stdout_lines }\}"

```

#### 1. Coletar o nome da VM a partir do IP

```yaml
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
```

**Explicação:**  
Se o `target` for um IP, o script busca entre as VMs do cluster aquela que possui esse IP associado, ignorando as VMs de gerenciamento (como as que contêm "NTNX").

***

#### 2. Registrar o nome da VM

```yaml
  - name: Register variable - Host - If initial param is "IP"
    set_fact:
        string_vm: "{\{ output_vm_name.stdout | from_yaml }\}"
    when: target|regex_search('^\\d{1,3}(\\.\\d{1,3}){3}$')

  - name: Register variable - VM
    set_fact:
        string_vm: "{\{ target }\}"
    when: target|regex_search('^[a-zA-Z]+.*')
```

**Explicação:**  
Dependendo se o `target` é um IP ou um nome, o playbook define a variável `string_vm` com o nome correto da VM.

***

#### 3. Exibir o nome da VM

```yaml
  - debug:
      msg: '{\{ string_vm }\}'
    when: target|regex_search('^[a-zA-Z]+.*')
```

**Explicação:**  
Exibe o nome da VM para fins de verificação, caso o `target` seja um nome.

***

#### 4. Executar o script de aumento de memória

```yaml
  - name: Run Scrip - UP MEMORY 10% Truncate
    shell: |
      #!/bin/bash
      vm_to_up='{\{ string_vm }\}'
      memory=$(/usr/local/nutanix/bin/acli vm.get $vm_to_up | grep 'memory_mb' -m 1 | awk '{print $2}')
      memory_GB=$(awk "BEGIN {printf \"%.0f \", $memory/1024}")
      echo "Total de Memória (GB) atual:" $memory_GB
      if (( $memory_GB > 4 )) ; then
        memory_up_to=$(awk "BEGIN {printf \"%.0f \", $memory_GB*1.1}" | sed 's/ /G/')
      else
        memory_up_to=$( expr $memory_GB + 1 | awk '{print $1"G"}')
      fi
      /usr/local/nutanix/bin/acli vm.update $vm_to_up memory=$memory_up_to
      echo "Status do processo de upgrade na VM:"
      memory=$(/usr/local/nutanix/bin/acli vm.get $vm_to_up | grep 'memory_mb' -m 1 | awk '{print $2}')
      memory_GB=$(awk "BEGIN {printf \"%.0f \", $memory/1024}")
      echo "Total de Memória (GB) alterado para:" $memory_GB
    register: output_script
```

**Explicação:**  
Este script coleta a memória atual da VM, calcula o novo valor (10% a mais ou +1 GB), aplica a alteração via `acli`, e exibe o novo valor de memória.

***

#### 5. Exibir resultado da execução

```yaml
  - name: Output run
    debug:
      msg: "{\{ output_script.stdout_lines }\}"
```

**Explicação:**  
Mostra no terminal o resultado da execução do script, incluindo a memória antes e depois do upgrade.

***

### Conclusão

Este playbook é uma ferramenta poderosa para administradores de ambientes Nutanix que desejam automatizar o processo de upgrade de memória em VMs. Ele reduz o tempo de execução, evita erros manuais e garante consistência nas alterações. Ele pode ser utilizado em conjunto com ambientes de monitoramento (Zabbix por exemplo) para expandir automaticamente recursos caso ocorra seu esgotamento.
