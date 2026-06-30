# 🐳 Docker Desktop Customizado no Windows 11 — Instalação 100% no Drive E:

> **Guia oficial para instalar e manter o Docker Desktop inteiramente fora do drive `C:\`** — programa, motor WSL e disco de imagens/containers, tudo isolado no drive `E:\`, mantendo o `C:\` livre.

[![Docker](https://img.shields.io/badge/-Docker%20Desktop-2496ED?style=flat-square&logo=docker&logoColor=white)](https://www.docker.com/products/docker-desktop/)
[![Windows](https://img.shields.io/badge/-Windows%2011-0078D4?style=flat-square&logo=windows&logoColor=white)](.)
[![WSL](https://img.shields.io/badge/-WSL2-FCC624?style=flat-square&logo=linux&logoColor=black)](.)
[![Nível](https://img.shields.io/badge/Nível-Intermediário-blue?style=flat-square)](.)

---

## 📋 Índice

- [🐳 Docker Desktop Customizado no Windows 11 — Instalação 100% no Drive E:](#-docker-desktop-customizado-no-windows-11--instalação-100-no-drive-e)
  - [📋 Índice](#-índice)
  - [🎯 Objetivo](#-objetivo)
  - [1. Instalação em Pasta Personalizada (Drive E:)](#1-instalação-em-pasta-personalizada-drive-e)
    - [Passo a passo](#passo-a-passo)
  - [2. Movendo o Disco de Imagens e Containers](#2-movendo-o-disco-de-imagens-e-containers)
    - [Passo a passo](#passo-a-passo-1)
    - [O que acontece nos bastidores](#o-que-acontece-nos-bastidores)
  - [3. Como Atualizar Sem Perder a Configuração](#3-como-atualizar-sem-perder-a-configuração)
    - [Passo a passo](#passo-a-passo-2)
  - [⚡ Referência Rápida](#-referência-rápida)
  - [🐛 Troubleshooting](#-troubleshooting)
  - [👤 Sobre o Autor](#-sobre-o-autor)
  - [📜 Licença](#-licença)

---

## 🎯 Objetivo

Este é o manual oficial da estrutura de instalação customizada do Docker Desktop, garantindo que:

- O drive `C:\` fique livre, sem o peso do motor Docker
- Todo o volume de dados (programa, WSL, imagens, containers, volumes) rode no drive `E:\`
- Atualizações futuras **não revertam** essa configuração

---

## 1. Instalação em Pasta Personalizada (Drive E:)

Por padrão, o instalador do Docker força os arquivos do programa no drive `C:\`. Para contornar isso, usamos o terminal para apontar o diretório de instalação desejado.

### Passo a passo

1. Baixe o instalador oficial (`DockerDesktopInstaller.exe`) do [site do Docker](https://www.docker.com/products/docker-desktop/)
2. Abra o **PowerShell como Administrador**
3. Navegue até a pasta onde o instalador foi baixado:

   ```powershell
   cd C:\Users\Bruno P Goulart\Downloads
   ```

4. Execute o comando apontando para o destino customizado:

   ```powershell
   ."DockerDesktopInstaller.exe" install --installation-dir="E:\software_install_windows\Docker"
   ```

5. Prossiga com as telas do instalador até a conclusão. O motor do Docker será extraído inteiramente no drive `E:\`

---

## 2. Movendo o Disco de Imagens e Containers

Mesmo com o programa instalado no drive `E:\`, o Docker cria o **arquivo de disco virtual** — onde ficam containers, volumes e imagens pesadas — no perfil do usuário, dentro de `C:\AppData`.

Para mover esse "tanque de armazenamento" para o drive `E:\` de forma segura, usamos a própria interface gráfica do Docker Desktop.

### Passo a passo

1. Abra o Docker Desktop e aguarde carregar (ícone fica verde)
2. Clique no ícone de **Engrenagem** (Settings) no topo superior direito
3. No menu lateral, acesse **Resources → Advanced**
4. Localize a seção **Disk image location**
5. Clique no botão azul **Browse** ao lado do caminho atual
6. Selecione sua pasta customizada no drive `E:\`

   > 💡 **Recomendado:** crie uma pasta chamada `wsl` ou `data` dentro de `E:\software_install_windows\Docker\`

7. Clique em **Apply & restart** (Aplicar e reiniciar), no canto inferior direito

### O que acontece nos bastidores

> ℹ️ O Docker move o arquivo `ext4.vhdx` — responsável por todo o peso do sistema — para o drive `E:`. A pasta original no drive `C:\` continua existindo, mas passa a conter apenas logs de texto leves e arquivos de trava (`.lock`) de 0 KB.

---

## 3. Como Atualizar Sem Perder a Configuração

> 🔴 **Nunca clique no botão "Update" dentro do painel do Docker Desktop.** Ele executa a rotina padrão e tenta jogar os arquivos de volta para o drive `C:\`.

Para atualizar mantendo a estrutura intacta — sem perder nenhuma imagem ou container existente — siga este rito sempre que houver uma nova versão.

### Passo a passo

1. Baixe manualmente o novo `DockerDesktopInstaller.exe` pelo site do Docker
2. Feche o Docker Desktop completamente: clique com o botão direito no ícone da barra de tarefas (perto do relógio) → **Quit Docker Desktop**
3. Garanta que o subsistema Linux foi desligado:

   ```powershell
   wsl --shutdown
   ```

4. Abra o **PowerShell como Administrador**, navegue até a pasta do novo instalador e rode o mesmo comando, apontando novamente para a pasta no drive `E:\`:

   ```powershell
   ."DockerDesktopInstaller.exe" install --installation-dir="E:\software_install_windows\Docker"
   ```

O instalador detecta a versão existente naquele caminho, atualiza apenas os arquivos do programa na pasta customizada, e **mantém o disco de imagens do drive `E:\` completamente intacto**.

---

## ⚡ Referência Rápida

```powershell
# Instalação inicial (ou reinstalação) apontando para o drive E:
."DockerDesktopInstaller.exe" install --installation-dir="E:\software_install_windows\Docker"

# Antes de qualquer atualização — desligar o WSL
wsl --shutdown
```

| Item | Local padrão (C:) | Local customizado (E:) |
|---|---|---|
| Arquivos do programa | `C:\Program Files\Docker` | `E:\software_install_windows\Docker` |
| Disco de imagens/containers (`ext4.vhdx`) | `C:\Users\<user>\AppData\...` | `E:\software_install_windows\Docker\wsl` (ou `data`) |
| Resíduo deixado no C: após migração | — | Apenas logs leves e arquivos `.lock` de 0 KB |

---

## 🐛 Troubleshooting

| Problema | Causa provável | Solução |
|---|---|---|
| Update reverte arquivos para o `C:\` | Atualização feita pelo botão "Update" do painel | Sempre atualizar via instalador manual com `--installation-dir` |
| Erro ao mover Disk image location | Docker Desktop ainda processando containers ativos | Pare todos os containers antes de mudar o local do disco |
| `wsl --shutdown` não resolve travamento | Processo do Docker ainda ativo em segundo plano | Confirme "Quit Docker Desktop" antes de rodar o comando |
| Instalador não reconhece pasta customizada | Caminho com espaço sem aspas no comando | Sempre envolver o caminho completo em aspas duplas |

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