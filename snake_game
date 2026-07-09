"""
Nokia-Retro Snake
------------------
A Tkinter Snake game styled like an old Nokia LCD screen combined
with a modern neon dashboard. Built as an internship project.

Run:
    python snake_game.py

Controls:
    Arrow keys  -> move
    Space / P   -> pause / resume
    Enter       -> restart after game over
"""

import tkinter as tk
import random
import json
import os

try:
    import winsound
    HAS_WINSOUND = True
except ImportError:
    HAS_WINSOUND = False


# ---------------------------------------------------------------------------
# Config / theme
# ---------------------------------------------------------------------------
CELL = 24
COLS = 20
ROWS = 20
CANVAS_W = CELL * COLS
CANVAS_H = CELL * ROWS

BG_DARK = "#1a1a1a"
PANEL_BG = "#232323"
LCD_BG = "#9EA779"
LCD_LINE = "#8b9468"
LCD_TEXT = "#2f3d1c"
NEON_GREEN = "#39FF14"
NEON_GREEN_DARK = "#2a9e12"
NEON_ORANGE = "#ff5722"
NEON_ORANGE_GLOW = "#ffb199"
BTN_BG = "#333333"
BTN_FG = "#eeeeee"
ACCENT_FONT = ("Consolas", 12, "bold")
TITLE_FONT = ("Consolas", 40, "bold")

HERE = os.path.dirname(os.path.abspath(__file__))
HIGHSCORE_FILE = os.path.join(HERE, "highscore.json")

INITIAL_SPEED = 160     # ms per movement tick (lower = faster)
MIN_SPEED = 70
SPEED_STEP = 8
NEXT_ROUND_SCORE_STEP = 50   # points needed to level up / "next round"

DIRS = {
    "Up": (0, -1),
    "Down": (0, 1),
    "Left": (-1, 0),
    "Right": (1, 0),
}
OPPOSITE = {"Up": "Down", "Down": "Up", "Left": "Right", "Right": "Left"}


def load_high_score():
    try:
        with open(HIGHSCORE_FILE, "r") as f:
            return int(json.load(f).get("high_score", 0))
    except Exception:
        return 0


def save_high_score(score):
    try:
        with open(HIGHSCORE_FILE, "w") as f:
            json.dump({"high_score": score}, f)
    except Exception:
        pass


def beep(freq=880, dur=80):
    """8-bit style beep. Falls back to silence on non-Windows systems."""
    if HAS_WINSOUND:
        try:
            winsound.Beep(freq, dur)
        except Exception:
            pass


def make_button(parent, text, command, accent=False):
    return tk.Button(
        parent,
        text=text,
        command=command,
        font=ACCENT_FONT,
        bg=NEON_GREEN if accent else BTN_BG,
        fg="#0d0d0d" if accent else BTN_FG,
        activebackground=NEON_GREEN_DARK if accent else "#444444",
        activeforeground="#0d0d0d" if accent else BTN_FG,
        relief="flat",
        bd=0,
        padx=14,
        pady=8,
        cursor="hand2",
    )


# ---------------------------------------------------------------------------
# App
# ---------------------------------------------------------------------------
class SnakeApp(tk.Tk):
    def __init__(self):
        super().__init__()
        self.title("Snake")
        self.resizable(False, False)
        self.configure(bg=BG_DARK)

        self.high_score = load_high_score()

        container = tk.Frame(self, bg=BG_DARK)
        container.pack(padx=18, pady=18)

        self.cover_screen = CoverScreen(container, self)
        self.game_screen = GameScreen(container, self)
        self.howto_screen = HowToScreen(container, self)

        for screen in (self.cover_screen, self.game_screen, self.howto_screen):
            screen.grid(row=0, column=0, sticky="nsew")

        self.show_cover()

    # -- navigation -----------------------------------------------------
    def show_cover(self):
        self.high_score = load_high_score()
        self.cover_screen.refresh()
        self.cover_screen.tkraise()

    def show_howto(self):
        self.howto_screen.tkraise()

    def start_game(self):
        self.game_screen.new_game()
        self.game_screen.tkraise()
        self.game_screen.canvas.focus_set()


# ---------------------------------------------------------------------------
# Cover / title screen
# ---------------------------------------------------------------------------
class CoverScreen(tk.Frame):
    def __init__(self, parent, app):
        super().__init__(parent, bg=BG_DARK)
        self.app = app

        card = tk.Frame(self, bg="#0d0d0d", padx=14, pady=14)
        card.pack()

        self.canvas = tk.Canvas(
            card, width=CANVAS_W, height=CANVAS_H,
            bg="#2f3d1c", highlightthickness=0
        )
        self.canvas.pack()
        self._draw_background_blobs()

        title_x, title_y = CANVAS_W // 2, CANVAS_H // 2 - 30
        title_text = "SNAKE GAME"
        title_font = ("Consolas", 32, "bold")

        # Glow halo: several dim, offset copies drawn first (behind),
        # then the crisp bright copy drawn last (on top) -> soft neon glow.
        glow_offsets = [(-3, 0), (3, 0), (0, -3), (0, 3),
                         (-2, -2), (2, -2), (-2, 2), (2, 2)]
        for ox, oy in glow_offsets:
            self.canvas.create_text(
                title_x + ox, title_y + oy,
                text=title_text, fill=NEON_GREEN_DARK, font=title_font
            )
        self.canvas.create_text(
            title_x, title_y,
            text=title_text, fill=NEON_GREEN, font=title_font
        )

        self.hi_text_id = self.canvas.create_text(
            CANVAS_W // 2, CANVAS_H // 2 + 100,
            text="HIGH SCORE: 0", fill="#7d8f5a", font=("Consolas", 10)
        )

        btn_frame = tk.Frame(self, bg=BG_DARK, pady=14)
        btn_frame.pack()

        make_button(btn_frame, "START", app.start_game, accent=True).grid(
            row=0, column=0, padx=6
        )
        make_button(btn_frame, "HOW TO PLAY", app.show_howto).grid(
            row=0, column=1, padx=6
        )
        make_button(btn_frame, "EXIT", app.destroy).grid(
            row=0, column=2, padx=6
        )

    def _draw_background_blobs(self):
        blobs = [(30, 30, 26), (250 * CANVAS_W // 280, 60, 22),
                 (20, 300 * CANVAS_H // 340, 30), (255 * CANVAS_W // 280, 300 * CANVAS_H // 340, 26)]
        for x, y, r in blobs:
            self.canvas.create_oval(
                x - r, y - r, x + r, y + r, fill="#3f5223", outline=""
            )

    def refresh(self):
        self.canvas.itemconfig(
            self.hi_text_id, text=f"HIGH SCORE: {self.app.high_score}"
        )


# ---------------------------------------------------------------------------
# How to play screen
# ---------------------------------------------------------------------------
class HowToScreen(tk.Frame):
    def __init__(self, parent, app):
        super().__init__(parent, bg=BG_DARK)
        self.app = app

        card = tk.Frame(self, bg=PANEL_BG, padx=24, pady=24)
        card.pack(padx=20, pady=20)

        tk.Label(
            card, text="HOW TO PLAY", font=("Consolas", 18, "bold"),
            bg=PANEL_BG, fg=NEON_GREEN
        ).pack(pady=(0, 16))

        lines = [
            "Use the ARROW KEYS to steer the snake.",
            "Eat the glowing food to grow and score points.",
            "Every 50 points you level up (Next Round) — speed increases.",
            "Don't hit the walls or your own tail.",
            "Press SPACE or P to pause / resume.",
            "Press ENTER to restart after game over.",
        ]
        for line in lines:
            tk.Label(
                card, text=f"- {line}", font=("Consolas", 12),
                bg=PANEL_BG, fg=BTN_FG, anchor="w", justify="left"
            ).pack(fill="x", pady=4)

        make_button(card, "BACK", app.show_cover, accent=True).pack(pady=(20, 0))


# ---------------------------------------------------------------------------
# Game screen
# ---------------------------------------------------------------------------
class GameScreen(tk.Frame):
    def __init__(self, parent, app):
        super().__init__(parent, bg=BG_DARK)
        self.app = app

        top_bar = tk.Frame(self, bg=BG_DARK)
        top_bar.pack(fill="x", pady=(0, 10))

        self.score_label = tk.Label(
            top_bar, text="SCORE 0", font=ACCENT_FONT,
            bg=BG_DARK, fg=NEON_GREEN
        )
        self.score_label.pack(side="left")

        self.hi_label = tk.Label(
            top_bar, text="HI 0", font=ACCENT_FONT,
            bg=BG_DARK, fg=NEON_ORANGE
        )
        self.hi_label.pack(side="right")

        card = tk.Frame(self, bg="#0d0d0d", padx=14, pady=14)
        card.pack()

        self.canvas = tk.Canvas(
            card, width=CANVAS_W, height=CANVAS_H,
            bg=LCD_BG, highlightthickness=0
        )
        self.canvas.pack()

        control_bar = tk.Frame(self, bg=BG_DARK, pady=12)
        control_bar.pack()
        make_button(control_bar, "PAUSE", self.toggle_pause).grid(row=0, column=0, padx=5)
        make_button(control_bar, "RESTART", self.new_game).grid(row=0, column=1, padx=5)
        make_button(control_bar, "MENU", self.go_menu).grid(row=0, column=2, padx=5)

        self.canvas.bind("<KeyPress>", self.on_key)
        self.canvas.focus_set()

        self._draw_lcd_texture()

    # -- setup ------------------------------------------------------------
    def _draw_lcd_texture(self):
        for y in range(0, CANVAS_H, 4):
            self.canvas.create_line(
                0, y, CANVAS_W, y, fill=LCD_LINE, width=1, tags="texture"
            )

    def new_game(self):
        self.canvas.delete("game")
        mid_x, mid_y = COLS // 2, ROWS // 2
        self.snake = [(mid_x - 2, mid_y), (mid_x - 1, mid_y), (mid_x, mid_y)]
        self.direction = "Right"
        self.pending_direction = "Right"
        self.score = 0
        self.level = 0
        self.speed = INITIAL_SPEED
        self.paused = False
        self.game_over = False
        self.food = self._random_food()
        self._update_labels()
        self._tick()

    def go_menu(self):
        self.app.show_cover()

    # -- input --------------------------------------------------------------
    def on_key(self, event):
        key = event.keysym
        if key in DIRS:
            if key != OPPOSITE.get(self.direction):
                self.pending_direction = key
        elif key in ("space", "p", "P"):
            self.toggle_pause()
        elif key in ("Return",) and self.game_over:
            self.new_game()

    def toggle_pause(self):
        if self.game_over:
            return
        self.paused = not self.paused
        if self.paused:
            self.canvas.create_text(
                CANVAS_W // 2, CANVAS_H // 2,
                text="PAUSED", fill=LCD_TEXT, font=("Consolas", 22, "bold"),
                tags=("game", "pause_text")
            )
        else:
            self.canvas.delete("pause_text")
            self._tick()

    # -- game loop ------------------------------------------------------------
    def _random_food(self):
        while True:
            pos = (random.randint(0, COLS - 1), random.randint(0, ROWS - 1))
            if pos not in self.snake:
                return pos

    def _tick(self):
        if self.paused or self.game_over:
            return

        self.direction = self.pending_direction
        dx, dy = DIRS[self.direction]
        head_x, head_y = self.snake[-1]
        new_head = (head_x + dx, head_y + dy)

        if (
            new_head[0] < 0 or new_head[0] >= COLS or
            new_head[1] < 0 or new_head[1] >= ROWS or
            new_head in self.snake
        ):
            self._end_game()
            return

        self.snake.append(new_head)

        if new_head == self.food:
            self.score += 10
            beep(1200, 60)
            self.food = self._random_food()
            self._maybe_next_round()
        else:
            self.snake.pop(0)

        self._update_labels()
        self._render()
        self.after(self.speed, self._tick)

    def _maybe_next_round(self):
        new_level = self.score // NEXT_ROUND_SCORE_STEP
        if new_level > self.level:
            self.level = new_level
            self.speed = max(MIN_SPEED, INITIAL_SPEED - self.level * SPEED_STEP)
            beep(600, 120)
            text_id = self.canvas.create_text(
                CANVAS_W // 2, CANVAS_H // 2,
                text=f"NEXT ROUND {self.level + 1}",
                fill=LCD_TEXT, font=("Consolas", 18, "bold"),
                tags="game"
            )
            self.after(700, lambda: self.canvas.delete(text_id))

    def _end_game(self):
        self.game_over = True
        beep(200, 300)
        if self.score > self.app.high_score:
            self.app.high_score = self.score
            save_high_score(self.score)
        self._update_labels()
        self._render()
        self.canvas.create_text(
            CANVAS_W // 2, CANVAS_H // 2 - 20,
            text="GAME OVER", fill="#7a1f13", font=("Consolas", 22, "bold"),
            tags="game"
        )
        self.canvas.create_text(
            CANVAS_W // 2, CANVAS_H // 2 + 14,
            text="Press ENTER to restart", fill=LCD_TEXT,
            font=("Consolas", 11), tags="game"
        )

    # -- drawing ------------------------------------------------------------
    def _update_labels(self):
        self.score_label.config(text=f"SCORE {self.score}")
        self.hi_label.config(text=f"HI {self.app.high_score}")

    def _render(self):
        self.canvas.delete("game")

        points = []
        for (gx, gy) in self.snake:
            px = gx * CELL + CELL // 2
            py = gy * CELL + CELL // 2
            points.extend([px, py])

        if len(points) >= 4:
            self.canvas.create_line(
                *points, fill=NEON_GREEN, width=int(CELL * 0.75),
                capstyle=tk.ROUND, joinstyle=tk.ROUND, tags="game"
            )
            self.canvas.create_line(
                *points, fill="#e8f0c9", width=2, dash=(1, 10),
                capstyle=tk.ROUND, joinstyle=tk.ROUND, tags="game"
            )

        # head with eyes + tongue
        head_gx, head_gy = self.snake[-1]
        hx = head_gx * CELL + CELL // 2
        hy = head_gy * CELL + CELL // 2
        r = int(CELL * 0.55)
        self.canvas.create_oval(
            hx - r, hy - r, hx + r, hy + r,
            fill="#43ff20", outline="", tags="game"
        )
        dx, dy = DIRS[self.direction]
        perp = (-dy, dx)
        eye_off = r * 0.4
        for sign in (-1, 1):
            ex = hx + perp[0] * eye_off * sign + dx * (r * 0.2)
            ey = hy + perp[1] * eye_off * sign + dy * (r * 0.2)
            self.canvas.create_oval(
                ex - 2, ey - 2, ex + 2, ey + 2, fill="#0d2b06", outline="", tags="game"
            )
        tx = hx + dx * (r + 8)
        ty = hy + dy * (r + 8)
        self.canvas.create_line(
            hx + dx * r, hy + dy * r, tx, ty,
            fill=NEON_ORANGE, width=2, tags="game"
        )

        # food with soft glow
        fx = self.food[0] * CELL + CELL // 2
        fy = self.food[1] * CELL + CELL // 2
        gr = int(CELL * 0.55)
        self.canvas.create_oval(
            fx - gr, fy - gr, fx + gr, fy + gr,
            fill=NEON_ORANGE_GLOW, outline="", tags="game"
        )
        r2 = int(CELL * 0.32)
        self.canvas.create_oval(
            fx - r2, fy - r2, fx + r2, fy + r2,
            fill=NEON_ORANGE, outline="", tags="game"
        )


if __name__ == "__main__":
    SnakeApp().mainloop()