# 变量

1、定义变量：变量名=值（等号两边没有空格）

```Shell
A=1
```

2、撤销变量：unset 变量名

```shell
unset A
```

3、声明静态变量：readonly 变量名=值（不能unset）

```Shell
readonly B=3
```

4、变量默认都是字符串类型，不能直接进行运算

5、export 可以把变量升级成全局变量

## 特殊变量

1、$ n
功能描述：n为数字，$ 0代表该脚本名称，$ 1-$ 9代表第一到第九个参数，十以上的参数，十以上的参数需要用大括号包含，如$ {10}

2、$ #
功能描述：获取所有输入参数个数，常用于循环

3、$ *
功能描述：这个变量代表命令行中所有的参数，$\* 把所有的参数看成一个整体

4、$ @
功能描述：这个变量也代表命令行中所有的参数，不过$ @把每个参数区分对待

5、$ ?
功能描述：最后一次执行的命令的返回状态。如果这个变量的值为0，证明上一个命令正确执行；如果这个变量的值为非0（具体是哪个数，由命令自己来决定），则证明上一个命令执行不正确了

# 运算符

expr + , - , *, /, % 加，减，乘，除，取余
**运算符的左右两侧有空格**

“$ ((运算式))”或“$ \[运算式\]”（注意其实没有空格）

```shell
s=$[(2+3)*4]
echo $s
```

# 条件判断

\[ condition \]（注意condition前后要有空格）

## 常用判断条件：

1、两个整数之间比较：

| 符号  | 描述  |
| --- | --- |
| -lt | 小于（less than） |
| -le | 小于等于（less equal） |
| -eq | 等于（equal） |
| -gt | 大于（greater than） |
| -ge | 大于等于（greater equal） |
| -ne | 不等于（Not equal） |

2、按照文件权限进行判断：

| 符号  | 描述  |
| --- | --- |
| -r  | 有读的权限（read） |
| -w  | 有写的权限（write） |
| -x  | 有执行的权限（execute） |

3、按照文件类型进行判断：

| 符号  | 描述  |
| --- | --- |
| -f  | 文件存在并且是一个常规的文件（file） |
| -e  | 文件存在（existence） |
| -d  | 文件存在并是一个目录（directory） |

## 举例

23是否大于等于22

```shell
[ 23 -le 25 ]
echo $?			# 输出0
```

helloworld.sh是否具有写权限

```shell
[ -w helloworld.sh ]
echo $?
```

/home/atguigu/cls.txt目录中的文件是否存在

```shell
[ -e /home/atguigu/cls.txt ]
echo $?
```

# 流程控制

## if语句

if 后面有空格

```shell
if [ 条件判断式 ];then 
  程序 
elif;then 
    程序
fi 
    
或者 
if [ 条件判断式 ] 
then 
        程序 
elif
then 
    程序
then
        程序
fi
```

## case语句

注意事项：

- case行尾必须为单词“in”，每一个模式匹配必须以右括号“）”结束。
- 双分号“;;”表示命令序列结束，相当于java中的break。
- 最后的“*）”表示默认模式，相当于java中的default。

```shell
case $变量名 in 
  "值1"） 
    如果变量的值等于值1，则执行程序1 
    ;; 
  "值2"） 
    如果变量的值等于值2，则执行程序2 
    ;; 
  …省略其他分支… 
  *） 
    如果变量的值都不是以上的值，则执行此程序 
    ;; 
esac
```

示例：

```Shell
!/bin/bash

case $1 in
"1")
        echo "banzhang"
;;

"2")
        echo "cls"
;;
*)
        echo "renyao"
```

## for循环

基本语法：

```Shell
# 第一种
for (( 初始值;循环控制条件;变量变化 )) 
  do 
    程序 
  done

# 第二种
for 变量 in 值1 值2 值3… 
  do 
    程序 
  done
```

示例：

```Shell
s=0
for((i=0;i<=100;i++))
do
        s=$[$s+$i]
done
echo $s
```

## while循环

基本语法：

```Shell
while [ 条件判断式 ] 
  do 
    程序
  done
```

示例：

```Shell
s=0
i=1
while [ $i -le 100 ]
do
        s=$[$s+$i]
        i=$[$i+1]
done

echo $s
```

# read读取控制台输入

基本语法：

```Shell
read(选项)(参数)
    选项：
-p：指定读取值时的提示符；
-t：指定读取值时等待的时间（秒）。
参数
    变量：指定读取值的变量名
```

示例：

```Shell
read -t 7 -p "Enter your name in 7 seconds " NAME
echo $NAME
```

# 函数

## 系统函数

basename:

```Shell
basename [string / pathname] [suffix]  	（功能描述：basename命令会删掉所有的前缀包括最后一个（‘/’）字符，然后将字符串显示出来。
选项：
suffix为后缀，如果suffix被指定了，basename会将pathname或string中的suffix去掉。
```

示例：

```Shell
[atguigu@hadoop101 datas]$ basename /home/atguigu/banzhang.txt 
banzhang.txt
[atguigu@hadoop101 datas]$ basename /home/atguigu/banzhang.txt .txt
banzhang
```

dirname:

```Shell
dirname 文件绝对路径		\
（功能描述：从给定的包含绝对路径的文件名中去除文件名（非目录的部分），然后返回剩下的路径（目录的部分）
```

示例：

```Shell
[atguigu@hadoop101 ~]$ dirname /home/atguigu/banzhang.txt 
/home/atguigu
```

## 自定义函数

基本语法：

```Shell
[ function ] funname[()]
{
    Action;
    [return int;]
}
funname
函数返回值，只能通过$?系统变量获得
```

示例：

```Shell
#!/bin/bash
function sum()
{
    s=0
    s=$[ $1 + $2 ]
    echo $s
}
read -p "Please input the number1: " n1
read -p "Please input the number2: " n2
sum $n1 $n2
```

# Shell工具

## Cut

基本语法：

```Shell
cut [选项参数]  filename
说明：默认分隔符是制表符
-f	列号，提取第几列
-d	分隔符，按照指定分隔符分割列
```

示例：

```Shell
[atguigu@hadoop101 datas]$ vim cut.txt
dong shen
guan zhen
wo  wo
lai  lai
le  le
[atguigu@hadoop101 datas]$ cut -d " " -f 2,3 cut.txt 
shen
zhen
 wo
 lai
 le
[atguigu@hadoop101 datas]$ cat cut.txt | grep "guan" | cut -d " " -f 1
guan

# 切割路径
[atguigu@hadoop101 datas]$ echo $PATH
/usr/lib64/qt-3.3/bin:/usr/local/bin:/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/sbin:/home/atguigu/bin
# 2- 表示后面所有列
[atguigu@hadoop102 datas]$ echo $PATH | cut -d: -f 2-
/usr/local/bin:/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/sbin:/home/atguigu/bin

# 切割 IP 地址
[atguigu@hadoop101 datas]$ ifconfig eth0 | grep "inet addr" | cut -d: -f 2 | cut -d" " -f 1
192.168.1.102
```

## Sed

sed是一种流编辑器，它一次处理一行内容。处理时，把当前处理的行存储在临时缓冲区中，称为“模式空间”，接着用sed命令处理缓冲区中的内容，处理完成后，把缓冲区的内容送往屏幕。接着处理下一行，这样不断重复，直到文件末尾。文件内容并没有改变，除非你使用重定向存储输出

sed \[选项参数\] ‘command’ filename

- -e 直接在指令列模式上进行sed的动作编辑。（多个命令执行的时候需要在每个命令前加上 -e）

3.  命令功能描述

- a 新增，a的后面可以接字串，在下一行出现
- d 删除
- s 查找并替换

示例：

```Shell
# 在第二行下面加 mei nv  但是源文件并不改变
[atguigu@hadoop102 datas]$ sed "2a mei nv" sed.txt 
dong shen
guan zhen
mei nv
wo  wo
lai  lai

le  le
# 删除文件中包含wo的行
[atguigu@hadoop102 datas]$ sed '/wo/d' sed.txt
dong shen
guan zhen
lai  lai

le  le

# 查找文件中的 wo 替换成 ni  g表示全局替换，否则只替换第一个
[atguigu@hadoop102 datas]$ sed 's/wo/ni/g' sed.txt 
dong shen
guan zhen
ni  ni
lai  lai

le  le
# 将sed.txt文件中的第二行删除并将wo替换为ni
[atguigu@hadoop102 datas]$ sed -e '2d' -e 's/wo/ni/g' sed.txt 
dong shen
ni  ni
lai  lai

le  le
```

## Awk

一个强大的文本分析工具，把文件逐行的读入，以空格为默认分隔符将每行切片，切开的部分再进行分析处理。

- -F 指定输入文件分隔符
- -v 赋值一个用户定义变量

基本用法：

```Shell
awk [选项参数] ‘pattern1{action1}  pattern2{action2}...’ filename
pattern：表示AWK在数据中查找的内容，就是匹配模式
action：在找到匹配内容时所执行的一系列命令
```

示例：

## Sort

sort命令是在Linux里非常有用，它将文件进行排序，并将排序结果标准输出。

sort(选项)(参数)

| 参数  | 描述  |
| --- | --- |
| -n  | 依照数值大小排序} |
| -t  | 以相反的顺序排序 |
| -t  | 设置排序时使用的分隔字符 |
| -k  | 指定需要排序的列 |

```Shell
[atguigu@hadoop102 datas]$ touch sort.sh
[atguigu@hadoop102 datas]$ vim sort.sh 
bb:40:5.4
bd:20:4.2
xz:50:2.3
cls:10:3.5
ss:30:1.6
[atguigu@hadoop102 datas]$ sort -t : -nrk 3  sort.sh 
bb:40:5.4
bd:20:4.2
cls:10:3.5
xz:50:2.3
ss:30:1.6
```