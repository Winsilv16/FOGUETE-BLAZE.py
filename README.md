# ESCAPE-FROM-IF-TEACHERS.pygame
Trabalho de programação

import pygame
import random
import math

# Inicializa o Pygame
pygame.init()

# Configurações da tela
LARGURA, ALTURA = 800, 460
tela = pygame.display.set_mode((LARGURA, ALTURA))
pygame.display.set_caption("Escape of teacher's")

# Cores
BRANCO = (255, 255, 255)
PRETO = (0, 0, 0)
AMARELO = (255, 255, 0)
VERMELHO = (255, 0, 0)

# Carregar imagens
imagem_jogador = pygame.image.load("FARDApersonagem.jpg")
imagem_boss = pygame.image.load("XANDÃO.jpg")
imagem_moeda = pygame.image.load("IFcoins.png")

# Redimensionar imagens
imagem_jogador = pygame.transform.scale(imagem_jogador, (30, 30))
imagem_boss = pygame.transform.scale(imagem_boss, (30, 30))
imagem_moeda = pygame.transform.scale(imagem_moeda, (20, 20))

# Classe do Jogador
class Jogador(pygame.sprite.Sprite):
    def __init__(self):
        super().__init__()
        self.image = imagem_jogador
        self.rect = self.image.get_rect()
        self.rect.center = (LARGURA // 2, ALTURA // 2)
        self.velocidade = 5

    def update(self, teclas):
        anterior = self.rect.copy()
        if teclas[pygame.K_UP]:
            self.rect.y -= self.velocidade
        if teclas[pygame.K_DOWN]:
            self.rect.y += self.velocidade
        if teclas[pygame.K_LEFT]:
            self.rect.x -= self.velocidade
        if teclas[pygame.K_RIGHT]:
            self.rect.x += self.velocidade
        if pygame.sprite.spritecollide(self, paredes, False):
            self.rect = anterior

# Classe do Boss
class Boss(pygame.sprite.Sprite):
    def __init__(self):
        super().__init__()
        self.image = imagem_boss
        self.rect = self.image.get_rect()
        self.rect.center = (random.randint(0, LARGURA), random.randint(0, ALTURA))
        self.velocidade = 3

    def update(self, jogador_rect):
        anterior = self.rect.copy()
        dx = jogador_rect.x - self.rect.x
        dy = jogador_rect.y - self.rect.y
        distancia = math.hypot(dx, dy)
        if distancia != 0:
            self.rect.x += self.velocidade * dx / distancia
            self.rect.y += self.velocidade * dy / distancia
        if pygame.sprite.spritecollide(self, paredes, False):
            self.rect = anterior

# Classe da Moeda
class Moeda(pygame.sprite.Sprite):
    def __init__(self):
        super().__init__()
        self.image = imagem_moeda
        self.rect = self.image.get_rect()
        self.rect.center = (random.randint(0, LARGURA), random.randint(0, ALTURA))

# Classe das Paredes
class Parede(pygame.sprite.Sprite):
    def __init__(self, x, y, largura, altura):
        super().__init__()
        self.image = pygame.Surface((largura, altura))
        self.image.fill(BRANCO)
        self.rect = self.image.get_rect()
        self.rect.topleft = (x, y)

# Função do Menu
def menu_inicial():
    tela.fill(PRETO)
    fonte = pygame.font.Font(None, 50)
    texto = fonte.render("Pressione Espaço para Jogar", True, BRANCO)
    tela.blit(texto, (LARGURA // 2 - 150, ALTURA // 2))
    pygame.display.flip()
    esperando = True
    while esperando:
        for evento in pygame.event.get():
            if evento.type == pygame.QUIT:
                pygame.quit()
                exit()
            if evento.type == pygame.KEYDOWN:
                if evento.key == pygame.K_SPACE:
                    esperando = False

menu_inicial()

# Grupos de sprites
todos_sprites = pygame.sprite.Group()
moedas = pygame.sprite.Group()
paredes = pygame.sprite.Group()

# Criar jogador e boss
jogador = Jogador()
boss = Boss()
todos_sprites.add(jogador, boss)

# Criar moedas
for _ in range(10):
    moeda = Moeda()
    todos_sprites.add(moeda)
    moedas.add(moeda)

# Criar paredes
lista_paredes = [
    (100, 100, 400, 10),
    (100, 200, 400, 10),
    (100, 300, 400, 10),
    (50, 50, 10, 300),
    (540, 50, 10, 300)
]

for parede_info in lista_paredes:
    parede = Parede(*parede_info)
    paredes.add(parede)
    todos_sprites.add(parede)

# Loop principal do jogo
rodando = True
relogio = pygame.time.Clock()
pontuacao = 0

while rodando:
    relogio.tick(30)
    for evento in pygame.event.get():
        if evento.type == pygame.QUIT:
            rodando = False

    teclas = pygame.key.get_pressed()
    jogador.update(teclas)
    boss.update(jogador.rect)

    moedas_coletadas = pygame.sprite.spritecollide(jogador, moedas, True)
    pontuacao += len(moedas_coletadas)
    if len(moedas) == 0:
        print(f"Você venceu! Pontuação final: {pontuacao}")
        rodando = False
    
    if pygame.sprite.collide_rect(jogador, boss):
        print(f"Você perdeu! Pontuação final: {pontuacao}")
        rodando = False

    tela.fill(PRETO)
    todos_sprites.draw(tela)
    fonte = pygame.font.Font(None, 36)
    texto_pontuacao = fonte.render(f"Pontuação: {pontuacao}", True, BRANCO)
    tela.blit(texto_pontuacao, (10, 10))
    pygame.display.flip()

pygame.quit()
