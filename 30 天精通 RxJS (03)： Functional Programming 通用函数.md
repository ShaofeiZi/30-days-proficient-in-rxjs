# 30 天精通 RxJS (03)： Functional Programming 通用函数

> 
> 
> 了解 Functional Programming 的通用函数，能让我们写出更简洁的代码，也能帮助我们学习 RxJS。
> 
> 

这是【30天精通 RxJS】的 03 篇，如果还没看过 02 篇可以往这边走：
[30 天精通 RxJS (02)： Functional Programming 基本观念](https://github.com/ShaofeiZi/30-days-proficient-in-rxjs/blob/master/30%20%E5%A4%A9%E7%B2%BE%E9%80%9A%20RxJS%20(02)%EF%BC%9A%20Functional%20Programming%20%E5%9F%BA%E6%9C%AC%E8%A7%82%E5%BF%B5.md)

读者可能会很好奇，我们的主题是 RxJS 为什么要特别讲 Functional Programming 的通用函数呢？ 实际上，RxJS 核心的 Observable 操作观念跟 FP 的数组操作是极为相近的，只学会以下几个基本的方法跟观念后，会让我们之后上手 Observable 简单很多！

今天的代码比较多，大家可以直接看视频！

## ForEach

> 
> 
> forEach 是 JavaScript 在 ES5 后，原生就有支援的方法。
> 
> 

原本我们可能要透过 for loop 取出数组中的每一个元素

```
var arr = ['Jerry', 'Anna'];

for(var i = 0; i < arr.length; i++) {
	console.log(arr[i]);
}

```

现在可以直接透过数组的 forEach 取出每一个元素。

```
var arr = ['Jerry', 'Anna'];

arr.forEach(item => console.log(item));

```

forEach 是 FP 操作数组的基本方法，我们可以用这个方法来实例下面三个我们今天要讲的重点分别为 map, filter, concatAll。

## Map

试着把 newCourseList 每个元素的 { id, title } 塞到新的数组 idAndTitlePairs

```
var newCourseList = [
	{
		"id": 511021,
		"title": "React for Beginners",
		"coverPng": "https://res.cloudinary.com/dohtkyi84/image/upload/v1481226146/react-cover.png",
		"rating": 5
	},
	{
		"id": 511022,
		"title": "Vue2 for Beginners",
		"coverPng": "https://res.cloudinary.com/dohtkyi84/image/upload/v1481226146/react-cover.png",
		"rating": 5
	},
	{
		"id": 511023,
		"title": "Angular2 for Beginners",
		"coverPng": "https://res.cloudinary.com/dohtkyi84/image/upload/v1481226146/react-cover.png",
		"rating": 5
	},
	{
		"id": 511024,
		"title": "Webpack for Beginners",
		"coverPng": "https://res.cloudinary.com/dohtkyi84/image/upload/v1481226146/react-cover.png",
		"rating": 4
	}
], idAndTitle = [];

newCourseList.forEach((course) => {
	idAndTitle.push({ id: course.id, title: course.title });
});

```

虽然我们成功的把 newCourseList 转成 idAndTitlePairs，但这样的写法还是显得有点太复杂了，我们可以用更抽象化的方式来完成。

上面我们练习到 newCourseList 转换成一个新的数组 idAndTitlePairs，这个转换的过程其实就是两件事

*   遍历 newCourseList 所有的元素
*   把每个元素的预期值给到新的数组

把这个过程抽象化成一个方法 map，以下是简化的基本思路：

1.  我们会让每个 数组 都有一个 map 方法
2.  这个方法会让使用者自订传入一个 callback function
3.  这个 callback function 会回传使用者预期的元素

> 
> 
> 虽然 ES5 之后原生的 JavaScript 数组有 map 方法了，但希望读者自己写一遍，能帮助理解。
> 
> 

```
// 我们希望每一个数组都有 map 这个方法，所以我们在 Array.prototype 扩充 map function
Array.prototype.map = function(callback) {
  var result = []; // map 最后一定会返回一个新数组，所以我们先创建一个新数组

  this.forEach(function(element, index) {
	  // this 就是呼叫 map 的数组
	  result.push(callback(element, index));
	  // 执行使用者定义的 callback， callback 会回传使用者预期的元素，所以我们把它 push 进新数组
  })

  return result;
}

```

> 
> 
> 这里用到了 JavaScript 的 prototype chain 以及 this 等观念，可以看此[视频](https://www.youtube.com/watch?v=BlT6pCG2M1I)了解！
> 
> 

到这里我们就实例完成 map 的方法了，让我们来试试这个方法吧！

```
var idAndTitle = newCourseList
                 .map((course) => {
                     return { id: course.id, title: course.title };
                 });

```

可以看到我们的代码更加的简洁！

## Filter

如果我们希望过滤一个数组，留下数组中我们想要的元素，并产生一个新的数组，要怎么做呢？
先让我们用 forEach 完成！

让我们过滤出 rating 值是 5 的元素

```
var ratingIsFive = [];

newCourseList.forEach((course) => {
	if(course.rating === 5) {
		ratingIsFive.push(course);
	}
});

```

同样的我们试着来简化这个过程，首先在这个转换的过程中，我们做了两件事：

1.  遍历 newCourseList 中的所有元素
2.  判断元素是否符合条件，符合则加到新的数组中

```
Array.prototype.filter = function(callback) {
	var result = [];
	this.forEach((item, index) => {
		if(callback(item, index))
			result.push(item);
	});
	return result;
}

```

试试这个方法

```
var ratingIsFive = newCourseList
                   .filter((course) => course.rating === 5);

```

会发现我们的代码又变简单了，接着我们试着把 filter, map 串起来。

如果我想要取出所有 rating 是 5 的所有 course title

```
var ratingIsFive = newCourseList
                   .filter((course) => course.rating === 5)
                   .map(course => course.title);

```

## ConcatAll

有时候我们会遇到组出一个二维数组，但我们希望数组是一维的，问题如下：

假如我们要取出 courseLists 中所有 rating 为 5 的课程，这时可能就会用到两个 forEach

```
var user = {
  id: 888,
  name: 'JerryHong',
  courseLists: [{
    "name": "My Courses",
    "courses": [{
      "id": 511019,
      "title": "React for Beginners",
      "coverPng": "https://res.cloudinary.com/dohtkyi84/image/upload/v1481226146/react-cover.png",
      "tags": [{ id: 1, name: "JavaScript" }],
      "rating": 5
    }, {
      "id": 511020,
      "title": "Front-End automat workflow",
      "coverPng": "https://res.cloudinary.com/dohtkyi84/image/upload/v1481226146/react-cover.png",
      "tags": [{ "id": 2, "name": "gulp" }, { "id": 3, "name": "webpack" }],
      "rating": 4
    }]
  }, {
    "name": "New Release",
    "courses": [{
      "id": 511022,
      "title": "Vue2 for Beginners",
      "coverPng": "https://res.cloudinary.com/dohtkyi84/image/upload/v1481226146/react-cover.png",
      "tags": [{ id: 1, name: "JavaScript" }],
      "rating": 5
    }, {
      "id": 511023,
      "title": "Angular2 for Beginners",
      "coverPng": "https://res.cloudinary.com/dohtkyi84/image/upload/v1481226146/react-cover.png",
      "tags": [{ id: 1, name: "JavaScript" }],
      "rating": 4
    }]
  }]
};

var allCourseIds = [];

user.courseLists.forEach(list => {
  list.courses
    .filter(item => item.rating === 5)
    .forEach(item => {
      allCourseIds.push(item)
    })
})

```

可以看到上面的代码，我们用了较为低阶的操作来解决这个问题，我们刚刚已经试着用抽象化的方式实例了 map 跟 filter，那我们同样也能够定义一个方法用来 摊平二维数组。

让我们来加入一个 concatAll 方法来简化这段代码吧！
concatAll 要做的事情很简单，就是把一个二维数组转成一维。

```
Array.prototype.concatAll = function() {
  var result = [];

  // 用 apply 完成
  this.forEach((array) => {
    result.push.apply(result, array);
  });

  // 用两个 forEach 完成
  // this.forEach((array) => {
  //   array.forEach(item => {
  //     result.push(item)
  //   })
  // });

  // 用 ES6 spread 完成
  // this.forEach((array) => {
  //   result.push(...array);
  // })

  return result;
};

```

同样的我们用前面定要好的 courseLists 来试试 concatAll 吧！

```
var allCourseIds = user.courseLists.map(list => {
	return list.courses.filter(course => course.rating === 5)
}).concatAll()

```

这边出一个比较难的题目，大家可以想想看要怎么解

```
var courseLists = [{
  "name": "My Courses",
  "courses": [{
    "id": 511019,
    "title": "React for Beginners",
    "covers": [{
      width: 150,
      height: 200,
      url: "http://placeimg.com/150/200/tech"
    }, {
      width: 200,
      height: 200,
      url: "http://placeimg.com/200/200/tech"
    }, {
      width: 300,
      height: 200,
      url: "http://placeimg.com/300/200/tech"
    }],
    "tags": [{
      id: 1,
      name: "JavaScript"
    }],
    "rating": 5
  }, {
    "id": 511020,
    "title": "Front-End automat workflow",
    "covers": [{
      width: 150,
      height: 200,
      url: "http://placeimg.com/150/200/arch"
    }, {
      width: 200,
      height: 200,
      url: "http://placeimg.com/200/200/arch"
    }, {
      width: 300,
      height: 200,
      url: "http://placeimg.com/300/200/arch"
    }],
    "tags": [{
      "id": 2,
      "name": "gulp"
    }, {
      "id": 3,
      "name": "webpack"
    }],
    "rating": 5
  }]
}, {
  "name": "New Release",
  "courses": [{
    "id": 511022,
    "title": "Vue2 for Beginners",
    "covers": [{
      width: 150,
      height: 200,
      url: "http://placeimg.com/150/200/nature"
    }, {
      width: 200,
      height: 200,
      url: "http://placeimg.com/200/200/nature"
    }, {
      width: 300,
      height: 200,
      url: "http://placeimg.com/300/200/nature"
    }],
    "tags": [{
      id: 1,
      name: "JavaScript"
    }],
    "rating": 5
  }, {
    "id": 511023,
    "title": "Angular2 for Beginners",
    "covers": [{
      width: 150,
      height: 200,
      url: "http://placeimg.com/150/200/people"
    }, {
      width: 200,
      height: 200,
      url: "http://placeimg.com/200/200/people"
    }, {
      width: 300,
      height: 200,
      url: "http://placeimg.com/300/200/people"
    }],
    "tags": [{
      id: 1,
      name: "JavaScript"
    }],
    "rating": 5
  }]
}];

/* 
var result = courseList
不得直接使用索引 covers[0]，请用 concatAll, map, filter, forEach 完成
result 结果为 [
    {
      id: 511019,
      title: "React for Beginners",
      cover: "http://placeimg.com/150/200/tech"
    }, {
      id: 511020,
      title: "Front-End automat workflow",
      cover: "http://placeimg.com/150/200/arch"
    }, {
      id: 511022,
      title: "Vue2 for Beginners",
      cover: "http://placeimg.com/150/200/nature"
    }, {
      id: 511023,
      title: "Angular2 for Beginners",
      cover: "http://placeimg.com/150/200/people"
    },
 ]
*/

```

练习连结： [JSBin](https://jsbin.com/wifulas/6/edit?js,output) | [JSFiddle](https://jsfiddle.net/s6323859/5wcgnf89/1/)

这题有点难，大家可以想想看，我把答案写在[这里](https://jsbin.com/rahezacane/edit?js,console)了！

如果大家还想做更多的练习可以到这个连结：<http://reactivex.io/learnrx/>

> 
> 
> 这个连结是 Jafar 大神为他的 RxJS workshop 所做的练习网站！
> 
> 

## 今日小结

今天讲了 FP 操作数组的三个通用函数 forEach, map, filter，以及我们自己定义的一个方法叫 concatAll。这几天我们把学习 RxJS 的前置观念跟知识基本上都讲完了，明天我们就开始进入 RxJS 的重点核心 Observable 喽！