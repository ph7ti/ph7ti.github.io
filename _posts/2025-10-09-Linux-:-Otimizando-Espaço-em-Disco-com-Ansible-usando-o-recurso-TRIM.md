# Linux: Otimizando Espaço em Disco com Ansible usando o recurso TRIM

![Linux: Otimizando Espaço em Disco com Ansible usando o recurso TRIM](https://github.com/ph7ti/ph7ti.github.io/blob/main/_posts/imgs/Linux-TRIM-Ansible.png?raw=true)

Manter o desempenho e a saúde dos discos em ambientes Linux é essencial, especialmente em máquinas virtuais ou SSDs. Uma das práticas recomendadas é o uso do comando `fstrim`, que informa ao sistema operacional quais blocos de dados não estão mais em uso e podem ser liberados pelo dispositivo de armazenamento. Recentemente tive que fazer uso desse recurso em VMs legadas (CentOS 7 e Ubuntu 18) em um ambiente Nutanix com SSDs.

Neste post, vamos explorar um playbook Ansible que automatiza essa tarefa, verificando o status do disco e executando o TRIM de forma segura e te poupando um bom tempo.

## 🎯 Objetivo

Este playbook realiza as seguintes ações:

1.  Verifica o uso da partição raiz.
2.  Confere se o disco suporta descarte de dados (`discard`).
3.  Executa o comando `fstrim` em todos os discos.
4.  Habilita e verifica o status do serviço `fstrim.timer`.

## 📜 Estrutura do Playbook

> Obs.: Remover o "\\" do YAML entre os cochetes {\\{ e }\\} das variáveis.

```yaml
- name: Trim DISK
  hosts: all
  gather_facts: yes

  tasks:

  - name: Check disk space in root partition
    shell: |
      df -h /
    register: root_partition

  - debug:
      msg: "{\{ root_partition.stdout_lines }\}"

  - name: Check discard status
    shell: |
      cat /sys/block/sda/queue/discard_zeroes_data
    register: discard_status

  - debug:
      msg: "{\{ discard_status.stdout_lines }\}"

  - name: Execute FSTRIM on disks
    shell: |
      /sbin/fstrim -v --all || true
    register: trim_return

  - debug:
      msg: "{\{ trim_return.stdout_lines }\}"

  - name: Enable FSTRIM task on VM
    shell: |
      if [[ $(systemctl is-active fstrim.timer) == "active" ]]; then
        systemctl status fstrim.timer
      else
        systemctl enable --now fstrim.timer ; systemctl status fstrim.timer
      fi
    register: trim_status

  - debug:
      msg: "{\{ trim_status.stdout_lines }\}"
```

## Explicação das Tarefas

### 1. `Check disk space in root partition`

Executa o comando `df -h /` para verificar o uso da partição raiz. Isso ajuda a entender o espaço disponível antes de aplicar o TRIM.

### 2. `Check discard status`

Verifica se o disco suporta operações de descarte (`discard`) lendo o arquivo `/sys/block/sda/queue/discard_zeroes_data`. Um valor diferente de zero indica suporte ao TRIM.

### 3. `Execute FSTRIM on disks`

Executa o comando `/sbin/fstrim -v --all`, que aplica o TRIM em todas as partições montadas que suportam a operação. O `|| true` garante que o playbook continue mesmo se algum disco não suportar TRIM.

### 4. `Enable FSTRIM task on VM`

Verifica se o serviço `fstrim.timer` está ativo. Caso não esteja, ele é habilitado e iniciado. Esse serviço garante que o TRIM seja executado automaticamente em intervalos regulares.

### 5. `debug` tasks

Cada etapa tem uma tarefa de debug que exibe os resultados diretamente na saída do playbook, facilitando a análise e validação.

## Conclusão

Este playbook é uma ferramenta poderosa para administradores que desejam manter seus discos otimizados e garantir que o espaço não utilizado seja corretamente liberado. Automatizar o TRIM com Ansible reduz o esforço manual e melhora a performance de sistemas baseados em SSD ou ambientes virtualizados.
