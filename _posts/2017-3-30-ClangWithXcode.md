---
layout:     post
title:      "Clang插件开发与Xcode集成"
subtitle:   ""
date:       2017-03-30
author:     "Chaos"
header-img: "img/post-sad.jpg"
tags:
    - IOS
image:
  feature: 
  teaser: mountains-teaser.jpg
  credit: Death to Stock Photo
  creditlink: ""
---

### 介绍

![](http://s2.mogucdn.com/p2/170118/1_33hljgkec17clg26efh96461cdjgc_1024x1024.png)

Clang作为LLVM提供的编译器前端，将用户的源代码(C/C\++/Objective-C)编译成语言/目标设备无关的IR实现。并且提供良好的插件支持，容许用户在编译时，**运行额外的自定义动作**。

### 自定义插件
**环境搭建**

```shell
 cd /opt

 sudo mkdir llvm

 sudo chown whoami llvm

 cd llvm

 export LLVM_HOME=pwd

 

 git clone -b release_39 git@github.com:llvm-mirror/llvm.git llvm

 git clone -b release_39 git@github.com:llvm-mirror/clang.git llvm/tools/clang

 git clone -b release_39 git@github.com:llvm-mirror/clang-tools-extra.git llvm/tools/clang/tools/extra

 git clone -b release_39 git@github.com:llvm-mirror/compiler-rt.git llvm/projects/compiler-rt

 

 mkdir llvm_build

 cd llvm_build

 cmake ../llvm -DCMAKE_BUILD_TYPE:STRING=Release

 make -jsysctl -n hw.logicalcpu
```

**插件编写[文档](http://clang.llvm.org/docs/ClangPlugins.html)**
> 继承：
> - clang::PluginASTAction(基于consumer的AST前端Action抽象基类)::
> - clang::ASTConsumer(用于客户读取AST的抽象基类)，::
> - clang::RecursiveASTVisitor(前序或后续地深度优先搜索整个AST，并访问每一个节点的基类)，::
>   注册插件：
>   - static FrontendPluginRegistry::Add<MyPlugin> X("my-plugin-name", "my plugin description");::


> 生成插件：
>
> ```shell 
>  clang -std=c++11 -stdlib=libc++ -L/opt/local/lib -
>
>  L/opt/llvm/llvm_build/lib  
>
>  -I/opt/llvm/llvm_build/tools/clang/include -
>
>  I/opt/llvm/llvm_build/include -
>
>  I/opt/llvm/llvm/tools/clang/include -I/opt/llvm/llvm/include -
>
>  dynamiclib -Wl,-headerpad_max_install_names -lclang -
>
>  lclangFrontend -lclangAST -lclangAnalysis -lclangBasic -
>
>  lclangCodeGen -lclangDriver -lclangFrontendTool -lclangLex -
>
>  lclangParse -lclangSema -lclangEdit -lclangSerialization -
>
>  lclangStaticAnalyzerCheckers -lclangStaticAnalyzerCore -
>
>  lclangStaticAnalyzerFrontend -lLLVMX86CodeGen -
>
>  lLLVMX86AsmParser -lLLVMX86Disassembler -lLLVMExecutionEngine 
>
>  -lLLVMAsmPrinter -lLLVMSelectionDAG -lLLVMX86AsmPrinter -
>
>  lLLVMX86Info -lLLVMMCParser -lLLVMCodeGen -lLLVMX86Utils -
>
>  lLLVMScalarOpts -lLLVMInstCombine -lLLVMTransformUtils -
>
>  lLLVMAnalysis -lLLVMTarget -lLLVMCore -lLLVMMC -lLLVMSupport -
>
>  lLLVMBitReader -lLLVMOption -lLLVMProfileData -lpthread -
>
>  lcurses -lz -lstdc++ -fPIC -fno-common -Woverloaded-virtual -
>
>  Wcast-qual -fno-strict-aliasing -pedantic -Wno-long-long -Wall 
>
>  -Wno-unused-parameter -Wwrite-strings -fno-rtti -fPIC 	
>
>  ./printClsPlugin.cpp -o ClangPlugin.dylib
>
> ```
>
> 

**插件使用**
> 编写测试文件：
>
> ```objc
>  #import<UIKit/UIKit.h>
>
>  @interface MyViewController : UIViewController
>
>  @end
>
>  
>
>  @implementation MyViewController
>
>  - (instancetype)init{
>
>  	if(self = [super init]){
>
>  	}
>
>  	return self;
>
>  }
>
> ```
>
> 


> 载入插件，编译测试文件：
>
> ```shell
>  /opt/llvm/llvm_build/bin/clang -isysroot	/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator10.0.sdk -I/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/include/c++/v1 
>
>  -mios-version-min=8.0 -Xclang -load -Xclang 		
>
>  ~/Desktop/ClangPlugin.dylib -Xclang -add-plugin -Xclang 
>
>  ClangPlugin -c ./MyViewController.m
>
> ```
>
> 


> 警告⚠️
>
> ```c 
>  DiagnosticsEngine &diagEngine = context->getDiagnostics();
>
>  unsigned diagID = diagEngine.getCustomDiagID(DiagnosticsEngine::Warning, "Class name should not start with lowercase letter");
>
>  SourceLocation location = declaration->getLocation();
>
>  diagEngine.Report(location, diagID);
> ```
>
> 


> 错误❌
>
> ```c
>  DiagnosticsEngine &diagEngine = context->getDiagnostics();
>
>  unsigned diagID = diagEngine.getCustomDiagID(DiagnosticsEngine::Error, "Class name with _ forbidden");
>
>  SourceLocation location = declaration->getLocation().getLocWithOffset(underscorePos);
>
>  diagEngine.Report(location, diagID);
> ```
>
> 


> 提示
>
> ```
>  FixItHint fixItHint = FixItHint::CreateReplacement(SourceRange(nameStart, nameEnd), replacement);
>
>          
>
>  DiagnosticsEngine &diagEngine = context->getDiagnostics();
>
>  unsigned diagID = diagEngine.getCustomDiagID(DiagnosticsEngine::Warning, "Class name should not start with lowercase letter");
>
>  SourceLocation location = declaration->getLocation();
>
>  diagEngine.Report(location, diagID).AddFixItHint(fixItHint);
> ```
>
> 

**与Xcode集成**
 下载[XcodeHacking](https://github.com/AlexDenisov/ToyClangPlugin/releases/download/0.0.1/XcodeHacking.zip)


 执行命令：
 ``` shell
  sudo mv HackedClang.xcplugin `xcode-select -print-path`/../PlugIns/Xcode3Core.ideplugin/Contents/SharedSupport/Developer/Library/Xcode/Plug-ins
 sudo mv HackedBuildSystem.xcspec `xcode-select -print-path`/Platforms/iPhoneSimulator.platform/Developer/Library/Xcode/Specifications
 ```


 选择编译的Clang:
 ![](http://s2.mogucdn.com/p2/170119/1_70d583eelgj90f02l924j3385b08h_1426x302.png)


 OTHER-CFLAGS增加加参数：
 ![](http://s11.mogucdn.com/p2/170119/1_0601i9fkece91adg3abeh9c0i2fgi_1398x690.png)






























