# Nutanix: Automatizando o Aumento de Memória em VMs com Ansible (v2)

A gestão de recursos em ambientes virtualizados pode ser desafiadora, especialmente quando há necessidade de escalar rapidamente a capacidade de uma VM. Este playbook Ansible foi criado para facilitar o processo de **aumento de memória** em VMs Nutanix, de forma segura, automatizada e parametrizável.

***

## 🎯 Objetivo

Permitir que o operador aumente a memória de uma VM específica (identificada por nome ou IP) em uma quantidade definida (em GB), utilizando comandos da CLI do Nutanix (`acli`).

***

## 📜 Estrutura do Playbook

```yaml
- name: Upgrade Memory (v2)
  hosts: all
  vars:
    string_vm: ''   # Nome da VM resolvido
    int_memory: ''  # Quantos GB deseja aumentar
    target: ''  	# Alvo pode ser o IP ou Nome da VM (não confundir com hostname do sistema)

  tasks:

#Coletar Hostname da VM
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
  - name: Run Scrip - UP MEMORY
    shell: |
      #!/bin/bash
      vm_to_up='{\{ string_vm }\}'
      memory_raise_more='{\{int_memory}\}'
      memory=$(/usr/local/nutanix/bin/acli vm.get $vm_to_up | grep 'memory_mb' -m 1 | awk '{print $2}')
      memory_GB=$(awk "BEGIN {printf \"%.0f \", $memory/1024}")
      echo "Total de Memória (GB) atual:" $memory_GB
      memory_raise_to=$( expr $memory_GB + $memory_raise_more | awk '{print $1"G"}')
      /usr/local/nutanix/bin/acli vm.update $vm_to_up memory=$memory_raise_to
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

***

### 🔍 1. Identificação da VM pelo IP

```yaml
  - name: Get VM Name
    shell: |
      host_ip="{\{ target }\}"
      ...
    register: output_vm_name
    when: target|regex_search('^\\d{1,3}(\\.\\d{1,3}){3}$')
```

**Explicação:**  
Se o `target` for um IP, o script percorre todas as VMs do cluster e compara os IPs associados até encontrar a correspondente. Isso evita a necessidade de saber o nome exato da VM.

***

### 🧠 2. Registro da variável `string_vm`

```yaml
  - name: Register variable - Host - If initial param is "IP"
    set_fact:
        string_vm: "{\{ output_vm_name.stdout | from_yaml }\}"
```

```yaml
  - name: Register variable - VM
    set_fact:
        string_vm: "{\{ target }\}"
```

**Explicação:**  
Define a variável `string_vm` com o nome da VM, seja ele obtido via IP ou diretamente informado.

***

### 🖥️ 3. Exibição do nome da VM

```yaml
  - debug:
      msg: '{\{ string_vm }\}'
```

**Explicação:**  
Exibe o nome da VM para conferência, útil para logs e validação.

***

### 🚀 4. Execução do script de aumento de memória

```yaml
  - name: Run Scrip - UP MEMORY
    shell: |
      ...
    register: output_script
```

**Explicação:**  
Este script:

1.  Obtém a memória atual da VM.
2.  Soma o valor definido em `int_memory`.
3.  Aplica a nova configuração com `acli vm.update`.
4.  Exibe a nova quantidade de memória alocada.

***

### 📤 5. Exibição do resultado

```yaml
  - name: Output run
    debug:
      msg: "{\{ output_script.stdout_lines }\}"
```

**Explicação:**  
Mostra o resultado da execução, incluindo a memória antes e depois da alteração.

***

## ✅ Conclusão

Este playbook é uma solução prática e eficiente para aumentar a memória de VMs em ambientes Nutanix. Ele reduz o esforço manual, evita erros operacionais e permite escalar recursos de forma controlada e auditável. Ideal para times de DevOps e infraestrutura que buscam automação e padronização. Ele pode ser utilizado em conjunto com ambientes de monitoramento (Zabbix por exemplo) para expandir automaticamente recursos caso ocorra seu esgotamento.
