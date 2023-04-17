# Aliyun视频上传
[Aliyun 视频上传](https://help.aliyun.com/document_detail/52204.html?spm=a2c4g.55398.0.0.1f7ea3939nI80I)
## 正常React项目中使用

>可以通过引入aliyun-upload-vod来使用（使用"aliyun-upload-vod": "^1.0.6",）进行使用

```js
import React,{useState}from 'react';
import { drawerWidth } from '../../../../utils/property';
import { Button, Upload, Drawer, Form, Space, Progress,message,Input, Row, Col} from 'antd';
import { UploadOutlined } from '@ant-design/icons';
import AliyunUpload from "aliyun-upload-vod";
import {refreshAliyuncred, getAliyuncred} from "@services/Conference";

let add=null;
const drawerFormItemLayout = {
    labelCol:   { span: 6 },
    wrapperCol: { span: 18 }
};
const UploadVideo = ({liveId,conferenceId,handleRefresh}) => {
    //上传
    const videoProps = {
        name: "video",
    
        customRequest: async (info) => {
          //手动上传
            const isJpgOrPng =
                info.file.type === "video/mp4" ||
                info.file.type === "video/rm" ||
                info.file.type === "video/rmvb";
            if (!isJpgOrPng) {
                message.error("只能上传 mp4/rm/rmvb 格式!");
                return;
            }
            if (info.file.type === "video/mp4") setVideoFormat("mp4");
            if (info.file.type === "video/rm") setVideoFormat("rm");
            if (info.file.type === "video/rmvb") setVideoFormat("rmvb");
            
            add = createUploader()
            var userData = '{"Vod":{}}';
            add.addFile(info.file, null, null, null, userData);
            setFileName(info.file.name)
            setVideoSize(info.file.size / 1024)
            await add.startUpload()
    
    
        },
        onChange(info) {
            const isJpgOrPng =
                info.file.type === "video/mp4" ||
                info.file.type === "video/rm" ||
                info.file.type === "video/rmvb";
            if (!isJpgOrPng) {
                info.file.status = "error";
            }
            if (info.file.status === "uploading") {
                info.file.status = "done";
            }
            setProgressPercent(0)
        },
    };

    //使用阿里云关键步骤
    const createUploader = () => {
        let uploader = new AliyunUpload.Vod({

            timeout: 1200000,
            partSize: 1048576,
            parallel: 100,
            retryCount: 3,
            retryDuration: 2,
            region: 'cn-shanghai',
            userId: '1303984639806000',
            // 添加文件成功
            addFileSuccess: (uploadInfo) => {
            if (uploadInfo !== null) {
                uploader.startUpload()
            }
            
            },
            // 开始上传
            onUploadstarted: async (uploadInfo) => {
            setBtnAble(true)
            if (!uploadInfo.videoId) {
                let params = {
                fileName: uploadInfo.file.name,
                title: uploadInfo.file.name.substring(0, uploadInfo.file.name.lastIndexOf(".")),
                companyCode: 'CONFERENCE', //传到哪里去
                mediaType: 1, //内容名称
                cateId:0 //要传的cateId
                }
            getAliyuncred(params).then(res => { //getAliyuncred获取阿里云的上传凭证（接口）
                if (res.status_code === 200) {
                    let data = res.data
                    let uploadAuth = data.uploadAuth
                    let uploadAddress = data.uploadAddress
                    let videoId = data.videoId
                    setVideoId(data.videoId)
                    uploader.setUploadAuthAndAddress(uploadInfo, uploadAuth, uploadAddress, videoId)
                }
                })
            } else {
                // 如果videoId有值，根据videoId刷新上传凭证
                refreshAliyuncred({ videoId: uploadInfo.videoId }).then(res => {  //refreshAliyuncred刷新阿里云的上传凭证（接口）
                if (res.status_code === 200) {
                    let data = res.data
                    let uploadAuth = data.uploadAuth
                    let uploadAddress = data.uploadAddress
                    let videoId = data.videoId
                    setVideoId(data.videoId)
                    uploader.setUploadAuthAndAddress(uploadInfo, uploadAuth, uploadAddress, videoId)
                }
                })
            }
            },
            // 文件上传成功
            onUploadSucceed: function (uploadInfo) {
            setVideoStatus(200)
            setBtnAble(false)
            },
            // 文件上传失败
            onUploadFailed: function (uploadInfo, code, message) {
            message.error('上传失败')
            setBtnAble(false)
            },
            // 取消文件上传
            onUploadCanceled: function (uploadInfo, code, message) {
            setBtnAble(false)
            },
            // 文件上传进度，单位：字节, 可以在这个函数中拿到上传进度并显示在页面上
            onUploadProgress: function (uploadInfo, totalSize, progress) {
            setProgressPercent(Math.ceil(progress * 100))
            },
            // 上传凭证超时
            onUploadTokenExpired: function (uploadInfo) {
            // 上传大文件超时, 如果是上传方式一即根据 UploadAuth 上传时
            // 需要根据 uploadInfo.videoId 调用刷新视频上传凭证接口
            refreshAliyuncred({ videoId: uploadInfo.videoId }).then(res => {
                if (res.status_code === 200) {
                let data = res.data
                let uploadAuth = data.uploadAuth
                setVideoId(data.videoId)
                uploader.resumeUploadWithAuth(uploadAuth)
                }
            })
            },
            // 全部文件上传结束
            onUploadEnd: function (uploadInfo) {
            setBtnAble(false)
            }
        })
        return uploader
    }
        
    return (
      
        <Form form={uploadReplayForm}  {...drawerFormItemLayout}>
            <Form.Item 
            label="回放视频名称"
            name='title'
            rules={[
                {
                    required: true,
                    message: '请输入回放视屏名称',
                },
            ]}
            >
                <Input/>
            </Form.Item>
            <Form.Item
                label="回放视频"
                className='uploadReplay'
                name='video'
                rules={[
                    {
                        required: true,
                        message: '',
                    },
                    {
                        validator: async (rule, value) => {
                            console.log(value);
                            if (value === undefined) {
                            throw "请上传视频！";
                            }
                        },
                        },
                ]}
            >
                <Upload {...videoProps} maxCount={1}>
                    <Button icon={<UploadOutlined />} disabled={btnAble}>上传视频</Button>
                </Upload>
            </Form.Item>
            <Row>
                <Col span={6}></Col>
                <Col span={18}>
                <Progress
                strokeColor={{
                    '0%': '#108ee9',
                    '100%': '#87d068',
                }}
                percent={progressPercent}
                style={{opacity:progressPercent>-1?1:0}}
                />
                </Col>
            </Row>
            
        </Form>
    );
    }

    export default UploadVideo;

```

## 在Next.js中使用阿里云视频上传
引入aliyun的sdk
>在Next项目中使用aliyun上传时可以直接在对应的项目中引用 import Script from 'next/script';直接使用`<Script></Script>`即可
```js
<Script src="/aliyun-upload-sdk-1.5.4/lib/es6-promise.min.js"></Script>
<Script src="/aliyun-upload-sdk-1.5.4/lib/aliyun-oss-sdk-6.17.1.min.js"></Script>
<Script src="/aliyun-upload-sdk-1.5.4/aliyun-upload-sdk-1.5.4.min.js"></Script>
```

使用方法同上
```` js

    const createUploader = () => {
        let uploader = new (window as any).AliyunUpload.Vod({

            timeout: 1200000,
            partSize: 1048576,
            parallel: 100,
            retryCount: 3,
            retryDuration: 2,
            region: 'cn-shanghai',
            userId: '1303984639806000',
            // 添加文件成功
            addFileSuccess: (uploadInfo) => {
                if (uploadInfo !== null) {
                    uploader.startUpload()
                }

            },
            // 开始上传
            onUploadstarted: async (uploadInfo) => {
                setShowProgress(true);
                if (!uploadInfo.videoId) {
                    let params = {
                        fileName: uploadInfo.file.name,
                        title: uploadInfo.file.name.substring(0, uploadInfo.file.name.lastIndexOf(".")),
                        companyCode: 'CASE',
                        mediaType: 1,
                        cateId: 14131
                    }
                    await getAliyuncred(params).then(res => {
                        if (res) {
                            let data: any = res;
                            let uploadAuth = data.uploadAuth;
                            let uploadAddress = data.uploadAddress;
                            let videoId = data.videoId;
                            setVideoId(data.videoId);
                            uploader.setUploadAuthAndAddress(uploadInfo, uploadAuth, uploadAddress, videoId);
                        }
                    })
                } else {
                    // 如果videoId有值，根据videoId刷新上传凭证
                    refreshAliyuncred({ videoId: uploadInfo.videoId }).then(res => {
                        if (res) {
                            let data = res as UploadAliyunParams;
                            let uploadAuth = data.uploadAuth;
                            let uploadAddress = data.uploadAddress;
                            let videoId = data.videoId;
                            setVideoId(data.videoId);
                            uploader.setUploadAuthAndAddress(uploadInfo, uploadAuth, uploadAddress, videoId);
                        }
                    })
                }
            },
            // 文件上传成功
            onUploadSucceed: function (uploadInfo) {
                console.log(uploadInfo)
                let videoItem = {
                    name: uploadInfo.file.name,
                    videoId: uploadInfo.videoId,
                }
                setVideoList(videoItem);
                caseForm.setFieldValue('video', videoItem);
                caseForm.setFields([
                    { name: 'video', errors: null }
                ]);
                setShowProgress(false);
            },
            // 文件上传失败
            onUploadFailed: function (uploadInfo, code, message) {
                message.error('上传失败')
            },
            // 取消文件上传
            onUploadCanceled: function (uploadInfo, code, message) {
            },
            // 文件上传进度，单位：字节, 可以在这个函数中拿到上传进度并显示在页面上
            onUploadProgress: function (uploadInfo, totalSize, progress) {
                setProgressPercent(Math.ceil(progress * 100));
            },
            // 上传凭证超时
            onUploadTokenExpired: function (uploadInfo) {
                // 上传大文件超时, 如果是上传方式一即根据 UploadAuth 上传时
                // 需要根据 uploadInfo.videoId 调用刷新视频上传凭证接口
                refreshAliyuncred({ videoId: uploadInfo.videoId }).then(res => {
                    if (res) {
                        let data = res as UploadAliyunParams;
                        let uploadAuth = data.uploadAuth;
                        setVideoId(data.videoId);
                        uploader.resumeUploadWithAuth(uploadAuth);
                    }
                })
            },
            // 全部文件上传结束
            onUploadEnd: function (uploadInfo) {
            }
        })
        return uploader
    }
````
