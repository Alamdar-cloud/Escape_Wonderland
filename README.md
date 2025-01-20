# Escape_Wonderland
import pygame
import random
import sys
import time
from pygame import mixer

pygame.init()
mixer.init()  

screen_width = 800
screen_height = 600
screen = pygame.display.set_mode((screen_width, screen_height))
pygame.display.set_caption("Alice's Adventure")

background_image = pygame.image.load("Wonderland.jpg")
background_image = pygame.transform.scale(background_image, (screen_width, screen_height))

alice_image = pygame.image.load("Alice.jpg")
alice_image = pygame.transform.scale(alice_image, (50, 50))

entity_image = pygame.image.load("Entity.jpg")
entity_image = pygame.transform.scale(entity_image, (40, 60))

golden_key_image = pygame.image.load("Gold_Key.png")
golden_key_image = pygame.transform.scale(golden_key_image, (40, 40))

menu_background_image = pygame.image.load("MENU..jpg")
menu_background_image = pygame.transform.scale(menu_background_image, (screen_width, screen_height))

overscreen_image = pygame.image.load("Wonderland_overscreen.jpg")
overscreen_image = pygame.transform.scale(overscreen_image, (screen_width, screen_height))

queen_image = pygame.image.load("Queen_Of_Hearts.png")
queen_image = pygame.transform.scale(queen_image, (100, 150))

nuke_image = pygame.image.load("NUKE.png")
nuke_image = pygame.transform.scale(nuke_image, (60, 60))

clock = pygame.time.Clock()

WHITE = (255, 255, 255)
GREEN = (0, 255, 0)
RED = (255, 0, 0)
YELLOW = (255, 255, 0)
BLACK = (0, 0, 0)
BUTTON_COLOR = (50, 50, 50)
BUTTON_HOVER_COLOR = (100, 100, 100)

font = pygame.font.SysFont("Arial", 36)
big_font = pygame.font.SysFont("Comic Sans MS", 72)
button_font = pygame.font.SysFont("Arial", 30)

alice_width = 50
alice_height = 50
alice_color = GREEN
max_speed = 7
acceleration = 0.5
friction = 0.1

bullet_width = 5
bullet_height = 10
bullet_speed = 7

key_width = 40
key_height = 40
key_color = YELLOW
total_keys = 14

score = 0
keys_collected = 0
enemy_speed = 5
difficulty_increase_rate = 5
level = 1

game_over = False
start_time = time.time()

velocity_x = 0
velocity_y = -3 

bullets = []

cards = []
nuke_active = False
cards_destroyed = 0
total_cards = 20

class Button:
    def __init__(self, text, x, y, width, height):
        self.text = text
        self.rect = pygame.Rect(x, y, width, height)
        self.color = BUTTON_COLOR
        self.hover_color = BUTTON_HOVER_COLOR
        self.font = button_font
        self.text_surface = self.font.render(self.text, True, WHITE)
        self.text_rect = self.text_surface.get_rect(center=self.rect.center)

    def draw(self, screen):
        mouse_x, mouse_y = pygame.mouse.get_pos()
        if self.rect.collidepoint(mouse_x, mouse_y):
            pygame.draw.rect(screen, self.hover_color, self.rect)
        else:
            pygame.draw.rect(screen, self.color, self.rect)
        screen.blit(self.text_surface, self.text_rect)

    def is_hovered(self):
        mouse_x, mouse_y = pygame.mouse.get_pos()
        return self.rect.collidepoint(mouse_x, mouse_y)

    def is_clicked(self):
        return self.is_hovered() and pygame.mouse.get_pressed()[0]

def draw_game_over():
    screen.blit(overscreen_image, (0, 0))  
    
    if keys_collected == total_keys:
        game_over_text = "Congratulations! You have collected all the keys!"
    else:
        game_over_text = "Game Over"
        
    game_over_font = pygame.font.SysFont("Comic Sans MS", 72)
    game_over_font = (game_over_text, game_over_font, screen_width - 20)

    final_score_font = pygame.font.SysFont("Arial", 36)
    final_score_font = (f"Score: {score}", final_score_font, screen_width - 20)

    try_again_button = Button("Try Again", screen_width // 2 - 100, screen_height // 1.5, 200, 50)
    quit_button = Button("Quit", screen_width // 2 - 100, screen_height // 1.2, 200, 50)

    mixer.music.load("Alice in Wonderland  GAME OVER Escape Rooms.mp3")
    mixer.music.play(loops=-1)

    screen.blit(game_over_font.render(game_over_text, True, WHITE), 
                (screen_width // 2 - game_over_font.size(game_over_text)[0] // 2, 200))

    screen.blit(final_score_font.render(f"Score: {score}", True, WHITE), 
                (screen_width // 2 - final_score_font.size(f"Score: {score}")[0] // 2, 300))

    try_again_button.draw(screen)
    quit_button.draw(screen)

    pygame.display.update()

    waiting_for_input = True
    while waiting_for_input:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()

            if event.type == pygame.MOUSEBUTTONDOWN:
                if try_again_button.is_clicked():
                    waiting_for_input = False  
                    game_loop() 
                elif quit_button.is_clicked():
                    pygame.quit()
                    sys.exit()


def draw_game_over():
    screen.blit(overscreen_image, (0, 0)) 
    game_over_text = big_font.render("Game Over", True, WHITE)
    final_score_text = font.render(f"Score: {score}", True, WHITE)
    try_again_button = Button("Try Again", screen_width // 2 - 100, screen_height // 1.5, 200, 50)
    quit_button = Button("Quit", screen_width // 2 - 100, screen_height // 1.2, 200, 50)
     
    
    mixer.music.load("Alice in Wonderland  GAME OVER Escape Rooms.mp3")
    mixer.music.play(loops=-1)

    screen.blit(game_over_text, (screen_width // 2 - game_over_text.get_width() // 2, 200))
    screen.blit(final_score_text, (screen_width // 2 - final_score_text.get_width() // 2, 300))
    try_again_button.draw(screen)
    quit_button.draw(screen)

    pygame.display.update()

    waiting_for_input = True
    while waiting_for_input:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()

            if event.type == pygame.MOUSEBUTTONDOWN:
                if try_again_button.is_clicked():
                    waiting_for_input = False 
                    game_loop()  
                elif quit_button.is_clicked():
                    pygame.quit()
                    sys.exit()

def draw_game():
    
    screen.blit(background_image, (0, 0))

   
    screen.blit(alice_image, (alice_x, alice_y))

    
    for key in keys:
        screen.blit(golden_key_image, key.topleft)

    
    for card in cards:
        screen.blit(entity_image, card)

    
    for bullet in bullets:
        pygame.draw.rect(screen, WHITE, bullet)

    
    if level == 2:
        screen.blit(queen_image, (screen_width // 2 - queen_image.get_width() // 2, 50))

    
    score_text = font.render(f"Score: {score}", True, WHITE)
    keys_text = font.render(f"Keys: {keys_collected}/{total_keys}", True, WHITE)
    screen.blit(score_text, (10, 10))
    screen.blit(keys_text, (screen_width - keys_text.get_width() - 10, 10))

    pygame.display.update()

def create_key():
    key_x = random.randint(0, screen_width - key_width)
    key_y = random.randint(0, screen_height - key_height)
    return pygame.Rect(key_x, key_y, key_width, key_height)


def create_card():
    card_x = random.randint(0, screen_width - 40)
    card_y = -60  
    return pygame.Rect(card_x, card_y, 40, 60)


def check_key_collision():
    global keys_collected
    alice_rect = pygame.Rect(alice_x, alice_y, alice_width, alice_height)
    for key in keys[:]:
        if alice_rect.colliderect(key):
            keys.remove(key)
            keys_collected += 1


def check_card_collision():
    alice_rect = pygame.Rect(alice_x, alice_y, alice_width, alice_height)
    for card in cards:
        if alice_rect.colliderect(card):
            return True  
    return False


def check_bullet_collision():
    global score, cards_destroyed
    for bullet in bullets[:]:
        for card in cards[:]:
            if bullet.colliderect(card):
                bullets.remove(bullet)  
                cards.remove(card)  
                score += 50  
                cards_destroyed += 1
                break 


def update_bullets():
    global bullets
    for bullet in bullets[:]:
        bullet.y -= bullet_speed  
        if bullet.y < 0:
            bullets.remove(bullet)  


def main_menu():
    global game_over
    menu_running = True
    start_button = Button("Start", screen_width // 2 - 100, screen_height // 2, 200, 50)
    quit_button = Button("Quit", screen_width // 2 - 100, screen_height // 1.5, 200, 50)

    mixer.music.load("Alice in Wonderland (Score) 2010- Alice Reprise #5.mp3")
    mixer.music.play(loops=-1)

    while menu_running:
        screen.blit(menu_background_image, (0, 0))
        title_text = big_font.render("Alice's Adventure", True, WHITE)
        screen.blit(title_text, (screen_width // 2 - title_text.get_width() // 2, screen_height // 4))
        start_button.draw(screen)
        quit_button.draw(screen)

        pygame.display.update()

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                menu_running = False
                pygame.quit()
                sys.exit()

            if event.type == pygame.MOUSEBUTTONDOWN:
                if start_button.is_clicked():
                    menu_running = False  
                    mixer.music.stop()
                    mixer.music.load("Going To Battle (From Alice in WonderlandScore).mp3")
                    mixer.music.play(loops=-1)
                    game_loop()
                elif quit_button.is_clicked():
                    menu_running = False  
                    pygame.quit()
                    sys.exit()


def game_loop():
    global alice_x, alice_y, score, cards, keys, keys_collected, enemy_speed, difficulty_increase_rate, game_over, velocity_x, velocity_y, start_time, level, bullets, nuke_active, cards_destroyed

    
    alice_x = screen_width // 2
    alice_y = screen_height - alice_height - 10
    score = 0
    keys_collected = 0
    cards = []
    keys = [create_key() for _ in range(total_keys)]
    cards_destroyed = 0
    nuke_active = False

    running = True
    while running:
        clock.tick(60)

        elapsed_time = time.time() - start_time
        score = int(elapsed_time) + keys_collected * 10 

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_SPACE and level == 2 and nuke_active:
                    cards.clear()
                    nuke_active = False  
                    score += 500  

        keys_pressed = pygame.key.get_pressed()

        if keys_pressed[pygame.K_LEFT]:
            velocity_x = max(velocity_x - acceleration, -max_speed)
        elif keys_pressed[pygame.K_RIGHT]:
            velocity_x = min(velocity_x + acceleration, max_speed)
        else:
            if velocity_x > 0:
                velocity_x = max(velocity_x - friction, 0)
            elif velocity_x < 0:
                velocity_x = min(velocity_x + friction, 0)

        if keys_pressed[pygame.K_UP]:
            velocity_y = max(velocity_y - acceleration, -max_speed)
        elif keys_pressed[pygame.K_DOWN]:
            velocity_y = min(velocity_y + acceleration, max_speed)
        else:
            if velocity_y > 0:
                velocity_y = max(velocity_y - friction, 0)
            elif velocity_y < 0:
                velocity_y = min(velocity_y + friction, 0)

        alice_y += velocity_y
        alice_x += velocity_x

        alice_x = max(0, min(screen_width - alice_width, alice_x))
        alice_y = max(0, min(screen_height - alice_height, alice_y))

        if random.randint(1, 30) == 1 and level == 1:
            cards.append(create_card())
        elif random.randint(1, 30) == 1 and level == 2:
            cards.append(create_card())

        for card in cards[:]:
            card.y += enemy_speed
            if card.y > screen_height:
                cards.remove(card)

        check_key_collision()
        if check_card_collision():
            game_over = True
            running = False

        if level == 2:
            update_bullets()
            check_bullet_collision()

        if keys_collected >= difficulty_increase_rate:
            enemy_speed += 1
            difficulty_increase_rate += 2

        draw_game()

        if keys_collected == total_keys:
            screen.fill(BLACK)
            level_up_text = big_font.render("Level 1 Complete! Proceed to Level 2?", True, WHITE)
            screen.blit(level_up_text, (screen_width // 2 - level_up_text.get_width() // 2, screen_height // 2 - 50))
            continue_button = Button("Continue", screen_width // 2 - 100, screen_height // 2 + 50, 200, 50)
            quit_button = Button("Quit", screen_width // 2 - 100, screen_height // 2 + 150, 200, 50)
            continue_button.draw(screen)
            quit_button.draw(screen)

            pygame.display.update()

            waiting_for_input = True
            while waiting_for_input:
                for event in pygame.event.get():
                    if event.type == pygame.QUIT:
                        pygame.quit()
                        sys.exit()

                    if event.type == pygame.MOUSEBUTTONDOWN:
                        if continue_button.is_clicked():
                            level = 2
                            start_time = time.time()
                            keys_collected = 0
                            waiting_for_input = False
                        elif quit_button.is_clicked():
                            pygame.quit()
                            sys.exit()

        if cards_destroyed == total_cards:
            level = 3
            screen.fill(BLACK)
            victory_text = big_font.render("You Win!", True, WHITE)
            screen.blit(victory_text, (screen_width // 2 - victory_text.get_width() // 2, screen_height // 2))
            pygame.display.update()
            pygame.time.wait(2000)
            running = False

    mixer.music.stop()
    draw_game_over()
    

    pygame.quit()
    sys.exit()

main_menu()










       

 
