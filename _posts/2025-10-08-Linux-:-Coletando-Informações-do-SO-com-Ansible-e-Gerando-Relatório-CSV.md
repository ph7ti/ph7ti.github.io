# Linux: Coletando Informações do SO com Ansible e Gerando Relatório CSV

Neste post, vamos explorar um playbook Ansible simples e eficiente que coleta informações do sistema operacional de máquinas Linux e gera um relatório em formato CSV. Esse tipo de automação é útil para administradores que precisam manter inventários atualizados de seus servidores em grandes ambientes.

## Objetivo do Playbook

O playbook tem como objetivo coletar dados como IP, sistema operacional, versão da distribuição, hostname e versão do kernel de cada máquina gerenciada e salvar essas informações em um arquivo CSV.

## Estrutura do Playbook

```yaml
---
- name: Generate a CSV file of server informations
  hosts: all
  gather_facts: true

  vars:
    csv_path: ./
    csv_filename: report.csv
    headers: IP;OS;Distro Ver;Hostname;Kernel

  tasks:

  - name: Save CSV headers
    ansible.builtin.lineinfile:
      dest: "{\{ csv_path }\}/{\{ csv_filename }\}"
      line: "{\{ headers }\}"
      create: true
      state: present
    delegate_to: localhost
    run_once: true

  - name: Build out CSV file by facts
    ansible.builtin.lineinfile:
      dest: "{\{ csv_path }\}/{\{ csv_filename }\}"
      line: "{\{ inventory_hostname }\};{\{ ansible_distribution }\};{\{ ansible_distribution_version }\};{\{ ansible_fqdn }\};{\{ ansible_kernel }\}"
      create: true
      state: present
    delegate_to: localhost
    when: ansible_facts is defined

  - name: Read in CSV to variable
    read_csv:
      path: "{\{ csv_path }\}/{\{ csv_filename }\}"
    register: csv_file
    delegate_to: localhost
    run_once: true
```

## Explicação das Tarefas

### 1. `Save CSV headers`

Esta tarefa grava os cabeçalhos no arquivo CSV. Ela é executada apenas uma vez (`run_once: true`) e localmente (`delegate_to: localhost`), garantindo que o arquivo seja criado com os títulos corretos antes de adicionar os dados dos hosts.

### 2. `Build out CSV file by facts`

Aqui, o playbook coleta os fatos do sistema de cada host e escreve uma linha no CSV com os dados formatados. A tarefa também é executada localmente, mas para cada host, e depende da existência dos `ansible_facts`.

### 3. `Read in CSV to variable`

Por fim, o conteúdo do CSV é lido e armazenado em uma variável (`csv_file`). Isso pode ser útil para validações posteriores ou para uso em outras tarefas dentro do playbook.

## Conclusão

Este playbook é uma solução prática para gerar relatórios de inventário de servidores Linux usando Ansible. Ele automatiza a coleta de informações essenciais e organiza os dados em um formato fácil de consultar e compartilhar. Ideal para equipes de infraestrutura que buscam mais visibilidade e controle sobre seus ambientes.
