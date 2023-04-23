# React-quill 富文本的使用

## 图像的可伸缩操作
![](http://rt3imiukk.hb-bkt.clouddn.com/WeChat761f75455919a13f103f6b0e0f6f986e.png)
使用React_quill时
首先要引入
````js
import ReactQuill, {Quill} from 'react-quill';
import 'react-quill/dist/quill.snow.css';
// 以上两条分别是引入 ReactQuill和ReactQuill的样式
import ImageResize from 'quill-image-resize-module-react';
// ImageResize是用来图片伸缩大小使用
//如果使用前提要使用Quill.register('modules/imageResize', ImageResize);
Quill.register('modules/imageResize', ImageResize);
    const refs = useRef()
    const [descriptions,setDescriptions] = useState(undefined);
    const onDescriptionsChange = (v) => {
            setDescriptions(v);
    };
        <ReactQuill
            ref={refs}
            modules={modules}
            theme="snow"
            value={descriptions}
            onChange={onDescriptionsChange}
            className="ql-editor  react-editor"
        />
````
在使用ReactQuill富文本工具栏的时候注意以下几点
1. ReactQuill的工具栏在标签中定义 modules属性(使用useMemo)
````js
       const modules = useMemo(() => {
        return {
            toolbar: {
                container: [
                    ["bold", "italic", "underline", "strike"],// bold:字体加粗 italic:字体斜体 underline:下划线 strike:横划线
                    ["blockquote", "code-block"],   //块引用，代码块
                    [{header: 1}, {header: 2}],  //H1,H2
                    [{list: "ordered"}, {list: "bullet"}], //有序无序列表
                    [{script: "sub"}, {script: "super"}], //下角标，上角标
                    [{indent: "-1"}, {indent: "+1"}], //行进格和退格
                    [{direction: "rtl"}],  //文本方向 从右到左
                    [{size: ["small", false, "large", "huge"]}], //字体设置
                    [
                        {
                            color: [],
                        },
                    ],//字体颜色
                    [
                        {
                            background: [],
                        },
                    ],//字体背景色
                    [{font: []}], //文字样式
                    [{align: []}], //位置
                    ["link", "image"], // a链接和图片的显示
                ],

                handlers: {
                    image: imageHander,
                },  //上传图片的方法
            },
            clipboard: {
                // toggle to add extra line breaks when pasting HTML:
                matchVisual: false
                },
            imageResize: {
                parchment: Quill.import('parchment'),
                modules: ['Resize', 'DisplaySize']
            },
        }; //当引入
    }, []);
````
2. modules工具栏中imageHander图片上传
````js
const option = { headers: { "Content-Type": "multipart/form-data" } };

const imageHander = async (action, v) => {
        const input = document.createElement("input");
        input.setAttribute("type", "file");
        input.setAttribute("accept", "image/*");
        input.click();
        input.onchange = () => {
            const file = input.files[0];
            let formData = new FormData();
            formData.append("file", file);
            //这个是啊传过去根据type保存的位置
            formData.append("type", 1);
            const hide = message.loading("上传中...", 0);
            fileApi(formData, option).then((res) => { //fileApi是上传接口，option是上传转换成form-data(看接口是否需要)
                const url = res.data; // 获取url,图片预览地址
                let quill = refs?.current?.getEditor(); //获取到编辑器本身
                const cursorPosition = quill.getSelection().index; //获取当前光标位置
                quill.insertEmbed(cursorPosition, "image", url); //插入图片
                quill.setSelection(cursorPosition + 1); //光标位置加1
                hide();
            });
        };
    };
````
## 工具栏扩展
当modules工具栏中的内容不足以满足时，使用扩展modules的方法
>使用方法如下
>
> 1.在ReactQuill上紧挨着ReactQuill定义tool工具标签
> ````js
>       <CustomToolbar />
>       <ReactQuill theme="snow" ref={quillRef}
>            modules={modules}
>            placeholder="请输入问题"
>            className="question-title-area"
>            onFocus={() => getBorder(questionIndex)}
>            defaultValue={item.body}
>            onBlur={() => changeTitle(questionIndex)}
>       />
>````
> 2.自定义工具栏
> 
>````js
>  //CustomButton就是自定义的按钮
> const CustomButton = () => <span className="iconfont">自定义按钮</span>
>
>const CustomToolbar = useCallback(() => {
>      const index = anserArea.findIndex((it) => it.id === item.id);
>      return ( <div id={`toolbar${index}`}  className="more-toolbar">
>           <button className="ql-bold"></button>    //字体加粗
>           <button className="ql-underline"></button>  //下划线
>           //颜色
>           <select
>               className="ql-color"
>               defaultValue={[]}
>               onChange={(e) => e.persist()}
>           >
>           </select>                           
>           //自定义按钮
>           <button className="ql-insertStar ">
>           <CustomButton />
>           </button>
>       </div>)
>    }, [])
>````
>⚠️注意：在自定义结束后如果有类似颜色选择select时要对select中的点击进行隔离（这一步当出现输入框点击出现工具栏，不点击的时候不出现工具栏的时候使用，如果一直显示则可省略这一步）
>````js
>useEffect(() => {
>        const index = anserArea.findIndex((it) => it.id === item.id);
>        setQuestionIndex(index + 1);
>        let c=document.querySelectorAll('.ql-color')
>        ;
>        setTimeout(() => {
>            c.forEach(item=>{
>                item.onmousedown=(e)=>{
>                    e.preventDefault();
>            }
>            })
>        }, 1000);
>        setOldValue(item.body)
>    }, []);
>````
>3.使用ReactQuill的modules结合自定义工具栏
>````js
>const modules = useMemo(() => {
>       const index = anserArea.findIndex((it) => it.id === item.id);
>        return {
>           toolbar: {
>               //需要插入的位置，在CustomToolbar中有定义id={`toolbar${index}`}，id定义什么都可以，但是两个id需要一致就可以
>               container: `#toolbar${index}`, 
>              //工具栏自定义方法集合
>               handlers: {
>                   //在CustomToolbar中定义了ql-insertStar
>                   //那么此时方法定义就为insertStar: fillBlank就是将fillBlank方法给在insertStar上
>                   insertStar: fillBlank,
>               },
>           },
>       };
>   }, []);
>````
>> fillBlank(自定义的方法)
>>
>>简单的demo在ReactQuill富文本输入后面通过点击啊自定义按钮来添加______的实现
>>````js
>> const fillBlank = () => {
>>      let quill = quillRef?.current?.getEditor(); //获取到编辑器本身
>>      const cursorPosition = quill.getSelection().index; //获取当前光标位置
>>      console.log(cursorPosition);
>>      let position = cursorPosition ? cursorPosition : 0;
>>      quill.insertText(position, '______');
>>      quill.setSelection(cursorPosition + 6); //光标新位置
>>  }
>> ````

