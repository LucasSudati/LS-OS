# LS-OS

> Sistema Linux pessoal em desenvolvimento, baseado em Arch Linux e Hyprland.

## Estado do projeto

O LS-OS está atualmente na fase de construção e estabilização do ambiente gráfico. O sistema base inicializa normalmente e o Hyprland já funciona dentro da máquina virtual de desenvolvimento. Antes da personalização visual definitiva, o foco atual é eliminar problemas da pilha gráfica no VirtualBox.

### Status geral

| Componente | Estado |
|---|---|
| Arch Linux | ✅ Funcional |
| Boot e TTY | ✅ Funcional |
| Usuário `lucas` / hostname `LS-OS` | ✅ Configurado |
| Hyprland | ✅ Inicia |
| Configuração Lua do Hyprland | ✅ Reconhecida |
| Mesa / DRM | ✅ Disponível |
| VirtualBox Guest Additions | ✅ Ativas |
| Waybar | ✅ Instalada |
| Wofi | ✅ Instalado |
| Thunar | ✅ Instalado |
| Mako | ✅ Instalado |
| Foot | ✅ Terminal padrão e estável na VM |
| Clipboard Windows ↔ Wayland | ⚠️ Em diagnóstico |
| Wallpaper LS-OS | ✅ Aplicado com AWWW |
| Identidade visual LS-OS | 🚧 Em implementação |
| Configuração definitiva do desktop | ⏳ Próxima etapa |

## Plataforma atual de desenvolvimento

O sistema está sendo desenvolvido em uma VM do VirtualBox.

- Distribuição base: Arch Linux
- Kernel observado: `7.2.9-arch1-1`
- Usuário: `lucas`
- Hostname: `LS-OS`
- Compositor: Hyprland
- Protocolo gráfico: Wayland + XWayland
- Controlador gráfico da VM: VMSVGA
- VRAM: 128 MB
- Aceleração 3D: habilitada
- Guest Additions: VirtualBox 7.2.20
- Driver gráfico observado: `vmwgfx`
- Mesa observado: 26.2.4

> O VirtualBox é somente a plataforma atual de desenvolvimento/testes e não faz parte da arquitetura final do LS-OS.

## Arquitetura atual

```text
LS-OS
│
├── Arch Linux
│   ├── kernel
│   ├── systemd
│   ├── pacman
│   └── usuário lucas
│
├── Pilha gráfica
│   ├── DRM
│   ├── Mesa
│   ├── vmwgfx
│   ├── Wayland
│   └── XWayland
│
├── Desktop
│   └── Hyprland
│       └── ~/.config/hypr/hyprland.lua
│
├── Interface
│   ├── Waybar
│   ├── Wofi
│   ├── Mako
│   └── Nerd Fonts
│
├── Aplicativos
│   ├── Foot
│   ├── Thunar
│   └── Pavucontrol
│
├── Integração
│   ├── NetworkManager
│   ├── Blueman
│   ├── GVFS
│   ├── MTP
│   ├── wl-clipboard
│   └── Polkit
│
├── Ferramentas
│   ├── grim
│   ├── slurp
│   └── brightnessctl
│
└── Ambiente de desenvolvimento
    └── VirtualBox
        ├── VMSVGA
        ├── 128 MB VRAM
        ├── aceleração 3D
        ├── Guest Additions
        └── Shared Clipboard
```

## Hyprland

O Hyprland está instalado e pode ser iniciado a partir do TTY com:

```bash
start-hyprland
```

A configuração atual está em:

```text
/home/lucas/.config/hypr/hyprland.lua
```

Existe também um backup:

```text
/home/lucas/.config/hypr/hyprland.lua.backup
```

A instalação usa a configuração Lua do Hyprland. Já foram observadas chamadas como:

```lua
hl.env("XCURSOR_SIZE", "24")
hl.env("HYPRCURSOR_SIZE", "24")
```

e:

```lua
hl.animation({
    leaf = "windows",
    enabled = true,
    speed = 4.79,
    spring = "easy"
})
```

### Descoberta sobre a API Lua

Foi testado anteriormente `hl.setenv(...)`, mas essa função não existe na API disponível nesta instalação. Isso provocou o modo de emergência do Hyprland:

```text
Emergency mode tripped!
A lua config error resulted in no binds being registered
```

A configuração foi recuperada. Alterações futuras devem usar a API Lua efetivamente disponível, sem assumir equivalência direta com exemplos tradicionais de `hyprland.conf`.

## Pacotes instalados para o desktop

### Interface

- `foot`
- `waybar`
- `wofi`
- `mako`

### Arquivos e dispositivos

- `thunar`
- `thunar-volman`
- `tumbler`
- `gvfs`
- `gvfs-mtp`

### Sistema

- `network-manager-applet`
- `blueman`
- `pavucontrol`
- `brightnessctl`
- `polkit-gnome`

### Wayland e captura

- `grim`
- `slurp`
- `wl-clipboard`

### Fontes

- `noto-fonts`
- `noto-fonts-emoji`
- `ttf-jetbrains-mono-nerd`

### Pendente

O pacote `playerctl` não foi instalado durante a configuração inicial porque o `pacman` retornou:

```text
erro: alvo não encontrado: playerctl
```

Isso não é atualmente um bloqueio para o sistema.

## VirtualBox Guest Additions

Foram observados módulos como:

```text
vboxsf
vboxvideo
vboxguest
```

O serviço `vboxservice.service` foi verificado e estava ativo.

O sistema também apresenta:

```text
/dev/dri/card0
/dev/dri/renderD128
```

O adaptador detectado no Linux aparece como VMware SVGA II Adapter, usando `vmwgfx`, comportamento esperado com VMSVGA.

## Renderização

Os testes de OpenGL/EGL mostraram caminhos de renderização VMware/SVGA3D e também `llvmpipe`.

Como teste de renderização por software foi utilizado:

```bash
export LIBGL_ALWAYS_SOFTWARE=1
```

Em uma etapa anterior também foi usado o seguinte início:

```bash
#!/bin/bash
export LIBGL_ALWAYS_SOFTWARE=1
exec start-hyprland
```

A solução definitiva ainda não foi escolhida.

## Terminal funcional: Foot

O crash do Kitty foi isolado e deixou de ser um bloqueio para o desenvolvimento.

Com VMSVGA, 128 MB e aceleração 3D habilitada, abrir o Kitty provocava crash do processo `VirtualBoxVM.exe` no host. O mesmo comportamento ocorreu mesmo após testar `LIBGL_ALWAYS_SOFTWARE=1`, portanto esse caminho não resolveu a incompatibilidade.

O terminal **Foot** foi instalado e testado diretamente no Wayland. Ele abriu normalmente e permaneceu estável. A variável de terminal do Hyprland foi então alterada de:

```lua
local terminal = "kitty"
```

para:

```lua
local terminal = "foot"
```

Após reiniciar o Hyprland, os atalhos passaram a abrir o Foot sem provocar crash. O Foot é, portanto, o terminal padrão atual do LS-OS na VM de desenvolvimento.

O Kitty não faz parte do conjunto recomendado de aplicativos. O histórico de sua falha fica documentado aqui apenas para evitar reintroduzir a incompatibilidade.

## Wallpaper e AWWW

O wallpaper oficial do LS-OS está em uso no ambiente Hyprland. O `hyprpaper` foi testado, mas apresentou incompatibilidade com a superfície Wayland/GBM no VirtualBox, incluindo falha de segmentação.

Como alternativa foi adotado o **AWWW** (sucessor do SWWW), que reconheceu corretamente o monitor virtual e conseguiu exibir o wallpaper.

O monitor atual é:

```text
Virtual-1
```

O wallpaper está em:

```text
/home/lucas/Pictures/Wallpapers/wallpaper.png
```

O AWWW é iniciado pelo evento `hyprland.start` da configuração Lua. O wallpaper já carrega automaticamente ao iniciar o Hyprland.

Também foram definidos:

```lua
force_default_wallpaper = 0
disable_hyprland_logo = true
```

Isso remove o wallpaper/logo padrão. Ainda está em refinamento a transição visual inicial: o compositor inicia e faz seu fade antes de o AWWW apresentar o wallpaper do LS-OS.

## Clipboard compartilhado

O `wl-clipboard` está instalado, fornecendo:

```text
wl-copy
wl-paste
```

Também foi executado:

```bash
VBoxClient --clipboard
```

com saída indicando:

```text
Session type is: VBGHDISPLAYSERVERTYPE_XWAYLAND
Service: Shared Clipboard
```

O clipboard do host Windows e o clipboard Wayland ainda parecem operar como ambientes separados em algumas situações. A integração bidirecional permanece em diagnóstico.

## Etapas do desenvolvimento

### 1. Sistema base — praticamente concluído

Arch Linux instalado, boot funcional, usuário configurado e conjunto inicial de pacotes disponível.

### 2. Ambiente gráfico — em estabilização

Hyprland inicia e o problema de terminal foi contornado com o Foot. A etapa atual é a implementação e refinamento visual do desktop.

### 3. Desktop LS-OS — em desenvolvimento

A personalização do desktop já começou. O wallpaper oficial está aplicado e os próximos componentes serão refinados:

- identidade visual própria;
- configuração definitiva do Hyprland;
- Waybar;
- launcher;
- notificações;
- wallpaper;
- menus;
- atalhos;
- lock screen;
- tela de login;
- configurações e utilitários do sistema.

### 4. Distribuição — futura

Quando o desktop estiver estável, serão estudados:

- configuração reproduzível;
- scripts de instalação;
- pacotes próprios;
- instalação limpa;
- imagem/ISO;
- processo de atualização;
- distribuição do LS-OS.

## Escopo da limpeza experimental

A branch `dev/experimental` concentra a simplificação do projeto sem modificar a `main`.

- **Manter como base documentada:** Arch Linux, boot/TTY, Hyprland, Foot, Thunar, wallpaper via AWWW, Waybar e Wofi, conforme os testes registrados anteriormente. Waybar e Wofi estão instalados, mas suas funções ainda precisam ser verificadas na VM.
- **Não usar como padrão:** Kitty e hyprpaper, que apresentaram falhas no VirtualBox; preservar apenas o registro técnico dos problemas.
- **Ainda não validar como funcionais:** clipboard bidirecional Windows/Wayland, personalização definitiva do desktop e recursos ainda em desenvolvimento.
- **Ordem aprovada:** primeiro estabilizar e testar os componentes existentes, mantendo Waybar e Wofi por enquanto; só depois reconstruir barra, launcher e demais componentes próprios, um por vez.
- **Próxima verificação necessária:** conferir os arquivos de configuração e scripts efetivamente versionados antes de apagar código. Pacotes instalados dentro da VM não são removidos por alterações no GitHub.
- **Interface web:** qualquer terminal simulado no `index.html` deve ser tratado separadamente do terminal Foot real do Linux; a documentação do ambiente Linux não comprova seu funcionamento.

Este documento descreve resultados relatados dos testes anteriores, não uma nova validação executada na VM.

## Histórico de desenvolvimento

### 29/10/2026 — Primeira verificação (America/Sao_Paulo)

Foram identificados **dois commits de documentação** na branch `dev/experimental`, anteriores a esta verificação:

- [`f885aa5`](https://github.com/LucasSudati/LS-OS/commit/f885aa501bd05627f2a10a6e979e7ef57a223750) — reorganizado o registro do terminal Foot e delimitado o escopo da limpeza experimental; Kitty e hyprpaper permanecem documentados como incompatibilidades observadas, sem indicação de remoção de pacotes na VM.
- [`cef5159`](https://github.com/LucasSudati/LS-OS/commit/cef5159cce83b9302b0527bac076dbda5bda36bf) — mantidos Waybar e Wofi na base documentada, com validação funcional ainda pendente; priorizados testes e estabilização antes de substituir componentes por implementações próprias.

**Arquivos modificados nesses commits:** somente `README.md`. Não foram verificadas alterações de código ou novos testes na VM nesses commits.

**Referência para a próxima comparação (HEAD verificado antes deste registro):** `cef5159cce83b9302b0527bac076dbda5bda36bf`. Esta é a primeira verificação deste histórico; não existe um registro anterior de comparação automática nesta seção.

### 30/10/2026 — Verificação diária (America/Sao_Paulo)

**Sem atividade registrada.** A comparação entre o commit da última verificação [`52911de`](https://github.com/LucasSudati/LS-OS/commit/52911de9d9a1b999795c78e08189d03381bc8ea1) e o estado da branch `dev/experimental` em 30/10/2026 retornou **0 novos commits e 0 arquivos alterados**.

Não foram verificadas novas modificações de código, configuração ou documentação desde o registro anterior. A situação funcional da VM não foi reavaliada nesta consulta.

**Referência para a próxima comparação (HEAD antes deste registro):** `52911de9d9a1b999795c78e08189d03381bc8ea1`.

## Filosofia do repositório

Este README funciona como documentação técnica viva do LS-OS.

Ele deve ser atualizado conforme o projeto evolui, registrando:

- arquitetura;
- componentes;
- decisões técnicas;
- testes;
- problemas conhecidos;
- soluções adotadas;
- estado atual;
- próximos passos.

O objetivo é evitar vários documentos de versão contendo informações contraditórias e manter uma referência principal atualizada.

---

**LS-OS está em desenvolvimento ativo.**
