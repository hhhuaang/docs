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
5. main.exe为python代码用nuitka打包生成的exe程序，如果害怕有毒，可以自己执行python代码，代码已经帖在了下面
```shell
#main.exe脚本环境安装
python -m pip install --upgrade pip
pip install requests
pip install tqdm
pip install pywin32
```
```python
import configparser
import os
import subprocess
import tkinter as tk
from tkinter import filedialog
import requests
from tqdm import tqdm
from urllib.parse import urlparse
import hashlib
import win32com.client


def calculate_md5(file_path):
    md5 = hashlib.md5()
    with open(file_path, 'rb') as f:
        for chunk in iter(lambda: f.read(4096), b''):
            md5.update(chunk)
    return md5.hexdigest()


def download_file(url, local_path, expected_md5=None):
    if os.path.exists(local_path):
        # File already exists, check MD5 if expected_md5 is provided
        if expected_md5 and calculate_md5(local_path) == expected_md5:
            # print(f"File '{os.path.basename(local_path)}' already exists and has the correct MD5. Skipping download.")
            return

    with requests.get(url, stream=True) as response:
        total_size = int(response.headers.get('content-length', 0))
        block_size = 1024
        progress_bar = tqdm(total=total_size, unit='B', unit_scale=True, desc=os.path.basename(local_path))
        with open(local_path, 'wb') as file:
            for data in response.iter_content(block_size):
                progress_bar.update(len(data))
                file.write(data)
        progress_bar.close()


def download_and_parse_config(config_url):
    response = requests.get(config_url)
    config_data = response.json()
    return config_data


def download_module_files(module, download_dir):
    for file in module['files']:
        file_url = file['url']
        file_name = os.path.basename(urlparse(file_url).path)
        file_path = os.path.join(download_dir, file_name)
        expected_md5 = file.get('md5')  # Get expected MD5 from config, if available
        download_file(file_url, file_path, expected_md5)


def download():
    oss_config_url = "https://zhihuicaochang.oss-cn-hangzhou.aliyuncs.com/dist/update.config.json"
    local_download_dir = "."

    # Download and parse the configuration file
    config_data = download_and_parse_config(oss_config_url)

    # Download module files
    for module in config_data['modules']:
        module_name = module['name']
        if module_name != 'install':
            module_download_dir = os.path.join(local_download_dir, module_name)
            os.makedirs(module_download_dir, exist_ok=True)
            download_module_files(module, module_download_dir)


# 判断python路径对应的版本是否为3.9
def is_python_39(file_path):
    try:
        # 判断file_path是否是python.exe文件
        if not file_path.endswith("python.exe"):
            return None
        # 判断file_path是否存在
        if not os.path.exists(file_path):
            return None
        # 获取文件的完整路径
        full_path = os.path.abspath(file_path)
        # 检测Python版本
        output = subprocess.check_output([full_path, "--version"], stderr=subprocess.STDOUT, text=True)
        if "Python 3.9" in output:
            return full_path
        return None
    except subprocess.CalledProcessError as e:
        print(f"Failed to get Python version: {e}")
        return None


# 让用户选择python.exe文件
def select_python_exe():
    # 初始化tkinter
    root = tk.Tk()
    root.withdraw()  # 不显示主窗口
    # 使用while循环确保用户选择了正确的文件
    while True:
        # 弹出文件选择对话框，限制文件类型只能是名为python.exe的可执行文件
        file_path = filedialog.askopenfilename(title="选择python.exe文件",
                                               filetypes=[("Executable files", "python.exe")])

        full_path = is_python_39(file_path)
        if full_path is not None:
            return full_path


# 创建或者更新config.ini文件
def create_or_update_config(package='DEFAULT', section='PythonExePath', value='python.exe'):
    config = configparser.ConfigParser()
    if os.path.exists('config.ini'):
        config.read('config.ini')
    # 如果不存在package则创建
    if not config.has_section(package):
        config['DEFAULT'] = {section: value}
    else:
        config[package][section] = value
    with open('config.ini', 'w') as configfile:
        config.write(configfile)
    print("The path and version have been saved to config.ini")


# 读取config.ini文件
def read_config(package='DEFAULT', section='PythonExePath'):
    # 检测是否存在config.ini文件
    if not os.path.exists('config.ini'):
        return None
    # 读取配置文件以获取信息
    config = configparser.ConfigParser()
    config.read('config.ini')
    # 如果不存在package则返回None
    if package not in config:
        return None
    # 如果不存在section则返回None
    if section not in config[package]:
        return None
    full_path = config[package][section]
    if is_python_39(full_path) is None:
        return None
    return full_path


def get_python_path():
    full_path = read_config()
    if full_path is None:
        full_path = select_python_exe()
        create_or_update_config(value=full_path)
    return full_path

def get_first_level_directories(directory="."):
    """
    获取指定目录下的第一层目录列表
    :param directory: 要遍历的目录路径
    :return: 第一层目录的名称列表
    """
    for root, dirs, files in os.walk(directory):
        # 由于os.walk会递归遍历所有子目录，所以只取第一层目录就可以返回
        return dirs  # 返回第一层目录列表


def create_desktop_shortcut(name, target_path, working_directory=None, icon_path=None):
    """
    创建一个Windows桌面快捷方式。

    :param name: 快捷方式的名称，不包含.lnk扩展名
    :param target_path: 目标应用程序的完整路径
    :param working_directory: 快捷方式的工作目录（可选）
    :param icon_path: 快捷方式使用的图标文件的完整路径（可选）
    """
    desktop = os.path.join(os.path.join(os.environ['USERPROFILE']), 'Desktop')
    shortcut_path = os.path.join(desktop, f"{name}.lnk")

    shell = win32com.client.Dispatch('WScript.Shell')
    shortcut = shell.CreateShortCut(shortcut_path)
    shortcut.TargetPath = target_path
    if working_directory:
        shortcut.WorkingDirectory = working_directory
    else:
        shortcut.WorkingDirectory = os.path.dirname(target_path)
    if icon_path:
        shortcut.IconLocation = icon_path
    else:
        shortcut.IconLocation = target_path
    shortcut.save()

# 为每个目录生成start.bat文件
def create_start_bat():
    # 获取python路径
    python_exe_path = get_python_path()

    # 遍历本目录下的文件夹
    # 获取当前目录
    current_directory = get_first_level_directories()
    for directory in current_directory:
        # 判断文件夹中是否存在一个名为name的文件
        name_file = os.path.join(directory, 'name')
        if os.path.exists(name_file):
            # 读取name文件中的内容
            with open(name_file, 'r', encoding='utf-8') as file:
                name = file.read()
            # 生成index.pyc文件
            command = f'{python_exe_path} index.pyc\n'
            # 拼接start.bat文件地址
            start_bat_path = os.path.join(directory, 'start.bat')
            # 写入命令
            with open(start_bat_path, 'w') as bat_file:
                bat_file.write(command)
            # 获取完整的start.bat文件路径
            start_bat_path = os.path.abspath(start_bat_path)
            print(f"Created {start_bat_path} named {name}")

            # 在桌面生成start.bat文件的快捷方式
            create_desktop_shortcut(name, start_bat_path)

        else:
            #判断文件夹内是否存在start.bat文件
            start_bat_path = os.path.join(directory, 'start.bat')
            if os.path.exists(start_bat_path):
                # 删除start.bat文件
                os.remove(start_bat_path)
                print(f"Deleted {start_bat_path}")


#检测全局变量中的python的版本是不是3.9.13
def check_python_version():
    # 获取python路径
    python_exe_path = 'python'
    # 检测Python版本
    output = subprocess.check_output([python_exe_path, "--version"], stderr=subprocess.STDOUT, text=True)
    if "Python 3.9.13" not in output:
        print("The version of Python is not 3.9.13")
        return False
    return True


def download_and_execute_batch(url, filename="temp_batch_file.bat"):
    """
    下载远程批处理文件并执行。

    :param url: 批处理文件的URL
    :param filename: 下载文件的本地保存名称，默认为temp_batch_file.bat
    """
    try:
        # 使用requests下载批处理文件
        print(f"Downloading batch file from: {url}")
        r = requests.get(url)
        with open(filename, "wb") as file:
            file.write(r.content)
        print(f"Batch file downloaded as: {filename}")

        # 执行下载的批处理文件
        print("Executing batch file...")
        subprocess.run([filename], shell=True)
        print("Batch file executed.")

    except Exception as e:
        print(f"Error: {e}")
    finally:
        # 清理，删除下载的批处理文件
        if os.path.exists(filename):
            os.remove(filename)
            print(f"Temporary file '{filename}' has been removed.")

def download_and_run_exe(exe_url):
    # 下载文件
    response = requests.get(exe_url)

    # 将文件保存到当前目录
    exe_file = "app.exe"
    with open(exe_file, "wb") as f:
        f.write(response.content)

    try:
        # 运行下载的exe文件
        subprocess.run([exe_file])
    finally:
        # 删除下载的exe文件
        os.remove(exe_file)

#创建venv环境并安装依赖包
def create_venv_and_install_packages():
    # 创建venv环境
    subprocess.run(["python", "-m", "venv", "venv"], shell=True)
    # 定义python路径
    python_path = 'venv/Scripts/python.exe'
    # 获取完整的python路径
    python_path = os.path.abspath(python_path)
    print(python_path)
    create_or_update_config(value=python_path)
    print("The version of Python is 3.9.13")
    url = "https://zhihuicaochang.oss-cn-hangzhou.aliyuncs.com/dist/install/install.bat"
    download_and_execute_batch(url)

if __name__ == "__main__":
    # 下载文件
    download()
    # 检测全局变量中的python的版本是不是3.9.13
    while True:
        if not check_python_version():
            create_venv_and_install_packages()
            break
        else:
            # 让用户选择是安装https://zhihuicaochang.oss-cn-hangzhou.aliyuncs.com/dist/install/python-3.9.13-amd64.exe还是选择自己的python路径
            print("请选择你python环境的来源：")
            print("1. 下载Python3.9.13安装包并手动安装")
            print("2. 选择自己的Python环境")
            choice = input("请选择1或2：")
            if choice == "1":
                url = "https://zhihuicaochang.oss-cn-hangzhou.aliyuncs.com/dist/install/python-3.9.13-amd64.exe"
                download_and_run_exe(url)
            elif choice == "2":
                python_path = select_python_exe()
                create_or_update_config(value=python_path)
                break
    # 生成start.bat文件
    create_start_bat()
    print("All done!")
    input("Press Enter to exit...")

```
