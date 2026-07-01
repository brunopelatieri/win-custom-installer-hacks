# 🔗 Cursor IDE — Movendo Dados para o Drive E: via Junction

> **Guia para mover `AppData\Roaming\Cursor` — histórico de IA, cache, configurações e banco de dados por projeto — para fora do drive `C:\`** usando um junction (link de pasta) transparente para o aplicativo.

[![Cursor](https://img.shields.io/badge/-Cursor%20IDE-000000?style=flat-square&logo=cursor&logoColor=white)](https://www.cursor.sh/)
[![Windows](https://img.shields.io/badge/-Windows%2011-0078D4?style=flat-square&logo=windows&logoColor=white)](.)
[![CMD](https://img.shields.io/badge/-CMD%20Admin-4D4D4D?style=flat-square&logo=windows-terminal&logoColor=white)](.)
[![Nível](https://img.shields.io/badge/Nível-Avançado-red?style=flat-square)](.)

---

## 📋 Índice

- [🔗 Cursor IDE — Movendo Dados para o Drive E: via Junction](#-cursor-ide--movendo-dados-para-o-drive-e-via-junction)
  - [📋 Índice](#-índice)
  - [🎯 Objetivo](#-objetivo)
  - [🔗 O que é um Junction e por que não usar Symlink aqui](#-o-que-é-um-junction-e-por-que-não-usar-symlink-aqui)
  - [📦 O que fica em AppData\\Roaming\\Cursor](#-o-que-fica-em-appdataroamingcursor)
  - [🗺️ Visão Geral Antes e Depois](#️-visão-geral-antes-e-depois)
  - [1. Diagnóstico — O que está pesando](#1-diagnóstico--o-que-está-pesando)
  - [2. Preparação — Copiar os dados para o drive E:](#2-preparação--copiar-os-dados-para-o-drive-e)
    - [Passo 1 — Criar a pasta de destino](#passo-1--criar-a-pasta-de-destino)
    - [Passo 2 — Copiar os dados com robocopy](#passo-2--copiar-os-dados-com-robocopy)
    - [Passo 3 — Renomear a pasta original como backup](#passo-3--renomear-a-pasta-original-como-backup)
  - [3. Criar o Junction](#3-criar-o-junction)
  - [4. Verificação](#4-verificação)
  - [5. Confirmar e limpar o backup](#5-confirmar-e-limpar-o-backup)
  - [🔄 E o `cursor-updater`?](#-e-o-cursor-updater)
  - [⚡ Referência Rápida](#-referência-rápida)
  - [🐛 Troubleshooting](#-troubleshooting)
  - [➡️ Próximo Passo](#️-próximo-passo)

---

## 🎯 Objetivo

Mesmo com o executável instalado no drive `E:\` (via [`INSTALLATION.md`](./INSTALLATION.md)), o Cursor continua gravando em `C:\Users\...\AppData\Roaming\Cursor`:

- Cache do motor Chromium/Electron
- Logs de sessão
- Histórico local de arquivos editados
- **Um banco SQLite por projeto** com todo o histórico de chat/Composer da IA
- Configurações e estado de extensões

Com uso intenso de Composer e vários projetos abertos, essa pasta cresce sem limite e sem limpeza automática.

Este guia cria um **junction** — um redirecionamento transparente no sistema de arquivos — que faz o Cursor "acreditar" que ainda grava no `C:\`, enquanto os dados ficam fisicamente no `E:\`.

---

## 🔗 O que é um Junction e por que não usar Symlink aqui

Um **junction** (`mklink /J`) é um link de pasta do Windows que redireciona transparentemente qualquer acesso a um caminho do `C:\` para uma pasta real em outro drive. O app nunca sabe que foi redirecionado.

```
C:\Users\...\AppData\Roaming\Cursor   ← junction (ponteiro, ~0 KB no C:)
         │
         └──────────────────────────> E:\AppDataCursor\Cursor
                                          (arquivos reais aqui)
```

**Por que junction e não symlink (`mklink /D`)?**

| | Junction (`mklink /J`) | Symlink de diretório (`mklink /D`) |
|---|---|---|
| Funciona sem permissão de Administrador para leitura | ✅ | ❌ |
| Funciona em caminhos locais para caminhos locais | ✅ | ✅ |
| Compatibilidade com apps Electron (VS Code/Cursor) | ✅ Comprovada | ⚠️ Pode ter edge cases |
| Visível como pasta normal no Explorer | ✅ | ✅ (ícone de atalho) |

Para este caso — redirecionar `AppData` local para outro drive local — o junction é a escolha mais robusta.

---

## 📦 O que fica em AppData\Roaming\Cursor

| Pasta / Arquivo | Conteúdo | Peso aproximado |
|---|---|---|
| `Cache`, `CachedData`, `Code Cache`, `GPUCache` | Cache do motor Chromium/Electron | Alto — cresce com uso |
| `logs\` | Logs de cada sessão (uma subpasta por data/hora) | Médio — acumula indefinidamente |
| `Crashpad\reports` | Relatórios de crash | Baixo |
| `User\History\` | Histórico local de versões de cada arquivo editado | Médio — proporcional a arquivos editados |
| `User\workspaceStorage\<hash>\` | Estado por projeto: abas + cache de indexação + **`state.vscdb` com histórico de chat/Composer da IA** | **Alto** — o principal responsável pelo crescimento |
| `User\globalStorage\state.vscdb` | Banco principal: conta logada, estado de extensões | Baixo-médio |
| `Backups\` | Cópias de arquivos não salvos no fechamento | Baixo |
| `Service Worker`, `blob_storage`, `IndexedDB` | Estruturas internas do Electron (cache de UI) | Médio |

---

## 🗺️ Visão Geral Antes e Depois

```
ANTES (padrão)
──────────────────────────────────────────────────────────
C:\Users\...\AppData\Roaming\Cursor\   ← tudo aqui, crescendo

DEPOIS (este guia)
──────────────────────────────────────────────────────────
C:\Users\...\AppData\Roaming\Cursor    ← junction (~0 KB, ponteiro)
         │
         └──> E:\AppDataCursor\Cursor\ ← dados reais aqui
```

---

## 1. Diagnóstico — O que está pesando

Antes de mover, vale saber o tamanho atual. Rode no PowerShell:

```powershell
Get-ChildItem "$env:APPDATA\Cursor" -Directory | ForEach-Object {
    $size = (Get-ChildItem $_.FullName -Recurse -File -ErrorAction SilentlyContinue |
             Measure-Object -Property Length -Sum).Sum
    [PSCustomObject]@{ Pasta = $_.Name; TamanhoGB = [math]::Round($size/1GB,2) }
} | Sort-Object TamanhoGB -Descending | Format-Table -AutoSize
```

Isso retorna um ranking das subpastas por tamanho — útil para entender o que está pesando mais antes de decidir o que fazer.

---

## 2. Preparação — Copiar os dados para o drive E:

> 🔴 **Feche o Cursor completamente antes de continuar.** Verifique no Gerenciador de Tarefas que nenhum processo `Cursor.exe` está ativo.

### Passo 1 — Criar a pasta de destino

```
E:\AppDataCursor\Cursor
```

### Passo 2 — Copiar os dados com robocopy

Abra o **PowerShell como Administrador** e rode:

```powershell
robocopy "%APPDATA%\Cursor" "E:\AppDataCursor\Cursor" /E /COPYALL /R:1 /W:1
```

| Flag | O que faz |
|---|---|
| `/E` | Copia todas as subpastas, incluindo as vazias |
| `/COPYALL` | Preserva atributos, timestamps e permissões |
| `/R:1 /W:1` | Em caso de erro, tenta 1 vez com 1 segundo de espera (evita travamento) |

### Passo 3 — Renomear a pasta original como backup

```powershell
Rename-Item "$env:APPDATA\Cursor" "$env:APPDATA\Cursor.bak"
```

> ✅ **Não apague ainda.** O `Cursor.bak` é sua rede de segurança — você só remove após confirmar que tudo funciona (Passo 5).

---

## 3. Criar o Junction

O `mklink /J` é um comando nativo do **CMD** — não do PowerShell.

1. Abra o **Prompt de Comando (CMD) como Administrador**
2. Execute:

```cmd
mklink /J "%APPDATA%\Cursor" "E:\AppDataCursor\Cursor"
```

Se funcionar, o terminal retorna:
```
Junction criado para C:\Users\Bruno P Goulart\AppData\Roaming\Cursor <<===>> E:\AppDataCursor\Cursor
```

> ⚠️ **O `mklink` roda no CMD, não no PowerShell.** Se rodar no PowerShell, o comando não é reconhecido ou se comporta de forma diferente. Abra o Prompt de Comando explicitamente.

---

## 4. Verificação

1. Abra o Explorer em `C:\Users\Bruno P Goulart\AppData\Roaming\`
2. A pasta `Cursor` deve aparecer com um **ícone de atalho** (seta pequena no canto), diferente das pastas comuns — isso confirma que é um junction
3. Abra o Cursor normalmente e confirme:
   - Login ainda está ativo
   - Configurações e extensões estão presentes
   - Histórico de chat dos projetos recentes ainda aparece no Composer

---

## 5. Confirmar e limpar o backup

Após alguns dias de uso normal sem problemas:

```powershell
Remove-Item "$env:APPDATA\Cursor.bak" -Recurse -Force
```

---

## 🔄 E o `cursor-updater`?

O Cursor mantém uma segunda pasta relevante em:
```
C:\Users\...\AppData\Local\cursor-updater\
```

Essa pasta acumula **instaladores antigos de atualização** — cada versão nova baixada para atualização automática fica guardada aqui. Vale checar o tamanho:

```powershell
Get-ChildItem "$env:LOCALAPPDATA\cursor-updater" -Directory | ForEach-Object {
    $size = (Get-ChildItem $_.FullName -Recurse -File -ErrorAction SilentlyContinue |
             Measure-Object Length -Sum).Sum
    [PSCustomObject]@{ Pasta = $_.Name; TamanhoMB = [math]::Round($size/1MB,1) }
} | Sort-Object TamanhoMB -Descending | Format-Table -AutoSize
```

Se estiver pesada, você pode aplicar o mesmo procedimento de junction:

```cmd
:: No CMD como Administrador
robocopy "%LOCALAPPDATA%\cursor-updater" "E:\AppDataCursor\cursor-updater" /E /COPYALL /R:1 /W:1
Rename-Item "%LOCALAPPDATA%\cursor-updater" "%LOCALAPPDATA%\cursor-updater.bak"
mklink /J "%LOCALAPPDATA%\cursor-updater" "E:\AppDataCursor\cursor-updater"
```

---

## ⚡ Referência Rápida

```powershell
# PowerShell como Admin — copiar dados e renomear original
robocopy "%APPDATA%\Cursor" "E:\AppDataCursor\Cursor" /E /COPYALL /R:1 /W:1
Rename-Item "$env:APPDATA\Cursor" "$env:APPDATA\Cursor.bak"
```

```cmd
:: CMD como Admin — criar o junction
mklink /J "%APPDATA%\Cursor" "E:\AppDataCursor\Cursor"
```

```powershell
# PowerShell — remover backup após confirmar que tudo funciona
Remove-Item "$env:APPDATA\Cursor.bak" -Recurse -Force
```

---

## 🐛 Troubleshooting

| Problema | Causa provável | Solução |
|---|---|---|
| `mklink` retorna "O arquivo já existe" | A pasta `Cursor.bak` ainda tem o nome original ou o rename não foi executado | Execute o `Rename-Item` antes do `mklink` e confirme que a pasta `Cursor` não existe mais no `AppData\Roaming` |
| `mklink` não reconhecido | Comando rodado no PowerShell em vez do CMD | Abra o **Prompt de Comando (CMD)** como Administrador — não o PowerShell |
| Cursor abre mas pede login novamente | Dados copiados incompletos ou permissões não preservadas | Confirme que o `robocopy` usou `/COPYALL` e terminou sem erros críticos |
| Histórico de chat sumiu | Junction criado antes de copiar os dados | Restaure o `Cursor.bak`, copie novamente com robocopy, depois recrie o junction |
| Pasta `Cursor` no Explorer não tem ícone de atalho | Junction não foi criado com sucesso | Re-execute o `mklink /J` no CMD como Administrador e verifique a mensagem de retorno |
| Cursor criou uma nova pasta `Cursor` no `AppData\Roaming` | Junction foi removido ou corrompido | Re-execute o procedimento completo do Passo 2 em diante |

---

## ➡️ Próximo Passo

Junction configurado? O Cursor agora grava tudo no `E:\` de forma permanente.

O próximo passo é configurar a **limpeza automática do cache** — que voltará a crescer com uso normal mesmo após o junction, porque esse é o comportamento do Electron.

→ Continue em **[`CLEANUP.md`](./CLEANUP.md)**