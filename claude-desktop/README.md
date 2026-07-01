# 🤖 Claude Desktop Avançado no Windows 11 — Executável + Dados 100% no Drive E:

> **Guia avançado para mover o Claude Desktop completamente para fora do drive `C:\`** — executável via parâmetro `/D` e dados/cache via Link Simbólico (`mklink`), garantindo que nada relevante ocupe o drive do sistema.

[![Claude](https://img.shields.io/badge/-Claude%20Desktop-CC785C?style=flat-square&logo=anthropic&logoColor=white)](https://claude.ai/download)
[![Windows](https://img.shields.io/badge/-Windows%2011-0078D4?style=flat-square&logo=windows&logoColor=white)](.)
[![PowerShell](https://img.shields.io/badge/-PowerShell-5391FE?style=flat-square&logo=powershell&logoColor=white)](.)
[![CMD](https://img.shields.io/badge/-CMD%20Admin-4D4D4D?style=flat-square&logo=windows-terminal&logoColor=white)](.)
[![Nível](https://img.shields.io/badge/Nível-Avançado-red?style=flat-square)](.)

---

## 📋 Índice

- [🤖 Claude Desktop Avançado no Windows 11 — Executável + Dados 100% no Drive E:](#-claude-desktop-avançado-no-windows-11--executável--dados-100-no-drive-e)
  - [📋 Índice](#-índice)
  - [🎯 Objetivo e Diferença em relação ao Guia Básico](#-objetivo-e-diferença-em-relação-ao-guia-básico)
  - [🔗 O que é um Link Simbólico (Symlink)?](#-o-que-é-um-link-simbólico-symlink)
  - [🗺️ Visão Geral do Que Será Movido](#️-visão-geral-do-que-será-movido)
  - [1. Link Simbólico — Movendo os Dados (AppData\\Roaming)](#1-link-simbólico--movendo-os-dados-appdataroaming)
    - [Passo 1 — Criar a pasta de destino no drive E:](#passo-1--criar-a-pasta-de-destino-no-drive-e)
    - [Passo 2 — Mover dados existentes (se o Claude já foi usado antes)](#passo-2--mover-dados-existentes-se-o-claude-já-foi-usado-antes)
    - [Passo 3 — Criar o Link Simbólico](#passo-3--criar-o-link-simbólico)
  - [2. Preparação da Pasta e do Instalador](#2-preparação-da-pasta-e-do-instalador)
  - [3. Instalação via PowerShell (modo controlado)](#3-instalação-via-powershell-modo-controlado)
  - [4. Limpeza Pós-Instalação](#4-limpeza-pós-instalação)
  - [🔄 Como Atualizar no Futuro](#-como-atualizar-no-futuro)
  - [⚡ Referência Rápida](#-referência-rápida)
    - [CMD (Administrador) — criar o symlink](#cmd-administrador--criar-o-symlink)
    - [PowerShell (Administrador) — instalar ou atualizar](#powershell-administrador--instalar-ou-atualizar)
    - [Estrutura final no drive E:](#estrutura-final-no-drive-e)
    - [O que fica no C: após a configuração](#o-que-fica-no-c-após-a-configuração)
  - [🐛 Troubleshooting](#-troubleshooting)
  - [👤 Sobre o Autor](#-sobre-o-autor)
  - [📜 Licença](#-licença)

---

## 🎯 Objetivo e Diferença em relação ao Guia Básico

O [guia básico](./claude_desktop_drive_e_guide.md) desta série move apenas o **executável do programa** para o drive `E:\` usando o parâmetro `/D` do instalador. Isso já elimina o peso maior da instalação do drive `C:\`.

Este guia vai além e move **também os dados de usuário** — histórico de conversas, configurações, cache de sessão — que o Claude grava em `C:\Users\...\AppData\Roaming\Claude`.

| O que é movido | Guia Básico | Este Guia |
|---|---|---|
| Executável do programa | ✅ `/D` no instalador | ✅ `/D` no instalador |
| Dados, cache e configurações (`AppData\Roaming\Claude`) | ❌ Permanece no `C:\` | ✅ Link simbólico para `E:\` |

---

## 🔗 O que é um Link Simbólico (Symlink)?

Um link simbólico (`mklink /d`) é um **atalho transparente no sistema de arquivos**. O Windows cria um "ponteiro" num caminho do `C:\` que redireciona silenciosamente todas as leituras e gravações para uma pasta real em outro drive.

```
C:\Users\...\AppData\Roaming\Claude   ← ponteiro (symlink)
         │
         └──────────────────────────> E:\software_install_windows\ClaudeData
                                          (arquivos reais aqui)
```

O Claude Desktop nunca saberá que foi redirecionado — ele continua "achando" que grava no caminho padrão do `C:\`, mas fisicamente tudo vai para o `E:\`.

---

## 🗺️ Visão Geral do Que Será Movido

```
ANTES (padrão)
─────────────────────────────────────────────
C:\AppData\Local\Anthropic\         ← executável
C:\Users\...\AppData\Roaming\Claude ← dados, cache, histórico

DEPOIS (este guia)
─────────────────────────────────────────────
E:\software_install_windows\Claude\     ← executável  (via /D)
E:\software_install_windows\ClaudeData\ ← dados, cache (via symlink)
C:\Users\...\AppData\Roaming\Claude     ← ponteiro simbólico (vazio, 0 KB)
```

---

## 1. Link Simbólico — Movendo os Dados (AppData\Roaming)

> ⚠️ **Esta etapa deve ser feita ANTES de rodar o instalador.** Se o Claude já estiver instalado, desinstale-o primeiro ou execute este passo antes da reinstalação.

### Passo 1 — Criar a pasta de destino no drive E:

Crie a pasta onde os dados reais ficarão armazenados:

```
E:\software_install_windows\ClaudeData
```

### Passo 2 — Mover dados existentes (se o Claude já foi usado antes)

Se você já utilizou o Claude Desktop anteriormente, os dados já existem no caminho padrão. Mova-os agora:

1. Abra o Explorer e navegue até:
   ```
   C:\Users\Bruno P Goulart\AppData\Roaming\
   ```
   > 💡 Se a pasta `AppData` não aparecer, ative "Itens ocultos" em **Exibir → Mostrar → Itens ocultos** no Explorer.

2. **Recorte** (Ctrl+X) a pasta `Claude` inteira
3. **Cole** (Ctrl+V) dentro de `E:\software_install_windows\ClaudeData`

> 🔴 **A pasta `Claude` não pode mais existir no caminho original do `C:\`** antes do próximo passo — o `mklink` falhará se já houver uma pasta com esse nome no destino.

### Passo 3 — Criar o Link Simbólico

> ⚠️ O comando `mklink` é nativo do **CMD**, não do PowerShell. Abrir no terminal errado é o erro mais comum nesta etapa.

1. Clique em **Iniciar**, digite `cmd`, clique com o botão direito e selecione **Executar como Administrador**
2. Execute o comando:

```cmd
mklink /d "C:\Users\Bruno P Goulart\AppData\Roaming\Claude" "E:\software_install_windows\ClaudeData"
```

Se funcionar, o terminal retorna:
```
Link simbólico criado para C:\Users\Bruno P Goulart\AppData\Roaming\Claude <<===>> E:\software_install_windows\ClaudeData
```

3. Verifique: abra o Explorer em `C:\Users\Bruno P Goulart\AppData\Roaming\` — a pasta `Claude` deve aparecer com um ícone de atalho (seta), diferente das pastas comuns.

---

## 2. Preparação da Pasta e do Instalador

1. Crie a pasta de destino para o executável do programa:
   ```
   E:\software_install_windows\Claude
   ```

2. Baixe o instalador oficial no [site da Anthropic](https://claude.ai/download)
3. Coloque o arquivo baixado dentro da pasta criada no passo 1
4. Renomeie o instalador exatamente para `ClaudeSetup.exe`

> ℹ️ O instalador do Claude pode vir com nome genérico ou com número de versão. Renomeie sempre para `ClaudeSetup.exe` para garantir que o comando do próximo passo funcione sem ajuste.

---

## 3. Instalação via PowerShell (modo controlado)

Neste guia usamos `Start-Process` com o argumento `-Wait` em vez do `& ".\..."` simples. Isso garante que o PowerShell aguarde a extração completa antes de liberar o terminal — especialmente importante em instaladores do ecossistema Electron/Squirrel, que rodam extração em segundo plano.

1. Abra o **PowerShell como Administrador**
2. Navegue até o diretório do instalador:

   ```powershell
   cd "E:\software_install_windows\Claude"
   ```

3. Execute a instalação controlada:

   ```powershell
   Start-Process ".\ClaudeSetup.exe" -ArgumentList "install", "/D=E:\software_install_windows\Claude" -Wait
   ```

> 💡 **Por que `Start-Process` e não o `&` direto?** O instalador do Claude (Squirrel) lança um processo filho e retorna imediatamente, dando a falsa impressão de que terminou. O `-Wait` força o PowerShell a aguardar o processo filho real, evitando que você feche o terminal antes da extração terminar.

---

## 4. Limpeza Pós-Instalação

Como usamos `-Wait`, o PowerShell só libera a linha de comando após a extração concluída.

- Verifique se os arquivos e pastas do Claude aparecem em `E:\software_install_windows\Claude\`
- Apague o `ClaudeSetup.exe` da pasta para manter o diretório limpo — o programa roda direto dos arquivos extraídos
- Abra o Claude Desktop normalmente e confirme que ele inicia sem erros — o link simbólico é transparente para o app

---

## 🔄 Como Atualizar no Futuro

> 🔴 **Não atualize pelo aviso interno do aplicativo.** A atualização automática do Claude tenta reinstalar os arquivos no caminho padrão do `C:\AppData\Local`.

O link simbólico criado no Passo 1 **é permanente** — você não precisa recriá-lo a cada atualização. Apenas o executável precisa ser atualizado.

Para atualizar mantendo toda a estrutura intacta:

1. Baixe o novo instalador no site oficial
2. Coloque em `E:\software_install_windows\Claude` e renomeie para `ClaudeSetup.exe`
3. Feche o Claude Desktop completamente (bandeja do sistema → botão direito → Sair)
4. Abra o PowerShell como Administrador e repita o comando do [Passo 3](#3-instalação-via-powershell-modo-controlado)
5. O instalador sobrescreve a versão antiga com os novos arquivos — suas configurações e histórico permanecem intactos no `E:\`, protegidos pelo symlink
6. Apague o `ClaudeSetup.exe` após a conclusão

---

## ⚡ Referência Rápida

### CMD (Administrador) — criar o symlink

```cmd
mklink /d "C:\Users\Bruno P Goulart\AppData\Roaming\Claude" "E:\software_install_windows\ClaudeData"
```

### PowerShell (Administrador) — instalar ou atualizar

```powershell
cd "E:\software_install_windows\Claude"
Start-Process ".\ClaudeSetup.exe" -ArgumentList "install", "/D=E:\software_install_windows\Claude" -Wait
```

### Estrutura final no drive E:

```text
E:\
└── software_install_windows\
    ├── Claude\          ← Executável e arquivos do programa (via /D)
    └── ClaudeData\      ← Dados, cache, histórico e configurações (via symlink)
```

### O que fica no C: após a configuração

```text
C:\Users\...\AppData\Roaming\Claude   ← ponteiro simbólico (ícone de atalho, 0 KB real)
```

---

## 🐛 Troubleshooting

| Problema | Causa provável | Solução |
|---|---|---|
| `mklink` retorna "O arquivo já existe" | A pasta `Claude` ainda está no caminho original do `C:\` | Recorte (não copie) a pasta `C:\...\AppData\Roaming\Claude` para `E:\...\ClaudeData` antes de rodar o mklink |
| `mklink` não é reconhecido | Comando rodado no PowerShell em vez do CMD | `mklink` é nativo do **CMD**. Abra o Prompt de Comando (não o PowerShell) como Administrador |
| Instalador ignora a pasta e vai para `C:\` | `/D` com sintaxe incorreta ou `-Wait` ausente | Use exatamente o `Start-Process` do Passo 3, sem espaços extras ao redor do `=` |
| Claude abre mas não carrega histórico | Symlink foi criado mas a pasta real no `E:\` está vazia | Confirme que os dados foram movidos para `E:\software_install_windows\ClaudeData` **antes** de criar o symlink |
| Pasta `Claude` no Explorer não tem ícone de atalho | Symlink não foi criado com sucesso | Re-execute o `mklink` no CMD como Administrador e verifique a mensagem de retorno |
| Claude travado ao abrir após atualização | Instalador rodou sem `-Wait` e fechou antes de terminar | Aguarde alguns minutos ou feche e reabra o Claude — e use sempre o `-Wait` nas próximas atualizações |
| PowerShell nega permissão no `Start-Process` | Não executado como Administrador | Reabra o PowerShell com **Executar como Administrador** |

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