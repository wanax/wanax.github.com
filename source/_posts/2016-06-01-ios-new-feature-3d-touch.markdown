---
layout: post
title: "iOS New Feature -- 3D Touch"
date: 2016-06-01 14:39
comments: true
categories: Tec
---

###一. 功能介绍

3D Touch 是苹果公司最新推出的一种应用在手机上的技术，基于iOS9系统与6s设备，它拓展了手机的操作维度，由平面的两维空间空间拓展为立体的三维空间。针对手机屏幕，在纵向上对压按进行响应，由此衍生出更多的操作场景。

>	 With iOS 9, new iPhone models add a third dimension to the user interface.
>	 
>	 * A user can now press your Home screen icon to immediately access functionality provided by your app.>
>	 
>	* Within your app, a user can now press views to see previews of additional content and gain accelerated access to features.

<!--more-->

根据苹果官方文档[Getting Started with 3D Touch](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/Adopting3DTouchOniPhone/index.html#//apple_ref/doc/uid/TP40016543-CH1-SW1)介绍，3D touch可增加以下三种UI新体验：
 
####1. Home Screen Quick Actions

按压Home screen上特定App图标，提供快捷方式跳转去App的不同功能场景。

![image](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/Adopting3DTouchOniPhone/Art/maps_directions_home_2x.png)

####2. Peek and Pop

App运行中时，按压某个View提供预览功能。

> You can also enable peek and pop for links in web views, as described in [Web View Peek and Pop](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/Adopting3DTouchOniPhone/3DTouchAPIs.html#//apple_ref/doc/uid/TP40016543-CH4-SW5).

![image](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/Adopting3DTouchOniPhone/Art/peek_2x.png)

####3. Force Properties

> In iOS 9, the `UITouch` class has two new properties to support custom implementation of 3D Touch in your app: [force](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UITouch_Class/index.html#//apple_ref/occ/instp/UITouch/force) and [maximumPossibleForce](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UITouch_Class/index.html#//apple_ref/occ/instp/UITouch/maximumPossibleForce). For the first time on iOS devices, these properties let you detect and respond to touch pressure in the `UIEvent` objects your app receives.

> The force of a touch has a high dynamic range, available as a floating point value to your app.

###二. 特性实现介绍

App内有统一接口判断当前设备与系统3D Touch是否可用。

```
override func viewDidLoad() {
    super.viewDidLoad()
    if traitCollection.forceTouchCapability == .Available {
        registerForPreviewingWithDelegate(self, sourceView: view)
    }
    else {
        // Create an alert to display to the user.
        alertController = UIAlertController(title: "3D Touch Not Available", message: "Unsupported device.", preferredStyle: .Alert)
    }
}
```

####1. Home Screen Quick Actions

基本数据结构：`UIApplicationShortcutItem`

```
public class UIApplicationShortcutItem : NSObject, NSCopying, NSMutableCopying {
    public init(type: String, localizedTitle: String, localizedSubtitle: String?, icon: UIApplicationShortcutIcon?, userInfo: [NSObject : AnyObject]?)
    public convenience init(type: String, localizedTitle: String)   
    // An application-specific string that identifies the type of action to perform.
    public var type: String { get }
    // Properties controlling how the item should be displayed on the home screen.
    public var localizedTitle: String { get }
    public var localizedSubtitle: String? { get }
    @NSCopying public var icon: UIApplicationShortcutIcon? { get }
    // Application-specific information needed to perform the action.
    // Will throw an exception if the NSDictionary is not plist-encodable.
    public var userInfo: [String : NSSecureCoding]? { get }
}
```

`UIApplicationShortcutItemType`与`UIApplicationShortcutItemTitle`是必填项，其它选填。

*注意事项：*

1. 快捷标签最多可以创建四个，包括静态的和动态的。
2. 每个标签的题目和icon最多两行，多出的会用...省略

添加`ShortcutItem`有两种方式：

* **Static quick actions**

	直接在App的系统plist文件里添加
	
	![image](https://developer.apple.com/library/ios/documentation/General/Reference/InfoPlistKeyReference/Art/UIApplicationShortcutItems_plist_editor_2x.png =700x250)

* **Dynamic quick actions**

	```
	lazy var dynamicShortcuts = UIApplication.sharedApplication().shortcutItems ?? []
	```
	
	```
	shortcutItem = UIApplicationShortcutItem(type: selectedShortcutItem.type, localizedTitle: titleTextField.text ?? "", localizedSubtitle: subtitleTextField.text, icon: icon, userInfo: [
	                    AppDelegate.applicationShortcutUserInfoIconKey: pickerView.selectedRowInComponent(0)
	                ]
	            )
	```
	
	```
	dynamicShortcuts[selected.row] = updatedShortcutItem
	// Update the application's `shortcutItems`.
	UIApplication.sharedApplication().shortcutItems = dynamicShortcuts
	```
	
点击后的事件响应在App delegate里处理：

```
    /* 
        Called when the user activates your application by selecting a shortcut on the home screen, except when 
        application(_:,willFinishLaunchingWithOptions:) or application(_:didFinishLaunchingWithOptions) returns `false`.
        You should handle the shortcut in those callbacks and return `false` if possible. In that case, this 
        callback is used if your application is already launched in the background.
    */
    func application(application: UIApplication, performActionForShortcutItem shortcutItem: UIApplicationShortcutItem, completionHandler: Bool -> Void) {
        let handledShortCutItem = handleShortCutItem(shortcutItem)
        completionHandler(handledShortCutItem)
    }
```

####2. Peek and Pop

关键数据结构：`UIPreviewAction`

```
public class UIPreviewAction : NSObject, NSCopying, UIPreviewActionItem {   
    public var handler: (UIPreviewActionItem, UIViewController) -> Void { get }
    public convenience init(title: String, style: UIPreviewActionStyle, handler: (UIPreviewAction, UIViewController) -> Void)
}
public class UIPreviewActionGroup : NSObject, NSCopying, UIPreviewActionItem {
    public convenience init(title: String, style: UIPreviewActionStyle, actions: [UIPreviewAction])
}
```

```
lazy var previewActions: [UIPreviewActionItem] = {
        func previewActionForTitle(title: String, style: UIPreviewActionStyle = .Default) -> UIPreviewAction {
            return UIPreviewAction(title: title, style: style) { previewAction, viewController in
                guard let detailViewController = viewController as? DetailViewController,
                          sampleTitle = detailViewController.sampleTitle else { return }
                print("\(previewAction.title) triggered from `DetailViewController` for item: \(sampleTitle)")
            }
        }
        let action1 = previewActionForTitle("Default Action")
        let action2 = previewActionForTitle("Destructive Action", style: .Destructive)
        let subAction1 = previewActionForTitle("Sub Action 1")
        let subAction2 = previewActionForTitle("Sub Action 2")
        let groupedActions = UIPreviewActionGroup(title: "Sub Actions…", style: .Default, actions: [subAction1, subAction2] )
        return [action1, action2, groupedActions]
    }()
```
通过重载

```
func previewingContext(previewingContext: UIViewControllerPreviewing, viewControllerForLocation location: CGPoint) -> UIViewController?
```
生成对应的快速预览图，本身重载

```
override func previewActionItems() -> [UIPreviewActionItem]
```
来生成下一步动作。

####3. Force Properties

在`UITouch`对象中新增了`force`与`maximumPossibleForce`，可以根据`force`定制化自己的需求

```
public class UITouch : NSObject {
    // Force of the touch, where 1.0 represents the force of an average touch
    @available(iOS 9.0, *)
    public var force: CGFloat { get }
    // Maximum possible force with this input mechanism
    @available(iOS 9.0, *)
    public var maximumPossibleForce: CGFloat { get }
}
```
































