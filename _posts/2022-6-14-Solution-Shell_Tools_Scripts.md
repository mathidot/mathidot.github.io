---
layout : post
titile : Shell tools and scripts
tags : Missing Semester
---

# Solution-Shell工具和脚本

1. 使用ls命令

   * 所有文件（包括隐藏文件）：-a
   * 文件打印以人类可以理解的格式输出：-h
   * 文件以最近访问顺序排序：-t
   * 以彩色文本显示输出结果--color=auto

2. 编写两个bash函数marco和polo执行下面的操作。每当你执行marco，当前的工作目录应当以某种形式保存，当执行polo时，无论现在处在什么目录下，都应当`cd` 回到当时执行marco的目录。为了方便debug，你可以把代码写在单独的文件marco.sh中，并通过`souce marco.sh`命令，（重新）加载函数。通过source来加载函数，随后可以在bash中直接使用。

   ```sh
   #!/bin/bash
   marco(){
   	echo "$(pwd)" > $HOME/marco_history.log
   	echo "save pwd $(pwd)"
   }
   
   polo(){
   	cd "$(cat "$HOME/marco_history.log")"
   }
   ```

   ```sh
   #!/bin/bash
   marco(){
   	export MARCO=$(pwd)
   }
   
   polo(){
   	cd "$MARCO"
   }
   ```

3. 假设你有一个命令，它很少出错。因此为了在出错时能够对其进行测试，需要花费大量的时间重现错误并捕获错误。编写一段bash脚本，运行如下的脚本知道它出错，将它的标准输出和标准错误流记录到文件，并在最后输出所有内容。加分项：报告脚本在失败前共运行了多少次。

   ```sh
    #!/usr/bin/env bash
   
    n=$(( RANDOM % 100 ))
   
    if [[ n -eq 42 ]]; then
        echo "Something went wrong"
        >&2 echo "The error was using magic numbers"
        exit 1
    fi
   
    echo "Everything went according to plan"
   
   ```

   ```sh
    count=1
   
    while true
    do
        ./buggy.sh 2> out.log
        if [[ $? -ne 0 ]]; then
            echo "failed after $count times"
            cat out.log
            break
        fi
        ((count++))
   
    done
   #-----------------------------------------
    for ((count=1;;count++))
    do
        ./buggy.sh 2> out.log
        if [[ $? -ne 0 ]]; then
            echo "failed after $count times"
            cat out.log
            break
   
        echo "$count try"
        fi
    done
   #-----------------------------------------
    count=0
    until [[ "$?" -ne 0 ]];
    do
        count=$((count+1))
        ./random.sh 2> out.txt
    done
   
    echo "found error after $count runs"
    cat out.txt
   
   ```

4. 本节课我们讲解的 find 命令中的 -exec 参数非常强大，它可以对我们查找的文件进行操作。 如果我们要对所有文件进行操作呢？例如创建一个zip压缩文件？我们已经知道，命令行可以从参数或标准输入接受输入。在用管道连接命令时，我们将标准输出和标准输入连接起来，但是有些命令，例如tar 则需要从参数接受输入。这里我们可以使用`xargs`命令，它可以使用标准输入中的内容作为参数。 例如 `ls | xargs rm `会删除当前目录中的所有文件。您的任务是编写一个命令，它可以递归地查找文件夹中所有的HTML文件，并将它们压缩成zip文件。注意，即使文件名中包含空格，您的命令也应该能够正确执行（提示：查看 `xargs`的参数-d

   1. 首先创建所需的文件

      ```sh
      mkdir html_root
      cd html_root
      touch {1..10}.html
      mkdir html
      cd html
      touch xxxx.html
      
      ```

   2. 执行find命令

      ```sh
        #for Linux
        find . -type f -name "*.html" | xargs -d '\n'  tar -cvzf html.zip
      ```

5. 编写一个命令或脚本递归的查找文件夹中最近使用的文件。更通用的做法，你可以按照最近的使用时间列出文件吗？`find . -type f -print0 | xargs -0 ls -lt | head -1`

​		当文件数量较多时，上面的解答会得出错误结果，解决办法是增加 `-mmin `条件，先将最近修改的文件进行初		步	筛选再交给ls进行排序显示 `find . -type f -mmin -60 -print0 | xargs -0 ls -lt | head -10`
