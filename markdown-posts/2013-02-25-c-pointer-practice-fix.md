---
layout: post
title: C语言指针学习 代码修订 感谢大家指导
date: 2013-02-25 23:04
author: onecoder
comments: true
categories: [c++ primer, C\C++, 指针]
---
<p>
	感谢新浪微博上<a href="http://weibo.com/2781971343">@薛非_品悟C</a> 朋友指出的问题。</p>
<blockquote>
	<p>
		(*fn_p)(&amp;a,&amp;b); 可以直接写作fn_p(&amp;a,&amp;b); (2月22日 11:40)</p>
	<p>
		int (*fn_p)()=swap;应写为int (*fn_p)(int *,int *)=swap;</p>
	<p>
		sum显然没有正确地初始化为0；sum/n这个表达式类型不对，应为(float)sum/(float)n</p>
</blockquote>
<p>
	<a href="http://www.coderli.com">OneCoder</a>马上对代码进行了修改和验证。代码修改如下：</p>
<pre class="brush:cpp;first-line:1;pad-line-numbers:true;highlight:null;collapse:false;">
//定义函数指针，用函数指针的方式调用函数,函数名即函数的首地址
int (*fn_p)(int *, int *)=swap;

// 再次交换a,b
(*fn_p)(&amp;a,&amp;b);

printf(&quot;swap again, a=%d,b=%d\n&quot;,a,b);
// 函数指针调用的另外一种方式

fn_p(&amp;a,&amp;b);
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
  int sum=0,i;

  for(i=0;i&lt;n;i++){
   sum+=*ap++;

  }
  return (float)sum/(float)n;

}

</pre>
<p>
	这里<a href="http://www.coderli.com">OneCoder</a>也查阅了一下书籍，关于函数指针调用：《C Primer Plus》P414页有如下说明：</p>
<blockquote>
	<p>
		（原书代码略）第一种方法，因为pf指向ToUpper函数，＊pf就是ToUpper函数，因此表达式(*pf)(mis)与ToUpper(mis)一样。从ToUpper和pf的声明中就能看出ToUpper和(*pf)是等价的。第二种方法：因为函数名是一个指针，可以互换地使用指针和函数名，因此pf(mis)和ToLower(mis)一样。从pf的赋值语句中就能看出pf和ToLower是等价的。历史上，贝尔实验室的C和UNIX的开发者采用第一种观点，而Berkeley的UNIX的扩展者采用第二种观点。K&amp;R C不允许第二种形式。但是为了保持与现有代码的兼容性。ANSI C把这二者作为等价的形势全部接受。</p>
</blockquote>
<p>
	再次感谢大家不吝指教，使我获益良多。多谢大家。欢迎大家多多给我留言指导。</p>

