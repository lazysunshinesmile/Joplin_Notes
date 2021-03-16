打patch步骤

打patch步骤

1、git add
2、git commit -m ""

3、git diff bb3dafdb66b0e54b7430b5369ac1fa6ce67496dd（newid） da1da74636d0399e04820bf6e10a64f9d62e9b55(oldid) --name-only >diff.txt

4、cat diff.txt | xargs zip new.zip
5、git checkout da1da74636d0399e04820bf6e10a64f9d62e9b55(oldid)
6、cat diff.txt | xargs zip old.zip


git的根目录 git apply path_file