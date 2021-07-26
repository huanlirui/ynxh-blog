话不多说。全在代码注释里了

```
 // //移动端的浏览器版本都比较高，可以监听DOMContentLoaded，而不是load，load会相对慢一点点，具体按事件使用情况而定。
      document.addEventListener('DOMContentLoaded', function () {
        var USER_Agent = navigator.userAgent;

        var MOBILE_IOS = /(iPhone|iPad|iPod|iOS)/i;
        var MOBILE_Android = /(Android)/i;
        var WX = /(micromessenger)/i;
        var androidButton = document.querySelectorAll('.button-container')[0];
        var IPhoneButton = document.querySelectorAll('.button-container')[1];

        androidButton.addEventListener('click', goAndroid);
        IPhoneButton.addEventListener('click', goIPone);

        function goAndroid(e) {
          e && e.preventDefault();
          let jumpUrl = 'https://a.app.qq.com/o/simple.jsp?pkgname=com.cce.yunnanuc'; //安卓腾讯应用宝市场
          // 'https://minio.yncce.cn/happly-owner/apk/happyliveapp_cce.apk'; //悦居安卓下载地址
          // if (WX.test(USER_Agent)) {
          //     //如果在微信中打开则只显示一个提示用户的罩层
          //     alert('微信屏蔽了app跳转，请您点击右上角在浏览器打开');
          //     return;
          // } else
          if (MOBILE_Android.test(USER_Agent)) {
            // document.getElementById("WXShadow").style.display = "none";
            window.location = jumpUrl; //安卓app跳转的协议
            return;
          } else {
            alert('您不是安卓手机');
            return;
          }
          //如果没有唤起app则1秒后会跳转到下载地址，反之则不会跳转到下载地址。
          setTimeout(function () {
            window.location.href = jumpUrl;
          }, 1000);
        }

        function goIPone(e) {
          e && e.preventDefault();
          if (!MOBILE_IOS.test(USER_Agent)) {
            alert('您不是苹果手机');
            return;
          }
          let jumpUrl = 'https://apps.apple.com/cn/app/悦居生活/id1352856463';
          if (WX.test(USER_Agent)) {
            //如果在微信中打开则只显示一个提示用户的罩层
            // alert('微信屏蔽了app跳转，请您点击右上角在浏览器打开');
            // window.location.href = jumpUrl; //下载地址
            window.location.href = 'yjSchemeLink://'; //苹果app跳转的协议
            return;
          } else if (MOBILE_IOS.test(USER_Agent)) {
            // document.getElementById("WXShadow").style.display = "none";

            window.location.href = 'yjSchemeLink://'; //苹果app跳转的协议
            // window.location.href = jumpUrl; //下载地址
            return;
          }

          // alert('直接唤醒app，若唤醒失败，1秒后跳转appstore下载页面');
          setTimeout(function () {
            window.location.href = jumpUrl; //下载地址
          }, 1000);
        }

        const { search } = window.location;

        if (search.includes('auto=1')) {
          if (MOBILE_Android.test(USER_Agent)) {
            goAndroid();
          }
          if (MOBILE_IOS.test(USER_Agent)) {
            goIPone();
          }
        }
      });

```
