# hexo-theme-grace
A simple and grace theme for hexo

## 风格
![style](https://github.com/beevesnoodles/hexo-theme-grace/raw/master/source/img/style.png)

## 安装

####  方法1

`git clone https://github.com/beevesnoodles/hexo-theme-grace.git  themes/grace`

#### 方法2

下载主题文件解压后放在 `themes` 目录下

#### 插件

主题采用的是 sass 编译

```
npm install hexo-renderer-sass --save
```



添加Rss

```
npm install hexo-generator-feed --save
```

博客根目录的_config.yml 中配置

```
feed:
  type: atom
  path: atom.xml
  limit: 20
```
或者themes/_config.yml 中添加

```
 rss: atom.xml
```

## 启用

修改 _config.yml 中的 theme 属性

`theme: hexo-theme-grace`

## License

MIT
