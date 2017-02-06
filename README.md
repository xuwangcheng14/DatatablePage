# DatatablePage
页面预览

 

预览-用户列表

预览-增加用户

预览-编辑用户

 

 

项目结构

项目结构

 

说明：说一下resource文件夹  template  里面为handlebars的模板文件    其他文件夹按自己的模块来建立  例如本例中user模块中有user此实体,对应一个html和同名的js文件

 

项目说明

 

user.html

本例中前端框架用的是H-ui（http://www.h-ui.net/Hui-overview.shtml），大家可以根据自己的项目使用不同的页面模板

除了页面标题和导航栏的内容需要改动下，其他的可以固定不变 

<div id="list-page"></div> 中为表格内容
<div id="edit-page" style="display: none;"></div> 中为js中会加载的编辑和增加的页面
<div id="template-page" style="display: none;"></div> 作用是为了加载template中的js模板

 

expand source
pageTemplate.htm



table-thead-template为表格thead中的生成模板

数据格式： ["姓名","学号","年龄"] btn-text-template为通用按钮的生成模板,可以生成多个并列按钮

数据格式：

[{
   type:"success",  //按钮样式,可选-success info warn danger等
   size:"S",    //按钮大小 可选 XL L M S MINI  依次减小
   markClass:"",  //标记用的className 为了绑定事件 可选
   iconFont:"",  //图标
   name:""    
},{}] 
 
btn-icon-template为图标形状的按钮,用法同btn-text-template

数据格式：
[{
   title:"",
   markClass:"",
   iconFont:""    
},{}]


form-control-template

 
form-control-template为生成编辑页面或者添加页面的表单模板

数据格式：

?
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
[{
   edit:flase,   //是否仅仅展示在编辑页面  模板为false 如果为true  该表单控件会加上一个 editFlag的className 并且只在编辑页面显示
   required:false,   //该控件是否为必选 ,默认为false  如果为true的话  会在该控件的label前加上一个红色*来凸出标记
   label:"",        //控件的label
   objText:"",   //默认为空  如果有值 就在该控件之前创建一个span的文本(id为该值,一般命名为该控件的name加上“Text”,如 nameText)来显示控件的value值   //需要哪种类型的控件就添加该类型控件的数据 可以有多个不同或者相同类型的控件
   input:[{   
      hidden:false,     //是否为隐藏域
      value:"",
      name:"",
      placeholder:"" 
      }],
   textarea:[{
      placeholder:"",
      value:"",
      name:""
      }],
   select:[{
      name:"",
      option:[{         //option的数据  可以有多个  
         value:"",
         text:"",
         selected:false   //是否被选中  默认false
         }]
      }],
   button:[{
      name:"",
      style:"",    
      markClass:"",
      value:""      
      }] 
}]
expand source

dcits.js


相关的模板渲染、页面初始化以及一些通用方法都在此JS中，可以自行查看，重点说下初始化用的publish数据

init：该方法为初始化的入口  其中模板的渲染只有一次(包括handlebars生成和list页面渲染),之后只有打开添加页面和编辑页面的时候才会再次调用到
renderParams：包含所有的渲染需要的参数,详细如下：

/**
   * 渲染所需参数
   * 不可选：
   * ifFirst: true 当前是否为本页面第一次初始化  防止事件被重复绑定 默认为true
   * 可选:
   * customCallBack:数据渲染 不是list  edit页面的自定义数据渲染方法  
   * templateCallBack:默认的模板渲染之后 自定义的二次渲染
   * eventList:页面event绑定 默认{}
   * renderType: 当前渲染模式 listPage还是editPage 默认为list 可选edit
   * userDefaultRender:是否使用默认的渲染  默认为true  可选 false为不使用,仅使用使用提供customCallBack回调方法中的渲染
   * 
   * 
   * 
   * list页面
   * 必须：
   * listUrl: Datatables请求数据地址 必要
   * tableObj:对应的DT容器 必要
   * columnsSetting:列的设置
   * 
   * 可选:
   * beforeInit:执行init方法之前需要执行的方法  请在该方法最后调用df.resolve();   
   * columnsJson:不参与排序的列
   * 
   * edit页面
   * 必须：
   * editUrl:编辑请求地址 必要
   * 
   * 可选：
   * beforeInit:执行init方法之前需要执行的参数  请在该方法最调用df.resolve();
   * formObj:对应的form容器 默认为 ".form-horizontal" 可选
   * modeFlag:指定时添加操作还是编辑操作  默认为0-add  可选 1-edit
   * objId: get实体时需要的id
   * getUrl:获取实体对象时的请求地址

   * rules:rules规则 定义验证规则
   * messages 提示 定义验证时展示的提示信息

   * closeFlag:成功提交并返回之后是否关闭当前编辑窗口  默认为true  可选false
   * ajaxCallbackFun: ajax提交中的回调函数  如传入null,则使用默认

   * renderCallback:function(obj){}  如果默认的渲染结果不是完整或者正确的,可以传入该回调重新或者附加渲染  obj 实体对象
   * 
   * 
   * templateParams参数：
   *参考template.htm
   * tableTheads:必须的  传入字符串数组,渲染成指定的表头  
   * btnTools:表格上方的工具栏 带文字和图标样式按钮
   * formControls: edit页面的表单控件渲染,参见下面的数据格式  需要某种类型的控件 必须需要将对应子节点下的flag设置为true
   * 
   */
 
renderTemplate：模板渲染的方法,回调方法中可以进行二次渲染
renderData：数据渲染的方法,回调方法中可以进行二次渲染
initListeners：使用代理的方法绑定事件
 
?
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67
68
69
70
71
72
73
74
75
76
77
78
79
80
81
82
83
84
85
86
87
88
89
90
91
92
93
94
95
96
97
98
99
100
101
102
103
104
105
106
107
108
109
110
111
112
113
114
115
116
117
118
119
120
121
122
123
124
125
126
127
128
129
130
131
132
133
134
135
136
137
138
139
140
141
142
143
144
145
146
147
148
149
150
151
152
153
154
155
156
157
158
159
160
161
162
163
164
165
166
167
168
169
170
171
172
173
174
175
176
177
178
179
180
181
182
183
184
185
186
187
188
189
190
191
192
193
194
195
196
197
198
199
200
201
202
203
204
205
206
207
208
209
210
211
212
213
214
215
216
217
218
219
220
221
222
223
224
225
226
227
228
229
230
231
232
233
234
235
236
237
238
239
240
241
242
243
244
245
246
247
248
249
250
251
252
253
254
255
256
257
258
259
260
261
262
263
264
265
266
267
268
269
270
271
272
273
274
275
276
277
278
279
280
281
282
283
284
285
286
287
288
289
290
291
292
293
294
295
296
297
298
299
300
301
302
303
304
305
306
307
308
309
310
311
312
313
314
315
316
317
318
319
320
321
322
323
324
325
326
327
328
329
330
331
332
333
334
335
336
337
338
339
340
341
342
343
344
345
346
347
348
349
350
351
352
353
354
355
356
357
358
359
360
361
362
363
364
365
366
367
368
369
370
371
372
373
374
375
376
377
378
379
380
381
382
383
384
385
386
387
388
389
390
391
392
393
394
395
396
397
398
399
400
401
402
403
404
405
406
407
408
409
410
411
412
413
414
415
416
417
418
419
420
421
422
423
424
425
426
427
428
429
430
431
432
433
434
435
436
437
/**
 * handlebars模板 
 */
var tableTheadTemplate;
var btnTextTemplate;
var btnIconTemplate;
var formControlTemplate;
 
var editHtml="";
var table;
 
$(function(){
    //加载对应的js文件 
    var new_element=document.createElement("script");
    new_element.setAttribute("type","text/javascript");
    var r = (window.location.pathname.split("."))[0].split("/");
    r = r[r.length-1]+".js";
    new_element.setAttribute("src",r);
    document.body.appendChild(new_element);
 
});
 
//设置jQuery Ajax全局的参数  
$.ajaxSetup({  
    error: function(jqXHR, textStatus, errorThrown){  
        layer.closeAll('dialog');
        switch (jqXHR.status){  
            case(500):  
                layer.alert("服务器系统内部错误",{icon:5});  
                break;  
            case(401):  
                layer.alert("未登录或者身份认证过期",{icon:5});  
                break;  
            case(403):  
                layer.alert("你的权限不够",{icon:5});  
                break;  
            case(408):  
                layer.alert("AJAX请求超时",{icon:5});
                break;  
            default:  
                layer.alert("AJAX调用失败",{icon:5});
        }        
    }
});
 
//DT的配置常量
var CONSTANT = {
        DATA_TABLES : { 
            DEFAULT_OPTION:{
            "aaSorting": [[ 1, "desc" ]],//默认第几个排序
            "bStateSave": true,//状态保存
            "processing": false,   //显示处理状态
            "serverSide": true,  //服务器处理
            "autoWidth": false,   //自动宽度
            "responsive": false,   //自动响应
            "language": {
                "url": "../../js/zh_CN.txt"
            },
            "lengthMenu": [[10, 15, 100], ['10', '15', '100']],  //显示数量设置
            //行回调
            "createdRow": function ( row, data, index ){
                $(row).addClass('text-c');
            }},
            //常用的COLUMN
            COLUMNFUN:{
                //过长内容省略号替代
                ELLIPSIS:function (data, type, row, meta) {
                    data = data||"";
                    return '<span title="' + data + '">' + data + '</span>';
                }
            }
        }
};
 
 
/**
 * 初始化方式
 * 通过以下方式来设置相关参数 
 * publish.renderParams = $.extend(true,publish.renderParams,mySetting);
 * 通过publish.init();来进行初始化
 */
var publish = {
     //模块初始化入口
     init : function(){
         var that = this;
        //模板渲染只有一次
         var df1 = $.Deferred();
         df1.done(function(){
             var df = $.Deferred();
             df.done(function(){
                 that.renderData(that.renderParams.customCallBack);
                 //防止事件被绑定多次
                 if(that.renderParams.ifFirst==true){
                     that.initListeners(that.renderParams.eventList); 
                 }
             }); 
             if(that.renderParams.renderType=="list"){
                 that.renderParams.listPage.beforeInit(df);
             }else if(that.renderParams.renderType=="edit"){
                 that.renderParams.editPage.beforeInit(df);
             }                 
         });
         if(that.renderParams.ifFirst==true){
             that.renderTemplate(df1,that.renderParams.templateCallBack); 
         }else{
             df1.resolve();
         }  
     },
   
     renderParams:{
         renderType:"list",
         userDefaultRender:true,         
         customCallBack:function(p){},
         templateCallBack:function(df){
             df.resolve();
         },
         eventList:{},
         ifFirst: true,
         listPage:{
             listUrl:"",
             beforeInit:function(df){
                 df.resolve();
             },
             tableObj:null,
             columnsSetting:{},
             columnsJson:[],
             dt:null
         },
         editPage:{
             beforeInit:function(df){
                 df.resolve();
             },
             modeFlag:0,
             objId:null,
             editUrl:"",
             formObj:".form-horizontal",
             getUrl:"",
             rules:{},
             messages:{},
             closeFlag:true,
             ajaxCallbackFun:null,
             renderCallback:function(obj){} 
         },  
         templateParams:{
             tableTheads:[],
             btnTools:[],
             formControls:[]
         }
     },
     /**
      * 在进行数据渲染之前先进行静态的模板渲染
      * @param callback
      */
     renderTemplate:function(df,callback){
         var t = this.renderParams.templateParams;
         var html = "";
        //预编译handlebars模板
        $("#template-page").load("../template/pageTemplate.htm",function(){
            //list表格头
            tableTheadTemplate = Handlebars.compile($("#table-thead-template").html());
            btnTextTemplate = Handlebars.compile($("#btn-text-template").html());
            btnIconTemplate = Handlebars.compile($("#btn-icon-template").html());
            formControlTemplate = Handlebars.compile($("#form-control-template").html());
            //渲染表头
             $("#table-thead").append(tableTheadTemplate(t.tableTheads));
             //渲染表格上方工具栏
             $("#btn-tools").append(btnTextTemplate(t.btnTools));
              
             //渲染editPage的表单控件
             editHtml = formControlTemplate(t.formControls);
             //传入回调进行二次的渲染如果对editPage做过自定义的渲染  那么必须重新定义全局变量 editHtml            
             callback(df);
        });      
         
     },
     /**
      * 内部所用的函数-渲染数据 不同的页面的渲染模式  通用为list(列表页) edit(编辑增加页) 其他类型自己扩展
      * @param callback  
      */
     renderData : function(callback){ 
         var p = this.renderParams;
          
         //默认渲染
         if(p.userDefaultRender==true){
             if(p.renderType=="list"){
                 var l = p.listPage;
                 table = initDT(l.tableObj,l.listUrl,l.columnsSetting,l.columnsJson);
             }           
             if(p.renderType=="edit"){
                 var e = p.editPage;
                 if(e.modeFlag==1){
                     ObjectEditPage(e.objId,e.getUrl,e.renderCallback);
                 }
                 formValidate(e.formObj,e.rules,e.messages,e.editUrl,e.closeFlag,e.ajaxCallbackFun);
             }
         }   
         callback(p);        
     },
     /**
      * 统一绑定监听器
      * @param eventList  事件列表 jsonObject
      * 没有传入{}
      * 格式: 
      * '.btn' : function(){}  点击事件
      * '.checkbox' : {'change' : function(){}}  其他事件  请参照jquery的event
      */
     initListeners : function(eventList){
         $(document.body).delegates(eventList);
         this.renderParams.ifFirst = false;
         this.renderParams.renderType = "edit";
     }
};
 
 
 
/**
 * 自定义扩展的jquery方法
 * 已配置的方式代理事件
 */
$.fn.delegates = function(configs) {
     el = $(this[0]);
     for (var name in configs) {
          var value = configs[name];
          if (typeof value == 'function') {
               var obj = {};
               obj.click = value;
               value = obj;
          };
          for (var type in value) {
               el.delegate(name, type, value[type]);
          }
     }
     return this;
};
 
 
/**
 * 
 * @param tableObj 对应表格dom的jquery对象
 * @param ajaxUrl  ajax请求数据的地址  string
 * @param columnsSetting columns设置  jsonArray
 * @param columnsJson  不参与排序的列 jsonArray
 * @returns table 返回对应的DataTable实例
 */
function initDT(tableObj,ajaxUrl,columnsSetting,columnsJson){
    //渲染前使用自定义遮罩层
    $wrapper.spinModal();
    var table = $(tableObj)
    //发送ajax之前
    .on('xhr.dt', function ( e, settings, json, xhr ) {
        if(json.returnCode!=0){
            layer.alert(json.msg,{icon:5});
        }
    })
    //初始化完毕
    .on( 'init.dt', function () {
        $wrapper.spinModal(false);
    } )
    .DataTable($.extend(true,{},CONSTANT.DATA_TABLES.DEFAULT_OPTION,{
        "ajax":ajaxUrl,
        "columns":columnsSetting,                                           
         "columnDefs": [{"orderable":false,"aTargets":columnsJson}]
          }));
     
    return table;
}
 
 
/**
 * 返回DT中checkbox的html
 * @param name  name属性,对象的name或者其他
 * @param val   value属性,对象的id
 * @param className  class属性,一般为select+对象
 */
function checkboxHmtl(name,val,className){
    return '<input type="checkbox" name="'+name+'" value="'+val+'" class="'+className+'">';
}
 
 
/**
 * 单个删除功能，通用
 * tip 确认提示
 * url 删除请求地址
 * id 删除的实体ID
 * obj 表格删除对应的内容
 */
function delObj(tip,url,id,obj){
    layer.confirm(tip,function(index){
        $wrapper.spinModal();
        $.post(url,{id:id},function(data){
            if(data.returnCode==0){
                $wrapper.spinModal(false);
                table.row($(obj).parents('tr')).remove().draw();
                layer.msg('已删除',{icon:1,time:1500});
            }else{
                $wrapper.spinModal(false);
                layer.alert(data.msg, {icon: 5});
            }
        });
    });
}
 
/**
 * 批量删除方法-表格为DT时
 * @param checkboxList  checkBox被选中的列表
 * @param url   删除url
 * @param tableObj   TD对象，默认名为table
 * @returns {Boolean}
 */
function batchDelObjs(checkboxList,url,tableObj){
    if(checkboxList.length<1){
        return false;
    }
    layer.confirm('确认删除选中的'+checkboxList.length+'条记录?',function(index){
        layer.close(index);
        $wrapper.spinModal();
        var delCount = 0;
        var errorTip = "";
            $.each(checkboxList,function(i,n){
            objId=$(n).val();//获取id
            objName=$(n).attr("name");  //name属性为对象的名称  
            layer.msg("正在删除"+objName+"...",{time:999999});    
                $.ajax({
                    type:"POST",
                    url:url,
                    data:{id:objId},
                    async:false,
                    success:function(data){
                        if(data.returnCode!=0){ 
                            layer.msg("删除"+objName+"失败!",{time:999999});
                            errorTip += "["+objName+"]";
                        }else{
                            delCount = i+1;
                            layer.msg("删除"+objName+"成功!",{time:999999});
                        }
                    }
                    });         
            });
            layer.closeAll('dialog');
            refreshTable();
            $wrapper.spinModal(false);
            if(errorTip!=""){
                errorTip="在删除"+errorTip+"数据时发生了错误,请查看错误日志!";
                layer.alert(errorTip,{icon:5},function(index){
                    layer.close(index);
                    layer.msg("共删除"+delCount+"条数据!",{icon:1,time:2000});
                });
            }else{
                layer.msg("共删除"+delCount+"条数据!",{icon:1,time:2000});
            }
 
    });
         
}
 
/**
 * 编辑页面 当modeflag为1(编辑)时 默认的页面渲染
 * @param id  指定实体的id
 * @param ajaxUrl  ajax地址
 * @param callback(function(obj){}) 如果默认的渲染结果不是完整或者正确的,可以传入该回调重新渲染  obj 实体对象
 */
function ObjectEditPage(id,ajaxUrl,callback){
    $(".form-horizontal").spinModal();
    //编辑模式时将某些隐藏的控件展示
    $(".editFlag").css("display","block");
    $.post(ajaxUrl,{id:id},function(data){
        if(data.returnCode==0){
            var o=data.object;
            //默认的渲染  object中同名id 控件 以及id名为 idText
            $.each(o,function(k,v){
                if(!(v instanceof Object)){
                    if($("#"+k)){
                        $("#"+k).val(v);
                    }
                    if($("#"+k+"Text")){
                        $("#"+k+"Text").text(v);
                    }
                }
            });         
            //该回调可以自行渲染默认没有渲染到的控件
            callback(o);    
            $(".form-horizontal").spinModal(false);
        }else{
            layer.alert(data.msg,{icon:5});
        }
    });
     
     
}
 
 
/**
 * 所有实体edit页面的jqueryValidate方法
 * @param formObj  表单obj
 * @param rules   rules规则 定义验证规则
 * @param messages 提示 定义验证时展示的提示信息
 * @param ajaxUrl  验证成功 ajax提交地址
 * @param closeFlag  成功提交并返回之后是否关闭当前窗口  true  false
 * @param ajaxCallbackFun  ajax提交中的回调函数  如传入null,则使用默认
 */
function formValidate(formObj,rules,messages,ajaxUrl,closeFlag,ajaxCallbackFun){
    var callbackFun = function(data){
        if(data.returnCode==0){ 
            refreshTable();
            if(closeFlag){
                //关闭当前的所有页面层
                layer.closeAll('page');
            }                   
        }else{
            layer.alert(data.msg, {icon: 5});
        }           
    };
    if(ajaxCallbackFun!=null){
        callbackFun=ajaxCallbackFun;
    }
    $(formObj).validate({
        rules:rules,
        messages:messages,
        ignore: "",
        onkeyup:false,
        focusCleanup:true,
        success:"valid",
        submitHandler:function(form){
            var formData = $(form).serialize();
            $.post(ajaxUrl,formData,callbackFun);           
        }
    });
}
 
/**
 * 刷新表格
 * 所有表格页面都是的DT对象名称都要命名为table
 */
function refreshTable(){
    table.ajax.reload(null,false);
 
}

user.js



相关的模板渲染数据都在此文件中，初始化只需要在ready方法中执行下面的代码就行了：
publish.renderParams = $.extend(true,publish.renderParams,mySetting);
publish.init();

配置数据主要分为下面的几部分：

columnsSetting 


?
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
Datatables的相关列设定 一些渲染可以在这里完成var columnsSetting = [{"data":null,
     "render":function(data, type, full, meta){
                return checkboxHmtl(data.username+'-'+data.realName,data.userId,"selectUser");
               }},
    {"data":"userId"},
    {"data":"username"},
    {"data":"realName"}, 
    {"data":"role.roleName"},                                          
    {
    "data":null,
    "render":function(data, type, full, meta ){
        var bstatus;
        var btnstyle;
        switch(data.status)
        {
        case "0":
            bstatus = "正常";
            btnstyle = "success";
            break;
        case "1":
            bstatus = "锁定";
            btnstyle = "danger";
            break;               
        }
        return htmlContent = '<span class="label label-'+btnstyle+' radius">'+bstatus+'</span>';
    }
    },
    {
        "data":"lastLoginTime",
        "className":"ellipsis",
        "render":CONSTANT.DATA_TABLES.COLUMNFUN.ELLIPSIS
     
    },
    {
           "data":"createTime",
        "className":"ellipsis",
        "render":CONSTANT.DATA_TABLES.COLUMNFUN.ELLIPSIS
    },
    {
           "data":null,
        "render":function(data, type, full, meta){
            var statusIcon='&#xe605;';
            if(data.status=="0"){
                statusIcon='&#xe60e;';
            }
             
            var context = [{
                title:"重置密码",
                markClass:"reset-pass",
                iconFont:"&#xe68f;"
            },{
                title:"编辑",
                markClass:"user-edit",
                iconFont:"&#xe6df;"
            },{
                title:"锁定",
                markClass:"user-lock",
                iconFont:statusIcon
            }
            ];              
            return btnIconTemplate(context);
             
        }}];
 
templateParams 

handlebars进行模板渲染的配置数据

tableTheads 表头
btnTools  表格上方的工具栏
formControls 编辑页面的控件设置

?
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67
68
69
70
71
72
73
74
75
76
77
78
79
80
var templateParams = {
    tableTheads:["用户名","姓名","角色","当前状态","最近登录","创建时间","操作"],
    btnTools:[{
        type:"primary",
        size:"M",
        markClass:"add-object",
        iconFont:"&#xe600;",
        name:"添加用户"
    }],
    formControls:[
    {
        edit:true,
        required:false,
        label:"&nbsp;&nbsp;ID",     
        objText:"userIdText",
        input:[{    
            hidden:true,
            name:"userId"
            }]
    },
    {
        edit:false,
        required:true,
        label:"用户名",            
        input:[{    
            hidden:false,
            name:"username",
            placeholder:"2到20个字符"
            }]
    },
    {
        edit:false,
        required:true,
        label:"姓&nbsp;名",           
        input:[{    
            hidden:false,
            name:"realName",
            placeholder:"2到20个字符"
            }]
    },
    {
        edit:false,
        required:true,
        label:"角&nbsp;色",           
        select:[{
            name:"roleId"
            }]
    },
    {
        edit:true,
        required:false,
        label:"创建时间",           
        objText:"createTimeText",
        input:[{    
            hidden:true,
            name:"createTime"
            }]
    },
    {
        edit:true,
        required:false,
        label:"当前状态",           
        objText:"statusText",
        input:[{    
            hidden:true,
            name:"status"
            }]
    },
    {
        edit:true,
        required:false,
        label:"最近登录",           
        objText:"lastLoginTimeText",
        input:[{    
            hidden:true,
            name:"lastLoginTime"
            }]
    },  
    ]       
};
beforeEditInit 

?
1
2
3
4
5
6
7
8
9
10
11
12
13
编辑数据渲染之前(renderData方法之前)的回调 本例中为先查询角色列表var beforeEditInit = function(df){
    $.get("../../mock/role-listAll.json",function(result){
        if(result.returnCode==0){
            var roles = result.data;
            var optionHtml='';
            for(var i=0;i<roles.length;i++){
                optionHtml+='<option value="'+roles[i].roleId+'">'+roles[i].roleName+'</option>';
            }
            $("#roleId").html(optionHtml);
            df.resolve();
        }
    });
};
eventList 

?
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
相关需要绑定的事件列表var eventList = {
        ".add-object":function(){
            publish.renderParams.editPage.modeFlag = 0;                 
            layer_show("增加用户", editHtml, "600", "300",1);
            publish.init();
             
        },
        ".batch-del":function(){
            var checkboxList = $(".selectUser:checked");
            batchDelObjs(checkboxList,"../../mock/user-del.json");
        },
        ".reset-pass":function(){
            //获取当前行数据
            var data = table.row( $(this).parents('tr') ).data();
            layer.confirm('确定要重置该用户的密码吗？',function(index){
                $.get("../../mock/user-resetPwd.json",{userId:data.userId},function(data){
                    if(data.returnCode==0){
                        layer.msg('密码已重置为111111',{icon:1,time:1000});
                    }else{
                        layer.alert(data.msg, {icon: 5});
                    }
                });
                 
            });
        },
        ".user-edit":function(){
            var data = table.row( $(this).parents('tr') ).data();
            if(data.username=="admin"){
                layer.msg('不能修改预置管理员用户信息!',{time:1500});
            }else{
                publish.renderParams.editPage.modeFlag = 1;
                publish.renderParams.editPage.objId = data.userId;
                layer_show("编辑用户信息", editHtml, "600", "480",1);
                publish.init(); 
                 
            }
        },
        ".user-lock":function(){
            var data = table.row( $(this).parents('tr') ).data();
            if(data.username=="admin"){
                layer.msg('不能锁定预置管理员用户!',{time:1500});
            }else{
                var mode = "0";
                var tipMsg = "确定需要解锁该用户吗?";
                var tipMsg1 = "该用户已解锁!";
                if(data.status=="0"){
                    mode = "1";
                    tipMsg = "确认要锁定该用户吗(请谨慎操作,被锁定的用户将不能登录)";
                    tipMsg1 = "已锁定该用户,该用户将不能登录!";
                }       
                layer.confirm(tipMsg,function(index){
                    layer.close(index);
                    $.get("../../mock/user-lock.json",{userId:data.userId,username:data.username,mode:mode},function(data){
                        if(data.returnCode==0){
                            table.ajax.reload(null,false);
                            layer.msg(tipMsg1,{icon:1,time:1000});
                        }else{
                            layer.alert(data.msg, {icon: 5});
                        }
                    });
                     
                });
            } 
        }
         
};
mySetting

?
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
var mySetting = {
        eventList:eventList,
        editPage:{
            beforeInit:beforeEditInit,
            editUrl:"../../mock/user-edit.json",
            getUrl:"../../mock/user-get.json",
            rules:{username:{required:true,minlength:2,maxlength:20},realName:{required:true,minlength: 2,maxlength: 20}},
            renderCallback:function(o){
                $("#roleId").val(o.role.roleId);
                var statusMsg='';
                o.status=="0"?statusMsg="正常":statusMsg="锁定";
                $("#statusText").text(statusMsg);
            }
        },
        listPage:{
            listUrl:"../../mock/user-list.json",
            tableObj:".table-sort",
            columnsSetting:columnsSetting,
            columnsJson:[0,7,8]
        },
        templateParams:templateParams       
    };
 

 

刚学习前端有关的东西，如果你问我为什么不用react或者angluar，其实是我不会哈！

可能用相关的mvvc的框架会更加简单，但是这样自己瞎几把乱搞还是比较有趣的。

代码仅供参考，不规范太啰嗦什么的什么就别在意了。

欢迎一起交流前端知识。
