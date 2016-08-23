---
layout: post
title:  "Swift ThPullRefresh"
date:   2015-10-08 17:50:00 +0800
categories: blogs update
---

　　最近自己写了一个下拉加载最新，上拉加载更多的刷新控件。借鉴了其他优秀开源代码的实现效果比如MJRefresh和DGElasticPullRefresh。主要是为了学习别人的优秀思想。

 

如何使用：

github地址：https://github.com/tanhuiya/ThPullRefresh [点击进入](https://github.com/tanhuiya/ThPullRefresh)

　　Cocoapods 导入：pod 'ThPullRefresh'，

　　在项目中 import 'ThPullRefresh'

　　手动导入：将'ThPullRefresh' 文件夹中的所有文件拽入项目中

 　　head与foot基本效果的添加
 　　

<img src="http://images2015.cnblogs.com/blog/884671/201601/884671-20160121112547937-1808285116.gif" width="270" height="480">　　　　

具体代码如下：

    override func viewDidLoad() {
        super.viewDidLoad()
        self.tableView.registerClass(UITableViewCell.classForCoder(), forCellReuseIdentifier: "tableViewCell")
        self.tableView.rowHeight = UITableViewAutomaticDimension
        self.tableView.estimatedRowHeight = 44
        self.tableView.tableFooterView = UIView()
	//        self.tableView.addHeadRefresh(self) { () -> () in
	//            self.loadNewData()
	//        }
        self.tableView.addHeadRefresh(self, action: "loadNewData")

        self.tableView.head?.hideTimeLabel=true
        self.tableView.addFootRefresh(self, action: "loadMoreData")
    }


    func loadNewData(){
        //延时模拟刷新
        self.index = 0
        DeLayTime(2.0, closure: { () -> () in
            self.dataArr.removeAllObjects()
            for (var i = 0 ;i<5;i++){
                let str = "最新5个cell，第\(self.index++)个"
                self.dataArr.addObject(str)
            }
            self.tableView.reloadData()
            self.tableView .tableHeadStopRefreshing()
        })
        
    }
 DelayTime是一个方法的宏

    
	func  DeLayTime(x:Double,closure:()->()){
	    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, Int64(x * Double(NSEC_PER_SEC))), dispatch_get_main_queue(), closure)
	 }
	 
 要实现果冻效果
 
<img src="http://images2015.cnblogs.com/blog/884671/201601/884671-20160121112857703-1924927269.gif" width="270" height="480">　


代码如下

几个有颜色的点可以忽略，那是开发用于调贝塞尔曲线的。

	/*
   	*bgColor 背景颜色
	*loadingColor 加载的颜色
	*/
	public func addBounceHeadRefresh(target:AnyObject?,bgColor:UIColor,loadingColor:UIColor,action : Selector);
	
	//实现如下
	self.tableView.addBounceHeadRefresh(self,bgColor:UIColor.orangeColor(),loadingColor:UIColor.blueColor(), action: "loadNewData")
	 停止头部刷新和底部刷新
	
	self.tableView.tableHeadStopRefreshing()
	self.tableView.tableFootStopRefreshing
	
 
