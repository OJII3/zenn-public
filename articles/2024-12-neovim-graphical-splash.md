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

まずは Neovim で画像を表示するプラグイン **3rd/image.nvim** の導入です。

```lua title="~/.config/nvim/lua/plugins/image.lua"
return {
        "3rd/image.nvim",
        enabled = os.getenv("TERM") == "xterm-kitty",
        dependencies = {
                { "MunifTanjim/nui.nvim" },
        },
  lazy = false,
}
```

Kitty のない環境でも同じ設定ファイルを使えるように工夫しています。

次に、起動画面をカスタマイズする設定です。**Alpha.nvim** を併用しています。

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
                        [[                                                                                                                        ......:o'.                               ]],
                        [[                                                                                                                     c:,,,;',cxxlc,                              ]],
                        [[                                                                                                                    '',;,,...,dol::,                             ]],
                        [[                                                                                                                   ,l''.'',;''00k;'''                            ]],
                        [[                                                                                                                  .dc:cc.  ,;lkk,'....                           ]],
                        [[                                                                                                                  'll;cc..  .ll.  '....                          ]],
                        [[                                                                                                                 .':l,.,,.,:,'c.  ......                         ]],
                        [[                                                                                                                ...',, .:dd,,'.    '.....                        ]],
                        [[                                                                                                                ...'., 'llcc;.. .  .'.....                       ]],
                        [[                                                                                                               ..',',;'':;,.;;. .  .;''....                      ]],
                        [[                                                                                                               .'cc:xo,:d::;dxddo   ,:'.....                     ]],
                        [[                                                                                                              ..lodko.,lco:;,o0ko   .lc'....                     ]],
                        [[                                                                                                               :oox:..xc,;;. ;00x    ;oc..  .                    ]],
                        [[                      .;        ;.                                                                          . ;dxxkd'lxxc;l. ,00O.   .lo'.   .                   ]],
                        [[                    .,l,'.      ;l;.                                           ;;;                         ..;l::;,;:lcdl,:c.;0Ok:    ;lc....  ;                 ]],
                        [[                   ;odl;,,'     ;bcc:                                                                     ..,l';l:::oo:'.ldc;d0xdk    .co,..... .                ]],
                        [[                   cddl;,;:,.   ;bccc.    :l.:'',.   :'''l.  .:''':. ox;   .dx.dx, ox:odxc'ldxxl.         .'dx;kKxOKO;;:,:d:cxXx:O;    ,dc........               ]],
                        [[                   cccl;,;:;,.  ;llll.    :d     b .c     ;..c     ;:.kk.  dO' kO; xO:  ;OO'  oOc        ',kOdx0OKXXKx,.'ck000XOcxx    .lo'........              ]],
                        [[                   cccl; .;;;;, ;llll.    :l     d.;d.....d.c'      d .kx lO:  kO; xO,  'OO.  cOl      .'.oKk0c:xOk0kOOd0Oll;OXOdok,    :l,.......'.             ]],
                        [[                   lccl;  .;;;;;;looo.    :l     d..c       ';     .l  ;OdOo   kO; xO,  'OO.  cOl     .'.'lokd,::,:dc:o:od;''KXkdlxx    ,l'........'             ]],
                        [[                   llll,    ,:;:;ldoo.    :l     d  '':..:'  ''...''    cdo    dx, x0'  .00.  :0l    .'.'odccc,....;. ...'';lxOkxddd    .c'........'             ]],
                        [[                   clll;     '::;ldoo.                                                              .:..lxdo;k0Okx:,    l:OOO:.....     .;'..........            ]],
                        [[                    .co;      .:;dl,                                                                :..;xkkd.O0kkOc.    .OOkOk          .'''''......;            ]],
                        [[                      .;        ;'                                                                 ....cdxkxlO0kkO,.     c0kO0,         '';:;,'......            ]],
                        [[                                                                                                   ;..,:lodx;0K00k'.     .000Ko        .',ooc:''....;            ]],
                        [[                                                                                                  ....';:coxl0K00d;.      lKKKO.       .'ldoo:;'.....            ]],
                        [[                                                                                                  ;...',,;d:x00kOko'      .0kkO,      .,:dool:;';. .             ]],
                        [[                                                                                                  . .'''':x.dK0xOl::       :OxOd      :clllccc:;..;              ]],
                        [[                                                                                                  ;..'''':o.xNKKx' ..       o00K:    ,dololccccc,,.              ]],
                        [[                                                                                                  ..';,,,ld.0NXK'            lK0Xl   oxodddoodo:;.               ]],
                        [[                                                                                                  .',cl::codKKKo              ;XXXl  ;:llooxxd;.                 ]],
                        [[                                                                                                   .,ooodddoKKK'               ,KKK'  .'clloc'                   ]],
                        [[                                                                                                     'lolcc.0KO                 :KKk    ;,,.                     ]],
                        [[                                                                                                       .',,.KKd                  oK0;    ;                       ]],
                        [[                                                                                                            O0d                   OOx.                           ]],
                        [[                                                                                                            kkd,                  dxx:                           ]],
                        [[                                                                                                            xdol.                :dlok                           ]],
                        [[                                                                                                            :Oddl               .dodd:                           ]],
                        [[                                                                                                             :;:                  :;:                            ]],
                }

                dashboard.config.opts.noautocmd = true
                vim.cmd([[
      autocmd User AlphaReady echo 'Ready!'
    ]])
                alpha.setup(dashboard.config)
        end,
}
```
