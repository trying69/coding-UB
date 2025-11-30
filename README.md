# coding-UB
import curses
from random import choice, randrange

# ========= 2048 GAME LOGIC ==========
class Game2048:
    def _init_(self, size=4):
        self.size = size
        self.reset()

    def reset(self):
        self.board = [[0] * self.size for _ in range(self.size)]
        self.spawn()
        self.spawn()

    def spawn(self):
        # Add a 2 or 4 at an empty location
        empty_cells = [(r, c) for r in range(self.size)
                       for c in range(self.size) if self.board[r][c] == 0]
        if empty_cells:
            r, c = choice(empty_cells)
            self.board[r][c] = 4 if randrange(10) == 0 else 2

    def compress(self, row):
        new_row = [num for num in row if num != 0]
        new_row += [0] * (self.size - len(new_row))
        return new_row

    def merge(self, row):
        for i in range(self.size - 1):
            if row[i] != 0 and row[i] == row[i + 1]:
                row[i] *= 2
                row[i + 1] = 0
        return row

    def move_left(self):
        moved = False
        new_board = []
        for row in self.board:
            compressed = self.compress(row)
            merged = self.merge(compressed)
            final = self.compress(merged)
            new_board.append(final)
            if final != row:
                moved = True
        self.board = new_board
        return moved

    def move_right(self):
        self.board = [row[::-1] for row in self.board]
        moved = self.move_left()
        self.board = [row[::-1] for row in self.board]
        return moved

    def move_up(self):
        self.board = list(map(list, zip(*self.board)))
        moved = self.move_left()
        self.board = list(map(list, zip(*self.board)))
        return moved

    def move_down(self):
        self.board = list(map(list, zip(*self.board)))
        moved = self.move_right()
        self.board = list(map(list, zip(*self.board)))
        return moved

    def can_move(self):
        for r in range(self.size):
            for c in range(self.size):
                if self.board[r][c] == 0:
                    return True
                if c + 1 < self.size and self.board[r][c] == self.board[r][c+1]:
                    return True
                if r + 1 < self.size and self.board[r][c] == self.board[r+1][c]:
                    return True
        return False


# ========= CURSES UI ==========
def draw_board(stdscr, game):
    stdscr.clear()
    h, w = stdscr.getmaxyx()

    # Draw title
    title = "2048 GAME - Use Arrow Keys, Q to Quit"
    stdscr.addstr(1, (w - len(title)) // 2, title)

    # Draw board grid
    for r in range(game.size):
        for c in range(game.size):
            value = game.board[r][c]
            box_h = 4
            box_w = 8
            y = 3 + r * box_h
            x = (w - game.size * box_w) // 2 + c * box_w

            # Draw cell border
            for i in range(box_w):
                stdscr.addch(y,     x+i, '-')
                stdscr.addch(y+3,   x+i, '-')
            for i in range(box_h):
                stdscr.addch(y+i, x,   '|')
                stdscr.addch(y+i, x+7, '|')

            # Draw number centered
            if value != 0:
                num_str = str(value)
                stdscr.addstr(y+1, x+(7-len(num_str))//2 + 1, num_str)

    stdscr.refresh()


def main(stdscr):
    curses.curs_set(0)
    stdscr.nodelay(False)
    stdscr.keypad(True)

    game = Game2048()

    while True:
        draw_board(stdscr, game)

        key = stdscr.getch()

        if key == curses.KEY_LEFT:
            moved = game.move_left()
        elif key == curses.KEY_RIGHT:
            moved = game.move_right()
        elif key == curses.KEY_UP:
            moved = game.move_up()
        elif key == curses.KEY_DOWN:
            moved = game.move_down()
        elif key in (ord('q'), ord('Q')):
            break
        else:
            moved = False

        if moved:
            game.spawn()

        if not game.can_move():
            stdscr.clear()
            msg = "GAME OVER! Press Q to Quit"
            h, w = stdscr.getmaxyx()
            stdscr.addstr(h // 2, (w - len(msg)) // 2, msg)
            stdscr.refresh()

            while True:
                if stdscr.getch() in (ord('q'), ord('Q')):
                    return


if _name_ == "_main_":
    curses.wrapper(main)
