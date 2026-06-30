# 🤖 Claude Desktop Customizado no Windows 11 — Instalação e Atualização no Drive E:

> **Guia oficial para instalar e atualizar o Claude Desktop fora do drive `C:\`**, usando o parâmetro `/D` do instalador via PowerShell, sem depender da interface gráfica padrão.

[![Claude](https://img.shields.io/badge/-Claude%20Desktop-CC785C?style=flat-square&logo=anthropic&logoColor=white)](https://claude.ai/download)
[![Windows](https://img.shields.io/badge/-Windows%2011-0078D4?style=flat-square&logo=windows&logoColor=white)](.)
[![PowerShell](https://img.shields.io/badge/-PowerShell-5391FE?style=flat-square&logo=powershell&logoColor=white)](.)
[![Nível](https://img.shields.io/badge/Nível-Intermediário-blue?style=flat-square)](.)

---

## 📋 Índice

- [🤖 Claude Desktop Customizado no Windows 11 — Instalação e Atualização no Drive E:](#-claude-desktop-customizado-no-windows-11--instalação-e-atualização-no-drive-e)
  - [📋 Índice](#-índice)
  - [🎯 Objetivo](#-objetivo)
  - [1. Preparação da Pasta e do Arquivo](#1-preparação-da-pasta-e-do-arquivo)
  - [2. Execução no Terminal](#2-execução-no-terminal)
  - [3. Limpeza Pós-Instalação](#3-limpeza-pós-instalação)
  - [🔄 Como Atualizar no Futuro](#-como-atualizar-no-futuro)
  - [⚡ Referência Rápida](#-referência-rápida)
  - [🐛 Troubleshooting](#-troubleshooting)
  - [👤 Sobre o Autor](#-sobre-o-autor)
  - [📜 Licença](#-licença)

---

## 🎯 Objetivo

Por padrão, o instalador do Claude Desktop (`ClaudeSetup.exe`) realiza uma instalação silenciosa e força todos os arquivos do programa para dentro do drive `C:\` (`AppData\Local\Anthropic`), sem dar opção de escolha ao usuário pela interface gráfica.

Este guia resolve isso usando o **PowerShell como Administrador** com o parâmetro universal `/D`, forçando tanto a instalação quanto as atualizações futuras para um segundo HD ou partição — como o drive `E:\`.

> 💡 O procedimento abaixo é **exatamente o mesmo** para a primeira instalação ou para atualizar quando uma nova versão for lançada.

---

## 1. Preparação da Pasta e do Arquivo

1. Crie a pasta de destino onde o programa deve residir (exemplo: `E:\software_install_windows\Claude`)
2. Baixe o instalador oficial no [site da Anthropic](https://claude.ai/download)
3. Coloque o arquivo baixado dentro da pasta personalizada criada no passo 1
4. Renomeie o instalador para `ClaudeSetup.exe`, caso venha com número de versão ou nome diferente do navegador — o nome precisa casar exatamente com o comando do terminal

---

## 2. Execução no Terminal

1. Clique no menu **Iniciar** do Windows, digite `PowerShell`, clique com o botão direito e selecione **Executar como Administrador**

2. Navegue até o diretório personalizado:

   ```powershell
   cd E:\software_install_windows\Claude
   ```

3. Execute o comando exato de instalação:

   ```powershell
   & ".\ClaudeSetup.exe" install /D="E:\software_install_windows\Claude"
   ```

---

## 3. Limpeza Pós-Instalação

O instalador roda de forma silenciosa em segundo plano por alguns segundos e extrai o programa completo na pasta personalizada.

- Assim que os arquivos e pastas do Claude aparecerem no diretório, o processo terminou
- Você pode apagar o `ClaudeSetup.exe` da pasta personalizada para deixar o ambiente limpo — o programa roda direto dos arquivos extraídos

---

## 🔄 Como Atualizar no Futuro

> 🔴 **Não atualize pelo aviso do próprio aplicativo.** Quando o Claude Desktop solicitar uma nova versão pela interface, ele tentará jogar os arquivos de volta para o drive `C:\`.

Para atualizar mantendo a estrutura intacta, repita rigorosamente o mesmo rito:

1. Baixe o instalador atualizado no site oficial
2. Coloque-o na pasta `E:\software_install_windows\Claude`
3. Renomeie o arquivo para `ClaudeSetup.exe`
4. Abra o PowerShell como Administrador, navegue até a pasta e rode o mesmo comando do [Passo 2](#2-execução-no-terminal)
5. O instalador sobrescreve a versão antiga com os arquivos novos, sem perder configurações
6. Delete o `ClaudeSetup.exe` novamente após a conclusão

---

## ⚡ Referência Rápida

```powershell
# Navegar até a pasta customizada
cd E:\software_install_windows\Claude

# Instalar ou atualizar — mesmo comando para os dois casos
& ".\ClaudeSetup.exe" install /D="E:\software_install_windows\Claude"
```

| Etapa | Ação |
|---|---|
| Instalação inicial | Baixar → renomear para `ClaudeSetup.exe` → rodar comando → deletar instalador |
| Atualização | Baixar nova versão → renomear → rodar o **mesmo** comando → deletar instalador |
| Local final do programa | `E:\software_install_windows\Claude` (nunca `C:\AppData\Local\Anthropic`) |

---

## 🐛 Troubleshooting

| Problema | Causa provável | Solução |
|---|---|---|
| Comando não encontra o instalador | Nome do arquivo diferente de `ClaudeSetup.exe` | Renomeie exatamente para `ClaudeSetup.exe` antes de rodar o comando |
| Instalação volta para o `C:\` | Atualização feita pelo aviso interno do app | Sempre atualizar manualmente via PowerShell com `/D` |
| `/D` parece ser ignorado | Caminho sem aspas ou com espaço mal formatado | Sempre envolver o caminho completo em aspas duplas no comando |
| PowerShell nega permissão | Não executado como Administrador | Reabra o PowerShell com **Executar como Administrador** |

---

## 👤 Sobre o Autor

<table>
<tr>
<td width="140">
<a href="https://bru.ia.br/">
<img src="https://bru.ia.br/001_repo_external/og-image.webp" width="120" alt="Bruno Goulart"/>
</a>
</td>
<td>

**Bruno Goulart** — AI Automation Specialist & Full Stack Developer

Uno a robustez de 18+ anos de código escrito na raça e forjado no braço à inteligência de LLMs, ferramentas hype de automação, DevOps e arquitetura full-stack — do MVP ao deploy em produção.

Projeto e entrego soluções escaláveis, seguras e inovadoras: integro LLMs em produtos existentes e construo do zero arquiteturas orientadas a agentes autônomos. Especialista em SDD (Specification-Driven Development), AI Agents (LangChain, LangGraph, MCP), automação n8n e infraestrutura DevOps (Docker, Portainer, GitLab/GitHub CI).

🔗 **[bru.ia.br](https://bru.ia.br/)**

</td>
</tr>
</table>

---

## 📜 Licença

MIT — use, adapte e distribua livremente, mantendo os créditos de autoria.

---

<div align="center">

*Guia mantido como referência para ambientes de desenvolvimento com múltiplos drives.*

</div>