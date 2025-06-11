import pygame
import random

# Settings
WIDTH, HEIGHT = 600, 600
ROWS, COLS = 10, 10
CELL_SIZE = WIDTH // COLS
WORDS = ["PYTHON", "CODE", "PUZZLE", "GAME", "DARK", "THEME"]
FONT_SIZE = 32
BG_COLOR = (30, 30, 40)
GRID_COLOR = (60, 60, 80)
HIGHLIGHT_COLOR = (80, 180, 70)
WORD_COLOR = (220, 220, 255)
FOUND_COLOR = (100, 250, 150)
FONT_COLOR = (200, 200, 210)

pygame.init()
screen = pygame.display.set_mode((WIDTH, HEIGHT + 100))
pygame.display.set_caption("Word Puzzle - Dark Theme")
font = pygame.font.SysFont("consolas", FONT_SIZE)
small_font = pygame.font.SysFont("consolas", 22)

# Generate grid and place words
def generate_grid():
    grid = [["" for _ in range(COLS)] for _ in range(ROWS)]
    placed_words = []
    for word in WORDS:
        placed = False
        attempts = 0
        while not placed and attempts < 100:
            attempts += 1
            dir = random.choice([(1, 0), (0, 1), (1, 1)])
            if dir == (1, 0):  # horizontal
                r, c = random.randint(0, ROWS-1), random.randint(0, COLS-len(word))
            elif dir == (0, 1):  # vertical
                r, c = random.randint(0, ROWS-len(word)), random.randint(0, COLS-1)
            else:  # diagonal
                r, c = random.randint(0, ROWS-len(word)), random.randint(0, COLS-len(word))
            ok = True
            for i in range(len(word)):
                rr, cc = r + dir[1]*i, c + dir[0]*i
                if grid[rr][cc] not in ("", word[i]):
                    ok = False
                    break
            if ok:
                for i in range(len(word)):
                    rr, cc = r + dir[1]*i, c + dir[0]*i
                    grid[rr][cc] = word[i]
                placed_words.append((word, r, c, dir))
                placed = True
    # Fill empty cells
    for r in range(ROWS):
        for c in range(COLS):
            if grid[r][c] == "":
                grid[r][c] = chr(random.randint(65, 90))
    return grid, placed_words

grid, placed_words = generate_grid()
selected = []
found_words = []

def get_cell(pos):
    x, y = pos
    if x < 0 or x >= WIDTH or y < 0 or y >= HEIGHT:
        return None
    return y // CELL_SIZE, x // CELL_SIZE

def draw_grid():
    for r in range(ROWS):
        for c in range(COLS):
            rect = pygame.Rect(c*CELL_SIZE, r*CELL_SIZE, CELL_SIZE, CELL_SIZE)
            color = GRID_COLOR
            if (r, c) in selected:
                color = HIGHLIGHT_COLOR
            pygame.draw.rect(screen, color, rect, 0 if (r, c) in selected else 1)
            letter = grid[r][c]
            text = font.render(letter, True, FONT_COLOR)
            text_rect = text.get_rect(center=rect.center)
            screen.blit(text, text_rect)

def draw_words():
    y = HEIGHT + 10
    for word in WORDS:
        color = FOUND_COLOR if word in found_words else WORD_COLOR
        label = small_font.render(word, True, color)
        screen.blit(label, (20, y))
        y += 28

def check_word(selected):
    if not selected:
        return
    word = "".join([grid[r][c] for r, c in selected])
    rev_word = word[::-1]
    for w in WORDS:
        if word == w or rev_word == w:
            if w not in found_words:
                found_words.append(w)
            return

def main():
    running = True
    dragging = False
    global selected
    while running:
        screen.fill(BG_COLOR)
        draw_grid()
        draw_words()
        pygame.display.flip()
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
            elif event.type == pygame.MOUSEBUTTONDOWN:
                selected = []
                dragging = True
                cell = get_cell(pygame.mouse.get_pos())
                if cell:
                    selected.append(cell)
            elif event.type == pygame.MOUSEMOTION and dragging:
                cell = get_cell(pygame.mouse.get_pos())
                if cell and cell not in selected:
                    selected.append(cell)
            elif event.type == pygame.MOUSEBUTTONUP and dragging:
                dragging = False
                check_word(selected)
                selected = []
    pygame.quit()

if __name__ == "__main__":
    main()
    
