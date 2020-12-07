---
title: R语言基础の笔记
tags: [R语言, Note]
categories: 「语言」
permalink: R-Notes
date: 2019-04-01 19:52:11
---
<center> <font color="#bababa">

***R语言基础の笔记***

</font></center>
<!--more-->

---

# R 语言基础  
## 1.一维数据结构-概念介绍
- 数据类型  
    + character：字符型
    + numeric：数值型，指实数或小数
    + integer：整型
    + complex：复数型
    + logical：逻辑型
- 数据结构
    + 向量（对应数值变量）、因子（对应分类变量）
    + 矩阵（内部数据类型一致）、数据框（内部每个向量内数据类型一致）
    + 数组（三维、更高维）、列表（可包含不同结构类型元素）
- 向量
    + c()
    + 冒号操作符:
    + seq(from, to, by, lenth.out, along.with)
    ·提取子集
    + 数字下标（正数、负数）
    + which()函数
- 因子
    + factor()
    + gl()

## 2.一维数据结构-代码演示
- 等号=跟<-是一样的赋值操作符
- (x=c(10,11,13,14))，最外的括号表示立即输出
- Ctrl+Enter为单行执行
- 因子的水平（level）在R语言内部用数字表示
- 一维数据结构

## 3.二位数据结构——矩阵
- 行和列
- 生成矩阵
    + matrix()
    + 由向量派生
    + 由向量组合生成
- 操作
    + 访问元素
    + 行、列命名
- 矩阵运算：向量运算按照矩阵运算统一
    + 加减运算
    + 乘法运算：数乘、对应元素相乘、矩阵乘法
    + 单位矩阵、对角矩阵
    + 矩阵转置
    + 矩阵的特征值和特征向量
    + 矩阵的逆
    + 求解线性方程组

## 4.二位数据结构——数据框
- 记录和域
- 创建数据框 data.frame()
- 操作：
- 访问元素 $、[[]]
- attach、detach
- with、within
- subset

## 5.高维数据结构——列表和数组
- 列表中：成分
- 创建列表：list()
- 操作：
- 列表成分names
- []、[[]]
- 数组：不同于其他语言的数组，是矩阵的延伸

## 6.数据类型转换
- 类型转换
    + 安全级别：字符>数字>布尔（更安全）
    + is.character、as.character...
- 数据结构
    + is.mastrix、as.mastrix...

## 7.分支结构
- if ... else ...结构
```r
if(condition){
    ...
}
else{
    ...
}
```
- ifelse()函数，支持向量化计算  
`ifelse(b,u,v)`  

## 8.循环结构
- for(n in x){...} （x是向量）
- while(codition){...}
- repeat{... break}
- break、next

## 9.函数和包：代码重用
- 自定义函数：(一类对象，可以随时创建）
```r
myfun=function(par1,par2..){
    ......
}
```
- 查看函数代码
    + 不带括号的函数名
    + page
- source函数加载文件（保存成.r文件）  

`source('E:/rTEST/06.function.r')`

## 10.向量化运算和apply家族
- 向量化运算
    + +-*/四则运算
    + <!=逻辑运算
- apply家族
    + apply
    + sapply、lapply
    + mapply、tapply

---

# 建立对数据的感性认识
## 1.变量类型
- 连续变量（数值变量）
- 离散变量（名义变量、分类变量）

## 2.数据分布
- 分布就是概率
- 分布函数
    + 概率密度函数（PDF）
    + 累计密度函数（CDF）
    + 
## 3.连续变量的典型分布
- 正态分布
- 中心极限定理
- 经验法则
- 切比雪夫定理

## 4.离散变量的典型分布
- 0-1分布
- 二项分布
- 泊松分布（n很大，p很小时对二项分布的近似）

## 5.单变量的集中趋势描述
- 集中趋势：一组数据向着一个中心靠拢的程度，也体现了数据中心点所在位置。
- 分类变量
    + 众数
- 连续变量
    + 均值（算数平均数、几何平均数、加权平均数）
    + 中位数
    + 分位数
- 被滥用的均值
    + 非单峰分布
    + 极值的影响
    + 简单的算术平均

## 6.单变量的离散程度描述
- 离散程度：一组数据远离其中心的程度。
    + 集中趋势从数据中选“典型代表”，“代表是否够典型”由离散程度检验。
- 离散统计量
    + 极差
    + 方差、标准差
    + Z分数
    + 变异系数
- 偏度（Skewness）描述某变量取值分不对称性。
    + 对称（=0）
    + 左偏（<0）均值落于中位数左侧，极小值多
    + 右偏（>0）均值落于中位数右侧，极大值多
- 峰度（Kurtosis）描述某变量所有取值分布形态陡缓程度。
    + 正态分布（0、3）
    + 尖顶峰（>0、3）
    + 平定峰（<0、3）

## 7.双变量的统计描述
- 相关性：两个变量共同变化的趋势。（是否有线性关系，相关不代表因果，正/负不代表相关性强弱）
    + 正相关、负相关、线性无关
    + 协方差 Cov(X,Y)=E[(X-E[X])(Y-E[Y])]=E[XY]-2E[Y]E[X]+E[X]E[Y]=E[XY]-E[X]E[Y]
    + 相关系数 ρXY=Cov(X,Y)/(D(X)^0.5*D(Y)^0.5)  （-1~1之间）
    + 协方差矩阵（对角线都是1，对称矩阵）

## 8.产生符合特定分布的测试数据
- 分布函数家族：*func()
*：
    + r：随机分布函数
    + d：概率密度函数
    + p：累积分布函数
    + q：分位数函数

```r
# 1.生成符合特定分布的数据
#   二项分布
#   x~(N,P)  N=100, P=0.7
str(rbinom)

#   生成随机数
x=rbinom(5,1,0.7)

x=rbinom(5,100000,0.7)

#   得到概率
y=dbinom(20,100,0.7) #100次实验中正好共发生20次的概率
y=dbinom(60:90,100,0.7)
sum(y)

y=dbinom(0:100,100,0.7)
plot(0:100,y,pch=16) #概率密度曲线

#   得到累积概率
y=pbinom(20,100,0.7) #100次实验中发生小于20次的概率

pbinom(40,100,0.7)-pbinom(20,100,0.7)

#   累积函数密度曲线
plot(0:100,pbinom(0:100,100,0.7),pch=16)

#   得到分位数
x=pbinom(0.4,100,0.7)
```

## 9.R的单变量统计函数
- 均值：mean
- 中位数：median
- 分位数：quantile
- 方差：var
- 标准差：sd
- 频数表：table
- 峰度
- 偏度
```r
# 2.单变量的描述统计
str(airquality)
summary(airquality)

mean(airquality$Ozone,na.rm=T) #变量中有NA（缺失值)时删除NA
median(airquality$Ozone,na.rm=T)

mean(airquality$Temp,na.rm=T,trim=.01) #trim是按百分比裁剪极值

temp100=rnorm(100,30,1)
w=1:100
(wtm=(weighted.mean(temp100,w,na.rm=T))) #加权平均数

x=c(.045,.021,.225,.018)
(xm=mean(x))
(xg=exp(mean(log(x)))) #几何平均数

(tmid=median(temp100,na.rm=T)) #中位数

quantile(airquality$Temp,na.rm=T) #分位数
quantile(airquality$Temp,probs=c(0,0.1,0.9,1))

(tv=var(temp100)) #方差
(ts=sd(temp100)) #标准差

fivenum(temp100)

# 自定义偏度、峰度计算函数
apply(airquality[,c(-5,-6)],2,FUN=mean,na.rm=T)
sapply(airquality[,c(-5,-6)],FUN=mean,na.rm=T)

mysummary=function(x,...){
  Av=mean(x,na.rm=T)
  Sd=sd(x,na.rm=T)
  N=length(x[!is.na(x)])
  Sk=sum((x[!is.na(x)]-Av)^3/Sd^3)/N
  Ku=sum((x[!is.na(x)]-Av)^3/Sd^4)/N-3
  
  result=c(avg=Av,sd=Sd,skew=Sk,kurt=Ku)
  return(result)
}

sapply(airquality[,c(-5,-6)],FUN=mysummary,na.rm=T)

# 非单峰分布不能简单计算均值
x=rnorm(100,50,6)
y=rnorm(200,150,8)
z=c(x,y)
plot(density(z))
abline(v=mean(z),col=3,lw=3)
```

## 10.R的双变量相关函数
- 协方差：cov
- 相关系数：cor
- 确实值处理：行删除、配对删除
```r
# 3.双变量的相关性
cov(airquality[,-5:-6],use='pairwise.complete.obs') #配对删除
cov(airquality[,-5:-6],use='complete.obs') #行删除

cor(airquality[,-5:-6],use='pairwise.complete.obs') #配对删除
cor(airquality[,-5:-6],use='complete.obs') #行删除
```

## 11.统计量的可视化
- 散点图
- 盒线图
- 直方图
- 核密度图
- 柱状图、条形图
- 饼图
- 点图
- 散点图集
```r
# 散点图
plot(airquality$Ozone)
plot(airquality$Ozone,airquality$Wind)

# 箱线图
boxplot(airquality$Temp)
boxplot(airquality$Temp,horizontal = T)

# 直方图
hist(airquality$Temp)
hist(airquality$Temp,breaks=30) #横轴块数
hist(airquality$Temp,prob=T) #纵轴从频数变成频率

# 密度
plot(density(airquality$Temp))

hist(airquality$Temp,prob=T)
lines(density(airquality$Temp),col=3,lw=4)

# 柱状图
barplot(table(airquality$Month))
barplot(table(airquality$Month),horiz = T)

# 饼图
pie(table(airquality$Month))

# 图集
plot(airquality[,1:4])
pairs(airquality[,1:4])
```

# 数据的组织和整理——输入和输出
## 1.基本输入输出
- 输入
    1）readline
    2）edit、fix
- 输出
    1）print
    2）cat
- 输出重定向
    1）sink
```r
# 基本输入输出
#   输入
x=readline('你好：')
x

mydata=data.frame(name=character(0),age=numeric(0),height=numeric(0))
mydata=edit(mydata) #注意要用=把临时变量赋回来
mydata

fix(mydata)
mydata

#   输出
x=rnorm(10,100,2)
print(x) #输出有编号，末尾自带换行符
cat(x) #更紧凑，无编号，可以打印到文件中
print(x,digits=4) #输出格式
print('helloword 1');print('welcome to R')
cat('hello world 1');cat('welcome to R') #输出末尾无换行符
cat(format(x,digits=3),'\n') #输出格式，换行符

cat('hello world',file='d:/app.log')

sink('d:/output.txt') #执行后，之后的print、cat输出都会被重新向到文件中
print('hello')
cat('sdfjksdf','\n')
sink()
```

## 2.模拟数据和自带数据集
- 模拟数据
- 自带数据集
```r
# 任意分布
# y=a*x+b+e
# x~N(0,2)
# e~N(0,1)
# b=0.5, a=2

set.seed(10) # 随机种子，用来产生相同的随机数
x=rnorm(100,0,2)
e=rnorm(100)
y=2*x+0.5+e
plot(x,y)

# 生成随机数
(x=rbinom(5,100,0.7))
(x=rbinom(5,100,0.7))

set.seed(10)
(x=rbinom(5,100,0.7))
(x=rbinom(5,100,0.7))
set.seed(10)
(x=rbinom(5,100,0.7))
(x=rbinom(5,100,0.7)) # 跟上面相同

data(package='datasets')
data()

# 查看系统所有包中的数据集
data(package=.packages(all.available = T))
library(arules)
```

## 3.文件数据源
- 文本文件：read.table read.csv read.delim
- Excel文件：四种方法
- SPSS文件：foreign::read.spss、Hmisc::spss.get
```r
# 文件数据源
getwd() #查看当前工作目录
setwd('d:/') #更改当前工作目录
x=rnorm(1000,10,2)
y=rnorm(1000,10,2)
z=rnorm(1000,10,2)
save(x,y,z,file='xyz.Rdata')
l=load('xyz.Rdata')
l # l=变量名字列表

x=read.table('xyz.Rdata',header=F,seq='',comment.char='@')
x
str(x)
x=read.csv('scan0.txt',header=F,seq='')
x=read.delim('scan0.txt',header=F) #这几个red都一样，只是参数缺省值不同
x=read.delim('clipborad',header=F) #从剪切板读

write.table(x,'d:/scan2.txt',seq=',',quote=F,col.names=T)

# Excel文件
#   1.csv文件
#   2.剪切板+read.delim
#   3.xlsx扩展包
#   4.rodbc数据源

library(foreign) #这个包提供了read.spss方法
cars=read.spss('d:/car_sales.sav')

library(Hmisc) #简单方法，需要额外装这个包
cust=spss.get('d:/car_sales.csv',use.value.labels=T)
```

## 4.关系型数据库（MySQL）
- ODBC方式（开放数据库连接（Open Database Connectivity）定义了访问数据库API的一个规范）
- RMySQL方式
```r
#RMySQL
install.packages('RMySQL')
library(RMySQL)
conn = dbConnect(MySQL(),dbname='rtest',username='rtest',password='rtest',host="192.168.1.100",port=3306)

dbListTables(conn)
dbListFields(conn,'t_user')
summary(MySQL(),verbose=T)

users=dbReadTable(conn,'t_user')
str(users)
users

tmpUser=data.frame( name=paste('user',1:100,sep=''),
                   age=rnorm(100,50,5))
tmpUser
dbWriteTable(conn,'t_user',tmpUser,append=T,row.names=FALSE)


dbWriteTable(conn,'t_stu',tmpUser,append=T)
dbReadTable(conn,'t_stu')


res=dbGetQuery(conn,'select * from t_user where age>10')
res

res=dbSendQuery(conn,'show databases') 
dl=fetch(res)  
dl

dbDisconnect(conn) # 断开连接

#RODBC    mysqlodbc驱动要装，system32文件下odbcad32.exe
install.packages('RODBC')
library(RODBC)
conn=odbcConnect("mysqlodbc")
conn=odbcConnect("mysqlodbc", uid="rtest", pwd='rtest')
sqlTables(conn)
users=sqlFetch(conn,'t_user')
users
str(users)

users=sqlQuery(conn,'select * from t_user where age>15')
users
 
odbcClose(conn)
```



# 数据的组织和整理——数据预处理（1）

## 1.日期时间、字符串的处理
·日期和时间
    - Date：日期类
    - POSIXct：日期时间类，精确到秒。内部用数字表示
```r
# 得到当前日期时间
(di=Sys.Date())
(d2=date())

myDate=as.Date('2007-08-09')
class(myDate)
mode(myDate)

# 日期转字符串
as.character(myDate)

dirDay=c('01/05/1986','08/11/1976')
dates=as.Date(dirDay,'%m/%d/%Y')
dates

td=Sys.Date()
format(td,format='%B  %d %Y')
format(td,format='%A,%a')

# 日期转换成个数字
as.integer(Sys.Date())
as.integer(as.Date('1970-1-1'))
as.integer(as.Date('1970-1-2'))

sdate=as.Date('1904-1-1')
edate=as.Date('2010-11-1')
days=edate-sdate
days

ws=difftime(Sys.Date(),as.Date('1956-10-12'),units='weeks')

# 把年月日拼成日期
(d=ISOdate(2011,10,2));class(d) # ISOdate()可以拼成日期，类型是POSIXct
as.Date(ISOdate(2011,10,2))

ISOdate(2011,2,30) #不存在的日期返回NA

# 批量转换成日期
years=c(2010,2011,2012,2013,2014,2015)
months=1
days=c(15,20,21,19,30,3)
as.Date(ISOdate(years,months,days)) # ISOdate()支持向量化运算

# 提取日期时间的一部分
p=as.POSIXlt(Sys.Date())
Sys.Date()
p$year+1900 # 年要+1900
p$mon+1 # 月要+1
p$mday
p$hour
p$min
p$sec
```

- 字符串
    - nchar(),length()
    - paste(),outer()
    - substr(),strsplit()
    - Sub(),gsub(),grep(),regexpr(),grepexpr()
    - POSIXlt：日期时间类，精确到秒。内部用列表表示
    - Sys.date(),date(),difftime(),ISOdate(),ISOdatetime()

```r
x='hello\rwld\n'
cat(x) # 会执行转义字符
print(x) # 不执行，都输出显示

# 字符串长度
nchar(x)
length(x) # 得到向量个数

# 字符串拼接
board=paste('b',1:4,seq='-')
board

mm=paste('mm',1:3,seq='-')
mm

outer(board,mm,paste,seq=':') # outer向量配对

# 拆分提取
board
substr(board,3,3) # 第3个开始，第3个结束
strsplit(board,'-',fixed=T) # fixed抑制正则表达式

# 修改
sub('-','.',board,fixed=T) # 只是修改了临时对象
board
mm
sub('m','p',mm) # sub替换第一个匹配项
gsub('m','p',mm) # gsub替换全部匹配项

# 查找
mm=c(mm,'mm4')
mm
grep('-',mm) # 返回第几个元素包含'-'
regexpr('-',mm) # 返返回位置信息
```

## 2.数据预处理概念
- 数据预处理的意义
·数据质量
    + 准确性，完整性，一致性，冗余性，时效性……
- 数据预处理的内容
    + 数据集成，数据转换，数据清洗，数据简约
    + 数据清洗时又脏又累的活

## 3.数据集成
- merge #数据框合成
- pylr::join # 包名::函数
```r
(customer=data.frame(Id=c(1:6),State=c(rep("北京",3),rep("上海",3))))
(ol=data.frame(Id=c(1,4,6,7),Product=c('IPhone','Vixo','mi','Note2')))

merge(customer,ol,by=('Id')) # inner join
merge(customer,ol,by=('Id'),all=T) # full jion 全连接
merge(customer,ol,by=('Id'),all.x=T) # left outer jion 左连接
merge(customer,ol,by=('Id'),all.y=T) # right outer jion 右连接

# union 去重 在df1和df2有相同的列名称下
rbind(df1,df2) # 不去重合并

merge(df1,df2,all=T) # 去重合并
merge(df1,df2,by=('id')) # 显示id相同的行 
```

## 4.数据转换
- 构造属性
- 规范化（极差化、标准化）
- 离散化
- 改善分布
```r
str(airquality)
head(airquality,3)

# 排序
airquality=airquality[order(airquality$Temp),]

head(airquality,5)
tail(airquality,5)

# 构造属性
quantile(airquality$Temp,probs=c(0,0.3,0.6,1.0))
airquality$isHot=ifelse(airquality$Temp>80,T,F)
airquality=within(airquality,{TempL=NA
TempL[Temp>80]='Hot'
TempL[Temp>70&Temp<=80]='Warm'
TempL[Temp<=70]='Cold'
})

airquality$TempL

airquality$TempL=factor(airquality$TempL,levels=c('Cold','Warm','Hot',ordered=TRUE))
unclass(airquality$TempL)

# 数据规范化
tmp=data.frame(TC=scale(airquality$Temp,scale=F),
               TZ=scale(airquality$Temp))
tmp

# 连续变量变离散变量
airquality=within(airquality,{
  TempL1=cut(Temp,breaks=c(56,73,81,97),include.lowest = T)
})
head(airquality,5)
airquality$TempL1

airquality=within(airquality,{
  TempL1=cut(Temp,breaks=quantile(Temp,probs=c(0,0.3,0.7,1)),include.lowest = T)
})

table(airquality$TempL1)
prop.table(table(airquality$TempL1))

library(Hmisc)
airquality=within(airquality,{
  TempL3=cut2(Temp,g=4)
})
```

# 数据的组织和整理——数据预处理（2）
## 1.发现缺失值
- 缺失值的表达方式：NA（没有数字）、NaN（无效的数字，如1/0）、Inf（正无穷）、-Inf（负无穷）
- 发现缺省值：summary(),is.na(),complete.case()
- 缺失值是否有业务含义
- 把不合理数据编码为确实值
```r
# 没有意识到“缺失值问题”的时候就得到的答案
library(VIM)

sleep.lm=lm(Dream~Sl)

# 发现缺失值
#   1.属性级别
is.na(sleep$Sleep) # 返回的是向量，可用于提取

sleep$Sleep[is.na(sleep$Sleep)]
sleep$Sleep[!(is.na(sleep$Sleep))]

mean(is.na(sleep$Sleep)) #缺失值比例

#   2.记录级别
complete.cases(sleep) #在记录级别来识别

# 数据集拆分
sleep.cleaned=sleep[complete.cases(sleep),] #拆分出没有缺失值的数据
#sleep.cleaned=na.omit(sleep)
sleep.cleaned=sleep[complete.cases(sleep),] #拆分出有缺失值的数据
```

## 2.缺失值模式
- 完全随机缺失 Missing Completely At Random, MCAR（缺失跟其他变量、跟自己都没关系）
- 随机缺失 Missing At Random, MAR （缺失跟其他变量有关系，跟自己没关系）
- 非随机缺失 NMAR （既跟其他变量有关系，也跟自己有关系）
- VIM包
- mice包
```r
#   3.概要信息
library(mice)
md.pattern(sleep)

aggr(sleep,number=T,prop=F)
matrixplot(sleep)
marginplot(sleep[,c('Dream','Sleep')])
```

## 3.删除确实值
- 分析函数
    + na.rm=T （=T计算这条函数时不考虑缺失值，但不是删除，=F包含缺失值）
- 行删除
    + na.omit
- 配对删除  （多个分析中数据可能来自不同的数据集
    + 典型函数cor(data.use="pairwise.complete.obs")
```r
na.omit(sleep) # 整行删除
cor(sleep,use="pairwise.complete.obs") # 配对删除
```

## 4.填补缺失值
- 简单抽样填补
- 均值填补
- 回归填补
```r
# 简单随机抽样填充
sub.index=which(is.na(sleep$Sleep)==T)
sleep.clean=sleep[-sub.index,]
sleep.na=sleep[sub.index,] # 数据集的拆分

sleep.na$Sleep=sample(sleep.clean$Sleep,
                      nrow(sleep.na),replace = T) # 抽样（抽样范围，抽样条数，有放回抽样）

# 均值填充
sub.index=which(is.na(sleep$Dleep)==T)
sleep.clean=sleep[-sub.index,]
sleep.na=sleep[sub.index,]

sleep.na$Dream=mean(sleep.clean$Dream)

# 回归填充
sub.index=which(is.na(sleep$Sleep)==T)
sleep.clean=sleep[-sub.index,]
sleep.na=sleep[sub.index,]

sleep.lm=lm(Sleep~Gest+Exp,data=sleep)
sleep.na$Sleep=round(predict(sleep.lm,sleep.na),2) # round(,2)保留小数点后两位

# 用哪两个变量做回归填充预测
cor(cleep,use="pairwise.complete.obs")
library(corrgram)
corrgram(cor(sleep,use="pairwise.complete.obs"),
         lower.panel.panel.conf,upper.panel=panel.pie) # 看Sleep这一列，sleep跟哪两个相关性最强
```

# 数据可视化——基础绘图体系
## 1.图形设备
- 图形设备
    + pdf,png,jpeg,bmp
    + win.metafile,postscript
    + dev.off
- 高水平绘图函数
    + plot,pairs,hist
- 低水平绘图函数  （需要依托于高水平函数）
    + points,lines,text,abline,legend,title,axis
```r
plot.women=function(){
  plot(women$height,women$weight,
      main='height-weight',
      xlab='height',
      ylab='weight',
      col='red', # 点颜色
      pch=19, # 点的状态
      xlim=c(0,100), #x坐标范围
      ylim=c(100,200))
}

plot.line=function(){
  lines(women$height,women$weight,col='blue')
}

plot.women()

pdf('d:/women.pdf') # 图都绘制到pdf设备中了
plot.women()
dev.off() # 及时关闭文件设备

plot.women()
plot.line()
```

## 2.数值型单变量的可视化（箱线图和直方图）
- 箱线图
    + 分组箱线图
- 直方图
```r
# 箱线图
bp.info=boxplot(airquality$Ozone,main='OZone',horizontal=T,axes=T) #有返回值，horizontal方向，axes是否显示坐标轴

abline(v=mean(airquality$Ozone,na.rm = T),col='red',lty='dotted')

bp.info

# 分组箱线图
bp.info=boxplot(airquality$Ozone~airquality$Month,main='OZone',horizontal=T,axes=T)

# 直方图
ht.info=hist(airquality$Ozone,breaks = 20,freq = F) #breaks=区间数量，freq=纵坐标频数/频率

lines(density(airquality$Ozone,na.rm = T))
```

## 3.数值型单变量的可视化（密度图和小提琴图）
```r
hist(airquality$Ozone,breaks = 20,freq = F) #breaks=区间数量，freq=纵坐标频数/频率

# 密度图
lines(density(airquality$Ozone,na.rm = T)) #核密度估计图

omean=mean(airquality$Ozone,na.rm = T)
osd=sd(airquality$Ozone,na.rm = T)

x=seq(from=min(airquality$Ozone,na.rm = T),
      to=max(airquality$Ozone,na.rm = T),
      length=1000)
y=dnorm(x,omean,osd)
lines(x,y,col='red',lwd=2)

# 小提琴图
library(vioplot)
help(package="vioplot")
</code>
4.离散型单变量的可视化
·条形图和柱状图
·分组条形图（柱状图）
·堆叠条形图（柱状图）
·饼图
<code>
score=read.csv('d:/ReprotCard.txt',header=T,seq=' ')
score=na.omit(score) # 干掉确实值
nGrade=tapply(score$avScore,score$avScore,length)
nGrade
barplot(nGrade,names=c('良','中','及格','不及格'),horiz=F) #horiz=T条形图，=F柱状图

# 分组柱状图
nGradeBySex=table(score&sex,score$avScore)
nGradeBySex
barplot(nGradeBySex,beside=F,col=(1,2),ylim=c(0,20)) #beside=T分组柱状图，=F堆叠柱状图
legend('topright',c('女','男')，pch=c(15,15),col=c(1,2),horiz=F,cex=0.8) # 图例

# 饼图
score=read.csv('d:/ReprotCard.txt',header=T,seq=' ')
score=na.omit(score)
nGrade=table(score$avScore)
pie(nGrade)
pct=round(nGrade/length(score$avScore)*100,2)
pct
labs=paste(c('良','中','及格','不及格'),pct,'%',seq='')
labs
library(RcolorBrewer)
pie(nGrade,labels=labs,main='各等级比例'，col=brewer.pal(length(nGrade),'Set1'))

# 3D饼图
library(plotrix)
pie3D(nGrade,labels=labs,main='各等级比例'，col=brewer.pal(length(nGrade),'Set1'),explode=0.2)
fan.plot(nGrade,labels=labs,main='各等级比例'，col=brewer.pal(length(nGrade),'Set1')) # 扇形图

par(mfrow=c(1,3)) #窗口画布布局一行三列
pie(nGrade,labels=labs,main='各等级比例'，col=brewer.pal(length(nGrade),'Set1'))
pie3D(nGrade,labels=labs,main='各等级比例'，col=brewer.pal(length(nGrade),'Set1'),explode=0.2)
fan.plot(nGrade,labels=labs,main='各等级比例'，col=brewer.pal(length(nGrade),'Set1')) # 扇形图
```


# 数据可视化——多变量关系可视化
## 1.双变量的可视化
- 连续型双变量
- 连续型变量+离散型变量
- 离散型变量
- 散点图
```r
x=rnorm(1000)
y=rnorm(1000)

plot(x,y,cex=1,pch=19,col='red',main='XY散点图') #cex点的大小，pch点的形状

x=rbinom(10000)
y=rbinom(10000,10,0.1) 
plot(x,y,cex=1,pch=19,col='red',main='XY散点图') # 点重叠，丢失密度信息

sunflowerplot(x,y,cex=1,col='red',main='XY散点图') #太阳花法

plot(jitter(x),jitter(y),cex=1,col='red',main='XY散点图') #加入抖动

#install.packages('scatterplot3d')
library(scatterplot3d)
s3d=scatterplot3d(airquality$Temp,airquality$Wind,airquality$Ozone
                  highlight.3d=TRUE,col.axis='blue',
                  col.grid="lightblue",angle=30)

attach(airquality)
plot(Wind,Temp)
# 1.一元线性回归
alm=lm(Temp~Wind)
abline(alm$coefficients)

#2.局部加权回归
alowess=loess(Temp~Wind)

ord=order(Wind)
lines(Wind[ord],alowess$fitted[ord],lwd=1,col=2)
detach(airquality)
```
## 2.多变量的可视化
- 矩阵散点图
- 相关系数图
```r
# 矩阵散点图
str(airquality)
pairs(~Ozone+Solar.R+Wind+Temp,data=airquality)
plot(~Ozone+Solar.R+Wind+Temp,data=airquality)

# 矩阵散点图中加线
library(car)
scatterplotMatrix(~Ozone+Solar.R+Wind+Temp,data=airquality,lty.smooth=2,spread=T)

#相关系数图
library(corrgram)
corrgram(airquality[,c(-5,-6)],
         lower.panel=panel.pie,
         upper.panel=panel.conf,
         text.panel=panel.txt)
```

## 3.分组可视化
- Lattice绘图体系
    + 面板和面板函数
```r
unique(iris$Species)

par(mfrow=c(2,2)) #定义3行1列布局

by(iris,iris$Species,FUN=function(data){
  plot(data$Sepal.Length,data$Sepal.Width,main=unique(data$Species))
})

# Lattice包
library(lattice)
head(iris,3)
xyplot(Sepal.Length~Sepal.Width,data=iris)

plot(Sepal.Length~Sepal.Width,data=iris)

xyplot(Sepal.Length~Sepal.Width,data=iris,groups=Species)

xyplot(Sepal.Length~Sepal.Width | Species,data=iris) #分组

# 箱线图
bwplot(Sepal.Length~Species,data=iris)
bwplot(~Sepal.Length | Species,data=iris)

# 直方图
histogram(~Sepal.Width | Species,data=iris)

# 密度图
densityplot(~Sepal.Width | Species,data=iris) 
densityplot(~Sepal.Width,groups=Species,data=iris) 

# 带状图
stripplot(Species~Sepal.Width,data=iris)

# 平行坐标图
parallelplot(~iris[1:4] | Species,iris)
parallelplot(~iris[1:4],groups=Species,iris,horizontal.axis = F)

# 3D图
cloud(Ozone~Wind*Temp|Month,data=airquality)
```

## 4.自定义面板函数实例
```r
panel.myplot=function(x,y){
  panel.grid(h=-1,y=-1) #网格
  panel.xyplot(x,y,pch=19) #散点图
  panel.loess(x,y,col='green',lwd=2,lty=3) #局部加权回归
  paney.lmline(x,y,col='red',lwd=2,lty=2) #线性回归
  panel.rug(x,y) #展示rug
  panel.abline(h=mean(y),lwd=1,lty=2,col='blue') #y的均值线
  panel.abline(v=mean(x),lwd=1,lty=2,col='blue')
}
xyplot(Sepal.Length~Sepal.Width|Species,data=iris,
       panel=panel.myplot,
       layout=c(2,2),aspect=1.5)
ls('package:lattice',pattern = '^panel.+')
```




# 假设检验
## 1.假设检验概述
- 什么是假设检验
- 假设检验的目的
- 两种误差：系统误差，随机误差
- 假设检验执行步骤
    + 简历要检验的假设，确定检验水准
    + 选择并计算适宜的统计量
    + 确定P值，做出推断
## 2.t分布和t检验
- t分布的特点
    + 单峰，以0为中心，左右对称
    + 分布形态和样本数量n有关
    + n -> ∞时，逼近标准正态曲线
    + t曲线不是一条曲线，而是一簇曲线
- t检验是基于t分布的比较平均数的检验方法

