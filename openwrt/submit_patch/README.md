# Working with patches in the build system

## Step 1: quilt 配置

```
cat > ~/.quiltrc <<EOF
QUILT_DIFF_ARGS="--no-timestamps --no-index -p ab --color=auto"
QUILT_REFRESH_ARGS="--no-timestamps --no-index -p ab"
QUILT_SERIES_ARGS="--color=auto"
QUILT_PATCH_OPTS="--unified"
QUILT_DIFF_OPTS="-p"
EDITOR="vim"
EOF
```

+ `EDITOR` 指定编辑补丁使用的编辑器  
+ 其他变量控制patch的属性，如使用 **a/file** 和 **b/file** 格式代替默认的 **dir.orig/file** 和 **dir/file**  

## Step 2: 创建新patch 

创建新patch，要先准备源码目录   
```
make package/example/{clean,prepare} V=s QUILT=1
```

接着切换到准备的源码目录  
```
cd build_dir/target-*/example-*
```

push存在的patch  
```
quilt push -a
```

创建新的patch  
```
quilt new 000-main_code_fix.patch
```
+ patch名要以数字开头表示patch序列，后紧跟一横线和剪短的描述  
+ patch的序列号应该要高于存在的patch(**quilt series** 命令可以查看存在的patches) 

编辑文件 
 
```
quilt edit src/main.c
``` 

查看修改的内容  
```
quilt diff
```

生成patch  
```
quilt refresh
```

切换回buildroot目录
```
cd ../../
```

更新package  
```
make package/example/update V=s 
```

## Step 3: 编辑已存在的patch  

准备源码目录  
```
make package/example/{clean,prepare} V=s QUILT=1
```

切换到已准备的源码目录  
```
cd build_dir/target-*/example-*
```

应用存在的patches  
```
quilt push -a
```

查看patch列表  
```
quilt push -a
```

将要编辑的patch置顶  
```
quilt pop 005-fix_bug.patch
```

编辑文件 
```
quilt edit src/main.c
``` 

查看修改的内容  
```
quilt diff
```

生成patch  
```
quilt refresh
```

切换回buildroot目录
```
cd ../../
```

更新package  
```
make package/example/update V=s 
```

可以发现，添加新的patch和编辑已有的patch，步骤差不多，只是编辑已有的patch多了一步将patch置顶操作
