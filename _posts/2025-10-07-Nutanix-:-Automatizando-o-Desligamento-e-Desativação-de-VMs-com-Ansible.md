## Nutanix: Automatizando o Desligamento e Desativação de VMs com Ansible

Em ambientes corporativos, é comum a necessidade de desativar máquinas virtuais (VMs) por motivos como encerramento de projetos, economia de recursos ou reestruturações. Este playbook Ansible foi desenvolvido para **automatizar o processo de desligamento e desativação de VMs** no Nutanix, garantindo rastreabilidade e padronização.

***

### 🎯 Objetivo

Desligar uma VM de forma segura e registrar sua desativação por meio de uma anotação personalizada, utilizando a CLI do Nutanix (`acli`). O alvo pode ser informado por **nome da VM** ou **endereço IP**.

***

### 📜 Estrutura do Playbook

```yaml
- name: Guest Shutdown and Disable VM
  hosts: 'all'
  gather_facts: no
  vars:
    target: ''                      # IP ou nome da VM
    string_vm: ''                   # Nome da VM resolvido
    new_description: 'VM Disabled'  # Anotação personalizada

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

    - debug:
        msg: '{\{ output_vm_name.stdout_lines }\}'
      when: target|regex_search('^\\d{1,3}(\\.\\d{1,3}){3}$')

  #- Registrar variáveis

    - name: Register variable - Host - If initial param is "HOSTNAME"
      set_fact:
          string_vm: "{\{ target }\}"
      when: target|regex_search('^[a-zA-Z]+.*')

    - name: Register variable - Host - If initial param is "IP"
      set_fact:
          string_vm: "{\{ output_vm_name.stdout | from_yaml }\}"
      when: target|regex_search('^\\d{1,3}(\\.\\d{1,3}){3}$')

    - name: Get VM ansible adjustment
      shell: |
          echo {\{ string_vm }\} | sed 's/-/_/g' | tr -d " "
      ignore_errors: false
      register: output_hostname

  #- Executar a troca dos dados da VM
    - name: Run Scrip - Update VM description
      shell: |
        echo "Realizando alteração:"
        /usr/local/nutanix/bin/acli vm.update {\{ string_vm }\} annotation="{\{ new_description }\}"
      register: output

  #- Retornar os dados na tela
    - debug:
        msg: "{\{ output.stdout_lines }\}"

  #- Desligar a VM
    - name: Shutdown VM
      shell: |
        echo "Shutting down..."
        /usr/local/nutanix/bin/acli vm.guest_shutdown {\{ string_vm }\}
      register: shutdown_output
    
    - debug:
        msg: "{\{ shutdown_output.stdout_lines }\}"

  #- Aguardar o desligamento da VM
    - name: Waiting VM poweroff
      shell: |
        state="kOn"
        while [ $state != "kOff" ]
        do
          state=$(/usr/local/nutanix/bin/acli vm.get {\{ string_vm }\} | grep "state:" | awk -F '"' '{print $2}' | tr -d " " )
          sleep 3 ;
        done
      async: 120
      ignore_errors: false
```

***

#### 🔍 1. Identificação da VM pelo IP

```yaml
- name: Get VM Name
  shell: |
    ...
  register: output_vm_name
  when: target|regex_search('^\\d{1,3}(\\.\\d{1,3}){3}$')
```

**Explicação:**  
Se o `target` for um IP, o script percorre as VMs do cluster e identifica aquela que possui o IP correspondente, ignorando VMs de gerenciamento como `NTNX`.

***

#### 🧠 2. Registro da variável `string_vm`

```yaml
- name: Register variable - Host - If initial param is "HOSTNAME"
  set_fact:
      string_vm: "{\{ target }\}"
```

```yaml
- name: Register variable - Host - If initial param is "IP"
  set_fact:
      string_vm: "{\{ output_vm_name.stdout | from_yaml }\}"
```

**Explicação:**  
Define a variável `string_vm` com o nome da VM, seja ele informado diretamente ou obtido via IP.

***

#### 🛠️ 3. Ajuste de nome para uso interno

```yaml
- name: Get VM ansible adjustment
  shell: |
      echo {\{ string_vm }\} | sed 's/-/_/g' | tr -d " "
```

**Explicação:**  
Realiza ajustes no nome da VM para evitar problemas com caracteres especiais em comandos subsequentes.

***

#### 📝 4. Atualização da descrição da VM

```yaml
- name: Run Scrip - Update VM description
  shell: |
    /usr/local/nutanix/bin/acli vm.update {\{ string_vm }\} annotation="{\{ new_description }\}"
```

**Explicação:**  
Adiciona uma anotação personalizada à VM, como "VM Disabled", podendo incluir data, ticket ou motivo da desativação.

***

#### ⏻ 5. Desligamento da VM

```yaml
- name: Shutdown VM
  shell: |
    /usr/local/nutanix/bin/acli vm.guest_shutdown {\{ string_vm }\}
```

**Explicação:**  
Executa o desligamento da VM de forma segura via `guest_shutdown`.

***

#### ⏳ 6. Aguardar desligamento completo

```yaml
- name: Waiting VM poweroff
  shell: |
    while [ $state != "kOff" ]; do ... done
  async: 120
```

**Explicação:**  
Monitora o estado da VM até que ela esteja completamente desligada (`kOff`), garantindo que o processo foi concluído com sucesso.

***

### ✅ Conclusão

Este playbook é uma ferramenta essencial para equipes de infraestrutura que precisam desativar VMs de forma segura, documentada e automatizada. Ele reduz o risco de erros manuais, melhora a rastreabilidade e economiza tempo em operações repetitivas. Ele pode ser utilizado em conjunto com ferramentas de agendamento de tarefas (AWX ou Rundeck por exemplo) para desativação planejada.