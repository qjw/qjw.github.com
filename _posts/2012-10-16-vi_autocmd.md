---
layout: post
title: VI自动化命令(autocmd)
category: bash
---

##Vi自动化命令
###参考[VI中文手册](http://vimcdoc.sourceforge.net/doc/help.html)

        autocmd BufReadPost *.sh inoremap { {}<ESC>i
        autocmd BufReadPost *.c inoremap { {<CR>}<ESC>O
        autocmd BufReadPost *.cc inoremap { {<CR>}<ESC>O
        autocmd BufReadPost *.cpp inoremap { {<CR>}<ESC>O
        autocmd BufReadPost *.h inoremap { {<CR>}<ESC>O
        
具体见[这里](http://vimcdoc.sourceforge.net/doc/autocmd.html#autocmd.txt)

进一步扩展见[函数](http://vimcdoc.sourceforge.net/doc/eval.html#eval.txt)

英文文档见[这里](http://vimdoc.sourceforge.net/htmldoc/usr_toc.html)




