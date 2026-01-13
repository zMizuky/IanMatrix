+++
title = "Criando uma Engine em SDL3 — Parte 1"
description = "Um passo a passo de como estou criando uma engine do zero usando SDL3."
date = "2026-01-12"
[extra]
[extra.cover]
image = "images/criandoengine01.png"
alt = "Engine em SDL3"

[taxonomies]
tags = ["engine", "gamedev"]
+++

Minha intenção não é substituir engines consolidadas como **Unity** ou **Unreal** — até porque eu não tenho a experiência e o time por trás desses projetos. O objetivo aqui é aprender como essas ferramentas realmente funcionam por dentro. Para isso, eu tenho que começar do básico (construir uma base forte). Antes de começar, quero informar que alguns links úteis está no final da página, como documentação ou artigos.
# A ideia por trás da engine
Todo projeto precisa de um bom plano para ser bem executado. Então, vou dizer alguns de meus planos para esse projeto:
## Ideia do jogo
Antes de tudo eu precisei pensar em um jogo simples. No qual, a engine, vai dar total suporte para esse modelo de jogo. Vai ser um jogo que se passa, primeiramente, numa nave. Onde essa nave é constantemente atacada por asteroides, ou aliens, e o jogador (ou, no futuro, jogadores. **SPOILER**) precisa consertá-la.

Eu sugiro, caso for seguir o post, que crie uma ideia própria para ver se realmente você entendeu o conteúdo. Mas indico uma ideia simples, como a minha.
## Para que tipo de jogo vai ser essa **engine**?
Temos que definir um público-alvo
- 3D ou 2D?
- Top-down, plataforma?
- Windows, Linux, Mac ou Android?
- e etc

### 3D ou 2D?
Gostaria muito de fazer algo em 3D, porém é algo muito complexo no momento (quem sabe no futuro?). Então a escolha é 2D.

### Categoria?
Para esse estilo de jogo penso que **top-down**, que é uma câmera com visão superior, funcionará melhor.
![Minish Cap](https://zmizuky.github.io/IanMatrix/images/topdowngames.png)
*Jogos como MinishCap, Stardew Valley, Pokemon Fire Red e até mesmo Space Invaders são feitos em top-down, por exemplo.*

### Plataforma?
E a plataforma que escolhi foi **Windows** e **Linux** (pela compatibilidade do SDL com ambas). Nesse momento irei focar em criar o executável para Linux por ser meu sistema principal. Fique à vontade para utilizar o Windows ou MacOS.

### Linguagem de programação
E por último, mas não menos importante, a linguagem de programação. Eu escolhi **C/C++** por ser leve, muito utilizada (principalmente no mercado de jogos). Mas, recomendo o leitor dar uma olhada em **Rust**, que vem ganhando um espaço em desempenho e confiabilidade. *Ambos com a documentação no final da página.* Posteriormente pretendo incluir [*Lua*](https://www.lua.org/portugues.html) para configuração em tempo real (**mais spoilers!**).

# Por que SDL?
Existem várias bibliotecas para criar jogos disponíveis por aí. E, além disso, várias engines também, como citado anteriormente. Outra biblioteca que também é bastante utilizada é o [Raylib](https://www.raylib.com/). O motivo da escolha ser o **SDL** é por possuir mais recursos, ser mais complexo, estar a mais tempo no mercado ([foi lançado em 1998](https://pt.wikipedia.org/wiki/SDL_(biblioteca)), faz 28 anos no momento que estou escrevendo esse texto) e ao mesmo tempo ser leve.
Mas fique à vontade de seguir o post usando [Raylib](https://www.raylib.com/), [GLFW](https://www.glfw.org/), [SFML](https://www.sfml-dev.org/) ou até mesmo alguma engine.

# Hello SDL
Vamos começar a diversão! Partiremos para a nossa primeira janela em SDL. Primeiramente precisamos criar dois arquivos: main.cpp e Makefile.
```text
engine/
 ├─ main.cpp
 └─ Makefile
```


## Makefile
Não irei explicar como o Make funciona, ou como instalar. Mas ele serve para criar o executável do seu código de forma mais automatizada. Também não irei explicar como fazer isso em Windows, pois assumo que você esteja no Linux (se não, venha para o lado Linux). Meu Makefile está configurado da seguinte forma:
```Makefile
CXX = g++
CXXFLAGS = -Wall -Wextra -std=c++20
LDFLAGS = -lSDL3

TARGET = game # Aqui é o nome do executável
SRC = main.cpp # Aqui é o seu .cpp. Se criou uma pasta coloque também (/src/main.cpp, por exemplo)

all: $(TARGET)

$(TARGET): $(SRC)
	$(CXX) $(SRC) $(CXXFLAGS) -o $(TARGET) $(LDFLAGS)

run: $(TARGET)
	./$(TARGET)

clean:
	rm -f $(TARGET)

```
## Incluir o SDL3
Além do Make, que estou supondo que você já tenha instalado no seu computador, precisamos instalar o SDL3. É só seguir o passo-a-passo que está no [Github](https://github.com/libsdl-org/SDL/blob/main/INSTALL.md) deles de acordo com seu sistema. Geralmente as distros colocam no gerenciador de pacotes. No arch, por exemplo, só precisa fazer um [`pacman -S SDL3`](https://archlinux.org/packages/extra/x86_64/sdl3/).

Logo em seguida, com tudo instalado, devemos criar nosso `main.cpp` e incluir o SDL3.
```cpp
#include <SDL3/SDL.h>
```
## Iniciando uma janela em SDL3
Devemos iniciar algumas variáveis globais também. Por enquanto faremos tudo isso no `main.cpp`. Mas adianto que esse modo é insustentável com um projeto desse tamanho e vou voltar nisso depois.
```cpp
char WINDOW_TITLE[] = "Hello SDL";
int WINDOW_WIDTH = 800;
int WINDOWS_HEIGHT = 600;

SDL_Window* window;
SDL_Renderer* renderer;

bool done = false;
```
Algumas variáveis são auto-explicativas, como `WINDOW_TITLE`, `WINDOW_WIDTH` e `WINDOWS_HEIGHT`. Ajuste como queira. O `SDL_Window*` cria um ponteiro para uma janela em SDL. O `SDL_Renderer*` cria um ponteiro para o renderizador do SDL que vai ser útil para exibir os conteúdos posteriormente. O boolean `done` vai servir como identificador que o jogo está rodando.

Agora, a parte mais legal, vamos exibir uma janela e renderizar uma cor nela. Para isso, na `int main()`, fazemos o seguinte:
```cpp
if(!SDL_Init(SDL_INIT_VIDEO)){
    SDL_Log("Erro ao iniciar SDL: %s", SDL_GetError());
    return -1;
}
window = SDL_CreateWindow(
    WINDOW_TITLE,
    WINDOW_WIDTH,
    WINDOWS_HEIGHT,
    0
);

if(!window){
    SDL_Log("Erro ao criar o window: %s", SDL_GetError());
    return -1;
}

renderer = SDL_CreateRenderer(window, NULL);

if(!renderer){
    SDL_Log("Erro ao criar o renderer: %s", SDL_GetError());
    return -1;
}

while(!done){
    SDL_Event event;
    while(SDL_PollEvent(&event)){
        if(event.type == SDL_EVENT_QUIT){
            done = true;
        }

        SDL_SetRenderDrawColor(renderer, 20, 20, 20, 255);
        SDL_RenderClear(renderer);

        SDL_RenderPresent(renderer);
    }
}
    
SDL_DestroyRenderer(renderer);
SDL_DestroyWindow(window);
SDL_Quit();
```
Se você executar o jogo, compilando com o comando `make` e depois executando o `.\game`, já vai conseguir o resultado de uma janela. Mas vamos entender o que cada linha faz.

```cpp
if(!SDL_Init(SDL_INIT_VIDEO)){
    SDL_Log("Erro ao iniciar SDL: %s", SDL_GetError());
    return -1;
}
```
`SDL_Init()` serve para iniciar recursos do SDL. Posteriormente vamos usar SDL_INIT_AUDIO, por exemplo. Logo, ele dentro do if irá tentar executar e retornará `true` caso tenha tido sucesso. SDL_Log() é o sistema de logger do SDL.

`SDL_CreateWindow()` é auto-explicativo, cria uma janela. O último argumento, que foi passado como 0, é onde ficará as *flags* do SDL (coisas como tela cheia é inserido aqui). `SDL_CreateRenderer()` é auto-explicativo também, cria o *renderer*. E o argumento, passado como `NULL` no meu caso, é onde configuramos o driver de vídeo (*NULL* permite o SDL escolher). Os dois `if`*s* já foram explicado anteriormente e funcionam da mesma forma.

```cpp
while(!done){
    SDL_Event event;
    while(SDL_PollEvent(&event)){
        if(event.type == SDL_EVENT_QUIT){
            done = true;
        }

        SDL_SetRenderDrawColor(renderer, 20, 20, 20, 255);
        SDL_RenderClear(renderer);

        SDL_RenderPresent(renderer);
    }
}
```
Esse `while` é o Loop da engine, por enquanto. Aqui, enquanto estiver rodando o jogo, vai ser verificado se o usuário apertou alguma tecla ou interagiu com a janela. Além disso, é onde iremos passar comandos para o SDL renderizar o que queremos. Nesse caso, com `SDL_SetRenderDrawColor`, usando o sistema RGBA, coloquei a cor do `renderer` meio cinza (`rgba(20 ,20 ,20 ,255)`). E depois eu pedi para limpar a tela para ser preenchida com essa cor. Logo depois indicamos pro SDL que queremos usar o `renderer`.

O `if(event.type == SDL_EVENT_QUIT)` verifica se o evento executado pelo usuário foi de sair. Logo, ele altera o boolean `done` para `true`.

## Memória

```cpp
SDL_DestroyRenderer(renderer);
SDL_DestroyWindow(window);
SDL_Quit();
```
Precisamos limpar a memória destruindo todos os processos antes de finalizar. O seu sistema operacional provavelmente já vai fazer isso. Mas iremos fazer manualmente como boas práticas.

## Código final
Com tudo isso pronto temos a nossa primeira janela em **SDL**. Próximo passo será separar o código em arquivos separados e desenhar nosso primeiro personagem na tela. O código completo ficou dessa forma:
```cpp
#include <SDL3/SDL.h>

char WINDOW_TITLE[] = "Hello SDL";

int WINDOW_WIDTH = 800;
int WINDOWS_HEIGHT = 600;

SDL_Window* window;
SDL_Renderer* renderer;

bool done = false;

int main(){
    if(!SDL_Init(SDL_INIT_VIDEO)){
        SDL_Log("Erro ao iniciar SDL: %s", SDL_GetError());
        return -1;
    }

    window = SDL_CreateWindow(
        WINDOW_TITLE,
        WINDOW_WIDTH,
        WINDOWS_HEIGHT,
        0
    );

    if(!window){
        SDL_Log("Erro ao criar o window: %s", SDL_GetError());
        return -1;
    }

    renderer = SDL_CreateRenderer(window, NULL);

    if(!renderer){
        SDL_Log("Erro ao criar o renderer: %s", SDL_GetError());
        return -1;
    }

    while(!done){
        SDL_Event event;
        while(SDL_PollEvent(&event)){
            if(event.type == SDL_EVENT_QUIT){
                done = true;
            }

            SDL_SetRenderDrawColor(renderer, 20, 20, 20, 255);
            SDL_RenderClear(renderer);

            SDL_RenderPresent(renderer);
        }
    }
    
    SDL_DestroyRenderer(renderer);
    SDL_DestroyWindow(window);
    SDL_Quit();

    return 0;
}
```
Pode ser acessado também pelo [GitHub](https://github.com/zMizuky/iMatrix):
```text
https://github.com/zMizuky/iMatrix
```
Acompanhe para a parte 2!

# Links úteis

[Learn C++](https://www.learncpp.com/)

[Documentação do Rust](https://prev.rust-lang.org/pt-BR/documentation.html)

[Documentação do SDL3](https://wiki.libsdl.org/SDL3/FrontPage)
