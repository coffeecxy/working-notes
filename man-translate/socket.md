# socket

## htonl

    #include <arpa/inet.h>
    uint32_t htonl(uint32_t hostlong);
    uint16_t htons(uint16_t hostshort);
    uint32_t ntohl(uint32_t netlong);
    uint16_t ntohs(uint16_t netshort);

进行byte order的转换. 因为在host上面的数据和网络传输的数据的大小端可能是不相同的,所以要对其进行大小端的转换.

hton表示从host转到network,l表示转换的是一个32位的数据,s表示转换一个16位的数据.

对于ntoh,就是类似的了.

    #include <arpa/inet.h>
    #include <stdio.h>
    int main ()
    {

    	uint32_t a = 0xfafbfcfd;
    	uint32_t b = htonl(a);
    	printf("a = %x,b =%x\n",a,b);

    	return 0;
    }

上面的代码,输出为

    ○ ./testhton
    a = fafbfcfd,b =fdfcfbfa

因为在x86中,使用的小端,而在网络中,使用的是大端.

## time

在linux中,对于一个时间的表示,有两种数据结构,`time_t`和`struct tm`.

time_t实际上是一个64的数,表示从 `1970.01.01 0:0:0 utc+0` 开始的秒数.被叫做是日历时间.

tm就是一个数据结构了,其中包含了年月日,时分秒这些所有的信息.被叫做broken down time,也就是细分时间.

    #include <time.h>
    time_t time(time_t *t);

得到当前的时间.这值会被返回,同时如果t不是Null,还会被放到t中.

    char *asctime(const struct tm *tm);
    char *asctime_r(const struct tm *tm, char *buf);

    char *ctime(const time_t *timep);
    char *ctime_r(const time_t *timep, char *buf);

    struct tm *gmtime(const time_t *timep);
    struct tm *gmtime_r(const time_t *timep, struct tm *result);

    struct tm *localtime(const time_t *timep);
    struct tm *localtime_r(const time_t *timep, struct tm *result);

    time_t mktime(struct tm *tm);

这几个函数是对时间进行类型转换的函数. 前面四个是将时间值转成字符串,也就是得到一个时间的字符串表示. 中间四个是将日历时间转为细分时间. 最后一个是将细分时间转为日历时间.

asctime将一个细分时间转为一个字符串,这个字符串是比较号阅读的.比如`"Wed Jun 30 21:49:08 1993\n"`这样的值,其中Web表示星,一个星期的7天都有相应的缩写. Jun表示月份,12个月都有自己的缩写. 返回的值是静态的存的,所以后面的类似的函数调用会改变返回的值.也就是得到这个返回值之后,如果想要保留的话,最好马上进行strdup.

asctime_r就多了一个参数,这个参数就会用来存放返回值. 当然,掉用的时候必须保证buf中至少可以存下26个字符.

gmtime和localtime都是将一个日历时间转为细分时间.它们的区别是gmtime使用的时区是utc,而localtime的时区是当前用户所在的时区,也就是`locale`设置的时区. 这儿需要注意的是,日历时间是一个绝对时间,细分时间是相对的. 比如在中国和utc0处,同一个日历时间,对应的细分时间是不同的,中国的要快了8个小时.

	time_t ct;
	ct = time(NULL);
	printf("time = %lds\n",ct);
	printf("gm=%s\n",asctime(gmtime(&ct)));
	printf("local=%s\n",asctime(localtime(&ct)));
	printf("local=%s,gm=%s\n",asctime(localtime(&ct)),asctime(gmtime(&ct)));
	printf("ctime = %s\n",ctime(&ct));

结果为

    time = 1431058122s
    gm=Fri May  8 04:08:42 2015

    local=Fri May  8 12:08:42 2015

    local=Fri May  8 12:08:42 2015
    ,gm=Fri May  8 12:08:42 2015

    ctime = Fri May  8 12:08:42 2015

注意到就爱那个local和gm同时打印的时候是有问题的,因为这两个参数实际上都是指向了同一个内部的字符串.

mktime是将一个细分时间转为日历时间.
