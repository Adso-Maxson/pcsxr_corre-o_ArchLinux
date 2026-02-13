# pcsxr_corre-o_ArchLinux
Guia de Compilação: PCSX-Reloaded em Sistemas Modernos (GCC 10+)  Este documento resume as correções manuais e as flags de compilação necessárias para contornar erros de linkagem, cabeçalhos ausentes e rigor de tipos no GTK3.

Link do PCSXR Original: https://ps1emulator.com/download

PS: Se não quiser realizar a correção manual, pode apenas baixar extrair e rodar "make" e "make install" no "pcsxr-1.9.93_corrigido_ArchLinux" que já é o pacote corrigido.

1. Dependências Necessárias

Antes de compilar, garanta que as bibliotecas de desenvolvimento estejam instaladas:

    build-essential, libgtk-3-dev, libsdl1.2-dev, libxtst-dev, libavcodec-dev, libavformat-dev.

2. Alterações no Código-Fonte
A. Inclusão de Cabeçalhos de Sistema (Erro waitpid)

O compilador moderno exige a declaração explícita da biblioteca para gerenciamento de processos.

    Arquivos: 1.  plugins/dfinput/pad.c
    2.  plugins/dfsound/cfg.c

    Correção: Adicionar no topo do arquivo:
    C

    #include <sys/wait.h>

B. Correção de Protótipo de Função (Erro OpenCdHandle)

Divergência entre a declaração no cabeçalho e a implementação no código Linux.

    Arquivo: plugins/dfcdrom/cdr.h (Linha 230)

    Correção: Alterar de int OpenCdHandle(); para:
    C

    int OpenCdHandle(const char *device);

C. Ajuste de Tipos GTK3 (Erro Incompatible Pointer Types)

O GTK3 moderno exige "casts" explícitos ao recuperar objetos do GtkBuilder.

    Arquivos:

        plugins/dfsound/spucfg-0.1df/main.c

        plugins/dfxvideo/gpucfg-0.1df/main.c

    Correção: Envolver as chamadas de gtk_builder_get_object com as macros de tipo correspondentes.

        Exemplo: GTK_WIDGET(gtk_builder_get_object(builder, "CfgWnd"))

        Exemplo: GTK_TOGGLE_BUTTON(...)

        Exemplo: GTK_ENTRY(...)

    Nota: Se houver muitos erros deste tipo, pode-se usar a flag de compilação -Wno-incompatible-pointer-types para ignorá-los.

3. Comandos de Compilação

Para contornar o comportamento padrão do GCC moderno (como o -fno-common), utilize o fluxo abaixo:
Passo 1: Configuração

O uso da flag --enable-interpreter é altamente recomendado para evitar falhas de segmentação (Core Dump) no motor de 64 bits.
Bash

CFLAGS="-O2 -fcommon -Wno-incompatible-pointer-types -Wno-implicit-function-declaration" ./configure --enable-interpreter

Passo 2: Compilação e Instalação
Bash

make
sudo make install
sudo ldconfig

4. Solução de Problemas Pós-Instalação
Sintoma	Causa	Solução
"interface could not be loaded"	Arquivo pcsxr.ui não encontrado.	Execute sudo make install para copiar os arquivos para /usr/local/share/pcsxr.
Janela fecha ao abrir ISO	Falha no Recompilador Dinâmico (Dynarec).	Vá em Configurações -> CPU e altere para Interpreter.
Plugins não aparecem	Cache de bibliotecas desatualizado.	Execute sudo ldconfig no terminal.
