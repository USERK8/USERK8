import pygame
import time
import random

# Initialize Pygame

pygame.init()

# Window dimensions

WIDTH, HEIGHT = 1365, 700

WIN = pygame.display.set_mode((WIDTH, HEIGHT))

pygame.display.set_caption("MAI3 THE SPAZE DODGER")



# Colors

WHITE = (255, 255, 255)

GREEN = (255, 0, 0)


# Load background image

BG = pygame.transform.scale(pygame.image.load("bg.jpeg"), (WIDTH, HEIGHT))



# Fonts

FONT = pygame.font.SysFont("comicsans", 30)

# Player dimensions and velocity
PLAYER_WIDTH = 40
PLAYER_HEIGHT = 60
PLAYER_VEL = 10  # Increased velocity for button-controlled movement

# Star dimensions and velocity
STAR_WIDTH = 10
STAR_HEIGHT = 20
STAR_VEL = 3

# Load music
pygame.mixer.music.load("/home/sohan/Downloads/Grand Theft Auto IV Theme Song [best quality].mp3")

# Function to draw text on screen
def draw_text(text, font, color, x, y):
    text_surface = font.render(text, True, color)
    text_rect = text_surface.get_rect(center=(x, y))
    WIN.blit(text_surface, text_rect)

# Function to draw main menu
def draw_menu():
    WIN.blit(BG, (0, 0))
    draw_text("SPACE DODGE", FONT, WHITE, WIDTH // 2, HEIGHT // 4)
    draw_text("PLAY GAME", FONT, WHITE, WIDTH // 2, HEIGHT // 2)
    pygame.display.update()

# Function to draw game over screen
def draw_game_over(score):
    WIN.fill(RED)
    draw_text("GAME OVER", FONT, WHITE, WIDTH // 2, HEIGHT // 2 - 50)
    draw_text(f"SCORE: {score}", FONT, WHITE, WIDTH // 2, HEIGHT // 2 + 50)
    draw_text("RETURN TO MENU", FONT, WHITE, WIDTH // 2, HEIGHT // 2 + 150)
    pygame.display.update()
    while True:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                quit()
            if event.type == pygame.MOUSEBUTTONDOWN:
                if WIDTH // 2 - 150 <= pygame.mouse.get_pos()[0] <= WIDTH // 2 + 150 and HEIGHT // 2 + 100 <= pygame.mouse.get_pos()[1] <= HEIGHT // 2 + 200:
                    return True

# Function to draw buttons
def draw_buttons():
    pygame.draw.rect(WIN, WHITE, (50, HEIGHT - 100, 100, 50))  # Left button
    pygame.draw.rect(WIN, WHITE, (WIDTH - 150, HEIGHT - 100, 100, 50))  # Right button
    draw_text("LEFT", FONT, GREEN, 100, HEIGHT - 75)
    draw_text("RIGHT", FONT, GREEN, WIDTH - 100, HEIGHT - 75)

# Main menu loop
def main_menu():
    run = True
    while run:
        draw_menu()
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                quit()
            if event.type == pygame.MOUSEBUTTONDOWN:
                if WIDTH // 2 - 150 <= pygame.mouse.get_pos()[0] <= WIDTH // 2 + 150 and HEIGHT // 2 - 50 <= pygame.mouse.get_pos()[1] <= HEIGHT // 2 + 50:
                    return True
        pygame.display.update()

# Game loop
def main():
    # Play music
    pygame.mixer.music.play(-1)  # -1 plays the music indefinitely

    while True:
        if not main_menu():
            break

        while True:
            player = pygame.Rect(WIDTH // 2 - PLAYER_WIDTH // 2, HEIGHT - PLAYER_HEIGHT, PLAYER_WIDTH, PLAYER_HEIGHT)
            stars = []
            clock = pygame.time.Clock()
            start_time = time.time()
            star_add_increment = 2000
            star_count = 0
            score = 0

            # Set the key repeat to handle continuous movement
            pygame.key.set_repeat(50, 50)

            game_over = False  # Flag to track if the game is over

            while not game_over:  # Continue the game loop until game_over is True
                for event in pygame.event.get():
                    if event.type == pygame.QUIT:
                        pygame.quit()
                        quit()
                    if event.type == pygame.KEYDOWN:
                        if event.key == pygame.K_LEFT:
                            player.x -= PLAYER_VEL
                        if event.key == pygame.K_RIGHT:
                            player.x += PLAYER_VEL
                    if event.type == pygame.MOUSEBUTTONDOWN:
                        if 50 <= pygame.mouse.get_pos()[0] <= 150 and HEIGHT - 100 <= pygame.mouse.get_pos()[1] <= HEIGHT - 50:
                            player.x -= PLAYER_VEL
                        if WIDTH - 150 <= pygame.mouse.get_pos()[0] <= WIDTH - 50 and HEIGHT - 100 <= pygame.mouse.get_pos()[1] <= HEIGHT - 50:
                            player.x += PLAYER_VEL

                keys = pygame.key.get_pressed()

                # Move player
                if keys[pygame.K_LEFT] and player.x - PLAYER_VEL >= 0:
                    player.x -= PLAYER_VEL
                if keys[pygame.K_RIGHT] and player.x + PLAYER_VEL + player.width <= WIDTH:
                    player.x += PLAYER_VEL

                # Add stars
                star_count += clock.tick(60)
                elapsed_time = time.time() - start_time
                if star_count > star_add_increment:
                    for _ in range(3):
                        star_x = random.randint(0, WIDTH - STAR_WIDTH)
                        star = pygame.Rect(star_x, -STAR_HEIGHT, STAR_WIDTH, STAR_HEIGHT)
                        stars.append(star)
                    star_add_increment = max(200, star_add_increment - 50)
                    star_count = 0

                # Move stars and check collision
                for star in stars[:]:
                    star.y += STAR_VEL
                    if star.y > HEIGHT:
                        stars.remove(star)
                    elif star.y + star.height >= player.y and star.colliderect(player):
                        game_over = True  # Set game_over to True if collision occurs
                        break
                    elif star.y + star.height >= player.y and not star.colliderect(player):
                        score += 1

                # Draw everything
                WIN.blit(BG, (0, 0))
                draw_text(f"Time: {round(elapsed_time)}s", FONT, WHITE, 70, 50)
                draw_text(f"Score: {score}", FONT, WHITE, WIDTH - 100, 50)
                pygame.draw.rect(WIN, (255, 0, 0), player)
                for star in stars:
                    pygame.draw.rect(WIN, WHITE, star)
                draw_buttons()
                pygame.display.update()

            # Once the game is over, display the game over screen
            if draw_game_over(score):  # Wait until player clicks the "RETURN TO MENU" button
                break  # Break out of the current game loop and return to main menu

if __name__ == "__main__":
    main()
