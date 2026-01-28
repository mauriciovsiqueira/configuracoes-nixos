# Configurações do NixOS.
Configurações do NixOS e programas linux.

---

*Para aprender a dominar o NixOS, siga este roteiro prático. Ele cobre desde a alteração de uma configuração simples até a atualização de versão do sistema.*

## Configuração do NixOS.
*Depois de ter feito a instalação do NixOS.*

### Primeira Atualização Geral.  
*Mesmo que a ISO seja recente, os pacotes mudam rápido. Sincronize o sistema:*
```bash
sudo nix-channel --update
sudo nixos-rebuild switch --upgrade
```

---

### O Fluxo de Trabalho Básico (Configurar).  
*No NixOS, você não instala coisas "soltas". Você as descreve no arquivo central.*


**Abra o arquivo:**
```bash
sudo nano /etc/nixos/configuration.nix
```

*Obs.: Faça sua modificação, caso queira usar o meu como base, pode baixar e usar, mas não esqueça essa é minha configuração e você tem que adaptar para seu uso.*  

*Obs.: Não esqueça de mudar o usuário, para não sobreescrever o seu.*  

*Onde fica:*
```bash
# Define a user account. Don't forget to set a password with ‘passwd’.
  users.users.linux = {
    isNormalUser = true;
    description = "linux";
    extraGroups = [ "networkmanager" "wheel" ];
    packages = with pkgs; [
    #  thunderbird
    ];
  };
```

---

### Sincronize (faça isso sempre que modificar o config.nix): 
```bash
sudo nixos-rebuild switch
```
*Obs.: Reinicie o pc, nesse caso vai ter que fazer essa primeira vez, pois está fazendo depois da instalação e depois não precisa reiniciar de novo quando modificar o arquivo novamente.*

---

### Habilite a Zram:  
```bash
    zramSwap = {
      enable = true;
      algorithm = "zstd";
      memoryPercent = 60;
      priority = 100;
    };
```

*Obs.: Para computadores mais fracos e se tiver o swap não tem problema.*

---

### Manutenção Semanal (Atualizar).  
*Faça isso uma vez por semana para manter o sistema seguro e em dia.*

**Sincronize e atualize tudo:** 
```bash
sudo nixos-rebuild switch --upgrade
```

*O que isso faz: Baixa as definições novas do canal e já aplica no seu sistema.*

---

**Atualização Automática do Sistema.**
```bash
system.autoUpgrade = {
    enable = true;
    allowReboot = false; # Mantenha false para o PC não reiniciar sozinho
    dates = "weekly";     # Frequência (pode ser "daily" ou um horário como "03:00")
    flags = [
      "--upgrade"
    ];
  };
```
*Obs.: Coloque o coletor de lixo automático para não aculmular no disco.*  

---

### Limpeza de Disco (Coletor de lixo).  
*Como o NixOS guarda versões antigas (gerações), o disco pode encher.*  
*Caso não tenha colocado na config.nix*  

Remova o que é velho: 
```bash
sudo nix-collect-garbage -d
```

*Dica: Só faça isso se o sistema atual estiver funcionando perfeitamente, pois isso apaga os pontos de restauração antigos.*

**Configure a Limpeza Automática:**
*Para não ter que se preocupar com o disco enchendo com versões antigas, adicione isso ao seu configuration.nix:*
```bash
nix.gc = {
    automatic = true;
    dates = "weekly";
    options = "--delete-older-than 30d"; # Apaga o que tiver mais de 30 dias
  };

  # Otimiza o armazenamento (remove ficheiros duplicados)
  nix.settings.auto-optimise-store = true;
```

---

### Habilite o "Não Livre" (Unfree).  
*Muitos programas (como Google Chrome, Discord, Drivers Nvidia) são proprietários. Para permitir que o NixOS os instale, adicione esta linha fora de qualquer bloco (geralmente no topo ou final do arquivo). 
```bash
nixpkgs.config.allowUnfree = true;
```

*Obs.: Ele já vai estar habilitado, na instalação normal você vai marcar ela.*

---

### Instalar Programas Temporários ou Permanentes.  
*Uma das melhores funções do NixOS para quem está aprendendo.*
*Caso não tenha feito na config.nix e queira só verificar se o programa está funcionando bem.*

Quer usar o vim ou o htop só agora (temporário):
```bash
nix-shell -p vim
```

*Saia da sessão, o programa não ocupa mais espaço no seu sistema "real".*

**Quer instalar permanente no usuário local:**
```bash
nix-env -iA nixos.vim
```

**Na config.nix (ele fica para todos os usuários):**
```bash
environment.systemPackages = with pkgs; [  
  git  
  vlc  
  vscode  
  Adicione qualquer programa aqui (veja o nome correto no site: search.nixos.org)
];  
```

---

### Drivers de Vídeo (Se tiver placa dedicada).  
*Na config.nix adicione:*  
```bash
services.xserver.videoDrivers = [ "nvidia" ];
hardware.opengl.enable = true;
```

---

### Adicionar Usuário.  
*Na config.nix adicione e troque usuario pelo nome:*  
```bash
users.users.usuario = {
  # Se é usuário normal
  isNormalUser = true;
  # Define a senha de forma direta e fixa
  password = "senha_escolhida_aqui"; 
  
  # Grupos básicos para uso normal (internet e multimídia)
  # SEM o grupo "wheel", ele não consegue quebrar o sistema
  extraGroups = [ "networkmanager" "video" "audio" "lp" ]; # lp é para impressoras

  # Para pacotes para o usuário em expecifico
  packages = with pkgs; [
    firefox
    libreoffice-fresh
    vlc
    telegram-desktop
    spotify
  ];
};
```

*Obs.: Nesse caso é para senha fixa, caso queira colocar a senha no início e usuário trocar (esse é o recomendado):*
```bash
# Esta senha serve apenas para o primeiro acesso
  initialPassword = "senha-temporaria-123";
```

*O que o usuário deve fazer:*  
*Assim que o usuário logar pela primeira vez com a senha que você definiu, ele deve abrir o terminal e digitar:*  
```bash
passwd
```
*Digitar a senha atual: senha-temporaria-123.*

*Digitar a nova senha pessoal dele.*

---

### Troca de Versão (Upgrade de 6 meses).
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
---

### O Botão de Pânico (Segurança).  
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
| **`Instalar pacote permanente no usuário local`** | nix-env -iA nixos.vim |
| **`Limpar versões antigas`** | sudo nix-collect-garbage -d |
| **`Desfazer última mudança`** | sudo nixos-rebuild switch --rollback |

*Parabéns pela instalação! O NixOS recém-instalado é como uma tela em branco. Aqui está o roteiro do que você deve fazer para deixar o sistema pronto para o uso diário:*

---

### Observaçôes finais.

**Rotina do Usuário (Cronograma).**  
Toda Semana: Rode sudo nixos-rebuild switch --upgrade para manter segurança e apps em dia.  
Uma vez por mês: Rode sudo nix-collect-garbage -d para não lotar o HD com versões antigas. 
sempre que mudar o config.nix: sudo nixos-rebuild switch para aplicar as mudanças.  

**Regras de Ouro.**  
Guarde uma cópia do seu arquivo configuration.nix no seu Google Drive ou GitHub. Se você precisar formatar o PC um dia, basta colar esse arquivo em um NixOS novo e, em 5 minutos, o seu PC estará exatamente como era antes.  
Não reinicie à toa: Só é necessário para Kernel, Drivers de vídeo ou Bootloader. O switch cuida do resto.  

Use o Rollback se algo quebrar, escolha a versão anterior no menu de boot. É o seu "ponto de restauração" garantido.  
Use a Busca: Pesquise nomes de pacotes em search.nixos.org.  


