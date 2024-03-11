# AI接入相关文档

## 安装

### 安装文件下载地址
https://zhihuicaochang.oss-cn-hangzhou.aliyuncs.com/dist/main.exe

### 注意事项
1. 不要多个python环境混用，会引起未知问题
2. 切换python环境需要先手动卸载现有的python环境
3. 如果你用anaconda环境，需要选择方案二，手动选择python安装的位置并手动安装python运行用到的依赖
```shell
#python环境手动安装脚本
pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu117
pip install wmi
pip install ultralytics
pip install pydub
pip install xmltodict
pip install oss2
pip install -U openmim
mim install mmengine
mim install "mmcv>=2.0.1"
mim install "mmpose>=1.1.0"
pip install "python-socketio[client]"
pip install pygame
```
4. 如果main.exe文件被杀毒软件报毒，需要手动添加为免杀
