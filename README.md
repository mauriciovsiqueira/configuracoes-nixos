# Configurações do NixOS e Linux em geral.
Configurações linux e mais com o NixOS.

## Instalação do NixOS.

*Para aprender a dominar o NixOS, siga este roteiro prático. Ele cobre desde a alteração de uma configuração simples até a atualização de versão do sistema.*

**Passo 1:** O Fluxo de Trabalho Básico (Configurar):
*No NixOS, você não instala coisas "soltas". Você as descreve no arquivo central.*

Abra o arquivo: sudo nano /etc/nixos/configuration.nix e copie esse aqui:
```json                                                        
 
```

**Passo 2:** Manutenção Semanal (Atualizar):
*Faça isso uma vez por semana para manter o sistema seguro e em dia.*

Sincronize e atualize tudo: 
```bash
sudo nixos-rebuild switch --upgrade
```

*O que isso faz: Baixa as definições novas do canal e já aplica no seu sistema.*

**Passo 3:** Testar sem Instalar (Temporário):
*Uma das melhores funções do NixOS para quem está aprendendo.*

Quer usar o python3 ou o htop só agora?:
```bash
nix-shell -p python3
```

*Saia da sessão, o programa não ocupa mais espaço no seu sistema "real".*

**Passo 4:** Limpeza de Disco (Faxina):
*Como o NixOS guarda versões antigas (gerações), o disco pode encher.*

Remova o que é velho: 
```bash
sudo nix-collect-garbage -d
```

*Dica: Só faça isso se o sistema atual estiver funcionando perfeitamente, pois isso apaga os pontos de restauração antigos.*

**Passo 5:** Troca de Versão (Upgrade de 6 meses)
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

**Passo 6:** O Botão de Pânico (Segurança):
*Se você fizer algo errado e o sistema não ligar ou o ambiente gráfico sumir:*

Reinicie o computador.

No menu inicial (Boot), escolha "NixOS - All configurations".

Selecione a versão de ontem ou a última que funcionava.

O sistema iniciará normalmente. 

*Para tornar essa versão a "oficial" de novo, basta rodar sudo nixos-rebuild switch enquanto estiver nela.*

Ação,Comando
Aplicar mudança na config,sudo nixos-rebuild switch
Atualização semanal,sudo nixos-rebuild switch --upgrade
Testar pacote rápido,nix-shell -p pacote
Limpar versões antigas,sudo nix-collect-garbage -d
Desfazer última mudança,sudo nixos-rebuild switch --rollback

Parabéns pela instalação! O NixOS recém-instalado é como uma tela em branco. Aqui está o roteiro do que você deve fazer para deixar o sistema pronto para o uso diário:

1. Primeira Atualização Geral
Mesmo que a ISO seja recente, os pacotes mudam rápido. Sincronize o sistema:

Bash

sudo nix-channel --update
sudo nixos-rebuild switch --upgrade


2. Configure o seu "DNA" (configuration.nix)
Abra o arquivo onde toda a mágica acontece:

Bash

sudo nano /etc/nixos/configuration.nix
Verifique e ajuste estes pontos essenciais:

Teclado: Procure por i18n e console. Para o padrão brasileiro (ABNT2):

Nix

i18n.defaultLocale = "pt_BR.UTF-8";
console.keyMap = "br-abnt2";
services.xserver.xkb.layout = "br";
Fuso Horário:

Nix

time.timeZone = "America/Sao_Paulo";
Drivers de Vídeo (Se tiver placa dedicada):

Nvidia: Adicione services.xserver.videoDrivers = [ "nvidia" ]; e hardware.opengl.enable = true;.

3. Adicione seus Programas
No mesmo arquivo, procure a lista environment.systemPackages. Descomente-a (remova o #) e adicione o que você usa:

Nix

environment.systemPackages = with pkgs; [
  firefox
  vscode
  git
  discord
  vlc
  htop
  wget
];
4. Habilite o "Não Livre" (Unfree)
Muitos programas (como Google Chrome, Discord, Drivers Nvidia) são proprietários. Para permitir que o NixOS os instale, adicione esta linha fora de qualquer bloco (geralmente no topo ou final do arquivo):

Nix

nixpkgs.config.allowUnfree = true;
5. Aplique as Mudanças
Após salvar o arquivo (Ctrl+O, Enter, Ctrl+X), rode o comando para o NixOS ler o arquivo e "se transformar":

Bash

sudo nixos-rebuild switch
6. Configure a Limpeza Automática
Para não ter que se preocupar com o disco enchendo com versões antigas, adicione isso ao seu configuration.nix:

Nix

nix.settings.auto-optimise-store = true;
nix.gc = {
  automatic = true;
  dates = "weekly";
  options = "--delete-older-than 30d";
};

Ação,Por que?
allowUnfree = true,"Para poder instalar Chrome, Spotify, Discord, etc."
Ajustar Teclado/Língua,Para não ficar com o sistema em inglês e teclas erradas.
Configurar nix.gc,Para o sistema se limpar sozinho toda semana.
nixos-rebuild switch,Para validar e ativar todas as mudanças acima.

Dica de Ouro: Guarde uma cópia do seu arquivo configuration.nix no seu Google Drive ou GitHub. Se você precisar formatar o PC um dia, basta colar esse arquivo em um NixOS novo e, em 5 minutos, o seu PC estará exatamente como era antes.

Gostaria que eu te ajudasse a encontrar o nome exato de algum programa específico no repositório do Nix?

Não, na maioria das vezes você não precisa reiniciar. Essa é uma das maiores vantagens do NixOS. O comando switch aplica as mudanças em tempo real sem interromper o que você está fazendo.

Onde NÃO precisa reiniciar:
Instalar/Remover Programas: Ao adicionar um app no configuration.nix e dar o switch, ele aparece no seu menu de aplicativos instantaneamente.

Configurar Serviços: Ativar um banco de dados, servidor web ou mudar configurações de rede. O NixOS reinicia apenas o serviço específico para você.

Usuários e Grupos: Criar novos usuários ou mudar permissões.

Onde É NECESSÁRIO reiniciar:
Existem poucas exceções onde o reboot é obrigatório para as mudanças entrarem em vigor:

Atualização de Kernel: Se você mudar a versão do Linux que o sistema usa.

Drivers de Vídeo: Mudanças profundas nos drivers da Nvidia ou AMD.

Bootloader: Alterações nas configurações do GRUB ou Systemd-boot.

Comando,O que ele faz?,Quando usar?
sudo nixos-rebuild switch,Aplica as mudanças agora e cria uma nova entrada no boot.,"Na maioria das vezes (instalar apps, mudar teclado)."
sudo nixos-rebuild boot,"Salva a configuração, mas ela só ativa quando você reiniciar.",Quando você atualiza o Kernel ou drivers e não quer mexer no sistema agora.
sudo nixos-rebuild test,"Aplica as mudanças agora, mas não cria uma entrada no boot.",Para testar uma configuração rápida sem poluir o menu de inicialização.

Ação,Comando
Aplicar mudanças,sudo nixos-rebuild switch
Atualizar e aplicar,sudo nixos-rebuild switch --upgrade
Testar sem instalar,nix-shell -p nome-do-pacote
Voltar atrás (Erro),sudo nixos-rebuild switch --rollback
Limpar lixo/espaço,sudo nix-collect-garbage -d
Editar configuração,sudo nano /etc/nixos/configuration.nix

3. Rotina do Usuário (Cronograma)
Toda Semana: Rode sudo nixos-rebuild switch --upgrade para manter segurança e apps em dia.

Uma vez por mês: Rode sudo nix-collect-garbage -d para não lotar o HD com versões antigas.

A cada 6 meses: Quando sair uma nova versão (ex: 25.11 para 26.05), mude o canal:

sudo nix-channel --add https://nixos.org/channels/nixos-26.05 nixos

sudo nix-channel --update

sudo nixos-rebuild switch --upgrade

4. Snippets Essenciais para o configuration.nix
Sempre que abrir o arquivo para editar, lembre-se destas configurações básicas:

Habilitar apps "não-livres" (Chrome, Discord, etc):

Nix

nixpkgs.config.allowUnfree = true;
Adicionar programas:

Nix

environment.systemPackages = with pkgs; [
  git
  vlc
  vscode
  # Adicione o que quiser aqui
];
Limpeza automática (Opcional, mas recomendado):

Nix

nix.gc = {
  automatic = true;
  dates = "weekly";
  options = "--delete-older-than 30d";
};
5. Regras de Ouro
Não reinicie à toa: Só é necessário para Kernel, Drivers de vídeo ou Bootloader. O switch cuida do resto.

Use o Rollback: Se algo quebrar, escolha a versão anterior no menu de boot. É o seu "ponto de restauração" garantido.

Use a Busca: Pesquise nomes de pacotes em search.nixos.org.

