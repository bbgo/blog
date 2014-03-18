---
layout: post
title: "Javascript模块化开发－轻巧自制"
date: 2014-01-26 02:12:52 +0800
comments: true
categories: javascript
---
### 一、前言

现在javascript的流行，前端的代码越来越复杂，所以我们需要软件工程的思想来开发前端。模块化是必不可少的，这样不仅能够提高代码的可维护性、可扩展性以及鲁棒性，更大的好处就是能够提升整个团队的开发效率，也能够让新进的程序员更快的接手工作。今天晚上根据前辈们的经验，写啦一个简单的模块定义的封装组件，当是练手吧。不过感觉还是蛮好用的。

### 二、学习模块化前我们应该先了解点什么呢？

其实突然就学习模块化的javascript开发，那还是比较丈二和尚，摸不着头脑的。不过如果是做过后台开发的程序员们，可能对于模块化的开发思想并不陌生，因为后台的这方面技术已经很熟悉了。那么这里我就分享一下前端javascript模块化开发的学习。

#### 1. 了解模块化开发思想

如果有软件工程背景，那么这一思想就是你自身就应该掌握的。模块（module）就是可以组合、分解以及更换的单元，其实也满足组合大于继承等这些带来的好处吧。如果看成一个系统的话，我们可以从软件体系结构来理解，模块是较大系统中的独立部件，功能、状态与接口反映外部特性，逻辑反映的是内部特性。

<!-- more -->

#### 2. 了解前端模块化开发带来的好处

模块化的开发模式为前端带来了新大陆，这点不得不承认，现在前端的越发成熟，需要软件工程的这种思想。
玉伯也发表过[前端模块化开发的价值](https://github.com/seajs/seajs/issues/547)

3.了解目前技术，哪些和模块化开发沾边

1) 开发功能模块的时候，可以采用Eva的解决方案（YUI3 + Minify）。

2) 使用流行的javascript模块加载框架：seajs。

3) 如果喜欢轻巧的东西，也可以尝试[带刀](http://stylechen.com/)的[easy.js](http://easyjs.org/)，不错的一个模块加载框架。

4) 也可以尝试支付宝的Alice，这是一款基于CMD规范的东东，首页倒是挺小清新的。

5) 如今比较火的NodeJS这是必须要了解和学习的。

好啦，了解完上面那些技术后，模块化的前端模式应该很熟悉了，如果想扎实一下的话还可以了解一下AMD、CMD规范，具体是什么东西，google一下。接下来我们就来构造一个简单的模块定义器吧，其实写的时候还挺简单的，不过主要是吸收思想，这样学习技术才不会跟不上时代。

### 三、轻巧范例

#### 1. 模块的数据结构（JSON表示）

    module: {
        //模块名称
        moduleName: moduleName,
        //模块依赖集合
        dependencies: dependencies,
        //模块实例工厂
        factory: factory
    }

#### 2. 模块定义

所以我们最后能够形成模块定义的代码如下：

    define: function ( moduleName, dependencies, factory ) {
        if( !modules[moduleName] ) {
            //模块信息
            var module = {
                moduleName: moduleName,
                dependencies: dependencies,
                factory: factory
            };

            modules[moduleName] = module;
        }
    		
        return modules[moduleName];
    }

#### 3. 模块调用

这样我们就定义好了模块，那么我们的入口在哪里呢？我们还需要定义一个use的方法，来成为所谓的main，这样绑定好了才能够调用，现在想来程序也都是这样的。下面这段代码通过递归的产生模块依赖的所有实例，但是这里浪费了一部分instances数组的空间，有时间可以再做哈优化。

    use: function ( moduleName ) {
        //使用括号的方式访问属性，实现动态的赋值（详情查阅“.”和[]的区别）
        var module = modules[moduleName];

        //产生单个实例
        if( !module.instance ) {
            var instances = [], 
                len = module.dependencies.length - 1;

                for( var i = 0; i <= len; i++ ) {
                    var dependency = module.dependencies[i],
                        instance = dependency.instance;

                    if( instance ) {
                        instances.push( instance );
                    } else {
                        //递归，将每次产生的实例放入数组
                        instances.push( arguments.callee( dependency ) );
                    }
                }
                //生成实例
                module.instance = module.factory.apply( null, instances );
        }

        return module.instance;
    }
		
#### 4. 完整代码

最后我形成完整的自己的小库。

    (function ( window, undefined ) {
		var modules = {};
		var Sky = {
			//定义模块的基本信息
			//1.模块名称，2.模块的依赖，3.产生实例的工厂
			define: function ( moduleName, dependencies, factory ) {
				if( !modules[moduleName] ) {
					//模块信息
					var module = {
						moduleName: moduleName,
						dependencies: dependencies,
						factory: factory
					};

					modules[moduleName] = module;
				}
				
				return modules[moduleName];
			},
			//使用依赖
			use: function ( moduleName ) {
				var module = modules[moduleName];

				//产生单个实例
				if( !module.instance ) {
					var instances = [], 
						len = module.dependencies.length - 1;

					for( var i = 0; i <= len; i++ ) {
						var dependency = module.dependencies[i],
							instance = dependency.instance;
						
						if( instance ) {
							instances.push( instance );
						} else {
							//递归，将每次产生的实例放入数组
							instances.push( arguments.callee( dependency ) );
						}
					}
					//生成实例
					module.instance = module.factory.apply( null, instances );
				}

				return module.instance;
			}
		};

		window.Sky = Sky || {};
	})(window);

下面我们来一个完整的例子来使用一下以上我们构建的轻巧代码。

    Sky.define("constant.PI", [], function() {
        return 3.1415926;
    });
     
    Sky.define("shape.Circle", ["constant.PI"], function( pi ) {
        function Circle(r) {
            this.r = r || 0;
        };

        Circle.prototype.area = function() {
            return pi * this.r * this.r;
        };

        return Circle;
    });
 
    Sky.define("shape.Rectangle", [], function() {
        function Rectangle(width, height) {
            this.width = width || 0;
            this.height = height || 0;
        };

        Rectangle.prototype.area = function() {
            return this.width * this.height;
        };

        return Rectangle;
    });
 
    Sky.define("ShapeTypes", ["shape.Circle", "shape.Rectangle"], function( Circle, Rectangle ) {
        return {
            'CIRCLE': Circle,
            'RECTANGLE': Rectangle
        };
    });
 
    Sky.define("ShapeFactory", ["ShapeTypes"], function( ShapeTypes ) {
        return {
            getShape: function(type) {
                var shape;

                switch (type) {
                case 'CIRCLE':
                    shape = new ShapeTypes[type](arguments[1]);
                    break;
                case 'RECTANGLE':
                    shape = new ShapeTypes[type](arguments[1], arguments[2]);
                    break;
                }
                return shape;
            }
        };
    });

    var ShapeFactory = Sky.use("ShapeFactory");
    console.log(ShapeFactory.getShape("CIRCLE").area());
    console.log(ShapeFactory.getShape("RECTANGLE", 2, 3).area());

是不是感觉js代码变得更加清爽了？嘿嘿，上面的例子也是面向接口的，大家也可以看看。

参考前辈：[http://blog.jobbole.com/43649/](http://blog.jobbole.com/43649/)

也许代码有出入，有些地方前辈写得不够细心的我补上了一些，嘿嘿，但是思路是参考这位前辈的。
