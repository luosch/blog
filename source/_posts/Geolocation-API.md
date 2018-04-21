title: Geolocation API
date: 2014-12-12 01:45:38
tags: [技术]
---
HTML Geolocation API 可以用来获得用户的地理位置

支持 Geolocation API 的浏览器/设备：
* IE9.0+
* Firefox
* Safari
* Chrome
* Opera
* iPhone 3+、Android 2.0+

也就是说除IE6~IE8外，其它最新的浏览器基本上都支持，包括最新的移动手机

Geolocation API 存在于 navigator对象中，只包含3个方法：

1. getCurrentPosition
2. watchPosition
3. clearWatch

##getCurrentPosition()
navigator.geolocation.getCurrentPosition(success_callback, error_callback, options);

参数说明：
* 第一个参数是用户允许浏览器共享 geolocation 成功后的回调方法
* 第二个参数是用获取地理位置信息失败的处理方法，传入错误对象，包含 code、message 两个属性
* 第三个参数是 geolocation 选项，所有的 geolocation 选项都是可选的，它包含的属性如下：
   * enableHighAccuracy(Boolean型，默认为false，是否尝试更精确地读取纬度和经度，移动设备上，这可能要使用手机上的GPS，这会消耗移动设备更多的电量)
   * timeout(单位为毫秒，默认值为0，在放弃并触发处理程序之前，可以等待的时间----用户选择期间是不计时的)
   * maximumAge(单位为毫秒，默认值为0。用来告诉浏览器是否使用最近缓存的位置数据，如果在maximumAge内有一个请求，将会返回它，而不请求新位置。 maximumAge如果为Infinity，则总是使用一个缓存的位置，如果为0则必须在每次请求时查找一个新位置)

``` javascript
window.onload = function() {
    // html5 geolocation
    var options = {  
        enableHighAccuracy: true,  
        maximunAge: 1000,  
        timeout: 45000  
    };
    if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(function(position) {
            baidu_map(position.coords.longitude, position.coords.latitude);
            confirm('您的位置:\n'+
                "经度 ==> "+position.coords.longitude+"\n"+
                "维度 ==> "+position.coords.latitude+"\n"+
                "经纬度精确到 "+ position.coords.accuracy +" 米");
            if (position.coords.altitude != null){
                confirm("海拔高度 ==> "+ position.coords.altitude);
            }
        }, function (error) {
            switch(error.code) {
                case error.PERMISSION_DENIED:
                    alert("User denied the request for Geolocation.")
                    break;
                case error.POSITION_UNAVAILABLE:
                    alert("Location information is unavailable.")
                    break;
                case error.TIMEOUT:
                    alert("The request to get user location timed out.")
                    break;
                case error.UNKNOWN_ERROR:
                    alert("An unknown error occurred.")
                    break;
            }
        }, options);
    } else {
        alert("Geolocation is not supported by this browser.")
    }
};
```
getCurrentPosition() 返回的数据如下:


| Property         | Description                       |
| ---------------- |:---------------------------------:|
| coords.latitude  | The latitude as a decimal number  |
| coords.longitude | The longitude as a decimal number |
| coords.accuracy  | The accuracy of position          |
| coords.altitude  | The altitude in meters above the mean sea level  |
| coords.altitudeAccuracy | The altitude accuracy of position |
| coords.heading  | The heading as degrees clockwise from North          |
| coords.speed  | The speed in meters per second  |
| timestamp | The date/time of the response |


##baidu_map()
调用百度地图API, 并把它包装成一个函数

``` javascript
function baidu_map(longitude, latitude) {
    // 百度地图API功能
    var map = new BMap.Map("allmap");
    map.centerAndZoom(new BMap.Point(longitude, latitude), 15);

    // 左上角，添加比例尺
    var top_left_control = new BMap.ScaleControl({anchor: BMAP_ANCHOR_TOP_LEFT});
    // 左上角，添加默认缩放平移控件
    var top_left_navigation = new BMap.NavigationControl();
    // 右上角，仅包含平移和缩放按钮
    var top_right_navigation = new BMap.NavigationControl({anchor: BMAP_ANCHOR_TOP_RIGHT, type: BMAP_NAVIGATION_CONTROL_SMALL});
    /*缩放控件type有四种类型:
    BMAP_NAVIGATION_CONTROL_SMALL：仅包含平移和缩放按钮；BMAP_NAVIGATION_CONTROL_PAN:仅包含平移按钮；BMAP_NAVIGATION_CONTROL_ZOOM：仅包含缩放按钮*/
    map.enableScrollWheelZoom();   // 启用滚轮放大缩小，默认禁用
    map.enableContinuousZoom();    // 启用地图惯性拖拽，默认禁用

    //添加控件和比例尺
    map.addControl(top_left_control);
    map.addControl(top_left_navigation);
    map.addControl(top_right_navigation);
    var geolocation = new BMap.Geolocation();

    // 标记当前位置并提示
    geolocation.getCurrentPosition(function(r) {
        if (this.getStatus() == BMAP_STATUS_SUCCESS){
            var mk = new BMap.Marker(r.point);
            map.addOverlay(mk);
            map.panTo(r.point);
        } else {
            alert('failed'+this.getStatus());
        }
    }, {enableHighAccuracy: true})
}
```
##其他方法
watchPosition() - 当用户移动时实时返回地理位置

clearWatch() - 结束 watchPosition()

watchPosition() 调用方法和 getCurrentPosition() 类似

我的 demo 放在 [github](http://luosch.github.io/get_you "在哪") 上
