---
    title: Git Notes
    author: shawshary
    date: 2022-11-05
---

\newpage

# Customizing Git #



## Git Attributes ##
> Reference: <https://git-scm.com/book/en/v2/Customizing-Git-Git-Attributes>



### Identifying Binary Files ###

To tell Git to treat `<binary_file>` as binary data, add the following line to
your `.gitattributes` file.

```bash
<binary_file> binary
```
Now, Git won’t try to convert or fix **CRLF issues**; nor will it try to compute
or print a **diff** for changes in this file when you run git show or git diff
on your project.


### Customizing Diffing ###

1.  对特定文件编码的文件使用带文件转码功能的diff脚本.

    如果文件编码不是utf8格式, 默认的diff会将其认为utf8编码, 导致在显示cp936中文
    编码时出现乱码问题. 如下图所示,cp936编码的文件会显示<B2>...之类的乱码, utf
    编码的可以正常显示.

    ![git diff 乱码问题][git diff encoding problem]

    `Git Attributes`提供了对某个文件(目录)自定义diff处理程序的功能. 官方文档里
    提供了两个处理word文档和image文件diff的例子. 参考其思路, 我们这里也使用该
    功能来对不同文件进行编码转换以解决diff乱码.

    ```bash
    # <file pattern> diff=<filter>
    # 通过观察发现<filter>好像可以任意指定, 只要后面配置git时对应上就行.
    # 将下面这一行加入你项目根路径下的`.gitattributes`里.
    新建文本文档cp936.txt diff=gbk2utf

    # 配置diff的<filter>使用<diff-with-iconv>可执行文件进行diff显示.
    # git config diff.<filter>.textconv <diff-with-iconv>

    chmod +x gbk2utf.sh
    export PATH=$PATH:/path/to/gbk2utf.sh
    git config diff.gbk2utf.textconv gbk2utf.sh
    ```

    <diff-with-iconv>带转码功能的diff脚本, 从cp936转换为utf8. 我将其命名为
    `gbk2utf.sh`

    ```bash
    #!/usr/bin/env bash
    iconv -f cp936 -t utf-8 "$1" | less
    ```

    ![git diff 乱码问题解决][git diff encoding problem solved]





<!-- Reference -->
[git diff encoding problem]: https://raw.githubusercontent.com/rainvestige/PicGo/master/2022/11/06/567f.png
[git diff encoding problem solved]: https://raw.githubusercontent.com/rainvestige/PicGo/master/2022/11/06/df2e.png
