### UIWebView in VIPER

Main idea - separate `UIWebView` responsibilities into two groups: `WebEngine` which works with Interactor and `WebPresentation` which works with View.

#### WebEngine responsibilities

- HTML loading.
- JavaScript scripts execution.
- Custom content rendering implementation.
- Notification for Interactor about drawing, loading and rendering are finished.

#### WebPresentation responsibilities

- Notifications for View about different user interactions: tap on link, image or video.
- Giving access to interface for receiving information about `UIWebView` as drawing object (i.e. access to content size, etc.).

![Module Scheme](Resources/webview-scheme.png)

#### Module workflow

1. Assembly setup two delegates for `WebEngineImplementation` - View and Interactor.
2. Presenter (or View in case of cell) is module entering point which handle raw html.
3. Presenter receive View's `WebEngine` and send it to Interactor.
4. Presenter send html data to Interactor.
5. Interactor setup `WebEngine` by passing to it few scripts for execution.
6. Interactor send html to `WebEngine`.
7. `WebEngine` notify Interactor about loading data progress.
8. Interactor send this callbacks to Presenter which start further logic.

Every step is rather easy - less than 5 strings. Let's dive in components implementation.

#### `View`

```objc
@interface PostContentCell : UITableViewCell <PostContentViewInput, WebPresentationDelegate>

@property (weak, nonatomic) IBOutlet UIWebView *contentWebView;
@property (strong, nonatomic) id<PostContentViewOutput> output;
@property (strong, nonatomic) id<WebPresentation> webPresentation;

@end

@implementation PostContentCell

- (BOOL)shouldUpdateCellWithObject:(PostContentCellObject *)object {
    [self.output didTriggerModuleSetupEventWithHtmlContent:object.htmlContent];

    return YES;
}

#pragma mark - PostContentViewInput

- (void)setupInitialState {
    [self.webPresentation setupWithWebView:self.contentWebView];
}

#pragma mark - WebPresentationDelegate

- (void)webPresentation:(id<WebPresentation>)webPresentation
             didTapLink:(NSURL *)link {
    [self.output didTriggerLinkTapEventWithURL:link];
}

- (void)webPresentation:(id<WebPresentation>)webPresentation
    didTapImageWithLink:(NSURL *)link {
    [self.output didTriggerImageTapWithURL:link];
}

@end
```

#### `PostContentPresenter`

```objc
@implementation PostContentPresenter

#pragma mark - PostContentViewOutput

- (void)didTriggerModuleSetupEventWithHtmlContent:(NSString *)htmlContent {
    self.rawHtmlContent = htmlContent;

    [self.interactor renderContent:self.rawHtmlContent];
}

- (void)didTriggerLinkTapEventWithURL:(NSURL *)url {
    [self.router showBrowserModuleWithURL:url];
}

- (void)didTriggerImageTapWithURL:(NSURL *)url {
    [self.router showFullscreenImageModuleWithImageURL:url];
}

#pragma mark - PostContentInteractorOutput

- (void)didCompleteRenderingContent {
    CGFloat contentHeight = [self.view obtainCurrentContentHeight];
    [self.moduleOutput didUpdatePostContentWithContentHeight:contentHeight];
}

@end
```

#### `PostContentInteractor`

```objc
@interface PostContentInteractor : NSObject <PostContentInteractorInput, WebEngineDelegate>

@property (nonatomic, weak) id<PostContentInteractorOutput> output;

@property (nonatomic, strong) id<WebEngine> webEngine;
@property (nonatomic, strong) ContentHTMLComposer *contentComposer;
@property (nonatomic, strong) JSScriptProvider *scriptProvider;
@property (nonatomic, strong) ContentRenderStrategyFactory *renderStrategyFactory;

@end

@implementation PostContentInteractor

#pragma mark - PostContentInteractorInput

- (void)renderContent:(NSString *)content {
    NSArray *scripts = [self.scriptProvider obtainScriptsForPostEnvironmentSetup];
    for (NSString *script in scripts) {
        [self.webEngine executeScript:script];
    }

    NSString *processedContent = [self.contentComposer composeValidHtmlForRenderingFromRawHtml:content];

    [self.webEngine loadHtml:processedContent];
}

#pragma mark - WebEngineDelegate

- (void)didCompleteLoadingContentWithWebEngine:(id<WebEngine>)webEngine {
    ContentRenderStrategy *renderStrategy = [self.renderStrategyFactory postContentRenderStrategy];
    [self.webEngine renderContentWithStrategy:renderStrategy];
}

- (void)didCompleteRenderingContentWithWebEngine:(id<WebEngine>)webEngine {
    [self.output didCompleteRenderingContent];
}

@end
```

#### `WebEngineImplementation`

```objc
@interface WebEngineImplementation : NSObject <WebPresentation, WebEngine>

@property (weak, nonatomic) id<WebEngineDelegate> webEngineDelegate;
@property (weak, nonatomic) id<WebPresentationDelegate> webPresentationDelegate;
@property (strong, nonatomic) WebBridgeFactory *bridgeFactory;

@end

@implementation WebEngineImplementation

#pragma mark - WebPresentation

- (void)setupWithWebView:(UIWebView *)webView {
    self.webView = webView;

    self.bridge = [self.bridgeFactory obtainBridgeForWebView:webView
                                             webViewDelegate:self];

    [self setupJavascriptHandlers];
}

#pragma mark - Handlers Setup

- (void)setupJavascriptHandlers {
    @weakify(self);
    [self.bridge registerHandler:kImageTapHandler
                         handler:^(id data, WVJBResponseCallback responseCallback) {
                             @strongify(self);
                             NSString *imageURLString = data[kDataLinkKey];
                             NSURL *imageURL = [NSURL URLWithString:imageURLString];
                             [self.webPresentationDelegate webPresentation:self
                                                       didTapImageWithLink:imageURL];
                         }];
}

#pragma mark - WebEngine

- (void)loadHtml:(NSString *)html {
    [self.webView loadHTMLString:html
                         baseURL:nil];
}

- (void)executeScript:(NSString *)script {
    [self.webView stringByEvaluatingJavaScriptFromString:script];
}

- (void)renderContentWithStrategy:(ContentRenderStrategy *)renderStrategy {
    dispatch_group_t group = dispatch_group_create();
    JSResponseBlock responseBlock = ^(NSDictionary *responseData) {
        NSString *logDescription = responseData[JSLogDescriptionKey];
        DDLogVerbose(@"%@", logDescription);
        dispatch_group_leave(group);
    };

    for (NSString *handlerKey in renderStrategy.scriptNames) {
        dispatch_group_enter(group);
        [self.bridge callHandler:handlerKey
                            data:renderStrategy.scriptPayloads[handlerKey]
                responseCallback:responseBlock];
    }

    [self.webEngineDelegate didCompleteRenderingContentWithWebEngine:self];
}

#pragma mark - UIWebViewDelegate

- (void)webViewDidFinishLoad:(UIWebView *)webView {
    [self.webEngineDelegate didCompleteLoadingContentWithWebEngine:self];
}

- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType {
    NSString *urlString = request.URL.absoluteString;

    if (navigationType == UIWebViewNavigationTypeLinkClicked) {
        [self.webPresentationDelegate webPresentation:self
                                           didTapLink:request.URL];
        return NO;
    }

    return YES;
}
```

#### Summary

Besides separation of responsibilities we hide fact about third party dependencies for connecting native code and JavaScript. It's just `WebEngine` protocol implementation details. Unit tests for all this components really easy. All you have to check is data flow correctness.