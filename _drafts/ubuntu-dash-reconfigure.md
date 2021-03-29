## ubuntu执行脚本数组括号报错

原因是默认用的dash不支持，最好换回bash

查看与使用

先用命令ls -l /bin/sh 看看

结果是： /bin/sh -> dash

我们会发现Ubuntu默认采用的是 dash

如果要修改默认的sh，可以采用命令

sudo dpkg-reconfigure dash

然后选择【否】

成功后再执行ls -l /bin/sh 看看

结果是： /bin/sh -> bash

修改成功！

当然我们也可以使用

sudo dpkg-reconfigure dash

把sh修改回去
