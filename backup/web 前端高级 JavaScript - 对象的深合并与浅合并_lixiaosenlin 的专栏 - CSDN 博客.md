> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/lixiaosenlin/article/details/109785219)

### 数据类型检测方法封装

> 在讲对象的深合并与浅合并前，我们先来封装几个数据类型检测的方法，便于后面使用。  
> 以下所有封装的方法在 [JQuery 源码分析 - 数据类型检测方法封装（数字、对象、数组类数组）](https://blog.csdn.net/lixiaosenlin/article/details/109996711)中已经有详细讲解。这里就不再多述。

*   通用数据类型检测 toType

```
var class2type = {};
var toString = class2type.toString;
var mapType = ["Boolean", "Number", "String", "Function", "Array", "Date", "RegExp", "Object", "Error", "Symbol"];
mapType.forEach(function(name){
	class2type["[object "+name+"]"] = name.toLowerCase();
});
function toType(obj){
	if(obj == null) {
		return obj + "";
	};
	return typeof obj === "object" || typeof obj === "function" ?
		class2type[toString.call(obj)] || "object" : typeof obj;
}
```

*   检测是否是纯对象 isPlainObject

```
function isPlainObject(obj){
	var proto, Ctor;
	if(!obj || Object.prototype.toString.call(obj) !== "[object Object]"){
		return false;
	}
	proto = Object.getPrototypeOf(obj);
	if(!proto) return true;
	Ctor = Object.prototype.hasOwnProperty.call(proto, "constructor") && proto.constructor;
	return typeof Ctor === "function" && Function.prototype.toString.call(Ctor) === Function.prototype.toString.call(Object);	
}
```

### 对象的浅合并

> 对象的浅合并就是指：在将两个对象进行合并时，将其中一个对象中的每一项替换到另一个对象中，而不管对象中是否还有子对象，则都只替换第一层。例如：

```
let obj1 = {
	name:'Alvin',
	age: 18,
	friends:{
		name:'张三',
		age: 18,
		friends2: {
			name:'李四',
			age: 20
		}
	}
}

let obj2 = {
	name: 'louyaqian',
	age: 30,
	sex: '女',
	friends:{
		name: 'Alvin',
		sex: '男',
		friends2:{
			sex: '女'
		}		
	}
}
```

> 在上面的两个对象进行浅合并时，会将 obj2 中的每一项键和值替换到 obj1 中，但是既然是浅合并，则只会把第一层级的键和值进行合并和替换，而其它层级的则不再进行合并而是直接替换。  
> **合并规律：**
> 
> *   obj1 和 obj2 都是对象，则遍历 obj2 依次替换 obj1（但只是第一层级）
> *   obj1 不是对象，obj2 是对象，则以 obj2 为准替换 obj1
> *   obj1 是对象，obj2 不是对象，则以 obj1 为准
> *   obj1 和 obj2 都不是对象，则直接用 obj2 替换 obj1
> 
> 上面代码中合并后的结果为：

```
let objMerge = {
	name: 'louyaqian',//替换obj1中的值
	age: 30,//替换obj1中的值
	sex: '女',//obj2中原有的键和值进行合并
	friends:{//obj2中原有的键和值直接替换，不再往下合并
		name: 'Alvin',
		sex: '男',
		friends2:{//obj2中原有的键和值直接替换，不再往下合并
			sex: '女'
		}		
	}
}
```

*   对象浅合并代码实现：

```
function shallowMerge(obj1, obj2){
	var isPlain1 = isPlainObject(obj1);
	var isPlain2 = isPlainObject(obj2);
	//只要obj1不是对象，那么不管obj2是不是对象，都用obj2直接替换obj1
	if(!isPlain1) return obj2;
	//走到这一步时，说明obj1肯定是对象，那如果obj2不是对象，则还是以obj1为主
	if(!isPlain2) return obj1;
	//如果上面两个条件都不成立，那说明obj1和obj2肯定都是对象， 则遍历obj2 进行合并
	let keys = [
		...Object.keys(obj2),
		...Object.getOwnPropertySymbols(obj2)
	]
	keys.forEach(function(key){
		obj1[key] = obj2[key];
	});
	
	return obj1;
}
```

### 对象的深合并

> 通过上面我们对浅合并的理解，相信对深合并差不多也有一些概念了。深合并则是把对象的每一个层级的键和值都会按照浅合并的规则进行替换和合并，不管对象中有多少层级，都会递归去合并。  
> **合并规律：**
> 
> *   obj1 和 obj2 都是对象，则遍历 obj2 依次替换 obj1（每一层级）
> *   obj1 不是对象，obj2 是对象，则以 obj2 为准替换 obj1
> *   obj1 是对象，obj2 不是对象，则以 obj1 为准
> *   obj1 和 obj2 都不是对象，则直接用 obj2 替换 obj1
> 
> 还是上面代码在进行深合并后的结果为：

```
let objMerge = {
	name: 'louyaqian',//替换obj1中的值
	age: 30,//替换obj1中的值
	sex: '女',//obj2中原有的键和值进行合并
	friends:{//继续往下合并
		name: 'Alvin',//obj2中的值合并过来
		age: 18, //obj1中原有的值
		sex: '男',//obj2中的值合并过来
		friends2:{//继续往下合并
			name:'李四',//obj1中原有的值
			age: 20,//obj1中原有的值
			sex: '女'//obj2中的值合并过来
		}		
	}
}
```

*   对象深合并代码实现：

```
function deepMerge(obj1, obj2){
	var isPlain1 = isPlainObject(obj1);
	var isPlain2 = isPlainObject(obj2);
	//obj1或obj2中只要其中一个不是对象，则按照浅合并的规则进行合并
	if(!isPlain1 || !isPlain2) return shallowMerge(obj1, obj2);
	//如果都是对象，则进行每一层级的递归合并
	let keys = [
		...Object.keys(obj2),
		...Object.getOwnPropertySymbols(obj2)
	]
	keys.forEach(function(key){
		obj1[key] =  deepMerge(obj1[key], obj2[key]);//这里递归调用
	});
	
	return obj1;
}
```

> 上面代码完全可以实现对象的深合并了，但是在 js 中运行自己引用自己，比如：let obj = {name:‘louyaqian’}; obj.obj1 = obj;  
> 为了防止这种死循环，我们还需再加一些逻辑

```
function deepMerge(obj1, obj2, cache){
	//防止死循环，这里需要把循环过的对象添加到数组中
	cache = !Array.isArray(cache) ? [] : cache;
	//因为后面只对obj2进行遍历，所以这里只要判断obj2就可以了，如果obj2已经比较合并过了则直接返回obj2，否则在继续合并	
	if(cache.indexOf(obj2)) return obj2;
	cache.push(obj2);
	var isPlain1 = isPlainObject(obj1);
	var isPlain2 = isPlainObject(obj2);
	//obj1或obj2中只要其中一个不是对象，则按照浅合并的规则进行合并
	if(!isPlain1 || !isPlain2) return shallowMerge(obj1, obj2);
	//如果都是对象，则进行每一层级的递归合并
	let keys = [
		...Object.keys(obj2),
		...Object.getOwnPropertySymbols(obj2)
	]
	keys.forEach(function(key){
		obj1[key] =  deepMerge(obj1[key], obj2[key], cache);//这里递归调用
	});
	
	return obj1;
}
```