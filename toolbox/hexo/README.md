# Hexo Personalization  

## 修改侧边栏头像为圆形  

修改文件themes\next\source\css\_common\components\sidebar\sidebar-author.style，修改其中的.site-author-image段代码，改为如下形式：  
```css  
.site-author-image {
  display: block;
  margin: 0 auto;
  max-width: 96px;
  height: auto;
  border: 2px solid #333;
  padding: 2px;
  border-radius: 50%;
}
```  

如果需要添加鼠标停留在上面发生旋转效果，那么添加如下代码：  
```css  
.site-author-image {
  display: block;
  margin: 0 auto;
  max-width: 96px;
  height: auto;
  border: 2px solid #333;
  padding: 2px;
  border-radius: 50%
  webkit-transition: 1.4s all;
  moz-transition: 1.4s all;
  ms-transition: 1.4s all;
  transition: 1.4s all;
}
.site-author-image:hover {
  background-color: #55DAE1;
  webkit-transform: rotate(360deg) scale(1.1);
  moz-transform: rotate(360deg) scale(1.1);
  ms-transform: rotate(360deg) scale(1.1);
  transform: rotate(360deg) scale(1.1);
}
```  

## 链接样式  

原来版本的内链样式跟默认的字体很类似，这里进行修改。将链接文本设置为蓝色，鼠标划过时文字颜色加深，并显示下划线。修改文件themes\next\source\css\_common\components\post\post.styl，**添加**如下css样式：  
```css  
.post-body p a{
  color: #0593d3;
  border-bottom: none;
  &:hover {
    color: #0477ab;
    text-decoration: underline;
  }
}
```    
  
## 文章链接唯一化  

默认设置下，文章是以发布时间为路径，修改文章内容重新生成静态文件时间会发生更改，不利于搜索和分享，创建唯一永久链接是很好的选择。  

安装插件：  
```  
$ npm install hexo-abbrlink --save
```  

在站点配置文件 _config.yml 中查找代码 permalink，将其更改为:  
```  
permalink: posts/:abbrlink/  # “posts/” 可自行更换
```  

并且添加如下代码：  
```  
# abbrlink config
abbrlink:
  alg: crc32  # 算法：crc16(default) and crc32 
  rep: hex    # 进制：dec(default) and hex
```  

可选择模式：  
- crc16 & hex  
- crc16 & dec  
- crc32 & hex  
- crc32 & dec  

配置完成之后，需要清除已生成的静态文件和缓存，然后重新生成：  
```  
hexo clean  
hexo g
```  
  

# References  

http://www.vitah.net/posts/20f300cc/


