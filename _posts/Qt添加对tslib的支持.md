# Qt添加对tslib的支持
```
-tslib \
-I tslib_install/include \
-L tslib_install/lib
```

make install 可能会出现安装失败的情况，这不是错误，直接再编译目录下重新执行命令，可能是编译arm平台的qt有残留，所以编译失败，

$ make clean
$ make -j4
$ make install