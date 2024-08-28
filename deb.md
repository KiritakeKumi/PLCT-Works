软件包整理的几个东西

1. so库生成最好有版本号 没版本号也行
2. 需要整理成git仓 然后能从 git仓生成dsc
3. etc目录下禁止放配置文件 这个是用户改的位置 一般要放/usr/share下面
4. 禁止放和系统版本不符合的python库
5. ko生成问题 重编内核就会失效 而且ko的位置是要随内核走的

相关材料
https://www.debian.org/doc/debian-policy/
https://www.debian.org/doc/manuals/maint-guide/
https://www.debian.org/doc/devel-manuals



做成一个大的git仓,然后然后生成细分的deb包
dpkg-deb直接封包 这个不可取