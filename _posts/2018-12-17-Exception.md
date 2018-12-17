---
layout: post
#标题配置
title:  Java基础-异常
#时间配置
date:   2018-12-17 14:19:00 +0800
#大类配置
categories: Java
#小类配置
tag: 教程
---

* content
{:toc}

# 异常的优势

## 剥离业务代码和错误代码
参考一个将整个文件读入内存的代码，例如：
```
readFile {
    open the file;
    determine its size;
    allocate that much memory;
    read the file into memory;
    close the file;
}
```
乍一看，这个功能似乎很简单，但它忽略了以下所有潜在的错误。

如果无法打开文件会怎么样？
如果无法确定文件的长度会发生什么？
如果无法分配足够的内存会怎样？
如果读取失败会发生什么？
如果文件无法关闭会发生什么？
要处理此类情况，该readFile函数必须具有更多代码才能执行错误检测，报告和处理。以下是函数可能的示例:
```
errorCodeType readFile {
    initialize errorCode = 0;
    
    open the file;
    if (theFileIsOpen) {
        determine the length of the file;
        if (gotTheFileLength) {
            allocate that much memory;
            if (gotEnoughMemory) {
                read the file into memory;
                if (readFailed) {
                    errorCode = -1;
                }
            } else {
                errorCode = -2;
            }
        } else {
            errorCode = -3;
        }
        close the file;
        if (theFileDidntClose && errorCode == 0) {
            errorCode = -4;
        } else {
            errorCode = errorCode and -4;
        }
    } else {
        errorCode = -5;
    }
    return errorCode;
}
```
这段代码有大量基于if...else的逻辑判断，代码的逻辑流程已经很难理解，试想这样的代码在编写完成三个月后进行重构的难度之大。

使用异常进行代码重构，示例如下：
```
readFile {
    try {
        open the file;
        determine its size;
        allocate that much memory;
        read the file into memory;
        close the file;
    } catch (fileOpenFailed) {
       doSomething;
    } catch (sizeDeterminationFailed) {
        doSomething;
    } catch (memoryAllocationFailed) {
        doSomething;
    } catch (readFailed) {
        doSomething;
    } catch (fileCloseFailed) {
        doSomething;
    }
}
```
**异常可以使我们的代码结构更清晰和合理**