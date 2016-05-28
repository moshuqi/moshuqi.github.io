---
layout: post
title:  iOS自动化测试 Automation
category: "iOS"
---



var target = UIATarget.localTarget();
var app = target.frontMostApp();
var window = app.mainWindow();

var runBtn = window.buttons()["开始跑步"];

target.delay(6);
runBtn.tap();


target.logElementTree();

target.tap({x:184.00, y:586.50});

var target = UIATarget.localTarget();
target.logElementTree();

var app = target.frontMostApp();
var window = app.mainWindow();

var rect = target.rect();
var w = rect.size.width;
var h = rect.size.height;

for (;;)
{
    var x = Math.random() * w;
    var y = Math.random() * h;
    target.tap({x, y});
}