[![MELPA](https://melpa.org/packages/run-command-badge.svg)](https://melpa.org/#/run-command)

# run-command

**Leave Emacs less**. Relocate those frequent shell commands to configurable, dynamic, context-sensitive lists, and run them at a fraction of the keystrokes via Helm or Ivy.

<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-refresh-toc -->

Table of Contents:

- [Demo](#demo)
- [Installing](#installing)
- [Quickstart](#quickstart)
- [Configuring](#configuring)
- [Invoking](#invoking)
- [Tutorial: adding commands](#tutorial-adding-commands)
  - [Readable command names](#readable-command-names)
  - [Specifying the working directory](#specifying-the-working-directory)
  - [Enabling and disabling depending on context](#enabling-and-disabling-depending-on-context)
  - [Generating commands on the fly](#generating-commands-on-the-fly)

<!-- markdown-toc end -->

## Demo

The screencast below shows using `run-command` to 1) create a project from a boilerplate, 2) execute a file on every save, and 3) start the test runner.

<p align="center"><img alt="demo" src="./demo.gif"></p>

## Installing

[Available from MELPA](https://melpa.org/#/run-command).

## Configuring

By default, commands are run in `compilation-mode`. See [Lightweight external command integration in Emacs via compilation mode](https://massimilianomirra.com/notes/lightweight-external-command-integration-in-emacs-via-compilation-mode/) for some notes on how to make the most of `compilation-mode`.

Alternatively (and experimentally), commands can be run in `term-mode` plus `compilation-minor-mode`, especially useful for commands with rich output such as colors, progress bars, and screen refreshes, while preserving `compilation-mode` functionality. Set `run-command-run-method` to `term` and please comment on [issue #2](https://github.com/bard/emacs-run-command/issues/2) if you find issues.

The auto-completion framework is automatically detected. It can be set manually by customizing `run-command-completion-method`.

## Quickstart

1. Add a "command recipe" to your init file, for example:

```emacs-lisp
(defun run-command-recipe-local ()
  (list
   (list :command-name "say-hello"
         :command-line "echo Hello, World!")
   (list :command-name "serve-http-dir"
         :command-line "python3 -m http.server 8000")
   (when (equal (buffer-name) "README.md")
     ;; uses https://github.com/joeyespo/grip
     (list :command-name "preview-github-readme"
           :command-line "grip --browser --norefresh"))))
```

2. Customize `run-command-recipes` and add `run-command-recipe-local` to the list.

3. Type `M-x run-command RET`.

Read more about [configuration](#configuring), [invocation](#invoking), and [how to add commands](#tutorial-adding-commands), or check out some [recipe examples](./examples).

## Invoking

Type `M-x run-command` or bind `run-command` to a key:

```emacs-lisp
(global-set-key (kbd "C-c c") 'run-command)
```

Or:

```emacs-lisp
(use-package run-command
  :bind ("C-c c" . run-command)
```

When using Helm, you can edit a command before running it by typing `C-u RET` instead of `RET`. (See [issue #1](https://github.com/bard/emacs-run-command/issues) if you can help bring that to Ivy.)

## Tutorial: adding commands

### Readable command names

To provide a more user-friendly name for a command, use the `:display` property:

```emacs-lisp
(defun run-command-recipe-local ()
  (list
   (list :command-name "serve-http-dir"
         :command-line "python3 -m http.server 8000"
         :display "Serve directory over HTTP port 8000")))
```

### Specifying the working directory

A command runs by default in the current buffer's directory. You can make it run in a different directory by setting `:working-dir`.

For example, you want to serve the current directory via HTTP, unless you're visiting a file that is somewhere below a `public_html` directory, in which case you want to serve `public_html` instead:

```emacs-lisp
(defun run-command-recipe-local ()
  (list
   (list :command-name "serve-http-dir"
         :command-line "python3 -m http.server 8000"
         :display "Serve directory over HTTP port 8000"
         :working-dir (let ((project-dir
                             (locate-dominating-file default-directory "public_html")))
                        (if project-dir
                            (concat project-dir "public_html")
                          default-directory)))))
```

See the [Hugo project recipe](examples/run-command-recipe-hugo.el) for a recipe that uses the project's directory for all commands.

### Enabling and disabling depending on context

To disable a command in certain circumstances, make its recipe return `nil` in its place.

For example, you want to enable a command only when a buffer's file is executable:

```emacs-lisp
(defun run-command-recipe-local ()
  (let* ((buffer-file (buffer-file-name))
         (executable-p (and buffer-file (file-executable-p buffer-file))))
    (list
     (if executable-p
         (list
          :command-name "run-buffer-file"
          :command-line buffer-file
          :display "Run this buffer's file")
       nil))))
```

See the [executable file recipe](examples/run-command-recipe-executables.el) for a variant that also re-runs the file on each save.

### Generating commands on the fly

Recipes are plain old Lisp functions, so they generate commands based on e.g. project setup.

See the [NPM project recipe](examples/run-command-recipe-package-json.el), which uses a JavaScript's project `package.json` file to generate commands, and the [Make project recipe](examples/run-command-recipe-make.el), which does the same for `Makefile` projects.
