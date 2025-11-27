# **PowerShell: Verificar se a conta do usuário está expirada no AD DS**

![alt text](https://github.com/ph7ti/ph7ti.github.io/blob/main/_posts/imgs/Powershell-Verificar-conta-expirada.png?raw=true)

## **Introdução**

Você já se deparou com aquele momento crítico em que um usuário não consegue acessar o sistema porque a senha expirou? Em ambientes corporativos, isso pode gerar atrasos e frustrações. Mas e se você pudesse identificar rapidamente quando a conta está prestes a expirar ou já expirou?  
Neste artigo, vamos apresentar um script simples em **PowerShell** que faz exatamente isso: verifica se a conta do usuário está ativa e mostra a data de expiração da senha. Preparado para facilitar sua vida como administrador?

***

## **Script Completo**

```powershell
$user = Read-Host "Digite o CN (Common-Name) do usuário"
Get-ADUser -filter {cn -eq $user -and Enabled -eq $True -and PasswordNeverExpires -eq $False} –Properties "DisplayName", "msDS-UserPasswordExpiryTimeComputed" |
Select-Object -Property "Displayname",@{Name="ExpiryDate";Expression={[datetime]::FromFileTime($_."msDS-UserPasswordExpiryTimeComputed")}}
```

***

## **Explicação Passo a Passo**

### **1. Capturando o nome do usuário**

```powershell
$user = Read-Host "Digite o CN (Common-Name) do usuário"
```

Aqui, usamos `Read-Host` para solicitar ao administrador o **CN** (Common Name) do usuário. Esse valor será usado como filtro na consulta ao Active Directory.

***

### **2. Consultando o Active Directory**

```powershell
Get-ADUser -filter {cn -eq $user -and Enabled -eq $True -and PasswordNeverExpires -eq $False} –Properties "DisplayName", "msDS-UserPasswordExpiryTimeComputed"
```

*   **Get-ADUser**: Comando para buscar informações do usuário no AD.
*   **Filtro**:
    *   `cn -eq $user`: Garante que estamos consultando o usuário correto.
    *   `Enabled -eq $True`: Apenas contas ativas.
    *   `PasswordNeverExpires -eq $False`: Ignora contas com senha que nunca expira.
*   **Propriedades adicionais**:
    *   `DisplayName`: Nome amigável do usuário.
    *   `msDS-UserPasswordExpiryTimeComputed`: Data de expiração da senha.

***

### **3. Formatando a saída**

```powershell
Select-Object -Property "Displayname",@{Name="ExpiryDate";Expression={[datetime]::FromFileTime($_."msDS-UserPasswordExpiryTimeComputed")}}
```

*   **Select-Object**: Escolhe quais propriedades serão exibidas.
*   **ExpiryDate**: Converte o valor retornado pelo AD (em formato FileTime) para uma data legível usando `[datetime]::FromFileTime()`.

***

## **Conclusão**

Com esse script, você pode rapidamente verificar se a senha de um usuário está expirada ou quando ela vai expirar. Isso ajuda a antecipar problemas e manter a produtividade da equipe.  
Adapte o script conforme sua necessidade e inclua em suas rotinas de administração para ganhar agilidade no suporte.