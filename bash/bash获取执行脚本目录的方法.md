### bash获取执行脚本的目录
* 方法1：
 
     BASE_NAME=$(cd `dirname $0`;pwd)

* 方法2：

    BASE_NAME=$(dirname $(readling -f $0))


