# 🧩 Modelo YAML Inicial para Playbooks Ansible no Nutanix

![alt text](https://github.com/ph7ti/ph7ti.github.io/blob/main/_posts/imgs/Nutanix-Ansible.png?raw=true)

Você já se pegou pensando: *“Como estruturar meu inventário Ansible para começar a automatizar tarefas no Nutanix?”* Se sim, essa postagem é pra você!

Neste artigo, vamos mostrar um modelo simples e funcional de arquivo YAML que serve como ponto de partida para executar playbooks Ansible em ambientes Nutanix. Ideal para quem está começando ou quer uma base limpa para expandir conforme o ambiente cresce.

## 🧾 Código Completo

```yaml
all:
  children:
    NTNXCL01:
      hosts:
        10.x.x.x
      vars:
        ansible_user: 
          "nutanix"
        ansible_password:
          'nutanix/4u'
```

## 🔍 Explicação Passo a Passo

### Definindo o escopo geral

```yaml
all:
```

Aqui estamos iniciando o inventário com o grupo `all`, que é o grupo raiz padrão no Ansible. Todos os hosts e grupos definidos estarão dentro dele.

### Criando um grupo de hosts

```yaml
  children:
    NTNXCL01:
```

Dentro de `children`, criamos um grupo chamado `NTNXCL01`. Esse nome pode representar um cluster Nutanix específico ou qualquer agrupamento lógico que faça sentido no seu ambiente.

### Listando os hosts

```yaml
      hosts:
        10.x.x.x
```

Aqui definimos o IP do host (neste caso, uma CVM do Nutanix). Substitua `10.x.x.x` pelo IP real da sua máquina.

### Definindo variáveis de conexão

```yaml
      vars:
        ansible_user: 
          "nutanix"
        ansible_password:
          'nutanix/4u'
```

Essas variáveis são essenciais para que o Ansible consiga se conectar ao host. O `ansible_user` e `ansible_password` são usados para autenticação. No exemplo, usamos o usuário padrão `nutanix` e a senha padrão `nutanix/4u`, mas é altamente recomendável que você altere isso para credenciais seguras no seu ambiente real.

## ✅ Conclusão

Esse modelo é direto ao ponto e serve como base para qualquer automação com Ansible em ambientes Nutanix. Com ele, você já pode começar a rodar seus playbooks e ganhar tempo na administração do ambiente.

Lembre-se: conforme seu ambiente cresce, você pode expandir esse inventário com múltiplos clusters, grupos e variáveis específicas para cada cenário.

Curtiu o modelo? Adapte conforme sua realidade e compartilhe com a galera do time DevOps! 🚀