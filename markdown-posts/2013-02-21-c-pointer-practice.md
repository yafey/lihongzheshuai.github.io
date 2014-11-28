---
layout: post
title: C语言指针学习 代码练习
date: 2013-02-21 23:08
author: onecoder
comments: true
categories: [C, C\C++, pointer, 指针]
---
<p>
	近两天在快读的浏览的谭浩强的《C语言教程》，<a href="http://www.coderli.com">OneCoder</a>并不想对C/C++做多深入的学习，仅仅想作为一种了解和上手，并把其作为算法学习的基础语言而已（真正想精通并且不能更不想放弃的还是Java）。所以，在阅读上感觉相似的部分快速带过，第十章的指针部门，是Java中没有的部分，作为重点的了解，遂作此笔记。C新手，见笑：）。</p>
<p>
	老习惯，还是将笔记记录穿插记录在代码注释中，上代码：</p>
<pre class="brush:cpp;first-line:1;pad-line-numbers:true;highlight:null;collapse:false;">
//C语言指针学习代码
//@author:OneCoder(http://www.coderli.com)
#include&lt;stdio.h&gt;
//需要在使用函数前，声明该函数。
//除非改函数默认返回int类型, 比如swap函数
float avg(int *ap, int n);
int main(){
	int i=3;
	printf(&quot;%d\n&quot;,i);
	//i_point为指向i的地址的指针,&amp;符号为取地址符号
	int *i_point=&amp;i;
	//相当于给i赋值
	*i_point=5;
	/*通过指针访问它所指向的一个变量是以间接访问的形式进行的,所以比直接访问一个变
	 *量要费时间,而且不直观，但由于指针是变量,我们可以通过改变它们的指向,以间接访问不同的变量
	 *,这给程序员带来灵活性,也使程序代码编写得更为简洁和有效。
	 */
	printf(&quot;%d\n&quot;,i);
	int a=1,b=2;
	printf(&quot;a=%d,b=%d\n&quot;,a,b);
	swap(&amp;a,&amp;b);
	printf(&quot;swap, a=%d,b=%d\n&quot;,a,b);
	//将数组首地址赋给指针变量，直接使用数组名或者首元素地址。
	int int_array[5]={2,4,6,8,10};
	int *array_p=int_array;//=&amp;int_array[0];
	//数组指针，移动一个整数位，指向数组中后面的元素
	printf(&quot;the third member in array is: %d\n&quot;, *(array_p + 2));
	//定义函数指针，用函数指针的方式调用函数,函数名即函数的首地址
	int (*fn_p)()=swap;
	// 再次交换a,b
	(*fn_p)(&amp;a,&amp;b);
	printf(&quot;swap again, a=%d,b=%d\n&quot;,a,b);
	//求数组平均值
	printf(&quot;average: %f\n&quot;, avg(int_array,5));
	//二维数组指针变量定义，使用练习
	int ab[2][2] = {{1,2},{3,4}};
	int (*tp)[2]=ab;
	printf(&quot;(0,0)=%d\n(0,1)=%d\n(1,0)=%d\n(1,1)=%d\n&quot;,*(*tp),*(*tp+1),tp[1][0],*(tp[1]+1));
	//字符串指针的使用练习
	char *str=&quot;I am OneCoder&quot;;
	str+=5;
	printf(&quot;%s\n&quot;,str);
	//指针数组和指向指针的指针小练
	int l[5]={1,2,3,4,5};
	//练习多种写法
	int *lp[5]={l,l+1,l+2,l+3,&amp;l[4]};
	int **lpp=lp;
	printf(&quot;point_point:%d\n&quot;,**(lpp+3));
}

//利用指针变量实现两个数值交换的功能
swap(int *n1, int *n2) {
	int temp=*n1;
	*n1=*n2;
	*n2=temp;	
}

//利用数组指针实现求数据的平均值
float avg(int *ap, int n) {
  int sum,i;
  for(i=0;i&lt;n;i++){
  	sum+=*ap++;
  }
  return sum/n;
}

</pre>
<p>
	书中还有关于结构体和位运算的部分需要继续学习，不过<a href="http://www.coderli.com">OneCoder</a>打算先通过实现几个简单的算法，巩固一下现在了解到的C语法，同时也算变换一下思维，为今后的算法学习做铺垫。</p>

