#与服务器通信

到现在为止，我们已经看到下面这些内容的主要部分，这些内容是你的Angular应用如何规划设计、不同的AngularJS部件如何装配在一起并正常工作以及AngularJS中的模板代码运行机制的一小部分内容。把这些结合在一起，你就可以搭建一些苗条性感的Web应用，但他们的运作主要还是限制在客户端.在前面第二章,我们看了一点用`$http`服务做服务器端通信的内容,但是在这一章,我们将会深入探讨一下如何在现实世界的应用中使用它(`$http`)

在这一章，我们将讨论一下AngularJS如何帮你同服务器端通信，这其中包括在最底抽象等级的层面或者用它提供的优雅的封装器。而且我们将会深入探讨AngularJS如何用内建缓存机制来帮你加速你的应用.如果你想用`SocketIO`开发一个实时的Angular应用,那么第八章有一个例子，演示了如何把·SocketIO·封装成一个指令然后如何使用这个指令，在这一章，我们就不涉及这方面内容了.

##通过$http进行通行

从Ajax应用(使用XMLHttpRequests)发动一个请求到服务器的传统方式包括：得到一个XMLHttpRequest对象的引用、发起请求、读取响应、检验错误代码然后最后处理服务器响应。它就是下面这样：
    
    var xmlhttp = new XMLHttpRequest();
    xmlhttp.onreadystatechange = function() {
        if (xmlhttp.readystate == 4 && xmlhttp.status == 200) {
            var response = xmlhttp.responseText;
        } else if (xmlhttp.status == 400) { // or really anything in the 4 series
            // Handle error gracefully
        }
    };
    // Setup connection
    xmlhttp.open(“GET”, “http://myserver/api”, true);
    // Make the request
    xmlhttp.send();
    
对于这样一个简单、常用且经常重复的任务，上面这个代码量比较大.如果你想重复性地做这件事,你最终可能会做一个封装或者使用现成的库.

AngularJS XHR(XMLHttpRequest) API遵循Promise接口.因为XHRs是异步方法调用，服务器响应将会在未来一个不定的时间返回(当然希望是越快越好).Promise接口保证了这样的响应将会如何处理,它允许Promise接口的消费者以一种可预计的方式使用这些响应.

假设我们想从我们的服务器取回用户的信息.如果接口在/api/user地址可用,并且接受id作为url参数，那么我们的XHR请求就可以像下面这样使用Angular的核心$http服务:
    
    $http.get('api/user', {params: {id: '5'}
    }).success(function(data, status, headers, config) {
    // Do something successful.
    }).error(function(data, status, headers, config) {
    // Handle the error
    });
    
如果你来自jQuery世界,你可能会注意到：AngularJS和jQuery处理异步需求的方式很相似.

我们上面例子中使用的$htttp.get方法仅仅是AngularJS核心服务$http提供的众多简便方法之一.类似的，如果你想使用AngularJS向相同URL带一些POST请求数据发起一个POST请求，你可以像下面这样做：

    var postData = {text: 'long blob of text'};
    // The next line gets appended to the URL as params
    // so it would become a post request to /api/user?id=5 
    var config = {params: {id: '5'}};
    $http.post('api/user', postData, config
    ).success(function(data, status, headers, config) {
    // Do something successful
    }).error(function(data, status, headers, config) {
    // Handle the error
    });

AngularJS为大多数常用请求类型都提供了类似的简便方法，他们包括：

+ GET
+ HEAD
+ POST
+ DELETE
+ PUT
+ JSONP

###进一步配置你的请求

有时，工具箱提供的标准请求配置还不够,它可能是因为你想做下面这些事情:

+ 你可能想为请求添加权限验证的头信息
+ 改变请求数据的缓存方式
+ 在请求被发送或者响应返回时，对数据以一些方式做一定的转换处理

在上面这样的情况之下,你可以进一步配置请求，通过可选的传递进请求的配置对象.在之前的例子中,我们使用配置对象来标明可选的URL参数，即便我们哪儿演示的GET和POST方法是简便方法。内部的原生方法可能看上面像相面这样：

    $http(config)

下面演示的是一个调用这个方法的伪代码模板:

    $http({
        method: string,
        url: string,
        params: object,
        data: string or object,
        headers: object,
        transformRequest: function transform(data, headersGetter) or an array of functions,
        transformResponse: function transform(data, headersGetter) or an array of functions,
        cache: boolean or Cache object,
        timeout: number,
        withCredentials: boolean
    });

GET、POST和其它的简便方法已经设置了请求的method类型,所以不需要再设置这个，config配置对象是传给·$http.get·、·$http.post·方法的最后一个参数,所以当你使用任何简便方法的时候，你任何能用这个config配置对象.

你也可以通过传入含有下面这些键的属性集config对象来改变已有的request对象

+ method : 一个表示http请求类型的字符串，比如GET,或者POST
+ url : 一个URL字符串代表要请求资源的绝对或相对URL
+ params : 一个对象(准确的说是键值映射)包含字符串到字符串内容，它代表了将会转换为URL参数的键值对，比如下面这样：
    [{key1: 'value1', key2: 'value2'}]
它将会被转换为:
    ?key1=value&key2=value2
这串字符将会加在URL后面，如果在value的位置你用一个对象取代字符串或数字，那这个对象将会转换为JSON字符串.
+ data ：一个字符串或一个对象，它将会被作为请求消息数据被发送.
+ timeout : 这是请求被认定为过期之前所要等待的毫秒数.

还有部分另外的选项可以被配置,在下面的章节中，我们将会深度探索这些选项.

###设定HTTP头信息(Headers)

AngularJS有一个默认的头信息,这个头信息将会对所有的发送请求使用,它包含以下信息:
    1.Accept: application/json, text/plain, /
    2.X-Requested-With:XMLHttpRequest

如果你想设置任何特定的头信息,这儿有两种方法来做这件事：

第一种方法,如果你相对所有的发送请求都使用这些特定头信息,那你需要把特定有信息设置为Angular默认头信息的一部分.可以在`$httpProvider.defaults.headers`配置对象里面设置这个,这个步骤通常会在你的app设置config部分来做.所以如果你想移除"Requested-With"头信息且对所有的GET请求启用"DO NOT TRACK"设置,你可以简单地通过以下代码来做:

    angular.module('MyApp',[]).
        config(function($httpProvider) {
            // Remove the default AngularJS X-Request-With header
            delete $httpProvider.default.headers.common['X-Requested-With'];
            // Set DO NOT TRACK for all Get requests
            $httpProvider.default.headers.get['DNT'] = '1';
    });
    
如果你只想对某个特定的请求设置头信息,而不是设置默认头信息.那么你可以通过给$http服务传递包含指定头信息的config对象来做.相同的定制头信息可以作为第二个参数传递给GET请求,第一个参数是URL字符串：
    
    $http.get('api/user', {
    // Set the Authorization header. In an actual app, you would get the auth
    // token from a service
    headers: {'Authorization': 'Basic Qzsda231231'},
    params: {id: 5}
    }).success(function() { // Handle success });
    
如何在应用中处理权限验证头信息的成熟示例将会在第八章的Cheetsheets示例部分给出.

###缓存响应数据

AngularJS为HTTP GET请求提供了一个开箱即用的简单缓存系统.缺省情况下,它对所有的请求都是禁用的,但是如果你想对你的请求启用缓存系统，你可以使用以下代码:

    $http.get('http://server/myapi', {
        cache: true
    }).success(function() { // Handle success });

这段代码启用了缓存系统，然后AngularJS将会缓存来自Server的响应数据.但对相同的URL的请求第二次发出时,AngularJS将会从缓存里面取出前一次的响应数据作为响应返回.这个缓存系统也很智能,即使你同时对相同URL发出多个请求,只有一个请求会发向Server,这个请求的响应数据将会反馈给所有(同时发起的)请求。

然而这种做法从可用性的角度看可能是有所冲突的,当一个用户首先看到旧的结果,然后新的结果突然冒出来，比如一个用户可能即将单击一个数据项,而实际上这个数据项后台已经发生了变化.

注意所有响应(即使是从缓存里取出的)本质上仍旧是异步响应.换句话说，期望你的利用缓存响应时的异步代码运行仍旧和他向后台服务器发出请求时的代码运行机制是一样的.

###对请求(Request)和响应(Response)的数据所做的转换

AngularJS对所有`$http`服务发起的请求和响应做一些基本的转换,它们包括:

+ 请求(Request)转换:
    如果请求的Config配置对象的data属性包含一个对象，将会把这个对象序列化为JSON格式.
+ 响应(Response)转换:
    如果探测到一个XSRF头,把它剥离掉.如果响应数据被探测为JSON格式,用JSON解析器把它反序列化为JSON对象.

如果你需要部分系统默认提供的转换,或者想使用你自己的转换,你可以把你的转换函数作为Config配置对象的一部分传递进去(后面有细述).这些转换函数得到HTTP请求和HTTP响应的数据主体以及它们的头信息.然后把序列化的修改后版本返回出来.在Config对象里面配置这些函数需要使用·transformRequest·键和·transformResponse·键,这些都可以通过使用`$httpProvider·服务在模块的config函数里面配置它.

我们什么时候使用这些哪?让我假设我们有一个服务器，它更习惯于jQuery运行的方式.它可能希望我们的POST数据以`key1=val1&key2=val2`字符串的形式传递，而不是以`{key1:val1,key2:val2}`这样的JSON格式.这个时候，我们可能相对每个请求做这样的转换,或者单个地增加transformRequest转换函数,为了达成这个示例这样的目标,我们将要设置一个通用transformRequet转换函数,以便对所有的发出请求,这个函数都可以把JSON格式转换为键值对字符串,下面代码演示了如何做这个工作:

    var module = angular.module('myApp');
    module.config(function ($httpProvider) {
        $httpProvider.defaults.transformRequest = function(data) {
            // We are using jQuery’s param method to convert our
            // JSON data into the string form
            return $.param(data);
       };
    });

##单元测试

目前为止，我们已经了解如何使用`$http`服务以及如何以可能的方式做你需要的配置.但是如何写一些单元测试来保证这些够真实有效的运行哪?

正如我们曾经三番五次的提到的那样,AngularJS一直以测试为先的原则而设计.所以Angualr有一个模拟服务器后端，在单元测试中,它可以帮你就可以测试你发出的请求是否正确,甚至可以精确控制模拟响应如何得到处理,什么时候得到处理.

让我们探索一下下面这样的单元测试场景:一个控制向服务器发起请求,从服务器得到数据,把它赋给作用域内的模型,然后以具体的模板格式显示出来.

我们的`NameListCtrl`控制器是一个非常简单的控制器.它的存在只有一个目的：访问`names`API接口，然后把得到数据存储在作用域scope模型内.

    function NamesListCtrl($scope, $http) {
        $http.get('http://server/names', {params: {filter: ‘none’}}).
            success(function(data) {
                $scope.names = data;
        });
    }

怎样对这个控制器做单元测试？在我们的单元测试中,我们必须保证下面这些条件:

+ `NamesListCtrl`能够找到所有的依赖项(并且正确的得到注入的这些依赖)》
+ 当控制器加载时尽可能快地立刻发情请求从服务器得到names数据.
+ 控制器能够正确地把响应数据存储到作用域scope的`names`变量属性中.

在我们的单元测试中构造一个控制器时，我们给它注入一个scope作用域和一个伪造的HTTP服务,在构建测试控制器的方式和生产中构建控制器的方式其实是一样的.这是推荐方法，尽管它看上去上有点复杂。让我看一下具体代码：
    
    describe('NamesListCtrl', function(){
        var scope, ctrl, mockBackend;

        // AngularJS is responsible for injecting these in tests
        beforeEach(inject(function(_$httpBackend_, $rootScope, $controller) {
            // This is a fake backend, so that you can control the requests
            // and responses from the server
            mockBackend = _$httpBackend_;

            // We set an expectation before creating our controller,
            // because this call will get triggered when the controller is created
            mockBackend.expectGET('http://server/names?filter=none').
                respond(['Brad', 'Shyam']);
            scope = $rootScope.$new();

            // Create a controller the same way AngularJS would in production
            ctrl = $controller(PhoneListCtrl, {$scope: scope});
        }));

        it('should fetch names from server on load', function() {
            // Initially, the request has not returned a response
            expect(scope.names).toBeUndefined();
            
            // Tell the fake backend to return responses to all current requests
            // that are in flight.
            mockBackend.flush();
            // Now names should be set on the scope
            expect(scope.names).toEqual(['Brad', 'Shyam’]);
        });
    });

##使用RESTful资源

·$http·服务提供一个比较底层的实现来帮你发起XHR请求,但是同时也给提供了很强的可控性和弹性.在大多数情况下,我们处理的是对象集或者是封装有一定属性和方法的对象模型,比如带有个人资料的自然人对象或者信用卡对象.

在上面这样的情况下，如果我们自己构建一个JS对象来表示这种较复杂对象模型，那做法就有点不够nice.如果我们仅仅想编辑某个对象的属性、保存或者更新一个对象，那我们如何让这些状态在服务器端持久化.

`$resource`正好给你提供这种能力.AngularJS resources可以帮助我们以描述的方式来定义对象模型，可以定义一下这些特征：

+ resource的服务器端URL
+ 这种请求常用参数的类型
+ (你可以免费自动得到get、save、query、remove和delete方法),除了那些方法，你可以定义其它的方法，这些方法封装了对象模型的特定功能和业务逻辑(比如信用卡模型的charge()付费方法)
+ 响应的期望类型(数组或者一个独立对象)
+ 头信息

------------------------------------------------------
什么时候你可以用Angular Resources组件？

只有你的服务器端设施是以RESTful方式提供服务的时候，你才应该用Angular resources组件.比如信用卡那个案例,我们将用它作为本章这一部分的例子，他将包括以下内容:

1. 给地址`/user/123/card`发送一个GET请求，将会得到用户123的信用卡列表.
2. 给地址`/user/123/card/15`发送一个GET请求,将会得到用户123本人的ID为15的信用卡信息
3. 给地址`/user/123/card`发送一个在POST请求数据部分包含信用卡信息的POST请求,将会为用户123新创建一个信用卡
4. 给地址`/user/123/card/15`发送一个在POST请求数据部分包含信用卡信息的POST请求,将会更新用户123的ID为5的信用卡的信息.
5. 给地址`/user/123/card/15`一个方法为DELETE类型的请求,将会删除掉用户123的ID为5的信用卡的数据.

-------------------------------------------------------

除了按照你的要求给你提供一个查询服务器端信息的对象,`$resource`还可以让你使用返回的数据对象就像他们是持久化数据模型一样，可以修改他们，还可以把你的修改持久化存储下来.

`ngResource`是一个单独的、可选的模块.要想使用它，你看需要做以下事情：

+ 在你的HTML文件里面引用angular-resource.js的实际地址
+ 在你的模块依赖里面声明对它的依赖(例如,angular.module('myModule',['ngResource'])).
+ 在需要的地方，注入$resource这个依赖项.

在我们看怎样用ngResource方法创建一个resource资源之前，我们先看一下怎样用基本的$http服务做类似的事情.比如我们的信用卡resource，我们想能够读取、查询、保存信用卡信息，另外还要能为信用卡还款.

这儿是上述需求一个可能的实现：

    myAppModule.factory('CreditCard', ['$http', function($http) {
        var baseUrl = '/user/123/card';
        return {
            get: function(cardId) {
                return $http.get(baseUrl + '/' + cardId);
            },
            save: function(card) {
                var url = card.id ? baseUrl + '/' + card.id : baseUrl;
                return $http.post(url, card);
            },
            query: function() {
                return $http.get(baseUrl);
            },
            charge: function(card) {
                return $http.post(baseUrl + '/' + card.id, card, {params: {charge: true}});
            }
        };
    }]);

取代以上方式，你也可以轻松创建一个在你的应用中始终如一的Angular资源服务，就像下面代码这样：
    
    myAppModule.factory('CreditCard', ['$resource', function($resource) {
        return $resource('/user/:userId/card/:cardId',
            {userId: 123, cardId: '@id'},
            {charge: {method:'POST', params:{charge:true}, isArray:false});
    }]);

做到现在，你就可以任何时候从Angular注入器里面请求一个CreditCard依赖，你就会得到一个Angular资源,默认情况下，这个资源会提供一些初始的可用方法.表格5-1列出了这些初始方法以及他们的运行行为，这样你就可以知道在服务器怎样配置来配合这些方法了.

表格5-1 一个信用卡reource
Function                           Method   URL                                        Expected Return
CreditCard.get({id: 11})           GET      /user/123/card/11                          Single JSON
CreditCard.save({}, ccard)         POST     /user/123/card with post data “ccard”      Single JSON
CreditCard.save({id: 11}, ccard)   POST     /user/123/card/11 with post data “ccard”   Single JSON
CreditCard.query()                 GET      /user/123/card                             JSON Array
CreditCard.remove({id: 11})        DELETE   /user/123/card/11                          Single JSON
CreditCard.delete({id: 11})        DELETE   /user/123/card/11                          Single JSON

让我们看一个信用卡resource使用的代码样例，这样可以让你理解起来觉得更浅显易懂.

    // Let us assume that the CreditCard service is injected here
    // We can retrieve a collection from the server which makes the request
    // GET: /user/123/card
    var cards = CreditCard.query();
    // We can get a single card, and work with it from the callback as well
    CreditCard.get({cardId: 456}, function(card) {
        // each item is an instance of CreditCard
        expect(card instanceof CreditCard).toEqual(true);
        card.name = "J. Smith";
        // non-GET methods are mapped onto the instances
        card.$save();
        // our custom method is mapped as well.
        card.$charge({amount:9.99});
        // Makes a POST: /user/123/card/456?amount=9.99&charge=true
        // with data {id:456, number:'1234', name:'J. Smith'}
    });

前面这个样例代码里面发生了很多事情，所以我们将会一个一个地认真讲解其中的重要部分:

###resource资源的声明

声明你自己的`$resource`非常简单，只要调用注入的$resource函数,并给他传入正确的参数即可。(你现在应该已经知道如何注入依赖,对吧?)

$resource函数只有一个必须参数,就是提供后台资源数据的URL地址,另外还有两个可选参数:默认request参数信息和其它的想在资源上要配置的动作.

请注意：第一个URL地址参数中的的变量数据是参数化可配置的(:冒号是参数变量的语法符号,比如`:userId`以为这个参数将会被实际的userId参数变量取代(译者注:写过参数化SQL语句的人应该很熟悉),而`:cardId`将会被cardId参数变量的值所取代),如果没有给函数传递这些参数变量,那那个位置将会被空字符取代.

函数的第二个参数则负责提供所有请求的默认参数变量信息.在这个案例中，我们给userId参数传递一个常量:123,cardId参数则更有意思,我们给cardId参数传递了一个"@id"字符串.这意味着如果我们使用一个从服务器返回的对象而且我们可以调用这个对象的任何方法(比如$save),那么这个对象的id属性将会被取出来赋给cardId字段.

函数的第三个参数是一些我们想要暴露的其它方法，这些方法是对定制资源做操作的方法.在下面的章节，我们将会深度讨论这个话题

###定制方法














