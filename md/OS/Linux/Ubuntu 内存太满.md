Ubuntu 内存太满

Ubuntu 内存太满

1、找到当前的kenerl
sudo uname -a
2、找到老的image
sudo dpkg --get-selections|grep linux 或者
aptitude search ~ilinux-image

3、删掉老的
sudo apt-get purge linux-image-4.13.0-36-generic

4、1 not to upgrade