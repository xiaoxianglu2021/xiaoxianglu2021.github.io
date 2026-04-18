---
layout:     post
title:      BYPhoneNumTF
subtitle:   一个电话号码格式的文本�?
date:       2017-02-04
author:     BY
header-img: img/init/post-bg-ios9-web.jpg
catalog: true
tags:
    - iOS
    - 轮子
---

>**BYPhoneNumTF** 一个电话号码格式的文本�?

# 功能

当在`TextField`输入数字时，会自动分隔为�?37 9922 2299 �?137-9922-2299

限制文本输入个数

限制只能输入数字
# 效果�?

![](http://ww2.sinaimg.cn/large/7853084cgw1fa3cqnu8s2g207i0dc4qp.gif)


# 实现方法

要实现电话号码格式的输入看似简单，但是实现起来坑非常多，至于坑是什么只有各位动手写了才能体会~

下面我们来实现该功能�?

首先要遵守协议`<UITextFieldDelegate>`

然后在`- (BOOL)textField:(UITextField *)textField shouldChangeCharactersInRange:(NSRange)range replacementString:(NSString *)string` 方法中实现我们的逻辑

代码�?


```
#import "LoginVC.h"

#define placeholder @" "

@interface LoginVC () <UITextFieldDelegate>

@property (weak, nonatomic) IBOutlet UITextField *phoneNumberTF;

@end


- (BOOL)textField:(UITextField *)textField shouldChangeCharactersInRange:(NSRange)range replacementString:(NSString *)string {

    NSString *phStr = placeholder;
    unichar phChar = ' ';
    if (phStr.length) {
        phChar = [phStr characterAtIndex:0];
    }
    
    
    if (textField) {
        NSString* text = textField.text;
        //删除
        if([string isEqualToString:@""]){
            
            //删除一�?
            if(range.length == 1){
                //最后一�?遇到空格则多删除一�?
                if (range.location == text.length - 1 ) {
                    if ([text characterAtIndex:text.length - 1] == phChar) {
                        [textField deleteBackward];
                    }
                    return YES;
                }
                //从中间删�?
                else{
                    NSInteger offset = range.location;
                    
                    if (range.location < text.length && [text characterAtIndex:range.location] == phChar && [textField.selectedTextRange isEmpty]) {
                        [textField deleteBackward];
                        offset --;
                    }
                    [textField deleteBackward];
                    textField.text = [self _parseString:textField.text];
                    UITextPosition *newPos = [textField positionFromPosition:textField.beginningOfDocument offset:offset];
                    textField.selectedTextRange = [textField textRangeFromPosition:newPos toPosition:newPos];
                    return NO;
                }
            }
            else if (range.length > 1) {
                BOOL isLast = NO;
                //如果是从最后一位开�?
                if(range.location + range.length == textField.text.length ){
                    isLast = YES;
                }
                [textField deleteBackward];
                textField.text = [self _parseString:textField.text];
                
                NSInteger offset = range.location;
                if (range.location == 3 || range.location  == 8) {
                    offset ++;
                }
                if (isLast) {
                    //光标直接在最后一位了
                }else{
                    UITextPosition *newPos = [textField positionFromPosition:textField.beginningOfDocument offset:offset];
                    textField.selectedTextRange = [textField textRangeFromPosition:newPos toPosition:newPos];
                }
                
                return NO;
            }
            
            else{
                return YES;
            }
        }
        
        else if(string.length >0){
            
            //限制输入字符个数
            if (([self _noneSpaseString:textField.text].length + string.length - range.length > 11) ) {
                return NO;
            }
            
            //判断是否是纯数字(搜狗，百度输入法，数字键盘居然可以输入其他字�?
            if(![self _isNum:string]){
                return NO;
            }
            [textField insertText:string];
            textField.text = [self _parseString:textField.text];
            
            NSInteger offset = range.location + string.length;
            if (range.location == 3 || range.location  == 8) {
                offset ++;
            }
            UITextPosition *newPos = [textField positionFromPosition:textField.beginningOfDocument offset:offset];
            textField.selectedTextRange = [textField textRangeFromPosition:newPos toPosition:newPos];
            return NO;
        }else{
            return YES;
        }
        
    }
    
    return YES;
    

}

- (NSString*)_parseString:(NSString*)string{
    
    if (!string) {
        return nil;
    }
    NSMutableString* mStr = [NSMutableString stringWithString:[string stringByReplacingOccurrencesOfString:placeholder withString:@""]];
    if (mStr.length >3) {
        [mStr insertString:placeholder atIndex:3];
    }if (mStr.length > 8) {
        [mStr insertString:placeholder atIndex:8];
        
    }
    
    return  mStr;
    
}

/** 获取正常电话号码（去掉空格） */
- (NSString*)_noneSpaseString:(NSString*)string{
    
    return [string stringByReplacingOccurrencesOfString:placeholder withString:@""];
    
}

- (BOOL)_isNum:(NSString *)checkedNumString {
    
    if (!checkedNumString) {
        return NO;
    }
    
    checkedNumString = [checkedNumString stringByTrimmingCharactersInSet:[NSCharacterSet decimalDigitCharacterSet]];
    
    if(checkedNumString.length > 0) {
        return NO;
    }
    
    return YES;
    
}

```


###封装方法

需要实现的代码就是要这么多，但这些代码写在ViewController显得太臃肿了，所以我对代码进行了封装:

```
//
//  BYPhoneNumTF.h
//
//  Created by BY on 16/12/2.
//  Copyright © 2016�?BY. All rights reserved.
//  电话号码类型的文本输入框，且只能输入数字
//  输入显示�?37 9922 1234 �?137-9922-1234
//  使用方法：在XIB中的TextField继承该类即可
//  修改占位符placeholder即可改变样式

#import <UIKit/UIKit.h>

// @" " or @"-"
#define placeholder @" "

@interface BYPhoneNumTF : UITextField

/** 去掉格式的电话号�?*/
@property (nonatomic, strong) NSString *plainPhoneNum;

@end
```

```
//
//  BYPhoneNumTF.m
//  dev-Jack
//
//  Created by BY on 16/12/2.
//  Copyright © 2016�?Jack. All rights reserved.
//

#import "BYPhoneNumTF.h"


@interface BYPhoneNumTF ()<UITextFieldDelegate>

@end

@implementation BYPhoneNumTF

- (NSString *)plainPhoneNum {
    return [self _noneSpaseString:self.text];
}

- (void)awakeFromNib {
    [super awakeFromNib];
    self.delegate = self;
}


- (BOOL)textField:(UITextField *)textField shouldChangeCharactersInRange:(NSRange)range replacementString:(NSString *)string {

    NSString *phStr = placeholder;
    unichar phChar = ' ';
    if (phStr.length) {
        phChar = [phStr characterAtIndex:0];
    }
    
    
    if (textField) {
        NSString* text = textField.text;
        //删除
        if([string isEqualToString:@""]){
            
            //删除一�?
            if(range.length == 1){
                //最后一�?遇到空格则多删除一�?
                if (range.location == text.length - 1 ) {
                    if ([text characterAtIndex:text.length - 1] == phChar) {
                        [textField deleteBackward];
                    }
                    return YES;
                }
                //从中间删�?
                else{
                    NSInteger offset = range.location;
                    
                    if (range.location < text.length && [text characterAtIndex:range.location] == phChar && [textField.selectedTextRange isEmpty]) {
                        [textField deleteBackward];
                        offset --;
                    }
                    [textField deleteBackward];
                    textField.text = [self _parseString:textField.text];
                    UITextPosition *newPos = [textField positionFromPosition:textField.beginningOfDocument offset:offset];
                    textField.selectedTextRange = [textField textRangeFromPosition:newPos toPosition:newPos];
                    return NO;
                }
            }
            else if (range.length > 1) {
                BOOL isLast = NO;
                //如果是从最后一位开�?
                if(range.location + range.length == textField.text.length ){
                    isLast = YES;
                }
                [textField deleteBackward];
                textField.text = [self _parseString:textField.text];
                
                NSInteger offset = range.location;
                if (range.location == 3 || range.location  == 8) {
                    offset ++;
                }
                if (isLast) {
                    //光标直接在最后一位了
                }else{
                    UITextPosition *newPos = [textField positionFromPosition:textField.beginningOfDocument offset:offset];
                    textField.selectedTextRange = [textField textRangeFromPosition:newPos toPosition:newPos];
                }
                
                return NO;
            }
            
            else{
                return YES;
            }
        }
        
        else if(string.length >0){
            
            //限制输入字符个数
            if (([self _noneSpaseString:textField.text].length + string.length - range.length > 11) ) {
                return NO;
            }
            
            //判断是否是纯数字(搜狗，百度输入法，数字键盘居然可以输入其他字�?
            if(![self _isNum:string]){
                return NO;
            }
            [textField insertText:string];
            textField.text = [self _parseString:textField.text];
            
            NSInteger offset = range.location + string.length;
            if (range.location == 3 || range.location  == 8) {
                offset ++;
            }
            UITextPosition *newPos = [textField positionFromPosition:textField.beginningOfDocument offset:offset];
            textField.selectedTextRange = [textField textRangeFromPosition:newPos toPosition:newPos];
            return NO;
        }else{
            return YES;
        }
        
    }
    
    return YES;
    

}

- (NSString*)_parseString:(NSString*)string{
    
    if (!string) {
        return nil;
    }
    NSMutableString* mStr = [NSMutableString stringWithString:[string stringByReplacingOccurrencesOfString:placeholder withString:@""]];
    if (mStr.length >3) {
        [mStr insertString:placeholder atIndex:3];
    }if (mStr.length > 8) {
        [mStr insertString:placeholder atIndex:8];
        
    }
    
    return  mStr;
    
}

/** 获取正常电话号码（去掉空格） */
- (NSString*)_noneSpaseString:(NSString*)string{
    
    return [string stringByReplacingOccurrencesOfString:placeholder withString:@""];
    
}

- (BOOL)_isNum:(NSString *)checkedNumString {
    
    if (!checkedNumString) {
        return NO;
    }
    
    checkedNumString = [checkedNumString stringByTrimmingCharactersInSet:[NSCharacterSet decimalDigitCharacterSet]];
    
    if(checkedNumString.length > 0) {
        return NO;
    }
    
    return YES;
    
}


@end

```

# 使用方法

在storyboard中的`TextField`控件的Calss类型选择该类`BYPhoneNumTF`即可�?

代码及Demo下载地址：[BYPhoneNumTF](https://github.com/qiubaiying/BYPhoneNumTF)
