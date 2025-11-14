# Automatizando a Inclusão de Usuários em Grupos de Distribuição no Exchange 365

![alt text](https://github.com/ph7ti/ph7ti.github.io/blob/main/_posts/imgs/Powershell-Grupos-Distribuicao-365.png?raw=true)

Você já se viu copiando e colando usuários manualmente em grupos de distribuição? 😩  
Se a resposta for sim, este post é pra você!  
Vamos mostrar como automatizar esse processo com um script simples em PowerShell, ideal para quem administra ambientes com Exchange Online e precisa ganhar tempo (e sanidade 😅).

## 💻 Código Completo

```powershell
$toGroup = (Read-Host "Digite o Nome do Grupo")
$UserPath = "C:\Automation\Add Bulk Users to Distribution Group"
$users_file = "$UserPath\users.csv"
$cred = "$UserPath\Cred.xml"
$cred = Import-CliXml -Path $cred
Import-Module ExchangeOnlineManagement
Connect-ExchangeOnline -Credential $cred -ShowBanner:$false
Import-CSV $users_file | foreach {  
 $UPN=$_.UPN 
 Write-Progress -Activity "Adding $UPN to group $toGroup " 
 Add-DistributionGroupMember –Identity $toGroup -Member $UPN  
 If($?)  
 {  
 Write-Host $UPN Successfully added -ForegroundColor Green 
 }  
 Else  
 {  
 Write-Host $UPN - Error occurred –ForegroundColor Red  
 } 
}
Disconnect-ExchangeOnline -Confirm:$false -InformationAction Ignore -ErrorAction SilentlyContinue
```

***

## 🧩 Explicação Passo a Passo

### 1. Recebendo o nome do grupo

```powershell
$toGroup = (Read-Host "Digite o Nome do Grupo")
```

Aqui o script solicita ao usuário o nome do grupo de distribuição que será atualizado. Isso torna o script reutilizável para diferentes grupos.

***

### 2. Definindo o caminho dos arquivos

```powershell
$UserPath = "C:\Automation\Add Bulk Users to Distribution Group"
$users_file = "$UserPath\users.csv"
$cred = "$UserPath\Cred.xml"
```

Define o caminho onde estão armazenados os arquivos necessários: o CSV com os usuários e o XML com as credenciais.

***

### 3. Importando credenciais

```powershell
$cred = Import-CliXml -Path $cred
```

Importa as credenciais salvas previamente em um arquivo XML. Isso evita digitação manual e facilita a automação.

***

### 4. Carregando o módulo do Exchange Online

```powershell
Import-Module ExchangeOnlineManagement
```

Carrega o módulo necessário para interagir com o Exchange Online via PowerShell. Ele precisa estar previamente instalado, não se esqueça disso!

***

### 5. Conectando ao Exchange Online

```powershell
Connect-ExchangeOnline -Credential $cred -ShowBanner:$false
```

Estabelece a conexão com o Exchange Online usando as credenciais importadas.

***

### 6. Importando usuários e adicionando ao grupo

```powershell
Import-CSV $users_file | foreach {  
 $UPN=$_.UPN 
 Write-Progress -Activity "Adding $UPN to group $toGroup " 
 Add-DistributionGroupMember –Identity $toGroup -Member $UPN  
 If($?)  
 {  
 Write-Host $UPN Successfully added -ForegroundColor Green 
 }  
 Else  
 {  
 Write-Host $UPN - Error occurred –ForegroundColor Red  
 } 
}
```

Lê o arquivo CSV com os usuários (espera-se que tenha uma coluna chamada `UPN`) e adiciona cada um ao grupo especificado.  
O uso de `Write-Progress` e `Write-Host` ajuda a acompanhar o andamento e identificar erros.

***

### 7. Finalizando a conexão

```powershell
Disconnect-ExchangeOnline -Confirm:$false -InformationAction Ignore -ErrorAction SilentlyContinue
```

Desconecta do Exchange Online de forma silenciosa, garantindo que a sessão seja encerrada corretamente.

***

## ✅ Conclusão

Esse script é uma mão na roda para quem precisa gerenciar grupos de distribuição com agilidade e segurança.  
Ideal para cenários de onboarding em massa, mudanças organizacionais ou simplesmente para evitar o trabalho manual repetitivo.

Adapte os caminhos e formatos conforme seu ambiente e aproveite o poder da automação no Exchange Online! 🚀
