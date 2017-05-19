Qinniu 远程文件上传

1.服务器端

```
引入官方sdk
require './Application/Admin/Common/phpsdk/sdk/autoload.php';
use Qiniu\Auth;
use Vendor\Response;
写接口去请求远程uptoken;
class QiniuController extends Controller {
    public function  GetToken(){
        // 用于签名的公钥和私钥
        $accessKey = C('ACCESS_KEY');
        $secretKey = C('SECRET_KEY');
        $bucket = C("BUCKET");
        // 初始化签权对象

        $auth = new Auth($accessKey, $secretKey);

        $upToken = $auth->uploadToken($bucket);
//        header('Access-Control-Allow-Origin:*');
        $arr=json_encode(array('uptoken' => $upToken));
        echo $arr;
    }
}
```

2.web端

引入js

```
<script type="text/javascript" src="{:C(NODE_MODULE)}plupload/js/moxie.js"></script>
<script type="text/javascript" src="{:C(NODE_MODULE)}plupload/js/plupload.min.js"></script>
<script type="text/javascript" src="{:C(NODE_MODULE)}qiniu/src/qiniu.js"></script>
```

```
 var uploader =new Qiniu.uploader({
     runtimes: 'html5,flash,html4',      // 上传模式，依次退化
     browse_button:'pickfiles',         // 上传选择的点选按钮，必需
     // 在初始化时，uptoken，uptoken_url，uptoken_func三个参数中必须有一个被设置
     // 切如果提供了多个，其优先级为uptoken > uptoken_url > uptoken_func
     // 其中uptoken是直接提供上传凭证，uptoken_url是提供了获取上传凭证的地址，如果需要定制获取uptoken的过程则可以设置uptoken_func
      // uptoken是上传凭证，由其他程序生成
      // uptoken:"{$up_token}",
      uptoken_url:"/index.php/Api/Qiniu/GetToken", //请求远程接口
     filters: {
       mime_types : [ //只允许上传图片和zip文件
         { title : "Image files", extensions : "jpg,gif,png,jpeg" },
       ],
     },
     // uptoken_func: function(file){    // 在需要获取uptoken时，该方法会被调用
     //    // do something
     //    return uptoken;
     // },
     get_new_uptoken: false,             // 设置上传文件的时候是否每次都重新获取新的uptoken
     // downtoken_url: '/downtoken',
     // Ajax请求downToken的Url，私有空间时使用，JS-SDK将向该地址POST文件的key和domain，服务端返回的JSON必须包含url字段，url值为该文件的下载地址
//     unique_names: true,              // 默认false，key为文件名。若开启该选项，JS-SDK会为每个文件自动生成key（文件名）
//     save_key: true,                  // 默认false。若在服务端生成uptoken的上传策略中指定了sava_key，则开启，SDK在前端将不对key进行任何处理
     domain: 'http://oq0x8b5jt.bkt.clouddn.com',     // bucket域名，下载资源时用到，必需
     max_file_size: '100mb',             // 最大文件体积限制
     flash_swf_url: '/Public/admin/node_modules/plupload/js/Moxie.swf',  //引入flash，相对路径
     max_retries: 3,                     // 上传失败最大重试次数
     dragdrop: true,                     // 开启可拖曳上传
     chunk_size: '4mb',                  // 分块上传时，每块的体积
     auto_start: true,                   // 选择文件后自动上传，若关闭需要自己绑定事件触发上传
     //x_vars : {
     //    查看自定义变量
     //    'time' : function(up,file) {
     //        var time = (new Date()).getTime();
     // do something with 'time'
     //        return time;
     //    },
     //    'size' : function(up,file) {
     //        var size = file.size;
     // do something with 'size'
     //        return size;
     //    }
     //},
     init: {
       'BeforeUpload': function(up, file) {

         var arr=file.type.split("/");
         var ext=arr[1];
         var allow=["jpg","jpeg","png","gif"];
         if($.inArray(ext,allow)==-1){
           layer.alert("请上传正确的图片格式");
           return false;
         }
       },
       'UploadProgress': function(up, file) {
         // 每个文件上传时，处理相关的事情
         var percent = file.percent;
          $(".progress").show();
          $(".progress-bar").attr("aria-valuenow",percent).css("width",percent+"%").text(percent+"%");
       },
       'FileUploaded': function(up, file, info) {
         // 每个文件上传成功后，处理相关的事情
         // 其中info是文件上传成功后，服务端返回的json，形式如：
         // {
         //    "hash": "Fh8xVqod2MQ1mocfI4S4KpRL6D98",
         //    "key": "gogopher.jpg"
         //  }
         // 查看简单反馈
         $(".progress").hide();
         var domain = up.getOption('domain');
         var res = $.parseJSON(info);
         var sourceLink = domain +"/"+ res.key; //获取上传成功后的文件的Url
         $(".file_img img").attr("src",sourceLink);
         $(".img_val").val(res.key);
         console.log(sourceLink);
       },
       'Error': function(up, err, errTip) {
         //上传出错时，处理相关的事情
         layer.alert(errTip);
       },
       'UploadComplete': function() {
         //队列文件处理完毕后，处理相关的事情
       },
       'Key': function(up, file) {
         console.log(file);
         var ext=file.type.split("/");
         // 若想在前端对每个文件的key进行个性化处理，可以配置该函数
         // 该配置必须要在unique_names: false，save_key: false时才生效
         var time=new Date().getTime()+"";
         var key = "pro_"+time.substring(0,6)+randomString(3)+"."+ext[1];//生成图片唯一标识码
         // do something with key here
         return key
       }
     }
   });
   //添加产品
    $(".addGoods").click(function (ev) {
        var This=$("this");
        ev.preventDefault();
        var goods_name=$(".pro_name").val();
        var index="";
        var image=$(".img_val").val();
        if(goods_name==""||goods_name.length<3){
          layer.alert("产品名称长度不能小于3位");
          return false;
        }
        if(image==""){
          layer.alert("请上传产品图片");
        }
        var datas=$(".goods_form").serialize();
        $.ajax({
          url:"{:U(adds)}",
          data:datas,
          dataType:"json",
          type:"post",
          success:function (msg) {
            if(msg.code==0){
              layer.close(index);
              layer.alert(msg.message);
              return false;
            }else if(msg.code==1){
                layer.close(index);
                window.location.reload();
            }
          },
          beforeSend: function(){
            index=layer.load(1);
          },
          error:function(){
            layer.close(index);
            layer.alert("服务器繁忙请稍后再试");
          }
        })

    })
  })
  function randomString(len) {
    len = len || 32;
    var $chars = 'ABCDEFGHJKMNPQRSTWXYZabcdefhijkmnprstwxyz2345678';    /****默认去掉了容易混淆的字符oOLl,9gq,Vv,Uu,I1****/
    var maxPos = $chars.length;
    var pwd = '';
    for (i = 0; i < len; i++) {
      pwd += $chars.charAt(Math.floor(Math.random() * maxPos));
    }
    return pwd;
  }
```

