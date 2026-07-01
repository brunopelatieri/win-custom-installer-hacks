# 🧹 Cursor IDE — Limpeza de Cache e Manutenção (Windows 11)

> **Guia completo para diagnosticar, limpar com segurança e automatizar a manutenção da pasta `AppData\Roaming\Cursor`** — incluindo o mapa de pastas, scripts PowerShell prontos para uso e configuração do Agendador de Tarefas.

[![Cursor](https://img.shields.io/badge/-Cursor%20IDE-000000?style=flat-square&logo=cursor&logoColor=white)](https://www.cursor.sh/)
[![Windows](https://img.shields.io/badge/-Windows%2011-0078D4?style=flat-square&logo=windows&logoColor=white)](.)
[![PowerShell](https://img.shields.io/badge/-PowerShell-5391FE?style=flat-square&logo=powershell&logoColor=white)](.)
[![Nível](https://img.shields.io/badge/Nível-Intermediário%20a%20Avançado-orange?style=flat-square)](.)

---

## 📋 Índice

- [🧹 Cursor IDE — Limpeza de Cache e Manutenção (Windows 11)](#-cursor-ide--limpeza-de-cache-e-manutenção-windows-11)
  - [📋 Índice](#-índice)
  - [🤔 Por que a pasta cresce sem parar](#-por-que-a-pasta-cresce-sem-parar)
  - [1. Diagnóstico — O que está pesando](#1-diagnóstico--o-que-está-pesando)
    - [Opção A — PowerShell (sem instalar nada)](#opção-a--powershell-sem-instalar-nada)
    - [Opção B — WizTree (ferramenta gráfica)](#opção-b--wiztree-ferramenta-gráfica)
  - [2. Mapa completo — O que pode e o que não pode apagar](#2-mapa-completo--o-que-pode-e-o-que-não-pode-apagar)
  - [3. Atenção especial: workspaceStorage e o histórico de IA](#3-atenção-especial-workspacestorage-e-o-histórico-de-ia)
    - [Script para identificar o que é cada pasta antes de apagar](#script-para-identificar-o-que-é-cada-pasta-antes-de-apagar)
  - [4. Roteiro de limpeza manual](#4-roteiro-de-limpeza-manual)
  - [5. Ajustes dentro do Cursor (Settings)](#5-ajustes-dentro-do-cursor-settings)
  - [6. Script de limpeza automática](#6-script-de-limpeza-automática)
  - [7. Agendador de Tarefas — automatizar o script](#7-agendador-de-tarefas--automatizar-o-script)
  - [8. Manutenção contínua — workspaceStorage mensal](#8-manutenção-contínua--workspacestorage-mensal)
  - [✅ Checklist de Manutenção](#-checklist-de-manutenção)
  - [🐛 Troubleshooting](#-troubleshooting)

---

## 🤔 Por que a pasta cresce sem parar

O Cursor é um fork do VS Code construído em Electron — o mesmo motor que roda o Chrome. Isso traz uma consequência direta: **o app acumula dados como um navegador**, mas sem o botão "Limpar dados de navegação".

O que empilha silenciosamente em `AppData\Roaming\Cursor`:

- Caches do motor Chromium/V8 (reconstroem a cada sessão)
- Logs de cada sessão aberta (uma subpasta por abertura do app)
- Relatórios de crash
- Histórico local de versões de cada arquivo editado (feature "Local History")
- **Um banco SQLite por projeto** (`state.vscdb`) guardando o histórico completo de chat e Composer com a IA, índice do codebase, estado de abas e mais
- Backups de arquivos não salvos no fechamento

Como nenhum desses é limpo automaticamente por padrão, a pasta só cresce com o tempo de uso.

---

## 1. Diagnóstico — O que está pesando

### Opção A — PowerShell (sem instalar nada)

```powershell
Get-ChildItem "$env:APPDATA\Cursor" -Directory | ForEach-Object {
    $size = (Get-ChildItem $_.FullName -Recurse -File -ErrorAction SilentlyContinue |
             Measure-Object -Property Length -Sum).Sum
    [PSCustomObject]@{ Pasta = $_.Name; TamanhoGB = [math]::Round($size/1GB,2) }
} | Sort-Object TamanhoGB -Descending | Format-Table -AutoSize
```

Retorna um ranking das subpastas por tamanho — pronto para colar no PowerShell e rodar.

### Opção B — WizTree (ferramenta gráfica)

Baixe o **WizTree** (gratuito), aponte para `%APPDATA%\Cursor` e em segundos você vê o mapa visual de espaço por pasta, usando a MFT do NTFS para varredura quase instantânea.

---

## 2. Mapa completo — O que pode e o que não pode apagar

| Pasta | O que guarda | Pode apagar? |
|---|---|---|
| `Cache` | Cache do motor Chromium | ✅ Sim — reconstrói automaticamente |
| `CachedData` | Cache de dados do Electron | ✅ Sim — reconstrói automaticamente |
| `Code Cache` | Cache de bytecode V8/JavaScript | ✅ Sim — reconstrói automaticamente |
| `GPUCache` | Cache de shaders de GPU | ✅ Sim — reconstrói automaticamente |
| `Crashpad\reports` | Relatórios de crash do app | ✅ Sim — só úteis para reportar bug à equipe do Cursor |
| `logs\` | Logs de sessão (uma subpasta por abertura) | ✅ Sim, exceto o mais recente |
| `Service Worker` | Estrutura interna do Electron | ✅ Sim — **feche o Cursor antes** |
| `blob_storage` | Estrutura interna do Electron | ✅ Sim — **feche o Cursor antes** |
| `IndexedDB` | Banco interno do Electron | ✅ Sim — **feche o Cursor antes** |
| `Local Storage` | Estado de interface do Electron | ✅ Sim — **feche o Cursor antes** |
| `Session Storage` | Sessão temporária do Electron | ✅ Sim — **feche o Cursor antes** |
| `Backups\` | Cópias de arquivos não salvos no fechamento | ⚠️ Cuidado — só apague se não houver edição pendente |
| `User\History\` | Histórico local ("Timeline") de versões de arquivos | ✅ Sim — você perde versões antigas, **não** o arquivo atual |
| `User\workspaceStorage\` | Banco SQLite com histórico de chat/Composer da IA **por projeto** | ⚠️ **Cuidado redobrado — ver seção 3** |
| `User\globalStorage\state.vscdb` | Banco principal: conta, extensões, estado global | ❌ Não apague o arquivo ativo |
| `User\globalStorage\state.vscdb.backup` | Backup automático do banco principal | ✅ Pode remover se estiver grande — é recriado |

---

## 3. Atenção especial: workspaceStorage e o histórico de IA

> ⚠️ **Leia esta seção antes de apagar qualquer coisa em `workspaceStorage`.**

Cada pasta dentro de `User\workspaceStorage\` corresponde a **um projeto que você já abriu no Cursor**. Dentro dela tem um `workspace.json` dizendo qual é o caminho do projeto, e um `state.vscdb` com o histórico de chat/Composer daquele projeto.

**Apagar essa pasta inteira = perder o histórico de IA de todos os projetos abertos no Cursor.**

### Script para identificar o que é cada pasta antes de apagar

```powershell
Get-ChildItem "$env:APPDATA\Cursor\User\workspaceStorage" -Directory | ForEach-Object {
    $wsJson = Join-Path $_.FullName "workspace.json"
    $projeto = if (Test-Path $wsJson) {
        (Get-Content $wsJson -Raw | ConvertFrom-Json).folder
    } else { "(sem workspace.json — provavelmente órfã)" }
    $size = (Get-ChildItem $_.FullName -Recurse -File -ErrorAction SilentlyContinue |
             Measure-Object Length -Sum).Sum
    [PSCustomObject]@{
        Pasta      = $_.Name
        Projeto    = $projeto
        TamanhoMB  = [math]::Round($size/1MB,1)
    }
} | Sort-Object TamanhoMB -Descending | Format-Table -AutoSize -Wrap
```

Com a lista em mãos, apague **somente** as pastas (hash) que correspondem a:

- Projetos que você não usa mais (clientes antigos, testes, repositórios abandonados)
- Projetos cujo histórico de chat você não precisa preservar

---

## 4. Roteiro de limpeza manual

1. **Feche o Cursor completamente** — confirme no Gerenciador de Tarefas que nenhum processo `Cursor.exe` está ativo
2. Abra `Win + R` → digite `%appdata%\Cursor` → Enter
3. Apague com segurança (risco zero): `Cache`, `CachedData`, `Code Cache`, `GPUCache`, `Crashpad\reports`, `Service Worker`, `blob_storage` e os logs antigos dentro de `logs\`
4. Revise `Backups\` — se vazia ou só lixo antigo, pode apagar
5. Use o script da seção 3 para revisar `workspaceStorage\` e apagar só hashes de projetos mortos
6. Reabra o Cursor — o primeiro boot pode ser um pouco mais lento enquanto os caches são reconstruídos. Isso é esperado e normal.

---

## 5. Ajustes dentro do Cursor (Settings)

Abra `Ctrl+Shift+P` → `Preferences: Open User Settings (JSON)` e ajuste:

```json
{
  "workbench.localHistory.enabled": true,
  "workbench.localHistory.maxFileEntries": 10,
  "files.hotExit": "onExitAndWindowClose"
}
```

| Configuração | Valor | Efeito |
|---|---|---|
| `workbench.localHistory.maxFileEntries` | `10` | Reduz de 50 (padrão) para 10 o número de versões antigas guardadas por arquivo em `User\History` |
| `workbench.localHistory.enabled` | `false` | Desliga o histórico local completamente — mais espaço economizado |
| `files.hotExit` | `"off"` | Para de acumular na pasta `Backups\` — você perde a recuperação de arquivos não salvos em caso de crash |

> 💡 Abra `Ctrl+,` e busque por "history" e "cache" — o Cursor recebe atualizações frequentes e pode ter novas opções de indexação de codebase ou IA desde a última vez que você verificou.

---

## 6. Script de limpeza automática

Salve como `C:\Scripts\limpa-cursor.ps1`. Limpa apenas o que é **100% seguro** — deliberadamente não toca em `workspaceStorage` nem `Backups`, que exigem revisão manual.

```powershell
# limpa-cursor.ps1
# Execute apenas com o Cursor FECHADO.
# Não toca em workspaceStorage nem Backups — esses exigem revisão manual.

$cursorRunning = Get-Process -Name "Cursor" -ErrorAction SilentlyContinue
if ($cursorRunning) {
    Write-Host "Cursor está aberto. Feche antes de rodar a limpeza." -ForegroundColor Yellow
    exit
}

$base = "$env:APPDATA\Cursor"

$pastasSeguras = @(
    "Cache",
    "CachedData",
    "Code Cache",
    "GPUCache",
    "Service Worker",
    "blob_storage",
    "IndexedDB",
    "Local Storage",
    "Session Storage",
    "Crashpad\reports"
)

foreach ($p in $pastasSeguras) {
    $caminho = Join-Path $base $p
    if (Test-Path $caminho) {
        Remove-Item $caminho -Recurse -Force -ErrorAction SilentlyContinue
        Write-Host "Limpo: $p"
    }
}

# Remove logs com mais de 14 dias, preservando os recentes
$logsPath = Join-Path $base "logs"
if (Test-Path $logsPath) {
    $removidos = 0
    Get-ChildItem $logsPath -Directory |
        Where-Object { $_.LastWriteTime -lt (Get-Date).AddDays(-14) } |
        ForEach-Object {
            Remove-Item $_.FullName -Recurse -Force
            $removidos++
        }
    Write-Host "Logs antigos removidos: $removidos pasta(s)"
}

Write-Host ""
Write-Host "Limpeza concluida." -ForegroundColor Green
Write-Host "Revise manualmente: workspaceStorage e Backups\" -ForegroundColor Cyan
```

---

## 7. Agendador de Tarefas — automatizar o script

1. Abra o **Agendador de Tarefas**: `Win + R` → `taskschd.msc`
2. Clique em **Criar Tarefa Básica** no painel direito
3. Configure:

| Campo | Valor |
|---|---|
| Nome | `Limpeza Cache Cursor` |
| Gatilho | Semanal — dia e hora em que o Cursor normalmente está fechado |
| Ação | Iniciar um programa |
| Programa | `powershell.exe` |
| Argumentos | `-ExecutionPolicy Bypass -File "C:\Scripts\limpa-cursor.ps1"` |

4. Na aba **Condições**, marque:
   - ✅ Iniciar a tarefa somente se o computador estiver ocioso há `10 minutos`
   - ✅ Parar se o computador deixar de estar ocioso

> 💡 Como o script já verifica se o Cursor está aberto e sai sem fazer nada em caso positivo, não há risco de conflito mesmo que a janela de ociosidade não seja respeitada.

---

## 8. Manutenção contínua — workspaceStorage mensal

O script automático não toca em `workspaceStorage` por design — apagar histórico de IA por engano é irreversível. Crie um lembrete mensal para rodar o script de diagnóstico da seção 3 e revisar manualmente quais projetos ainda importam.

A combinação mais eficiente para uso intenso do Cursor:

```
Script automático semanal    → limpa caches/logs sem risco
Revisão manual mensal        → decide o que apagar em workspaceStorage
Junction para drive E:       → resolve o problema de espaço na raiz
```

---

## ✅ Checklist de Manutenção

| Ação | Impacto | Risco | Frequência |
|---|---|---|---|
| Limpar caches Chromium/Electron | Médio-alto | Nenhum | Automático via script |
| Limpar `Crashpad\reports` e `logs` antigos | Baixo-médio | Nenhum | Automático via script |
| Revisar `workspaceStorage` por projeto | **Alto** | Perda de histórico de IA se não revisar | Mensal, manual |
| Reduzir `workbench.localHistory.maxFileEntries` | Médio (cumulativo) | Nenhum | Configurar uma vez |
| Junction para outro drive (`SYMLINK.md`) | Resolve na raiz | Baixo (com backup) | Uma vez |

---

## 🐛 Troubleshooting

| Problema | Causa provável | Solução |
|---|---|---|
| Script retorna erro de permissão | Rodado sem elevação | Rode o PowerShell como Administrador |
| Script não detecta o Cursor aberto | Nome de processo diferente na versão instalada | Substitua `"Cursor"` por `"cursor"` (minúsculo) no `Get-Process` e teste |
| Agendador executa mas não apaga nada | Cursor estava aberto quando a tarefa disparou | Normal — configure a condição de ociosidade na aba Condições |
| Histórico de IA sumiu após limpeza | `workspaceStorage` apagado por engano | Não há recuperação — use o script de diagnóstico (seção 3) **antes** de apagar |
| Pasta cresce de volta rapidamente | `workspaceStorage` crescendo com uso intenso de Composer | Comportamento esperado — revise mensalmente via script de diagnóstico |