# Shmup
# Pygame template
import pygame
import random
from os import path

img_dir = path.join(path.dirname(__file__), "img")

WIDTH = 400
HEIGHT = 600
FPS =  60
POWERUP_TIME = 5000
score = 0

# define colors
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
RED = (255, 0, 0)
GREEN = (0, 255, 0)
BLUE = (0, 0, 255)
YELLOW = (255, 255, 0)

POWERUP_TIME = 5000

# Initialize pygame and window
pygame.init()
pygame.mixer.init()
wn = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Shmup(Shoot 'em up) @ 8 years old")
clock = pygame.time.Clock()

font_name = pygame.font.match_font("Geometric sans-serif")


def draw_text(surf, text, size, x, y):
    font = pygame.font.Font(font_name, size)
    text_surface = font.render(text, True, RED)
    text_rect = text_surface.get_rect()
    text_rect.midtop = (x, y)
    surf.blit(text_surface, text_rect)

def newmob():
	m = Mob()
	all_sprites.add(m)
	mobs.add(m)

def draw_shield_bar(surf, x, y, pct):
	if pct < 0:
		pct = 0
	BAR_LENGTH = 100
	BAR_HEIGHT = 10
	fill = (pct / 100) * BAR_LENGTH
	outline_rect = pygame.Rect(x, y, BAR_LENGTH, BAR_HEIGHT)
	fill_rect = pygame.Rect(x, y, fill, BAR_HEIGHT)
	pygame.draw.rect(surf, BLUE, fill_rect)
	pygame.draw.rect(surf, WHITE, outline_rect, 2)


class Player(pygame.sprite.Sprite):
    def __init__(self):
        pygame.sprite.Sprite.__init__(self)
        self.image = pygame.transform.scale(player_img,(50, 38))
        self.image.set_colorkey(BLACK)
        self.rect = self.image.get_rect()
        self.radius = 20
        self.rect.centerx = WIDTH / 2
        self.rect.bottom = HEIGHT - 10
        self.speedx = 0
        self.shield = 100
        self.shoot_delay = 250
        self.last_shot = pygame.time.get_ticks()
        self.power = 1
        self.power_time = pygame.time.get_ticks()

    def update(self):
    	self.speedx = 0
    	keystate = pygame.key.get_pressed()
    	if keystate[pygame.K_LEFT]:
            self.speedx = -7
    	if keystate[pygame.K_RIGHT]:
            self.speedx = 7
    	if keystate[pygame.K_SPACE]:
           	self.shoot()
    	self.rect.x += self.speedx
    	if self.rect.right > WIDTH:
    		self.rect.right = WIDTH
    	if self.rect.left < 0:
            self.rect.left = 0
    def powerup(self):
    	self.power += 1
    	self.power_time = pygame.time.get_ticks()

    def shoot(self):
    	now = pygame.time.get_ticks()
    	if now - self.last_shot > self.shoot_delay:
    		self.last_shot = now
    		if self.power == 1:
    			bullet = Bullet(self.rect.centerx, self.rect.top)
    			all_sprites.add(bullet)
    			bullets.add(bullet)
    		if self.power >= 2:
    			bullet1 = Bullet(self.rect.left, self.rect.centery)
    			bullet2 = Bullet(self.rect.right, self.rect.centery)
    			all_sprites.add(bullet1)
    			all_sprites.add(bullet2)
    			bullets.add(bullet1)
    			bullets.add(bullet2)

    		if self.power_time > POWERUP_TIME:
    			power_time = pygame.time.get_ticks()

class Mob(pygame.sprite.Sprite):
    def __init__(self):
        pygame.sprite.Sprite.__init__(self)
        self.image = meteor_img
        self.image.set_colorkey(BLACK)
        self.rect = self.image.get_rect()
        self.radius = int(self.rect.width * .85 / 2)
        self.rect.x = random.randrange(WIDTH - self.rect.width)
        self.rect.y = random.randrange(-100, -40)
        self.speedy = random.randrange(1, 8)
        self.speedx = random.randrange(-3, 3)



    def update(self):
    	self.rect.x += self.speedx
    	self.rect.y += self.speedy
    	if self.rect.top > HEIGHT + 10 or self.rect.left < -25 or self.rect.right > WIDTH + 20:
    		self.rect.x = random.randrange(WIDTH - self.rect.width)
    		self.rect.y = random.randrange(-100, -40)
    		self.speedy = random.randrange(1, 8)


class Bullet(pygame.sprite.Sprite):
    def __init__(self, x, y):
        pygame.sprite.Sprite.__init__(self)
        self.image = laser_img
        self.image.set_colorkey(BLACK)
        self.rect = self.image.get_rect()
        self.rect.bottom = y
        self.rect.centerx = x
        self.speedy = -15

    def update(self):
        self.rect.y += self.speedy
        # Kill it if it leaves the screen
        if self.rect.bottom < 0:
            self.kill()

class Pow(pygame.sprite.Sprite):
    def __init__(self, center):
        pygame.sprite.Sprite.__init__(self)
        self.type = random.choice(["shield", "gun"])
        self.image = powerup_images[self.type]
        self.image.set_colorkey(BLACK)
        self.rect = self.image.get_rect()
        self.rect.center = center
        self.speedy = 4

    def update(self):
        self.rect.y += self.speedy
        # Kill it if it leaves the screen
        if self.rect.top > HEIGHT:
            self.kill()

class Explosion(pygame.sprite.Sprite):
	def __init__(self, center, size):
		pygame.sprite.Sprite.__init__(self)
		self.size = size
		self.image = explosion_anim[self.size][0]
		self.rect = self.image.get_rect()
		self.rect.center = center
		self.frame = 0
		self.last_update = pygame.time.get_ticks()
		self.frame_rate = 75

	def update(self):
		now = pygame.time.get_ticks()
		if now - self.last_update > self.frame_rate:
			self.last_update = now
			self.frame += 1
			if self.frame == len(explosion_anim[self.size]):
				self.kill()
			else:
				center = self.rect.center
				self.image = explosion_anim[self.size][self.frame]
				self.rect = self.image.get_rect()
				self.rect.center = center



# Load all game graphics
background = pygame.image.load(path.join(img_dir, "starfield.png")).convert()
background_rect = background.get_rect()
player_img = pygame.image.load(path.join(img_dir, "Player.png")).convert()
laser_img = pygame.image.load(path.join(img_dir, "laserRed16.png")).convert()
meteor_img = pygame.image.load(path.join(img_dir, "meteorSmall.png")).convert()
powerup_images = {}
powerup_images["shield"] = pygame.image.load(path.join(img_dir, "shield_gold.png"))
powerup_images["gun"] = pygame.image.load(path.join(img_dir, "bolt_gold.png"))

explosion_anim = {}
explosion_anim["lg"] = []
explosion_anim["sm"] = []
explosion_anim["player"] = []
for i in range(9):
	filename = "sonicExplosion0{}.png".format(i)
	img = pygame.image.load(path.join(img_dir, filename)).convert()
	img.set_colorkey(BLACK)
	img_lg = pygame.transform.scale(img, (75, 75))
	explosion_anim["lg"].append(img_lg)
	img_sm = pygame.transform.scale(img, (32, 32))
	explosion_anim["sm"].append(img_sm)
	filename = "sonicExplosion0{}.png".format(i)
	img = pygame.image.load(path.join(img_dir, filename)).convert()
	explosion_anim["player"].append(img)

all_sprites = pygame.sprite.Group()
mobs = pygame.sprite.Group()
bullets = pygame.sprite.Group()
powerups = pygame.sprite.Group()
player = Player()
all_sprites.add(player)
for i in range(10):
    newmob()
# Game loop
running = True
while running:
    # keep loop running at the right speed.
    clock.tick(FPS)
    # Process input (events)
    for event in pygame.event.get():
        # check for closing window
        if event.type == pygame.QUIT:
            running = False

    	# Update
    all_sprites.update()

    # Check mb collision with Player
    hits = pygame.sprite.spritecollide(player, mobs, True, pygame.sprite.collide_circle)
    for hit in hits:
        player.shield -= hit.radius * 2
        expl = Explosion(hit.rect.center, "sm")
        all_sprites.add(expl)
        if player.shield <= 0:
        	death_explosion = Explosion(player.rect.center, "player")
        	all_sprites.add(death_explosion)
        	running = False

    # Check bl collision with mob
    hits = pygame.sprite.groupcollide(mobs, bullets, True, True)
    for hit in hits:
        score += 50 - hit.radius
        expl = Explosion(hit.rect.center, "lg")
        all_sprites.add(expl)
        if random.random() > 0.8:
        	pow = Pow(hit.rect.center)
        	all_sprites.add(pow)
        	powerups.add(pow)
        newmob()

        #Collision check of player and powerup
        hits = pygame.sprite.spritecollide(player, powerups, True, pygame.sprite.collide_circle)
        for hit in hits:
        	if hit.type == "shield":
        		player.shield += random.randrange(10, 30)
        	if player.shield >= 100:
        		player.shield = 100
        	if hit.type == "gun":
        		player.powerup()


    # Draw / render
    wn.fill(BLACK)
    wn.blit(background, background_rect)
    all_sprites.draw(wn)
    draw_text(wn, str(score), 18, WIDTH / 2, 10)
    draw_shield_bar(wn, 5, 5, player.shield)
    # after drawing everything, flip the display
    pygame.display.flip()
