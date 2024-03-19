---
title: fpga-assembly
date: 2023-11-13 20:19:42
tags:
typora-root-url: fpga-assembly
---

# FPGA Assembly

FASM 是一种文件格式，用于指定 FPGA 比特流中需要置1或清0的位。FASM 的设计初衷是提供一个中间层，令 FPGA 布局布线工具无需关心实际运行的 FPGA 比特流文件格式而工作。FASM 文件格式具有以下特性：

- 从文件中删除任意一行都不会影响其有效性（注：这里的“有效性”指的是能正常运作，即删除的一行不会影响其他行的功能，其他基于行的操作也不会影响有效性，如连接和排序等）。
- 允许创建带有人或计算机可读注释（comment）的注解（annotation）。
- 拥有用于表示查找表初始位/内存/其他大型数据数组的语法糖。
- 拥有书写规范格式。
- 不依赖于任何特定的比特流文件格式。

<img src="workflow.png" alt="workflow" style="width:75%;" />

## FASM 的语法特性

下面从 FASM 的行、注解、语法规则、格式规范四个方面介绍 FASM 的语法特性。

### FASM 的行

根据 FASM 文件格式的定义，FASM 不支持跨行的表示。下面给出几个例子。

**启用特性的规范**

```
# Set a single feature bit to 1 (with an implicit 1)
INT_L_X10Y146.SW6BEG0.WW2END0
CLBLL_L_X12Y124.SLICEL_X0.BLUT.INIT[17]
```

**推荐的比特数组**

```
# Setting a bitarray
CLBLL_R_X13Y132.SLICEL_X0.ALUT.INIT[63:32] = 32'b11110000111100001111000011110000
```

**允许的高级变化**

```
# The bitarray syntax also allows explicit 1 and explicit 0, if verbosity is desired.
# An explicit 1 is the same as an implicit 1.
INT_L_X10Y146.SW6BEG0.WW2END0 = 1
CLBLL_L_X12Y124.SLICEL_X0.BLUT.INIT[17] = 1
# Explicit bit range to 1
INT_L_X10Y146.SW6BEG0.WW2END0[0:0] = 1'b1
CLBLL_L_X12Y124.SLICEL_X0.BLUT.INIT[17:17] = 1'b1

# An explicit 0 has no effect on the bitstream output.
INT_L_X10Y146.SW6BEG0.WW2END0 = 0
CLBLL_L_X12Y124.SLICEL_X0.BLUT.INIT[17] = 0
# Explicit bit range to 0
INT_L_X10Y146.SW6BEG0.WW2END0[0:0] = 1'b0
CLBLL_L_X12Y124.SLICEL_X0.BLUT.INIT[17:17] = 1'b0
```

### FASM 的注解

注解不会改变 FASM 的输出结果。注解可以与 FASM 特性一起放在一行中，也可以单独放在一行中。与 FASM 特性在同一行的注解与该特性相关联，其他地方的注解与 FASM 文件相关联。下面是注解的实例。

```
# Annotation on a FASM feature
INT_L_X10Y146.SW6BEG0.WW2END0 { .attr = "" }
INT_L_X10Y146.SW6BEG0.WW2END0 { .filename = "/a/b/c.txt" }
INT_L_X10Y146.SW6BEG0.WW2END0 { module = "top", file = "/a/b/d.txt", line_number = "123" }

# Annotation by itself
{ .top_module = "/a/b/c/d.txt" }

# Annotation with FASM feature and comment
INT_L_X10Y146.SW6BEG0.WW2END0 { .top_module = "/a/b/c/d.txt" } # This is a comment
```

### FASM 的语法规则

```
Identifier ::= [a-zA-Z] [0-9a-zA-Z]*
Feature ::= Identifier ( '.' Identifier )*
S ::= #x9 | #x20

DecimalValue ::= [0-9_]*
HexidecimalValue ::= [0-9a-fA-F_]+
OctalValue ::= [0-7_]+
BinaryValue ::= [01_]+

VerilogValue ::= (( DecimalValue? S* "'" ( 'h' S* HexidecimalValue | 'b' S* BinaryValue | 'd' S*  DecimalValue | 'o' S* OctalValue ) | DecimalValue )

FeatureAddress ::= '[' DecimalValue (':' DecimalValue)? ']'

Any ::= [^#xA#]
Comment ::= '#' Any*

AnnotationName ::= [.a-zA-Z] [_0-9a-zA-Z]*
NonEscapeCharacters ::= [^\"]
EscapeSequences ::= '\\' | '\"'
Annotation ::= AnnotationName S* '=' S* '"' (NonEscapeCharacters | EscapeSequences)* '"'
Annotations ::= '{' S* Annotation ( ',' S* Annotation )* S* '}'

SetFasmFeature ::= Feature FeatureAddress? S* ('=' S* VerilogValue)?
FasmLine ::= S* SetFasmFeature? S* Annotations? S* Comment?
```

### FASM 的格式规范

1. 扁平化任何大于1的`FeatureAddress`。
   - 对于`FeatureAddress`宽度大于1的 `SetFasmFeature`行，`SetFasmFeature`为原`FeatureAdress`的宽度。
   - 当扁平化时，如果扁平化后的地址为0，则不发出该地址。
2. 移除所有的注释和注解。
3. 如果`FeatureValue`为0，则移除该 FASM 行。
4. 如果`FeatureValue`为1，如果`Feature`的地址不是0，则只输出`Feature`和`FeatureAdress`。
5. 删除任何不修改默认比特流功能的行，比如赛灵思部件中的伪 PIP。
6. 对 FASM 文件进行行排序。

## FASM的语句含义

在 FASM 文件中启用某个特性的行由`SetFasmFeature`进行定义。`SetFasmFeature`解析有三部分，要设置的特性`Feature`，要设置的特性中的地址`FeatureAddress`和特性的值`FeatureValue`。其中`FeatureAddress`和`FeatureValue`都是可选的。

| YYYY.XXXXX   | [A:B]            | =C             |
| ------------ | ---------------- | -------------- |
| `Feature`    | `FeatureAddress` | `FeatureValue` |
| **Required** | *Optional*       | *Optional*     |

### FASM的特性

`Feature`应该在比特流中唯一地指定一个特性。如果该特性在 FPGA 元素中重复出现，则需要前缀标识符来唯一地标识该特性所在的位置。

例如，所有的 SLICE 分片都有 ALUT，但是，每个 CLBLL_L 块实际上有两个 SLICEL，并且许多7系列 FPGA 带有 CLBLL_L 块。因此，需要一个唯一的路径来指定正在设置哪个分片，以及正在设置分片中的哪个 SLICEL。

### FASM的特性地址与特性值

如果没有指定`FeatureAddress`，则默认选择的地址为0，如果未指定`FeatureValue`，则默认设定该值为1。

如果`FeatureAddress`被指定为一个特定的位而不是一个范围（例如“[5]”），那么`FeatureAddress`宽度必须是1位宽（例如0或1）。如果`FeatureAddress`是一个范围（例如“[15:0]”），那么`FeatureValue`宽度必须等于或小于`FeatureAddress`宽度。指定一个比`FeatureAddress`宽的`FeatureValue`是无效的。例如，如果`FeatureAddress`为[15:0]，则地址宽度为16位，而`FeatureValue`必须小于等于16位。因此，`FeatureValue`为`16'hFFFF`是有效的，但`FeatureValue`为`17'h10000`是无效的。

当`FeatureAddress`大于1位时，在启用或禁用该特性之前，将对每个特定地址的`FeatureValue`进行移位和掩码。所以对于一个[7:4]的`FeatureAddress`，地址4的特性被设置为一个值`(FeatureValue >> 0) & 1`，地址5的特征设置值为`(FeatureValue >> 1) & 1`，等等。

如果某个特征的值为1，则输出的比特流必须清除并置位指定的比特位。如果特征的值为0，则不会对默认比特流进行更改。

注意，没有 FASM 特征行并不意味着该特征被设置为0。它只意味着相关的比特是从特定于实现的默认比特流中使用的。