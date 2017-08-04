# Mobile application development guide with the iOS API

## Table of contents

<!-- MarkdownTOC depth="3" -->

- [Who is this Guide for?](#who-is-this-guide-for)
- [General information about iOS and accessibility](#general-information-about-ios-and-accessibility)
  - [Accessibility in iOS](#accessibility-in-ios)
  - [The UI Accessibility API](#the-ui-accessibility-api)
  - [Attributes for accessibility](#attributes-for-accessibility)
- [Use the basic elements provided by UIKit](#use-the-basic-elements-provided-by-uikit)
  - [Compatible Widgets](#compatible-widgets)
  - [Incompatible widgets](#incompatible-widgets)
- [Test for accessibility](#test-for-accessibility)
- [Declare the accessibility of views](#declare-the-accessibility-of-views)
  - [Individual views](#individual-views)
  - [The "container" views](#the-container-views)
- [Describe interface elements](#describe-interface-elements)
- [Identify interface elements traits](#identify-interface-elements-traits)
- [Frame the rendering area of ​​an item](#frame-the-rendering-area-of-%E2%80%8B%E2%80%8Ban-item)
- [Update descriptions and traits](#update-descriptions-and-traits)
- [Get information about the status of accessibility options](#get-information-about-the-status-of-accessibility-options)
- [Send notifications to assistive tools](#send-notifications-to-assistive-tools)
- [Capture and process notifications sent by UIKit](#capture-and-process-notifications-sent-by-uikit)
- [Some particular cases](#some-particular-cases)
  - [Modals](#modals)
  - [Table views \(TableView\)](#table-views-tableview)
- [Related documents](#related-documents)
- [External resources and references](#external-resources-and-references)
- [Licence](#licence)

<!-- /MarkdownTOC -->

## Who is this Guide for?

This guide presents elements of the iOS API useful for developing applications accessible to people with disabilities. It is aimed at:

* Developers
* Designers responsible for drafting technical specifications
* Project Managers

Prerequisites:

* Proficiency in the Objective C language
* Proficiency in iOS programming principles
* Mastering the concepts used to manage user interfaces on iOS
* Knowing how to use XCode

## General information about iOS and accessibility

### Accessibility in iOS
The iOS API provides developers with features to create accessible applications. These features are present in the classes that allow the creation of the basic elements of a user interface as well as in classes dedicated to accessibility.

iOS applications can be used by people with disabilities (visual, motor, etc.), by enabling accessibility features on the device used to run applications, and sometimes by using external devices (keyboards, switches, braille displays, etc.). The features built directly into iOS are for example:

* Zoom: possibility to magnify the contents on screen
* Display in white on black background (color inversion)
* Audio output in mono only: mixes both channels of a sound signal into one
* Auto-completion: pronounces suggestions for corrections during text input by the user
* Voice control: allows a user to access certain features of the device through vocal input
* Screen reader: iOS comes with VoiceOver, that allows to control the device without seeing the screen (text-to speech output, redefinition of gestures, driving of a Braille display, etc.)

The accessibility features can be activated by going to the "Settings" menu of the device, then in the "Accessibility" tab.

The simple activation of accessibility features is not enough to make an application fully accessible: the developer must implement special techniques when coding the application to make its content accessible by software that supports accessibility.

In iOS, an application is said to be accessible when all the elements of its interface with which the user can interact are accessible, that is to say when each element declares itself as an element providing accessibility. In addition, an item must have accurate and relevant information that can be interpreted by software that supports accessibility: screen position, name, behavior, value, and type.

### The UI Accessibility API

Since version 3.0, iOS has an API dedicated to accessibility ("UI Accessibility"). This API provides software that supports accessibility with all the information needed to describe the user interface of an application.

The UI Accessibility API is part of UIKit. Its features are implemented by default in iOS basic user interface views and elements. Thus, accessible applications can easily be created when these basic elements are used. If more complex graphical interfaces are to be created (by using custom views), the efforts to make them accessible will be more important.

The UI Accessibility API consists of two protocols, a class, a function, and a set of constants:

* The `UIAccessibility` protocol: Objects that implement this protocol carry information about their accessibility (i.e. whether they declare being accessible or not) and provide information describing them. The basic views and elements implement this protocol by default.
* The `UIAccessibilityContainer` protocol: this protocol allows a subclass of UIView to make all or part of the objects it contains accessible as separate elements. This is useful when objects in a view are not themselves UIView subclasses that are not automatically considered accessible.
* The `UIAccessibilityElement` class: This class defines an object that can be returned via the `UIAccessibilityContainer` protocol. An instance of `UIAccessibilityContainer` can be created to represent an element that is not automatically considered to be accessible (for example, an object that does not inherit from UIView).
* Constants (defined in `UIAccessibilityConstants.h`): these are the characteristics exposed by an element and the notifications that an application can send.

### Attributes for accessibility

Controls and views contain attributes that provide information about their accessibility to assistive technologies. These are for example:

* `isAccessibilityElement`: indicates whether the item is accessible.
* `accessibilityLabel`: gives a short description of an element that allows to understand its content or purpose. This description should not include information about the item type. Example: "Play", "Add", and so on. The content of this attribute will be read by VoiceOver instead of the title of the component if it has one: in this case, it allows to make explicit or specify the title of the component.
* `accessibilityLanguage`: indicates the language used to describe the element. This information is useful for text-to-speech tools so that the appropriate voice is automatically selected.
* `accessibilityTraits`: contains a combination of one or more characteristics that describes the state, behavior, or role of an element.
* `accessibilityHint`: provides a brief description that allows the user to know the consequence of an action on an item.
* `accessibilityValue`: The current value of an item. For example, 30% for a potentiometer to set the volume.
* `accessibilityFrame`: the position of an item on screen.
* `accessibilityElementIsHidden`: indicates whether or not an item should be rendered by VoiceOver.


Accessibility-enabled elements contain these attributes: this information can be provided in the Interface Builder when creating the interface or by the programmer in the Objective C code. Attributes `accessibilityLabel` and `accessibilityFrame` are always required. The `accessibilityValue` attribute is useful when the content of an element is editable and cannot be described by `accessibilityLabel`.

## Use the basic elements provided by UIKit

UIKit provides basic elements (widgets) that correctly implement accessibility. Using them as a first choice is recommended, while defining new ones only when required.

We provide below lists of elements that are either compatible or not compatible with accessibility tools. They have been tested with iOS 6, 7, 8 and 9.


### Compatible Widgets

Views:

* UIAlertView
* UIPickerView
* UIScrollView
* UITableView
* UIWebView

Components:

* UIButton
* UIImage
* UILabel
* UISegmentedControl
* UISlider
* UISwitch
* UITextField

### Incompatible widgets
On the other hand, the following widgets are not directly compatible with the accessibility tools. They will need to be adapted on a case-by-case basis:

* CollectionView
* UIPageControl
* MapKitView

## Test for accessibility

In the iOS API, the accessibility features have been "stabilized" since version 5 (there is no major change in the API). However, accessibility tools (VoiceOver in particular) are evolving rapidly and sometimes change their interpretation of certain interface elements or accessibility notifications. The interpretation by the accessibility services of the common widgets is relatively stable: the tests performed did not reveal regressions. On the other hand, the interpretation of custom views may vary, but these variations are extremely complex to document. It is therefore necessary to test the accessibility of an iOS application in order to verify that the recommendations presented below are taken into account by the accessibility services (see [Mobile Application Audit Guide](https://github.com/DISIC/guide-mobile_app_audit/tree/english)).

## Declare the accessibility of views

It is up to the developer to ensure the accessibility of the custom views he creates. There are two situations: individual views and container views.

### Individual views
These are views that contain a single element with which the user can interact. It is necessary to declare these views as accessible. Several ways are available: either in the Interface Builder or in the Objective C code of the application.

A first possibility is to declare the `setIsAccessibilityElement` attribute when instantiating the view:
```
  @implementation CustomViewController
  - (id)init
   {
     _view = [[[CustomView alloc] initWithFrame:CGRectZero] autorelease];
     [_view setIsAccessibilityElement:YES];
     ...
   }
```

Another option is to implement the `isAccessibilityElement` method of the `UIAccessibility` protocol:
```
  @implementation CustomView
  - (BOOL)isAccessibilityElement
   {
      return YES;
   }
```

### The "container" views

The "container" views group together other views that constitute the elements of the interface with which the user interacts. It is these aggregated items that must be declared as accessible: the "container" view must not be declared as accessible, but must declare the list of accessible items it contains. To do this, use the `UIAccessibilityContainer` protocol.

Declare the list of accessible items contained in the view using `accessibleElements`:
```
  - (NSArray *)accessibleElements
  {
     if ( _accessibleElements != nil )
     {
        return _accessibleElements;
     }
     _accessibleElements = [[NSMutableArray alloc] init];
     /* Creates a first element and initializes it as contained in the view */
     UIAccessibilityElement *element1 = [[[UIAccessibilityElement alloc] initWithAccessibilityContainer:self] autorelease];
     /* Adds attributes to this element */
     [_accessibleElements addObject:element1];
     /* Creates a second element... */
     UIAccessibilityElement *element2 = [[[UIAccessibilityElement alloc] initWithAccessibilityContainer:self] autorelease];
     /* Adds attributes */
     [_accessibleElements addObject:element2];
     /* etc... */
     ...
     return _accessibleElements;
  }
```
Then declare that this view is not accessible:
```
  - (BOOL)isAccessibilityElement
  {
    return NO;
  }
```

Finally, implement the `accessibilityElementCount`, `accessibilityElementAtIndex` and `indexOfAccessibilityElement` methods of the `UIAccessibilityContainer protocol:
```
  - (NSInteger)accessibilityElementCount
  {
     return [[self accessibleElements] count];
  }

  - (id)accessibilityElementAtIndex:(NSInteger)index
  {
     return [[self accessibleElements] objectAtIndex:index];
  }

  - (NSInteger)indexOfAccessibilityElement:(id)element
  {
     return [[self accessibleElements] indexOfObject:element];
  }
```

## Describe interface elements
Many elements of a user interface provide information about their use or meaning through visual indications. For example, a note-taking application uses an element containing the image of a "plus" sign to indicate that the user can add a new note. An input field may have an indication in the form of an image, making it possible to specify the information that the user must provide. A blind user cannot perceive these visual indications. It is therefore necessary to use a means to add an alternative to these indications. To do this, you must use the attributes `accessibilityLabel` and `accessibilityHint` in the definition of an interface element. Assistive software use these attributes as a basis to communicate to users the information needed to understand interface elements. When the human language of descriptions changes, it is necessary to use the `accessibilityLanguage` attribute to ensure that the text is correctly pronounced by VoiceOver.

Example:
```
  [e1 setAccessibilityLabel:@"Open"];
  [e2 setAccessibilityLabel:@"Ouvrir"];
  [e2 setAccessibilityLanguage:@”fr”];
```

NB: proper nouns, words of foreign origin officially accepted in the reference dictionary, and words of foreign origin commonly used (e.g.: "hasta la vista"), should not be subject to language change.


## Identify interface elements traits

The traits allow to provide precise information to the assistance tools on the behavior and the role of the interface elements.

The basic elements of UIKit are already set up with the appropriate traits. For customized item or custom views, it is necessary to declare their traits. An element can have one or more traits that will be combined with the `OR` operator.

The Accessibility API offers two categories of traits:

* Those that describe the type of an element (button, image, etc.)
* Those that define the behavior of an element (search field, play a sound, etc.)

Traits provided by the API include:

* `UIAccessibilityTraitNone`: The element has no defined behavior. This feature removes all others (it cannot be used in conjunction with others).
* `UIAccessibilityTraitButton`: The element should be considered as a button. The word "button" will be pronounced by VoiceOver after the "label" of the element.
* `UIAccessibilityTraitLink`: the item should be considered as a link. The word "link" will be pronounced by VoiceOver after the "label" of the item. This trait should be used instead of `UIAccessibilityTraitButton` for actions that open a page in a browser.
* `UIAccessibilityTraitSearchField`: the element should be considered as an input field for searching. This trait applies to searches restricted to the context of the application. For example, the input field of a web search engine should not have this trait.
* `UIAccessibilityTraitImage`: the element must be considered as an image. The word "image" will be pronounced by VoiceOver after the "label" of the element. This trait can be combined with others (`UIAccessibilityTraitButton` or` UIAccessibilityTraitLink` for example). `UIAccessibilityTraitImage` must be used when the appearance of the image conveys information that is essential for understanding: it should not be used for decorative images. For buttons that are embedded as an image, but whose appearance does not provide essential information to understand their purpose, only `UIAccessibilityTraitButton` should be used.
* `UIAccessibilityTraitSelected`: the item should be considered as selected. The word "selected" will be pronounced by VoiceOver after the "label" of the item. For example, this feature can be used to indicate the state of a panel in a tab system.
* `UIAccessibilityTraitPlaysSound`: the item allows to start playing a sound.
* `UIAccessibilityTraitKeyboardKey`: the element behaves like a keyboard key.
* `UIAccessibilityTraitStaticText`: the element must be considered as static text, which cannot be modified.
* `UIAccessibilityTraitSummaryElement`: represents a summary of the current state of a rendered element when the application starts. (NB: the focus is not placed on this element; the summary is only rendered once when the application is opened, but not every time the interface is displayed again). This trait can be used to draw the attention of the VoiceOver user to an important element of the home screen that does not immediately appear in the focus order.
* `UIAccessibilityTraitNotEnabled`: indicates that an item is disabled or is not intended to respond to interactions. This characteristic can be used, for example, in the case where a button appears in the interface but whose operation is temporarily deactivated. `UIAccessibilityTraitNotEnabled` can be combined with other traits (such as `UIAccessibilityTraitButton`).
* `UIAccessibilityTraitUpdatesFrequently`: indicates that the content of the item is updated too frequently to make it relevant to send notifications to assistive software every time. This prevents "saturation" of content rendered by VoiceOver when an item changes very quickly (e.g. content displayed by a stopwatch).
* `UIAccessibilityTraitStartsMediaSession`: indicates that the item allows to start playback of multimedia content, and tell VoiceOver that it should not output speech (media playback can be started without being disrupted by VoiceOver's speech output).
* `UIAccessibilityTraitAdjustable`: indicates that the value of an item (such as a slider) can be adjusted. The word "adjustable" will be spoken out by VoiceOver after the label, as well as the message "swipe up or down to change the value". Using this trait causes a change in the default behavior of VoiceOver, which redefines upward and downward "swipe" gestures to adjust the element's value: the default action for these gestures (for example navigation by titles) is therefore no longer available in this context.
* `UIAccessibilityTraitAllowsDirectInteraction`: represents an element with which the user can interact directly via the touchscreen. When this trait is enabled, VoiceOver no longer interprets the user's actions and leaves this interpretation to the application.
* `UIAccessibilityTraitCausesPageTurn`: represents an element that automatically moves to the next page when assistive tools reach the end of the content. A use case of this trait is, for example, a digital book reading application, to automate the passage from one page to the next when VoiceOver renders the text, without the user having to manually intervene.
* `UIAccessibilityTraitHeader`: indicates that the element with this trait is used to divide content (title, navigation bar, etc.). It allows VoiceOver to facilitate navigation within the application.

Characteristics can be set when initializing a view:
```
   - (id)init
   {
     _view = [[CustomView alloc] initWithFrame:CGRectZero];
     ...
     [_view setAccessibilityTraits:UIAccessibilityTraitButton | UIAccessibilityTraitPlaysSound];
     ...
   }
```

The traits can also be initialized by defining the `accessibilityTraits` methods:
```
   - (UIAccessibilityTraits)accessibilityTraits
   {
      return UIAccessibilityTraitButton | UIAccessibilityTraitPlaysSound;
   }
```

## Frame the rendering area of ​​an item

It is possible to frame the rendering area of a component, i.e. the area within which a touch interaction will trigger the rendering of the component's content by VoiceOver. This can be useful to limit the risk of mistaking the element from neighboring elements during exploration by touch. To do this, use the `setAccessibilityFrame` function.

Example:
```
  [_c setAccessibilityFrame:CGRectMake(10,10,130,130)];
```

## Update descriptions and traits

When the interface elements of an application change, it is necessary to update their descriptions and traits. Methods for specifying attributes useful for accessibility (`accessibilityLabel`, `accessibilityTraits`, etc.) must therefore return values ​​according to the precise context of the element and not "initialized once and for all".

## Get information about the status of accessibility options

iOS provides methods for retrieving information about the status of certain accessibility options to adjust the behavior of the app accordingly.

For example:

* `UIAccessibilityIsVoiceOverRunning`: indicates whether VoiceOver is active.
* `UIAccessibilityIsMonoAudioEnabled`: indicates whether the audio system is in mono mode.
* `UIAccessibilityIsClosedCaptioningEnabled`: indicates whether closed captioning is enabled.



## Send notifications to assistive tools

When an interface element changes, it is necessary to send a notification so that the assistive tools are notified of the change. Notifications must be sent when a "significant" change occurs: avoid overloading the assistive tools with information that they would not be able to render in real time.

Sending notifications to assistive tools is done using the `UIAccessibilityPostNotification` method.

For example, to notify the change of an interface item:
```
  UIAccessibilityPostNotification(UIAccessibilityLayoutChangedNotification, nil);
```

`UIAccessibility` provides notifications that can be sent by applications, including:

* `UIAccessibilityLayoutChangedNotification`: allows to send a notification to be rendered by the assistive tools (a message can be passed via a string as a parameter). This notification must be used to notify VoiceOver users of the change of an item in the interface.
* `UIAccessibilityAnnouncementNotification`: allows to send a notification to be rendered by the assistive tools (a message can be passed via a string as a parameter). This notification must be used to notify users of an event that has not changed an interface element or that has changed it for a short time.
* `UIAccessibilityPageScrolledNotification`: provides information about the screen content after the user has scrolled with VoiceOver.
* `UIAccessibilityPauseAssistiveTechnologyNotification` and `UIAccessibilityResumeAssistiveTechnologyNotification`: allows to temporarily pause and resume the actions performed by the assistive tools specified in parameters.
* `UIAccessibilityScreenChangedNotification`: allows to notify assistive tools when a new view appears that takes up an important part of the screen. A string, that will be rendered by the assistive tools, or an interface element where the assistive tool's focus is, can be passed as a parameter.

## Capture and process notifications sent by UIKit

UIKit sends accessibility notifications that can be handled to improve the behavior of the application. Capturing these notifications allows you to customize the behavior of the application for support tool users.

These notifications can be captured via the notification center. For example, to capture the notification `UIAccessibilityVoiceOverStatusChanged`:
```
[[NSNotificationCenter defaultCenter]
    addObserver:self
    selector:@selector(updateUserInterfaceForVoiceOver:)
    name:UIAccessibilityVoiceOverStatusChanged
    object:nil];
```

Accessibility notifications provided by the API include:

* `UIAccessibilityVoiceOverStatusChanged`: sent by UIKit when VoiceOver starts or is stopped.
* `UIAccessibilityAnnouncementDidFinishNotification`: sent by UIKit when the rendering of a notification through assistive tools is complete. This notification returns a "dictionary" containing two keys, `UIAccessibilityAnnouncementKeyStringValue` and `UIAccessibilityAnnouncementKeyWasSuccessful`, which contain the message that was rendered and whether or not the restitution was successful.

## Some particular cases

### Modals

UIKit views that allow the implementation of modals (`UIAlertView` for example) are accessible: they allow the user of assistive tools to not interact with content outside of the view. For custom views, use the `accessibilityViewIsModal` attribute: when this attribute is set to 'YES', the elements inside the neighboring views will be ignored by the assistive tools.

### Table views (TableView)

The cells in a table view are automatically considered as containers: so it is not necessary to implement the `UIAccessibilityContainer` protocol to declare the list of contained elements.
In order to facilitate the rendering by the assistive tools, it may be useful to group the information contained in several cells into a single description, in order to minimize the number of operations that the user must perform to access it. For example, it is appropriate to group the title of a product, its weight and price in a text that will be returned to the user at once, without the user having to make gestures to access each of the elements separately.

The `accessibilityLabel` method can then be defined in this way:
```
  - (NSString *)accessibilityLabel
  {
    NSString *prodNameText = [self.prodName accessibilityLabel];
    NSString *prodWeightText = [self.prodWeight accessibilityLabel];
    NSString *prodPriceText = [self.prodPrice accessibilityLabel];

    return [NSString stringWithFormat:@"%@, %@, %@", prodNameText, prodWeightText, prodPriceText];
}
```


## Related documents

The following guides can be consulted in addition:

* [Mobile Application Audit Guide](https://github.com/DISIC/guide-mobile_app_audit/tree/english)
* [Accessible Mobile Application Design Guide](https://github.com/DISIC/guide-mobile_app_conception/tree/english)
* [Mobile Application Development Guide accessible with Ionic and OnsenUI](https://github.com/DISIC/guide-mobile_app_dev_hybride/tree/english)


## External resources and references

Sources:

* <a href="https://developer.apple.com/reference/uikit/">UIKit Framework Reference</a>
* <a href="https://developer.apple.com/accessibility/ios/">Accessibility for iOS Developers</a>
* <a href="http://www.bbc.co.uk/guidelines/futuremedia/accessibility/mobile">BBC Mobile Accessibility Guidelines</a>

## Licence

This document is the property of the <span lang="fr">Secrétariat général à la modernisation de l'action publique</span> (SGMAP). It is placed under [Open Licence 1.0 or later (PDF, 541 kb)](http://ddata.over-blog.com/xxxyyy/4/37/99/26/licence/Licence-Ouverte-Open-Licence-ENG.pdf), equivalent to a Creative Commons BY licence. To indicate authorship, add a link to the original version of the document available on the [DINSIC's GitHub account](https://github.com/DISIC).