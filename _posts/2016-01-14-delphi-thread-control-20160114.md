---
layout: post
title: 测试Delphi多线程访问VCL（还包括Delphi弹出框的使用说明）
categories: delphi之多线程 delphi之控件
tags: delphi 多线程 控件 vcl 弹出框
---

首先关于Delphi多线程操作控件可以先看一下这篇文章：[《Delphi：与VCL同步（Synchronize()、用消息来同步）》](http://www.xumenger.com/delphi-vcl-synchronize/)

VCL（Visual Component Library），可视化组件库，包括Graphics、classes、Controls等与类和组件相关的单元

因为VCL不是线程安全的，所以对VCL的访问只能在主线程中。这将意味着：所有需要与用户打交道的代码都只能在主线程的环境中执行。这是其结构上明显的不足，并且这种需求看起来只局限在表面上，但它实际上有一些优点

开发多线程项目的主要需要考虑的一点就是同步多线程使用资源，不要产生冲突，其实想Delphi的VCL组件也是一种资源，但是VCL不是线程安全的，不能让其他的线程使用，只能通过主线程来使用它。

>多线程开发中，子线程和主线程是有很多差别的，很多子线程做不了的事只能主线程来做，所以虽然是多线程，但是子线程和主线程的地位是不同的

其实子线程并不是不能访问VCL，只是这样开发出来的程序的安全性得不到保障，下面我就来展示直接在子线程中操作VCL的几种方式

## 直接在子线程中操作控件

* 直接使用子线程操作控件一般也是没问题，但是弄不清楚哪天就出现一个诡异的问题，所以在开发时一定要规范，否则出现问题时根本没办法找出真实的原因
* 比如是如果做到一个持续性的任务中，那么就不知道在什么时候给你来一个诡异的问题，这个是很难排查的
* 比如通过下面的代码编译出来的程序，如果在子线程持续访问控件的时候去不断拖动主窗体以及反复最大化和最小化，狂虐了程序一段时间后就可能发现窗体出现一些异样：可能是突然控件显示乱七八糟、更常见的是窗体在最大化后再想恢复正常发现没有办法变小了……
* 上面所说的还只是一些显示上的问题、乱象，但是还不知道除此之外还会出现什么诡异的情况：线程异常、线程冲突、内存异常……

不多说，直接上代码（其中Label、Edit控件的autosize都在Object Inspector中设置为False），请重视其中的注释

* 关于线程的控制
* 关于线程对象的释放、线程对象指针为什么要置nil

```
unit Unit1;

interface

uses
  Windows, Messages, SysUtils, Variants, Classes, Graphics, Controls, Forms,
  Dialogs, StdCtrls;

type
  TForm1 = class(TForm)
    lbl1: TLabel;
    btn1: TButton;
    edt1: TEdit;
    procedure btn1Click(Sender: TObject);
  private
    { Private declarations }
  public
    { Public declarations }
  end;

  TTestThread = class(TThread)
  protected
    procedure Execute; override;
  end;

var
  Form1: TForm1;
  testThread: TTestThread;

implementation

{$R *.dfm}

procedure TTestThread.Execute;
begin
  inherited;
  FreeOnTerminate:= True;
  //FreeOnTerminate为True表示线程执行结束后会自动释放线程对象
  try
    while not Self.Terminated do
    begin
      try
        Application.ProcessMessages;
        Form1.lbl1.Left:= 60;
        Form1.lbl1.Top:= 10;
        Form1.lbl1.Caption:= '线程修改Label控件的值';
        Form1.lbl1.Color:= ClYellow;
        Form1.edt1.Text:= 'multiThread';
        Sleep(10);

        Form1.lbl1.Left:= 40;
        Form1.lbl1.Top:= 40;
        Form1.lbl1.Caption:= '啦啦啦啦啦';
        Form1.lbl1.Color:= ClRed;
        Form1.edt1.Text:= 'multiThread Again';
        Sleep(10);
      except
        on E: Exception do
        begin
          Application.MessageBox(PChar(E.Message),'警告',MB_OK);
        end;
      end;
    end;
  finally
    Application.MessageBox('线程结束运行','提示',MB_OK);
  end;
end;

procedure TForm1.btn1Click(Sender: TObject);
begin
  if not Assigned(testThread) then
  begin
    testThread:= TTestThread.Create(True);
    testThread.Resume;
    Self.btn1.Caption:= 'end';
  end
  else if Assigned(testThread) then
  begin
    testThread.Terminate;
    //线程类的Terminate方法是将其Terminated属性置为False，通知线程结束
    //可以看到上面的线程实现中在线程循环中会一直根据Terminated属性来判断是否继续执行，因此可以通过改变其值来通知线程结束
    
    testThread:= nil;
    {线程通过将FreeOnTerminate设置为True，会在线程运行结束后自动释放
          如果这里没有显式将线程对象指针置为nil
          那么在线程释放后该线程对象指针还是指向原来的地址
          那么后面通过Assigned判断线程对象是否被创建就会出现意外
          可以试一下将testThread:= nil;注释掉再看运行的现象，会有问题，请自己动手试一下
    }
    Self.btn1.Caption:= 'start';
  end;
end;

end.
```

## 使用Synchronize来进行同步

还是直接上代码（其中Label、Edit控件的autosize都在Object Inspector中设置为False），上面代码编出来的程序有的问题，我试了下面的程序，并没有出现类似的异样，还是很健壮的。另外请注意其中的注释说明

* 关于几种弹出框在各种情景下使用时候的注意事项
* 关于线程的控制
* 关于在需要使用Synchronize同步的访问控件的方法中来操作控件的注意事项（Change方法中的注释说明）
* 关于线程的释放、线程对象指针为什么要置nil

```
unit Unit1;

interface

uses
  Windows, Messages, SysUtils, Variants, Classes, Graphics, Controls, Forms,
  Dialogs, StdCtrls;

type
  TForm1 = class(TForm)
    btn1: TButton;
    lbl1: TLabel;
    edt1: TEdit;
    procedure btn1Click(Sender: TObject);
  private
    { Private declarations }
  public
    { Public declarations }
  end;

  TTestThread = class(TThread)
  protected
    procedure Execute; override;
    procedure Change;
    procedure ChangeAgain;
  end;

var
  Form1: TForm1;
  testThread: TTestThread;

implementation

{$R *.dfm}

procedure TTestThread.Execute;
begin
  FreeOnTerminate:= True;
  //FreeOnTerminate为True表示线程执行结束后会自动释放线程对象
  try
    while not Self.Terminated do
    begin
      try
        Application.ProcessMessages;
        Synchronize(Change);
        Sleep(10);
        Synchronize(ChangeAgain);
        Sleep(10);
      except
        on E: Exception do
        begin
          //ShowMessage(E.Message);
          Application.MessageBox(PChar(E.Message),'警告',MB_OK);
        end;
      end;
    end;
  finally
    //ShowMessage('线程结束执行');
    //使用ShowMessage会出现一些诡异的现象，比如弹出框变大、边长、没有文字
    //在出现弹出框的情况下还是能够操作主窗体（测试在子线程中有这个问题，主线程没有问题）
    //在子线程中使用ShowMessage弹出框会有一些问题，在主线程中就没有

    //MessageBox(0,'线程结束运行','提示',MB_OK);
    //使用MessageBox在子线程中弹出框也有问题：弹出框后还是可以操作主窗体
    //不过暂时没有发现弹出框变形等异常现象

    Application.MessageBox('线程结束运行','提示',MB_OK);
    //在子线程中使用Application.MessageBox暂时没有发现什么问题
  end;
end;

procedure TTestThread.Change;
begin
  Form1.lbl1.Left:= 60;
  Form1.lbl1.Top:= 10;
  Form1.lbl1.Caption:= '线程修改Label控件的值';
  Form1.lbl1.Color:= ClRed;

  Form1.edt1.Text:= 'Synchronize';

//  Sleep(10);
//  Form1.lbl1.Left:= 40;
//  Form1.lbl1.Top:= 40;
//  Form1.lbl1.Caption:= '啦啦啦啦啦';
//  Form1.lbl1.Color:= ClYellow;
//  Form1.edt1.Text:= 'Synchronize Again';
//  测试在使用Synchronize进行同步的方法中两次操作控件会出现问题，具体可以自己将上面的注释去掉看运行现象
end;

procedure TTestThread.ChangeAgain;
begin
  Form1.lbl1.Left:= 40;
  Form1.lbl1.Top:= 40;
  Form1.lbl1.Caption:= '啦啦啦啦啦';
  Form1.lbl1.Color:= ClYellow;

  Form1.edt1.Text:= 'Synchronize Again';
end;

procedure TForm1.btn1Click(Sender: TObject);
begin
  if not Assigned(testThread) then
  begin
    testThread:= TTestThread.Create(True);
    testThread.Resume;
    Self.btn1.Caption:= 'end';
  end
  else if Assigned(testThread) then
  begin
    testThread.Terminate;
    testThread:= nil;
    {线程通过将FreeOnTerminate设置为True，会在线程运行结束后自动释放
          如果这里没有显式将线程对象指针置为nil
          那么在线程释放后该线程对象指针还是指向原来的地址
          那么后面通过Assigned判断线程对象是否被创建就会出现意外
          可以试一下将testThread:= nil;注释掉再看运行的现象，会有问题，请自己动手试一下
    }
    Self.btn1.Caption:= 'start';

    //ShowMessage('start');
    //测试在主线程中使用ShowMessage是不会出现有了弹出框之后还依然能操作主窗体的问题
  end;
end;

end.
```
