---
layout: post
title: Meteor Display Date
date: 2017-02-03
tags: Meteor
---
網路抄的

main.html :

```
<head>
        <title>textcircle</title>
</head>

<body>
        <h1>Date Display</h1>

        {{> date_display}}
</body>

<template name="date_display">
        {{current_date}}
</template>
```

main.js :

```
if (Meteor.isClient){
	// uodate session variable very 1000 ms
	Meteor.setInterval(function(){
		Session.set("current_date", new Date());
	}, 1000);
	// display session var
	Template.date_display.helpers({
		current_date:function(){
			return Session.get("current_date");
		}
	});
}

if (Meteor.isServer){
	Meteor.startup(function(){
		// code to run on server at startup
	});
}
```

<img src="/images/posts/Server/p60.png">

配合

https://www.echochio.nctu.me/2017/04/meteor_docker/

放到 Android 上 .... XD
