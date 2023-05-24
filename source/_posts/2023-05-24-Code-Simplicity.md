---
title: Code Simplicity
date: 2023-05-24 22:33:29
categories: "2023-05"
tags: [读书]
---

看了一本书[Code Simplicity](https://www.codesimplicity.com/book.pdf)，准确点来说应该是一本小册子，全书也就80来页而已。里面有些观点在这里摘录一下。

> There are, thankfully, various ways to balance these two needs of writing features and handling complexity. One of the best ways is to do your redesigning purely with the goal of making some specific feature easier to implement, and then implementing that feature. That way, you switch regularly between redesign work and feature work. This also helps your new design fit your needs well, because you’re creating it with a real use in mind. Your system will slowly get less complex over time, and you will still keep pace with your users’ needs. You can even do this for bugs—if you see that some bug would be easier to fix with a different design, redesign the code before fixing it.

这也是小步迭代的方向，每次写代码都要设计，需要思考过，而并不是仅仅把功能实现就可以。

> Many difficult design problems can be solved by simply drawing or writing them out on paper. 

我自己也逐渐在养成一种习惯，在开始写代吗时先用文字的方式在自己的logseq 上写下来要做什么，具体步骤有哪些一步一步列出来，也会把具体的需求或者问题描述一边。一旦我这么做了后
开始写代码时就会思路清晰很多。

> A technology’s survival potential is the likelihood that it will continue to be maintained. If you get stuck with a library or some dependency that becomes obsolete and un- maintained, you’re really in for some trouble 

曾经听到一个观点：一个东西/技术存活了多久，它很大可能也就会再存活多久，老技术更能说明它可靠。

> Thankfully, there are three factors you can look at to determine if a technology is “bad” before you even start using it: **survival potential, interoperability, and attention to quality**.

这也许是选择技术框架时的一个思路。这门技术是否有比较强的持续存下去的可能，这里说到的interoperability，我理解来说是容易替换也就是说符合标准的而不是定制化的，在面向接口编程时往往可以不关注实现，
关注质量这个的话，可能要关注它的社区，如果能看到源代码那就要看它的源代码。

> Complexity builds on complexity—it’s not just a linear thing. That is, you can’t make assumptions like: “We have 10 features, so adding 1 more will only add 10 percent more time.” In fact, that one new feature will have to be coordinated with all 10 of your existing features. So, if it takes 10 hours of coding time to implement the feature itself, it may well take another 10 hours of coding time to make the 10 existing features all interact properly with the new feature. The more features there are, the higher the cost of adding a feature gets. You can minimize this problem by having an excellent software design, but there will still always be some slight extra cost for every new feature. 


这个观点在人曰神话中出现过了，这里作者也提到了这本书。尤其是在复杂都比较高的系统中，往往不能简单地这样计算。

> On the other side of things, when you’ve designed well, there’s often not a whole lot of credit that comes your way. Catastrophic failures in design are big and noticeable, whereas small increments of work toward a good design are invisible to people who aren’t intimately connected with the code. This can make being a designer a difficult job. Handling a big failure gets you a lot of thanks, but preventing one from ever hap- pening...well, nobody’s likely to notice. 

程序员工作悖论，架构做的好，代码写得好，没有出任何故障，可用率达到6个9，最后却被优化了。而天天救火的人反倒获得了组织认可，因为他天天出现在领导面前。

> The real purpose of comments is to explain why you did something, when the reason isn’t obvious. If you don’t explain that, other programmers may be confused, and when they go to change your code they might remove important parts of it if those parts don’t seem to have a reason to exist

代码注释的功能不是讲述这段代码做什么，而是讲述为什么这么做。做什么的说明应该是通过代码本身去表达。

> There are situations in programming where it doesn’t matter how you do things, as long as you always do them that way. Theoretically, you could write your code in some crazy complex way, but as long as you were consistent with it, people would learn how to read it. (Of course, it’s better to be consistent and simple, but if you can’t be totally simple, at least be consistent.)  
> Code that isn’t consistent is harder for a programmer to understand and read.

一致性，先从变量命名开始，尤其是API 中，用了驼峰就全部都是驼峰，别一会下划线一会驼峰。

> However, just because a user reports something doesn’t mean it’s a problem. Some- times the user will simply not have realized that your program had some feature already, and so asked you to implement something else unnecessarily 
> Never “fix” anything unless it’s a problem, and you have evidence showing that the problem really exists. It’s important to have evidence of problems before you address them. Otherwise, you might be developing features that don’t solve anybody’s problem, or you might be “fixing” things that aren’t broken. 

不要着急去“Fix bugs”，先确认清楚情况。尤其看到一些年轻开发者，一旦别人提了个问题，马上就自己吭哧就写代码了。

> This is actually a combination of two methods: one called “incremental development” and another called “incremental design.” Incremental development is a method of building up a whole system by doing work in small pieces. In our list, each step that started with “Implement” was part of the incremental development process. Incre- mental design is similarly a method of creating and improving the system’s design in small increments. Each step that started with “Fix up the system’s design” or “Plan” was part of the incremental design process. 
> In general, when your design makes things more complex instead of simplifying things, you’re overengineering. 
> Code should be designed based on what you know now, not on what you think will happen in the future. 
>	1. Make too many assumptions about the future. 2. Write code without enough design 

永远不要假想以后会怎么样。

> It’s a good rule, but it’s misnamed. You actually might need the code in the future, but since you can’t predict the future you don’t know how the code needs to work yet. If you write it now, before you need it, you’re going to have to redesign it for your real needs once you actually start using it 
> There are three broad mistakes that software designers make when attempting to cope with the Law of Change, listed here in order of how common they are: 
> 1. Writing code that isn’t needed 
> 2. Not making the code easy to change 
> 3. Being too generic 


> Often, designing a system that will have decreasing maintenance effort requires a significantly larger effort of implementation—quite a bit more design work and planning are required. However, remember that the effort of implementation is nearly always an insignificant factor in making design decisions, and should mostly be ignored. 
> And in fact, nearly all decisions in software design reduce entirely to measuring the future value of a change versus its effort of maintenance. There are situations in which the present value and the implementation effort are large enough to be significant in a decision, but they are extremely rare. In general, software systems are maintained for so long that the value now and the effort of implementation are guaranteed to become insignificant in almost all cases when compared to the long-term future value and effort of maintenance. 
> Features that have no users have no immediate value. These could include features that users can’t find, features that are too difficult to use, or features that simply don’t help anybody. They may have value in the future, but they have no value now. 

重长远的角度看，如果一个软件会被长期使用，那么维护成本往往比当初开始开发时的成本要高，那么可维护行就很重要了。


> Probability of value and potential value Value is actually composed of two factors: the probability of value (how likely it is that this change will help a user), and the potential value (how much this change will help a user during those times when it does help that person). 

这个说法跟我自己的理解/感受很像，产品出了事故大家往往评价产生了多大影响（Probability），但是鲜少去关心每个受影响的用户会受到多大的影响。在互联网公司呆久了往往看到一些事故分析都是关心公司有没有受到资产损失，
有多少用户受到了影响，这些都是大数，可能每个人的影响不一样的。在用户投诉时，客服往往又不会有什么权力去安慰（赔偿）用户。


> The difference between a bad programmer and a good programmer is understanding.   
> The average software system becomes large enough that no human being could hope to hold all of its code in their mind at once. This isn’t good or bad, it’s just a fact 

作者在书中提到软件的目的就是帮助人，你是否同意呢？
