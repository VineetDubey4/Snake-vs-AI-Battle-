# Snake-vs-AI-Battle-

import pygame
import random
import sys

pygame.init()


WIDTH, HEIGHT = 600, 400
BLOCK_SIZE = 20
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Realistic AI Snake Battle")
clock = pygame.time.Clock()
FONT = pygame.font.SysFont(None, 30)

BLACK = (0, 0, 0)
GRAY = (40, 40, 40)
WHITE = (255, 255, 255)
GREEN = (0, 200, 0)
BLUE = (0, 150, 255)
RED = (220, 50, 50)

 
def draw_grid():
    for x in range(0, WIDTH, BLOCK_SIZE):
        pygame.draw.line(screen, GRAY, (x, 0), (x, HEIGHT))
    for y in range(0, HEIGHT, BLOCK_SIZE):
        pygame.draw.line(screen, GRAY, (0, y), (WIDTH, y))
 
def draw_snake(snake_blocks, base_color):
    total = len(snake_blocks)
    for i, block in enumerate(snake_blocks):
        fade = int(255 * (1 - i / total))
        color = (base_color[0], base_color[1], fade)
        pygame.draw.rect(screen, color, [block[0], block[1], BLOCK_SIZE, BLOCK_SIZE], border_radius=10)

         
        if i == 0:
            eye_size = 3
            offset = 5
            pygame.draw.circle(screen, BLACK, (block[0] + offset, block[1] + offset), eye_size)
            pygame.draw.circle(screen, BLACK, (block[0] + BLOCK_SIZE - offset, block[1] + offset), eye_size)

 
def draw_food(food, frame_count):
    blink_color = (255, 255 - (frame_count % 40) * 6, 0)  # Blinking yellow-orange
    pygame.draw.rect(screen, blink_color, [food[0], food[1], BLOCK_SIZE, BLOCK_SIZE], border_radius=5)


def generate_food():
    return [random.randrange(0, WIDTH, BLOCK_SIZE), random.randrange(0, HEIGHT, BLOCK_SIZE)]


def show_score(p_score, ai_score):
    text = FONT.render(f'You: {p_score} | AI: {ai_score}', True, WHITE)
    screen.blit(text, [10, 10])


def move_ai(ai_head, food):
    ax, ay = ai_head
    fx, fy = food
    if ax < fx:
        ax += BLOCK_SIZE
    elif ax > fx:
        ax -= BLOCK_SIZE
    elif ay < fy:
        ay += BLOCK_SIZE
    elif ay > fy:
        ay -= BLOCK_SIZE
    return [ax, ay]

 
def check_collision(head, body):
    return head in body or head[0] < 0 or head[1] < 0 or head[0] >= WIDTH or head[1] >= HEIGHT

 
def game_loop():
    game_over = False
    player_snake = [[100, 100]]
    player_direction = 'RIGHT'
    ai_snake = [[400, 300]]
    ai_score = 0
    p_score = 0
    food = generate_food()
    frame_count = 0

    while not game_over:
        frame_count += 1
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                sys.exit()
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_LEFT and player_direction != 'RIGHT':
                    player_direction = 'LEFT'
                elif event.key == pygame.K_RIGHT and player_direction != 'LEFT':
                    player_direction = 'RIGHT'
                elif event.key == pygame.K_UP and player_direction != 'DOWN':
                    player_direction = 'UP'
                elif event.key == pygame.K_DOWN and player_direction != 'UP':
                    player_direction = 'DOWN'

         
        px, py = player_snake[0]
        if player_direction == 'LEFT':
            px -= BLOCK_SIZE
        elif player_direction == 'RIGHT':
            px += BLOCK_SIZE
        elif player_direction == 'UP':
            py -= BLOCK_SIZE
        elif player_direction == 'DOWN':
            py += BLOCK_SIZE
        new_head = [px, py]
        player_snake.insert(0, new_head)

         
        ai_head = ai_snake[0]
        new_ai_head = move_ai(ai_head, food)
        ai_snake.insert(0, new_ai_head)

         
        if player_snake[0] == food:
            p_score += 1
            food = generate_food()
        else:
            player_snake.pop()

        if ai_snake[0] == food:
            ai_score += 1
            food = generate_food()
        else:
            ai_snake.pop()

         
        if check_collision(player_snake[0], player_snake[1:]) or player_snake[0] in ai_snake:
            game_over = True
        if check_collision(ai_snake[0], ai_snake[1:]) or ai_snake[0] in player_snake:
            game_over = True

        if p_score >= 10 or ai_score >= 10:
            game_over = True

        screen.fill(BLACK)
        draw_grid()
        draw_snake(player_snake, BLUE)
        draw_snake(ai_snake, RED)
        draw_food(food, frame_count)
        show_score(p_score, ai_score)
        pygame.display.update()
        clock.tick(12)

     
    screen.fill(BLACK)
    msg = ""
    if p_score > ai_score:
        msg = "🎉 You Win! Press R to Restart or Q to Quit"
    elif ai_score > p_score:
        msg = "🤖 AI Wins! Press R to Restart or Q to Quit"
    else:
        msg = "⚔️ Draw! Press R to Restart or Q to Quit"
    text = FONT.render(msg, True, WHITE)
    screen.blit(text, [WIDTH // 6, HEIGHT // 2])
    pygame.display.update()

     
    while True:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                sys.exit()
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_q:
                    sys.exit()
                if event.key == pygame.K_r:
                    game_loop()

 
game_loop()
