## Nutanix: Automatizando a Listagem de Snapshots com Ansible

![alt text](https://github.com/ph7ti/ph7ti.github.io/blob/main/_posts/imgs/Nutanix-Ansible.png?raw=true)

A praticidade de executar um snapshot no ambiente Nutanix pode as vezes nos deixar preguiços na ação de conferir e sanitizar oa ambiente. Quem nunca esqueceu no cluster snapshot de mais de 3 meses que atire a primeira pedra! (Duvido que venha pedra rsrs)...

Neste post, vamos explorar um exemplo prático de automação com **Ansible** para listar snapshots em um ambiente **Nutanix**. O script abaixo utiliza o módulo `shell` para executar comandos diretamente no terminal do host alvo, e exibe os snapshots existentes no cluster Nutanix.

### 🎯 Objetivo

Listar todos os snapshots presentes no cluster Nutanix e exibir suas informações de forma organizada.

### 📜 Código do Playbook

> Obs.: Remover o "\\" do YAML entre os cochetes {\\{ e }\\} das variáveis.

```yaml
- name: List Snapshots
  hosts: all

  tasks:

  - name: Run Script - Get all Snapshots
    shell: |
      #! /bin/bash
      echo "Snapshots presentes no Cluster"
      /usr/local/nutanix/bin/acli snapshot.list | sed 's/ \+ /;/g' | sed 's/ /_/g' | awk -F';' '{print $3,$4,$1}' OFS=' | '
    register: output_list

  - name: Output - Get all Snapshots
    debug:
      msg: "{\{ output_list.stdout_lines }\}"
```

### Explicação Passo a Passo

#### 1. **Definição do Playbook**

```yaml
- name: List Snapshots
  hosts: all
```

Este playbook será executado em todos os hosts definidos no inventário (`hosts: all`).

***

#### 2. **Execução do Script Bash**

```yaml
- name: Run Script - Get all Snapshots
  shell: |
    #! /bin/bash
    echo "Snapshots presentes no Cluster"
    /usr/local/nutanix/bin/acli snapshot.list | sed 's/ \+ /;/g' | sed 's/ /_/g' | awk -F';' '{print $3,$4,$1}' OFS=' | '
  register: output_list
```

*   **`acli snapshot.list`**: Comando da CLI Nutanix que lista todos os snapshots.
*   **`sed 's/ \+ /;/g'`**: Substitui múltiplos espaços por ponto e vírgula para facilitar o parsing.
*   **`sed 's/ /_/g'`**: Substitui espaços por underscores para evitar problemas com nomes compostos.
*   **`awk -F';' '{print $3,$4,$1}' OFS=' | '`**: Reorganiza os campos para exibir a data de criação, nome da VM e nome do snapshot, separados por `|`.

***

#### 3. **Exibição dos Resultados**

```yaml
- name: Output - Get all Snapshots
  debug:
    msg: "{\{ output_list.stdout_lines }\}"
```

Este passo utiliza o módulo `debug` para exibir as linhas de saída do comando anterior, mostrando os snapshots diretamente no terminal ou no log da execução do playbook.

***

### Conclusão

Esse playbook é uma forma simples e eficaz de visualizar snapshots em um ambiente Nutanix usando Ansible. Ele pode ser facilmente adaptado para incluir filtros, exportar os dados para arquivos ou até mesmo realizar ações como exclusão de snapshots antigos.

Se você trabalha com automação de infraestrutura e utiliza Nutanix, vale a pena incorporar esse tipo de script ao seu arsenal!

***