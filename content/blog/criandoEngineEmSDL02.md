+++
title = "Criando uma Engine em SDL3 — Parte 2"
description = "Um passo a passo de como estou criando uma engine do zero usando SDL3."
date = 2026-01-13T12:00:00-03:00
[extra]
[extra.sitemap]
priority = "0.9"
[extra.cover]
image = "images/criandoengine01.png"
alt = "Engine em SDL3"

[taxonomies]
tags = ["engine", "gamedev"]
+++

# Sprites
Certo, paramos na criação da janela na parte 1. Agora, vamos renderizar o sprite do jogador!
## SDL_image
O SDL3 em si não tem um gerenciador de imagem. Se você for um lunático por low code, o que não é meu caso (eu acho), você vai querer fazer do zero. Não vai ser nosso caso. Vamos utilizar a biblioteca [SDL_image](https://github.com/libsdl-org/SDL_image). Outras alternativas para isso são:
- stbimage.h
- OpenCV
- CImg

Novamente, não vou ensinar como instalar por aqui. Tem tudo documentado no Github deles.

Mas, em Linux, geralmente é só instalar pelo gerenciador de pacotes. `pacman -S SDL_image`, no meu caso.

## Sprite do Player
Eu criei um pequeno *spritesheet* (uma imagem que contém todos os sprites, ótimo para animações e otimização). Vou disponibilizar aqui nesse post. Mas fique à vontade para criar seu próprio *spritesheet*. Recomendo ferramentas como [Paint.NET](https://www.getpaint.net/) ou [Aseprite](https://www.aseprite.org/) para criação de *pixel art*.
![spritesheet](../../images/character.png)
Você pode criar a seguinte estrutura na pasta:
```text
engine/
 └─ /assets
    └─ /sprites
       └─ character.png
 ├─ main.cpp
 └─ Makefile
```
## Makefile
Antes de dar continuação, devemos primeiro atualizar o arquivo **Makefile** para incluir o **SDL_image**. Precisa só atualizar a seguinte linha:
```Makefile
LDFLAGS = -lSDL3 -lSDL3_image
```
## Código
Começamos incluindo a biblioteca.
```cpp
#include <SDL3_image/SDL_image.h>
```
### Criando a textura
Precisamos definir uma textura que o SDL carregará na placa de vídeo. Iniciamos as variáveis globais:
```cpp
const char player_texture_path[] = "./assets/sprites/character.png"; // o spritesheet do player
SDL_Texture* player_texture;
```
Na `main`, antes do `loop`:
```cpp
player_texture = IMG_LoadTexture(renderer, player_texture_path);
```
Esse comando irá carregar o arquivo de imagem, que definimos o local anteriormente, na textura. Ficará armazenada na VRAM (memória da placa de vídeo), que torna o acesso mais rápido para renderizar. Logo, é importante limpar após a execução. Insira antes de destruir o `renderer` o seguinte comando:
```cpp
SDL_DestroyTexture(player_texture);
```
Agora, após o `SDL_RenderClear(renderer);`, vamos renderizar o sprite da seguinte forma:
```cpp
SDL_RenderTexture(renderer, player_texture, NULL, NULL);
```
Precisamos somente dizer onde vai ser renderizado, passar a textura. O terceiro argumento define qual parte da imagem será desenhada (posição e tamanho dentro do spritesheet). O quarto define onde e com qual tamanho ela será desenhada na tela. Ambos inseri `NULL`, que vai renderizar todo o arquivo e na tela toda.
Executando o código obtemos o seguinte resultado:
![exemplo01](../../images/exemploengine01.png)
Talvez o seu sprite não tenha ficado com a mesma definição. Isso ocorre por conta do filtro padrão que está configurado no SDL. Para resolver esse problema é só inserir esse comando depois de carregar o arquivo na textura:
```cpp
SDL_SetTextureScaleMode(player_texture, SDL_SCALEMODE_NEAREST);
```
O padrão é `SDL_SCALEMODE_LINEAR`.

A imagem inteira provavelmente ainda não é o que você deseja. Vamos consertar isso iniciando duas variáveis acima da renderização da textura.
```cpp
SDL_FRect player_frame = {
    0,      //inicio, x
    0,      //inicio, y
    16,     //tamanho em x
    32      //tamanho em y
};
SDL_FRect player_pos = {
    250, 
    250, 
    16, 
    32
};
```
`SDL_FRect` é um tipo do SDL para definição de retângulos. Usamos nesse caso, no `player_frame`, para definir de onde o sprite é recortado dentro do **arquivo** (spritesheet). A mesma ideia segue no player_pos. O sprite será renderizado na posição 250, 250 da janela e com um tamanho de 16x32. Passando essas definições para renderizar:
```cpp
SDL_RenderTexture(renderer, player_texture, &player_frame, &player_pos);
```
Obtemos o seguinte resultado:
![exemplo02](../../images/exemploengine02.png)
## Movimento
Podemos fazer um movimento simples criando uma struct global para o player (mais tarde irei organizar isso melhor):
```c++
struct Player {
    float x = 250;
    float y = 250;
};Player player;
```
Logo em seguida atualizando o player_pos:
```c++
SDL_FRect player_pos = {
    player.x, 
    player.y, 
    16, 
    32
};
```
Com isto temos o poder de alterar a posição do sprite por meio do player.x e player.y. Então, o que queremos agora é detectar quando a tecla é pressionada para mover o jogador. No loop, após a verificação se a janela foi fechada, inserimos o seguinte código:
```c++
if (event.type == SDL_EVENT_KEY_DOWN) {
    if (event.key.key == SDLK_A) player.x -= 5;
    if (event.key.key == SDLK_D) player.x += 5;
    if (event.key.key == SDLK_W) player.y -= 5;
    if (event.key.key == SDLK_S) player.y += 5;
}
```
Esse if verifica se foi pressionado alguma tecla (SDL_EVENT_KEY_DOWN). E executa uma comparação com o código da tecla. Nesse caso eu escolhi as teclas W, A, S, D para o movimento. [Aqui está uma lista de todos os códigos](https://wiki.libsdl.org/SDL3/SDL_Keycode).
## Código final
```c++
#include <SDL3/SDL.h>
#include <SDL3_image/SDL_image.h>

char WINDOW_TITLE[] = "Hello SDL";

int WINDOW_WIDTH = 800;
int WINDOWS_HEIGHT = 600;

SDL_Window* window;
SDL_Renderer* renderer;

struct Player {
    float x = 250;
    float y = 250;
};Player player;

const char player_texture_path[] = "./assets/sprites/character.png"; // o spritesheet do player
SDL_Texture* player_texture;

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
    
    player_texture = IMG_LoadTexture(renderer, player_texture_path);
    SDL_SetTextureScaleMode(player_texture, SDL_SCALEMODE_NEAREST);

    while(!done){
        SDL_Event event;
        while(SDL_PollEvent(&event)){
            if(event.type == SDL_EVENT_QUIT){
                done = true;
            }
            if (event.type == SDL_EVENT_KEY_DOWN) {
                if (event.key.key == SDLK_A) player.x -= 5;
                if (event.key.key == SDLK_D) player.x += 5;
                if (event.key.key == SDLK_W) player.y -= 5;
                if (event.key.key == SDLK_S) player.y += 5;
            }

            SDL_SetRenderDrawColor(renderer, 20, 20, 20, 255);
            SDL_RenderClear(renderer);
            
            SDL_FRect player_frame = {
                0,      //inicio, x
                0,      //inicio, y
                16,     //tamanho em x
                32      //tamanho em y
            };
            SDL_FRect player_pos = {
                player.x,
                player.y, 
                16, 
                32
            };

            SDL_RenderTexture(renderer, player_texture, &player_frame, &player_pos);
            
            SDL_RenderPresent(renderer);
        }
    }
    
    SDL_DestroyTexture(player_texture);
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
Acompanhe para a parte 3!

# Links úteis

[SDL_image](https://github.com/libsdl-org/SDL_image)

[SDL keycodes](https://wiki.libsdl.org/SDL3/SDL_Keycode)
