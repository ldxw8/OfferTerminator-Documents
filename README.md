# OfferTerminator Documents

 > 偏向于软件工程师的面试资料整理。

## 推荐资料
- 📖 | 书籍 | [何海涛. 剑指 Offer [M]. 第2版. 电子工业出版社, 2017](https://book.douban.com/subject/27008702/)
- 📖 | 书籍 | [《编程之美》小组. 编程之美 [M]. 第1版. 电子工业出版社, 2018](https://book.douban.com/subject/30351275/)
- 📝 | 文章 | [ApacheCN. 简历指南 + LeetCode + Kaggle [OL]. github.com](https://github.com/apachecn/Interview) (👍)
	- `笔试`：[ApacheCN. 面试必备算法题目 [OL]. github.com](https://github.com/apachecn/Interview/tree/master/docs/Algorithm)
	- `面试`：[0voice. BAT 等技术面试题目/答案/专家出题人分析汇总 [OL]. github.com](https://github.com/0voice/interview_internal_reference)
- 📝 | 文章 | [CyC2018. CS-Notes [OL]. github.com](https://github.com/CyC2018/CS-Notes) (👍)

	>  技术面试必备基础知识、Leetcode 题解、Java、C++、Python、后端面试、操作系统、计算机网络、系统设计等。
	
- 🌝 | 面试 | [小土刀. 小土刀的面试刷题笔记. wdxtub.com](https://wdxtub.com/interview/index.html) (👍)

- ⌨️ | 算法 | [MisterBooo. LeetCodeAnimation [OL]. github.com](https://github.com/MisterBooo/LeetCodeAnimation)

	> 本开源项目是用动画的形式呈现 LeetCode 的解题思路。
	
- ⌨️ | 算法 | [Blankj. Awesome-Java-Leetcode [OL]. github.com](https://github.com/Blankj/awesome-java-leetcode)

	> 实现语言是 Java，题库按简单/中等/困难分类，且解释足够详细。

- ⌨️ | 算法 | [azl397985856. Leetcode-Solutions. github.com](https://github.com/azl397985856/leetcode)

	> 实现语言是 JavaScript，题库按简单/中等/困难分类，题目按照算法思想、数据结构归档分类，解题思路图示等。

## 在线编程
- [牛客 -- 剑指Offer -- 编程题](https://www.nowcoder.com/ta/coding-interviews?page=3)
- [力扣 -- Leetcode -- 算法 + 数据库](https://leetcode-cn.com/problemset/all/)
- [AppStore -- Daily Code -- Leetcode 客户端](https://apps.apple.com/cn/app/leetcode-%E6%AF%8F%E6%97%A5%E5%8A%9B%E6%89%A3/id1475606624)

## 阅读工具
- 本项目的笔记资料均由 [Markdown](https://zh.wikipedia.org/wiki/Markdown) 进行排版、编写，注重知识模块间的联系性，以及获得良好阅读体验，推荐使用 [Typera](https://www.typora.io/) 工具进行阅读。
- 本项目支持 Gitbook 生成离线电子书 (pdf、epub、html 等)。

## 核心内容
### 读书笔记
> 命名格式：书籍 -- 主题描述 -- 立项时间

#### 通用岗
- [剑指 Offer -- 面经与解题启发性笔记 -- 2019.08.29](doc/Coding-Interviews-Questions-Analysis-and-Solutions.md)

- [CyC2018 技术面试必备基础知识 -- Java -- 知识点总结](https://cyc2018.github.io/CS-Notes/#/README?id=%e2%98%95%ef%b8%8f-java)
	- [Java 基础 -- 2019.10.07](doc/CyC2018-CS-Notes-Java-Foundation.md)
	- [Java 容器 -- 2019.09.21](doc/CyC2018-CS-Notes-Java-Container.md)
	- [Java 并发 -- 2019.09.25](doc/CyC2018-CS-Notes-Java-Concurrency.md)
	- [Java JVM -- 2019.10.01](doc/CyC2018-CS-Notes-Java-JVM.md)
	- [Java I / O -- 2019.10.07](doc/CyC2018-CS-Notes-Java-IO.md)

- [CyC2018 技术面试必备基础知识 -- 数据库篇 -- 知识点总结 -- 2019.09.01](doc/CyC2018-CS-Notes-Database.md)
- [CyC2018 技术面试必备基础知识 -- 操作系统 -- 知识点总结 -- 2019.09.04](doc/CyC2018-CS-Notes-OS.md)
- [CyC2018 技术面试必备基础知识 -- 计算机网络 -- 知识点总结 -- 2019.09.10](doc/CyC2018-CS-Notes-Network.md)
- [Kofe 技术面试必备基础知识 -- 数据结构 -- 2019.09.18](doc/Kofe-CS-Notes-DataStruct.md)

#### 客户端
- [Kofe 技术面试必备基础知识 -- Android -- 2019.09.22](doc/Kofe-CS-Notes-Android.md)

#### 算法岗

### 面试经验
> 命名格式：面试企业 -- 职业 -- 面试时间 -- 面试人 ( 个人 / 其他人 )

- [百度|阿里|腾讯|华为|字节跳动|网易游戏|顺丰科技 -- 研发 -- CyC2018](https://www.nowcoder.com/discuss/137593)

### 笔试算法
#### 项目使用说明

|![开源项目结构说明](img/OfferTerminator-documents_1-1.png)|
|:---:|
|图 1-1 开源项目结构说明|



- 使用集成开发环境 `IntelliJ Idea` 构建工程项目，并且每一题以单独 `Module` 立项。
- 每一题目以 `类` 进行封装，同一题目的解题思路以 `函数` 为实现载体。
- 项目命名规范：`来源-序号-题目名称` / `来源-题目名称`。

	> 例如：JzOffer-05-ReplaceSpaces、JzOffer-07-RebuildBinaryTree、Leetcode-TwoSum 等。

- 函数命名规范：`驼峰式命名法`
	
	> 例如：pulbic String rebuildBinaryTree(StringBuffer src){...}

- 设计测试用例：便于解题、复习的用途，使用单元测试框架 Junit 编写测试用例。
	- 有效等价类：根据取值范围、数据类型、限制条件或规则等，求得合理的、有意义的输入数据构成的集合。
	- 无效等价类：刚好与有效等价类的概念相反。
	- 边界值分析法：作为对等价类划分法的补充，通常其测试用例来自等价类的边界。

#### 解题源码整理
- [ldxw8. OfferTerminator-Solutions. github.com](https://github.com/ldxw8/OfferTerminator-Solutions)

	> 题目集中于剑指 Offer、Leetcode 的经典题目。且参考另一位兄弟的开源项目，可直接复用他的题目命名和题目描述。 [Jchanghong. CodingInterviewChinese2. github.com](https://github.com/jchanghong/CodingInterviewChinese2)
	
	- JzOffer-03-DuplicationInArray：数组中重复的数字
	- JzOffer-20-NumericString：表示数值的字符串
	- JzOffer-67-Str2Int：字符串转整数