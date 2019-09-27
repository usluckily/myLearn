<template>
    <div class="my-video-box" :style="{width:videoWidth,height:isRotate ? videoWidth + 'px' : 'auto'}">
        <video 
        :style="{width:'100%',transform:'rotate('+rotate+'deg)',bottom:videoBottom, position:isRotate ? 'absolute' : 'relative'}"
        id="video" 
        name="media"
        autoplay
        @click="playVideo"
        @canplay="videoLoad()"
        @error="errorFunc"
        @timeupdate="getCurrentTime"
        :src="link">
        </video>
        
        <div id="myVideo-control-bar" :style="controlBarStyle" v-if="canplay">
            <div class="control">
                <div class="left">
                    {{currentTime | currentTimeFilter}} / {{duration | durationFilter}}
                </div>
                
                <div class="right">
                    <!-- <i class="el-icon-full-screen" @click="fullScreen"></i> -->
                    <!-- &nbsp;&nbsp; -->
                    <i class="el-icon-refresh-right" @click="rotateVideo"></i>
                    &nbsp;&nbsp;
                    <div @click="playVideo">
                        <i class="el-icon-video-play"  v-if="!play"></i>
                        <i class="el-icon-video-pause"  v-else></i>
                    </div>
                </div>
            </div>
            
            <div class="progress" @click="setProgress">
                <div class="item" :style="{width:progressW}" :class="{'play':play}"></div>
            </div>
            <div>

            </div>
        </div>
    </div>
</template>

<script>

function formatTime(t){
	var h = parseInt(t/3600)
	h = h<10?'0'+h:h 
	var m = parseInt(t%3600/60)
	m = m<10?'0'+m:m
	var s = parseInt(t%60)
	s = s<10?'0'+s:s
	return h+':'+m+':'+s
}

export default {
    props:{
        link:{
            type: String,
            default:'https://demo-res.cloudinary.com/video/upload/w_0.4,a_20/l_cloudinary_icon,o_50,w_60,g_south_east,y_15,x_60/dog.webm'
        }
    },
    data(){
        return{
            loading:{},
            _video:{},
            play:false,
            duration:0,
            currentTime:0,
            rotate:0,
            videoH:0,
            canplay:false,
            videoWidth:'700'
        }
    },
    mounted(){
        this._video = document.getElementById('video')
        this.videoWidth = this._video.offsetWidth
        this.loading = this.$loading(
            {
                target:'.my-video-box',
                text:'视频加载中',
                background:"rgba(0,0,0,0)"
            })
    },
    methods:{
        videoLoad(){
            this.duration = this._video.duration
            this.videoH = this._video.offsetHeight
            this.canplay = true
            this.loading.close()
        },
        getCurrentTime(){
            this.currentTime = this._video.currentTime
        },
        playVideo(){
            this.play = !this.play
        },
        fullScreen(){
            this._video.webkitRequestFullScreen();
        },
        rotateVideo(){
            this.rotate += 90
        },
        errorFunc(){
            this.$message.error('视频加载失败')
            this.loading.close()
        },
        setProgress(e){
            let target = e.target
            if(target.className != 'progress') target = target.parentNode
            let progress = e.offsetX, count = target.offsetWidth
            this._video.currentTime = (progress/count)*this.duration
        }
    },
    computed:{
        progressW(){
            if(!this.currentTime || !this.duration) return 0
            let W = document.getElementsByClassName('progress')[0].offsetWidth
            // return (this.currentTime / this.duration) * W + 'px'
            return (this.currentTime / this.duration) * 100 + '%'
        },
        isRotate(){
            return this.rotate%360 == 90 || this.rotate%360 == 270
        },
        videoBottom(){
            if(this.isRotate){
                return (this.videoWidth - this.videoH) / 2 + 'px'
            }else{
                return ''
            }
        },
        controlBarStyle(){
            if(this.isRotate){
                return {
                    width:this.videoH + 'px',
                    left:(this.videoWidth - this.videoH) / 2 + 'px'
                }
            }else{
                return ''
            }
        }
    },
    filters:{
        durationFilter(val){
            return formatTime(val)
        },
        currentTimeFilter(val){
            return formatTime(val)
        },
    },
    watch:{
        play(val){
            if(val){
                this._video.play()
            }else{
                this._video.pause()
            }
        },
        currentTime(val){
            if(val == this.duration){
                this.play = false
            }
        }
    }
}
</script>

<style lang="less">
@width:400px;
@transitionTime:.5s;
.my-video-box{
    position:relative;
    overflow: hidden;
    #video{
        transition: transform @transitionTime ease;
        background:#000;
    }
    i{
        font-size:24px;
        cursor: pointer;
    }
}
// .video-box:hover{
//     #video-control-bar{
//         bottom:10px
//     }
// }
#myVideo-control-bar{
    color:#fff;
    position:absolute;
    padding:0 10px;
    bottom:10px;
    z-index:999;
    width:100%;
    transition: bottom @transitionTime ease 1s, width @transitionTime ease;
    .control{
        display:flex;
        justify-content: space-between;
        align-items: center;
        margin-bottom:5px;
        width:100%;
        .left{
            font-size: 12px;
        }
        .right{
            display: flex;
            justify-content: space-between;
            align-items:center;
        }
    }
    .progress{
        @height:5px;
        position:relative;
        height:@height;
        background:#BBB;
        width:100%;
        border-radius:2px;
        overflow: hidden;
        cursor: pointer;
        .item{
            position:absolute;
            top:0;
            left:0;
            height:@height;
            display: block;
            background: #fff;
        }
        .item.play{
            // transition: width 1s ease;
        }
    }
}
</style>
