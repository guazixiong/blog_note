# IdeaVim

配置文件

```bash
" 在normal模式下保持英文输入法（IdeaVimExtension）
:set keep-english-in-normal
" 在normal模式下保持英文输入法，插入模式下回复插入模式时的输入法（IdeaVimExtension）
:set keep-english-in-normal-and-restore-in-insert
" 显示行号  显示绝对行号
set nu
" 显示相对行号
set relativenumber

" 开启语法高亮
syntax on

" 显示状态栏
set laststatus=4
" 将tab替换为相应数量的空格
set expandtab

set tabstop=4

set shiftwidth=4

" 设置历史记录的数量为 2000 条
set history=2000

" 禁用备份文件和交换文件的创建
set nobackup
set noswapfile

" 将 <Leader> 映射键设置为空格
let mapleader=' '

" 退出、保存关闭  用 <Leader>q 退出当前文件
nnoremap <Leader>q :q<CR>
" 用<Leader>Q 强制退出所有文件
nnoremap <Leader>Q :qa!<CR>

" redo
nnoremap U <C-r>

" 跳转到实体类
nnoremap <Leader>c :action GotoClass<CR>

" 跳转到声明
nnoremap <Leader>d :action GotoDeclaration<CR>

" 查看断点
nnoremap <Leader>b :action ViewBreakpoints<CR>

" 跳到实现类
nnoremap <Leader>i :action GotoImplementation<CR>

" 跳转到文件
nnoremap <Leader>f :action GotoFile<CR>

" 跳转Action
nnoremap <Leader>a :action GotoAction<CR>

" 跳到测试类，没有自动创建
nnoremap <Leader>t :action GotoTest<CR>

" 注释
nnoremap <Leader>/ :action CommentByLineComment<CR>

" 优化导包
nnoremap <Leader>O :action OptimizeImports<CR>

" run
nnoremap <Leader>R :action Run<CR>

" debug
nnoremap <Leader>D :action Debug<CR>

" 重命名 文件
nnoremap <Leader>N :action RenameFile<CR>
" 重命名 变量、类名
nnoremap <Leader>N :action RenameElement<CR>

" 复制class 引用
nnoremap <Leader>cr :action CopyReference<CR>

" 抽取方法
nnoremap <Leader>M :action ExtractMethod<CR>

" 格式化代码
nnoremap <Leader>F :action ReformatCode<CR>

" 展示类继承关系
nnoremap <Leader>H :action TypeHierarchy<CR>

" 实现方法
nnoremap <Leader>m :action ImplementMethods<CR>

" 覆盖方法
nnoremap <Leader>o :action OverrideMethods<CR>

" 上一个tab
nnoremap <Leader>h :action PreviousTab<CR>

" 下一个tab
nnoremap <Leader>l :action NextTab<CR>

" 展示错误信息
nnoremap <Leader>E :action ShowErrorDescription<CR>

" 展示错误信息
nnoremap <Leader>nE :action GotoNextError<CR>

" 展示tab页面
nnoremap <Leader>T :action Switcher<CR>
```

