+++ 
draft = true
date = 2021-09-02T17:22:50+08:00
title = "Vim 包管理"
description = ""
slug = "" 
tags = []
categories = []
externalLink = ""
series = []
+++


# `.vim` 文件夹
打开 Vim 执行 `:help runtimepath`

```
                                'runtimepath' 'rtp' vimfiles
'runtimepath' 'rtp'     string  (default:
                                        Unix:  "$HOME/.vim,
                                                $VIM/vimfiles,
                                                $VIMRUNTIME,
                                                $VIM/vimfiles/after,
                                                $HOME/.vim/after"
                                        Amiga: "home:vimfiles,
                                                $VIM/vimfiles,
                                                $VIMRUNTIME,
                                                $VIM/vimfiles/after,
                                                home:vimfiles/after"
                                        MS-Windows: "$HOME/vimfiles,
                                                $VIM/vimfiles,
                                                $VIMRUNTIME,
                                                $VIM/vimfiles/after,
                                                $HOME/vimfiles/after"
                                        macOS: "$VIM:vimfiles,
                                                $VIMRUNTIME,
                                                $VIM:vimfiles:after"
                                        Haiku: "$BE_USER_SETTINGS/vim,
                                                $VIM/vimfiles,
                                                $VIMRUNTIME,
                                                $VIM/vimfiles/after,
                                                $BE_USER_SETTINGS/vim/after"
                                        VMS:   "sys$login:vimfiles,
                                                $VIM/vimfiles,
                                                $VIMRUNTIME,
                                                $VIM/vimfiles/after,
                                                sys$login:vimfiles/after")
                        global
        This is a list of directories which will be searched for runtime
        files:
          filetype.vim  filetypes by file name new-filetype
          scripts.vim   filetypes by file contents new-filetype-scripts
          autoload/     automatically loaded scripts autoload-functions
          colors/       color scheme files :colorscheme
          compiler/     compiler files :compiler
          doc/          documentation write-local-help
          ftplugin/     filetype plugins write-filetype-plugin
          import/       files that are found by :import
          indent/       indent scripts indent-expression
          keymap/       key mapping files mbyte-keymap
          lang/         menu translations :menutrans
          menu.vim      GUI menus menu.vim
          pack/         packages :packadd
          plugin/       plugin scripts write-plugin
          print/        files for printing postscript-print-encoding
          spell/        spell checking files spell
          syntax/       syntax files mysyntaxfile
          tutor/        files for vimtutor tutor
        
        And any other file searched for with the :runtime command.
        
        The defaults for most systems are setup to search five locations:
        1. In your home directory, for your personal preferences.
        2. In a system-wide Vim directory, for preferences from the system
           administrator.
        3. In $VIMRUNTIME, for files distributed with Vim.
                                                        after-directory
        4. In the "after" directory in the system-wide Vim directory.  This is
           for the system administrator to overrule or add to the distributed
           defaults (rarely needed)
        5. In the "after" directory in your home directory.  This is for
           personal preferences to overrule or add to the distributed defaults
           or system-wide settings (rarely needed).
        
        More entries are added when using packages.  If it gets very long
        then :set rtp will be truncated, use :echo &rtp to see the full
        string.
        
        Note that, unlike 'path', no wildcards like "**" are allowed.  Normal
        wildcards are allowed, but can significantly slow down searching for
        runtime files.  For speed, use as few items as possible and avoid
        wildcards.
        See :runtime.
        Example:
                :set runtimepath=~/vimruntime,/mygroup/vim,$VIMRUNTIME
        This will use the directory "~/vimruntime" first (containing your
        personal Vim runtime files), then "/mygroup/vim" (shared between a
        group of people) and finally "$VIMRUNTIME" (the distributed runtime
        files).
        You probably should always include $VIMRUNTIME somewhere, to use the
        distributed runtime files.  You can put a directory before $VIMRUNTIME
        to find files which replace a distributed runtime files.  You can put
        a directory after $VIMRUNTIME to find files which add to distributed
        runtime files.
        When Vim is started with --clean the home directory entries are not
        included.
        This option cannot be set from a modeline or in the sandbox, for security reasons.
        
```

# 常用插件
## golang
> https://github.com/fatih/vim-go

```
git clone https://github.com/fatih/vim-go.git ~/.vim/pack/plugins/start/vim-go
```

# 参考
- Vim 8 Packages, https://vimhelp.org/repeat.txt.html#packages
- Vim 8 内置包管理使用指南, https://blog.hulifa.cn/2019-10-20-Vim-8%E5%86%85%E7%BD%AE%E5%8C%85%E7%AE%A1%E7%90%86%E4%BD%BF%E7%94%A8%E6%8C%87%E5%8D%97/
- 使用 Git 管理插件, https://taoshu.in/vim/plug-git.html
- Is the ~/.vim directory used for anything other than plugins?. https://vi.stackexchange.com/questions/4969/is-the-vim-directory-used-for-anything-other-than-plugins