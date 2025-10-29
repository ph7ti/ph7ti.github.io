# Relatório de Usuários em um Grupo do AD DS com PowerShell

Você já precisou saber **quem está em determinado grupo do Active Directory**? Seja para auditoria, controle de acesso ou simples curiosidade, esse tipo de informação é essencial para manter a casa em ordem — ou melhor, o domínio!

Neste post, vamos mostrar como criar um relatório com os usuários ativos de um grupo específico do AD DS. E o melhor: com direito a CSV na área de trabalho e abertura automática no Notepad. Tudo isso com um script simples e direto ao ponto!

***

## Código 

```powershell
$groupname = Read-Host "Grupo:"
$DateStr = (Get-Date).ToString("dd_MM_yyyy")
$fileOutput = 'C:'+$env:HOMEPATH+"\Desktop\Relatório de usuários ativos no grupo "+$groupname+" do AD - "+$DateStr+".csv"
$result = @()
$users = Get-ADGroupMember -Identity $groupname | ? {$_.objectclass -eq "user"}
foreach ($activeusers in $users) { $result += (Get-ADUser -Identity $activeusers | select Name, SamAccountName, UserPrincipalName )}
Write "Total de usuários no grupo $groupname :" $result.Count 
$result | Sort-Object Name | Export-CSV -NoTypeInformation -Encoding UTF8 -path $fileOutput
[System.Reflection.Assembly]::LoadWithPartialName('System.Windows.Forms')
[System.Windows.Forms.MessageBox]::Show('Relatório foi criado com sucesso! Local do arquivo: '+$fileOutput,'PROCESSO CONCLUÍDO')
& 'C:\WINDOWS\system32\notepad.exe' $fileOutput
```

## O Script Explicado Passo a Passo

```powershell
$groupname = Read-Host "Grupo:"
```

**O que faz?**  
Solicita ao usuário que digite o nome do grupo do AD que deseja consultar. Isso torna o script interativo e reutilizável para diferentes grupos.

***

```powershell
$DateStr = (Get-Date).ToString("dd_MM_yyyy")
```

**O que faz?**  
Gera uma string com a data atual no formato `dd_MM_yyyy`, usada para nomear o arquivo de forma organizada e única.

***

```powershell
$fileOutput = 'C:'+$env:HOMEPATH+"\Desktop\Relatório de usuários ativos no grupo "+$groupname+" do AD - "+$DateStr+".csv"
```

**O que faz?**  
Cria o caminho completo para salvar o relatório na área de trabalho do usuário, incluindo o nome do grupo e a data.

***

```powershell
$result = @()
```

**O que faz?**  
Inicializa um array vazio que será preenchido com os dados dos usuários do grupo.

***

```powershell
$users = Get-ADGroupMember -Identity $groupname | ? {$_.objectclass -eq "user"}
```

**O que faz?**  
Busca os membros do grupo especificado e filtra apenas os objetos do tipo "user", ignorando computadores ou grupos aninhados.

***

```powershell
foreach ($activeusers in $users) {
    $result += (Get-ADUser -Identity $activeusers | select Name, SamAccountName, UserPrincipalName)
}
```

**O que faz?**  
Para cada usuário encontrado, coleta informações relevantes como nome, login (SamAccountName) e UPN (UserPrincipalName), adicionando ao array `$result`.

***

```powershell
Write "Total de usuários no grupo $groupname :" $result.Count 
```

**O que faz?**  
Exibe no console o total de usuários encontrados no grupo.

***

```powershell
$result | Sort-Object Name | Export-CSV -NoTypeInformation -Encoding UTF8 -path $fileOutput
```

**O que faz?**  
Ordena os usuários pelo nome e exporta os dados para um arquivo CSV codificado em UTF-8.

***

```powershell
[System.Reflection.Assembly]::LoadWithPartialName('System.Windows.Forms')
[System.Windows.Forms.MessageBox]::Show('Relatório foi criado com sucesso! Local do arquivo: '+$fileOutput,'PROCESSO CONCLUÍDO')
```

**O que faz?**  
Exibe uma mensagem pop-up informando que o relatório foi gerado com sucesso e mostra o caminho do arquivo.

***

```powershell
& 'C:\WINDOWS\system32\notepad.exe' $fileOutput
```

**O que faz?**  
Abre automaticamente o relatório no Bloco de Notas para visualização imediata.

***

## Conclusão

Esse script é uma ferramenta poderosa para quem precisa **auditar grupos do AD DS**, verificar acessos ou simplesmente manter a documentação em dia. Com poucos comandos, você gera um relatório limpo, direto e pronto para ser compartilhado com a equipe de segurança, compliance ou gestão.

Quer mais dicas de automação com PowerShell? Fique de olho no blog e compartilhe com seus colegas de TI!
