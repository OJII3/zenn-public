---
title: "Neovim にグラフィカルな起動画面は要りませんか？"
emoji: "👋"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [linux, neovim]
published: false
---

# はじめに

Neovim とは、ターミナル上で動作するテキストエディタです。
豊富なプラグインを用いて設定をすると、VSCodeのような高機能エディタへと化けることができます。
使う理由は人により様々ですが、熱狂的な信者が多いですね笑。

私を含め、多くのユーザーはイケてるオレオレ設定を作る遊びをしているのですが、
せっかく設定したので一部紹介しようと思い記事にしました。
Neovim 遊びに興味を持ってくれたら嬉しいです。

# 今回のテーマ

Neovim が起動したときの画面をグラフィカルにしてかっこよくする↓

比較対象として、デフォルトの起動画面はこんな感じです。

# 必要なもの

- OS: Linux Desktop または MacOS
- ターミナルエミュレーター: [Kitty](https://github.com/kovidgoyal/kitty)
- エディタ: Neovim
- その他: ImageMagick

ターミナル上に画像を表示する都合上、環境の制約が厳しくなっています。
詳細は以下のページをご覧ください [3rd/image.nvim](https://github.com/3rd/image.nvim) 

# カスタマイズ

プラグインマネージャーには **Lazy.nvim** を使用しています。

## Image.nvim

まずは Neovim で画像を表示するプラグイン **3rd/image.nvim** の導入です。

Kitty 以外の環境でも同じ設定ファイルを使いたいので、以下の式を使い、このプラグインの有効・無効を自動で切り替えるようにします。

```lua
os.getenv("TERM") == "xterm-kitty" -- kitty であれば true
```

Image.nvim には画像を扱うためのAPIが用意されており、以下のように位置を指定して使うことができます。

```lua
local api = require("image")
                        local image = api.from_file("/path-to-image.png", {
                                x = math.floor(vim.api.nvim_win_get_width(0) / 2 - 50), // 中央に配置
                                y = 10,
                                width = 100,
                        })
```

このようにして画像を読み込んで


```lua
image:show()
image:clear()
```

のようにして簡単に表示非表示を切り替えられます。

## Alpha.nvim

次に、起動画面をカスタマイズする設定です。**Alpha.nvim** を併用しています。

**image.nvim** はAPIを提供しているので、任意のタイミングで任意の場所に画像を出すことができます。

**Alpha.nvim** 起動時の Autocmd で画像表示を行い、`BufEnter` の Autocmd で画像をクリアします。`AlphaClosed` という Autocmd もあるのですが、こちらだとバッファをフロートで開いたときには発火しないようで、重なって見づらくなってしまうので使用しませんでした。

```lua
local image = ...
...

vim.api.nvim_create_autocmd({ "User" }, {
                                        callback = function()
                                                image:render()
                                        end,
                                        pattern = "AlphaReady",
                                })

                                vim.api.nvim_create_autocmd({ "BufEnter" }, {
                                        callback = function()
                                                image:clear()
                                        end,
```

先程と同様、Kitty がある時とないときで処理を分岐させています。

```lua title="~/.config/nvim/lua/plugins/alpha.lua"
return {
        "goolord/alpha-nvim",
        dependencies = {
                { "nvim-tree/nvim-web-devicons" },
                { "3rd/image.nvim", opts = true, lazy = false },
        },
        config = function()
                local alpha = require("alpha")

                vim.api.nvim_set_hl(0, "CustomAlphaHeader", { fg = "#00ffff", bold = true })
                vim.api.nvim_set_hl(0, "CustomAlphaFooter", { fg = "#00ffff", bold = true })
                local dashboard = require("alpha.themes.dashboard")
                dashboard.section.header.val = {}
                dashboard.section.header.opts.hl = "CustomAlphaHeader"
                dashboard.section.footer.opts.hl = "CustomAlphaFooter"
                dashboard.section.header.opts.position = "center"
                vim.api.nvim_set_hl(0, "CustomAlphaHeader", { fg = "#039393", bold = true })
                dashboard.section.buttons.val = {}

                if os.getenv("TERM") == "xterm-kitty" then
                        local api = require("image")
                        local image = api.from_file("/home/ojii3/dotfiles/images/Bronie_Haxxor_Bunny_M.png", {
                                x = math.floor(vim.api.nvim_win_get_width(0) / 2) - 50,
                                y = 10,
                                width = 100,
                        })
                        if image ~= nil then
                                -- dashboard.section.buttons.val = {}
                                dashboard.section.header.val = {
                                        [[        .;        ;.                                                                    ]],
                                        [[      .,l,'.      ;l;.                                           ;;;                    ]],
                                        [[     ;odl;,,'     ;bcc:                                                                 ]],
                                        [[     cddl;,;:,.   ;bccc.    :l.:'',.   :'''l.  .:''':. ox;   .dx.dx, ox:odxc'ldxxl.     ]],
                                        [[     cccl;,;:;,.  ;llll.    :d     b .c     ;..c     ;:.kk.  dO' kO; xO:  ;OO'  oOc     ]],
                                        [[     cccl; .;;;;, ;llll.    :l     d.;d.....d.c'      d .kx lO:  kO; xO,  'OO.  cOl     ]],
                                        [[     lccl;  .;;;;;;looo.    :l     d..c       ';     .l  ;OdOo   kO; xO,  'OO.  cOl     ]],
                                        [[     llll,    ,:;:;ldoo.    :l     d  '':..:'  ''...''    cdo    dx, x0'  .00.  :0l     ]],
                                        [[     clll;     '::;ldoo.                                                                ]],
                                        [[      .co;      .:;dl,                                                                  ]],
                                        [[        .;        ;'                                                                    ]],
                                }

                                vim.api.nvim_create_autocmd({ "User" }, {
                                        callback = function()
                                                image:render()
                                        end,
                                        pattern = "AlphaReady",
                                })

                                vim.api.nvim_create_autocmd({ "User" }, {
                                        callback = function()
                                                image:clear()
                                        end,
                                        pattern = { "AlphaClosed" },
                                })
                                vim.api.nvim_create_autocmd({ "BufEnter" }, {
                                        callback = function()
                                                image.clear(image)
                                        end,
                                        pattern = { "*" },
                                })

                                -- dashboard.config.opts.noautocmd = true
                                vim.cmd([[
          autocmd User AlphaReady echo 'Ready!'
        ]])
                                alpha.setup(dashboard.config)
                                return
                        end
                end
                dashboard.section.buttons.val = {}
                dashboard.section.header.val = {
                        [[                                                          ]],
                        [[                                                          ]],
                        [[         some ascii art or something here                 ]],
                        [[                                                          ]],
                }

                dashboard.config.opts.noautocmd = true
                vim.cmd([[
      autocmd User AlphaReady echo 'Ready!'
    ]])
                alpha.setup(dashboard.config)
        end,
}
```


