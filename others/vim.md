### VIM

##### normal模式
J: 合并两行  
u: 撤销  
U: 撤销对整行的全部操作  
Ctrl+r: 重做,对撤销的撤销  
ZZ: 保存退出  
Ctrl+]: 跳至标签  
Ctrl+t,Ctrl+o: 跳回之前的位置  

##### 移动光标
w: 光标移至下一个单词首字符 （backward）  
W: 光标后移至词首，以空格为分隔符  
b: 光标移至上一个单词首字符 （end od word）  
B: 光标前移至词首，以空格为分隔符  
e: 光标移至下一个单词尾字符  
E: 光标后移至词尾，以空格为分隔符  
ge: 光标移至上一个单词尾字符  

$: 光标移至当前行尾  
n$: 光标移至下n-1行行尾  
^: 光标移至当前行首  
0: 光标移至绝对行首  

fx: 查找下一个x字符  
nfx: 查找下n个x字符  
Fx: 向前搜索x字符  

tx/Tx: 同fx但将光标停留在x字符前一个字符上  

%: 跳转至与当前光标下括号相匹配的括号上  
n%: 跳转至%n行  
nG: 跳转至第n行  

\`\`: 当使用`G`命令跳转时，vim会标记起跳的位置，\`\`命令可以跳回，再次使用这个命令又会跳转回去，\`\`命令可以在两点之间来回跳转    
Ctrl+o:跳转到更早些时间停置光标的位置    
Ctrl+i:跳转到后来停置光标的更新位置，同`Tab`键   
:jumps :可以列出曾经跳转列表  

###### 屏幕范围跳转
H: 跳转至当前屏幕首行  
M: 跳转至当前屏幕中间  
L: 跳转至当前屏幕末行 

###### 滚屏
Ctrl+u: 光标向上滚动半屏  
Ctrl+d: 光标向下滚动半屏  

Ctrl+f: 向前滚动一屏 forward  
Ctrl+b: 向后滚动一屏 backward  

zz: 把当前行置于屏幕中间   
zt: 把当前行置于屏幕顶端 top  
zb: 把当前行至于屏幕底端 bottom 

###### 具名标记
mx: 设置具名标记，可以a-z自定义标记，使用\`x跳转至标记所在位置，'x可以跳转至标记所在行行首   
:marks :可以查看标记列表  
': 进行此次跳转之前的起跳点  
": 上次编辑该文件时光标最后停留的位置  
\[: 最后一次修改的起始位置  
\]: 最后一次修改的结束位置  
 
###### 显示当前位置
Ctrl+g  
:set num  
:set ruler  

###### 查找WORD
将光标放在`WORD`上，按`*`向下搜索，按`#`向上搜索  
/\<WORD\>查找WORD 

###### 替换
r: 替换单个字符
.: 重复上一次做出的改动
~: 改变当前光标下字符大小写，并将光标移至下一个字符

###### 小幅修改
x: 删除一个字符  
X: 删除光标左边的一个字符  
d: `d`命令后可跟任何一个位移命令，它将删除当前光标位置至位移处的文本内容，d4w, d$，dd等,  
dG：删除当前行至文件尾    
dgg: 删除当前行至文件头  
D: 同d$，删除至行尾  
c: change，与`d`命令类似，但在命令执行后进入Insert模式, `cc`，删除整行    
C: 同c$  
p: 将删除的内容取回了，`P`  
xp: 交换两个字符，x删除一个字符，并放在寄存器中，p将字符取回来放在光标之后     
y: 复制  

###### 正则匹配
^: 行首  
$: 行尾  
.: 匹配单个任意字符    

##### 虚拟模式
v：虚拟字符模式  
V: 虚拟行模式  
Ctrl+v: 虚拟块模式  
o: other end,将光标至于被选文本的另一端  
O: 当使用虚拟块模式时，`o`命令会将光标至于对角线的另一端，`O`可以在同行的两个角上移动  

##### 文本对象
不管当前光标所在的位置而把整个文本对象作为操作对象  
daw: delete a word  
cis: inner sentence, `as` a sentence    

##### 命令行模式
:set ignorecase :忽略大小写  
:set noignorecase  

:set hlsearch: 设置搜索高亮  
:set nohlsearch  
:nohlsearch： 仅去掉当前高亮  

:set incsearch: 增量搜索  
:set nowrapscan: 搜索至文件开头或结尾停止，不循环搜索  

:set showmode   
:set showcmd  
:set ruler  
+-------------------------------------------------+  
|text in the Vim window                           |  
|~                                                |  
|~                                                |  
|-- VISUAL --                         2f 43,8 17% |  
+-------------------------------------------------+  
^^^^^^^^^^^\n\n\n\n\n\n\n^^^^^^^^ ^^^^^^^^^^  
'showmode'                      'showcmd' 'ruler'  

###### 命令映射  
:map :设置映射，`:map <F5> i{<Esc>ea}<Esc>`  
:autocmd: 设置自动执行的命令，`autocmd FileType text setlocal textwidth=78`, "autocmd FileType text"是一个自动命令。它所定义的是每当文件类型被设置为"text"时就自动执行它后面的命令。"setlocal textwidth=78"把"textwidth"选项的值设置为78，但这种设置只对当前的一个文件有效。  

#### Plugin
###### 全局plugin
- $VIMRUNTIME/macros  /usr/share/vim/vim74/macros  
- http://www.vim.org  






