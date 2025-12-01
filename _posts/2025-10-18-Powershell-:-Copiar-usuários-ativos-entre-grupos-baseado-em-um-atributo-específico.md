# **PowerShell – Copiar Usuários Ativos Entre Grupos Baseado em um Atributo Específico**

![alt text](https://github.com/ph7ti/ph7ti.github.io/blob/main/_posts/imgs/Powershell-Copiar-usuários-atributo-específico.png?raw=true)

Quem nunca precisou sincronizar usuários entre grupos do AD, mas com *filtros mais inteligentes* do que apenas pegar todo mundo? 😅
No post de hoje, vamos falar sobre um script PowerShell que copia usuários ativos de um grupo para outro **baseando-se em um atributo personalizado** — no caso, o famoso `extensionAttribute1`.

Esse tipo de automação é perfeito para cenários de governança, auditoria ou delegação de acessos, especialmente em ambientes híbridos ou que dependem de grupos para controlar permissões.

---

## **🔧 Código Completo**

```powershell
$groupnamesource = "Domain Users"
$groupnamedestn = "Destination Group Users" 
$result = @()
$users = Get-ADGroupMember -Identity $groupnamesource | ? {$_.objectclass -eq "user"}
Write "Total de usuários no grupo $groupnamesource :" $users.Count
foreach ($activeusers in $users) { $result += (Get-ADUser -Identity $activeusers -Properties * | ? {$_.enabled -eq $true -and $_.extensionAttribute1 -ne $null} | select SamAccountName, Name, UserPrincipalName, Enabled ) }
Write "Total de usuários filtrados :" $result.Count
foreach ($aduser in $result) {
    Add-ADGroupMember -Identity $groupnamedestn -Members $aduser.SamAccountName
}
$useringroup = Get-ADGroupMember -Identity $groupnamedestn
Write "Total de usuários no grupo $groupnamedestn :" $useringroup.Count
```

---

# 🧩 Explicação Passo a Passo

## **1. Definindo os grupos de origem e destino**

```powershell
$groupnamesource = "Domain Users"
$groupnamedestn = "Destination Group Users" 
```

Aqui definimos de onde os usuários serão copiados e para qual grupo eles irão.
Simples, direto e essencial para toda a lógica do script.

---

## **2. Criando a estrutura para armazenar o resultado**

```powershell
$result = @()
```

Criamos um array para armazenar apenas os usuários que passam pelos filtros — ou seja, aqueles realmente aptos a serem copiados.

---

## **3. Coletando membros do grupo fonte**

```powershell
$users = Get-ADGroupMember -Identity $groupnamesource | ? {$_.objectclass -eq "user"}
Write "Total de usuários no grupo $groupnamesource :" $users.Count
```

Primeiro buscamos todos os membros do grupo.
Depois, filtramos para garantir que estamos lidando apenas com objetos do tipo **user** (já que grupos aninhados podem existir).
Por fim, exibimos a quantidade total.

---

## **4. Filtrando usuários ativos com `extensionAttribute1` preenchido**

```powershell
foreach ($activeusers in $users) { 
    $result += (
        Get-ADUser -Identity $activeusers -Properties * |
        ? { $_.enabled -eq $true -and $_.extensionAttribute1 -ne $null } |
        select SamAccountName, Name, UserPrincipalName, Enabled
    )
}
```

Essa etapa é a *cereja do bolo*.
Para cada usuário, buscamos suas propriedades e aplicamos dois filtros essenciais:

* **Usuário ativo (`enabled -eq $true`)**
* **Atributo `extensionAttribute1` preenchido**

Somente quem passar nessa triagem entra no array `$result`.

O script também seleciona apenas as propriedades úteis para manipular mais à frente.

---

## **5. Mostrando o total de usuários filtrados**

```powershell
Write "Total de usuários filtrados :" $result.Count
```

Aqui temos o número final de usuários elegíveis para serem copiados.

---

## **6. Adicionando usuários ao grupo de destino**

```powershell
foreach ($aduser in $result) {
    Add-ADGroupMember -Identity $groupnamedestn -Members $aduser.SamAccountName
}
```

Agora sim: cada usuário filtrado é incluído no grupo de destino.
O script usa `SamAccountName`, a forma mais comum e estável de referenciar um usuário.

---

## **7. Exibindo o total de usuários no grupo final**

```powershell
$useringroup = Get-ADGroupMember -Identity $groupnamedestn
Write "Total de usuários no grupo $groupnamedestn :" $useringroup.Count
```

Por fim, fazemos uma consulta final para mostrar quantos usuários estão no grupo de destino — incluindo os recém-adicionados.

---

# 🎯 **Conclusão**

Com poucas linhas de PowerShell, construímos um fluxo prático para:

* Ler usuários de um grupo;
* Filtrar somente os ativos e com um atributo personalizado preenchido;
* Replicar esses usuários automaticamente para outro grupo.

Esse tipo de script ajuda muito em rotinas de **gestão de acesso**, **padronização de grupos**, **controle de compliance** e **redução de tarefas manuais repetitivas**.

Sinta-se à vontade para adaptar os atributos, filtros e lógicas conforme a necessidade do seu ambiente. PowerShell está aí justamente para isso: **automatizar o que ninguém merece fazer manualmente.** 😄