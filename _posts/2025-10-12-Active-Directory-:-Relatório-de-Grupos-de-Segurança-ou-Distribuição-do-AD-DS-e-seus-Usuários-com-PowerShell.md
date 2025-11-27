# Relatório de Grupos de Segurança ou Distribuição do AD DS e seus Usuários com PowerShell

![alt text](https://github.com/ph7ti/ph7ti.github.io/blob/main/_posts/imgs/Relatorio-usuarios-grupos-AD-powershell.png?raw=true)

Você já se viu perdido tentando descobrir **quem está em qual grupo** no Active Directory? Seja para auditoria, controle de acesso ou só para matar a curiosidade, saber a composição dos grupos de segurança ou distribuição é essencial para manter a infraestrutura organizada.

Neste post, vamos mostrar como gerar um relatório completo com os grupos de um tipo específico (segurança ou distribuição) e listar seus respectivos usuários. Tudo isso com um script em PowerShell que salva o resultado em um arquivo `.log` direto na sua área de trabalho. Simples, direto e muito útil!

***

## 📜 Código Completo

```powershell
Write-Host "Escolha o tipo de grupo:"
Write-Host "1 - Distribution"
Write-Host "2 - Security"
$opcao = Read-Host "Digite o número da opção desejada (1 ou 2)"

switch ($opcao) {
    '1' { $grouptype = "Distribution" }
    '2' { $grouptype = "Security" }
    default {
        Write-Host "Opção inválida. Usando 'Security' como padrão."
        $grouptype = "Security"
    }
}

$DateStr = (Get-Date).ToString("dd_MM_yyyy")
$fileOutput = 'C:'+$env:HOMEPATH+"\Desktop\Relatório de Grupos de Segurança e Usuários do AD - '+$DateStr+'.log'"
$Groups = Get-ADGroup -Filter { (GroupCategory -eq $grouptype) } -SearchBase "OU=Industria Vital,DC=meudominio,DC=com,DC=br" -Properties * | Select-Object name 
Write-Output $Groups.GetType()
Start-Transcript -Append $fileOutput
foreach ($Grupo in $Groups) {
    Write-Output "#########################`nGroup:"
    Get-ADGroup -identity $Grupo[0].name | Select-Object name, distinguishedName, GroupCategory, GroupScope
    $temp = Write-Output $Grupo[0].name
    Write-Output "Users:"
    Get-ADGroupMember -Identity "$temp" -Recursive | Select-Object name, SamAccountName, distinguishedName
}
Stop-Transcript
[System.Reflection.Assembly]::LoadWithPartialName('System.Windows.Forms')
[System.Windows.Forms.MessageBox]::Show('Relatório foi criado com sucesso! Local do arquivo: '+$fileOutput,'PROCESSO CONCLUÍDO')
& 'C:\WINDOWS\system32\notepad.exe' $fileOutput
```

***

## Explicando o Script Passo a Passo

### 1. Escolha do tipo de grupo

```powershell

Write-Host "Escolha o tipo de grupo:"
Write-Host "1 - Distribution"
Write-Host "2 - Security"
$opcao = Read-Host "Digite o número da opção desejada (1 ou 2)"

switch ($opcao) {
    '1' { $grouptype = "Distribution" }
    '2' { $grouptype = "Security" }
    default {
        Write-Host "Opção inválida. Usando 'Security' como padrão."
        $grouptype = "Security"
    }
}
```

O script começa com um menu interativo que permite ao usuário escolher entre grupos de Distribuição (1) ou Segurança (2). Se a entrada for inválida, ele assume "Security" como padrão.

***

### 2. Geração da data atual

```powershell
$DateStr = (Get-Date).ToString("dd_MM_yyyy")
```

Cria uma string com a data atual para ser usada no nome do arquivo de saída.

***

### 3. Definição do caminho do arquivo

```powershell
$fileOutput = 'C:'+$env:HOMEPATH+"\Desktop\Relatório de Grupos de Segurança e Usuários do AD - '+$DateStr+'.log'"
```

Define o caminho completo do arquivo `.log` que será salvo na área de trabalho do usuário.

***

### 4. Coleta dos grupos

```powershell
$Groups = Get-ADGroup -Filter { (GroupCategory -eq $grouptype) } -SearchBase "OU=Industria Vital,DC=meudominio,DC=com,DC=br" -Properties * | Select-Object name 
```

Busca todos os grupos do tipo especificado dentro da OU "Industria Vital" e seleciona apenas o nome de cada grupo.

***

### 5. Exibição do tipo de objeto retornado

```powershell
Write-Output $Groups.GetType()
```

Mostra no console o tipo de objeto retornado pela consulta (geralmente uma coleção).

***

### 6. Início da gravação do relatório

```powershell
Start-Transcript -Append $fileOutput
```

Inicia a gravação de todas as saídas do console no arquivo `.log`.

***

### 7. Iteração pelos grupos e coleta de usuários

```powershell
foreach ($Grupo in $Groups) {
    Write-Output "#########################`nGroup:"
    Get-ADGroup -identity $Grupo[0].name | Select-Object name, distinguishedName, GroupCategory, GroupScope
    $temp = Write-Output $Grupo[0].name
    Write-Output "Users:"
    Get-ADGroupMember -Identity "$temp" -Recursive | Select-Object name, SamAccountName, distinguishedName
}
```

Para cada grupo:

*   Exibe informações básicas do grupo.
*   Coleta e exibe os usuários pertencentes ao grupo, incluindo nome, login e DN.

***

### 8. Finalização do relatório

```powershell
Stop-Transcript
```

Encerra a gravação do arquivo `.log`.

***

### 9. Mensagem de sucesso

```powershell
[System.Reflection.Assembly]::LoadWithPartialName('System.Windows.Forms')
[System.Windows.Forms.MessageBox]::Show('Relatório foi criado com sucesso! Local do arquivo: '+$fileOutput,'PROCESSO CONCLUÍDO')
```

Exibe uma mensagem pop-up informando que o relatório foi criado com sucesso.

***

### 10. Abertura automática do relatório

```powershell
& 'C:\WINDOWS\system32\notepad.exe' $fileOutput
```

Abre o arquivo `.log` no Bloco de Notas para visualização imediata.

***

## Conclusão

Esse script é uma ferramenta poderosa para quem precisa **auditar grupos do AD DS**, seja para verificar membros de grupos de segurança ou distribuição. Ele automatiza a coleta de dados, organiza tudo em um relatório legível e ainda abre o resultado para você na hora. Ideal para administradores que querem agilidade e clareza na gestão de acessos!

Se quiser, posso corrigir o pequeno detalhe no código e gerar um arquivo `.ps1` ou `.md` para você. Deseja que eu faça isso?
