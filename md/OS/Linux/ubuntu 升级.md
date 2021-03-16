ubuntu 升级

ubuntu 升级

1. sudo apt update && sudo apt dist-upgrade && sudo apt autoremove
2. sudo do-release-upgrade -d

3. 如果出现 Your python3 install is corrupted. Please fix the '/usr/bin/python3' symlink. 错误

sudo ln -sf /usr/bin/python3.5 /usr/bin/python3 再继续执行2