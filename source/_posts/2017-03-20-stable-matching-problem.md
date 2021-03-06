title: 简述稳定匹配问题
---
# 什么是稳定匹配问题？
稳定匹配问题最早是在1962年由David Gale与Lloyd Shapley提出的。这个问题最早是为了实现大学招生或者公司招聘的自动化而提出的。我们在叙述这个问题的时候常常将大学与大学生比喻成男人与女人，将招生环节比喻成婚姻匹配环节，在这样的设定下，我们再来看一下这个问题的完整叙述：假设现在有n个男人与n个女人，每个人男人的手中都有一张对于这n个女人的喜好排序表，每个女人的手中都有一张对于这n个男人的喜好排序表。那么有没有一种1对1的婚姻匹配方法能使得在最后的匹配中，每一个人都有结婚对象并且匹配中没有出现不稳定因子（instability）。不稳定因子也就是有这么一对男女，他们虽然没有在最后的匹配中成为结婚对象，但是他们比起现在的结婚对象，都更喜欢对方。我们来看一下不稳定因子的实际案例
{% codeblock lang:html line_number:false %}
现在我们有两位男性m1与m2，和两位女性w1与w2，以下是每个人对异性的喜好排序
m1为 w1 > w2
m2为 w1 > w2
w1为 m1 > m2
w2为 m1 > m2

那么从这个例子中我们可以产生两种不同的匹配方式
第一种匹配方式为 m1与w2，m2与w1
第二种匹配方式为 m1与w1，m2与w2
{% endcodeblock %}
在上面两种匹配方式中，我们可以看到第一种匹配方式是存在不稳定因子的。因为在m1的喜好排序中他更为喜欢w1，而在w1的喜好列表中，w1也更为喜欢m1。在这种情况下，我们给出`m1与w2`，`m2与w1`这样的匹配方式是没有达到匹配最优化的，这种解也并不是我们想要的。相反来看第二种匹配方式，在这种匹配方式中我们无法找到任何不稳定因子，那么这种匹配方式也就是我们所说的稳定匹配。

# Gale-Shapley算法
David Gale与Lloyd Shapley提出这个问题后，他们就研究出了这种名为Gale-Shapley的算法来解决这个问题。先来看一下以下的Gale-Shapley算法的伪代码实现
{% codeblock lang:html %}
将所有男人与女人先初始化成未匹配的状态
while（有一个未匹配的男人m并且他没有对所有的女人求过婚）{
	w = m列表上未被m求过婚的排名最高的女人；
	if(w是未匹配的状态){
		m，w结为一对；
	}
	else{
		w已经与某个男人mk结为一对了；
		if(w比起mk更喜欢m){
			m，w结为一对；
			mk变成未匹配状态；
		}
		else{
			w更喜欢现匹配对象mk；
			mk，w的匹配关系保持不变；
		}
	}
}
{% endcodeblock %}
来简单分析下这个算法，我们可以看出这个算法基本上就是遍历所有未被匹配的男人，每个男人都会从他的女性喜好列表中由高至低地向女人发起求婚，如果这个男人的求婚对象是未匹配状态，那么我们就会将他们列为一对；如果这个男人的求婚对象是已匹配状态，这时选择权交由女性来处理，如果她更喜欢这个向她求婚的男人，那么她就会抛弃当前的匹配对象，转而投入这个男人的怀抱，被抛弃的男人又会成为未匹配男人列表中的一员；如果她更喜欢当前的匹配对象，那么她就不会对这个男人的求婚行为做出任何理睬。

# Gale-Shapley算法的正确性
当我们仅仅浏览了这个算法的伪代码后，我们很难直观地看出这个算法的正确性，那么这个算法到底为什么能够100%产生1对1的，没有遗漏并且没有不稳定因子的配对结果呢？我们来看一下下面两个命题的简单证明

## Gale-Shapley算法永远生成1对1的，没有遗漏的匹配
假设这个算法生成了一个匹配结果，在这个结果中，有一个男人m是未匹配的状态。算法的终止也就意味着这个男人已经向他女性喜好列表中的所有女性都求过一次婚。从算法中我们可以发现，女人在受到第一次求婚后永远都是保持已匹配状态的，这也就意味着当前所有的女人都已经找到了结婚对象，那么矛盾点来了，我们一共只有n个男人与n个女人，n个女人是如何在还有一个未匹配男人的情况下都找到了结婚对象的呢？矛盾的发现也就意味着假设的不成立，那么也就证明这个算法生成的结果中是不可能有未匹配男人或者未匹配女人的存在。

## Gale-Shapley算法生成的匹配中，是没有不稳定因子的
假设这个算法生成了一个匹配结果，在这个结果中有这么两对匹配`m 与 wk`, `mk 与 w`，但是m比起wk更喜欢w，w比起mk也更喜欢m。那么问题就来了，m在这个算法中曾经向他心中的女神w求婚过吗？如果他求婚过，那么也就意味着他被w所拒绝了，w有一个更喜欢的人，我们称他为ml，那既然她在最后选择了mk，也就是意味着在她的心目中mk >= ml > m，矛盾点来了，她在一开始口口声声说比起mk更喜欢m？是不是矛盾了？如果m没有向他所谓的女神w求婚过，那么wk肯定比w在m心中的顺位高，那么m在一开始又说比起wk更喜欢w，是不是又产生矛盾了。综上所述，Gale-Shapley算法生成的匹配中，是没有不稳定因子的。

# 小结
通过以上的叙述，我们了解到了稳定匹配问题与Gale-Shapley算法的基本概念，其实Gale-Shapley算法还有更多的内容与延展，如果想要更进一步了解Gale-Shapley算法可以参考Algorithm Design(Jon Kleinberg & Eva Tardos)中更多的命题的证明，或者可以参考Geeksforgeeks中对[Gale-Shapley算法的实现](http://www.geeksforgeeks.org/stable-marriage-problem/)。