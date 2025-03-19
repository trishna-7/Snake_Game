import pygame
import random
import os

# Initialize pygame
pygame.init()

# Game Constants
WIDTH, HEIGHT = 600, 400
GRID_SIZE = 20
LEVELS = {'Easy': 5, 'Medium': 7, 'Hard': 9}  # Different speeds

# Colors
DARK_BG = (30, 30, 30)
SNAKE_HEAD = (0, 200, 0)
SNAKE_BODY = (0, 255, 0)
FOOD_COLOR = (255, 0, 0)
TEXT_COLOR = (255, 255, 255)

# Font
FONT = pygame.font.Font(None, 36)

# Load High Score
HIGH_SCORE_FILE = "highscore.txt"

def load_high_score():
    if os.path.exists(HIGH_SCORE_FILE):
        with open(HIGH_SCORE_FILE, "r") as file:
            return int(file.read())
    return 0

def save_high_score(score):
    with open(HIGH_SCORE_FILE, "w") as file:
        file.write(str(score))

# Game Variables
high_score = load_high_score()
selected_level = None
FPS = LEVELS['Easy']  # Default FPS

# Initialize screen
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Snake Game")

# Functions to Draw Text
def draw_text(text, x, y):
    screen.blit(FONT.render(text, True, TEXT_COLOR), (x, y))

# Main Menu
def main_menu():
    while True:
        screen.fill(DARK_BG)
        screen.blit(FONT.render("SNAKE GAME", True, (0, 255, 0)), (WIDTH // 2 - 100, HEIGHT // 2 - 80))  # Green text
        draw_text("1. Start Game", WIDTH // 2 - 80, HEIGHT // 2 - 20)
        draw_text("2. Help", WIDTH // 2 - 50, HEIGHT // 2 + 20)
        draw_text("3. Exit", WIDTH // 2 - 50, HEIGHT // 2 + 60)

        pygame.display.flip()

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                exit()
            elif event.type == pygame.KEYDOWN:
                if event.key == pygame.K_1:
                    select_level()
                    return
                elif event.key == pygame.K_2:
                    show_help()
                elif event.key == pygame.K_3:
                    pygame.quit()
                    exit()

# Help Screen
def show_help():
    showing_help = True
    while showing_help:
        screen.fill(DARK_BG)
        draw_text("HELP", WIDTH // 2 - 50, HEIGHT // 2 - 80)
        draw_text("Use Arrow keys to move the snake", WIDTH // 2 - 150, HEIGHT // 2 - 40)
        draw_text("Eat food to grow and gain points", WIDTH // 2 - 150, HEIGHT // 2)
        draw_text("Avoid hitting the walls and yourself", WIDTH // 2 - 150, HEIGHT // 2 + 40)
        draw_text("Press B to go back", WIDTH // 2 - 70, HEIGHT // 2 + 80)

        pygame.display.flip()
        
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                exit()
            elif event.type == pygame.KEYDOWN and event.key == pygame.K_b:
                showing_help = False

# Level Selection Menu
def select_level():
    global FPS, selected_level
    selecting = True
    while selecting:
        screen.fill(DARK_BG)
        draw_text("Select Level", WIDTH // 2 - 100, HEIGHT // 2 - 80)
        draw_text("1. Easy", WIDTH // 2 - 50, HEIGHT // 2 - 20)
        draw_text("2. Medium", WIDTH // 2 - 70, HEIGHT // 2 + 20)
        draw_text("3. Hard", WIDTH // 2 - 50, HEIGHT // 2 + 60)

        pygame.display.flip()
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                exit()
            elif event.type == pygame.KEYDOWN:
                if event.key == pygame.K_1:
                    selected_level = "Easy"
                    FPS = LEVELS['Easy']
                    selecting = False
                elif event.key == pygame.K_2:
                    selected_level = "Medium"
                    FPS = LEVELS['Medium']
                    selecting = False
                elif event.key == pygame.K_3:
                    selected_level = "Hard"
                    FPS = LEVELS['Hard']
                    selecting = False

    game_loop()

# Game Over Menu
def game_over(score):
    global high_score
    if score > high_score:
        high_score = score
        save_high_score(high_score)
    
    game_over_screen = True
    while game_over_screen:
        screen.fill(DARK_BG)
        screen.blit(FONT.render("GAME OVER", True, (255, 0, 0)), (WIDTH // 2 - 100, HEIGHT // 2 - 80))  # Red text
        draw_text(f"High Score: {high_score}", WIDTH // 2 - 70, HEIGHT // 2 - 40)
        draw_text("1. New Game", WIDTH // 2 - 80, HEIGHT // 2 + 20)
        draw_text("2. Exit", WIDTH // 2 - 50, HEIGHT // 2 + 60)

        pygame.display.flip()

        
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                exit()
            elif event.type == pygame.KEYDOWN:
                if event.key == pygame.K_1:  # New Game
                    select_level()  # Restart level selection
                elif event.key == pygame.K_2:  # Exit
                    pygame.quit()
                    exit()

# Game Loop
def game_loop():
    global high_score

    snake = [(WIDTH // 2, HEIGHT // 2)]
    direction = (GRID_SIZE, 0)
    food = (random.randint(0, WIDTH // GRID_SIZE - 1) * GRID_SIZE,
            random.randint(0, HEIGHT // GRID_SIZE - 1) * GRID_SIZE)
    score = 0

    def draw_snake():
        for i, segment in enumerate(snake):
            color = SNAKE_HEAD if i == 0 else SNAKE_BODY
            pygame.draw.rect(screen, color, (*segment, GRID_SIZE, GRID_SIZE), border_radius=8)

    def draw_food():
        pygame.draw.circle(screen, FOOD_COLOR, (food[0] + GRID_SIZE//2, food[1] + GRID_SIZE//2), GRID_SIZE//2)

    def check_collision():
        head = snake[0]
        return (head in snake[1:] or
                head[0] < 0 or head[1] < 0 or
                head[0] >= WIDTH or head[1] >= HEIGHT)

    running = True
    while running:
        screen.fill(DARK_BG)
        
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
            elif event.type == pygame.KEYDOWN:
                if event.key == pygame.K_UP and direction != (0, GRID_SIZE):
                    direction = (0, -GRID_SIZE)
                elif event.key == pygame.K_DOWN and direction != (0, -GRID_SIZE):
                    direction = (0, GRID_SIZE)
                elif event.key == pygame.K_LEFT and direction != (GRID_SIZE, 0):
                    direction = (-GRID_SIZE, 0)
                elif event.key == pygame.K_RIGHT and direction != (-GRID_SIZE, 0):
                    direction = (GRID_SIZE, 0)
        
        new_head = (snake[0][0] + direction[0], snake[0][1] + direction[1])
        snake.insert(0, new_head)
        
        if new_head == food:
            score += 10
            food = (random.randint(0, WIDTH // GRID_SIZE - 1) * GRID_SIZE,
                    random.randint(0, HEIGHT // GRID_SIZE - 1) * GRID_SIZE)
        else:
            snake.pop()
        
        if check_collision():
            game_over(score)
        
        draw_snake()
        draw_food()
        draw_text(f"Score: {score}", 10, 10)  # Score is displayed while playing

        pygame.display.flip()
        pygame.time.Clock().tick(FPS)

    pygame.quit()

# Start the game
main_menu()
