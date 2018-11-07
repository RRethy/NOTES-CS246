# Notes on programming C++ inside Vim

## Navigating make errors inside Vim

```vim
" Make sure you write your changes
:wa

:make

" Open quickfix window
:cope
```

* `set autowrite` can be useful to avoid having to `:wa` before `:make` each time, but I prefer to not use it since it will write the file when it becomes `hidden` (`:h 'hidden'` for more info)

## Linting C++ programs in Vim

* Install plugin [ALE](https://github.com/w0rp/ale)
* Add the following to ~/.vimrc

```vim
let g:ale_c_gcc_executable='g++'
let g:ale_c_gcc_options='-std=c++14 -Wall -MMD -g'
```

* [vim-unimpared](https://github.com/tpope/vim-unimpaired) is useful for navigating errors

## Improved syntax highlighting

[Use this plugin](https://github.com/bfrg/vim-cpp-modern)

## Editing/Reading files over ssh/scp

[Gist I wrote](https://gist.github.com/RRethy/ad8a9a3b1112a48226ec3336fa981224)
