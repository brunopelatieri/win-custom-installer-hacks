# 🖱️ Cursor IDE — Guia Completo de Instalação, Dados e Manutenção no Windows 11

> **Repositório de referência para instalar o Cursor IDE fora do drive `C:\`, mover seus dados via link simbólico, limpar o cache de forma segura e automatizar a manutenção.**

[![Cursor](https://img.shields.io/badge/-Cursor%20IDE-000000?style=flat-square&logo=cursor&logoColor=white)](https://www.cursor.sh/)
[![Windows](https://img.shields.io/badge/-Windows%2011-0078D4?style=flat-square&logo=windows&logoColor=white)](.)
[![PowerShell](https://img.shields.io/badge/-PowerShell-5391FE?style=flat-square&logo=powershell&logoColor=white)](.)
[![CMD](https://img.shields.io/badge/-CMD%20Admin-4D4D4D?style=flat-square&logo=windows-terminal&logoColor=white)](.)
[![Nível](https://img.shields.io/badge/Nível-Intermediário%20a%20Avançado-orange?style=flat-square)](.)

---

## 📋 Índice

- [🖱️ Cursor IDE — Guia Completo de Instalação, Dados e Manutenção no Windows 11](#️-cursor-ide--guia-completo-de-instalação-dados-e-manutenção-no-windows-11)
  - [📋 Índice](#-índice)
  - [🎯 Por que este repositório existe](#-por-que-este-repositório-existe)
  - [📁 Estrutura dos Arquivos](#-estrutura-dos-arquivos)
    - [Quando usar cada arquivo](#quando-usar-cada-arquivo)
  - [🚀 Por onde começar](#-por-onde-começar)
  - [👤 Sobre o Autor](#-sobre-o-autor)
  - [📜 Licença](#-licença)

---

## 🎯 Por que este repositório existe

O Cursor é um fork do VS Code construído em Electron. Esse modelo de app tem um comportamento conhecido: **acumula dados silenciosamente em `AppData\Roaming` sem limpeza automática**, incluindo caches do motor Chromium, logs por sessão, histórico local de arquivos e — o mais pesado — um banco SQLite por projeto contendo todo o histórico de chat e Composer com a IA.

Somado ao fato de que o instalador padrão também vai para o `C:\`, isso cria dois problemas distintos que merecem soluções distintas:

| Problema | Solução | Arquivo |
|---|---|---|
| Executável instalado no `C:\` por padrão | Parâmetro `/D` no instalador via PowerShell | [`INSTALLATION.md`](./INSTALLATION.md) |
| Dados, cache e histórico de IA crescendo no `C:\` | Junction (link de pasta) para outro drive | [`SYMLINK.md`](./SYMLINK.md) |
| Cache acumulando com uso normal após configuração | Script de limpeza + Agendador de Tarefas | [`CLEANUP.md`](./CLEANUP.md) |

---

## 📁 Estrutura dos Arquivos

```
cursor-guide/
├── README.md          ← Este arquivo — visão geral e ponto de entrada
├── INSTALLATION.md    ← Instalação do Cursor no drive E: via parâmetro /D
├── SYMLINK.md         ← Mover AppData\Roaming\Cursor para drive E: via junction
└── CLEANUP.md         ← Limpeza segura do cache, script automático e manutenção
```

### Quando usar cada arquivo

| Situação | Arquivo |
|---|---|
| Instalação nova, quer evitar o `C:\` desde o início | [`INSTALLATION.md`](./INSTALLATION.md) |
| Cursor já instalado, quer mover só os dados para outro drive | [`SYMLINK.md`](./SYMLINK.md) |
| `AppData\Roaming\Cursor` está gigante, quer entender o que pode apagar | [`CLEANUP.md`](./CLEANUP.md) |
| Quer automatizar a limpeza periódica do cache | [`CLEANUP.md`](./CLEANUP.md) — seção Script Automático |
| Quer mover tudo (executável + dados) para o drive `E:\` | [`INSTALLATION.md`](./INSTALLATION.md) + [`SYMLINK.md`](./SYMLINK.md) |

---

## 🚀 Por onde começar

```
Instalação nova no drive E: (situação mais comum)?
   → Comece em INSTALLATION.md, depois SYMLINK.md

Cursor já instalado, só quer limpar espaço no C:?
   → Direto no CLEANUP.md

Cursor já instalado, quer mover tudo para E:?
   → SYMLINK.md (dados) é suficiente — o executável pode ficar no C: se preferir
   → Ou desinstale, reinstale com INSTALLATION.md e configure SYMLINK.md em seguida
```

> 💡 **Parte desta série:** este guia segue o mesmo padrão dos guias de [Docker Desktop](../docker_desktop_drive_e_guide.md), [Claude Desktop](../claude_desktop_drive_e_guide.md) e [CapCut](../capcut_drive_e_guide.md) para Windows 11 com múltiplos drives.

---

## 👤 Sobre o Autor

<table>
<tr>
<td width="120">
<a href="https://bru.ia.br/">
<img src="https://bru.ia.br/001_repo_external/og-image.webp" width="100" alt="Bruno Goulart"/>
</a>
</td>
<td>

**Bruno Goulart** — AI Automation Specialist & Full Stack Developer

Uno a robustez de 18+ anos de código escrito na raça e forjado no braço à inteligência de LLMs, ferramentas hype de automação, DevOps e arquitetura full-stack — do MVP ao deploy em produção.

🔗 **[bru.ia.br](https://bru.ia.br/)**

</td>
</tr>
</table>

---

## 📜 Licença

MIT — use, adapte e distribua livremente, mantendo os créditos de autoria.