# Html2Canvas 使用

可以根据dom直接生成下载的图片
使用方法
````js
  const onErweimaClick =async () => {
        shareDownload('erweima_wrap', true) // 要下载的dom的className/id，开
    }

    const shareDownload =async (domId, isErweima) => {
        // shareCanvas方法中调用Html2Canvas domId=>要下载的dom的className/id, isErweima=>开（true）
        let res = await shareCanvas(domId, isErweima) 
        let linkA = document.createElement('a');       //创建a标签
        linkA.href = res;                // 将图片的src赋值给a节点的href
        linkA.download = location.state?.title || '问卷标题';     // 设置a节点的download属性值(设置下载名称)
        let event = new MouseEvent('click');           // 模拟鼠标click点击事件
        linkA.dispatchEvent(event);
    }


    const shareCanvas = async (domId, isErweima) => {
        // 获取想要转换的 DOM 节点
        const dom = document.getElementById(domId);
        // const dom = document.querySelector('printHtml');
        const box = window.getComputedStyle(dom);
        // DOM 节点计算后宽高
        const width = dom.offsetWidth // 获取dom 宽度
        const height = dom.offsetHeight // 获取dom 高度

        const scale = 4 // 定义任意放大倍数 支持小数
        // 获取像素比-防止模糊
        const scaleBy = DPR();
        // 创建自定义 canvas 元素
        const canvas = document.createElement('canvas');

        const rect = dom.getBoundingClientRect()

        if(isErweima){
            // 设定 canvas 元素属性宽高为 DOM 节点宽高 * 像素比
            canvas.width = width * scale;
            canvas.height = height * scale;
        }else{
            // 设置固定下载宽度
            canvas.width = 330;
            canvas.height = 620;
        }
        // 设定 canvas css宽高为 DOM 节点宽高
        canvas.style.width = `${width/2}px`;
        canvas.style.height = `${height/2}px`;
        // 获取画笔
        const context = canvas.getContext('2d');

        // 将所有绘制内容放大像素比倍
        context.scale(scale, scale);
        //关闭抗锯齿
        context.mozImageSmoothingEnabled = false;
        context.webkitImageSmoothingEnabled = false;
        context.msImageSmoothingEnabled = false;
        context.imageSmoothingEnabled = false;


        // 将自定义 canvas 作为配置项传入，开始绘制
        return await html2canvas(dom,
            { // 转换为图片
                scale: scale, // 添加的scale 参数
                width: width, // dom 原始宽度
                height: height,
                useCORS: true, // 开启跨域
                dpi: window.devicePixelRatio * 10
              }

            ).then(canvas => {
            //document.querySelector("#canvasContainer").appendChild(canvas);
            //return canvas.toDataURL("image/png");
            return canvas.toDataURL("image/png");
        });
    };
````

缩放功能

| scale | window.devicePixelRatio | The scale to use for rendering. Defaults to the browsers device pixel ratio.(用于规模呈现。默认浏览器设备像素比例。) |
| ----- | ----------------------- | ------------------------------------------------------------ |

出现原因点击下载利用Html2Canvas生成的下载图片利用window.devicePixelRatio每次都发生变化所以scale给固定值

⚠️注意：一般情况下不要使用window.devicePixelRatio
