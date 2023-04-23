 # CSS

## 文本省略...

单行省略
````css
        flex: 1;
        margin-top: 12px;
        height: 16px;
        font-size: 14px;
        font-weight: 400;
        color: #707070;
        line-height: 12px;
        width: 270px;
        height:24px;
        text-overflow:ellipsis;
        white-space:nowrap; 
        overflow:hidden;
````
多行省略
>多行省略关键
>````css
>   display: -webkit-box; //多行省略时 webkit-box
>   -webkit-line-clamp: 2; //行数
>   -webkit-box-orient: vertical;
>   transition: all 1s;
>````
````css
    height: 44px;
    font-size: 14px;
    font-weight: 600;
    color: #333;
    overflow: hidden;
    word-break: break-all; //注意文本中出现较多数字和字母时可能出现环行问题 ，加上这行就可以解决
    text-overflow: ellipsis; // 省略号
    display: -webkit-box; //多行省略时 webkit-box
    -webkit-line-clamp: 2; //行数
    -webkit-box-orient: vertical;
    transition: all 1s;
````
没有加`word-break: break-all;`之前
![](http://rt3imiukk.hb-bkt.clouddn.com/%E6%88%AA%E5%B1%8F2023-04-23%2016.17.09.png)
当加上`word-break: break-all;`之后
![](http://rt3imiukk.hb-bkt.clouddn.com/%E6%88%AA%E5%B1%8F2023-04-23%2016.16.23.png)
