## JSON-RPC 2.0 规范学习以及前端接口封装

### JSON-RPC 2.0 规范学习

* 1 Json-rpc 2.0与传统json的后台接口不同的地方
	 - 最明显的不同体现在接口的调用上，传统的方法，一般是后端暴露接口给前端，前端进行接口联调；Json-rpc 2.0，在接口的调用上，实际上是调用了后台地址上的一个method方法，由method方法，再去映射到后端服务器的对应逻辑上。
	 - **example**
		 <pre>
		 {
	    "jsonrpc" : "2.0",
	    <strong>"method" : "subtract",</strong>
	    "params" : {
	        "subtrahend" : 23,
	        "minuend" : 42
	    },
	    "id" : 3
	     }
	    </pre>

* 2 请求对象
  	- 发送一个请求对象至服务端代表一个rpc调用，一个请求对象包含下列成员：
  	- **jsonrpc**
  		<pre>指定JSON-RPC协议版本的字符串，必须准确写为“2.0”</pre>
    - **method**
  		<pre>包含所要调用方法名称的字符串，以rpc开头的方法名，用英文句号（U+002E or ASCII 46）
		连接的为预留给rpc内部的方法名及扩展名，且不能在其他地方使用。</pre>
    - **params**
  		<pre>调用方法所需要的结构化参数值，该成员参数可以被省略。</pre>
    - **id**
  		<pre>已建立客户端的唯一标识id，值必须包含一个字符串、数值或NULL空值。
		如果不包含该成员则被认定为是一个通知。该值一般不为NULL[1]，若为数值则不应该包含小数[2]</pre>

    -     <pre>[1]在请求对象中不建议使用NULL作为id值，因为该规范将使用空值认定为未知id的请求。
		另外，由于JSON-RPC 1.0 的通知使用了空值，这可能引起处理上的混淆。
     [2]使用小数是不确定性的，因为许多十进制小数不能精准的表达为二进制小数。</pre>

* 3 响应对象
	- 当发起一个rpc调用时，除通知外都必须回复响应。响应表示为一个json对象，使用以下成员：
	- **jsonrpc**
		<pre>指定JSON-RPC协议版本的字符串，必须准确写为“2.0”</pre>
	- **result**
		<pre>该成员在成功时必须包含。</pre>
		<pre>当调用方法引起错误时必须不包含该成员。</pre>
		<pre>服务端中的被调用方法决定了该成员的值。</pre>
	- **error**
		<pre>该成员在失败是必须包含。</pre>
		<pre>当没有引起错误的时必须不包含该成员。</pre>
		<pre>该成员参数值必须为5.1中定义的对象。</pre>
	- **id**
		<pre>该成员必须包含。。</pre>
		<pre>该成员值必须于请求对象中的id成员值一致。</pre>
		<pre>若在检查请求对象id时错误（例如参数错误或无效请求），则该值必须为空值。</pre>

* 4 错误对象
	- 当一个rpc调用遇到错误时，返回的响应对象必须包含错误成员参数，并且为带有下列成员参数的对象
	- **code**
	- <pre>使用数值表示该异常的错误类型。
		必须为整数。
	  </pre>
	- **message**
	- <pre>对该错误的简单描述字符串。
		该描述应尽量限定在简短的一句话。
	  </pre>
	- **data**
	- <pre>包含关于错误附加信息的基本类型或结构化类型。该成员可忽略。
		该成员值由服务端定义（例如详细的错误信息，嵌套的错误等）。
	  </pre>
	![](http://i4.piimg.com/567571/ff156c44b7c37eb5.png)


### 前端接口封装
	
	.service('dataService', ['$http','$q','showMsgService',function ($http, $q, showMsgService) {
	//	post方法
			this.postJsonData = function (parameters) {
	            var deferred = $q.defer();
	            
	            parameters = {
	            	"jsonrpc" : "2.0",
	            	 parameters
	            };
	
	            $http({
	                method:'POST',
	                url: serverUrl,      //字符串，请求的目标
	                data: parameters,    //在发送post请求时使用，作为消息体发送到服务器
	                headers: {'Content-Type':'application/raw'},    //一个列表，每个元素都是一个函数，返回http头
	                transformRequest: function (parameters) {       //数或者函数数组，用来对http请求的请求体和头信息进行转换，并返回转换后的结果。
	                    var str = [];
	                    for (var p in parameters)
	                        str.push(encodeURIComponent(p) + "=" + encodeURIComponent(parameters[p]));
	                    return str.join("&");
	                }
	
	            }).success(function (data, status, headers, config) {
	                if (data.result != 200) {
	                    showMsgService.showMsg(data.msg);
	                    deferred.reject(data);
	                } else {
	                    deferred.resolve(data);
	                }
	            }).error(function (data, status, headers, config) {
	                if(status == 400) {
	                    showMsgService.showMsg('请求参数错误！');
	                } else if(status == 405){
	                    showMsgService.showMsg('无效的请求！');
	                } else {
	                    showMsgService.showMsg('网络连接异常，请检查网络后重试！');
	                }
	            });
	            return deferred.promise;
	        }	
	        
	//	get方法
	        this.getJsonData = function (url, parameters) {
	            var deferred = $q.defer();
	
	            parameters = {
	            	"jsonrpc" : "2.0",
	            	 parameters
	            };
	
	            $http.get(serverUrl, {params: parameters})
	            .success(function (data, status, headers, config) {
	                if (data.code != 200) {
	                    showMsgService.showMsg(data.desc);
	                    deferred.reject(data);
	                } else {
	                    deferred.resolve(data);
	                }
	            }).error(function (data, status, headers, config) {
	                if(status == 400) {
	                    showMsgService.showMsg('请求参数错误！');
	                } else if(status == 405){
	                    showMsgService.showMsg('无效的请求！');
	                } else {
	                    showMsgService.showMsg('网络连接异常，请检查网络后重试！');
	                }
	            });
	            return deferred.promise;
	        };
	
	    }])
	
