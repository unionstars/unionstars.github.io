---
layout: post
title:  "打造自己的Mac"
date:   2018-07-11 12:00:00

categories: tool
tags: tool life
author: "XueGang Zhang"
image: /images/logo.png
comments: true
published: true
---

#### 背景介绍
对于软件开发者而言，MacBoook Pro以其强大的终端，良好的生态以及优异的性能等诸多优势，逐渐成为软件开发的首先。本文记录一下，快速打造自己的mac为一个良好的开发工具，作为换笔记本的一个备份。


#### 基本设置

1. 升级到新版本操作系统(macOS Mojave 10.14.3)
2. 关闭菜单栏效果, 减少资源占用和产生的热量
> 系统偏好设置->辅助功能->显示, 勾选 (减弱动态效果、减少透明度) 
3. 配置睡眠保护
> 系统偏好设置->安全性与隐私->通用, 勾选(进入睡眠或开始保护程序 `立即` 要求输入密码)
4. 配置触发角
> 系统偏好设置->屏幕保护程序->触发角, 选择(右上桌面，左下启动台，右下启动屏幕保护)

#### 工具安装
1. Xcode Command Line Tools
```shell
xcode-select --install
```
2. 安装[Homebrew](https://brew.sh/)
```shell
# 这里必须设置 代理地址，否则无法安装brew
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install
```
2.1 查看[brew cask list](https://github.com/Homebrew/homebrew-cask/tree/master/Casks)

2.2 brew 安装常用工具


| 工具名称 | 安装命令 | 备注 | 
| -- | -- | -- |
| git | brew install git | 版本管理工具 | 
| wget | brew install wget | 下载工具 | 
| curl | brew install curl | 下载工具 | 
| tmux | brew install tmux | 终端复用神器 | 
| vim | brew install vim | 升级vim到8.x版本，替换系统自带的vim | 
| JDK | brew cask install java | open jdk java | 
| tree | brew install tree | 目录展示工具 | 
| lua | brew install lua | lua编程语言 | 
| core | brew install coreutils | gnu 核心工具 | 
| bin | brew install binutils | gnu bin工具 | 
| diff | brew install diffutils | gnu比较工具 | 
| find | brew install findutils | gnu查找工具 | 
| ag | brew install the_silver_searcher | 高效内容查找工具，比grep、ack更快的递归搜索文件内容 |
| go | brew install go | go语言 | 
| node | brew install node | node语言 | 
| jq | brew install jq | 命令行json处理工具与python -m json.tool 功能类似 | 
| htop | brew install htop | 命令行json处理工具与python -m json.tool 替代top | 
| axel | brew install axel | 多线程下载工具 | 
| cloc | brew install cloc | 代码统计工具 | 
| shellcheck | brew install shellcheck | shell脚本检查工具 | 
| tldr | brew install tldr | 命令行示例 | 
| ncdu | brew install ncdu | 磁盘空间占用分 |
| glances | brew install glances | 监控工具 | 
| figlet | brew install figlet | 艺术字转换 | 
| screenFetch | brew install screenFetch | 系统信息 | 
| nmap | brew install nmap | 网络扫描工具 | 
| ctop | brew install ctop | docker 容器监控工具 |
| pstree | brew install pstree | 进程树查看 |
| bash-completion | brew install bash-completion | bash不全 | 
| lolcat | brew install lolcat | 彩虹文字 |
| peco | brew install peco | go 写的极简过滤工具 |
| cowsay | brew install cowsay | ascii图片 |
| graphviz | brew install graphviz | plantuml绘图工具底层依赖 |
| cask | brew tap caskroom/cask | cask |
| cask update | brew update && brew upgrade brew cask && brew cleanup| - |
| iterm2 | brew cask install iterm2 | 终端神器 |
| google-chrome | brew cask install google-chrome | chrome |
| firefox | brew cask install firefox | firefox |
| alfred | brew cask install alfred | 快捷键工具 |
| dash | brew cask install dash | 文档工具 |
| vscode | brew cask install visual-studio-code | 微软出品的文本编辑器，首选 |
| sublimetext | brew cask install sublime-text |文本编辑器，不再推荐  |
| atom | brew cask install atom |github出品的文本编辑器，太卡，不再推荐  |
| postman | brew cask install postman |http 模拟 app  |
| wireshark | brew cask install wireshark |网络抓包工具  |
| xmind | brew cask install xmind |思维导图工具  |
| beyond-compare | brew cask install beyond-compare |比较工具  |
| youdaodict | brew cask install youdaodict |有道词典  |
| typora | brew cask install typora |markdown 工具 |
| docker | brew cask install docker | docker|
| moon | brew cask install moon | 窗口大小调整 |
| snip | brew cask install snip | 截图工具 |
| the-unarchiver | brew cask install the-unarchiver | 解压工具，可解rar |
| keka | brew cask install keka | 解压工具 |
| fonts | brew tap caskroom/fonts | 安装字体 |
| source-code-pro | brew cask install source-code-pro | Adobe 出品编程字体 |
| andriod-file-transfer | brew cask install andriod-file-transfer | 安卓文件传输工具 |
| licecap | brew cask install licecap | 录屏软件 |
| kap | brew cask install kap | 视频录屏 |
| sougou | brew cask install sougouinput | 搜狗拼音 |
| scroll-reverser | brew cask install scroll-reverser | 触摸板滑动反转 |
| go2shell | brew install go2shell | go2shell finder 直接跳转到命令行 |
| rdm | brew install rdm | rdm redis desktop manager |
| imagemagick | brew install imagemagick | 图片编辑工具 |
| msyql | brew install mysql | - |
| redis | brew install redis | - |

#### 其他设置

1. 替换HomeBrew的源

2. 配置iterm2 + oh my zsh

2.1 安装iterm2
> iterm2 > preference > profiles > colors > Color Presets > solarized dark

2.2 安装 zsh，oh-my-zsh
```bash
# 安装 zsh 及 补全
brew install zsh zsh-completions

# 安装 oh-my-zsh
curl -L https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh | sh

# 编辑 /etc/shells
sudo vim /etc/shells

# 添加 zsh
/usr/local/bin/zsh

# 修改默认shell
sudo chsh -s /usr/local/bin/zsh

# 将环境变量移到~/.env.sh
vim ~/.zshrc

# 设置主题
ZSH_THEME=pygmalion
# 设置插件
plugins=(git colored-man colorize github jira vagrant virtualenv pip python brew osx zsh-syntax-highlighting)

# ls 配色生效
unset LSCOLORS
export CLICOLOR=1
export CLICOLOR_FORCE=1

# 生效
source ~/.env.sh
```

2.3 solarized 主题配色
```bash
# clone 之
git clone https://github.com/altercation/solarized

# 配置 vim 主题
cd solarized/vim-colors-solarized/
mkdir -p ~/.vim/colors
cp colors/solarized.vim ~/.vim/colors/

# 配置 vim
vim ~/.vimrc
syntax on
set background=dark
colorscheme solarized
set backspace=2
```

2.4 设置vscode快捷命令
> Command + Shift + P -> Shell Command "install code command in PATH"

3. vscode插件安装：vscode-icons、python、go、markdown preview enhance...

4. 设置idea快捷命令
> Tools->Create Launcher Script->/usr/local/bin/idea

5. 设置sublime快捷命令
> ln -s "/Applications/Sublime Text.app/Contents/SharedSupport/bin/subl" ~/bin/subl

