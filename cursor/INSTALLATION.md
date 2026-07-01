# ⚙️ Cursor IDE — Instalação no Drive E: (Windows 11)

> **Guia para instalar o Cursor IDE fora do drive `C:\`**, usando o parâmetro `/D` do instalador via PowerShell — mesmo padrão dos guias de Docker Desktop, Claude Desktop e CapCut desta série.

[![Cursor](https://img.shields.io/badge/-Cursor%20IDE-000000?style=flat-square&logo=cursor&logoColor=white)](https://www.cursor.sh/)
[![Windows](https://img.shields.io/badge/-Windows%2011-0078D4?style=flat-square&logo=windows&logoColor=white)](.)
[![PowerShell](https://img.shields.io/badge/-PowerShell-5391FE?style=flat-square&logo=powershell&logoColor=white)](.)
[![Nível](https://img.shields.io/badge/Nível-Intermediário-blue?style=flat-square)](.)

---

## 📋 Índice

- [⚙️ Cursor IDE — Instalação no Drive E: (Windows 11)](#️-cursor-ide--instalação-no-drive-e-windows-11)
  - [📋 Índice](#-índice)
  - [🎯 Objetivo](#-objetivo)
  - [1. Preparação da Pasta e do Arquivo](#1-preparação-da-pasta-e-do-arquivo)
  - [2. Execução no Terminal](#2-execução-no-terminal)
  - [3. Limpeza Pós-Instalação](#3-limpeza-pós-instalação)
  - [🔄 Como Atualizar no Futuro](#-como-atualizar-no-futuro)
  - [⚡ Referência Rápida](#-referência-rápida)
  - [🐛 Troubleshooting](#-troubleshooting)
  - [➡️ Próximo Passo](#️-próximo-passo)

---

## 🎯 Objetivo

Por padrão, o instalador do Cursor força o executável para `C:\Users\...\AppData\Local\Programs\cursor\`, sem opção de escolha pela interface gráfica.

Este guia move o **executável e os arquivos do programa** para o drive `E:\` via parâmetro `/D` no instalador — a mesma abordagem desta série para Docker, Claude Desktop e CapCut.

> ⚠️ **Atenção:** mover o executável não move os dados de usuário (`AppData\Roaming\Cursor`). Para mover também o histórico de IA, cache e configurações, veja [`SYMLINK.md`](./SYMLINK.md) após concluir este guia.

---

## 1. Preparação da Pasta e do Arquivo

1. Crie a pasta de destino para o executável:
   ```
   E:\software_install_windows\Cursor
   ```

2. Baixe o instalador oficial em [cursor.sh](https://www.cursor.sh/)
3. Coloque o arquivo baixado dentro da pasta criada no passo 1
4. Renomeie o instalador exatamente para `CursorSetup.exe`

> ℹ️ O instalador do Cursor costuma vir com nome genérico (`CursorSetup-x64.exe`, `CursorUserSetup-x64.exe` ou similar). Renomeie sempre para `CursorSetup.exe` para garantir que o comando funcione sem ajuste.

---

## 2. Execução no Terminal

1. Clique em **Iniciar**, digite `PowerShell`, clique com o botão direito e selecione **Executar como Administrador**

2. Navegue até o diretório do instalador:

   ```powershell
   cd "E:\software_install_windows\Cursor"
   ```

3. Execute a instalação controlada com `-Wait`:

   ```powershell
   Start-Process ".\CursorSetup.exe" -ArgumentList "install", "/D=E:\software_install_windows\Cursor" -Wait
   ```

> 💡 **Por que `Start-Process` com `-Wait`?** O Cursor usa o instalador Squirrel (igual ao Claude Desktop), que lança um processo filho e retorna imediatamente — dando a falsa impressão de que terminou. O `-Wait` garante que o PowerShell aguarde a extração completa antes de liberar o terminal.

---

## 3. Limpeza Pós-Instalação

O instalador roda silenciosamente. Como usamos `-Wait`, o PowerShell só libera o terminal após a extração concluída.

- Verifique se os arquivos e pastas do Cursor aparecem em `E:\software_install_windows\Cursor\`
- Apague o `CursorSetup.exe` da pasta — o programa roda direto dos arquivos extraídos
- Abra o Cursor normalmente e confirme que inicia sem erros

---

## 🔄 Como Atualizar no Futuro

> 🔴 **Não use o botão "Check for Updates" dentro do Cursor.** A atualização automática reinstala o executável no caminho padrão do `C:\`.

Para atualizar mantendo a estrutura intacta:

1. Baixe o novo instalador no site oficial
2. Coloque em `E:\software_install_windows\Cursor` e renomeie para `CursorSetup.exe`
3. Feche o Cursor completamente (verifique no Gerenciador de Tarefas que não há processo `Cursor.exe` ativo)
4. Abra o PowerShell como Administrador e repita o comando do [Passo 2](#2-execução-no-terminal)
5. O instalador sobrescreve a versão anterior — configurações e histórico não são afetados
6. Apague o `CursorSetup.exe` após a conclusão

---

## ⚡ Referência Rápida

```powershell
# Navegar até a pasta customizada
cd "E:\software_install_windows\Cursor"

# Instalar ou atualizar — mesmo comando para os dois casos
Start-Process ".\CursorSetup.exe" -ArgumentList "install", "/D=E:\software_install_windows\Cursor" -Wait
```

| Etapa | Ação |
|---|---|
| Instalação inicial | Baixar → renomear para `CursorSetup.exe` → rodar comando → deletar instalador |
| Atualização | Fechar Cursor → baixar nova versão → renomear → rodar o **mesmo** comando → deletar instalador |
| Local final do executável | `E:\software_install_windows\Cursor\` (nunca `C:\AppData\Local\Programs\cursor\`) |

---

## 🐛 Troubleshooting

| Problema | Causa provável | Solução |
|---|---|---|
| Instalador não encontrado | Nome do arquivo diferente de `CursorSetup.exe` | Renomeie exatamente para `CursorSetup.exe` antes de rodar o comando |
| Cursor instala em `C:\` mesmo com o comando | `/D` ignorado por sintaxe incorreta | Use exatamente o `Start-Process` do Passo 2, sem espaços ao redor do `=` |
| Cursor abre mas não encontra extensões/configurações | Dados de usuário em caminho diferente | Normal — os dados ficam em `AppData\Roaming\Cursor`; veja [`SYMLINK.md`](./SYMLINK.md) para mover também |
| PowerShell nega permissão | Não executado como Administrador | Reabra o PowerShell com **Executar como Administrador** |
| Atualização voltou para `C:\` | Atualização feita pelo botão interno do Cursor | Sempre atualizar manualmente via PowerShell com o comando `/D` |

---

## ➡️ Próximo Passo

Executável instalado no drive `E:\`? O próximo passo é mover também os dados de usuário — histórico de chat, cache e configurações — que ainda ficam em `C:\AppData\Roaming\Cursor`.

→ Continue em **[`SYMLINK.md`](./SYMLINK.md)**