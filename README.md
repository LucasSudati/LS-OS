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
| Kitty | ⚠️ Instalado, mas dispara crash na VM |
| Clipboard Windows ↔ Wayland | ⚠️ Em diagnóstico |
| Identidade visual LS-OS | ⏳ Próxima etapa |
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
│   ├── Kitty
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

- `kitty`
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

## Problema atual: crash ao abrir o Kitty

Este é o principal bloqueio técnico atual.

Com a VM configurada com VMSVGA, 128 MB e aceleração 3D habilitada:

1. O Arch Linux inicia normalmente.
2. O login no TTY funciona.
3. `start-hyprland` abre o desktop.
4. O Hyprland permanece estável por vários minutos quando parado.
5. Ao pressionar `SUPER + Q` e abrir o Kitty, o processo da VM sofre crash no host.
6. No Windows é apresentado o erro de que a memória não pôde ser lida.

### Teste com 3D desabilitado

Com VMSVGA + 128 MB e aceleração 3D desabilitada, o Hyprland não iniciou corretamente.

Por enquanto, a aceleração 3D precisa permanecer habilitada.

### Hipótese atual

O comportamento aponta para um problema no caminho gráfico usado ao criar/renderizar uma nova janela:

```text
Kitty
  ↓
Wayland / OpenGL
  ↓
Mesa
  ↓
vmwgfx / VMSVGA
  ↓
VirtualBox 3D
  ↓
Windows host
  ↓
CRASH
```

Isso ainda é uma hipótese de diagnóstico, não uma causa confirmada.

### Próximo teste

No próximo teste será iniciado o sistema pelo TTY e executado:

```bash
export LIBGL_ALWAYS_SOFTWARE=1
start-hyprland
```

Depois será aberto o Kitty com `SUPER + Q`.

O objetivo é verificar se a renderização por software evita o crash ao criar a janela.

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

Hyprland inicia, mas o caminho gráfico VirtualBox/Kitty ainda precisa ser estabilizado ou contornado.

### 3. Desktop LS-OS — próximo estágio

Depois da estabilização serão desenvolvidos:

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
