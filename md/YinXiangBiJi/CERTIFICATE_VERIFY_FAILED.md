CERTIFICATE_VERIFY_FAILED

CERTIFICATE_VERIFY_FAILED

**5、repo upload权限问题(http://192.168.120.246/: <urlopen error [SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed (_ssl.c:590)>)**

（1）export PYTHONHTTPSVERIFY=0

（2）repo upload --re="xiangsun" --no-verify