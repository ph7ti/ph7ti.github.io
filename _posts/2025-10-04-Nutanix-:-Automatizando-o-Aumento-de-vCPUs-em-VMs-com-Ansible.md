## Automatizando o Aumento de vCPUs em VMs Nutanix com Ansible

A escalabilidade de recursos em máquinas virtuais é uma tarefa comum na administração de ambientes virtualizados. Neste artigo, vamos explorar um **playbook Ansible** que automatiza o processo de **acréscimo de núcleos de CPU (vCPUs)** em uma VM Nutanix, com base em parâmetros definidos pelo usuário.

***

### 🎯 Objetivo

Permitir o aumento de vCPUs em uma VM Nutanix de forma automatizada, utilizando:

*   Nome da VM ou IP como referência.
*   Quantidade de núcleos desejada ou cálculo automático de 10% adicional.
*   Exibição do resultado da operação.

***

### 📄 Código do Playbook

```yaml
- name: Upgrade vCPU
  hosts: all
  vars:
    string_vm: ''   # Nome da VM resolvido
    int_vcpu: ''    # Quantidade de núcleos que deseja adicionar, se o campo estiver vazio vai adicionar 10%
    target: ''		  # Alvo pode ser o IP ou Nome da VM (não confundir com hostname do sistema)

  tasks:

#- Coletar Hostname da VM se "target" for endereço IP
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

  - name: Register variable - VM - If initial param is not "IP"
    set_fact:
        string_vm: "{\{ target }\}"
    when: target|regex_search('^[a-zA-Z]+.*')

  - debug:
      msg: '{\{ string_vm }\}'
    when: target|regex_search('^[a-zA-Z]+.*')

#- expandir cpu do host
  - name: Upgrade vCPUs units
    shell: |
      #!/bin/bash
      vm_to_up='{\{ string_vm }\}'
      vcpu_raise_more='{\{ int_vcpu }\}'
      output=$(/usr/local/nutanix/bin/acli vm.get $vm_to_up | grep '_vcpu')
      cores=$(echo $output | awk -F ' ' '{print $2}' | sed 's/ //')
      vcpu=$(echo $output | awk -F ' ' '{print $4}' | sed 's/ //')
      total=$(expr $cores \* $vcpu)
      echo "Total de CPUs atuais:" $total
      if   (($vcpu_raise_more < $cores)); then vcpu_raise_to=$(echo "scale=0; ($vcpu + 1)" | bc -l) ;
      elif [ "$(( $vcpu_raise_more % $cores ))" -eq 0 -o $vcpu_raise_more -lt $total ]; then vcpu_raise_to=$(echo "scale=0; ($vcpu_raise_more / $cores)+$vcpu" | bc -l) ;
      elif (( vcpu_raise_more == total + 1 )); then vcpu_raise_to=4 ;
      else vcpu_raise_to=$(echo "scale=0; ((($vcpu_raise_more / $cores)+($vcpu / $cores))*$cores)" | bc -l) ;
      fi
      echo "Status do processo de upgrade na VM:"
      /usr/local/nutanix/bin/acli vm.update $vm_to_up num_vcpus=$vcpu_raise_to
      output=$(/usr/local/nutanix/bin/acli vm.get $vm_to_up | grep '_vcpu')
      cores=$(echo $output | awk -F ' ' '{print $2}' | sed 's/ //')
      vcpu=$(echo $output | awk -F ' ' '{print $4}' | sed 's/ //')
      echo "Total de CPUs alterado para: "$(echo "$cores*$vcpu" | bc)
    register: output_script

#- Output da execução do script
  - name: Output run
    debug:
      msg: "{\{ output_script.stdout_lines }\}"

```

***

### 🧩 Explicação das Tarefas

#### 🔹 1. **Identificar a VM pelo IP**

```yaml
- name: Get VM Name
```

Se o parâmetro `target` for um IP, essa tarefa:

*   Lista todas as VMs do cluster.
*   Verifica o IP de cada VM usando `vm.nic_list`.
*   Quando encontra uma correspondência, registra o nome da VM.

> Isso é útil quando o usuário não sabe o nome exato da VM, mas tem o IP.

***

#### 🔹 2. **Registrar o Nome da VM**

```yaml
- name: Register variable - Host - If initial param is "IP"
- name: Register variable - VM - If initial param is not "IP"
```

Dependendo do tipo de entrada (`IP` ou `nome`), o playbook registra a variável `string_vm` com o nome correto da VM.

***

#### 🔹 3. **Exibir o Nome da VM**

```yaml
- name: Debug
```

Mostra no terminal o nome da VM que será usada na operação. Serve como validação visual.

***

#### 🔹 4. **Executar o Upgrade de vCPUs**

```yaml
- name: Upgrade vCPUs units
```

Essa é a tarefa principal do playbook. Ela:

1.  Coleta os valores atuais de `num_cores_per_vcpu` e `num_vcpus`.
2.  Calcula o total de CPUs atuais.
3.  Com base no valor de `int_vcpu`, decide:
    *   Se deve adicionar 10% automaticamente.
    *   Se deve aplicar o valor informado.
4.  Executa o comando `vm.update` para alterar a quantidade de vCPUs.
5.  Exibe o novo total de CPUs após a alteração.

> A lógica de cálculo é inteligente e tenta manter a proporção entre núcleos e vCPUs.

***

#### 🔹 5. **Exibir Resultado da Execução**

```yaml
- name: Output run
```

Mostra no terminal a saída do script de upgrade, incluindo:

*   Total de CPUs antes e depois.
*   Comando executado.
*   Status da operação.

***

### ✅ Benefícios

*   **Automação completa**: Desde a identificação da VM até a alteração de recursos.
*   **Flexibilidade**: Aceita IP ou nome da VM como entrada.
*   **Segurança**: Exibe os dados antes e depois da operação.
*   **Escalabilidade**: Pode ser adaptado para múltiplas VMs ou integrado a pipelines de CI/CD.

***

### ✅ Conclusão

Esse playbook é uma excelente base para automatizar a expansão automática de CPU nas VMs em ambientes Nutanix. Ele pode ser utilizado em conjunto com ambientes de monitoramento (Zabbix por exemplo) para expandir automaticamente recursos caso ocorra seu esgotamento.