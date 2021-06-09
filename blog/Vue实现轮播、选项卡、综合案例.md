> 最近在学习vue基础，写了几个实例，记录一下。
[toc]
#### 轮播
###### 代码：
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <script src="./lib/vue.js"></script>
    <style>
        #app {
            text-align: center;
            padding: 50px;
        }
        
        .banner {
            width: 495px;
            height: 212px;
            line-height: 212px;
            box-shadow: 5px 5px 10px #888888;
            background-color: #033592;
            margin: 0px auto;
            border-radius: 20px;
            background-size: 100%;
        }
    </style>
</head>

<body>
    <div id="app">
        <div :style="imageUrl" :class="bannerClass"></div>

    </div>

    <script>
        //创建Vue实例，得到ViewModel
        var vm = new Vue({
            el: '#app',
            data: {
                index: 1,
                imageUrl: {
                    "background-image": "url(./images/blogcover1.jpg)"
                },
                bannerClass: {
                    banner: true
                }
            },
            mounted: function() {
                var _this = this;
                this.timer = setInterval(function() {
                    _this.imageUrl = {
                        backgroundImage: "url(./images/blogcover" + (_this.index % 3 + 1) + ".jpg)",
                        transition: "all 1.5s 0s"
                    };
                    _this.index++;
                }, 3000);
            },
            beforeDestroy: function() {
                if (this.timer) {
                    clearInterval(this.timer);
                }

            },
        });
    </script>
</body>

</html>
```
###### 预览：
![Vue实现轮播](https://cdn.jsdelivr.net/gh/yihansix6/cloudimage/blog-imgblog4-1.gif "Vue实现轮播")
#### 选项卡
###### 代码：
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <style>
        * {
            margin: 0;
            padding: 0;
        }
        
        #app {
            width: 400px;
            padding: 50px;
        }
        
        ul {
            list-style: none;
            overflow: hidden;
        }
        
        ul li {
            float: left;
            width: 100px;
            text-align: center;
            color: #ffffff;
            border-radius: 5px 5px 0 0;
        }
        
        .box {
            box-shadow: 0 15px 10px #888888;
            height: 200px;
            color: #ffffff;
            border-radius: 0 10px 10px 10px;
            padding: 10px;
        }
    </style>
    <script src="./lib/vue.js"></script>
</head>

<body>
    <div id="app">
        <ul>
            <li :style="nav1Style" @click="doChange(1)">工作技能<li>
            <li :style="nav2Style" @click="doChange(2)">工作积极性<li>
        </ul>
        <div id="panel1" class="box" :style="pnl1Style">工作技能是指完成工作所需具备的知识，技能和经验等，是岗位胜任者和卓越绩效者所需的实际操作技能</div>
        <div id="panel2" class="box" :style="pnl2Style">阿巴阿巴阿巴阿巴阿巴阿巴阿巴阿巴阿巴阿巴阿巴阿巴阿巴阿巴阿巴阿巴阿巴阿巴</div>
    </div>

    <script>
        //创建Vue实例，得到ViewModel
        var vm = new Vue({
            el: '#app',
            data: {
                nav1Style: {
                    background: "#f16565"
                },
                nav2Style: {
                    background: "#5757fc"
                },
                pnl1Style: {
                    display: "block",
                    backgroundColor: "#f16565"
                },
                pnl2Style: {
                    display: "none",
                    backgroundColor: "#5757fc"
                }
            },
            methods: {
                doChange: function(index) {
                    if (index == 1) {
                        this.pnl1Style = {
                            display: "block",
                            backgroundColor: "#f16565"
                        };
                        this.pnl2Style = {
                            display: "none"
                        };
                    } else {
                        this.pnl1Style = {
                            display: "none"
                        };
                        this.pnl2Style = {
                            display: "block",
                            backgroundColor: "#5757fc"
                        };
                    }
                }
            }
        });
    </script>
</body>

</html>
```
###### 预览：
![Vue实现选项卡](https://cdn.jsdelivr.net/gh/yihansix6/cloudimage/blog-imgblog4-2.gif "Vue实现选项卡")
#### 综合案例
###### 代码：
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <style>
        #app{
            height: 300px;
            width: 300px;
            position: absolute;
            left: 35%;
        }
    </style>
    <script src="./lib/vue.js"></script>
</head>

<body>
    <div id="app">
        <fieldset>
            <legend>新商品信息</legend>
            <table>
                <tr>
                    <td>名称：</td>
                    <td><input type="text" v-model="productObj.name"></td>
                </tr>
                <tr>
                    <td>价格：</td>
                    <td><input type="number" v-model="productObj.price"></td>
                </tr>
                <tr>
                    <td>类别：</td>
                    <td>
                        <select v-model="productObj.categroy">
                            <option v-for="cate in categories" :value="cate">{{cate}}</option>
                        </select>
                    </td>
                </tr>
                <tr>
                    <td></td>
                    <td><input type="button" value="添加" @click="add"></td>
                </tr>
            </table>
        </fieldset>
        <div>
            查询关键字：
            <input type="text" v-model="wordkey">
        </div>
        <table>
            <tr>
                <th>名称</th><th>价格</th><th>类别</th><th>删除</th>
            </tr>
            <tr v-for="(product,index) in tempList">
                <td>{{product.name}}</td>
                <td>{{product.price}}</td>
                <td>{{product.categroy}}</td>
                <td><input type="button" value="删除" @click="del(index)"></td>
            </tr>
        </table>
    </div>

    <script>
        var vm=new Vue({
           el:'#app',
           data:{
               categories:["手机/数码","电脑/办公","家居/家具","男装/女装","房产/汽车"],
               //商品对象数组
               productlist:[
                   {
                       name:"XPhone 10",
                       price:4999,
                       categroy:"手机/数码"
                   },{
                       name:"T恤",
                       price:288,
                       categroy:"男装/女装"
                   }
               ],
               productObj:{
                    name:"",
                    price:0,
                    categroy:"手机/数码"
               },
               wordkey:""
           },
           methods: {
               add:function(){
                   this.productlist.push(this.productObj);
                   this.productObj = {
                        name:"",
                        price:0,
                        categroy:"手机/数码"
                   };
               },
               del:function(key){
                    if(confirm("确认删除吗？")){
                        this.productlist.splice(key,1);
                    }
               }
           },
           computed: {
               tempList:function(){
                   var _this = this;
                   return _this.productlist.filter(item => {
                       if(item.name.includes(_this.wordkey)){
                        return item;
                   }
                   })
               }
           },
        });
    </script>
</body>

</html>
```
###### 预览：
![Vue综合案例](https://cdn.jsdelivr.net/gh/yihansix6/cloudimage/blog-imgblog4-3.gif "Vue综合案例")