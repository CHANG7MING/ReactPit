# Aliyun播放器
[Aliyun Web播放器](https://help.aliyun.com/document_detail/125547.html?spm=a2c4g.125548.0.0.5e603981aDgpd2)
## 正常React项目中使用
>需要引入阿里云播放器的sdk  
⚠️注意 在不同下引用不同
React
````js
  <link rel="stylesheet" href="https://g.alicdn.com/de/prismplayer/2.13.2/skins/default/aliplayer-min.css"/>
  <script type="text/javascript" charset="utf-8" src="https://g.alicdn.com/de/prismplayer/2.13.2/aliplayer-min.js"></script>
````

使用 Aliyun播放器
````js
import React, { Fragment, useEffect } from 'react';
import { Modal } from 'antd';
import * as _ from 'lodash';  //引入lodash
import {getVideoUrlApi} from "@services/Conference"; //获取播放凭证


let player = null;
const VideoReplay = ({videoOpen,videoClose,liveReplayId,videoName}) => {
    //视频播放器配置
    const playerSkinLayout = [
        {name: "bigPlayButton", align: "blabs", x: 30, y: 80},
        {name: "H5Loading", align: "cc"},
        {name: "errorDisplay", align: "tlabs", x: 0, y: 0},
        {name: "infoDisplay"},
        {name:"tooltip", align:"blabs",x: 0, y: 56},
        {name: "thumbnail"},
        {
            name: "controlBar", align: "blabs", x: 0, y: 0,
            children: [
                {name: "progress", align: "blabs", x: 0, y: 44},
                {name: "playButton", align: "tl", x: 15, y: 12},
                {name: "timeDisplay", align: "tl", x: 10, y: 7},
                {name: "fullScreenButton", align: "tr", x: 10, y: 12},
                // {name:"subtitle", align:"tr",x:15, y:12},
                {name:"setting", align:"tr",x:15, y:12},
                {name: "volume", align: "tr", x: 5, y: 10}
            ]
        }
    ];

    //播放器播放地址
    const playMp4WithoutAuth = videoUrl => {
        player = new  window.Aliplayer({ //因为使用了sdk引入
            id: 'J_prismPlayer',//播放器绑定
            width: '100%',
            height: '100%',
            source: videoUrl,//播放来源
            autoplay: true,
            preload: true,
            controlBarVisibility: 'hover',
            useH5Prism: true,
            skinLayout: playerSkinLayout
        }, function (player) {
            player.play();
        });

        player.on('ready', function (e) {
        });

        player.on('play', function (e) {
        });

        player.on('pause', function (e) {
        });

        player.on('completeSeek', function (e) {
        });

        player.on('error', function (e) {
        });
    };

    const getVideoPlayInfo =  (playauth,videoId,status) => {

        if (!!playauth &&  window.Aliplayer) {
            if (status === 'Normal') {
                player = new  window.Aliplayer({
                    id: 'J_prismPlayer',
                    width: '100%',
                    height: '100%',
                    vid:videoId,  //aliyun中的播放视频的videoId
                    playauth:playauth, //aliyun中播放视频的播放凭证
                    autoplay: true,
                    preload: true,
                    controlBarVisibility: 'hover',
                    useH5Prism: true,
                    skinLayout: playerSkinLayout
                }, function (player) {
                    player.play();
                });

                player.on('ready', function (e) {
                });

                player.on('play', function (e) {
                });

                player.on('pause', function (e) {
                });

                player.on('completeSeek', function (e) {
                });

                player.on('error', function (e) {
                });
            }else{}     
        } else {
            //
        }
    };
    useEffect(() => {
        if(videoOpen){
            setTimeout(()=>{
                getVideoUrlApi({liveReplayId:liveReplayId}).then(res=>{ //获取上传凭证
                    if(res.status_code === 200){
                        if(res.data.type===2)getVideoPlayInfo(res.data.videoUrl.playAuth,res.data.videoId,res.data.videoUrl.status)
                        else playMp4WithoutAuth(res.data.videoUrl)
                    }
                })
            },1000)             
        }
    }, []);
    return(
        
        <Fragment>
            <Modal
                title={`预览视频--${videoName}`}
                width={1000}
                destroyOnClose
                open={videoOpen}
                onClose={videoClose}
                onCancel={videoClose}
                maskClosable={false}
                footer={null}
            >

            //aliyun主要播放视频方法
            <div className='videoBox'>
                <div id="J_prismPlayer"></div>
            </div>

            </Modal>

    </Fragment>
    )
        
}

export default VideoReplay;

````

## React+Next使用时
>在Next项目中使用aliyun上传时可以直接在对应的项目中引用 import Script from 'next/script';直接使用`<Script></Script>`即可
````js
 <Script  src="https://g.alicdn.com/de/prismplayer/2.13.2/aliplayer-min.js"/>
````

使用aliyun播放器
````js
import {useEffect} from 'react';
import Script from 'next/script';
import {getVodMediaInfo} from '@/services/Resource.service';


let player = null;

export default function () {
   
// 视频点播 - 主题配置
    const playerSkinLayout = [
        { name: "bigPlayButton", align: "blabs", x: 30, y: 80 },
        { name: "H5Loading", align: "cc" },
        { name: "errorDisplay", align: "tlabs", x: 0, y: 0 },
        { name: "infoDisplay" },
        { name: "tooltip", align: "blabs", x: 0, y: 56 },
        { name: "thumbnail" },
        {
            name: "controlBar", align: "blabs", x: 0, y: 0,
            children: [
                { name: "progress", align: "blabs", x: 0, y: 44 },
                { name: "playButton", align: "tl", x: 15, y: 12 },
                { name: "timeDisplay", align: "tl", x: 10, y: 7 },
                { name: "fullScreenButton", align: "tr", x: 10, y: 12 },
                // {name:"subtitle", align:"tr",x:15, y:12},
                { name: "setting", align: "tr", x: 15, y: 12 },
                { name: "volume", align: "tr", x: 5, y: 10 }
            ]
        }
    ];

    // 视频预览
    const videoPreview = async (mediaId) => {
        if (!mediaId) {
            return;
        }
        if (player) {
            player.dispose();
        }

        const playInfo: any = await getVodMediaInfo({ mediaId }); //获取playauth
        if (!!playInfo && (window as any).Aliplayer) {
            if (playInfo.status === 'Normal') {
                const playauth = playInfo.playAuth;
                player = new (window as any).Aliplayer({
                    id: 'J_prismPlayer',
                    width: "760px",
                    height: "510px",
                    vid: mediaId,
                    playauth,
                    autoplay: false,
                    isLive: false,
                    rePlay: false,
                    playsinline: false,
                    preload: true,
                    skinLayout: playerSkinLayout
                }, function (player) {

                });
            }
        }
    };


    return <>
        <Script
            src="https://g.alicdn.com/de/prismplayer/2.13.2/aliplayer-min.js"
            onLoad={() => {
                videoPreview();
            }}
        />

                <div >
                    <div id="J_prismPlayer"></div>
                </div>

    </>;
}

````