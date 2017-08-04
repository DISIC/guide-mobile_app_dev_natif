# Mobile application development guide with the Android API

## Table of contents
<!-- MarkdownTOC depth="3" -->

- [Who is this Guide for?](#who-is-this-guide-for)
- [Generalities on Android and Accessibility](#generalities-on-android-and-accessibility)
- [Use the basic elements provided by the Android API](#use-the-basic-elements-provided-by-the-android-api)
  - [Compatible Widgets](#compatible-widgets)
  - [Incompatible Widgets](#incompatible-widgets)
- [Test Accessibility](#test-accessibility)
- [Describe interface elements](#describe-interface-elements)
  - [Static creation examples](#static-creation-examples)
    - [ImageView and ImageButton](#imageview-and-imagebutton)
    - [CheckBox](#checkbox)
    - [EditText](#edittext)
  - [Dynamic content description update](#dynamic-content-description-update)
- [Allow font size changes](#allow-font-size-changes)
- [Focus management](#focus-management)
  - [Turn on focus](#turn-on-focus)
  - [Check reading order](#check-reading-order)
- [Display focus](#display-focus)
- [Hide decorative elements](#hide-decorative-elements)
- [Force restitution of important non-focusable elements](#force-restitution-of-important-non-focusable-elements)
- [Items grouping](#items-grouping)
- [Manage accessibility-related events](#manage-accessibility-related-events)
  - [AccessibilityEvent](#accessibilityevent)
  - [Event Traversal](#event-traversal)
  - [Retrieve more information about the context with AccessibilityNodeInfo](#retrieve-more-information-about-the-context-with-accessibilitynodeinfo)
- [Create accessible custom views](#create-accessible-custom-views)
  - [Handling navigation with directional controllers](#handling-navigation-with-directional-controllers)
  - [Implementing methods for accessibility](#implementing-methods-for-accessibility)
    - [onInitializeAccessibilityEvent\(\)](#oninitializeaccessibilityevent)
    - [onInitializeAccessibilityNodeInfo\(\)](#oninitializeaccessibilitynodeinfo)
    - [onPopulateAccessibilityEvent\(\)](#onpopulateaccessibilityevent)
  - [Handling custom gestures](#handling-custom-gestures)
- [To go further](#to-go-further)
- [Related documents](#related-documents)
- [External resources and references](#external-resources-and-references)
- [Licence](#licence)

<!-- /MarkdownTOC -->




## Who is this Guide for?

This guide presents elements of the Android APIs useful for developing applications accessible to people with disabilities. It is aimed at:

* Developers
* Designers responsible for drafting technical specifications
* Project Managers


Prerequisites:

* Proficiency in Java and XML
* Proficiency in Android programming principles
* Knowledge of the basic classes of the Android environment (including "activity" and "service")
* Mastery of events management
* Mastery of the concepts used to manage the user interfaces on Android (views, layouts, etc.) and the main elements of the API (TextView, CheckBox, RadioButton, etc.)


## Generalities on Android and Accessibility

The Android API provides developers with features to create accessible applications. These features are present in the classes that allow the creation of the basic elements of a user interface as well as in classes dedicated to accessibility.

Android applications can be used by people with disabilities (visual, motor, etc.), by activating services dedicated to accessibility on the device used to run the application; and sometimes, by using external input devices (keyboards, switches, braille ranges, etc.). Accessibility services can be activated in the settings of the device in the "Accessibility" tab.

Google develops several services:

* The TalkBack screen reader, which vocalizes the user interface. It offers several features:
  * Exploring by touch: the user explores the content of the application by browsing the screen with her fingers. TalkBack drives a speech synthesis software that pronounces the information about the elements under the user's fingers. In this mode, TalkBack redefines the "standard" gestures: the activation of an element is performed, for example, by a "double tap".
  * Simplified gesture navigation: TalkBack redefines gestures allowing the user to easily navigate between the various elements of the interface.
  * Haptic feedback: when interface elements are activated (action in an input area for example), TalkBack informs the user through vibrations.
* BrailleBack, which allows users with Braille displays, connected through USB or Bluetooth to their Android device, to read and interact with the content of applications.

Some manufacturers offer modified versions of the Android system; similarly, there are modified versions of accessibility services: for example, some Samsung phones that come with the Galaxy TalkBack service, a version of the screen reader based on the one developed by Google.

The mere activation of accessibility services is not enough to make an application fully accessible: the developer must implement particular techniques when coding the application, to make its content accessible to accessibility services.

When the user interface of an application is made up of the basic elements provided by the Android framework, the implementation of accessibility is relatively simple. The approach is:

1. Describe the interface elements for their content to be rendered by accessibility services;
2. Ensure that all elements that call for interaction can be reached by a directional controller (trackball, D-pad, etc.);
3. Ensure that audio messages are accompanied by visual and/or haptic notifications to be perceived by persons who are deaf or hard of hearing.

In order to develop more complex and customized interfaces that require to extend the View class, it will be necessary to ensure that these elements are compatible with accessibility services by redefining certain methods, in particular to manage appropriately the accessibility-related events.

## Use the basic elements provided by the Android API

The Android API provides basic elements (widgets, views) that correctly implement accessibility in most cases. Using them as a first choice is recommended, while defining new ones only when required.

We provide below lists of elements that are either compatible or not compatible with accessibility services. They have been tested with versions of Android 4.3, 4.4 and 5.1.

### Compatible Widgets

The following  widgets  have been tested and can be used to create interfaces that are compatible with accessibility services (as long as content is described and focus correctly managed, as mentioned previously).

* EditText
* CheckBox
* ToggleButton
* Switch
* RadioGroup and RadioButton
* Chronometer
* TextClock
* AutoCompleteTextView
* Button
* ProgressBar
* RatingBar
* SeekBar
* NumberPicker
* TextSwitcher


Layouts:

* LinearLayout
* TableLayout

Views:

* TextView
* ImageView
* ListView
* SearchView
* WebView

### Incompatible Widgets

On the other hand, the following widgets  are not compatible with accessibility services. They will need to be adapted on a case-by-case basis:

* TabHost: The contents of the tabs are not rendered correctly by accessibility services
* DatePicker
* TimePicker
* Spinner
* ToolBar

Views:

* CalendarView
* ExpandableListView
* StackView

## Test Accessibility

In the Android API, the accessibility features have been "stabilized" since version 4.0 (there is no major change in the API). However, accessibility services (TalkBack in particular) are changing rapidly and sometimes changing their interpretation of certain interface elements or accessibility-related events. For example, version 4.3.1 of TalkBack published in October 2015 is able to render changes in values ​​from "sliders", which was not the case in previous versions (it was necessary to use the `contentDescription` attribute to output the content). The interpretation by the accessibility services of the common widgets is relatively stable: the tests performed did not reveal regressions. On the other hand, the interpretation of custom views may vary, but these variations are extremely complex to document. In addition, some hardware manufacturers may add overlays to the Android system, which sometimes has an impact on accessibility. It is therefore necessary to test the accessibility of an Android application to verify that the recommendations presented below are taken into account by the accessibility services (see [Mobile Application Audit Guide](https://github.com/DISIC/guide-mobile_app_audit/tree/english)).

## Describe interface elements

Many elements of a user interface provide information about their use or meaning through visual indications. For example, a note-taking application uses an `ImageButton` element containing the image of a plus sign to indicate that the user can add a new note. An `EditText` component can have a label to specify the information that the user needs to enter. A blind user cannot  perceive these visual indications. It is therefore necessary to use a means to add a textual equivalent of these indications. To do this, use the `android:contentDescription` attribute in the XML description of an interface element or the `setContentDescription`  method in the Java code. The text in this description will not be displayed on screen, but will be processed by accessibility services when the user has enabled them. With TalkBack, the text will be spoken out by the speech synthesizer; with BrailleBack, the text will appear near the item on the Braille display.

### Static creation examples

#### ImageView and ImageButton

`ImageView` displays an image on the screen. `ImageButton` is a subclass of `ImageView` that adds the behaviors of a "standard" button. These classes have an `android:src` attribute that points to a graphical resource corresponding to the image that will be displayed and used to represent the button in the case of `ImageButton`. Because this graphical resource cannot  be interpreted by accessibility services, the `android:contentDescription` attribute must be used to provide a textual alternative to this image.

Example with `ImageButton`:
```xml
  <ImageButton
    android:id="@+id/imageButton2"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:src="@drawable/ic_launcher"
    android:contentDescription="Add a note">
```

Expected behavior with TalkBack: the content of the `android:contentDescription` attribute, preceded by the word 'button', are spoken out when taking focus, or hovering the image.


#### CheckBox

`CheckBox` allows you to implement a check box.

```xml
  <CheckBox android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:id="@+id/checkBox"
    android:text="Recommend"
    contentDescription="Recommend peonies"/>
```

The `android:text` attribute contains the text displayed on screen. It may be relevant to add context elements in the `android:contentDescription` attribute to provide the user with details about what action will be taken when checking or unchecking the box. For example, a view may have several checkboxes next to a list of products, some of which the user can recommend. If only the "android:text" attribute is populated with the "Recommend" text, the user will not have enough information to know which action corresponds with ticking the box. Adding "Recommend peonies" in the  `android:contentDescription` attribute will provide enough information when using an accessibility service.

Expected behavior with TalkBack: speech output of the box current state, followed by "Checkbox" and then the contents of the description.

#### EditText

`EditText` allows to create a text input area. For `EditText`, unlike most other  widgets  available in the  Android framework, the `android:contentDescription` attribute cannot  be used to convey information describing the element (the content of the attribute will not be rendered by the accessibility services).

The use of `android:hint` allows the user to understand the nature of the expected information: the content of `android:hint` is an input help displayed when the input field is empty, and when the area contains input text, the screen reader returns the entered text instead of the attribute.
```xml
  <EditText android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:id="@+id/editText"
    android:hint="e-mail address" />
```

However, the content of `android:hint` disappears as soon as the user enters a character and cannot  be restored, even if the area is emptied: this causes difficulties especially when several input fields are proposed in a single view. The best way to ensure the accessibility of an input field is to associate a visible label with the `android:labelFor` attribute.

In XML:
```xml
  <TextView
    android:labelFor="Zip Code"
    .../>

  <EditText android:id="@+id/edit_text"/>
```

In Java, the `setLabelFor(View label)` or `setLabeledBy(View label)` functions can be used.

Expected behavior with TalkBack: 

* The content of `android:hint` is rendered only when no text has been entered; when a text has been entered, it is output by TalkBack; when a text has been entered and then deleted, TalkBack only indicates the presence of the input field without rendering the contents of `android:hint`.
* The content of `android:labelFor` is rendered in any context.

### Dynamic content description update

When the content of an element changes over time, it is necessary to be careful to update its description, in particular when the state of the element cannot  be interpreted by any other means than this description. To do this, use the `setContentDescription` method in the Java code of the application. For example, dynamic updating of the description is useful in the following cases when creating custom  widgets:

* Color selection in a palette: the colors must be identified by a textual description and the description indicating the selected color must be updated so that the user knows it when the widget receives focus back; 
* Date selection: the description can be used to recall the entire date selected after the user has updated one of the items (day, month, year);
* To indicate the change of function of an image button when its meaning is conveyed by shape or color: a "Play" button changed to "Pause" for example.

A method for updating the description is to use  listeners  to monitor the change of state of a widget, as to take the necessary measures at the appropriate time.

The example below uses a `SeekBar`, a  widget meant to enter a value picked within a specified interval. TalkBack, in versions lower than 4.3.1, did not monitor the state change of the `SeekBar` and was therefore not able to inform the user of changes in values. This behavior was identical with BrailleBack. The proposed code monitors the modification of the `SeekBar` and dynamically updates the description provided to the accessibility services when the value changes. Thus, the value will be indicated to the user after each change, which would not have been the case by default.
NB: this code is given as an example. With TalkBack 4.3.1, its use causes a double restitution when changing the value of the `SeekBar`. It should therefore be considered as an illustration of the principle of dynamic updating of the description, but its use is not necessarily relevant in all contexts.

XML description:
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="fill_parent"
        android:layout_height="fill_parent"
        android:orientation="vertical">

  <SeekBar android:layout_width="fill_parent"
     android:layout_height="wrap_content"
     android:id="@+id/reading_speed"
     android:max="10"/>
</LinearLayout>
```


Java code:
```java
  package com.braillenet.android_accessibility;

  import android.app.Activity;
  import android.os.Bundle;
  import android.widget.SeekBar;

  public class ProgressBarActivity extends Activity {

    private SeekBar mReadingSpeed;
    String name;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);
      setContentView(R.layout.progressbar);
      mReadingSpeed = (SeekBar)findViewById(R.id.reading_speed);
      name = "Reading speed";
      mReadingSpeed.setOnSeekBarChangeListener(new SeekBarListener(name));
      mReadingSpeed.setContentDescription(name);
    }

    private class SeekBarListener implements SeekBar.OnSeekBarChangeListener {
      private String mName;
      public SeekBarListener(String name) {
        mName = name;
      }
      public void onProgressChanged(SeekBar seekBar, int progress,
        boolean fromUser) {
        int percent = 100 * seekBar.getProgress() / seekBar.getMax();
        seekBar.setContentDescription(mName + " " + percent + "%");
      }
      public void onStartTrackingTouch(SeekBar seekBar) {
      }
      public void onStopTrackingTouch(SeekBar seekBar) {
        seekBar.setContentDescription(mName);
      }
    }
  }
```

Expected behavior with TalkBack: When the value is changed (you can use the "+" and "-" volume buttons for this), TalkBack speaks out "Reading speed x%".

## Allow font size changes

Android allows the user to set the size of characters (Settings menu, Display, then Font size: Small, Normal, Large, etc.). For an application to correctly respond to this setting, it is necessary to use the "sp" (scale-independent pixel) unit when defining text sizes.
Example: `<TextView ... android:textSize="12sp"/>`

It is possible to know the user-selected adjustment factor applied to the characters' size, which makes it possible to adapt the behavior of the application accordingly (modify the number of columns of an array for example). To do this, access the `Settings.System.FONT_SCALE` value from the `Settings.System.CONTENT_URI` table. (1.0f is the value corresponding to the "normal" size).

Example:
```java
  ContentResolver r = getContentResolver();
  Cursor c = r.query(Settings.System.CONTENT_URI,
                     new String[] { Settings.System.VALUE },
         Settings.System.NAME + "= ?",
         new String[] { Settings.System.FONT_SCALE }, null);
  float fs = c.getFloat(0);
  if(fs ...) {
    ...
  } else {
    ...
  }
```

## Focus management

It is necessary to allow users to navigate between the various elements of the interface through a directional controller. Such a controller can be a piece of hardware (trackball, D-Pad, keyboard arrows, etc.), or virtual, such as the on-screen keyboard provided by an application, or the gestural navigation mode available since Android 4.1. Navigation using directional controllers is widely used, especially by people with motor disabilities.

To ensure that such navigation works, it is necessary to verify that all the elements of an application are reachable without using tactile interactions. It is also necessary to make sure that clicking the center button (or the OK button) on a directional controller has the same effect as touching a focused item on the screen.

### Turn on focus

An element of a user interface is reachable by using a directional controller when the `android:focusable` attribute is set to 'true': this allows the user to set focus on that element and interact with the content. The basic elements provided by the Android framework  are focusable by default. Whether or not an element has focus is indicated by a change in its appearance.
The Android API provides several methods for controlling that an item is actually focusable, as well as setting focus on an element:

* `setFocusable()`: indicates that the element will be able to get focus
* `isFocusable()`: indicates whether an element can receive focus or not
* `requestFocus()`: sets focus to an element

If a view cannot  receive the focus by default, you can use the `android:focusable` attribute with the value 'true' or use the `setFocusable()` method to change this behavior.

### Check reading order

When a user navigates an application using a directional controller, the focus moves from one element of the interface to the other in a sequence determined by an algorithm that searches for the closest element in the desired direction. In some cases, the order calculated by this algorithm may not be consistent with the logic of the application. To work around this problem, it is possible to define relationships between the different elements. The `android:nextFocusDown` (`android:nextFocusUp`, `android:nextFocusLeft` and `android:nextFocusRight`) attributes let you define the next element that will receive focus when the user moves down (up, left and right, respectively).

Example:
```xml
  <LinearLayout android:orientation="horizontal"
        ... >
    <EditText android:id="@+id/edit"
      android:nextFocusDown="@+id/text" 
      ... />
    <TextView android:id="@+id/text"
      android:focusable="true" 
      android:text="I am a focusable TextView"
      android:nextFocusUp="@id/edit" 
      ... />
  </LinearLayout>
```

This technique may for example be applied to make the focus describe a circle around a set of elements constituting a clock. At 12, the desired behavior is to move to 1 by pressing "right" or "down", and to 11  by pressing "left" or "down". We will have these types of relations:
```xml
  <Button
    android:text="12"
    android:id="@+id/button12h"
    android:nextFocusUp="@+id/button11h"
    android:nextFocusLeft="@+id/button11h"
    android:nextFocusRight="@+id/button1h"
    android:nextFocusDown="@+id/button1h"
  </Button>
  <Button
    android:text="1"
    android:id="@+id/button1h"
    android:layout_below="@+id/button12h"
    android:layout_toRightOf="@+id/button12h"
    android:nextFocusUp="@+id/button12h"
    android:nextFocusLeft="@+id/button12h"
    android:nextFocusRight="@+id/button2h"
    android:nextFocusDown="@+id/button2h">
  </Button>
  ...
  <Button
    android:text="11h"
    android:id="@+id/button11h"
    android:layout_below="@+id/button12h"
    android:layout_toLeftOf="@+id/button12h"
    android:nextFocusUp="@+id/button10h"
    android:nextFocusLeft="@+id/button10h"
    android:nextFocusRight="@+id/button12h"
    android:nextFocusDown="@+id/button12h">
  </Button>
```

## Display focus

It is necessary to indicate visually which element has the focus. To do this, you must create a `drawable` and assign the `android:drawable` attribute for `state_focusable="true"`.

Example:
```xml
  <selector xmlns:android="http://schemas.android.com/apk/res/android">
    ...
    <item android:state_focused="true" android:state_enabled="true" android:drawable="@drawable/button_is_focused" />
  </selector>
```


## Hide decorative elements

It is necessary to hide the decorative elements (e.g. an image that does not provide additional information in the context in which it appears) so that they are not rendered by TalkBack. To do this, use the `android:importantForAccessibility="no"` attribute.

## Force restitution of important non-focusable elements

Some non-focusable elements, such as non-clickable images, are not rendered by TalkBack. This behavior is problematic if these elements are not decorative (i.e. they contain information essential for comprehension). It is then necessary to force their rendering by the accessibility services using the attribute `android:importantForAccessibility="yes"`.

## Items grouping

To make the output by assistive technologies more meaningful, it may be useful to group the information contained in several elements into a single description, in order to minimize the number of operations that the user must perform to access it. For example, it is appropriate to group the title of a product, its weight and price into a text that will be returned to the user in one go.

To do this, you need to create a parent layout with the attribute `android:importantForAccessibility="yes"`, to associate children to it, that will have the  `android:importantForAccessibility="no"` attribute, and to place the description of the grouped elements in the `android:contentDescription` attribute of the parent layout.

## Manage accessibility-related events

In the Android framework core elements, when content is selected, when the focus moves, or an item is hovered, events of the  `AccessibilityEvent` type are triggered. These events are caught and interpreted by the accessibility services to provide vocal feedback to users, for example.

Generating events can be useful when designing a user interface with custom views; catching events will allow accessibility services to interpret the behavior of an application.

### AccessibilityEvent

An event can be triggered using the `sendAccessibilityEvent(int)` method, with a parameter representing the type of event that occurred.
The complete list of event types is available in the  <a href="https://developer.android.com/reference/android/view/accessibility/AccessibilityEvent.html">AccessibilityEvent</a> class description. Examples:

* `TYPE_VIEW_FOCUSED`: triggered when the focus is set on a view
* `TYPE_VIEW_CLICKED`: triggered when clicking on a view (`Button`, `CompoundButton`, etc.)
* `TYPE_VIEW_TEXT_CHANGED`: triggered when text is changed in an edit box (`EditText`)
* `TYPE_ANNOUNCEMENT`: "generic" event that can be used when no other type of event is appropriate, to indicate a change of context or to send specific information to be rendered by the accessibility services. Note that the `View` class provides the `announceForAccessibility(CharSequence text)` method, which is a simple way to send this event.

Each event has properties that can be accessed through different methods when it is intercepted by an accessibility service. For example:

* `getClassName()`: the name of the class from which the event was issued
* `getPackageName()`: the name of the package from which the event was issued
* `getContentDescription()`: the contents of the source `contentDescription` attribute
* `isPassword()`: if the source is a field intended to receive a password

Other methods allow you to initialize or modify properties of the event source. For example:

* `setContentDescription`: defines the description (corresponds to the information contained in the `android:contentDescription` attribute mentioned above)
* `setPassword`: defines whether an item is intended to receive a password

### Event Traversal

In addition to the `sendAccessibilityEvent` method, the `View` class provides other methods for managing accessibility events:

* `sendAccessibilityEventUnchecked`: this method is used when the calling code directly handles the verification of activation of accessibility-related functions (by calling `AccessibilityManager.isEnabled()`).
* `onPopulateAccessibilityEvent()`: defines the text that will be spoken out when the AccessibilityEvent is triggered for the view concerned.
* `onInitializeAccessibilityEvent()`: method for obtaining additional information about the state of the view.
* `onInitializeAccessibilityNodeInfo()`: provides information to accessibility services about the state of the view.
* `onRequestSendAccessibilityEvent()`: method called when a child of a view triggered an 'AccessibilityEvent`. This allows, for example, the parent view to modify the child's event to add information.

To understand the role of the methods mentioned above, it is necessary to know the traversal of an event:

* When an accessibility service is active and the user performs an action (click, focus, hover, etc.), the view is notified by a call to `sendAccessibilityEvent` (or sendAccessibilityEventUnchecked`) 
* When called, the `sendAccessibilityEvent()` and `sendAccessibilityEventUnchecked()` methods use a  callback  mechanism that uses `onInitializeAccessibilityEvent()` to initialize the event
* Once initialized, if the event type requires that it be propagated with text information, the view receives a call to `dispatchPopulateAccessibilityEvent()`;
* Through a  callback  mechanism, `dispatchPopulateAccessibilityEvent()` calls `onPopulateAccessibilityEvent()`;
* The view then returns the event to the views hierarchy by calling `requestSendAccessibilityEvent()` on the parent view. Each ancestor view then has the ability to add information about accessibility by adding an `AccessibilityRecord` until the event reaches the root view
* Once at the root, the event is sent to the `AccessibilityManager` via `sendAccessibilityEvent()`.

### Retrieve more information about the context with AccessibilityNodeInfo

The main role of an event is to expose enough information to an accessibility service so that it provides relevant information to the user. Sometimes, an accessibility service may need more information than that conveyed by the event. In this case, it is possible to retrieve an object of type `AccessibilityNodeInfo` that represents the state of a view, which allows an accessibility service to explore the contents of the window (for security reasons, access to the source of an event is a privilege that must be explicitly requested by an application).

`AccessibilityNodeInfo` represents a node of the contents of the window as well as the actions that can be requested from the source. There are many ways to explore the contents of the window. For example:

* `findAccessibilityNodeInfosByViewId`: returns a list of elements of type `AccessibilityNodeInfo` corresponding to an identifier
* `findFocus`: finds the view that has the focus
(Refer to the <a href="https://developer.android.com/reference/android/view/accessibility/AccessibilityNodeInfo.html"> AccessibilityNodeInfo</a> class for an exhaustive list.)

## Create accessible custom views

The Android framework   offers a large number of basic components: widgets  (`Button`, `CheckBox`, etc.) and layouts  (`LinearLayout`, `FrameLayout`, etc.). However, it is possible to create custom components, by extending the `View` class: this allows for example to precisely control the appearance and functionality of an element. It is then necessary to ensure that these customized elements are accessible.

### Handling navigation with directional controllers

On most devices, clicking in a view using a directional controller sends a `KeyEvent` of type `KEYCODE_DPAD_CENTER` to the view that has the focus. All basic Android views process this type `KEYCODE_DPAD_CENTER` appropriately. When a custom view is implemented, ensure that this event has the same effect as touching the view on the touchscreen. In addition, it is also necessary to process the `KEYCODE_ENTER` event in the same way as `KEYCODE_DPAD_CENTER`, in order to facilitate keyboard navigation.

For example:
```java
  public boolean onKeyUp(int keyCode, KeyEvent event) {
    if ((keyCode == KeyEvent.KEYCODE_DPAD_CENTER) || (keyCode == KeyEvent.KEYCODE_ENTER)) {
      ...
    }
  }
```

### Implementing methods for accessibility

In order to support accessibility in a custom view, it is necessary to override and implement the methods described at the end of the section "Managing accessibility-related events".

Before giving examples of how to implement these methods to create custom views that are accessible, it is important to understand the order in which accessibility-related events are handled.

#### onInitializeAccessibilityEvent()

This method is called by the system to get information about the state of a view. If a custom component offers additional functionality over a component that it inherits, you should override the onInitializeAccessibilityEvent method and add additional information related to the custom component. To override the method, you must first call the method of the inherited class and then modify properties that were not initialized by the method of the inherited class.

For example, suppose that you want to extend the `BaseToggleButton` class to make sure that the proposed button is checked by default. We will have a piece of code like:

```java
  public static class AccessibleCompoundButtonInheritance extends BaseToggleButton {
    ...
    // This method will be called each time sendAccessibilityEvent() is called
    @Override
    public void onInitializeAccessibilityEvent(AccessibilityEvent event) {
      // The implementation of onInitializeAccessibilityEvent of the inherited class is called
      // then we initialize the "checked" property which is not in the inherited class
      super.onInitializeAccessibilityEvent(event);
      event.setChecked(isChecked());
    }
  }
```

#### onInitializeAccessibilityNodeInfo()

`onInitializeAccessibilityNodeInfo` is used to initialize information about the state of the view that will be used for accessibility services, when they check the source (type `AccessibilityNodeInfo`) of an event. As above, this method must be overridden to provide additional information in addition to that provided by the inherited class.

AccessibilityNodeInfo objects created by this method are used by accessibility services to explore the hierarchy of a view that has generated an accessibility event after it has been received, which allows the accessibility services to get more information about the context of the occurrence of the event.

An example of an implementation of `onInitializeAccessibilityNodeInfo` is:
```java
  public static class AccessibleCompoundButtonInheritance extends BaseToggleButton {
    ...
    @Override
    public void onInitializeAccessibilityNodeInfo(AccessibilityNodeInfo info) {
      super.onInitializeAccessibilityNodeInfo(info);
      // We call the implementation of the inherited class and we add
      // properties: checkable, isChecked, and text
      info.setCheckable(true);
      info.setChecked(isChecked());
      CharSequence text = getText();
      if (!TextUtils.isEmpty(text)) {
        info.setText(text);
      }
    }
  }
```

#### onPopulateAccessibilityEvent()

Each `AccessibilityEvent` has a set of properties that describe the current state of the view. When creating a custom view, it is necessary to provide information about the context and the characteristics of the view: this may be the label of a button, but also additional information that will be provided to the user.

The `onPopulateAccessibilityEvent()` method allows you to provide this additional information about the state of a view, which is called automatically by the system for each `AccessibilityEvent`. This method is therefore the appropriate place to describe the state of a custom view in order to provide specific instructions to the user of accessibility services.

For example, the following code allows the accessibility services to speak out the "drag" message when they set focus on the view:
```java
  public class TestView extends View {
    ...

    // Method called to propagate events related to accessibility.
    public void onPopulateAccessibilityEvent(AccessibilityEvent event) {
      // We start by calling the method of the inherited class
      super.onPopulateAccessibilityEvent(event);
      // We retrieve the event type
      int eventType = event.getEventType();
      // If it corresponds to a capture of focus by an accessibility service...
      if (eventType == AccessibilityEvent.TYPE_VIEW_ACCESSIBILITY_FOCUSED) {
        // ... we modify the event to add text that will be
// spoken out by the service
        event.getText().add("Drag");
      }
    }
  }
```

### Handling custom gestures

Customized views can implement non-standard gestural interactions. This is done by using the `onTouchEvent` method to detect certain types of events. Accessibility services often redefine the basic gestures that allow you to navigate within an application and the actions associated with it. It is therefore necessary to ensure that personalized gesture interactions remain compatible with accessibility services, that they will generate actions that can be interpreted by them, and that these actions can be carried out by means other than the use of the touch screen.

The code that generates the touch events capture must perform the following actions:

1. Generate an appropriate AccessibilityEvent` for the click action
2. Allow accessibility services to perform the action by means other than the touch screen

To do this, it is necessary to override the `performClick()` method which must first execute the method of the inherited class and then perform the actions corresponding to the captured 'click' event. The `performClick()` method must then be called.

For example:
```java
  public boolean onTouchEvent(MotionEvent event) {
    super.onTouchEvent(event);
    switch (event.getAction()) {
      case MotionEvent....:
        ...
  return true;
      case MotionEvent....:
        // capture an interesting event that should trigger a click
  // We perform this click by calling performClick(): this will allow
  // the accessibility services to be aware of it
  performClick();
        return true;
    }
    return false;
  }

  @Override
  public boolean performClick() {
    // We call the implementation of the inherited class that
    // generate an AccessibilityEvent and capture events from
    // click with onClick()
    super.performClick();
    // We then perform the specific actions to be carried out in
    // the custom view
    blabla();
    return true;
  }
```


## To go further

It can happen that implementations based on custom views do not allow accessibility services to retrieve enough context elements to render the interface accurately enough. This is the case, for example, of a set of views for which action on one of the views modifies the content of one or more other views. In this case, an accessibility service cannot  be warned that one action on one view modifies another, because the relationship between the views is not explicit. To work around this issue, it is possible to create "virtual views" that will only be exposed to accessibility services. These virtual views make it possible to better represent the information and behavior of the interface, and to make accessibility services access information relating to the "logic" of the application.
The implementation of a virtual view is complex: it involves extending the `AccessibilityNodeProvider` class. For more details, see <a href="https://github.com/appium/android-apidemos/blob/master/src/io/appium/android/apis/accessibility/AccessibilityNodeProviderActivity.java">this example provided by The Android Open Source Project</a>.

For very specific purposes, "standard" accessibility services such as TalkBack may not be entirely satisfying: the designer of an application may wish to have full control over the way elements of the interface are rendered. For example, this may be the case for applications that need to control multiple text-to-speech tools to distinguish content of different natures (this behavior is not possible with TalkBack): use a "standard" voice to read the user interface elements of the application, and an improved voice to read the contents of a document. The Android API addresses this need by giving the developer the opportunity to create their own accessibility services. The primary role of such a service will be to process and interpret accessibility events. A service has a defined type that determines the feedback it provides to the user: `FEEDBACK_SPOKEN`, `FEEDBACK_BRAILLE`, `FEEDBACK_VISUAL`, and so on. A service can capture all events or filter them according to their type (capture only focus events for example). Finally, it may be limited to a  package  or a set of  packages  to be limited to a specific application, or have a "global" scope to handle all applications in the system. To create a service, it is necessary to extend the `AccessibilityService` class. This is a complex operation and we invite you to check <a href = "https://sites.google.com/site/gdevelopercodelabs/android/accessibility">this example of service implementation</a> proposed by Google for more details.

## Related documents

The following guides can be consulted in addition:

* [Mobile Application Audit Guide](https://github.com/DISIC/guide-mobile_app_audit/tree/english)
* [Accessible Mobile Application Design Guide](https://github.com/DISIC/guide-mobile_app_conception/tree/english)
* [Mobile Application Development Guide accessible with Ionic and OnsenUI](https://github.com/DISIC/guide-mobile_app_dev_hybride/tree/english)


## External resources and references

Sources:

* <a href="https://developer.android.com/reference/packages.html">Android API packages</a>
* <a href="https://developer.android.com/guide/topics/ui/accessibility/apps.html">Making Apps More Accessible</a>
* <a href="https://developer.android.com/training/accessibility/service.html">Developing an Accessibility Service</a>
* <a href="http://www.bbc.co.uk/guidelines/futuremedia/accessibility/mobile">BBC Mobile Accessibility Guidelines</a>

## Licence

This document is the property of the <span lang="fr">Secrétariat général à la modernisation de l'action publique</span> (SGMAP). It is placed under [Open Licence 1.0 or later (PDF, 541 kb)](http://ddata.over-blog.com/xxxyyy/4/37/99/26/licence/Licence-Ouverte-Open-Licence-ENG.pdf), equivalent to a Creative Commons BY licence. To indicate authorship, add a link to the original version of the document available on the [DINSIC's GitHub account](https://github.com/DISIC).