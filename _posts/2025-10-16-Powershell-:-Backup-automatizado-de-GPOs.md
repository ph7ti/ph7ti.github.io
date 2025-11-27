# **Backup Automátizado de GPOs com PowerShell e Task Scheduler**

## **Introdução**

Manter um histórico das suas **GPOs (Group Policy Objects)** é essencial para garantir segurança e controle no ambiente corporativo. Já pensou perder uma configuração crítica por falta de backup? 😱  
Com este script em **PowerShell**, você poderá automatizar o backup das GPOs, garantindo que cada alteração gere uma nova versão sem apagar as anteriores. E o melhor: tudo isso pode ser agendado pelo **Task Scheduler** para rodar sem intervenção manual.

***

## **Script Completo**

```powershell
$AllGPOs = Get-GPO -All
$folder = "c:\GPO_Backup"

if (-Not (Test-Path -Path $folder)) {
    New-Item -ItemType Directory -Path $folder | Out-Null
}

foreach ($GPO in $AllGPOs) {
  $pattern = '[^a-zA-Z0-9\s]'
  $DisplayName = $GPO.DisplayName -replace $pattern,''
  $ModificationTime = $GPO.ModificationTime -replace '[/:]','_'
  $ModificationTime = $ModificationTime -replace ' ','_'
  $BackupDestination = $folder + '\' + $DisplayName + '\' + $ModificationTime + '\'
  if (-Not (Test-Path $BackupDestination)) {
      New-Item -Path $BackupDestination -ItemType directory | Out-Null
      Backup-GPO -Name $GPO.DisplayName -Path $BackupDestination
  }
}
```

***

## **Explicação Passo a Passo**

### **1. Coleta de todas as GPOs**

```powershell
$AllGPOs = Get-GPO -All
```

Este comando obtém todas as GPOs do domínio. É a base para o loop que fará o backup individual.

***

### **2. Definição da pasta de backup**

```powershell
$folder = "c:\GPO_Backup"
```

Aqui você define onde os backups serão armazenados. Pode alterar conforme sua necessidade.

***

### **3. Verificação e criação da pasta**

```powershell
if (-Not (Test-Path -Path $folder)) {
    New-Item -ItemType Directory -Path $folder | Out-Null
}
```

Garante que a pasta existe. Caso contrário, cria automaticamente.

***

### **4. Loop para backup das GPOs**

```powershell
foreach ($GPO in $AllGPOs) {
  $pattern = '[^a-zA-Z0-9\s]'
  $DisplayName = $GPO.DisplayName -replace $pattern,''
  $ModificationTime = $GPO.ModificationTime -replace '[/:]','_'
  $ModificationTime = $ModificationTime -replace ' ','_'
  $BackupDestination = $folder + '\' + $DisplayName + '\' + $ModificationTime + '\'
  if (-Not (Test-Path $BackupDestination)) {
      New-Item -Path $BackupDestination -ItemType directory | Out-Null
      Backup-GPO -Name $GPO.DisplayName -Path $BackupDestination
  }
}
```

*   **Sanitização do nome da GPO**: Remove caracteres especiais para evitar erros no caminho.
*   **Criação de subpastas por data de modificação**: Cada alteração gera uma nova versão.
*   **Backup-GPO**: Comando nativo do PowerShell para efetuar o backup.

***

## **Agendamento com Task Scheduler**

Para automatizar:

1.  Abra o **Agendador de Tarefas**.
2.  Crie uma nova tarefa.
3.  Configure para executar o PowerShell com o script:
    ```powershell
    powershell.exe -File "C:\Scripts\BackupGPO.ps1"
    ```
4.  Defina a periodicidade (diária, semanal, etc.).

***

## **Conclusão**

Com este script, você garante um histórico completo das suas GPOs, evitando surpresas desagradáveis e facilitando auditorias. É simples, eficiente e totalmente automatizável.  
Adapte conforme seu ambiente e mantenha sua infraestrutura segura e organizada!
