xxx:指文件名

git update-index --assume-unchanged xxx  //pull时候忽略xxx这个文件

git update-index --no-assume-unchanged   xxx  //pull时候取消忽略xxx这个文件

//忽略.idea  这个文件夹，注意后面带着  /   斜杠

git update-index --assume-unchanged .idea/