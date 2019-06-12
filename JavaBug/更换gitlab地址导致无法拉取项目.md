最近更换了gitlab的ip地址,我在hosts改完之后导致无法拉取项目,这时候错误如下:

![QQ截图20190611160333.png](https://upload-images.jianshu.io/upload_images/5660329-21282a2cefec3d9b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这时候打开用户目录下的.ssh文件夹,找到known_hosts这个文件把和我们项目的gitlab地址那一条删除即可
