import pygame
import random

pygame.init()

largura = 800
altura = 600
tela = pygame.display.set_mode((largura, altura))
pygame.display.set_caption("Foguete Blaze")

branco = (255, 255, 255)
preto = (0, 0, 0)
vermelho = (255, 0, 0)

# Carregar imagens (ajuste os caminhos conforme necessário)
nave_img = pygame.image.load("Foguete Blaze.png")
asteroide_img = pygame.image.load("asteroide_screen.png")
fundo_img = pygame.image.load("Espaçosideral_screen.jpg")

# Redimensionar imagens (opcional)
nave_img = pygame.transform.scale(nave_img, (50, 50))
asteroide_img = pygame.transform.scale(asteroide_img, (50, 50))
fundo_img = pygame.transform.scale(fundo_img, (largura, altura))

# Classe da Nave
class Nave(pygame.sprite.Sprite):
    def __init__(self):
        pygame.sprite.Sprite.__init__(self)
        self.image = nave_img  # Usar a imagem da nave
        self.rect = self.image.get_rect()
        self.rect.centerx = largura // 2
        self.rect.bottom = altura - 10
        self.speedx = 0

    def update(self):
        self.speedx = 0
        keystate = pygame.key.get_pressed()
        if keystate[pygame.K_LEFT]:
            self.speedx = -5
        if keystate[pygame.K_RIGHT]:
            self.speedx = 5
        self.rect.x += self.speedx
        if self.rect.left < 0:
            self.rect.left = 0
        if self.rect.right > largura:
            self.rect.right = largura

# Classe do Asteroide
class Asteroide(pygame.sprite.Sprite):
    def __init__(self):
        pygame.sprite.Sprite.__init__(self)
        self.image = asteroide_img  # Usar a imagem do asteroide
        self.rect = self.image.get_rect()
        self.rect.x = random.randrange(largura - self.rect.width)
        self.rect.y = random.randrange(-100, -40)
        self.speedy = random.randrange(1, 8)
        self.speedx = random.randrange(-3, 3)

    def update(self):
        self.rect.x += self.speedx
        self.rect.y += self.speedy
        if self.rect.top > altura + 10 or self.rect.left < -25 or self.rect.right > largura + 20:
            self.rect.x = random.randrange(largura - self.rect.width)
            self.rect.y = random.randrange(-100, -40)
            self.speedy = random.randrange(1, 8)

# Função para exibir o placar
def exibir_placar(surf, texto, tamanho, x, y):
    fonte = pygame.font.Font(None, tamanho)  # Usando a fonte padrão
    texto_surface = fonte.render(texto, True, branco)
    texto_rect = texto_surface.get_rect()
    texto_rect.midtop = (x, y)  
    surf.blit(texto_surface, texto_rect)

# Função para desenhar o texto centralizado
def desenhar_texto(surf, texto, tamanho, cor, x, y):
    fonte = pygame.font.Font(None, tamanho)  # Usando a fonte padrão
    texto_surface = fonte.render(texto, True, cor)  # True para anti-aliasing
    texto_rect = texto_surface.get_rect()
    texto_rect.center = (x, y)
    surf.blit(texto_surface, texto_rect)

# Carregar o recorde do arquivo
def carregar_recorde():
    try:
        with open("recorde.txt", "r") as f:
            return int(f.read())
    except FileNotFoundError:
        return 0

# Salvar o recorde no arquivo
def salvar_recorde(recorde):
    with open("recorde.txt", "w") as f:
        f.write(str(recorde))

# Loop principal do jogo
clock = pygame.time.Clock()
pontuacao = 0
recorde = carregar_recorde()  # Carrega o recorde ao iniciar o jogo
game_over = False
running = True
no_menu = True

#MENU
while no_menu:
    tela.blit(fundo_img, (0, 0))
    desenhar_texto(tela, "Foguete Blaze", 64, branco, largura // 2, altura // 4)
    desenhar_texto(tela, "Aperte ESPAÇO para começar", 32, branco, largura // 2, altura // 2)
    desenhar_texto(tela, "Aperte ESC para sair", 32, branco, largura // 2, (altura // 2) + 50)
    pygame.display.flip()
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            quit()
        if event.type == pygame.KEYDOWN:
            if event.key == pygame.K_SPACE:
                no_menu = False
            if event.key == pygame.K_ESCAPE:
                pygame.quit()
                quit()

todos_sprites = pygame.sprite.Group()
asteroides = pygame.sprite.Group()
nave = Nave()
todos_sprites.add(nave)

for i in range(8):
    a = Asteroide()
    todos_sprites.add(a)
    asteroides.add(a)

while running:
    if game_over:
        tela.fill(preto)
        desenhar_texto(tela, "GAME OVER", 64, vermelho, largura // 2, altura // 4)
        desenhar_texto(tela, "Pontuação: " + str(pontuacao), 32, branco, largura // 2, altura // 2)
        desenhar_texto(tela, "Recorde: " + str(recorde), 32, branco, largura // 2, altura // 2 + 50)
        desenhar_texto(tela, "Pressione qualquer tecla para jogar novamente", 24, branco, largura // 2, altura * 3 // 4)
        pygame.display.flip()

        aguardando = True
        while aguardando:
            clock.tick(30)
            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    running = False
                    aguardando = False
                if event.type == pygame.KEYUP:
                    aguardando = False
                    game_over = False
                    # Reinicializar o jogo
                    todos_sprites = pygame.sprite.Group()
                    asteroides = pygame.sprite.Group()
                    nave = Nave()
                    todos_sprites.add(nave)
                    for i in range(8):
                        a = Asteroide()
                        todos_sprites.add(a)
                        asteroides.add(a)
                    pontuacao = 0

    else:
        clock.tick(60)
        # Eventos
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False

        # Update
        todos_sprites.update()

        # Colisões
        colisoes = pygame.sprite.spritecollide(nave, asteroides, False)
        if colisoes:
            game_over = True
            if pontuacao > recorde:
                recorde = pontuacao
                salvar_recorde(recorde)  # Salva o novo recorde

        # Render
        tela.blit(fundo_img, (0, 0))  # Desenha o fundo
        todos_sprites.draw(tela)
        exibir_placar(tela, "Pontuação: " + str(pontuacao), 24, largura // 2, 10)
        exibir_placar(tela, "Recorde: " + str(recorde), 24, largura // 4, 10)  # Exibe o recorde
        pygame.display.flip()

        # Aumentar a pontuação
        pontuacao += 1


pygame.quit()
