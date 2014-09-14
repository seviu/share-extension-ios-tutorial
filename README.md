**How to write a custom Sharing extension on iOS 8**
----------------------------------------------------

One of the most exciting features of iOS 8 and OS X v10.10 is the ability to extend the functionality of your app through extionsions. Apple has documented this in https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/ExtensibilityPG/index.html

There are 7 types of extensios which range from widgets (today) to photo editing. 

The extension I am going to talk about is the Share extension.

HOI is an application which I developed with good friends (@markiv, @harryf and @moritzadler). It is a nanosocial application that lets you 'Hoi' your friends just by sending push notifications. It is similar to YO, but it has a main difference: it lets you Hoi your friends at a rate of around 3 Hois per second. We did it just for fun. 

We little knew how popular it would become. It has now several thousand users, and hundred of them are active every day.

Soon we started adding features: sending memes, your location, photos... And then extensions and iOS 8 came along.

And we built a share extension.

What is an extension?

One can think about a extension like a less powerful App. It has a storyboard, and a view controller. However it does not have an Application. It also can not access the content of your App.

A share exception that does not have your friends list is a pointless one. Therefore the first thing you do is to enable App Groups, in your Target capabilities:

AppGroupsHOI.png

Once that is done, just open 'File' / 'New Target' / 'Application Extension' and choose 'Share Extension'. Go to its Target capabilities and enable App Groups as well

AppGroupsHOIExtension.png

Xcode created an extension called ShareViewController which extends SLComposeServiceViewController. SLComposeServiceViewController is the default class that is used by iOS to share with facebook or Twitter. It is pretty powerful, but it only lets you post content on a news feed. With Hoi, we must target specific users. We need to show our custom view. 

A share extension lets you show a view controller modally. Therefore we designed a simple table view and put it on top of a transparent view so that our controller shows a glipmpse what is behind it.

If you want to read about how to write a standard Share extension, take a look at this fantastic article from apple:

https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/ExtensibilityPG/ShareSheet.html

Otherwise, and if you like to learn by examples, like I do, just keep on reading.

XcodeInterface.png

Our interface looks like this:

    @interface ShareViewController : UIViewController<UITableViewDelegate, UITableViewDataSource>
    @property (weak, nonatomic) IBOutlet UITableView *tableView;
    @property (weak, nonatomic) IBOutlet UIButton *cancelButton;
    @end

Our table view is a list of friends. We have a utility class that loads them. For that we can use Group Access for the Keychain or NSUserDefaults. For shake of simplicity, lets use NSUserDefaults.

Our app group will be called "group.ch.iphonso.hoi"

When a user uses Hoi, the user name is added to NSUserDefaults, and the friends list will be saved like this:
 

       NSUserDefaults *extensionUserDefaults = [[NSUserDefaults alloc]initWithSuiteName:@"group.ch.iphonso.hoi"];
        [extensionUserDefaults setObject:self.friends forKey:@"HoiFriends"];
        [extensionUserDefaults synchronize];

Loading the friends list is equally easy:
  

     NSUserDefaults *userDefaults = [[NSUserDefaults alloc]initWithSuiteName:@"group.ch.iphonso.hoi"];
       self.friends = [[userDefaults objectForKey:@"HoiFriends"] mutableCopy] ?: [NSMutableArray arrayWithCapacity:10];

Once the extension loads, we give it a nice animation effect, otherwise it will just appear on our screen. We apply a transform on viewWillAppear:

    - (void)viewWillAppear:(BOOL)animated
    {
        [super viewWillAppear:animated];
        self.view.transform = CGAffineTransformMakeTranslation(0, self.view.frame.size.height);
        [UIView animateWithDuration:0.25 animations:^{
            self.view.transform = CGAffineTransformIdentity;
        }];
    }

And a dismiss animation when the extension is dismissed:

    - (IBAction)dismiss
    {
        [UIView animateWithDuration:0.20 animations:^{
            self.view.transform = CGAffineTransformMakeTranslation(0, self.view.frame.size.height);
        } completion:^(BOOL finished) {
            [self.extensionContext completeRequestReturningItems:nil completionHandler:nil];
        }];
    }


That's all the implementation! The dismiss method uses the extension context (NSExtensionContext), which lets you signal the host application with result items. Our app just does a fire and forget action (ending a hoi with a picture or a URL). There is no need to return an object. Calling [self.extensionContext completeRequestReturningItems:nil completionHandler:nil]; signals that the extension has finished doing what it was meant to do.

Info.plist

A share extension needs to say which kind of elements it wants to share. We want to share images and urls. For Hoi we have the following:


    <dict>
    	<key>NSExtensionAttributes</key>
    	<dict>
    		<key>NSExtensionActivationRule</key>
    		<dict>
    			<key>NSExtensionActivationRule</key>
    			<string>TRUEPREDICATE</string>
    			<key>NSExtensionActivationSupportsFileWithMaxCount</key>
    			<integer>0</integer>
    			<key>NSExtensionActivationSupportsImageWithMaxCount</key>
    			<integer>1</integer>
    			<key>NSExtensionActivationSupportsMovieWithMaxCount</key>
    			<integer>0</integer>
    			<key>NSExtensionActivationSupportsText</key>
    			<false/>
    			<key>NSExtensionActivationSupportsWebURLWithMaxCount</key>
    			<integer>1</integer>
    		</dict>
    	</dict>
    	<key>NSExtensionMainStoryboard</key>
    	<string>MainInterface</string>
    	<key>NSExtensionPointIdentifier</key>
    	<string>com.apple.share-services</string>
    </dict>

The keys that can be used in NSExtensionActivationRule are the following big keys:

NSExtensionActivationRule: Share, Action: A dictionary that specifies the semantic data types that a Share extension or Action extension supports. See NSExtensionActivationRule for details.

And kind of content to share:

    NSExtensionActivationSupportsAttachmentsWithMaxCount
    NSExtensionActivationSupportsAttachmentsWithMinCount
    NSExtensionActivationSupportsFileWithMaxCount
    NSExtensionActivationSupportsImageWithMaxCount
    NSExtensionActivationSupportsMovieWithMaxCount
    NSExtensionActivationSupportsText
    NSExtensionActivationSupportsWebURLWithMaxCount
    NSExtensionActivationSupportsWebPageWithMaxCount

And one extra NSExtensionActivationRule that is created by Xcode which defaults to TRUE. It is not documented.

Since with HOI we share images and URLs, we only use `NSExtensionActivationSupportsImageWithMaxCount and NSExtensionActivationSupportsWebURLWithMaxCount`

There is no much more to be said about how to write an extension. On Android we have widgets and url scheme handling. Widgets are pretty boring and not powerful at all. Url scheme handling is pretty powerful, and I doubt iOS will ever support it. Extensions are a kind of mix bag of those, plus some more. I am very looking forward to seeing what kind of extensions will appear when iOS 8 is launched.

And if you want to try Hoi, just download it for iOS and Android from http://gethoi.ch

If you have more questions, you can reach me @seviu in Twitter and HOI

