### UIWebView in VIPER

The fundamental idea is to separate `UIWebView` responsibilities in two groups: `WebEngine`, which is a dependency of the Interactor, and `WebPresentation` which belongs to the View.

#### WebEngine responsibilities

- Loads HTML code.
- Executes JavaScript.
- Renders content with a set of predefined actions.
- Notifies the Interactor of drawing, loading and rendering processes completion.

#### WebPresentation responsibilities

- Notifies the View of different events: links, images and different embeds taps.
- Provides an interface of obtaining real content size and other view properties.

![Module Scheme](../Resources/webview-scheme.png)

#### Module workflow

1. The Assembly setups two delegates for the `WebEngineImplementation` object - one is the View and another is the Interactor.
2. The Presenter (or the View in case of a cell) is an entry point of the module. It's only input is raw html data.
3. The Presenter receives View's `WebEngine` and passes it to the Interactor.
4. The Presenter passes html data to the Interactor.
5. The Interactor setups the `WebEngine` environment by executing a number of JavaScript scripts.
6. The Interactor passes html data to the `WebEngine`.
7. `WebEngine` notifies the Interactor on rendering completion.
8. The Interactor notifies the Presenter, which obtains the content size from the View and passes it to the module output.

These are very simple steps - each method consists of no more than 5 lines of code.

Let's investigate the implementation of each component.

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

Besides the clear separation of concerns, we've also hidden the fact of using a third party library for bridging - it's just an implementation detail of our `WebEngine`. Unit tests for this setup are easy and reliable - all that we have to test is the data flow.
