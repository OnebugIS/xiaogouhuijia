<script>
	export default {
		onLaunch: function() {
			console.warn('当前组件仅支持 uni_modules 目录结构 ，请升级 HBuilderX 到 3.1.0 版本以上！')
			console.log('App Launch')
		},
		onShow: function() {
			console.log('App Show')
		},
		onHide: function() {
			console.log('App Hide')
		}
	}
</script>

<style lang="scss">
	/*每个页面公共css */
	@import '@/uni_modules/uni-scss/index.scss';
	/* #ifndef APP-NVUE */
	@import '@/static/customicons.css';
 
.container {
    position: relative;
    width: 100%;
    height: 100vh;
  }
  .input-box {
    position: absolute;
    left: 50%;
    transform: translateX(-50%);
    bottom: 50%;
  }
  .d_r {
    position: absolute;
    right: 10px;
    bottom: calc(179px + env(safe-area-inset-bottom));
    width: 44px;
    height: 44px;
  }
  .control-panel {
    position: absolute;
    bottom: calc(30px + env(safe-area-inset-bottom));
    left: 50%;
    transform: translateX(-50%);
    padding: 6px;
    background: #FF8C27;
    border-radius: 24px;
    .outer-layer {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 0 10px 6px;
      .avatar-box {
        width: 32px;
        height: 32px;
        // -webkit-box-shadow: 0 1px 1px -1px #fff;
        // box-shadow: 0px 0px 1px 0px #FFFFFF;
        border-radius: 50%;
        background-color: #FFFFFF;
        display:flex;
        justify-content: center;
        align-items: center;
        .avatar {
          width: 30px;
          height: 30px;
          border-radius: 50%;
        }
      }
      .nickname {
        margin-left: 6px;
        font-weight: 500;
        font-size: 14px;
        line-height: 20px;
        color: #FFFFFF;
      }
      .dotted {
        margin-left: 6px;
        width: 56px;
        height: 17px;
      }
      .refresh {
        width: 20px;
        height: 18px;
      }
      .update-time {
        margin-left: 6px;
        font-weight: 500;
        font-size: 12px;
        color: #FFFFFF;
        line-height: 20px;
      }
      .close {
        margin-left: 14px;
        width: 20px;
        height: 20px;
      }
    }
  }
 
  .inner-layer {
    display: flex;
    justify-content: space-between;
    align-items: center;
    width: 339px;
    height: 87px;
    background: #FFFFFF;
    border-radius: 24px;
    .location-icon {
      width: 30px;
      height: 30px;
      margin-left: 10px;
    }
    .txt-box {
      margin-left: 6px;
      width: 187px;
      display: flex;
      flex-direction: column;
      .address {
        // margin-left: 6px;
        // width: 187px;
        text-overflow: -o-ellipsis-lastline;
        overflow: hidden;
        text-overflow: ellipsis;
        display: -webkit-box;
        -webkit-line-clamp: 2;
        -webkit-box-orient: vertical;
      }
      .meter {
        font-weight: 500;
        font-size: 12px;
        color: #28BA36;
    }
  }
      .action-button {
        display: flex;
        flex-direction: column;
        align-items: center;
        .button-icon {
          width: 30px;
          height: 30px;
        }
        .button-text {
          margin-top: 5px;
          font-weight: 500;
          font-size: 12px;
          color: #151516;
        }
      }
      .mar {
        margin: 0 14px 0 16px;
      }
      
}
  .left {
    display: flex;
    align-items: center;
  }
  .right {
    display: flex;
    align-items: center;
  }
.mode_box {
  position: absolute;
  left: 50%; 
  top: 50%;
  transform: translate(-50%, -50%); /* 50%为自身尺寸的一半 */
  -webkit-transform: translate(-50%, -50%);
  background-color: #FFFFFF;
  border-radius: 12px;
  // padding: 20px 20px 10px;
  width: 260px;
  text-align: center;
  .mode-style {
    .title {
      font-weight: 500;
      font-size: 14px;
      line-height: 20px;
      margin-top: 10px;
    }
    .sub {
      font-size: 12px;
      margin-top: 4px;
      margin-bottom: 10px;
    }
    
  }


}
.hover {
  background-color: #FF8C27;
}

	//透明度 渐入
.fadeIn {
  -webkit-animation-name: fadeIn;
  animation-name: fadeIn
}
@keyframes fadeIn {
  0% {
    opacity: 0
  }
  to {
    opacity: 1
  }
}

//透明度 渐出
.fadeOut {
  -webkit-animation-name: fadeOut;
  animation-name: fadeOut
}
@keyframes fadeOut {
  0% {
    opacity: 1
  }
  to {
    opacity: 0
  }
}
</style>
