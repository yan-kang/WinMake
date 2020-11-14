<h2>Windows下使用gcc/g++及make工具</h2>
<h3>1.前言</h3>

Windows下有很多好用的集成IDE，但用起来总觉得不得劲，于是便自己打造了这一套工具。  
编辑器：Sublime Text。这是一个轻量，高颜值，跨平台的编辑器，插件多，可自定义程度高。可胜任近乎所有编辑任务，同时能编译近乎所有编程语言程序，当然前提要安装编译环境。  
编译工具：tdm-gcc/g++,mingw32-make  

本次只讲gcc工具的安装及make的基本使用，Sublime Text的安装及配置请自行查阅相关文章。  

---

<h3>2.安装及配置</h3>

<h4>2.1.gcc/g++的安装</h4>

1.gcc编译器下载，[TDM-GCC](https://jmeubank.github.io/tdm-gcc/),打开根据自己电脑的类型选择合适的程序,此处我选择tdm64-gcc。  
2.下载完成后，以管理员方式打开安装程序，点击Create后选择相应版本,选择安装路径(不带中文，简易所有应用安装路径都不带中文，会有莫名其妙的错误)。  
3.最后点击Install进行安装，安装完成后，可以在自己设置的安装路径下有这么一个文件夹“TDM-GCC-64”，设置系统环境变量，使其指向上述路径的bin目录  
4.打开cmd输入gcc -v查看，安装配置正确会输出gcc版本信息，否则会提示命令不存在。  
<h4>2.2.make命令的使用</h4>

输入mingw32-make -v查看make信息，相对于Linux的make不同，这里是mingw32-make，在cmd里用cd命令切换在Makeflie文件所在目录，命令行输入mingw32-make回车运行，就行执行Makefile文件，进行自动化编译。此处可将gcc安装目录下的ming32w-make重命名为make即可使用make命令而不需要输入mingw32-make。  

---
<h3>3.makefile的编写</h3>

<h4>3.1.makefile的规则</h4>

	target ... : prerequisites ...  
	    command  
	    ...  
	    ...  
	target  
可以是一个object file（目标文件），也可以是一个执行文件，还可以是一个标签（label）。  
prerequisites  
生成该target所依赖的文件和/或target  
command  
该target要执行的命令（任意的shell命令），这是一个文件的依赖关系，也就是说，target这一个或多个的目标文件依赖于prerequisites中的文件，其生成规则定义在command中。  

<h4>3.2.一个例子</h4>

1.创建一个main.cpp,test.cpp,test.h文件以及一个makefile文件，makefile文件内容如下:    

	test:main.o test.o  
		g++ main.o test.o -o test  
	main.o:main.cpp test.h  
		g++ main.cpp -c -o main.o  
	test.o:test.cpp test.h  
		g++ test.cpp -c -o test.o  
	.PHONY : clean  
	clean:  
		del *.o test.exe -rf  
2.打开cmd，在当前目录输入make即可自动编译链接，输入make clean自动删除编译生成的文件：所有.o文件以及test.exe，需要注意的是Linux和Unix下用的是rm但Windows cmd不支持，需用del。  
在这个makefile中，目标文件（target）包含：执行文件test和中间目标文件（ *.o ），依赖文件（prerequisites）就是冒号后面的那些 .c 文件和 .h 文件。每一个 .o 文件都有一组依赖文件，而这些 .o 文件又是执行文件 test 的依赖文件。依赖关系的实质就是说明了目标文件是由哪些文件生成的，换言之，目标文件是哪些文件更新的。  
在定义好依赖关系后，后续的那一行定义了如何生成目标文件的操作系统命令，一定要以一个 Tab 键作为开头。记住，make并不管命令是怎么工作的，他只管执行所定义的命令。make会比较targets文件和prerequisites文件的修改日期，如果prerequisites文件的日期要比targets文件的日期要新，或者target不存在的话，那么，make就会执行后续定义的命令。  
这里要说明一点的是， clean 不是一个文件，它只不过是一个动作名字，有点像c语言中的label一样，其冒号后什么也没有，那么，make就不会自动去找它的依赖性，也就不会自动执行其后所定义的命令。要执行其后的命令，就要在make命令后明显得指出这个label的名字。这样的方法非常有用，我们可以在一个makefile中定义不用的编译或是和编译无关的命令，比如程序的打包，程序的备份，等等。  

<h4>3.3.make如何工作</h4>

在默认的方式下，也就是我们只输入 make 命令。那么，  
1.make会在当前目录下找名字叫“Makefile”或“makefile”的文件。  
2.如果找到，它会找文件中的第一个目标文件（target），在上面的例子中，他会找到“test”这个文件，并把这个文件作为最终的目标文件。  
3.如果test文件不存在，或是test所依赖的后面的.o文件的文件修改时间要比test这个文件新，那么，他就会执行后面所定义的命令来生成 test这个文件。  
4.如果test所依赖的.o文件也不存在，那么make会在当前文件中找目标为.o文件的依赖性，如果找到则再根据那一个规则生成.o文件。（这有点像一个堆栈的过程)。  
5.当然，你的Cpp文件和H文件是存在的啦，于是make会生成.o文件，然后再用 .o 文件生成make的终极任务，也就是执行文件test了。

<h4>3.4.使用变量及自动推导</h4>

z在上面的例子中我们可以看到.o文件的字符串被重复了两次，如果我们的工程需要加入一个新的.o文件，那么我们需要在两个地方加（应该是三个地方，还有一个地方在clean中）。当然，我们的makefile并不复杂，所以在两个地方加也不累，但如果makefile变得复杂，那么我们就有可能会忘掉一个需要加入的地方，而导致编译失败。所以，为了makefile的易维护，在makefile中我们可以使用变量。makefile的变量也就是一个字符串，理解成C语言中的宏可能会更好。  
比如，我们声明一个变量，叫 objects ， OBJECTS ， objs ， OBJS ，obj或是OBJ反正不管什么啦，只要能够表示obj文件就行了。  
于是，我们就可以很方便地在我们的makefile中以$(objects)的方式来使用这个变量了。  
GNU的make很强大，它可以自动推导文件以及文件依赖关系后面的命令，于是我们就没必要去在每一个 .o 文件后都写上类似的命令，因为，我们的make会自动识别，并自己推导命令。  
只要make看到一个.o文件，它就会自动的把.cpp文件加在依赖关系中，如果make找到一个 whatever.o ，那么 whatever.cpp 就会是 whatever.o 的依赖文件。并且 g++ -c whatever.cpp也会被推导出来，于是，我们的makefile再也不用写得这么复杂。  
改写上述例子：  

	objects=main.o test.o  
	edit:$(objects)  
		g++ -o test $(objects)  
	$(objects):test.h  
	.PHONY : clean  
	clean:  
		del test.exe $(objects)  
需要了解的是，其中g++ -o test &(objects)可写成g++ $(objects) -o test。  

<h4>3.5.清空目标文件规则</h4>

每个Makefile中都应该写一个清空目标文件（.o和执行文件）的规则，这不仅便于重编译，也很利于保持文件的清洁。一般写为：  

	clean:  
		del test.exe $(objects)  
稳健的写法：  

	.PHONY : clean  
	clean :  
		del test.exe $(objects)  
.PHONY 表示 clean 是一个“伪目标”。  

<h4>3.6.vpath及VPATH的使用</h4>

在一些大的工程中，有大量的源文件，我们通常的做法是把这许多的源文件分类，并存放在不同的目录中。所以，当make需要去找寻文件的依赖关系时，你可以在文件前加上路径，但最好的方法是把一个路径告诉make，让make在自动去找。  
Makefile文件中的特殊变量“VPATH”就是完成这个功能的，如果没有指明这个变量，make只会在当前的目录中去找寻依赖文件和目标文件。如果定义了这个变量，那么，make就会在当当前目录找不到的情况下，到所指定的目录中去找寻文件了。  

    VPATH = src:../headers

上面的的定义指定两个目录，“src”和“../headers”，make会按照这个顺序进行搜索。目录由“冒号”分隔。（当然，当前目录永远是最高优先搜索的地方）。  
另一个设置文件搜索路径的方法是使用make的“vpath”关键字（注意，它是全小写的）， 这不是变量，这是一个make的关键字，这和上面提到的那个VPATH变量很类似，但是它更为灵活。它可以指定不同的文件在不同的搜索目录中。这是一个很 灵活的功能。它的使用方法有三种：

    vpath <pattern> <directories>

为符合模式&lt;pattern&gt;的文件指定搜索目录&lt;directories&gt;。

    vpath <pattern>

清除符合模式&lt;pattern&gt;的文件的搜索目录。

    vpath

清除所有已被设置好了的文件搜索目录。  
vapth使用方法中的&lt;pattern&gt;需要包含“%”字符。“%”的意思是匹 配零或若干字符，例如，“%.h”表示所有以“.h”结尾的文件。&lt;pattern&gt;指定了要搜索的文件集， 而&lt;directories&gt;则指定了&lt;pattern&gt;的文件集的搜索的目录。例如：

    vpath %.h ../headers

该语句表示，要求make在“../headers”目录下搜索所有以“.h”结尾的文件。（如果某文件在当前目录没有找到的话）我们可以连续地使用vpath语句，以指定不同搜索策略。如果连续的vpath语句中出现了相同的&lt;pattern&gt;，或是被重复了的&lt;pattern&gt;，那么，make会按照vpath语句的先后顺序来执行搜索。如：

    vpath %.c foo
    vpath %   blish
    vpath %.c bar

其表示“.c”结尾的文件，先在“foo”目录，然后是“blish”，最后是“bar”目录。

    vpath %.c foo:bar
    vpath %   blish

而上面的语句则表示“.c”结尾的文件，先在“foo”目录，然后是“bar”目录，最后才是“blish”目录。