# macOS Mojave 连接外部显示器

我的 MacBook Pro 笔记本电脑的屏幕只有 13 英寸，编程或者看电影的时候就感觉太小了，因此我特意买了一台 24 英寸的外部显示器来解决这个问题。

由于不是 Apple 自家生产的显示器，所以无法做到开箱即用，只有在完成必要的设置之后才能够完美的工作。

# 连接外部显示器的注意事项

> - 外部显示器需要占用 CPU、内存和显卡。比较老的电脑，可能带不动 2K/4K 屏幕，尤其在播放视屏、滚动网页时会掉帧、卡顿。
> - 请关注刷新率。macOS 上实现 4K 60Hz 对线材和接口是有要求的。
> 
> From: https://blog.iwyc.cn/hidpi

# 让操作系统识别外部显示器

硬件：

- 13 英寸 Retina MacBook Pro（2013 年底生产）
- DELL P2416D 24 英寸液晶显示器，分辨率 2560x1440（2K）
- DP-Thunderbolt 转接线

连接步骤：

1. MacBook Pro 开机，
2. 外部显示器通电，
3. 连接 DP 接口，
4. 连接 Thunderbolt 接口，
5. 操作系统自动识别外部显示器，并显示相同的壁纸和菜单栏。

如果没能自动识别，就断开转接线的两端，关闭外部显示器电源之后再试一次。

# 针对外部显示器启用 HiDPI

所谓的 HiDPI，指的是：

> HiDPI 本质上是用软件的方式实现单位面积内的高密度像素。在我们的惯性思维里，高分辨率意味着更小的字体和图标，两者只能舍其一。而通过开启 HiDPI 渲染，可以**在保证分辨率不变的情况下，使得字体和图标变大**。
> 
> From: https://zhuanlan.zhihu.com/p/36913571 (written by 明基色彩管理显示器)

如果不开启 HiDPI，外部显示器上的文字和图标都非常小，和内置的 Retina 屏幕的显示效果差异非常大，当视线在两块屏幕来回切换时，眼睛很快就会疲劳。

如果有条件，**外部显示器优先选择 4K 屏，可以自动开启 HiDPI**。低于 4K 的屏幕则需要非官方途径开启。需要注意的是，即使是在 2K 分辨率的外部显示器上强制开启 HiDPI，也没有原生 Retina 细腻。

强制开启 HiDPI 的方法不止一种，但我个人喜欢用 SwitchResX 这个应用程序来开启，简单有效，步骤如下：

1. 临时关闭「系统完整性保护（System Integrity Protection, SIP）」：
   1. 重启电脑，在开机音乐响起后按住 `Command + R` 进入「恢复模式（Recovery mode）」，
   2. 打开「终端（Terminal）」，在终端输入关闭 SIP 的命令行语句：`csrutil disable` ，
   3. 重启电脑。
2. [下载 SwitchResX](https://www.madrau.com/srx_download/download.html) ，
3. 安装，然后从「系统偏好设置」里面打开 SwitchResX，
4. 选中自动识别出来的外部显示器名称，例如「DELL P2416D」，再选择「Custom Resolutions」标签，点击左下角的加号按钮，
5. 在弹出的对话框中创建一个「Scaled resolution」，横向（Horizontal）**3840** 像素，纵向（Vertical）**2160** 像素，点击 OK 按钮保存，
6. 重启电脑，打开 SwitchResX，选中外部显示器名称，这次选择「Current Resolutions」标签，最后选择「1920 x 1080, 60Hz (NTSC), HiDPI」这个**新增**的分辨率，外部显示器会自动调整，HiDPI 成功启用。
7. 重新开启 SIP：
   1. 进入恢复模式，
   2. 在终端输入：`csrutil enable` 。

# 启用次像素渲染

**次像素渲染**（subpixel antialiasing）也叫做**字体平滑**（font smoothing）。macOS Mojave 默认关闭了次像素渲染，会导致外部显示器字体模糊，可以通过下面的命令重新启用这个功能：

```bash
# https://www.howtogeek.com/358596/how-to-fix-blurry-fonts-on-macos-mojave-with-subpixel-antialiasing/

# re-enable subpixel antialiasing (font smoothing)
defaults write -g CGFontRenderingFontSmoothingDisabled -bool NO

# check the current value
defaults read -g CGFontRenderingFontSmoothingDisabled
```

注销后重新登录即可生效。

# 总结

完成以上三步，外部显示器就可以完美工作了。更多详细信息，请参考以下链接：

- [HiDPI 基础知识](https://zhuanlan.zhihu.com/p/36913571)
- [用 SwitchResX 开启 HiDPI 可能遇到的各种问题及解决方法](https://www.zhihu.com/question/35300978)
