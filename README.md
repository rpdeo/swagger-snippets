# yaml-mode snippets for Swagger

Snippet files supporting writing of [Swagger](https://swagger.io)/OpenAPI spec API definition files in Emacs with Yasnippet.

Swagger already provides [swagger-editor](https://swagger.io/swagger-editor/) which is a browser-based editor to edit Swagger spec files. Easiest way to get it going on your local machine is with docker. But we are Emacs addicts, hence this repository (sigh).

`yaml-mode` is convenient in `Emacs` to edit Swagger specification files.
With addition of `outline-minor-mode` for tree folding and unfolding within a yaml-mode buffer and snippet files from this repo, editing Swagger spec is a breeze. The snippet files take inspiration from swagger-editor's editor-autosuggest-snippets plugin and in few cases extends them.

Feel free to suggest bug fixes or further additions, if this is useful to you.

# How to configure Emacs so this is useful

My `emacs` `yaml-mode` config is as below. As explained `outline-minor-mode`
helps with folding/unfolding of subtrees (Swagger files will get large even
for trivial APIs).

``` elisp
(use-package yaml-mode
  :defer t
  :config
  (defconst yaml-outline-regex
    (concat "\\( *\\)\\(?:\\(?:--- \\)?\\|{\\|\\(?:[-,] +\\)+\\) *"
            "\\(?:" yaml-tag-re " +\\)?"
            "\\(" yaml-bare-scalar-re "\\) *:"
            "\\(?: +\\|$\\)")
    "Regexp matching a single YAML hash key. This is adds a
    capture group to `yaml-hash-key-re' for the
    indentation.")

  (defun yaml-outline-level ()
    "Return the depth to which a statement is nested in the outline."
    (- (match-end 1) (match-beginning 1)))

  (defun my-yaml-mode-hook()
    (outline-minor-mode)
    (define-key yaml-mode-map (kbd "<backtab>") 'outline-toggle-children)
    (define-key yaml-mode-map (kbd "C-c x a") 'outline-hide-subtree)
    (define-key yaml-mode-map (kbd "C-c x e") 'outline-show-subtree)
    (setq-local outline-regexp yaml-outline-regex)
    (setq-local outline-level #'yaml-outline-level)
    (yas-minor-mode))

    ;; one might use orgstruct-mode also, but it has not worked for me yet for yaml.

  (add-hook 'yaml-mode-hook #'my-yaml-mode-hook))
```

My `yasnippet` configuration is as below, only important point is to point `yas-snippet-dirs` to snippets for swagger. Files from this repo reside under `~/.emacs.d/snippets/yaml-mode/...` for me.

``` elisp
(require 'yasnippet)
(setq yas-snippet-dirs
        '("~/.emacs.d/snippets/" ;; personal snippets
          "~/.emacs.d/snippets/yasnippet-snippets" ;; https://github.com/AndreaCrotti/yasnippet-snippets.git collection
          ))
;; load yas for prog-modes
(yas-reload-all)
(add-hook 'prog-mode-hook #'yas-minor-mode)
;; avoid clashes with TAB
(define-key yas-minor-mode-map (kbd "<tab>") nil)
(define-key yas-minor-mode-map (kbd "TAB") nil)
(define-key yas-minor-mode-map (kbd "SPC") yas-maybe-expand)
(define-key yas-minor-mode-map (kbd "C-c y") #'yas-expand)
```

# Plan

- Add swagger backend for company-mode for keyword/enum completions, for complete spec support in emacs.
