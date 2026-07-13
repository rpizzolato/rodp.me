---
title: "WSL: Recursos do Linux no seu Windows"
date: 2026-07-12
draft: false
tags: ["wsl", "linux", "windows"]
categories: ["infrastructure"]
description: "O que é o WSL, como instalar o WSL2, gerenciar distribuições, entender os dois sistemas de arquivos, executar comandos cruzados e limitar recursos com o .wslconfig."
---

> 🎥 Este artigo acompanha um vídeo do meu [canal no YouTube](https://www.youtube.com/@ropizzolato). O link do vídeo será adicionado aqui assim que for publicado.
<!-- Quando publicar o vídeo, substitua a linha acima por: {{</* youtube VIDEO_ID */>}} -->

O **WSL (Windows Subsystem for Linux)** é um módulo para Windows 10 e 11 que permite usar recursos reais do Linux diretamente do Windows, de forma oficial. Não é novidade, mas continua sendo uma das formas mais práticas de estudar e trabalhar com Linux — sem dual-boot e sem precisar configurar um virtualizador. Se você é estudante de TI começando agora, essa é provavelmente a porta de entrada mais simples.

## O que é o WSL?

O WSL é uma camada de compatibilidade que roda **Linux de verdade** dentro do Windows. Não é emulação tradicional nem uma máquina virtual clássica — embora, por trás dos panos, o WSL2 use virtualização leve via Hyper-V.

Atualmente o WSL está na **versão 2**. A versão 1, mais simples e menos compatível, está praticamente obsoleta — evite usá-la. As vantagens do WSL2:

- Kernel Linux real
- Compatibilidade quase total
- Melhor performance em muitos cenários
- Virtualização leve (via Hyper-V)

## O que dá para fazer com WSL?

Muita coisa: usar `bash`, `ssh`, `grep` e outros comandos; gerenciadores de pacotes como `apt` e `yum`; rodar Python, Node, Go, Docker e Git; compilar código; automatizar tarefas; acessar servidores Linux. Para muitos profissionais, o WSL substituiu completamente o dual-boot.

Um diferencial gigantesco é a **integração com o Windows**: você acessa o `C:\` do seu sistema pelo ponto de montagem `/mnt/c`, executa comandos Linux no PowerShell (e comandos Windows dentro do Linux), e ainda integra com o VS Code.

### Limitações e cuidados

- O WSL **não substitui um servidor Linux em produção** — a performance de I/O pode variar bastante.
- Atenção à segurança: trate arquivos sensíveis com cuidado.
- Nem todo cenário corporativo permite o WSL.

Onde recomendo: ambientes de estudo, desenvolvimento, automação de tarefas e ambientes híbridos — quando você vive nativamente no Windows, mas ainda precisa de Linux.

## Instalando o WSL2

A documentação oficial está em [learn.microsoft.com/windows/wsl/install](https://learn.microsoft.com/en-us/windows/wsl/install). Pré-requisitos: Windows 10 versão 2004+ (build 19041+) ou Windows 11 — o mais comum hoje, já que o suporte ao Windows 10 terminou em outubro de 2025.

Abra o **PowerShell como administrador** e execute:

```powershell
wsl --install
```

Aguarde a conclusão e verifique a versão instalada:

```powershell
wsl --version
```

## Gerenciando distribuições

Liste o que está instalado e o que está disponível online:

```powershell
wsl --list --verbose
wsl --list --online
```

Vou instalar o Ubuntu 24.04 (versão LTS, com suporte estendido) e também o Fedora 43:

```powershell
wsl --install -d Ubuntu-24.04
wsl --install -d FedoraLinux-43
```

Após cada instalação, defina usuário e senha. Para confirmar que você está num Linux de verdade:

```bash
cat /etc/os-release
```

Para sair do WSL e voltar ao PowerShell, digite `exit`.

Outros comandos úteis:

```powershell
wsl --list                       # lista as distros instaladas
wsl --unregister <NomeDaDistro>  # remove uma distro
wsl --setdefault Ubuntu-24.04    # define a distro padrão (ou -s)
wsl -u root                      # acessa como root (útil para redefinir senha com passwd)
wsl --help                       # todas as opções
```

Depois de autenticar na sua distro, o primeiro comando que sugiro é atualizar o sistema:

```bash
sudo apt update && sudo apt upgrade
```

O `&&` encadeia comandos: o segundo só roda se o primeiro tiver sucesso (código de retorno 0). Você pode conferir o código de retorno do último comando com `echo $?` — rode um `ls` num caminho inexistente e veja o retorno mudar para 2. O `man ls` mostra o *Exit Status* na documentação.

## Os dois sistemas de arquivos

Este é o conceito mais importante para usar bem o WSL:

| Caminho | Sistema de arquivos | O que é |
| --- | --- | --- |
| `/mnt/c/...` | NTFS (Windows) | O seu `C:\`, montado dentro do Linux |
| `/home/<usuario>/...` | ext4 (Linux) | Disco virtual gerenciado pelo WSL |
| `\\wsl$\<Distro>\...` | ext4 (Linux) | Ponte para o Windows acessar o filesystem Linux |

O caminho `C:\Users\<Usuario>\Projeto` equivale a `/mnt/c/Users/<Usuario>/Projeto` — são os mesmos arquivos. Já `/home/<usuario>` vive num disco virtual ext4 que não fica diretamente no NTFS.

Teste você mesmo: dentro do WSL, rode `cd /mnt/c/`, crie uma pasta com `mkdir` e um arquivo com `touch`, e confira no Explorador de Arquivos do Windows que eles estão lá.

**Dica importante:** para ambientes de desenvolvimento, evite trabalhar em `/mnt/c` — crie seus projetos na `/home` do seu usuário, onde a performance é muito melhor.

## Integração com o Windows e VS Code

De dentro da sua `/home`, você pode chamar programas do Windows:

```bash
explorer.exe .   # abre o Explorador de Arquivos na pasta atual
code .           # abre o VS Code na pasta atual (via extensão WSL)
```

O ponto final indica a pasta corrente (use `pwd` para saber onde está). No VS Code, instale a extensão oficial **WSL** da Microsoft se ela não abrir automaticamente no modo remoto.

Vamos testar algo que só funcionaria no Linux. Crie uma pasta `wsl`, e dentro dela um arquivo `hello.sh`:

```bash
#!/bin/bash
echo "Hello from Linux via WSL"
date
uname -a
```

Torne-o executável e rode:

```bash
chmod +x hello.sh
./hello.sh
```

O `uname -a` mostra as informações do sistema — incluindo o kernel Linux real que está rodando dentro do Windows.

## Comandos cruzados

Dá para misturar os dois mundos. No **PowerShell**, executando um comando Linux:

```powershell
ipconfig | wsl grep -i ipv4
```

O `ipconfig` é nativo do Windows; o `grep` filtra a saída no Linux (o `-i` ignora maiúsculas/minúsculas). Também funciona um simples `wsl ls` no lugar do `dir`.

O contrário também é possível — comandos Windows **dentro do WSL** (não esqueça o `.exe`):

```bash
notepad.exe .wslconfig
ls -la | findstr.exe "termo_da_pesquisa"
```

Lembre-se: para rodar uma ferramenta Linux via PowerShell, ela precisa estar instalada na distro. No Ubuntu/Debian, verifique com `dpkg -l | grep <pacote>` e instale com `apt install`. No Fedora/Red Hat, use `rpm -qa | grep <pacote>` e `yum install`. Exemplo clássico: o `ifconfig` (legado, substituído pelo comando `ip`) exige o pacote `net-tools`:

```bash
sudo apt install net-tools -y    # Ubuntu/Debian
sudo yum install net-tools -y    # Fedora/Red Hat
```

O `-y` responde "sim" automaticamente à confirmação do gerenciador de pacotes. Vale citar que o `apt` e o `yum` resolvem dependências automaticamente — diferente do `dpkg` e do `rpm` puros.

## Limitando recursos com o .wslconfig

O WSL adora consumir recursos — parece até o Chrome devorando RAM. Sem limites, ele consome até o sistema precisar fazer cache em disco, ficando lento. A solução é o arquivo `.wslconfig`, que fica em `%UserProfile%\.wslconfig` (ou seja, `C:\Users\<seu_usuario>\.wslconfig`):

```ini
[wsl2]
memory=4GB
processors=2
swap=2GB
swapFile=C:\\wsl\\swap.vhdx
localhostForwarding=true
```

- `memory`, `processors` e `swap` limitam o consumo global do WSL2.
- `swapFile` define onde fica o arquivo de swap (a barra invertida é dobrada porque serve como caractere de escape). Sem essa linha, o padrão é `%UserProfile%\AppData\Local\Temp\swap.vhdx`.
- `localhostForwarding=true` é ótimo para expor aplicações — por exemplo, `python3 -m http.server 8000` no WSL e acessar `localhost:8000` no navegador do Windows.

A [lista completa de configurações está na documentação oficial](https://learn.microsoft.com/en-us/windows/wsl/wsl-config). Também é possível limitar recursos **por distro**, mas isso fica para outro artigo.

## Encerrando o WSL

Uma boa prática é desligar o WSL quando terminar de usar, para liberar memória e CPU:

```powershell
wsl --shutdown
```

Em casos raros o processo insiste em continuar rodando. Se acontecer, encerre à força:

```powershell
taskkill /f /im wslservice.exe
```

O `/f` força o encerramento e o `/im` especifica o nome do processo (dá para usar `/pid` também).

## Conclusão

O WSL tem muitos outros recursos — este artigo é a base para os próximos conteúdos que vou trazer sobre WSL, Linux e administração de sistemas. Se ficou com alguma dúvida ou quer sugerir um tema, me encontre no [YouTube](https://www.youtube.com/@ropizzolato), [GitHub](https://github.com/rpizzolato) ou [LinkedIn](https://linkedin.com/in/rpizzolato).
