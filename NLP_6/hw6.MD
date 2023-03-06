

## 实验目的

任务描述：使用工具CRF++进行实体识别训练。  
提供train.data，test.data分别作为训练集与测试集。  
使用CRF++，提供模板template，训练模型。  
使用模型对测试集进行预测，计算评估准确率（保留四位小数），并输出前100个词的识别结果。

## 实验平台

jupyter
CRF

## 实验步骤

1. 首先下载并解压CRF++-0.58文件包，并在对应的路径下（`C:\CRF++-0.58\example\hwFinal`）设置好对应的文件

![在这里插入图片描述](https://img-blog.csdnimg.cn/2c8d0ecd4f4c4e46984b68eff05a1205.png)

这里只需要红线画出的部分，分别为`crf_learn.exe crf_test.exe libcrfpp.dll hw6-data-template hw6-data-test.data hw6-data-train.data`

2. 打开CMD跳转到对应的路径

![在这里插入图片描述](https://img-blog.csdnimg.cn/b3949dd6c920400a83aa500ee43deaaa.png)

运行以下命令

`crf_learn hw6-data-template hw6-data-train.data model`

![在这里插入图片描述](https://img-blog.csdnimg.cn/edd28e7bba814e6188b1ef0fd0ebb7fc.png)

会发现文件夹中多出了`model`，并且使用`crf_learn hw6-data-template hw6-data-train.data model >> model_out.txt`可以保存文字信息。程序运行结束
![在这里插入图片描述](https://img-blog.csdnimg.cn/02b2b1499de24da087ff38055f70e884.png)

3. 继续执行新的命令

`crf_test -m model hw6-data-test.data`

结果如下
![在这里插入图片描述](https://img-blog.csdnimg.cn/e44fa095258f41eea80bc1ef28e6bbf5.png)


4. 将得到的数据结果进行如下操作
```py
test_out = []
for line in open("./test_out.txt"):
    temp = line.split()
    #if len(temp) != 0:
    test_out.append(temp)

all = 0
acc = 0
cnt = 1
cognization = []
for line in test_out:
    if len(line) == 0: # 空白行需要输出or跳过
        if len(cognization) == 0: #多个空白行同时出现，跳过
            continue

        # 输出
        print("sentence %d" % cnt)
        cnt += 1
        identifier = cognization[0][0]
        to_print = cognization[0][1]
        for i in range(1, len(cognization)):
            if identifier[2:] == cognization[i][0][2:]:
                to_print += " " + cognization[i][1]
            else:
                print(identifier[2:], to_print)
                identifier = cognization[i][0]
                to_print = cognization[i][1]
        
        print(identifier[2:], to_print)
        cognization = []
        print("--------------")
        continue

    # ACCURACY特征
    if line[3] != 'O':
        cognization.append([line[3] , line[0]])
    if line[2] == line[3]:
        acc += 1
    all += 1

print(acc, all, "ACC: %.4f" % (acc/all))

```

得到辨识结果和最后的ACCURACY数据
![在这里插入图片描述](https://img-blog.csdnimg.cn/c02f00d038d2477abff1bced2ec22cdd.png)

## 分析template模板

template模板内容如下
```py
# Unigram
U00:%x[-2,0]
U01:%x[-1,0]
U02:%x[0,0]
U03:%x[1,0]
U04:%x[2,0]
U05:%x[-1,0]/%x[0,0]
U06:%x[0,0]/%x[1,0]

U10:%x[-2,1]
U11:%x[-1,1]
U12:%x[0,1]
U13:%x[1,1]
U14:%x[2,1]
U15:%x[-2,1]/%x[-1,1]
U16:%x[-1,1]/%x[0,1]
U17:%x[0,1]/%x[1,1]
U18:%x[1,1]/%x[2,1]

U20:%x[-2,1]/%x[-1,1]/%x[0,1]
U21:%x[-1,1]/%x[0,1]/%x[1,1]
U22:%x[0,1]/%x[1,1]/%x[2,1]

# Bigram
B

```

拿下列文本举例子
```
is	    VBZ	O	
a	    DT	O	
clear	JJ	O	
signal	NN	O	
that	IN	O	
```

当前假定测试到了clear


|原文| 意义|对应内容|
|-|-|-|
U00:%x[-2,0]|-2行，0列|is
U01:%x[-1,0]|-1行，0列|a
U02:%x[0,0]|0行，0列|clear
U03:%x[1,0]|1行，0列|signal
U04:%x[2,0]|2行，0列|that
U05:%x[-1,0]/%x[0,0]|-1行0列与0行0列的组合|a/clear
U06:%x[0,0]/%x[1,0]|0行0列与1行0列的组合|clear/signal|
U10:%x[-2,1]|-2行，1列|VBZ
U11:%x[-1,1]|-1行，1列|DT
U12:%x[0,1]|0行，1列|JJ
U13:%x[1,1]|1行，1列|NN
U14:%x[2,1]|2行，1列|IN
U15:%x[-2,1]/%x[-1,1]|-2行1列与-1行1列组合|VBZ/DT
U16:%x[-1,1]/%x[0,1]|-1行1列与0行1列组合|DT/JJ
U17:%x[0,1]/%x[1,1]|0行1列与1行1列组合|JJ/NN
U18:%x[1,1]/%x[2,1]|1行1列与2行1列组合|NN/IN
U20:%x[-2,1]/%x[-1,1]/%x[0,1]|-2行1列,-1行1列,0行1列组合|VBZ/DT/JJ
U21:%x[-1,1]/%x[0,1]/%x[1,1]|-1行1列,0行1列,1行1列组合|DT/JJ/NN
U22:%x[0,1]/%x[1,1]/%x[2,1]|0行1列,1行1列,2行1列组合|JJ/NN/IN