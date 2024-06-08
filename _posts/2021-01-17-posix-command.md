# 命令
一些常用命令的使用方式和选项
## find
- -name -iname 否定加! `find . ! -name '*.txt'`
- -o -a `find . \( -name '*.txt' -o '*.png' \)`
- -path `find . -path '*/slynux/*' -name '*.txt'`
- -maxdepth -mindepth `find -maxdepth 1 -name '*.txt'`
- -L 跟随符号链接
- -type 文件类型
- -size `find . -size +2k`
- -user `find . -user slynux`

### 时间戳
- -atime 最后一次访问
- -mtime 最后一次修改
- -ctime 元数据（权限或所有权）最后一次改变
- -mmin 分钟
``` sh
# 1天前，7天内修改的文件
find . -mtime -7 -mtime +1
```

## xargs
从标准输入中读取数据，转换为另一个命令的参数输入。默认会执行`echo`指令。

- -d 分隔符 `echo "123X456X789" |xargs -d X -n 1`
- -n 每次调用命令的参数个数 `printf "1 2 3\n4 5 6\n7 8" |xargs -n 3`
- -I {} 用{}代替字符串，默认带有-n 1 `echo "1 2" |xargs -d" " -I {} echo {}`

``` sh
find . -print0 |xargs -0
```

## tr
- -d delete
- -c 补集 `tr -d -c '0-9'`
- -s 删除重复字符 `tr -s '\n'`

``` sh
printf '1 2 3 4 5 ' |echo $[ $(tr ' ' '+') 0 ]

## md5sum
``` sh
md5sum * >file_sum.md5
md5sum -c file_sum.md5
```

## gpg
[查看详细](http://en.wikipedia.org/wiki/GNU_Privacy_Guard)

## sort uniq
### sort
- -n 数字排序
- -r 逆序排序
- -k 选择列号
- -b 忽略前导空白
- -C 验证是否已排序 `if sort -C file; then echo sorted; fi`

### uniq
- -u 输出唯一的行
- -d 输出重复的行
- -c 统计次数
- -z 以\0作为分隔符输出

## mktemp
- -d 创建目录
- -u 只生成文件名，不创建

## ivonv
``` sh
# 日文转utf-8
iconv -c -f SHIFT-JIS -t utf-8 filename
```

## midecode
用于查看硬件信息