import pgzrun
import random
import time
import math

WIDTH = 800
HEIGHT = 600

# Música
music_file = "bg_music"
sounds_on = True

# Estados
game_state = "menu"
start_time = 0
TIME_LIMIT = 60  # segundos
victory_time = 0
defeat_time = 0

# Raio de ataque do herói
ATTACK_RADIUS = 40

# Botões do menu
menu_buttons = [
    {"label": "Start Game", "rect": Rect((WIDTH//2 - 100, 200), (200, 50))},
    {"label": "Toggle Sound", "rect": Rect((WIDTH//2 - 100, 300), (200, 50))},
    {"label": "Quit", "rect": Rect((WIDTH//2 - 100, 400), (200, 50))}
]

# Classes
class AnimatedActor:
    def __init__(self, idle_img, walk_img, pos, name="enemy", attack_imgs=None):
        self.idle_frames = [Actor(idle_img, pos), Actor(idle_img, pos)]
        self.walk_frames = [Actor(walk_img, pos), Actor(walk_img, pos)]
        self.attack_frames = [Actor(attack_imgs[0], pos), Actor(attack_imgs[1], pos)] if attack_imgs else None
        self.pos = pos
        self.mode = "idle"
        self.frame_index = 0
        self.animation_speed = 0.15
        self.animation_timer = 0
        self.name = name
        self.hp = 100
        self.damage = 10
        self.attack_timer = 0

    def update(self):
        self.animation_timer += self.animation_speed
        if self.animation_timer >= 1:
            self.animation_timer = 0
            self.frame_index = (self.frame_index + 1) % 2

        if self.mode == "attack":
            self.attack_timer += 1
            if self.attack_timer >= 10:
                self.mode = "idle"
                self.attack_timer = 0

    def draw(self):
        frame = self.idle_frames if self.mode == "idle" else self.walk_frames
        if self.mode == "attack" and self.attack_frames:
            frame = self.attack_frames
        frame[self.frame_index].pos = self.pos
        frame[self.frame_index].draw()

    def move_to(self, new_pos):
        x = max(16, min(WIDTH - 16, new_pos[0]))
        y = max(16, min(HEIGHT - 16, new_pos[1]))
        self.pos = (x, y)
        for frame in self.idle_frames + self.walk_frames:
            frame.pos = self.pos
        if self.attack_frames:
            for frame in self.attack_frames:
                frame.pos = self.pos

    def get_rect(self):
        return Rect((self.pos[0] - 16, self.pos[1] - 16), (32, 32))

# Hero
hero = AnimatedActor(
    idle_img="hero_walk",
    walk_img="hero_walk",
    pos=(WIDTH//2, HEIGHT//2),
    name="hero",
    attack_imgs=["hero_attack1", "hero_attack2"]
)
hero.hp = 100
hero.level = 1
hero.inventory = []

# Inimigos
enemy_types = [
    ("goblion_idle", "goblion_walk"),
    ("troll_idle", "troll_walk"),
    ("cyplops_idle", "cyplops_walk"),
]
enemies = []
# Poções
potions = []
potion_image = "potion"

def spawn_enemy():
    img_idle, img_walk = random.choice(enemy_types)
    x = random.randint(50, WIDTH - 50)
    y = random.randint(50, HEIGHT - 50)
    enemy = AnimatedActor(img_idle, img_walk, (x, y))
    enemy.hp = 50
    enemies.append(enemy)

def spawn_potion():
    x = random.randint(50, WIDTH - 50)
    y = random.randint(50, HEIGHT - 50)
    potions.append(Actor(potion_image, (x, y)))

# Tile de fundo
tile_image = "cave_bg"
MAP_WIDTH = WIDTH // 64
MAP_HEIGHT = HEIGHT // 64

def update():
    global game_state, victory_time, defeat_time

    if game_state == "game":
        hero.update()

        elapsed = time.time() - start_time
        if elapsed >= TIME_LIMIT:
            "game_over"()

        for enemy in enemies[:]:
            enemy.update()
            if random.random() < 0.02:
                dx = random.choice([-32, 0, 32])
                dy = random.choice([-32, 0, 32])
                new_x = enemy.pos[0] + dx
                new_y = enemy.pos[1] + dy
                enemy.move_to((new_x, new_y))
                enemy.mode = "walk"
            else:
                enemy.mode = "idle"

            if enemy.get_rect().colliderect(hero.get_rect()):
                hero.hp -= 50
                enemies.remove(enemy)
                if hero.hp <= 0:
                    game_state = "defeat"
                    defeat_time = time.time()

        for potion in potions[:]:
            if potion.colliderect(hero.get_rect()):
                hero.hp = min(100, hero.hp + 50)
                potions.remove(potion)

        if not enemies:
            game_state = "victory"
            victory_time = time.time()

    elif game_state == "victory":
        if time.time() - victory_time > 5:
            reset_game()

    elif game_state == "defeat":
        if time.time() - defeat_time > 5:
            reset_game()

def draw():
    screen.clear()
    if game_state == "menu":
        draw_menu()
    elif game_state == "game":
        draw_game()
    elif game_state == "victory":
        draw_victory()
    elif game_state == "defeat":
        draw_defeat()

def draw_menu():
    screen.fill((30, 30, 30))
    for btn in menu_buttons:
        screen.draw.filled_rect(btn["rect"], (80, 80, 80))
        screen.draw.text(
            btn["label"],
            center=btn["rect"].center,
            fontsize=36,
            color="white"
        )

def draw_background():
    for x in range(MAP_WIDTH):
        for y in range(MAP_HEIGHT):
            screen.blit(tile_image, (x * 0, y * 0))

def draw_hp_bar():
    bar_width = 200
    bar_height = 20
    hp_percentage = max(hero.hp, 0) / 100
    filled = int(bar_width * hp_percentage)
    screen.draw.filled_rect(Rect((20, 20), (bar_width, bar_height)), (60, 60, 60))
    screen.draw.filled_rect(Rect((20, 20), (filled, bar_height)), (200, 0, 0))
    screen.draw.text(f"HP: {hero.hp}/100", (30, 22), fontsize=20, color="white")

def draw_inventory():
    screen.draw.text(f"Inventory: {', '.join(hero.inventory) or 'Empty'}", (20, 50), fontsize=20, color="white")

def draw_level():
    screen.draw.text(f"Level: {hero.level}", (20, 80), fontsize=20, color="white")

def draw_timer():
    remaining = max(0, TIME_LIMIT - int(time.time() - start_time))
    screen.draw.text(f"Time: {remaining}s", (WIDTH - 150, 20), fontsize=24, color="white")

def draw_game():
    draw_background()
    for enemy in enemies:
        enemy.draw()
    for potion in potions:
        potion.draw()
    hero.draw()
    draw_hp_bar()
    draw_inventory()
    draw_level()
    draw_timer()

def draw_victory():
    screen.fill((0, 0, 0))
    screen.draw.text("Congratulations, you survived!", center=(WIDTH//2, HEIGHT//2), fontsize=48, color="green")

def draw_defeat():
    screen.fill((0, 0, 0))
    screen.draw.text("Game Over", center=(WIDTH//2, HEIGHT//2), fontsize=48, color="red")

def on_mouse_down(pos):
    global game_state, sounds_on, start_time
    if game_state == "menu":
        for btn in menu_buttons:
            if btn["rect"].collidepoint(pos):
                if btn["label"] == "Start Game":
                    game_state = "game"
                    enemies.clear()
                    potions.clear()
                    hero.level = 1
                    hero.hp = 100
                    for _ in range(5):
                        spawn_enemy()
                    for _ in range(2):
                        spawn_potion()
                    start_time = time.time()
                    if sounds_on:
                        music.play(music_file)
                elif btn["label"] == "Toggle Sound":
                    sounds_on = not sounds_on
                    if not sounds_on:
                        music.stop()
                elif btn["label"] == "Quit":
                    exit()

def on_key_down(key):
    if game_state != "game":
        return

    dx, dy = 0, 0
    if key == keys.A:
        dx = -32
    elif key == keys.D:
        dx = 32
    elif key == keys.W:
        dy = -32
    elif key == keys.S:
        dy = 32
    elif key == keys.SPACE:
        attack()

    if dx or dy:
        new_x = hero.pos[0] + dx
        new_y = hero.pos[1] + dy
        hero.move_to((new_x, new_y))
        hero.mode = "walk"
        if sounds_on:
            try:
                sounds.move.play()
            except:
                pass
    else:
        if hero.mode != "attack":
            hero.mode = "idle"

def attack():
    hero.mode = "attack"
    for enemy in enemies[:]:
        dist = math.dist(hero.pos, enemy.pos)
        if dist <= ATTACK_RADIUS:
            enemies.remove(enemy)
            hero.level += 1
            hero.inventory.append("Loot")

def reset_game():
    global game_state
    game_state = "menu"
    hero.hp = 100
    hero.inventory = []
    hero.level = 1
    enemies.clear()
    potions.clear()
pgzrun.go()

