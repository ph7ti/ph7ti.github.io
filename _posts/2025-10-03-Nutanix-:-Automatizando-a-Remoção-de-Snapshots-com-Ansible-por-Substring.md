## Nutanix: Automatizando a Remoção de Snapshots com Ansible por Substring

Snapshots são recursos valiosos para proteção e recuperação de dados, em ações rápidas nas VMs, mas podem consumir espaço significativo se não forem gerenciados adequadamente. A falta de sanitização de snapshots muitas vezes gera alertas de limite de armazenamento "topando" o threshold.

Neste artigo, vamos explorar um **playbook Ansible** que automatiza a **remoção de snapshots** em um ambiente **Nutanix**, com base em uma substring no nome do snapshot.

***

### 🎯 Objetivo

Remover snapshots de VMs Nutanix que contenham uma determinada substring no nome, com opção de **execução em modo de teste (dry-run)** e **registro em log**.

***

### 📄 Código do Playbook

> Obs.: Remover o "\\" do YAML entre os cochetes {\\{ e }\\} das variáveis.

```yaml
- name: Delete Snapshots
  hosts: all
  vars:
    dry_run: 'false'
    output_local_file: './Snapshots_removidos.log'          # Log local no servidor Ansible
    output_file: '/home/nutanix/Snapshots_removidos.log'    # Log no servidor Nutanix
    string_snapshot: ''                                     # Nome da VM resolvido

  tasks:

  - name: Remove old Snapshots
    shell: |
      #! /bin/bash
      keyword='{\{ string_snapshot }\}'
      outputfile='{\{ output_file }\}'
      debug='{\{ dry_run }\}'
      echo "" > $outputfile
      get_list=$(/usr/local/nutanix/bin/acli snapshot.list | sed 's/ \+ /;/g' | sed 's/ /_/g' | grep "$keyword")
      for item in $get_list ; do
          echo $item >> $outputfile
          id=$( echo $item | sed 's/ /\n/g' | awk -F';' '{print $2}')
          /usr/local/nutanix/bin/acli snapshot.get $id | grep "name:" | tail -n +2 | awk -F':' '{print $2}' >> $outputfile
          if [ $debug = false ] ; then
            echo "yes" | /usr/local/nutanix/bin/acli snapshot.delete $id >> $outputfile
          else
            echo "/usr/local/nutanix/bin/acli snapshot.delete $id" >> $outputfile
          fi
      done
      cat $outputfile
    register: output_script
    when: string_snapshot is defined and string_snapshot | length > 0

  - name: Debug - Output Snapshot run - Show errors
    debug:
      msg: "{\{ output_script.stdout_lines }\}"
    when: string_snapshot is defined and string_snapshot | length > 0 and output_script.stdout_lines | length > 0

# Tasks abaixo servem apenas para debug ou registro para evidência, coletando o log gerado no servidor Nutanix
  - name: Collected data
    shell: cat '{\{ output_file }\}'
    register: o_file
    tags: debug
    when: string_snapshot is defined and string_snapshot | length > 0

  - name: Output script run
    debug:
      msg: "{\{ o_file.stdout_lines }\}"
    tags: debug, output
    when: string_snapshot is defined and string_snapshot | length > 0

  - name: Delete old file
    file:
      dest: "{\{ output_local_file }\}"
      state: absent
    delegate_to: localhost
    run_once: true
    tags: output_log

  - name: Create file
    file:
      dest: "{\{ output_local_file }\}"
      state: touch
    delegate_to: localhost
    run_once: true
    tags: output_log

  - name: Build out LOG file
    lineinfile:
      path: "{\{ output_local_file }\}"
      line: "{\{ o_file.stdout }\}"
      create: true
      state: present
    delegate_to: localhost
    tags: output_log
```

***

### 🧩 Explicação Passo a Passo

#### 1. **Definição de Variáveis**

*   `string_snapshot`: Substring usada para identificar os snapshots a serem removidos.
*   `dry_run`: Define se a execução será apenas de teste (`true`) ou efetiva (`false`).
*   `output_file`: Caminho do log no servidor Nutanix.
*   `output_local_file`: Caminho do log local no servidor Ansible.

***

#### 2. **Task "Remove old Snapshots": Execução do Script de Remoção**

Essa tarefa executa um script Bash que:

*   Recebe a substring (string_snapshot) como filtro.
*   Lista todos os snapshots que contêm essa substring.
*   O script busca todos os snapshots que contenham a substring definida.
*   Se `dry_run` for `false`, os snapshots são removidos.
*   Se `dry_run` for `true`, apenas o comando de remoção é exibido no log (sem execução).

***

#### 3. **Registro e Exibição dos Logs**

*   O conteúdo do log gerado no servidor Nutanix é coletado.
*   Exibe no terminal a saída do script anterior (stdout_lines), útil para verificar se os snapshots foram encontrados e se houve erros durante a execução.
*   O log é copiado para um arquivo local no servidor Ansible.
*   Isso permite auditoria e rastreabilidade das ações realizadas.

***

#### 4. **Exemplo de Execução**

Substitua os espaços ( ) por underline (_). Exemplo: 'Before upgrade' -> 'Before_upgrade'

```bash
ansible-playbook -i host.yml remove_snapshots.yml --extra-vars "string_snapshot=Before_upgrade dry_run=true"
```

***

### ✅ Benefícios

*   **Segurança**: O modo `dry_run` permite testar antes de executar.
*   **Automação**: Reduz o esforço manual na limpeza de snapshots.
*   **Auditoria**: Geração de logs locais e remotos para rastreabilidade.

***

### ✅ Conclusão

Esse playbook é uma solução robusta para:

*   **Identificar e remover snapshots** com base em nomes.
*   **Testar antes de executar** com `dry_run`.
*   **Registrar logs** local e remotamente para auditoria.
