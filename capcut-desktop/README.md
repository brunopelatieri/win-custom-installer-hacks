# 🎬 CapCut Customizado no Windows 11 — Instalação e Cache no Drive E:

> **Guia oficial para instalar o CapCut fora do drive `C:\`** usando o parâmetro `/D` do instalador via PowerShell, e configurar todas as pastas de rascunho, cache e renderização para rodar inteiramente no drive `E:\`.

[![CapCut](https://img.shields.io/badge/-CapCut-00D2C3?style=flat-square&logoColor=white)](https://www.capcut.com/)
[![Windows](https://img.shields.io/badge/-Windows%2011-0078D4?style=flat-square&logo=windows&logoColor=white)](.)
[![PowerShell](https://img.shields.io/badge/-PowerShell-5391FE?style=flat-square&logo=powershell&logoColor=white)](.)
[![Nível](https://img.shields.io/badge/Nível-Intermediário-blue?style=flat-square)](.)

---

## 📋 Índice

- [🎬 CapCut Customizado no Windows 11 — Instalação e Cache no Drive E:](#-capcut-customizado-no-windows-11--instalação-e-cache-no-drive-e)
  - [📋 Índice](#-índice)
  - [🎯 Objetivo](#-objetivo)
  - [1. Preparação da Pasta e do Arquivo](#1-preparação-da-pasta-e-do-arquivo)
  - [2. Execução no Terminal](#2-execução-no-terminal)
  - [3. Limpeza Pós-Instalação](#3-limpeza-pós-instalação)
  - [4. Configuração de Pastas — Rascunho](#4-configuração-de-pastas--rascunho)
    - [Onde configurar](#onde-configurar)
    - [Caminhos recomendados](#caminhos-recomendados)
    - [Cache de Rascunho](#cache-de-rascunho)
    - [Sincronização automática](#sincronização-automática)
  - [5. Configuração de Pastas — Desempenho](#5-configuração-de-pastas--desempenho)
    - [Onde configurar](#onde-configurar-1)
    - [Caminhos recomendados](#caminhos-recomendados-1)
    - [Configurações de performance recomendadas](#configurações-de-performance-recomendadas)
  - [⚡ Referência Rápida](#-referência-rápida)
    - [Estrutura final recomendada no drive E:](#estrutura-final-recomendada-no-drive-e)
  - [🐛 Troubleshooting](#-troubleshooting)

---

## 🎯 Objetivo

Por padrão, o instalador do CapCut força a instalação do programa no drive `C:\`, sem opção de escolha pela interface gráfica. Além disso, mesmo após mover o programa, o app continua salvando rascunhos, cache de renderização e arquivos de proxy em pastas internas do sistema.

Este guia cobre as duas frentes necessárias para isolar o CapCut completamente do drive `C:\`:

1. **Instalação do programa** via PowerShell com o parâmetro `/D`
2. **Configuração interna** das pastas de Rascunho e Desempenho, direto no painel de Configurações do app

> 💡 O procedimento de instalação abaixo é o mesmo usado nos guias de [Docker Desktop](./docker_desktop_drive_e_guide.md) e [Claude Desktop](./claude_desktop_drive_e_guide.md) — mesma lógica, instalador diferente.

---

## 1. Preparação da Pasta e do Arquivo

1. Crie a pasta de destino onde o programa deve residir (exemplo: `E:\software_install_windows\CapCut`)
2. Baixe o instalador oficial no site do CapCut
3. Coloque o arquivo baixado dentro da pasta personalizada criada no passo 1

> ℹ️ O instalador do CapCut já vem com um nome longo (`CapCut_7656608844948340737_installer.exe`) — diferente do Claude, **não é necessário renomear**. Use o nome exatamente como ele foi baixado.

---

## 2. Execução no Terminal

1. Clique no menu **Iniciar** do Windows, digite `PowerShell`, clique com o botão direito e selecione **Executar como Administrador**

2. Navegue até o diretório personalizado:

   ```powershell
   cd E:\software_install_windows\CapCut
   ```

3. Execute o comando exato de instalação:

   ```powershell
   & ".\CapCut_7656608844948340737_installer.exe" install /D="E:\software_install_windows\CapCut"
   ```

---

## 3. Limpeza Pós-Instalação

O instalador roda de forma silenciosa em segundo plano por alguns segundos e extrai o programa completo na pasta personalizada.

- Assim que os arquivos e pastas do CapCut aparecerem no diretório, o processo terminou
- Você pode apagar o instalador `.exe` da pasta personalizada para deixar o ambiente limpo — o programa roda direto dos arquivos extraídos

---

## 4. Configuração de Pastas — Rascunho

Com o programa instalado no drive `E:\`, o próximo passo é garantir que **todo o conteúdo gerado pelo uso** — rascunhos, gravações de tela, downloads e presets — também fique fora do `C:\`.

### Onde configurar

```
CapCut → ícone de Configurações → aba "Rascunho"
```

### Caminhos recomendados

| Campo | Caminho sugerido | Função |
|---|---|---|
| **Salvar em** | `E:\Videos\CapCut\Cloud files\CapCut Drafts` | Onde os projetos/rascunhos em edição são salvos |
| **Gravação de tela** | `E:\Videos\CapCut\CapCut Screen Record` | Destino dos arquivos gravados pela função de tela |
| **Baixar para** | `E:\Videos\CapCut\Cloud files` | Destino de downloads feitos a partir da nuvem do CapCut |
| **Predefinir caminho para salvar** | `E:\Videos\CapCut\Presets` | Local padrão sugerido ao exportar/salvar manualmente |

### Cache de Rascunho

| Opção | Recomendação |
|---|---|
| **Não excluir** | Evite — acumula cache indefinidamente |
| **Excluir automaticamente o cache de `30` dias atrás** | ✅ Recomendado — mantém o disco saudável sem perder rascunhos recentes |

> 💡 O campo **Tamanho do cache** mostra o volume atual ocupado (ex: `272.06MB`) e tem um botão de lixeira ao lado para limpeza manual imediata, sem esperar os 30 dias.

### Sincronização automática

A opção **Sincronização automática** (upload para nuvem) é independente do local de salvamento em disco — ative apenas se você usa a conta CapCut Cloud e quer backup automático dos projetos.

---

## 5. Configuração de Pastas — Desempenho

A aba **Desempenho** controla onde ficam os arquivos pesados gerados durante a edição: cache de renderização, proxy e remux.

### Onde configurar

```
CapCut → ícone de Configurações → aba "Desempenho"
```

### Caminhos recomendados

| Campo | Caminho sugerido | Função |
|---|---|---|
| **Salvar arquivos em** (Remuxar automaticamente) | `E:\Videos\CapCut\RemuxCache` | Arquivos convertidos automaticamente para `.MOV` para evitar atraso na reprodução |
| **Salvar proxies em** | `E:\Videos\CapCut\agencycache` | Versões de baixa resolução usadas para edição fluida sem perder qualidade no export final |
| **Renderizando localização do cache** | `E:\Videos\CapCut\prerender` | Cache de pré-renderização da parte selecionada do vídeo |

### Configurações de performance recomendadas

| Opção | Recomendação | Motivo |
|---|---|---|
| **Acelerar codificação de hardware** | ✅ Ativado | Usa a GPU para codificar — exporta mais rápido |
| **Acelerar descodificação de hardware** | ✅ Ativado | Usa a GPU para decodificar — preview mais fluido |
| **Renderizar interface com GPU** | ✅ Ativado | Interface do app mais responsiva |
| **Otimização automática** | `Nenhum` (ou ajustar conforme a máquina) | Evita que o CapCut force ajustes automáticos sem necessidade |
| **Remuxar automaticamente** | Opcional — ative se os vídeos importados derem atraso na reprodução | Conversão automática para `.MOV` tem custo de espaço em disco |
| **Proxy** | Opcional — ative em projetos pesados (4K, múltiplas camadas) | Acelera a edição em troca de espaço extra no disco |
| **Apagar automaticamente** (cache de pré-renderização) | `10 GB` (ajustável) | Define o teto de espaço antes da limpeza automática do cache de pré-render |
| **Renderizar automaticamente** | Opcional | Renderiza a parte selecionada do vídeo em segundo plano para preview mais suave |

> ℹ️ Os campos **Tamanho do arquivo** (Remux) e **Tamanho do proxy** mostram `0B` até que essas funções sejam efetivamente usadas — é esperado ficarem zerados se Remux e Proxy estiverem desativados.

---

## ⚡ Referência Rápida

```powershell
# Navegar até a pasta customizada
cd E:\software_install_windows\CapCut

# Instalar — substitua pelo nome exato do instalador baixado
& ".\CapCut_7656608844948340737_installer.exe" install /D="E:\software_install_windows\CapCut"
```

### Estrutura final recomendada no drive E:

```text
E:\
├── software_install_windows\
│   └── CapCut\                      ← Programa instalado
└── Videos\
    └── CapCut\
        ├── Cloud files\
        │   └── CapCut Drafts\       ← Rascunhos/projetos
        ├── CapCut Screen Record\    ← Gravações de tela
        ├── Presets\                 ← Predefinições de exportação
        ├── RemuxCache\              ← Cache de remux automático
        ├── agencycache\             ← Cache de proxy
        └── prerender\               ← Cache de pré-renderização
```

| Etapa | Ação |
|---|---|
| Instalação | Baixar instalador → colocar na pasta → rodar comando com `/D` |
| Pós-instalação | Apagar o instalador `.exe` da pasta personalizada |
| Configuração de conteúdo | Aba **Rascunho** → redirecionar todos os 4 caminhos para `E:\Videos\CapCut\` |
| Configuração de performance | Aba **Desempenho** → redirecionar Remux, Proxy e Pré-render para `E:\Videos\CapCut\` |

---

## 🐛 Troubleshooting

| Problema | Causa provável | Solução |
|---|---|---|
| Comando não encontra o instalador | Nome do arquivo digitado errado no terminal | Copie o nome exato do `.exe` baixado, incluindo os números de versão |
| CapCut volta a salvar em `C:\` | Caminho não foi alterado em **todas** as abas (Rascunho **e** Desempenho) | Revise as duas abas — são seções independentes dentro de Configurações |
| Cache cresce rapidamente mesmo com limpeza automática configurada | Limite de "Excluir automaticamente" definido para muitos dias (ex: 30) | Reduza o intervalo ou use o botão de lixeira manual ao lado do tamanho do cache |
| Proxy/Remux com `0B` mesmo após uso | Função não estava ativada antes da edição | Ative o toggle **antes** de importar/editar os vídeos — cache não é retroativo |
| PowerShell nega permissão | Não executado como Administrador | Reabra o PowerShell com **Executar como Administrador** |

---

<div align="center">

*Guia mantido como referência para ambientes de desenvolvimento com múltiplos drives.*

</div>