**来源：https://github.com/revyos/revyos/issues/7**

**标题：gnome桌面环境下VLC播放器黑屏**

**描述：**

**反馈人：https://github.com/FIFCC**





### 复现情况：



使用1026版本的revyos+gdm3+gnome成功复现



使用xfce也观测到了同样的情况



![image-20231205153116857](C:\Users\Nico\Documents\GitHub\PLCT-Works\revyos-issues\issues-7.assets\image-20231205153116857.png)





通过vlc的debug输出未提示明显错误



![image-20231205153130509](C:\Users\Nico\Documents\GitHub\PLCT-Works\revyos-issues\issues-7.assets\image-20231205153130509.png)



![image-20231205153205672](C:\Users\Nico\Documents\GitHub\PLCT-Works\revyos-issues\issues-7.assets\image-20231205153205672.png)

vlc本身可以正常识别文件音视频格式  但是音视频均无法输出（疑似后端ffmpeg问题）



同时尝试关闭硬件加速/开启 均无改善



同时发现一旦出现问题关闭vlc后无法再次启动GUI  需重启系统



### 初步判断：