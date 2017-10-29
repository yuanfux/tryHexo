title: 哲学家就餐问题
---
哲学家就餐问题是大师Edsger Dijkstra提出的一个有趣的问题。理解这个问题对于解决多线程编程中的资源分配与避免死锁有着相当重要的作用。我们先来看一下这个问题的描述

<img width=40% src="http://otowvkh6b.bkt.clouddn.com/dp2.png">

有5个哲学家有一天想在一起吃个面，他们分好了面，找了一张圆桌坐了下来。正要开始吃面的时候，发现他们只有5个叉子。5个叉子讲道理吃个面也已经够了，但他们是哲学家，坚守的一个底线就是“1人必须要有2个叉子才能吃面”，于是他们决定共享叉子来吃面。从上图中我们可以看到，每个哲学家的左右手边都有2个叉子，那么这个问题的关键就是哲学家们该以什么样的方式来共享叉子才能让每一个哲学家都能吃上面呢？

# 死锁（deadlock）
在这个问题中，如果哲学家用叉的行为没有经过仔细的思考与设计，那么就很容易出现哲学家们没面吃的情况，也就是多线程编程中常常提到的死锁。当程序运行到达了死锁阶段，那么这个程序就如同卡住了一般，是没法继续正常运行的。我们先来看一下下面这个算法
{% codeblock lang:java line_number:false %}
//每个哲学家都按照下面的方式用叉
//1. 如果左手边的叉子没人用，拿起左手边的叉子；如果有人用，那么等待
//2. 如果右手边的叉子没人用，拿起右手边的叉子；如果有人用，那么等待
//3. 当拿到了2个叉子时，吃5分钟的面
//4. 将左手边的叉子放回去
//5. 将右手边的叉子放回去
//6. 再次从1.开始执行
{% endcodeblock %}
可能这个算法在乍看之下并没有什么问题，但仔细想想就会发现这个算法会导致死锁。当每个哲学家都拿着左手边的叉子在等待右手边叉子的时候，他们就再也吃不到面了。究其根本原因，就是因为哲学家们在这个算法中表现得过于“贪婪”了，他们在拿着左叉的同时又在等待右叉，而在这个阶段，左叉这个资源是没有得到任何利用的。下面我们来看一下这个问题的两种经典答案与实际代码。

# 全拿或者全不拿
来看一下这种改进后的算法
{% codeblock lang:java line_number:false %}
//每个哲学家都按照下面的方式用叉
//1. 检查左右两边的叉子是否可用，如果任意一个叉子不可用，那么等待
//2. 拿起左边的叉子
//3. 拿起右边的叉子
//4. 吃5分钟的面
//5. 将左手边的叉子放回去
//6. 将右手边的叉子放回去
//7. 再次从1.开始执行
{% endcodeblock %}
这个算法对原先导致死锁算法的改进就是，每个哲学家只会在能同时拿2个叉子的时候出手，这样就保证了资源不会被拿在手里白白浪费掉。我们来看一下java中对这个算法的实现
{% codeblock lang:java %}
 class Fork(){ //叉子类
	private Lock lock; //用Lock来模拟叉子空闲与被使用的状态
	public Fork(){
		lock = new ReentrantLock();
	}
	public boolean pickUp(){
		return lock.tryLock(); //尝试拿起叉子给Lock上锁
	}
	public void putDown(){
		lock.unlock(); //放下叉子等同于对Lock解锁
	}
 }

 class Philosopher extends Thread { //哲学家类继承线程类
	private int total = 5; //假定一碗面一共需要吃5次
	private Fork left, right; //每个哲学家有左右两个叉子
	public Philosopher(Fork left, Fork right){
		this.left = left;
		this.right = right;
	}
	public void eat(){
		if(pickUp()){ //两叉可用并拿起两叉
			eat5Min(); //吃5分钟
			putDown(); //放下两叉
		}
	}
	public boolean pickUp(){
		if(!left.pickUp()){ //不能拿左叉
			return false;
		}
		if(!right.pickUp()){ //不能拿右叉
			left.putDown();
			return false;
		}
		return true; //两个叉都能拿，并已经拿到
	}
	public void eat5Min(){ /*模拟吃5分钟*/ }
	public void putDown(){
		left.putDown();
		right.putDown();
	}
	public void run(){
		for(int i = 0 ; i < total; i++){
			eat(); //一共调用eat()5次
		}
	}
 }
{% endcodeblock %}
以上就是在java中对这个算法的一个简单实现，细心的读者可能已经发现了这个算法依然存在一个可能的问题。如果有那么一瞬间，所有的叉子都可用，并且所有的哲学家同时执行了pickUp()函数内的位于第28行的left.pickUp()，那么所有的哲学家是依然有可能会同时拿起左手边的叉子并造成死锁的。虽然这个概率极小但也是有可能发生的，那么我们再来看另外一个算法。

# Dijkstra的算法
大神Dijkstra在提出这个问题的时候就已经对这个问题作出了解答。Dijkstra的思路是将每一个叉子都从1-5标上相应的序号，再按照下面的算法来执行
{% codeblock lang:java line_number:false %}
//每个哲学家都按照下面的方式用叉
//1. 拿起左右两边叉子中序号较小的一个，如果不能拿起就等待
//2. 拿起左右两边叉子中序号较大的一个，如果不能拿起就等待
//3. 吃5分钟的面
//4. 将序号小的叉子放回去
//5. 将序号大的叉子放回去
//6. 再次从1.开始执行
{% endcodeblock %}
我们来分析一下这个算法，如果四个哲学家都同时拿起了序号较小的叉子，那么剩下在桌面上的一个序号最大的叉子是不可能被第五个哲学家拿起的，Dijkstra通过给叉子标序号的方法成功地避免了5个哲学家同时拿起一把叉子的情况。我们来看一下Java代码实例（我们规定逆时针标注叉子序号，也就是4个哲学家的左叉序号小于右叉，而1个哲学家的左叉序号大于右叉）
{% codeblock lang:java %}
 class Fork(){ //叉子类
	private Lock lock; //用Lock来模拟叉子空闲与被使用的状态
	private int number; //叉子的序号
	public Fork(int n){
		lock = new ReentrantLock();
		number = n;
	}
	public void pickUp(){
		 lock.Lock(); //尝试拿起叉子给Lock上锁
	}
	public void putDown(){
		lock.unlock(); //放下叉子等同于对Lock解锁
	}
	public int getNumber(){
		return number; //返回此叉子的序号
	}
 }

 class Philosopher extends Thread { //哲学家类继承线程类
	private int total = 5; //假定一碗面一共需要吃5次
	private Fork small, big; //每个哲学家有低序号,高序号两个叉子

	public Philosopher(Fork left, Fork right){
		//将左右叉转化为低序号叉与高序号叉
		if(left.getNumber() < right.getNumber()){
			this.small = left;
			this.big = right;
		}
		else{
			this.small = right;
			this.big = left;
		}
	}
	public void eat(){
			pickUp();  //尝试拿起两叉
			eat5Min(); //吃5分钟
			putDown(); //放下两叉
	}
	public void pickUp(){
			small.pickUp(); //先拿起序号小的叉子
			big.pickUp(); //再拿起序号大的叉子
	}
	public void eat5Min(){ /*模拟吃5分钟*/ }
	public void putDown(){
		small.putDown(); //注意这里放下叉子的顺序无关紧要
		big.putDown();
	}
	public void run(){
		for(int i = 0 ; i < total; i++){
			eat(); //一共调用eat()5次
		}
	}
 }
{% endcodeblock %}
通过这种给叉子标注序号的方法，我们就彻底避免了死锁的发生。

# 小结
哲学家用餐问题是一个所有多线程编程问题的缩影，当你在多线程编程问题中遭遇死锁的情况时，请回来想想这个问题，这个问题的解可能会对你解决死锁问题会有一定的帮助。 