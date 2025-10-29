# Active Directory: Criando um Relatório de Usuários do AD DS com PowerShell

Você já se perguntou quantos usuários estão no seu Active Directory? Ou quem nunca trocou a senha desde 2012? Pois é, o AD DS (Active Directory Domain Services) guarda muitos segredos — e com um simples script em PowerShell, você pode revelá-los todos em um relatório organizado e prontinho para análise!

Neste post, vamos explorar um script que coleta informações valiosas dos usuários do AD e gera um relatório em CSV direto na sua área de trabalho. Simples, útil e com um toque de automação que todo profissional de infraestrutura ama!

***

## Código 

```powershell
$DateStr = (Get-Date).ToString("dd_MM_yyyy")
fileOutput = 'C:'+$env:HOMEPATH+"\Desktop\Relatório de usuários do AD - "+$DateStr+".csv"
Get-ADUser -Filter * -SearchBase "DC=meudominio,DC=com,DC=br" -Properties * | Select-Object DisplayName, Description, Office, EmailAddress, Title, Department, Company, Passwordlastset, Passwordneverexpires, WhenChanged, AdminCount | Sort-Object DisplayName | export-csv -NoTypeInformation -Encoding UTF8 -path $fileOutput
[System.Reflection.Assembly]::LoadWithPartialName('System.Windows.Forms')
[System.Windows.Forms.MessageBox]::Show('Relatório foi criado com sucesso! Local do arquivo: '+$fileOutput,'PROCESSO CONCLUÍDO')
& 'C:\WINDOWS\system32\notepad.exe' $fileOutput
```

## O Script Explicado Passo a Passo

```powershell
$DateStr = (Get-Date).ToString("dd_MM_yyyy")
```

**O que faz?**  
Gera uma string com a data atual no formato `dd_MM_yyyy`. Isso será usado para nomear o arquivo de forma única e organizada.

***

```powershell
$fileOutput = 'C:'+$env:HOMEPATH+"\Desktop\Relatório de usuários do AD - "+$DateStr+".csv"
```

**O que faz?**  
Cria o caminho completo para salvar o arquivo CSV na área de trabalho do usuário, incluindo a data no nome para facilitar a identificação.

***

```powershell
Get-ADUser -Filter * -SearchBase "DC=meudominio,DC=com,DC=br" -Properties * |
Select-Object DisplayName, Description, Office, EmailAddress, Title, Department, Company, PasswordLastSet, PasswordNeverExpires, WhenChanged, AdminCount |
Sort-Object DisplayName |
Export-Csv -NoTypeInformation -Encoding UTF8 -Path $fileOutput
```

**O que faz?**

*   Usa o `Get-ADUser` para buscar todos os usuários do AD.
*   Filtra os campos mais relevantes para o relatório.
*   Ordena os usuários pelo nome de exibição.
*   Exporta tudo para um arquivo CSV codificado em UTF-8.

⚠️ **Importante:** Altere o `SearchBase` para refletir o seu domínio real!

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

Esse script é uma mão na roda para quem precisa auditar usuários do AD DS, verificar senhas, cargos, departamentos e muito mais. Com poucos comandos, você gera um relatório completo, organizado e pronto para ser analisado ou compartilhado com a equipe de segurança ou RH.

Quer mais dicas de automação com PowerShell? Fique ligado no blog e compartilhe com quem precisa dar um upgrade na gestão do AD!

***