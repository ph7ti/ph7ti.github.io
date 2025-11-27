# **Automatizando Backup de Sites para GitLab com Shell Script**

![alt text](https://github.com/ph7ti/ph7ti.github.io/blob/main/_posts/imgs/Bash-Backup-Sites-GitLab.png?raw=true)

Você já se pegou fazendo backup manual do seu site, baixando arquivos via FTP e depois subindo para o GitLab? Além de ser demorado, é um processo sujeito a erros. Que tal automatizar tudo isso com um simples script em **Bash**?

Neste artigo, vou mostrar um script que compacta os arquivos do site em um servidor Linux, baixa para um diretório local e envia para um repositório no GitLab. Tudo isso com um único comando!

***

## **Script Completo**

```bash
#!/bin/bash

if [[ $1 ]] ; then
        echo "Remoção do arquivo baixado"
        rm /home/contoso_corp/site/public_html_prod.tar.gz
        rm -rf /home/contoso_corp/site/Prod/*
fi
echo "Iniciando acesso para compactação..."
ssh contoso@ftp.contoso.hospedagemdesites.ws "tar -zcpf public_html_prod.tar.gz public_html &"
echo "Compactação OK!"
echo "Iniciando Download..."
scp -o Cipher=arcfour contoso@ftp.contoso.hospedagemdesites.ws:public_html_prod.tar.gz /home/contoso_corp/site/public_html_prod.tar.gz
echo "Download OK!"
echo "Iniciando extração do arquivo compactado"
tar -xzf /home/contoso_corp/site/public_html_prod.tar.gz -C /home/contoso_corp/site/Prod/
echo "Extração OK!"
echo "Iniciando processos do GIT"
cd /home/contoso_corp/site/Prod/
git init
git config --local user.email "nome.sobrenome@contoso.com.br"
git config --local user.name "Meu User Name"
git config --list
git remote add origin git@gitlab.devops.contoso.com.br:tic/site-institucional-contoso.git
git checkout -b prod
git add .
git commit -m "TIC - Automatic Backup Script"
git push -f -u origin prod
echo "Processo do GIT Finalizado"
echo "FIM"
```

***

## **Explicação Passo a Passo**

### **1. Limpeza de arquivos antigos**

```bash
if [[ $1 ]] ; then
    rm /home/contoso_corp/site/public_html_prod.tar.gz
    rm -rf /home/contoso_corp/site/Prod/*
fi
```

Esse trecho remove arquivos antigos caso o script seja chamado com um parâmetro. É útil para evitar acúmulo de backups.

***

### **2. Compactação no servidor remoto**

```bash
ssh contoso@ftp.contoso.hospedagemdesites.ws "tar -zcpf public_html_prod.tar.gz public_html &"
```

Aqui, o script acessa o servidor via SSH e compacta a pasta `public_html`. O uso do `tar` com `-zcpf` garante compressão eficiente.

***

### **3. Download do arquivo compactado**

```bash
scp -o Cipher=arcfour contoso@ftp.contoso.hospedagemdesites.ws:public_html_prod.tar.gz /home/contoso_corp/site/public_html_prod.tar.gz
```

O `scp` faz a transferência do arquivo para o diretório local. A opção `Cipher=arcfour` acelera a cópia.

***

### **4. Extração do backup**

```bash
tar -xzf /home/contoso_corp/site/public_html_prod.tar.gz -C /home/contoso_corp/site/Prod/
```

Descompacta os arquivos no diretório `Prod`, preparando para o versionamento.

***

### **5. Versionamento com Git**

```bash
cd /home/contoso_corp/site/Prod/
git init
git config --local user.email "nome.sobrenome@contoso.com.br"
git config --local user.name "Meu User Name"
git remote add origin git@gitlab.devops.contoso.com.br:tic/site-institucional-contoso.git
git checkout -b prod
git add .
git commit -m "TIC - Automatic Backup Script"
git push -f -u origin prod
```

Aqui está a mágica: inicializa o repositório, configura usuário, adiciona remoto, cria branch e envia tudo para o GitLab.

***

## **Conclusão**

Esse script é um verdadeiro salva-vidas para quem precisa manter backups organizados e versionados. Automatizar esse processo reduz riscos, economiza tempo e garante que você tenha sempre uma cópia segura do seu site.

Adapte conforme seu ambiente e aproveite para integrar com **cron jobs** para backups automáticos!
