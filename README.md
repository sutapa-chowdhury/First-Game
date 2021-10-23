# First-Game
A 2D game that can be played by two players using same keyboard. It's a really simple game.

# Code
import pygame
import os
from pygame import mixer
pygame.font.init()
mixer.init()

WIDTH, HEIGHT = 900, 500
WIN = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption('Mega Spaceship Battle')
ORANGE = (204, 255, 229)
YELLOW = (255, 255, 0)
RED = (255, 0, 0)
FPS = 60
BLACK = (0, 0, 0)
BULLET_VEL = 7
MAX_BULLETS = 3
SPACESHIP_WIDTH, SPACESHIP_HEIGHT = 50, 40
YELLOW_SPACESHIP_IMAGE = pygame.transform.rotate(pygame.image.load(os.path.join('Assets', 'spaceship_yellow.png')), 90)
YELLOW_SPACESHIP = pygame.transform.scale(YELLOW_SPACESHIP_IMAGE, (SPACESHIP_WIDTH, SPACESHIP_HEIGHT))
RED_SPACESHIP_IMAGE = pygame.transform.rotate(pygame.image.load(os.path.join('Assets', 'spaceship_red.png')), 270)
RED_SPACESHIP = pygame.transform.scale(RED_SPACESHIP_IMAGE, (SPACESHIP_WIDTH, SPACESHIP_HEIGHT))
VEL = 5
BORDER = pygame.Rect((WIDTH//2) - 5, 0, 10, HEIGHT)
HEALTH_FONT = pygame.font.SysFont('chiller', 40)
WINNER_FONT = pygame.font.SysFont('Chiller', 100)
BULLET_HIT_SOUND = mixer.Sound(os.path.join('Assets', 'Grenade+1.mp3'))
BULLET_FIRE_SOUND = mixer.Sound(os.path.join('Assets', 'Gun+Silencer.mp3'))
YELLOW_HIT = pygame.USEREVENT + 1
RED_HIT = pygame.USEREVENT + 2
SPACE = pygame.transform.scale(pygame.image.load(os.path.join('Assets', 'space1.jpg')), (WIDTH, HEIGHT))


def draw_window(yellow, red, bullets_yellow, bullets_red, red_health, yellow_health):
    WIN.blit(SPACE, (0, 0))
    pygame.draw.rect(WIN, BLACK, BORDER)
    red_health_text = HEALTH_FONT.render("Health: " + str(red_health), True, ORANGE)
    yellow_health_text = HEALTH_FONT.render("Health: " + str(yellow_health), True, ORANGE)
    WIN.blit(red_health_text, (WIDTH - red_health_text.get_width() - 10, 10))
    WIN.blit(yellow_health_text, (10, 10))
    WIN.blit(YELLOW_SPACESHIP, (yellow.x, yellow.y))
    WIN.blit(RED_SPACESHIP, (red.x, red.y))
    for bullet in bullets_yellow:
        pygame.draw.rect(WIN, YELLOW, bullet)
    for bullet in bullets_red:
        pygame.draw.rect(WIN, RED, bullet)
    pygame.display.update()


def yellow_handle_movement(keys_pressed, yellow):
    if keys_pressed[pygame.K_a] and yellow.x - VEL > 0:  # Left
        yellow.x -= VEL
    if keys_pressed[pygame.K_d] and yellow.x + VEL + yellow.width < BORDER.x:  # Right
        yellow.x += VEL
    if keys_pressed[pygame.K_w] and yellow.y - VEL > 0:  # UP
        yellow.y -= VEL
    if keys_pressed[pygame.K_s] and yellow.y + VEL + yellow.height < HEIGHT:  # DOWN
        yellow.y += VEL


def red_handle_movement(keys_pressed, red):
    if keys_pressed[pygame.K_LEFT] and red.x - VEL > BORDER.x + BORDER.width:
        red.x -= VEL
    if keys_pressed[pygame.K_RIGHT] and red.x + VEL + red.width < WIDTH:
        red.x += VEL
    if keys_pressed[pygame.K_UP] and red.y - VEL > 0:
        red.y -= VEL
    if keys_pressed[pygame.K_DOWN] and red.y + VEL + red.height < HEIGHT:
        red.y += VEL


def handle_bullets(bullets_yellow, bullets_red, yellow, red):
    for bullet in bullets_yellow:
        bullet.x += BULLET_VEL
        if red.colliderect(bullet):
            pygame.event.post(pygame.event.Event(RED_HIT))
            bullets_yellow.remove(bullet)
        elif bullet.x > WIDTH:
            bullets_yellow.remove(bullet)
    for bullet in bullets_red:
        bullet.x -= BULLET_VEL
        if yellow.colliderect(bullet):
            pygame.event.post(pygame.event.Event(YELLOW_HIT))  # creating event
            bullets_red.remove(bullet)
        elif bullet.x < 0:
            bullets_red.remove(bullet)


def draw_winner(text):
    health_text = WINNER_FONT.render(text, True, ORANGE)
    WIN.blit(health_text, (WIDTH // 2 - health_text.get_width() // 2, HEIGHT // 2 - health_text.get_height() // 2))
    pygame.display.update()
    pygame.time.delay(5000)


def main():
    yellow = pygame.Rect(100, 300, SPACESHIP_WIDTH, SPACESHIP_HEIGHT)
    red = pygame.Rect(700, 300, SPACESHIP_WIDTH, SPACESHIP_HEIGHT)
    clock = pygame.time.Clock()
    bullets_yellow = []
    bullets_red = []
    red_health = 10
    yellow_health = 10

    run = True
    while run:
        clock.tick(FPS)
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                run = False
                pygame.quit()

            if event.type == pygame.KEYDOWN:
                # left ctrl key for firing from yellow spaceship
                if event.key == pygame.K_LCTRL and len(bullets_yellow) < MAX_BULLETS:
                    bullet = pygame.Rect(yellow.x + yellow.width, yellow.y + yellow.height//2 - 2, 10, 5)
                    bullets_yellow.append(bullet)
                    BULLET_FIRE_SOUND.play()
                # right ctrl key for firing bullets for red spaceship
                if event.key == pygame.K_RCTRL and len(bullets_red) < MAX_BULLETS:
                    bullet = pygame.Rect(red.x, red.y + red.height//2 - 2, 10, 5)
                    bullets_red.append(bullet)
                    BULLET_FIRE_SOUND.play()

            if event.type == RED_HIT:
                red_health -= 1
                BULLET_HIT_SOUND.play()
            if event.type == YELLOW_HIT:
                yellow_health -= 1
                BULLET_HIT_SOUND.play()
        winner_text = ""
        if yellow_health <= 0:
            winner_text = "Red Mega Spaceship Wins!"
        if red_health <= 0:
            winner_text = "Yellow Mega Spaceship Wins!"
        if winner_text != "":
            draw_winner(winner_text)
            break

        handle_bullets(bullets_yellow, bullets_red, yellow, red)
        draw_window(yellow, red, bullets_yellow, bullets_red, red_health, yellow_health)
        keys_pressed = pygame.key.get_pressed()
        yellow_handle_movement(keys_pressed, yellow)
        red_handle_movement(keys_pressed, red)

    main()


if __name__ == '__main__':
    main()
