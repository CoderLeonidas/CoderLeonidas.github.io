# 如何实现镂空的window效果

## 前言 

最近有个需求，是需要一个windowA作为容器。这个windowA的层次可能在最上层。当有其他app的windowB出现时，这个windowA会在最上面。这样一来，就会将其他active的app的windowB给遮挡住。但是我又不想改变这个windowA的层次，想始终让它保持在最上层，
	
那该怎么做呢？有一种做法就是，镂空这个windowA与windowB叠加的部分，使镂空区域的下方显示出windowB的界面。当然这个镂空区域也要能够响应windowB的事件，这也就意味着在镂空区域能够点击到windowB的按钮等控件，并响应。


	
	