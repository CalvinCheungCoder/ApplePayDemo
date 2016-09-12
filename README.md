# ApplePayDemo
ApplePay详细集成

实现效果：

![](https://github.com/CalvinCheungCoder/ApplePayDemo/blob/master/ApplePayDemo.gif)

1.在Apple开发者中心配置 AppleID 和 Merchant IDs

![](https://github.com/CalvinCheungCoder/ApplePayDemo/blob/master/ApplePay01.png)

![](https://github.com/CalvinCheungCoder/ApplePayDemo/blob/master/ApplePay02.png)

2.配置好证书后在Xcode中开启Apple Pay

![](https://github.com/CalvinCheungCoder/ApplePayDemo/blob/master/ApplePay03.png)

3.主要代码实现

3.1 判断是否支持Apple Pay，如果支持又将支持哪些银行卡。

```
    // 判断是否支持Apple Pay
    if (![PKPaymentAuthorizationViewController canMakePayments]) {
        
        NSLog(@"不支持Apple Pay");
        
        // 如果没有绑定VISA或者银联卡，点击按钮去绑定银行卡
    }else if (![PKPaymentAuthorizationViewController canMakePaymentsUsingNetworks:@[PKPaymentNetworkVisa,PKPaymentNetworkChinaUnionPay]]){
        
        PKPaymentButton *addPayBtn = [PKPaymentButton buttonWithType:PKPaymentButtonTypeBuy style:PKPaymentButtonStyleWhiteOutline];
        [addPayBtn addTarget:self action:@selector(addPay) forControlEvents:UIControlEventTouchUpInside];
        addPayBtn.center = self.view.center;
        [self.view addSubview:addPayBtn];
        
        // 如果存在VISA或者银联卡，点击按钮去支付
    }else{
        
        PKPaymentButton *payBtn = [PKPaymentButton buttonWithType:PKPaymentButtonTypeBuy style:PKPaymentButtonStyleBlack];
        [payBtn addTarget:self action:@selector(buy) forControlEvents:UIControlEventTouchUpInside];
        payBtn.center = self.view.center;
        [self.view addSubview:payBtn];
    }
    
```

3.2 如果没有绑定VISA卡或者银联卡时，点击支付按钮会跳转到绑卡页面

```
// 添加银行卡
-(void)addPay{
    
    PKPassLibrary *pay = [[PKPassLibrary alloc]init];
    [pay openPaymentSetup];
}

```
3.3 如果已经绑定银行卡，则去支付，并设置支付信息和商户信息

```
// 支付
-(void)buy{
    
    // 1.创建支付请求
    PKPaymentRequest *request = [[PKPaymentRequest alloc]init];
    // 商户号
    request.merchantIdentifier = @"merchant.com.zhangdinghao.ApplePay";
    // 货币代码和国家代码
    request.countryCode = @"CN";
    request.currencyCode = @"CNY";
    // 请求支付的网络（和之前判断的网络保持一致）
    request.supportedNetworks = @[PKPaymentNetworkVisa,PKPaymentNetworkChinaUnionPay];
    // 商户处理方式
    request.merchantCapabilities = PKMerchantCapability3DS;
    // 商品1
    NSDecimalNumber *price = [NSDecimalNumber decimalNumberWithString:@"7999.00"];
    PKPaymentSummaryItem *item = [PKPaymentSummaryItem summaryItemWithLabel:@"iPhone 7" amount:price];
    // 商品2
    NSDecimalNumber *price2 = [NSDecimalNumber decimalNumberWithString:@"149.00"];
    PKPaymentSummaryItem *item2 = [PKPaymentSummaryItem summaryItemWithLabel:@"iPhone数据线" amount:price2];
    // 商品汇总
    NSDecimalNumber *price3 = [NSDecimalNumber decimalNumberWithString:@"8148.00"];
    PKPaymentSummaryItem *item3 = [PKPaymentSummaryItem summaryItemWithLabel:@"Apple Store" amount:price3];
    
    request.paymentSummaryItems = @[item,item2,item3];
    // 账单或者发票接收地址
    request.requiredBillingAddressFields = PKAddressFieldAll;
    // 快递地址
    request.requiredShippingAddressFields = PKAddressFieldAll;
    
    NSDecimalNumber *kuaidi = [NSDecimalNumber decimalNumberWithString:@"0.00"];
    PKShippingMethod *method = [PKShippingMethod summaryItemWithLabel:@"顺丰" amount:kuaidi];
    method.identifier = @"shangmen";
    method.detail = @"72小时内送货上门";
    request.shippingMethods = @[method];
    
    // 2.验证支付
    PKPaymentAuthorizationViewController *avc = [[PKPaymentAuthorizationViewController alloc]initWithPaymentRequest:request];
    avc.delegate = self;
    [self presentViewController:avc animated:YES completion:nil];
    
}

```

3.4 实现支付的代理方法

```
-(void)paymentAuthorizationViewController:(PKPaymentAuthorizationViewController *)controller didAuthorizePayment:(PKPayment *)payment completion:(void (^)(PKPaymentAuthorizationStatus))completion{
    
    // 拿到支付信息发送给服务器，服务器处理完成后返回支付状态
    BOOL isSucess = YES;
    
    if (isSucess) {
        completion(PKPaymentAuthorizationStatusSuccess);
    }else{
        
        completion(PKPaymentAuthorizationStatusFailure);
    }
}

-(void)paymentAuthorizationViewControllerDidFinish:(PKPaymentAuthorizationViewController *)controller{
    
    [self dismissViewControllerAnimated:controller completion:nil];
}

```
