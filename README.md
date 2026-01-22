# Configurações do NixOS e Linux em geral.
Configurações linux e mais com o NixOS.

---

*Para aprender a dominar o NixOS, siga este roteiro prático. Ele cobre desde a alteração de uma configuração simples até a atualização de versão do sistema.*

## Instalação do NixOS.
*Faça a instalação normal dele*

**Primeira Atualização Geral**
*Mesmo que a ISO seja recente, os pacotes mudam rápido. Sincronize o sistema:*
```bash
sudo nix-channel --update
sudo nixos-rebuild switch --upgrade
```

**O Fluxo de Trabalho Básico (Configurar):**
*No NixOS, você não instala coisas "soltas". Você as descreve no arquivo central.*

Abra o arquivo: sudo nano /etc/nixos/configuration.nix e copie esse aqui:  
*Obs.: Esse é minha configuração!!!*
```bash
# Edit this configuration file to define what should be installed on
# your system.  Help is available in the configuration.nix(5) man page
# and in the NixOS manual (accessible by running ‘nixos-help’).

{ config, pkgs, ... }:

{
  imports =
    [ # Include the results of the hardware scan.
      ./hardware-configuration.nix
    ];

  # Bootloader.
  boot.loader.systemd-boot.enable = true;
  boot.loader.efi.canTouchEfiVariables = true;

  networking.hostName = "nixos"; # Define your hostname.
  # networking.wireless.enable = true;  # Enables wireless support via wpa_supplicant.

  # Configure network proxy if necessary
  # networking.proxy.default = "http://user:password@proxy:port/";
  # networking.proxy.noProxy = "127.0.0.1,localhost,internal.domain";

  # Enable networking
  networking.networkmanager.enable = true;

  # Set your time zone.
  time.timeZone = "America/Belem";

  # Select internationalisation properties.
  i18n.defaultLocale = "pt_BR.UTF-8";

  i18n.extraLocaleSettings = {
    LC_ADDRESS = "pt_BR.UTF-8";
    LC_IDENTIFICATION = "pt_BR.UTF-8";
    LC_MEASUREMENT = "pt_BR.UTF-8";
    LC_MONETARY = "pt_BR.UTF-8";
    LC_NAME = "pt_BR.UTF-8";
    LC_NUMERIC = "pt_BR.UTF-8";
    LC_PAPER = "pt_BR.UTF-8";
    LC_TELEPHONE = "pt_BR.UTF-8";
    LC_TIME = "pt_BR.UTF-8";
  };

  # Enable the X11 windowing system.
  services.xserver.enable = true;
  
  # Configurações extras de XDG
  services.dbus.enable = true;
  
  # Enable the GNOME Desktop Environment.
  services.xserver.displayManager.gdm.enable = true;
  services.xserver.desktopManager.gnome.enable = true;

  # Configure keymap in X11
  services.xserver.xkb = {
    layout = "br";
    variant = "";
  };

  # Configure console keymap
  console.keyMap = "br-abnt2";
  
  # Bluetooth
  hardware.bluetooth = {
    enable = true;
    powerOnBoot = true;
  };

  services.blueman.enable = true;  

  # Enable CUPS to print documents.
  services.printing.enable = true;

  # Enable sound with pipewire.
  services.pulseaudio.enable = false;
  security.rtkit.enable = true;
  services.pipewire = {
    enable = true;
    alsa.enable = true;
    alsa.support32Bit = true;
    pulse.enable = true;
    # If you want to use JACK applications, uncomment this
    #jack.enable = true;

    # use the example session manager (no others are packaged yet so this is enabled by default,
    # no need to redefine it in your config for now)
    #media-session.enable = true;
  };

  # Enable touchpad support (enabled default in most desktopManager).
  # services.xserver.libinput.enable = true;

  # Define a user account. Don't forget to set a password with ‘passwd’.
  users.users.linux = {
    isNormalUser = true;
    description = "linux";
    extraGroups = [ "networkmanager" "wheel" ];
    packages = with pkgs; [
    #  thunderbird
    ];
  };

  # Zram
    zramSwap = {
      enable = true;
      algorithm = "zstd";
      memoryPercent = 60;
      priority = 100;
    };

  # Allow unfree packages
  nixpkgs.config.allowUnfree = true;

  # List packages installed in system profile. To search, run:
  # $ nix search wget
  environment.systemPackages = with pkgs; [
  #  vim # Do not forget to add an editor to edit configuration.nix! The Nano editor is also installed by default.
    xdg-user-dirs
    vim
    wget
    curl
    git
  # Ajustes gnome
    gnome-tweaks
  # Extensões
    gnome-extension-manager
  # Navegador
    google-chrome
  # Editor de código
    vscode
  # NodeJS
    nodejs_24
  # Nodemon
    nodemon  
  # Office
    libreoffice-still    
  ];

  # Some programs need SUID wrappers, can be configured further or are
  # started in user sessions.
  # programs.mtr.enable = true;
  # programs.gnupg.agent = {
  #   enable = true;
  #   enableSSHSupport = true;
  # };

  # List services that you want to enable:

  # Enable the OpenSSH daemon.
  # services.openssh.enable = true;

  # Open ports in the firewall.
  # networking.firewall.allowedTCPPorts = [ ... ];
  # networking.firewall.allowedUDPPorts = [ ... ];
  # Or disable the firewall altogether.
  # networking.firewall.enable = false;

  # This value determines the NixOS release from which the default
  # settings for stateful data, like file locations and database versions
  # on your system were taken. It‘s perfectly fine and recommended to leave
  # this value at the release version of the first install of this system.
  # Before changing this value read the documentation for this option
  # (e.g. man configuration.nix or on https://nixos.org/nixos/options.html).
  system.stateVersion = "25.11"; # Did you read the comment?

} 
```

Sincronize: 
```bash
sudo nixos-rebuild switch
```
*Obs.: Reinicie o pc, nesse caso vai ter que fazer essa primeira vez*

**Manutenção Semanal (Atualizar):**
*Faça isso uma vez por semana para manter o sistema seguro e em dia.*

Sincronize e atualize tudo: 
```bash
sudo nixos-rebuild switch --upgrade
```

*O que isso faz: Baixa as definições novas do canal e já aplica no seu sistema.*

**Habilite o "Não Livre" (Unfree).**
*Muitos programas (como Google Chrome, Discord, Drivers Nvidia) são proprietários. Para permitir que o NixOS os instale, adicione esta linha fora de qualquer bloco (geralmente no topo ou final do arquivo):*
nixpkgs.config.allowUnfree = true;

*Obs.: Ele já vai estar habilitado, na instalação normal você vai marcar ela.*

**Instalar Programas Temporários ou Permanentes:**
*Uma das melhores funções do NixOS para quem está aprendendo.*
*Caso não tenha feito na config.nix e queira só verificar se o programa está funcionando bem.*

Quer usar o vim ou o htop só agora:
```bash
nix-shell -p vim
```

*Saia da sessão, o programa não ocupa mais espaço no seu sistema "real".*

Quer instalar permanente no usuário local:
```bash
nix-env -iA nixos.vim
```

Na config.nix (ele fica para todos os usuários):

environment.systemPackages = with pkgs; [
  git
  vlc
  vscode
  
  *Adicione o que quiser aqui*
];

**Drivers de Vídeo (Se tiver placa dedicada).**
*Na config.nix:*
Adicione services.xserver.videoDrivers = [ "nvidia" ]; e hardware.opengl.enable = true;.

**Limpeza de Disco (Faxina):**
*Como o NixOS guarda versões antigas (gerações), o disco pode encher.*
*Caso não tenha colocado na config.nix*

Remova o que é velho: 
```bash
sudo nix-collect-garbage -d
```

*Dica: Só faça isso se o sistema atual estiver funcionando perfeitamente, pois isso apaga os pontos de restauração antigos.*

Configure a Limpeza Automática:
*Para não ter que se preocupar com o disco enchendo com versões antigas, adicione isso ao seu configuration.nix:*

nix.settings.auto-optimise-store = true;
nix.gc = {
  automatic = true;
  dates = "weekly";
  options = "--delete-older-than 30d";
};

**Troca de Versão (Upgrade de 6 meses).**
*Quando sair uma versão nova (ex: mudar da 25.11 para a 26.05).*

Mude o "trilho" (canal): 
```bash
sudo nix-channel --add https://nixos.org/channels/nixos-26.05 nixos
```

Atualize a lista: 
```bash
sudo nix-channel --update
```

Migre o sistema: 
```bash
sudo nixos-rebuild switch --upgrade
```

**O Botão de Pânico (Segurança):**
*Se você fizer algo errado e o sistema não ligar ou o ambiente gráfico sumir:*

Reinicie o computador.

No menu inicial (Boot), escolha "NixOS - All configurations".

Selecione a versão de ontem ou a última que funcionava.

O sistema iniciará normalmente. 

*Para tornar essa versão a "oficial" de novo, basta rodar sudo nixos-rebuild switch enquanto estiver nela.*

| Ação | Comando |
| :--- | :--- |
| **`Aplicar mudança na config`** | sudo nixos-rebuild switch |
| **`Atualização semanal`** | sudo nixos-rebuild switch --upgrade |
| **`Testar pacote rápido`** | nix-shell -p pacote |
| **`git commit -m "..."`** | Cria uma nova versão oficial do código com uma etiqueta descritiva. |
| **`Limpar versões antigas`** | sudo nix-collect-garbage -d |
| **`Desfazer última mudança`** | sudo nixos-rebuild switch --rollback |

*Parabéns pela instalação! O NixOS recém-instalado é como uma tela em branco. Aqui está o roteiro do que você deve fazer para deixar o sistema pronto para o uso diário:*

---











3. Rotina do Usuário (Cronograma)
Toda Semana: Rode sudo nixos-rebuild switch --upgrade para manter segurança e apps em dia.

Uma vez por mês: Rode sudo nix-collect-garbage -d para não lotar o HD com versões antigas.






**5. Regras de Ouro:**
Guarde uma cópia do seu arquivo configuration.nix no seu Google Drive ou GitHub. Se você precisar formatar o PC um dia, basta colar esse arquivo em um NixOS novo e, em 5 minutos, o seu PC estará exatamente como era antes.

Não reinicie à toa: Só é necessário para Kernel, Drivers de vídeo ou Bootloader. O switch cuida do resto.

Use o Rollback: Se algo quebrar, escolha a versão anterior no menu de boot. É o seu "ponto de restauração" garantido.

Use a Busca: Pesquise nomes de pacotes em search.nixos.org.

