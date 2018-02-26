---
title: Vim的使用及YouCompleteMe的配置
date: 2018-02-26 21:06:19
tags:
  - Vim
  - Linux
categories:
   - Vim
---

今天想来介绍一下目前算是经常用的一个编辑器，那就是VIM。这个东西基本上自从我使用Linux开始就伴随着我使用Linux的过程，对于为什么使用这个上古编辑器，我会在下面解释。

## 为什么

在现在的那些IDE的功能都非常好的情况下，为什么我要使用这么一个上古级别的编辑器呢？难道只是为了装逼，还是为了其它的。我在配置Vim以及Emacs的时候想过一个问题，我们是需要一个简单的代码编辑器呢还是需要一个功能齐全的大杂烩，像许多IDE一样什么功能都有。在那么强的IDE下面写代码有的时候感觉写代码就像是在学习一个IDE在怎么使用，不是在进行编程，感觉像是IDE的努力奴隶一样。当然对于初学者来说使用IDE能省下许多的力气，可以将大部分的精力放在学习上面。

对于我使用Vim的需求来说，主要差不多就只有亮点，代码高亮以及补全，对于调试来说我基本不太需要，因为使用Vim的大部分场景都是在服务器上面，主要面对的语言还是Python之类的脚本语言为主，其他时候的编码我大部分是在VSCode以及Idea上面完成的。

## 配置

对于Vim的安装我不想做介绍，没有意义，我就来介绍一下我写的配置，完整配置位于我的Github上面：[vimrc](https://github.com/wjpworking/vimrc)中。使用Vim，但是不是简单的使用它的快捷键，还应该使用其丰富的插件才行，这样感觉才爽。其实更多的时候使用vim键位其实比使用纯vim更加能提高开发的效率。并且我还建议参考下[Spacevim](https://github.com/SpaceVim/SpaceVim)和[spf13](https://github.com/spf13/spf13-vim)这两个的配置来编写适合自己的配置。

### vim-plug

对于官方给的安装方式，需要使用的是Git手动安装，但是既然我们都使用了vim了，为什么我们不卸载Vim的配置文件中让其自动加载呢，所以我们可以在`.vimrc`中写入如下代码：

```vim
if !filereadable(vimplug_exists)
  if !executable("curl")
    echoerr "You have to install curl or first install vim-plug yourself!"
    execute "q!"
  endif
  echo "Installing Vim-Plug..."
  echo ""
  silent !\curl -fLo ~/.vim/autoload/plug.vim --create-dirs https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
  let g:not_finish_vimplug = "yes"

  autocmd VimEnter * PlugInstall
endif
```

这样在使用vim的时候会自动检查vimplug这个插件是否存在，当不存在的时候会去GitHub上面将[Vim-plug](https://github.com/junegunn/vim-plug)这个插件下载下来，这样就可以使用了。

对于vim的插件来说，我使用的有这些：

```vim
call plug#begin(expand('~/.vim/plugged'))

"***********************
"" Plug install packages
"***********************
Plug 'scrooloose/nerdtree'
Plug 'tpope/vim-fugitive'
Plug 'airblade/vim-gitgutter'  " git历史
Plug 'jiangmiao/auto-pairs' " 自动补全括号
Plug 'majutsushi/tagbar'  " 代码组成
Plug 'w0rp/ale'  " 语法检测
Plug 'Yggdroot/indentLine' " 对齐线
Plug 'sheerun/vim-polyglot'  " 语言集合
Plug 'junegunn/fzf', { 'dir': '~/.fzf', 'do': './install --bin' }
Plug 'Shougo/vimproc.vim', {'do': 'make'}
Plug 'haya14busa/incsearch.vim'  " 高亮搜索内容
Plug 'Valloric/YouCompleteMe', { 'do': './install.py --clang-completer --system-libclang --go-completer  --js-completer --rust-completer' }
Plug 'junegunn/vim-easy-align'  " 等号对齐
Plug 'luochen1990/rainbow'  " 高亮括号
Plug 'ntpeters/vim-better-whitespace'  " 去除多余括号
Plug 'scrooloose/nerdcommenter'  " 代码注释
Plug 'ryanoasis/vim-devicons'  " Vim Dev Icons

" themes
Plug 'vim-airline/vim-airline'
Plug 'vim-airline/vim-airline-themes'
Plug 'morhetz/gruvbox'

" fzf
Plug 'junegunn/fzf', { 'dir': '~/.fzf', 'do': './install --all' }
Plug 'junegunn/fzf.vim'

"" Snippets
Plug 'SirVer/ultisnips'
Plug 'honza/vim-snippets'

"" Color
Plug 'flazz/vim-colorschemes'

"" editconfig
Plug 'editorconfig/editorconfig-vim'

"" python
Plug 'raimon49/requirements.txt.vim'
Plug 'google/yapf', { 'rtp': 'plugins/vim', 'for': 'python' }
Plug 'fisadev/vim-isort'

"" html
Plug 'mattn/emmet-vim'
Plug 'hail2u/vim-css3-syntax'
Plug 'jelera/vim-javascript-syntax'

" vue
" Plug 'posva/vim-vue'

"" fcitx
Plug 'lilydjwg/fcitx.vim'

"" markdown
Plug 'plasticboy/vim-markdown', {'on_ft' : 'markdown'}

call plug#end()
```

对于我使用的大部分的插件我都不想细说，但是其中[YouCompleteMe](https://github.com/Valloric/YouCompleteMe)我想好好的说一说。

### YouCompleteMe

对于补全，尤其是C/C++这类复杂的语言，尤其有模板和泛型的存在，基本上补全十分麻烦；还有就是像Python这样的动态语言，因为其没有类型检查，语义上的补全和推导十分麻烦，甚至pycharm在这方面都不好，但是YouCompleteMe这就做的比较好的。对于补全，我需要C的，python、Go以及Rust的，所以得先在系统中安装这些软件，并且构建的时候需要C环境，所以得先安装clang，并且需要cmake来构建编译过程。我的建议是使用系统的自带的clang，虽然官方建议使用最新的，那是因为可能编译不通过，但是对于国内这种情况就算了吧。

对于编译，我们只需要在Vim-plug的配置中加入如下代码就行:

``` vim
Plug 'Valloric/YouCompleteMe', { 'do': './install.py --clang-completer --system-libclang --go-completer  --js-completer --rust-completer' }
```

对于补全，我开启了语义补全、注释补全以及字符串补全，并且从一个字符开始就除非，这个比较符合我们使用IDE的习惯，这些设置的配置如下：

```vim
let g:ycm_global_ycm_extra_conf = '~/.ycm_extra_conf.py'
let g:ycm_python_binary_path = '/usr/bin/python3'
let g:ycm_server_python_interpreter = '/usr/bin/python3'

" 补全
set completeopt=longest,menu  " 下拉的补全菜单

" 回车选中当前项
let g:ycm_key_list_stop_completion = ['<CR>']

" 键位
nnoremap <leader>jd :YcmCompleter GoToDefinitionElseDeclaration<CR>

" 配置
let g:ycm_collect_identifiers_from_tags_files = 1  " 基于标签的引擎
let g:ycm_min_num_of_chars_for_completion = 1  " 1个字符开始补全
let g:ycm_cache_omnifunc = 0  " 禁止缓存
let g:ycm_seed_identifiers_with_syntax = 1  " 语法关键字补全
let g:ycm_complete_in_comments = 1  " 补全注释
let g:ycm_complete_in_strings = 1  " 补全字符串
let g:ycm_auto_trigger = 1
let g:ycm_collect_identifiers_from_comments_and_strings = 1  " 将注释和字符串收录到补全信息中
let g:ycm_confirm_extra_conf=0  " 关闭加载.ycm_extra_conf.py的提示

" 补全的开始位置的设置
let g:ycm_semantic_triggers = get(g:, 'ycm_semantic_triggers', {})

function! s:set_ft_triggers(ft, expr, override) abort
  if a:override
    let g:ycm_semantic_triggers[a:ft] = a:expr
  elseif !has_key(g:ycm_semantic_triggers, a:ft)
    let g:ycm_semantic_triggers[a:ft] = a:expr
  endif
endfunction

call s:set_ft_triggers('c', ['->', '.'], 0)
call s:set_ft_triggers('cpp', ['->', '.', '::'], 0)
call s:set_ft_triggers('perl', ['->'], 0)
call s:set_ft_triggers('javascript,python,go', ['.'], 0)
call s:set_ft_triggers('java,jsp', ['.'], 0)
call s:set_ft_triggers('vim', ['re![_a-zA-Z]+[_\w]*\.'], 0)
call s:set_ft_triggers('sh', ['re![\w-]{2}', '/', '-'], 0)
call s:set_ft_triggers('zsh', ['re![\w-]{2}', '/', '-'], 0)

" 自动关闭窗口
let g:ycm_autoclose_preview_window_after_completion = 1
let g:ycm_autoclose_preview_window_after_insertion = 1
```

还有在安装好之后，需要在`$HOME`目录下面创建一个空的`.tern_project`文件（需要配置的请参考文档）和将`$HOME/.vim/plugged/YouCompleteMe/third_party/ycmd/examples/.ycm_extra_conf.py`拷贝到`$HOME`下，大部分的情况下这个已经能满足绝大部分的需求了。

至此，我对Vim的配置就完成了。