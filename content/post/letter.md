---
title: "字符动画制作"
date: 2023-09-08T20:06:23+08:00
draft: false
tags: [photo,image,svg]
categories: [tech]
---
## 视频逐帧提取

- 运用脚本来完成手动工作，即n秒一次截屏

```
tell application "IINA"
 reopen
 activate
 delay 0.5
 tell application "System Events"
  keystroke (ASCII character 32)
  delay 0.1
 end tell
 repeat
  tell application "System Events"
   keystroke "s" using {command down}
  end tell
  delay 0.5
 end repeat
end tell
```

- 这里的第三个delay是来控制截屏间隙的，```delay 0.5```就是0.5秒截一次屏。

- 这个脚本只试用于Mac电脑，因为command+s是Mac上的快捷键。

## png转svg，再转png

- svg就是字符矢量图，但是它不能导入视频软件来制作视频，所以还要再转回png。

1. copy pics to tencent server

```
scp $HOME/Pictures/Screenshots/*.png mahaoliang:/home/ubuntu/works/pics/
```

- 这里我是上传到服务器上搞的，

2. install app

```
sudo apt-get install caca-utils
sudo apt-get install librsvg2-bin
sudo apt install imagemagick
```

3. run script

```
./run.sh
```

- 因为有很多张图片，所以写了一个脚本run.sh，就是将png->svg->png的动作重复，命令如下：

```
# clean
mkdir -p "${svg}"
mkdir -p "${output}"

# 使用循环遍历目录中的每个文件
for file in "${pics}"/*.png; do
    # 检查文件是否是普通文件
    if [[ -f $file ]]; then
        filename=$(basename "$file")
        filename="${filename%.*}"
        echo "processing $file"
        # 这里可以添加你的处理逻辑
        img2txt -W 200 -f svg "$file" >${svg}/"${filename}".svg
    fi
done

for file in "${svg}"/*.svg; do
    # 检查文件是否是普通文件
    if [[ -f $file ]]; then
        filename=$(basename "$file")
        filename="${filename%.*}"
        echo "processing $file"
        # 这里可以添加你的处理逻辑
        rsvg-convert "${file}" >${output}/"${filename}".png
    fi
done

#convert ${output}/*.png  output.gif
tar -zcvf output-${timestamp}.tar.gz ${output}
```

- 这里面```#convert ${output}/*.png  output.gif```,可以把#去掉，这样就可以直接制作字符动图了（gif）。

4. download

```
scp mahaoliang:/home/ubuntu/works/output.png .
```

## 动画制作

- 我用的是苹果的final cut pro，试用期是90天，假如到期了话就可以输入一下代码，重置时限：

```
cd ~
cd Library/Application\ Support   
ll -a
rm .ffuserdata
```

- 将所有图片依次导入软件，建立复合片段，再调整速度就好了。

![图片不见啦](https://cdn.mahaoliang.tech/images/202309082216720.png)

## 结尾

最后送大家一个GIF

![图片不见啦](https://cdn.mahaoliang.tech/images/202309082257419.gif)

视频 <https://cdn.mahaoliang.tech/images/202309082303161.mp4>
