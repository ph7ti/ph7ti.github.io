# Powershell: Automatizando a inclusão de usuários em grupos de distribuição no Exchange 365

![alt text](https://github.com/ph7ti/ph7ti.github.io/blob/main/_posts/imgs/Powershell-Grupos-Distribuicao-365.png?raw=true)


Você já se pegou adicionando manualmente usuários a grupos de distribuição no Exchange Online, um por um? Se sim, sabe o quanto isso pode ser repetitivo e propenso a erros. Que tal automatizar esse processo com um script simples em PowerShell?

Neste post, vou te mostrar como criar um script que adiciona um colaborador a um grupo de distribuição no Exchange 365 de forma rápida e segura. Ideal para quem trabalha com infraestrutura e precisa otimizar tarefas do dia a dia!

***

## 💻 Script completo

```powershell
param ($Email, $Grupo)
$UserPath = "C:\Automation\Add Bulk Users to Distribution Group"
$cred = "$UserPath\Cred.xml"
$cred = Import-CliXml -Path $cred
Import-Module ExchangeOnlineManagement
Connect-ExchangeOnline -Credential $cred -ShowBanner:$false
Add-DistributionGroupMember –Identity $Grupo -Member $Email  
If($?)  
{  
Write-Host $Email Successfully added -ForegroundColor Green 
}  
Else  
{  
Write-Host $Email - Error occurred –ForegroundColor Red  
}
Disconnect-ExchangeOnline -Confirm:$false -InformationAction Ignore -ErrorAction SilentlyContinue
```

***

## 🧩 Explicação passo a passo

### 1. Parâmetros de entrada

```powershell
param ($Email, $Grupo)
```

Aqui definimos dois parâmetros que serão passados ao script: o e-mail do colaborador a ser adicionado e o nome do grupo de distribuição.

***

### 2. Caminho para credenciais

```powershell
$UserPath = "C:\Automation\Add Bulk Users to Distribution Group"
$cred = "$UserPath\Cred.xml"
```

Define o caminho onde está armazenado o arquivo XML com as credenciais de acesso ao Exchange Online. Isso evita digitar a senha toda vez que o script for executado.

***

### 3. Importando as credenciais

```powershell
$cred = Import-CliXml -Path $cred
```

Importa as credenciais salvas no arquivo XML para uso na autenticação.

***

### 4. Carregando o módulo do Exchange Online

```powershell
Import-Module ExchangeOnlineManagement
```

Carrega o módulo necessário para executar comandos no Exchange Online via PowerShell.

***

### 5. Conectando ao Exchange Online

```powershell
Connect-ExchangeOnline -Credential $cred -ShowBanner:$false
```

Estabelece a conexão com o Exchange Online usando as credenciais importadas. O parâmetro `-ShowBanner:$false` evita que o banner de boas-vindas seja exibido.

***

### 6. Adicionando o usuário ao grupo

```powershell
Add-DistributionGroupMember –Identity $Grupo -Member $Email  
```

Este é o comando principal: adiciona o e-mail informado ao grupo de distribuição especificado.

***

### 7. Verificando sucesso ou erro

```powershell
If($?)  
{  
Write-Host $Email Successfully added -ForegroundColor Green 
}  
Else  
{  
Write-Host $Email - Error occurred –ForegroundColor Red  
}
```

Verifica se o comando anterior foi executado com sucesso. Se sim, exibe uma mensagem verde; se não, uma mensagem vermelha.

***

### 8. Desconectando do Exchange Online

```powershell
Disconnect-ExchangeOnline -Confirm:$false -InformationAction Ignore -ErrorAction SilentlyContinue
```

Finaliza a sessão com o Exchange Online de forma silenciosa, sem pedir confirmação ou exibir mensagens.

***

## 🚀 Conclusão

Esse script é uma mão na roda para quem precisa gerenciar grupos de distribuição no Exchange 365 com agilidade e segurança. Automatizar tarefas como essa economiza tempo, evita erros manuais e garante mais eficiência na administração do ambiente.

Você pode adaptá-lo para incluir múltiplos usuários, ler de um CSV ou até integrá-lo em rotinas maiores de provisionamento. Teste, ajuste conforme sua realidade e compartilhe com a equipe!
