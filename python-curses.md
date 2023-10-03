Python `curses`
++++++++++++++

```python
import curses

def main(win):
    win.clear()
    win.addstr("Press any key to continue...")
    win.refresh()
    win.getch()

curses.wrapper(main)
```