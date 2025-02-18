# Normal-Snake-Game-
It is normal snake game made from python 
import pygame
import random
import time

pygame.init()

# Set up the game window
WIDTH = 1000
HEIGHT = 800
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Snake Game")

# Colors
BLACK = (0, 0, 0)
WHITE = (255, 255, 255)
RED = (255, 0, 0)
GREEN = (0, 255, 0)
SNAKE_HEAD_COLOR = (0, 100, 0)
SNAKE_BODY_COLOR = (0, 150, 0)

# Game variables
snake_block = 20
base_speed = 5  

# Initialize clock and font
clock = pygame.time.Clock()
font = pygame.font.SysFont(None, 50)

def read_high_score():
    try:
        with open("high_score.txt", "r") as file:
            return int(file.read())
    except:
        return 0

def write_high_score(score):
    with open("high_score.txt", "w") as file:
        file.write(str(score))

def display_score(score, high_score):
    color = WHITE
    if score > high_score:
        color = GREEN
    text = font.render(f"Score: {score}  High Score: {high_score}", True, color)
    screen.blit(text, (10, 10))

def draw_snake(snake_list):
    for i, block in enumerate(snake_list):
        if i == len(snake_list) - 1:
            head_x, head_y = block
            center = (head_x + snake_block // 2, head_y + snake_block // 2)
            pygame.draw.circle(screen, SNAKE_HEAD_COLOR, center, snake_block // 2)
            eye_radius = snake_block // 8
            eye_offset_x = snake_block // 4
            eye_offset_y = snake_block // 4
            pygame.draw.circle(screen, WHITE, (center[0] - eye_offset_x, center[1] - eye_offset_y), eye_radius)
            pygame.draw.circle(screen, WHITE, (center[0] + eye_offset_x, center[1] - eye_offset_y), eye_radius)
            tongue_start = (center[0], center[1] + snake_block // 2 - 2)
            tongue_end = (center[0], center[1] + snake_block // 2 + 4)
            pygame.draw.line(screen, RED, tongue_start, tongue_end, 2)
        else:
            pygame.draw.ellipse(screen, SNAKE_BODY_COLOR, (block[0], block[1], snake_block, snake_block))

def spawn_food():
    x = round(random.randrange(0, WIDTH - snake_block) / snake_block) * snake_block
    y = round(random.randrange(0, HEIGHT - snake_block) / snake_block) * snake_block
    return [x, y]

def game_loop():
    high_score = read_high_score()
    game_over = False
    game_close = False

    x = WIDTH // 2
    y = HEIGHT // 2
    dx = 0
    dy = 0

    snake_list = []
    snake_length = 1

    num_food = 3
    foods = [spawn_food() for _ in range(num_food)]

    while not game_over:
        while game_close:
            screen.fill(BLACK)
            current_score = snake_length - 1
            game_over_text = font.render("Game Over! Press Q-Quit or C-Play Again", True, RED)
            screen.blit(game_over_text, (WIDTH // 4, HEIGHT // 3))
            display_score(current_score, high_score)
            
            if current_score > high_score:
                new_high_text = font.render("New High Score!", True, GREEN)
                screen.blit(new_high_text, (WIDTH // 4, HEIGHT // 2))
            
            pygame.display.update()

            for event in pygame.event.get():
                if event.type == pygame.KEYDOWN:
                    if event.key == pygame.K_q:
                        game_over = True
                        game_close = False
                    if event.key == pygame.K_c:
                        game_loop()

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                game_over = True
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_LEFT and dx == 0:
                    dx = -snake_block
                    dy = 0
                elif event.key == pygame.K_RIGHT and dx == 0:
                    dx = snake_block
                    dy = 0
                elif event.key == pygame.K_UP and dy == 0:
                    dy = -snake_block
                    dx = 0
                elif event.key == pygame.K_DOWN and dy == 0:
                    dy = snake_block
                    dx = 0

        if x >= WIDTH or x < 0 or y >= HEIGHT or y < 0:
            game_close = True

        x += dx
        y += dy
        screen.fill(BLACK)

        for food in foods:
            pygame.draw.circle(screen, RED, (food[0] + snake_block // 2, food[1] + snake_block // 2), snake_block // 2)

        snake_head = [x, y]
        snake_list.append(snake_head)
        if len(snake_list) > snake_length:
            del snake_list[0]

        for block in snake_list[:-1]:
            if block == snake_head:
                game_close = True

        draw_snake(snake_list)
        display_score(snake_length - 1, high_score)
        pygame.display.update()

        current_score = snake_length - 1
        if current_score > high_score:
            high_score = current_score
            write_high_score(high_score)

        for food in foods[:]:
            if x == food[0] and y == food[1]:
                foods.remove(food)
                snake_length += 1
                foods.append(spawn_food())

        current_speed = base_speed + (snake_length - 1) // 2
        clock.tick(current_speed)

    pygame.quit()
    quit()

game_loop()
