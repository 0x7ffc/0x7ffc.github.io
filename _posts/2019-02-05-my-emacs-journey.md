---
layout: post
title: "My Emacs Journey"
description: "A journey of me writing .emacs.d . I use Dvorak keyboard layout and evil-mode which makes my experience challenging and interesting."
---

There are some problems while I'm using spacemacs.

- I use Dvorak keyboard layout.
- Spacemacs and other popular configs are oriented toward Qwerty layout.
- I use evil mode which basically is vim inside emacs and it uses "HJKL".
- I want to use "HTNS" instead of "HJKL" on Dvorak.
- I want all my key bindings to accommodate Dvorak layout.
- Spacemacs and most other configs are too "heavy" for me.

These problems are getting bigger and my Spacemacs config are getting messier. So I decided to write [my own `.emacs.d`](https://github.com/0x7ffc/lain-emacs). After writing some code I really feel like that everyone should write their own Emacs configuration, here is why:

- Different people emphasize on different stacks, you don\'t need EVERYTHING.
- Some prefer **emacs** key bindings while others prefer **vim** key bindings.
- There EXIST people who use keyboard layout other than QWERTY.
- The problem with large configuration like **spacemacs** is the same with any complex enough IDE:
  - Their is a learning curve before you can be productive.
  - Even you think you are *productive*, you still just use, like 10% of what it offers.
  - Configuration generally separates into two parts, the default part, or **core** if you prefer, and more flexible part, or **module**. And to be honest, I find it difficult to distinguish them. Because you still need to change the configuration if you want to tweak it no matter which part it from. Code is code.

I know that my problem is unique that not everybody is using emacs with
evil-mode on a dvorak keyboard. But after writing some lisp myself. I
find it quite satisfying and enjoyable:

- Emacs Lisp is extremely well documented.
  - `C-h k` documentation for a key stroke
  - `C-h f` documentation for a function
  - `C-h v` documentation for a variable
  - `C-h b` list of all keybindings available in current buffer
  - `C-h S` search symbol in Emacs manual
- Emacs ecosystem has advanced a lot since version 22/23. With proper tools one can easily replicated what minimal spacemacs offers.
  - `use-package` keep your code tidy and fast
  - `general.el` drop-in replacement for various keybinding methods for both stock emacs an evil-mode
  - `evil-mode` use Vim inside Emacs
  - `which-key` display key bindings following your currently entered incomplete command in a popup window
- There are a lot of great tutorials and source codes out there that helps you write anything you want.
  - *The offical GUN Emacs manual*
  - [elisp-guide](https://github.com/chrisdone/elisp-guide)
  - [evil-guide](https://github.com/noctuid/evil-guide)
  - [A list of people with nice emacs config files](https://github.com/caisah/emacs.dz)
  - [How to build your own spacemacs](https://sam217pa.github.io/2016/09/02/how-to-build-your-own-spacemacs/)
  - [How to Write Fast(er) Emacs Lisp](https://nullprogram.com/blog/2017/01/30/)
  - [How is Doom's startup so fast?](https://github.com/hlissner/doom-emacs/wiki/FAQ#how-is-dooms-startup-so-fast)

I also read a lot `Doom`, `Prelude` and `Spacemacs` source code. I learned a lot even just by copy-pasting their code. I check every variable and every function I see with `C-h` so that I know what they do.

Actually, Spacemacs use `use-package` an `which-key` internally. I think with this approach the ideal configuration should look something like this:

![code emacs-lisp use-packages](/assets/images/use-package-fold.png)

Just bunch of `use-package` forms.

That being said, there are some minor issues:

- `use-package` doesn\'t offer a convenient why to write code dependent on multiple packages, here is an [issue](https://github.com/jwiegley/use-package/issues/315) that describes it. `Spacemacs` solves it by create it\'s own [layer loading mechanism](https://github.com/syl20bnr/spacemacs/blob/develop/doc/LAYERS.org). but it\'s excessive as long as you aren\'t writing that much code. You can just use our old friend `with-eval-after-load`. Or don\'t use anything, just make sure your packages are loaded in order.
- Need quite some time to figure out how to glue these five packages together. I haven\'t found one single `.emacs.d` on github that uses all five package, let alone one with **DVORAK** keybindings support (I can\'t type on QWERTY).
- Need to RTFM. Like, a LOT. And some of them are confusing, like buffers and windows.

Once having a basic setup, one can extend it with ease and efficiency. By saying basic setup, I meant to write Emacs Lisp comfortably:

- basic theme/font/ui setup that doesn\'t give me eyesore
- a package manager that doesn\'t suck, e.g. `straight.el`, `el-get`, `quelpa` ...
- an autocomplete framework, e.g. `company`, `auto-complete`
- a completion framework, e.g. `ivy`, `helm`
- a lisp **Structural Editing** framework, e.g. `smartparens`, `paredit`
- a lisp **Structural Editing** framework that works with `evil-mode`, e.g. `evil-cleverparen`

Then, just extend Emacs with bunch of `use-package` forms.

Anyway, here I am trying to achieve this. My configuration is optimized for DVORAK keyboard. You may not be using DVORAK, but apart from that it\'s still a good reference to get started hacking Emacs. It\'s using \"htns\" instead of \"hjkl\" to move around. It also tries to remap possible \"C-j\" \"C-k\" \"C-n\" \"C-p\" to \"C-t\" \"C-n\". And it works with `evil-mode`. It has a fast startup time, but I usually use emacs daemon (see scripts directory). Check the tasks list for what I\'ve been doing and what will be implemented in the future.

Hope you find my experience helpful.

## The Workflow

I guess not everybody is gonna read all the code (not too much TBH), so
I\'ll describe what my typical workflow looks like (keep in mind that
this is entry level stuff, don\'t laugh at me):

- Run `ec` in terminal to fire up Emacs server and connect to it (export PATH=\"\$PATH:\$HOME/.emacs.d/scripts/).
- Switch to a project using `SPC p p`, or `SPC f f` to navigate to a file in a new project.
- Use `SPC p f` to find file in a project.
- In case of `projectile` couldn\'t find a newly created file or still showing deleted file, run `SPC p I`
- File related keybindings are in `SPC f`, e.g.
  - `SPC f f` get a list of files in current directory to open with
  - `SPC f D` delete current file and its buffer
  - `SPC f R` rename current file and its buffer
  but I usually find myself using `ranger` to manage file (press `-`)
- Buffer related keybindings are in `SPC b`, e.g.
  - `SPC b b` get a list of buffers to switch to
  - `SPC b d` kill current buffer, but its window is still there
  - `SPC b x` kill current buffer and its window
  - `SPC b D` get a list of buffers and choose one to kill
  - `SPC b t` next buffer
  - `SPC b n` previous buffer
- Window related keybindings are in `SPC w`, e.g.
  - `SPC w 2` split window vertically
  - `SPC w 3` split window horizontally
  - `SPC w h/t/n/s` move to the left/down/up/right window
  - `SPC 1/2.../9` switch to a window by number
  - `SPC w d` delete current window
- While editing a file
  - `C-s` to search text in current buffer
  - `SPC /` to search text in current project (using `rg`)
  - `:%s/from/to/g` to find and replace text in current buffer
- After editing some file, I fire up magit: `SPC g s`
  - `gu` go to the unstaged changes
  - `s y` stage all the changes
  - `c c` write my commit message and \"C-c C-c\"
  - `P p` push to origin
  - `q` quit magit
- `C-x C-c` or `SPC e q` to exit Emacs

Some editing notes:

- Parentheses are paired using `smartparens`, and `evil-cleverparens` to provide evil integration. Some keybindings I use most:
  - `M-(` wrap an expression in parentheses
  - `M-a` insert at end of an expression
  - `M-i` insert at beginning of an expression
  - `M-r` raise an expression
  - `M-s` splice an expression
  - `<` and `>` to slurp expression
  - `_` move to the first non opening character
  - `dd` will not break parenthesis and keep our s-expression correct
- `evil-commentary` add comment operator, e.g.
  - `gcap` to comment current paragraph
- `evil-surround` can emulates surround.vim, e.g.
  - `ysW"` to wrap to word with `"`
  - `csW"(` change surrounding of a word from `"` to `(`
- `evil-lion` add align text operator, e.g.
  - `glap'` to align current paragraph using `'`
- `expand-region` is integrated with evil. For example, in the string (hello \"foo\| oo\"):
  - double press `v` it will select \"foo\"
  - then \"\\\"foo\\\"\"
  - then \"hello \\\"foooo\\\"\"
  - then the whole expression with the parenthesis.
- If you\'ve seen [emacsrocks episode1](http://emacsrocks.com/e01.html), you may wonder how to do this in evil-mode
  - in normal state press `C-v` which calls `evil-visual-block`
  - move to the space before `l` however you want (avy isearch swiper)
  - press `R`, now anything typed will only show up on the first line, but when one returns to normal state, by pressing ESC, then the typed characters will appear on each line of the block/rectangle.
- ivy is integrated with wgrep, so you can edit your search result:
  - while searching with counsel-rg/swiper, press `C-c C-o` to run `ivy-occur`, it\'ll bring up a new buffer with all search result.
  - then press `w` to enter editable state if you want to edit it, at last press `C-c C-c` to save or `C-c C-k` to abort.

## Note about SPEED

Emacs will initialize tool-bar/menu-bar even if you have disabled them in your configuration, to avoid this:

```shell
cp ~/.emacs.d/.Xresources.example ~/.Xresources
xrdb ~/.Xresources
```
You may want to put last line in your zshrc or bashrc.

## Note about RSI

You are gonna use your pinky a lot, and this will result in RSI if not handled properly. To avoid this, first I recommend reading xah\'s blog [\"How to Avoid Emacs Pinky\"](http://ergoemacs.org/emacs/emacs_pinky.html).

Now here are some additional methods which are not mentioned in the blog post:

- If you\'re using a thinkpad as I do, remap the two keys above touchpad and below spacebar to the keys you like.
- Use spacebar as control: Pressing and releasing results in space as normal, but if held while pressing other keys it acts like control. You can achieve this by using xcape if you\'re on linux and karabiner if on OSX. Both provide additional features like generate the `Escape` key when `Left Control` is pressed and released on its own, it plays nicely with evil-mode(Vim). Also I\'m on WSL, it works fine with X server.

If these two doesn\'t suit you, I think at least you should swap keys around, or use sticky keys....

## External Programs

You\'ll also need these.

- ripgrep
- fira code
  - [symbol](https://github.com/tonsky/FiraCode/files/412440/FiraCode-Regular-Symbol.zip) for prettify-symbols-mode (ligature)
  - actual font
- fd

Good luck my fellow Emacsers, happy hacking!